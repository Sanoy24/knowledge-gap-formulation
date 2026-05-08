# Sources — Day 4

**For:** [`explainer.md`](explainer.md) (Yonas Eshete writing for Charlie Lijalem)

## Canonical sources cited

### 1. Li, Su, Zhao, Ding, Zhang et al. — *Preference Leakage: A Contamination Problem in LLM-as-a-Judge*

arXiv: [2502.01534](https://arxiv.org/abs/2502.01534). 2025.

This paper is already cited in Charlie's own repo — `Sales-Agent-Evaluation-Bench/papers/path_specific/path_b/preference_leakage_memo.md` applies its three-leakage-vector taxonomy (same-model, same-family, prompt-template) to audit the Tenacious pipeline. Load-bearing in the explainer because it establishes the precise definition of **preference leakage** that cross-family rotation is designed to fix — and which is distinct from self-preference bias. Without this distinction the question cannot be answered, because the two biases require different mitigations at different pipeline stages.

### 2. Zheng, Chiang, Sheng, Zhuang, Wu, Zhuang, Lin, Li, Li, Xing, Zhang, Gonzalez, Stoica — *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*

NeurIPS 2023. arXiv: [2306.05685](https://arxiv.org/abs/2306.05685).

Section 3.3 of this paper documents **self-preference bias** empirically across multiple model families: GPT-4 consistently rates GPT-4 outputs higher than human annotators do, Claude consistently rates Claude outputs higher, and the gap is systematic across dimensions including tone and instruction-following quality. This is the canonical empirical reference for why cross-family rotation is necessary but not sufficient — it closes preference leakage but leaves self-preference untouched because self-preference operates at inference time, not at the data contamination level. The paper is load-bearing in the explainer's central argument: the distinction between what rotation fixes and what it does not.

## Tool / pattern used

A stratification script that partitions held-out scores by the `generation_model` metadata field and compares means across model families. The script is in the "Empirical test you can run today" section of the explainer. It uses:

- `ablation_results.json` — per-task evaluation scores already computed, no additional API calls needed
- `generation_model` field logged per task in `generation/judge_filter.py:334-336`

If Gemini-authored tasks (90 bulk multi-LLM synthesis tasks) score materially higher under Gemini evaluation than DeepSeek-authored or trace-derived tasks, that gap is the empirical measure of self-preference operating in the evaluation phase.

## Repo files that were load-bearing in the analysis

All files read in Charlie's `Sales-Agent-Evaluation-Bench/` repo:

| File | Role in explainer |
| --- | --- |
| `generation/judge_filter.py:72-81` | JUDGE_ROUTING table — verifies cross-family rotation is correctly implemented for task creation |
| `generation/judge_filter.py:229-233` | `get_judge_model()` — confirms routing is enforced at runtime |
| `generation/judge_filter.py:334-336` | `generation_model` and `judge_model` logged per task — confirms stratification is possible |
| `generation/scripts/multi_llm_synthesis.py:14-15` | DeepSeek (hard seeds) + Gemini (bulk variants) as generators — confirms 90 Gemini-authored tasks exist |
| `evaluation/scoring_evaluator.py:283` | `_gemini_call()` — confirms Gemini 2.5-flash is the evaluation-phase judge for LLM-backed dimensions |
| `evaluation/scoring_evaluator.py:214-266` | `tone_checker_fn()` and `objection_ack_fn()` — the two dimensions where self-preference operates |
| `training/generate_preference_pairs.py:12` | Gemini generates chosen outputs — confirms the closed self-preference loop in Path 2 |
| `model_card.md:82` | Explicit acknowledgment: "Cross-family leakage prevention was not applied (Gemini both generates and judges)" |
| `papers/path_specific/path_b/preference_leakage_memo.md:20-42` | Pipeline audit and recommended mitigation (stratify by generation_model) — already names part of the gap |
| `papers/common/llm_judge_memo.md:44-46` | Acknowledges Claude Haiku/Sonnet within-family pair and self-preference risk |
| `dataset/inter_rater_agreement.md` | κ=1.000 after two-tier rule fix — confirms IRA partially but not fully mitigates |
| `ablation_results.json` | Delta A=0.454 (p=0.001) — confirms headline finding is robust to self-preference bias |

## What I deliberately did not cite

There is a 2024 paper commonly cited on self-preference — "LLM Evaluators Recognize and Favor Their Own Generations" (Panickssery et al.) — that is directly relevant. I chose not to cite it because I am not sufficiently confident in the exact arXiv ID to include it without risk of hallucination. The Zheng et al. (2023) paper documents the same phenomenon with equally strong empirical evidence and I am certain of its arXiv ID. For a follow-on reader who wants to go deeper, the Panickssery paper is worth searching for, but I am not placing it in this sources file.
