# Question — Day 1

**Asker:** Yonas Eshete
**Topic:** Training and post-training mechanics — _What LoRA actually adapts; why low rank works at all; what changes at higher rank_
**Date:** 2026-05-04

---

## The Question

In my Week 11 `model_card.md` (published artifact: `sanoy24/tenacious-judge-qwen25-3b-gamma15` on HuggingFace), the Technical Specifications section states:

> _"Architecture: LoRA adapter (rank 16, alpha 32) applied to Qwen attention and feedforward layers"_

And in `methodology_rationale.md`:

> _"LoRA rank 16, alpha 32, seed 42"_

Both entries carry no derivation. The values were copied from the Unsloth starter notebook defaults.

**What is the mathematical property of fine-tuning gradient updates that makes low-rank decomposition sufficient for domain adaptation — and what does that property tell me about how to choose rank for a 618-pair binary preference task on a narrow single-domain vocabulary?**

I know the LoRA formula: ΔW = B·A scaled by alpha/rank. What I cannot explain is the mechanism beneath it — why the weight changes a pre-trained model needs for narrow domain adaptation are inherently low-rank in the first place, and therefore what principled rule would tell me whether rank 16 captures the full adaptation signal for this task or whether rank 4 or rank 32 would have produced a meaningfully different outcome.

---

## Artifact Connection

Knowing this would let me revise the **Technical Specifications section of `model_card.md`** in `sanoy24/tenacious-judge-qwen25-3b-gamma15`, which currently states rank 16 and alpha 32 as bare numbers with no justification for why those values match the adaptation difficulty of this task.

More broadly, this gap changes how I approach model adaptation decisions on FDE engagements. Any client asking me to fine-tune a small backbone on domain-specific data will need a rank choice. Without understanding why low rank works, I have no principled way to set it, validate it, or explain it. I am always one "why?" away from having no answer.
