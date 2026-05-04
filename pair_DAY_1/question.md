# Question — Day 1

**Asker:** Yonas Mekonnen
**Topic:** Training and post-training mechanics — *What LoRA actually adapts; why low rank works at all; what changes at higher rank*
**Date:** 2026-05-04

---

## The Question

In my Week 11 `model_card.md` (published artifact: `sanoy24/tenacious-judge-qwen25-3b-gamma15` on HuggingFace), the Technical Specifications section states:

> *"Architecture: LoRA adapter (rank 16, alpha 32) applied to Qwen attention and feedforward layers"*

And in `methodology_rationale.md`:

> *"LoRA rank 16, alpha 32, seed 42"*

Both entries carry no derivation. The values were copied from the Unsloth starter notebook defaults.

**What is the mathematical property of fine-tuning gradient updates that makes low-rank decomposition sufficient for domain adaptation — and what does that property tell me about how to choose rank for a 618-pair binary preference task on a narrow single-domain vocabulary?**

I know the LoRA formula: ΔW = B·A scaled by alpha/rank. What I cannot explain is the mechanism beneath it — why the weight changes a pre-trained model needs for narrow domain adaptation are inherently low-rank in the first place, and therefore what principled rule would tell me whether rank 16 captures the full adaptation signal for this task or whether rank 4 or rank 32 would have produced a meaningfully different outcome.

---

## Artifact Connection

Knowing this would let me revise the **Technical Specifications section of `model_card.md`** in `sanoy24/tenacious-judge-qwen25-3b-gamma15`, which currently states rank 16 and alpha 32 as bare numbers with no justification for why those values match the adaptation difficulty of this task.

That paragraph is the first thing a senior ML engineer or FDE hiring manager would challenge. My only honest answer today is "Unsloth defaults." After this gap closes, the model card should instead name: the mathematical reason low-rank updates are sufficient for pre-trained model adaptation, what that implies about rank selection for a narrow 618-pair preference task, and whether my choice was approximately right or accidentally correct.

More broadly, this gap changes how I approach model adaptation decisions on FDE engagements. Any client asking me to fine-tune a small backbone on domain-specific data will need a rank choice. Without understanding why low rank works, I have no principled way to set it, validate it, or explain it — I am always one "why?" away from having no answer.

---

## Four-Property Self-Check

| Property | Assessment |
| --- | --- |
| **Diagnostic** | Names one specific mechanism — the intrinsic low-rank structure of fine-tuning gradients — and connects it to one practical decision: rank selection for this task shape. Not "how does LoRA work?" |
| **Grounded in your work** | Points to an exact underjustified line in a published public artifact (`model_card.md`, `sanoy24/tenacious-judge-qwen25-3b-gamma15`) and the matching bare assertion in `methodology_rationale.md` |
| **Generalizable** | Every FDE adapting a small backbone for a client-specific task faces rank selection. The intrinsic dimensionality argument is the reasoning any FDE needs to stop deferring to defaults |
| **Resolvable in one explainer** | One mechanism (why gradient updates are low-rank) + one empirical question (what changes at rank 4/8/32 on a small dataset) is closable in 600–1,000 words with a short rank-sweep code demonstration |
