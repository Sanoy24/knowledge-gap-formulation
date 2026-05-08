# Evening Call Summary — Day 4

**Pair:** Charlie Lijalem and Yonas Eshete
**Date:** Day 4, Week 12
**Confirmed by:** both partners

---

## Charlie's feedback on Yonas's explainer (cross-family rotation and self-preference)

Charlie said the leakage versus self-preference distinction was the part he had needed most. He had been thinking of them as one problem under "cross-family rotation," and the explainer gave him the language to separate them. That framing landed.

Two things he pushed back on. The phrase "closed self-preference loop" in Path 2 read as a conclusion before the argument had been made. He could follow it but the first time through it felt like jargon. Yonas moved the `model_card.md:82` quote earlier in that section so the claim was grounded in Charlie's own words before the loop framing appeared. The second issue: the empirical test script used `"dataset/partitions/held_out.jsonl"` as the path, which does not match Charlie's actual schema. Charlie confirmed the real scores live in `ablation_results.json` with `generation_mode` as the metadata field. Yonas updated the script accordingly.

Charlie signed off with the gap closed. He said he would run the stratification before publishing the blog and update the model card with whatever the numbers show.

## Yonas's feedback on Charlie's explainer (paired bootstrap CIs at ceiling)

Yonas said the cleanest part was the explicit derivation of why the CI of [0.0%, 6.8%] is bounded below at zero rather than symmetric around the observed 2.27pp. That had been the part of the model card he could not defend. After reading the explainer, Yonas understood the bound is a consequence of the resampled differences being mostly zero (because 43 of the 44 paired observations are tied at correct-correct), not a feature of the underlying distribution. The CI is doing what it should — it just looks degenerate because the data is degenerate.

What did not land on the first read: the recommendation between McNemar's exact test and a different evaluation mode was hedged. Yonas asked Charlie to commit. Charlie revised to a single recommendation: McNemar's exact test on the 1 discordant pair is the right small-n test for this regime, but the answer it gives (one-sided p ≈ 0.5 on a single discordant pair) tells Yonas nothing useful about whether the adapter helped. The actionable conclusion is that the held-out partition at this size and baseline cannot answer the question. To get a real answer, Yonas needs either generation-mode scoring on a larger eval set or an adversarial held-out partition where the base accuracy is below 90%.

Yonas signed off with the gap closed. He committed to rewriting the model card's Delta A interpretation paragraph to say "this evaluation regime is structurally underpowered for this baseline" rather than "non-significant due to small n," which the original phrasing had implied could be fixed by simply adding more pairs.

## Both signoffs

- Charlie: closed
- Yonas: closed
