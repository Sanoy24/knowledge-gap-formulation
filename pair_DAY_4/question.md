# Question - Day 4

**Asker:** Yonas Eshete  
**Partner:** Chalie Lijalem  
**Topic:** Evaluation and statistics — paired bootstrap CIs at ceiling  
**Date:** 2026-05-08

---

## The Question

My `tenacious-bench` model card reports the following evaluation result on 44 held-out preference pairs:

| Condition                        | Pairwise Accuracy | Delta       |
| -------------------------------- | ----------------- | ----------- |
| Base model (zero-shot)           | 97.73% (43/44)    | —           |
| Prompt-engineered base (Delta B) | 97.73% (43/44)    | +0.00pp     |
| SimPO LoRA adapter (Delta A)     | 100.00% (44/44)   | **+2.27pp** |

I ran a paired bootstrap test (1000 iterations, seed=42) and got: **p=0.372, 95% CI=[0.0%, 6.8%]**. I attributed non-significance to small n and a ceiling effect — the base model already passes 43 out of 44 pairs. The per-pair diagnostic (`ablations/per_pair_diagnostic_dev100.jsonl`) shows that 76 of 100 dev pairs were in the `silent_passenger` regime (margin ≥ 1.75 at the start of training) and contributed near-zero gradient. The two pairs the adapter actually flipped (`tb-syn-047`, `tb-prog-dsl-054`) are non-adversarial easy edges — none of the adversarial P007/P011/P027 probes that drove the Path B decision were flipped.

**I can report these numbers. I cannot defend what they mean.**

Specifically: does `95% CI=[0.0%, 6.8%]` mean the true improvement is likely somewhere between 0% and 6.8%? Or does a 97.73% baseline violate the assumptions the paired bootstrap is built on, making that interval uninterpretable? And if this evaluation is underpowered, what would I need — more held-out pairs, a different evaluation mode (generation-mode scoring rather than pairwise log-prob), or a different test statistic — to reach a defensible conclusion about whether the SimPO adapter changed anything real?

---

## Artifact Connection

The gap sits directly in `tenacious-bench/model_card.md` (lines 63–67) and `tenacious-bench/methodology.md` (lines 193–206). The Delta A/B/C table is the primary evidence I use to claim the LoRA adapter improved the judge. If p=0.372 means the result is noise, the claim in my model card is unsupported. If the ceiling effect invalidates the bootstrap interval, the CI I report is misleading rather than informative.

Understanding the mechanism would let me do one of three things:

1. Correctly qualify the Delta A result in the model card — not "not significant due to small n" but a precise statement of what can and cannot be concluded from CI=[0.0%, 6.8%] at this baseline.
2. Identify whether I need more tasks or a different evaluation mode (generation-mode vs pairwise log-prob) to get a real answer.
3. Know whether the current held-out slice of 44 pairs, with a 97.73% baseline, is structurally incapable of measuring the improvement the bench was designed to detect.

This matters beyond this benchmark. In any FDE engagement where a client asks "did your fine-tuned model improve over the baseline?", the answer requires knowing when small-n evaluation results are trustworthy, when ceiling effects make improvement unmeasurable, and what an honest CI actually communicates.
