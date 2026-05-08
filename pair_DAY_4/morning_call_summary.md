# Morning Call Summary — Day 4

**Pair:** Charlie Lijalem (asker on self-preference) and Yonas Eshete (asker on bootstrap CIs at ceiling)
**Date:** Day 4, Week 12
**Confirmed by:** both partners

---

## What we sharpened in Charlie's question

Yonas pushed on the conflation first. Charlie's draft treated preference leakage and self-preference as one problem under the "cross-family rotation" header. Yonas asked: are these the same bias or different ones? After a few minutes Charlie agreed they live at different layers. Leakage is a training-data contamination story (Li et al., 2025). Self-preference is an inference-time stylistic favoritism story (Zheng et al., 2023). The routing table fixes the first. It does not touch the second.

Once that landed, Yonas pushed for specifics. Where in the pipeline does self-preference actually survive after rotation? Charlie walked through his code and surfaced two paths: Gemini authoring 90 bulk synthesis tasks and also evaluating tone on those tasks at held-out time, and Gemini generating chosen preference pairs while also judging the tone dimension. Both were already in `model_card.md:82` as an acknowledged gap, but Charlie had not connected them to the self-preference question. The final committed question names both paths and asks for the empirical test that separates them: stratify held-out scores by `generation_model` against scores already in `ablation_results.json`.

## What we sharpened in Yonas's question

Charlie pushed on the framing. Yonas's draft asked "is the paired bootstrap CI interpretable when the baseline is 97.73%?" Charlie reframed it: the question is not whether bootstrap is broken in general, it is whether the specific assumption that bootstrap relies on (the resampled distribution approximating the true sampling distribution) holds when 43 of 44 outcomes are identical. Yonas had been treating "small n" and "ceiling effect" as one combined problem. Charlie split them: small n affects power, ceiling affects which test is appropriate at all.

Yonas also had three sub-questions about what to do next (more tasks, different evaluation mode, different test statistic). Charlie said pick one — the explainer cannot answer all three in 1,000 words. Yonas committed to "what does the CI of [0.0%, 6.8%] actually mean at a 97.73% baseline, and is paired bootstrap the wrong test for this regime in the first place?" The follow-on questions move to Day 5 if the explainer warrants it.

## What both of us committed to before the explainers arrive

Charlie agreed to verify the actual field name for stratification (`generation_model` versus something else in his schema) before Yonas writes the diagnostic snippet. Yonas agreed to confirm the exact n in the bootstrap (44 pairs after 4 skipped tasks, not 48) before Charlie writes the CI explainer. Both small details, but the partner cannot write a tight explainer without them.
