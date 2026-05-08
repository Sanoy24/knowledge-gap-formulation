# Knowledge Gap Formulation for Compounding

A four-day paired research sprint that systematically closes understanding gaps in production AI systems — from LoRA adaptation mechanics to LLM-as-judge evaluation integrity — and publishes each finding as a standalone public explainer.

## What This Is

Building AI systems is fast. Understanding why they work the way they do is slow. This project takes the systems I shipped — a sales-agent evaluation benchmark, a fine-tuned judge model, a preference dataset, a conversion engine — and audits them for gaps between what I built and what I can actually defend under scrutiny.

Each day follows the same loop:

1. **Surface a gap** — find the specific mechanism in my own shipped work that I cannot explain if pressed
2. **Sharpen it with a partner** — morning call to interrogate and tighten the question
3. **Research and write** — read canonical sources, run experiments, write an explainer for the partner's gap
4. **Revise and commit** — evening call for feedback, then commit a concrete improvement to existing portfolio work

Every research cycle produces two public artifacts (a blog post and a tweet thread) and one grounding commit that improves a previously shipped artifact.

## Grounding Target

All research is grounded against **[Tenacious-Bench](https://github.com/Sanoy24/tenacious-bench)** — a sales-agent evaluation benchmark featuring a fine-tuned judge model (Qwen2.5-3B, SimPO-trained on 618 preference pairs).

| Artifact             | Link                                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------- |
| Evaluation benchmark | [Sanoy24/tenacious-bench](https://github.com/Sanoy24/tenacious-bench)                                         |
| Preference dataset   | [sanoy24/tenacious_bench_v0.1](https://huggingface.co/datasets/sanoy24/tenacious_bench_v0.1)                  |
| Fine-tuned judge     | [sanoy24/tenacious-judge-qwen25-3b-gamma15](https://huggingface.co/sanoy24/tenacious-judge-qwen25-3b-gamma15) |

---

## Published Explainers

### Blog Posts

| Day | Title                                                                       | Link                                                                                            |
| --- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 1   | Why Your LLM Sales Agent Gives the Same Answer to Every Question            | [Substack →](https://open.substack.com/pub/yonasmekonnen/p/why-your-sales-agent-gives-the-same) |
| 2   | How LLM Intent Classification Actually Works at the Token Level             | [Substack →](https://open.substack.com/pub/yonasmekonnen/p/your-llm-intent-classifier-is-not)   |
| 3   | Your Chosen-Side Training Data Is Your Reward Function                      | [Substack →](https://yonasmekonnen.substack.com/p/your-chosen-side-training-data-is)            |
| 4   | Cross-Family Rotation Closes the Leakage Path, Not the Self-Preference Path | [Substack →](https://open.substack.com/pub/yonasmekonnen/p/cross-family-rotation-closes-the)    |

### Tweet / LinkedIn Threads

| Day | Link                                                  |
| --- | ----------------------------------------------------- |
| 1   | https://x.com/yorom24/status/2051623189063549014?s=20 |
| 2   | https://x.com/yorom24/status/2052073735360749643?s=20 |
| 3   | https://x.com/yorom24/status/2052449132246176066?s=20 |
| 4   | https://x.com/yorom24/status/2052794844913750064?s=20 |

---

## Research Log

| Day | Topic                              | Gap Investigated                                                                                                                                                                              | Partner         | Grounding Commit                                                                                                                                                                                                                                                                    |
| --- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Training & post-training mechanics | Why LoRA fine-tuning gradients are inherently low-rank, and what that means for rank selection on a 618-pair preference task                                                                  | Amare Kassa     | [`37b46c2`](https://github.com/Sanoy24/tenacious-bench/commit/37b46c2055b73834f951442a70118d28a0ee5a4b) — added intrinsic-dimensionality justification to `model_card.md`                                                                                                           |
| 2   | Agent and tool-use internals       | Why function-calling LLMs fabricate missing required arguments instead of asking clarification questions — traced in the SCAP patch in `eval/run_heldout.py` of the Week 10 Conversion Engine | Mamaru Yirga    | [`94b31a9`](https://github.com/mamee13/signal-driven-sales-conversion-engine/commit/94b31a9e84bd54604d9f64d70bc59681339195c1) — enum repair, tier logging, and keyword removal in `reply_agent.py` (partner's portfolio; enabled by Yonas's explainer)                              |
| 3   | Training & post-training mechanics | Per-pair gradient trajectory during SimPO fine-tuning — which pairs contributed real signal vs sat silent from step zero                                                                      | Amir Ahmedin    | [`dfb74936`](https://github.com/Sanoy24/tenacious-bench/commit/dfb74936cbe3612c22c6a27c4ab18d9d07f50f7f) — per-pair gradient derivation in `methodology_rationale.md`, diagnostic script `ablations/diagnose_per_pair_margins.py`, corrected `ablation_results.json` interpretation |
| 4   | Evaluation & statistics            | Whether CI=[0.0%, 6.8%] at a 97.73% baseline is interpretable, and what would constitute a defensible evaluation of whether the SimPO adapter changed anything real                           | Charlie Lijalem | [`6ae23eb`](https://github.com/Sanoy24/tenacious-bench/commit/6ae23eb20a8a717f157b6b2d669be69876570580) — bootstrap CI interpretation and Limitations section rewritten in `model_card.md`                                                                                          |

---

## Repository Structure

```text
├── pair_DAY_1/          # Day 1 — LoRA rank selection & response collapse
│   ├── question.md              # Final sharpened question with artifact connection
│   ├── morning_call_summary.md  # How the question was interrogated and tightened
│   ├── explainer.md             # Blog post written for partner's gap
│   ├── thread.md                # Tweet thread, ready to publish
│   ├── evening_call_summary.md  # Feedback given and revisions made
│   ├── signoff.md               # Gap-closure judgment
│   ├── grounding_commit.md      # Pointer to concrete portfolio edit
│   └── sources.md               # Canonical sources and tools used
├── pair_DAY_2/          # Day 2 — Premature tool execution & intent classification
├── pair_DAY_3/          # Day 3 — SimPO per-pair gradient trajectory
├── pair_DAY_4/          # Day 4 — Bootstrap CIs at ceiling & self-preference bias
│
├── synthesis.md          # Eight gaps closed, most surprising finding, reading list
├── canonical_list.md     # Annotated reading list — 10 papers, 4 tools, 5 patterns
├── portfolio_update.md   # How grounding commits improve the shipped portfolio
└── docs/                 # Challenge specification and reference material
```

Each `pair_DAY_N/` folder captures the full research cycle: the question that was asked, the negotiation that sharpened it, the explainer that answered it, the feedback that revised it, and the commit that grounded it in real work.

---

## Key Concepts Explored

- **LoRA intrinsic dimensionality** — why low-rank adaptation works for narrow-domain fine-tuning, and how to reason about rank selection without relying on starter-notebook defaults
- **Reward model overoptimization** — how proxy reward signals diverge from task performance (Gao et al., ICML 2023); why chosen-side template collapse is the same phenomenon at the data level
- **SimPO per-pair gradient decay** — the formula `|∂L/∂θ| ∝ σ(−s)` where `s = β·M − γ`; why 76 of 100 dev pairs contributed near-zero gradient from step zero
- **Degenerate text generation** — why likelihood maximization produces generic outputs (Holtzman et al., ICLR 2020); how to detect and measure response collapse
- **Premature tool execution bias** — why function-calling LLMs prefer fabricating missing arguments over asking clarifying questions; structural vs prompt-based fixes
- **LLM intent classification at the token level** — next-token generation under prompt pressure; lexical vs compositional intent; when keyword pre-screening is the right tool
- **Preference leakage vs self-preference bias** — two distinct biases requiring different mitigations at different pipeline stages (Li et al., 2025; Zheng et al., NeurIPS 2023)
- **Bootstrap CIs at ceiling** — when the CI reflects sampling variability of a single discriminating pair, not a range of plausible true effect sizes; what evaluation designs actually fix the constraint

---

## Author

Yonas Mekonnen

- GitHub: [@Sanoy24](https://github.com/Sanoy24)
- Substack: [yonasmekonnen](https://open.substack.com/pub/yonasmekonnen)
