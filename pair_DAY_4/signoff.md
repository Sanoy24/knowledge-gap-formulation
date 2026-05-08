# Sign-Off — Day 4

**Asker:** Yonas Eshete
**Explainer author:** Charlie Lijalem
**Date:** Day 4, Week 12

## Gap Closure Status

**Status:** closed

## What I understand now that I didn't before

My p=0.372 result is not a small-n problem. It is a one-pair problem. With base=43/44 and adapter=44/44, only one pair discriminates between the two systems. The paired bootstrap resamples those 44 pairs with replacement, and in roughly 36% of draws the critical pair is absent — that is where the 0.372 p-value comes from. The CI of [0.0%, 6.8%] is not a range of plausible true effect sizes; the 6.8% upper bound is the apparent improvement when the critical pair happens to be drawn three times in one bootstrap resample. It is a resampling artifact, not a signal about reality.

The framing I had before — "non-significant due to small n, ceiling effect" — implied the problem was fixable by adding more data or switching to a different test. Charlie's explainer killed both of those exits. Adding more non-adversarial pairs at 97%+ baseline still leaves one discriminating pair. A permutation test or McNemar's exact test on one discordant pair returns the same answer. The problem is the data structure, not the test or the sample size.

What would actually give a real answer: a 10–15 pair adversarial slice where the base model scores 60–70%, or generation-mode scoring (continuous, harder to saturate), or continuous reward margin instead of binary win/lose. I had built the bench to stress adversarial probes but evaluated on a slice where the base model was already saturated on non-adversarial cases. The evaluation regime and the training target were not aligned.

I updated `tenacious-bench/model_card.md` — both the bootstrap interpretation paragraph and the Limitations bullet — to say this clearly instead of implying a fixable-with-more-data problem.
