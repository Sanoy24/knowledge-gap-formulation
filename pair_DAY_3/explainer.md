# Your chosen-side data IS your reward function, and a 16-template reward function cannot generalize

**For:** Amir Ahmedin
**Question:** Given that my 599 chosen outputs collapse to 111 unique strings from a 16-template library in `get_chosen()`, and my eval split is drawn from the same template pool as training, what does this imply for any preference-optimization recipe I run on this data, and what is the minimum structural change to my data pipeline that would make preference training capable of generalizing to the held-out scorer?
**Author:** Yonas Eshete
**Status:** revised after Amir's feedback (added diversity threshold, fixed scorer interface, softened the "templates pass in isolation" assumption)

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

You can confirm or refute this in fifteen minutes using your real scorer interface (`TenaciousBenchEvaluator.score_task(task, agent_output)` from `evaluation/scoring_evaluator.py`):

```python
import json
from evaluation.scoring_evaluator import TenaciousBenchEvaluator

evaluator = TenaciousBenchEvaluator()

# Load task definitions so we can score by task_id
tasks_by_id = {}
for partition in ["train", "dev"]:
    for task in json.load(open(f"dataset/partitions/{partition}.json")):
        tasks_by_id[task["task_id"]] = task

# Pairs with task_id metadata (your prepare_training_data.py writes this file)
pairs = json.load(open("training/preference_pairs_with_metadata.json"))

passed, checked = 0, 0
for pair in pairs:
    task = tasks_by_id.get(pair["task_id"])
    if not task:
        continue
    chosen = pair["chosen"]
    # Parse the "Subject: X\n\nBody" string back into the structured agent_output
    if "Subject:" in chosen:
        subject = chosen.split("Subject:", 1)[1].split("\n", 1)[0].strip()
        body = chosen.split("\n\n", 1)[1] if "\n\n" in chosen else ""
    else:
        subject, body = "", chosen
    result = evaluator.score_task(task, {"email_subject": subject, "email_body": body})
    passed += int(result.passed)
    checked += 1

print(f"Chosen-side pass rate against held-out scorer: {passed/checked:.1%} ({passed}/{checked})")
```

Two outcomes are possible. If the pass rate is high (say 80%+), the templates do satisfy the scorer in isolation, and the failure at generation time is mode collapse — the model imperfectly imitates them. If the pass rate is low (under 60%), your chosen data does not even satisfy the scorer it is meant to teach the model to satisfy, and the training objective is misaligned end-to-end. I do not have a strong prior on which outcome you will see. Most of your TONE_PASSING and WORKFLOW_PASSING templates were hand-authored to pass, so those probably score high. But your RESOURCE_HONESTY_PASSING templates interpolate `{available}` and `{requested}` from `bench_state`, and depending on what those numbers are for a given task, some chosen rewrites might fail the bench-honesty check even though they were intended to pass. The diagnostic resolves it without me guessing.

## Three adjacent concepts that make the picture land

**Reward overoptimization** (Gao et al. 2023). The policy exploits whatever signal the reward gives it. A narrow reward produces a narrow policy. Your chosen-side is a narrow reward.

**Mode collapse and coverage in preference learning**. With 111 unique chosen strings, your policy's training distribution covers 111 modes. The held-out evaluation distribution is much wider (any rubric-satisfying generation should pass). The policy cannot escape the 111-mode region without being pulled there by training, and your training pulls it nowhere else.

**Train-eval contamination via shared generation source**. Your `train_test_split(test_size=0.1, seed=42)` puts 10% of the same 16 templates in eval. Your eval loss curve dropping from 1.99 to 0.96 measures how well the model learned the 16 templates, not whether it generalizes. This is why your in-loop signal looked clean while your held-out collapsed.

## How much chosen-side diversity is enough

You asked for a number. Here is the empirical anchor I have, with two data points (yours and mine):

| Dataset                    | Pairs | Unique chosen | Ratio     | Held-out outcome                                                                                  |
| -------------------------- | ----- | ------------- | --------- | ------------------------------------------------------------------------------------------------- |
| Your tenacious-sales-bench | 599   | 111           | **0.185** | trained 30% < heuristic 34% < prompted-base 58% (catastrophic regression)                         |
| My tenacious-bench (train) | 618   | 618           | **1.000** | trained 100% / 44 vs base 97.7% / 44 on held-out, p=0.372 (no regression, marginal lift)          |
| My tenacious-bench (dev)   | 347   | 346           | **0.997** | —                                                                                                 |

Two data points are not a paper, but they are a strong directional signal. My pipeline (DeepSeek V3 chosen-rewrites, scorer-gated, one rewrite per task) yields essentially every chosen string unique. Yours (template interpolation, 16 templates) yields 5.4 pairs per unique chosen on average. Recommendation: **target unique-chosen-to-total ratio ≥ 0.9**, which is the natural output of any LLM-call-per-task pipeline. Treat **0.5 as the danger floor** — at 0.5, half your pairs share chosen strings with at least one other pair, and the gradient overweights whatever surface pattern those duplicates share.

A more principled diversity measure would use pairwise embedding cosine similarity (target: median pairwise cosine below some threshold like 0.7 on a sentence encoder), but the unique-string ratio is a fast first check that catches the worst case. If your post-fix pipeline yields ratio ≥ 0.9 you are almost certainly fine. If it yields 0.5–0.9 because your tasks are structurally similar enough that even an LLM produces near-duplicates, the embedding-similarity check is the next thing to run.

## The minimum structural change

The strict minimum is two changes in `prepare_training_data.py`, run together:

1. The chosen output is generated by an actual LLM call, not a template lookup. DeepSeek V3, GPT-4o, Claude, any of them. Real generations have natural diversity that templates do not, so the gradient stops latching onto template-surface markers. Empirically this gets the unique-chosen-to-total ratio close to 1.0.
2. Every candidate chosen output is run through `evaluation/scoring_evaluator.py` before it enters the training set. If `result.passed` is false, retry; if retries fail, drop the task. No pair enters training unless its chosen side passes the same scorer that evaluates held-out. This aligns the training reward with the held-out reward at the data level, before the trainer ever sees the data.

Neither change works alone. (1) without (2) gives you diverse chosen outputs that may or may not satisfy the rubric, so the gradient still trains on a misaligned objective. (2) without (1) is what you already do conceptually (your templates were authored to pass the rubric), and that is exactly what produced the 30% trained pass rate. You need both.

A third change is necessary if you want to *measure* whether the fix worked: replace the random 10% eval split with an eval set drawn from a different generation pipeline than your training data (hand-authored adversarial cases, a separate LLM, or held-out tasks that share no templates with training). Without this, your eval loss continues to measure template memorization rather than generalization, and you cannot tell from in-loop signal whether the data fix worked. This is a measurement change, not a training change, but it is necessary for the first two to be defensible.

None of this requires switching SimPO, raising rank, adding KL, or doing anything inside the trainer. The trainer is fine. The data is the constraint.

## Pointers

- Meng, Xia, Chen, *SimPO: Simple Preference Optimization with a Reference-Free Reward*, NeurIPS 2024. The loss formulation that was used in your run.
- Rafailov et al., *Direct Preference Optimization*, NeurIPS 2023. The reference-model term that SimPO drops, and that some practitioners re-introduce as a KL anchor when they hit overoptimization.
- Gao, Schulman, Hilton, *Scaling Laws for Reward Model Overoptimization*, ICML 2023, arXiv:2210.10760. The canonical reference for "narrow reward produces narrow policy," the dynamic you are seeing in a different form.
- Tool to run today: the diagnostic above against your `training/preference_pairs_with_metadata.json` chosen field, scored by `evaluation/scoring_evaluator.py`. The pass rate is the load-bearing number for whether you are in the mode-collapse case or the misaligned-objective case.
