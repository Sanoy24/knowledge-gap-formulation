# Knowledge Gap Formulation for Compounding

A five-day paired research sprint that systematically closes understanding gaps in production AI systems — from LoRA adaptation mechanics to response-collapse detection — and publishes each finding as a standalone public explainer.

## What This Is

Building AI systems is fast. Understanding *why* they work the way they do is slow. This project takes the systems I shipped — a sales-agent evaluation benchmark, a fine-tuned judge model, a preference dataset — and audits them for gaps between what I built and what I can actually defend under scrutiny.

Each day follows the same loop:

1. **Surface a gap** — find the specific mechanism in my own shipped work that I cannot explain if pressed
2. **Sharpen it with a partner** — morning call to interrogate and tighten the question
3. **Research and write** — read canonical sources, run experiments, write an explainer for the partner's gap
4. **Revise and commit** — evening call for feedback, then commit a concrete improvement to existing portfolio work

Every research cycle produces two public artifacts (a blog post and a tweet thread) and one grounding commit that improves a previously shipped artifact.

## Grounding Target

All research is grounded against **[Tenacious-Bench](https://github.com/Sanoy24/tenacious-bench)** — a sales-agent evaluation benchmark featuring a fine-tuned judge model (Qwen2.5-3B, SimPO-trained on 618 preference pairs).

| Artifact | Link |
| --- | --- |
| Evaluation benchmark | [Sanoy24/tenacious-bench](https://github.com/Sanoy24/tenacious-bench) |
| Preference dataset | [sanoy24/tenacious_bench_v0.1](https://huggingface.co/datasets/sanoy24/tenacious_bench_v0.1) |
| Fine-tuned judge | [sanoy24/tenacious-judge-qwen25-3b-gamma15](https://huggingface.co/sanoy24/tenacious-judge-qwen25-3b-gamma15) |

---

## Published Explainers

### Blog Posts

| Day | Title | Link |
| --- | --- | --- |
| 1 | Why Your LLM Sales Agent Gives the Same Answer to Every Question | [Substack →](https://open.substack.com/pub/yonasmekonnen/p/why-your-sales-agent-gives-the-same) |

### Tweet / LinkedIn Threads

*Links added as threads ship.*

---

## Research Log

| Day | Topic | Gap Investigated | Partner | Grounding Commit |
| --- | --- | --- | --- | --- |
| 1 | Training & post-training mechanics | Why LoRA fine-tuning gradients are inherently low-rank, and what that means for rank selection on a 618-pair preference task | Amare Kassa | [`37b46c2`](https://github.com/Sanoy24/tenacious-bench/commit/37b46c2055b73834f951442a70118d28a0ee5a4b) — added intrinsic-dimensionality justification to `model_card.md` |

---

## Repository Structure

```
├── pair_DAY_1/          # Day 1 — LoRA rank selection & response collapse
│   ├── question.md              # Final sharpened question with artifact connection
│   ├── morning_call_summary.md  # How the question was interrogated and tightened
│   ├── explainer.md             # Blog post written for partner's gap
│   ├── thread.md                # Tweet thread, ready to publish
│   ├── evening_call_summary.md  # Feedback given and revisions made
│   ├── signoff.md               # Gap-closure judgment
│   ├── grounding_commit.md      # Pointer to concrete portfolio edit
│   └── sources.md               # Canonical sources and tools used
├── pair_DAY_2/ … pair_DAY_5/    # Same structure, one folder per day
│
├── synthesis.md          # Cross-week patterns across all gaps closed
├── canonical_list.md     # Annotated reading list — papers, tools, patterns
├── portfolio_update.md   # How grounding commits improve the shipped portfolio
└── docs/                 # Challenge specification and reference material
```

Each `pair_DAY_N/` folder captures the full research cycle: the question that was asked, the negotiation that sharpened it, the explainer that answered it, the feedback that revised it, and the commit that grounded it in real work.

---

## Key Concepts Explored

- **LoRA intrinsic dimensionality** — why low-rank adaptation works for narrow-domain fine-tuning, and how to reason about rank selection
- **Response collapse in LLM agents** — detecting mode collapse via pairwise embedding similarity, and fixing it through contrastive preference pairs
- **Reward model overoptimization** — how proxy reward signals diverge from task performance (Gao et al., 2022)
- **Degenerate text generation** — why likelihood maximization produces generic outputs (Holtzman et al., 2020)

---

## Author

**Yonas Mekonnen**

- GitHub: [@Sanoy24](https://github.com/Sanoy24)
- Substack: [yonasmekonnen](https://open.substack.com/pub/yonasmekonnen)
