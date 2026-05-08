# Portfolio Update

**Trainee:** Yonas Eshete
**Week:** 12 — Knowledge Gap Formulation
**Audience:** FDE hiring manager

---

## Summary

Four days of paired research produced four grounding commits to two portfolio repositories — `Sanoy24/tenacious-bench` (Week 11) and an indirect contribution to a partner's Week 10 conversion engine. The commits do not add features. They correct specific claims that were either unjustified, incorrectly framed, or structurally misleading. After this week, every major technical claim in the tenacious-bench model card traces to a mechanism I can explain rather than a default I copied.

The pattern across all four days is the same: I was using language that sounded precise but was not grounded. "Rank 16" had no derivation. "Not significant due to small n" implied the wrong fix. "Your chosen-side data is your reward function" — I knew the phrase but had not traced through what that meant for Amir's 16-template chosen side. The research forced the grounding.

---

## The Four Grounding Commits

| Day | Artifact edited | Commit | What changed |
| --- | --- | --- | --- |
| Day 1 | `tenacious-bench/model_card.md` | [`37b46c2`](https://github.com/Sanoy24/tenacious-bench/commit/37b46c2055b73834f951442a70118d28a0ee5a4b) | Added intrinsic-dimensionality justification for LoRA rank 16 |
| Day 2 | `signal-driven-sales-conversion-engine/agent/llm/reply_agent.py` (partner's repo) | [`94b31a9`](https://github.com/mamee13/signal-driven-sales-conversion-engine/commit/94b31a9e84bd54604d9f64d70bc59681339195c1) | Enum repair, tier logging, keyword removal — changes enabled by Yonas's explainer |
| Day 3 | `tenacious-bench/methodology_rationale.md`, `ablations/diagnose_per_pair_margins.py`, `ablation_results.json` | [`dfb74936`](https://github.com/Sanoy24/tenacious-bench/commit/dfb74936cbe3612c22c6a27c4ab18d9d07f50f7f) | Per-pair gradient derivation, diagnostic script, corrected Delta A interpretation |
| Day 4 | `tenacious-bench/model_card.md` | [`6ae23eb`](https://github.com/Sanoy24/tenacious-bench/commit/6ae23eb20a8a717f157b6b2d669be69876570580) | Bootstrap CI interpretation rewritten; Limitations corrected |

---

## Before / After: Portfolio Defensibility

### Day 1 — LoRA rank justification

**Before:** `model_card.md` listed "LoRA Rank: 16, LoRA Alpha: 32" as bare numbers. The values came from an Unsloth starter notebook. There was no explanation of why rank 16 was appropriate for this task.

**After:** The model card now explains the mechanism: fine-tuning a pretrained model reweights existing features rather than learning new ones. For a 618-pair narrow-domain preference task distinguishing a small number of latent factors (persuasion quality, tone alignment, constraint violations), the effective adaptation signal likely lies in the rank 4–8 range. Rank 16 is described as a conservative upper bound — correct but not optimal — and the model card notes that a rank sweep would likely confirm the extra capacity went unused. A senior engineer asking "why rank 16?" now gets a derivable answer, not a shrug.

### Day 2 — Intent classifier safety (partner's portfolio, enabled by Yonas's explainer)

**Before:** Mamaru's `classify_reply()` used `json.loads()` directly on LLM output, treated `"interested"` as a positive keyword without qualification, and had no field recording which tier (keyword vs LLM fallback) handled each reply. The claim that "80% of cases are handled by keyword pre-screening" was an estimate with no logging to verify it.

**After:** Three changes grounded in the token-level mechanism Yonas's explainer traced: markdown fence stripping and enum normalization before parse failure, a `classification_tier` field on every classification result, and removal of the `"interested"` keyword that would match negative sentences like "I am not interested enough to justify the switch." The 80/20 claim is now measurable from logs.

### Day 3 — Per-pair gradient mechanics and Delta A interpretation

**Before:** `methodology_rationale.md` mentioned LoRA rank but contained no derivation of the per-pair gradient decay formula. The Delta A result (+2.27pp, p=0.001) was reported with no account of which pairs actually contributed training signal. `ablation_results.json` stated the headline finding without qualification.

**After:** Three changes in one commit. `methodology_rationale.md` now contains the per-pair gradient decay derivation: `|∂L/∂θ| ∝ σ(−s)` where `s = β·M − γ`, showing why pairs with margin `M ≥ 1.75` contribute under 12% of max gradient. The diagnostic script (`diagnose_per_pair_margins.py`) runs against the actual model and shows 76 of 100 dev pairs were silent passengers from step zero, and only 2 pairs flipped — both non-adversarial. `ablation_results.json` now includes a `delta_a.interpretation` field stating "lift came from non-adversarial easy edges, not adversarial probes." The model card no longer implies the adapter fixed the cases it was built to fix.

### Day 4 — Bootstrap CI interpretation

**Before:** The model card stated the evaluation result as "p=0.372, 95% CI=[0.0%, 6.8%] — not statistically significant due to high baseline and n=44." The Limitations section said "a larger adversarial held-out slice is needed." Both phrasings implied the problem was fixable by adding more data or running a different test.

**After:** The evaluation paragraph now explains the structural problem: with base=43/44 and adapter=44/44, only one pair discriminated between the two systems. The CI reflects the sampling variability of that single pair, not a range of plausible true effect sizes. The Limitations section no longer suggests that adding pairs or switching to a permutation test would fix the evaluation — it names the three paths that would actually give a real answer: an adversarial slice at 60–70% base accuracy, generation-mode scoring, or continuous reward margin. A hiring manager reading this can tell the difference between an engineer who treats "p > 0.05" as a data-collection problem and one who understands when the evaluation regime itself is the constraint.
