# Grounding Commit — Day 4

**Asker:** Yonas Eshete
**Explainer author:** Charlie Lijalem
**Date:** Day 4, Week 12
**Commit:** `6ae23eb20a8a717f157b6b2d669be69876570580`

## Artifact Edited

**File:** `tenacious-bench/model_card.md`
**Lines changed:** evaluation results paragraph (line 67) and Limitations bullet (line 84)

## What Changed and Why

### Change 1 — Evaluation results interpretation

**Before:**

> Paired bootstrap test (1000 iterations, seed=42): p=0.372, 95% CI=[0.0%, 6.8%] — not statistically significant due to high baseline and n=44. The ceiling effect is the primary constraint; see blog post for full interpretation.

**After:**

> Paired bootstrap test (1000 iterations, seed=42): p=0.372, 95% CI=[0.0%, 6.8%] (n=44). This result should not be read as "the improvement is likely somewhere between 0% and 6.8%." With a 97.73% baseline, only one pair discriminated between the adapter and the base model; the CI reflects the sampling variability of that single pair, not a range of plausible true effect sizes. The evaluation cannot confirm or rule out a real improvement — it is structurally underpowered for the regime the adapter was trained on.

**Why:** The original phrasing framed the result as "not statistically significant due to small n" — which implies the problem is fixable by adding more pairs. Charlie's explainer showed this is wrong. With base=43/44 and adapter=44/44, only one pair discriminates between the two systems. The p-value of 0.372 is approximately the probability that the critical pair is absent from a bootstrap draw: P(absent) = (43/44)^44 ≈ 0.362. The CI upper bound of 6.8% is not an estimate of plausible true effect — it is the apparent improvement in the luckiest bootstrap resample (critical pair drawn 3 times). The CI is mathematically correct; it is nearly uninformative about true effect size. The structural problem is not n=44 — it is that 43 of 44 pairs returned the same answer for both systems.

### Change 2 — Limitations bullet

**Before:**

> Statistical lift cannot be proven at n=44; a larger adversarial held-out slice is needed for conclusive measurement.

**After:**

> Statistical lift cannot be confirmed with this evaluation structure. Adding more non-adversarial pairs does not fix this, nor does switching to a permutation test or Fisher's exact test — a different test on the same one-discriminating-pair data returns the same answer. The three paths that would give a real answer: (1) a 10–15 pair adversarial slice where the base model scores 60–70%, (2) generation-mode scoring which is continuous rather than binary and harder to saturate, or (3) continuous reward margin (r_chosen − r_rejected) rather than binary win/lose.

**Why:** The original Limitations framing implied more data or a different test statistic would fix the problem. Charlie's explainer is explicit: a different test on the same one-discriminating-pair data gives the same answer, and more non-adversarial pairs at a 97%+ baseline still leave only one discriminating pair. The three paths Charlie identified address the actual constraint — not enough signal at the current baseline and evaluation mode.

## Source for both changes

Charlie Lijalem's explainer — `my-pairs.md` (published at [medium.com/@chalielijalem](https://medium.com/@chalielijalem/explainer-what-p-0-372-actually-says-when-your-baseline-is-97-73-58b4fcdc1668)). The replacement text in Change 1 matches the "Honest Statement for the Model Card" block in Charlie's explainer verbatim, with minor compression.
