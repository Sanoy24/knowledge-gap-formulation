# Your chosen-side data IS your reward function, and a 16-template reward function cannot generalize

**For:** Amir Ahmedin
**Question:** Given that my 599 chosen outputs collapse to 111 unique strings from a 16-template library in `get_chosen()`, and my eval split is drawn from the same template pool as training, what does this imply for any preference-optimization recipe I run on this data, and what is the minimum structural change to my data pipeline that would make preference training capable of generalizing to the held-out scorer?
**Author:** Yonas Eshete

---

The headline answer: no preference-optimization recipe survives this setup. DPO, SimPO, ORPO, KTO, IPO, all of them push the policy to assign higher probability to chosen than rejected. With 111 unique chosen strings sharing a tight cluster of surface markers ("Hi {name}, to be upfront", "I want to be honest", "I want to be transparent") and 273 unique rejected strings sharing the opposite cluster ("Guaranteed!", "Consider it done", "no problem at all"), the strongest separating feature in your data is the cluster identity, not the rubric your held-out scorer measures. Whatever recipe you run, the policy converges to "produce chosen-cluster surface markers." Your held-out scorer in `evaluation/scoring_evaluator.py` does not reward those markers. It checks bench honesty, timezone correctness, banned phrases. Those are different objectives. So the trained model loses to the prompted base, and switching from SimPO to DPO or raising the LoRA rank does not change that.

## The mechanism: chosen-side data is your implicit reward function

Think about what the SimPO loss is actually doing. The loss is `−log σ(β · (1/|y_w|·log π(y_w|x) − 1/|y_l|·log π(y_l|x)) − γ)`. It pushes the length-normalized log-prob margin between chosen and rejected past γ. The signal it has access to is "this string is chosen, that string is rejected." Anything that separates the two clusters becomes the gradient direction. If your chosen and rejected differ on factual content (one says "we have engineers," the other lies about capacity), the gradient learns factual content. If they differ on surface phrasing (one says "to be upfront," the other says "Guaranteed"), the gradient learns surface phrasing.

You wrote the chosen and rejected templates yourself in `prepare_training_data.py`. The TONE_PASSING list has four templates. RESOURCE_HONESTY_PASSING has three. SIGNAL_GROUNDING_PASSING has three. WORKFLOW_PASSING has six. Every chosen string is one of those plus a name interpolation. The strongest, lowest-entropy separator between chosen and rejected in your data is the surface pattern of the templates themselves. The model finds that signal early in training (your `training_run.json` shows reward_accuracy hitting 1.0 at step 100, the start of epoch 2) and rides it for the rest of the run.

This is reward overoptimization (Gao, Schulman, Hilton, ICML 2023) showing up in a different form. Their paper studies it with explicit reward models. The same dynamic appears when your "reward model" is the chosen-side data itself: a narrow signal produces a narrow policy.

## Show it: your own numbers say this

Three pass rates from `ablation_results.json`, all measured by `evaluation/scoring_evaluator.py` on the same 50 held-out tasks:

- Heuristic baseline (regex only, no LLM): 34%
- Trained SimPO judge: 30%
- Prompted base model: 58%

The trained model lost to a regex script. It also lost to its own untrained backbone by 28 percentage points. The most consistent explanation for that 28pp regression is mode collapse around the chosen-template manifold: training pushed the policy off the region where the held-out scorer rewards it, and onto a 111-mode subset where the model produces template-approximate strings with errors that the regex scorer punishes. That story is consistent with all three numbers; the alternative explanations (catastrophic forgetting, optimization failure) would not produce the specific pattern of trained < heuristic < prompted-base.

You can confirm or refute this in fifteen minutes. Open `evaluation/scoring_evaluator.py`, find whatever entry point it uses to score a (prompt, response) pair against rubric checks, and run something like this:

```python
import json
from evaluation.scoring_evaluator import <your_scorer_entry_point>

pairs = [json.loads(line) for line in open("training/training_data.jsonl")]
passed = 0
for p in pairs:
    score = <your_scorer_entry_point>(prompt=p["prompt"], response=p["chosen"])
    if score["passed_all_checks"]:  # adapt to whatever your evaluator returns
        passed += 1
print(f"Chosen-side pass rate against held-out scorer: {passed/len(pairs):.1%}")
```

(I don't know your exact scorer signature, so the snippet uses a placeholder. Adapt to your `scoring_evaluator.py` API.)

Two outcomes are possible. If the pass rate is high (say 80%+), then the templates you wrote do satisfy the held-out scorer in isolation, and the failure is pure mode collapse: the model imperfectly imitates them at generation time. If the pass rate is low (under 60%), then your chosen data does not even satisfy the scorer it is meant to teach the model to satisfy, and the training objective is misaligned end-to-end. Either result is decisive. My read of `prepare_training_data.py` says you will land in the first case (templates were hand-authored to pass the rubric), but the test removes the doubt.

## Three adjacent concepts that make the picture land

**Reward overoptimization** (Gao et al. 2023). The policy exploits whatever signal the reward gives it. A narrow reward produces a narrow policy. Your chosen-side is a narrow reward.

**Mode collapse and coverage in preference learning**. With 111 unique chosen strings, your policy's training distribution covers 111 modes. The held-out evaluation distribution is much wider (any rubric-satisfying generation should pass). The policy cannot escape the 111-mode region without being pulled there by training, and your training pulls it nowhere else.

**Train-eval contamination via shared generation source**. Your `train_test_split(test_size=0.1, seed=42)` puts 10% of the same 16 templates in eval. Your eval loss curve dropping from 1.99 to 0.96 measures how well the model learned the 16 templates, not whether it generalizes. This is why your in-loop signal looked clean while your held-out collapsed.

## The minimum structural change

The strict minimum is two changes in `prepare_training_data.py`, run together:

1. The chosen output is generated by an actual LLM call, not a template lookup. DeepSeek V3, GPT-4o, Claude, any of them. Real generations have natural diversity that templates do not, so the gradient stops latching onto template-surface markers.
2. Every candidate chosen output is run through `evaluation/scoring_evaluator.py` before it enters the training set. If `passed_all_checks` is false, retry; if retries fail, drop the task. No pair enters training unless its chosen side passes the same scorer that evaluates held-out. This aligns the training reward with the held-out reward at the data level, before the trainer ever sees the data.

Neither change works alone. (1) without (2) gives you diverse chosen outputs that may or may not satisfy the rubric, so the gradient still trains on a misaligned objective. (2) without (1) is what you already do conceptually (your templates were authored to pass the rubric), and that is exactly what produced the 30% trained pass rate. You need both.

A third change is necessary if you want to *measure* whether the fix worked: replace the random 10% eval split with an eval set drawn from a different generation pipeline than your training data (hand-authored adversarial cases, a separate LLM, or held-out tasks that share no templates with training). Without this, your eval loss continues to measure template memorization rather than generalization, and you cannot tell from in-loop signal whether the data fix worked. This is a measurement change, not a training change, but it is necessary for the first two to be defensible.

None of this requires switching SimPO, raising rank, adding KL, or doing anything inside the trainer. The trainer is fine. The data is the constraint.

## Pointers

- Meng, Xia, Chen, *SimPO: Simple Preference Optimization with a Reference-Free Reward*, NeurIPS 2024. The loss formulation that was used in your run.
- Rafailov et al., *Direct Preference Optimization*, NeurIPS 2023. The reference-model term that SimPO drops, and that some practitioners re-introduce as a KL anchor when they hit overoptimization.
- Gao, Schulman, Hilton, *Scaling Laws for Reward Model Overoptimization*, ICML 2023, arXiv:2210.10760. The canonical reference for "narrow reward produces narrow policy," the dynamic you are seeing in a different form.
- Tool to run today: the diagnose snippet above against your `training_data.jsonl` chosen field, scored by `evaluation/scoring_evaluator.py`. The pass rate is the load-bearing number for whether you are in the mode-collapse case or the misaligned-objective case.
