# Canonical List

**Trainee:** Yonas Eshete
**Week:** 12 — Knowledge Gap Formulation
**Grounding target:** Tenacious-Bench (Sales Agent Evaluation Bench)

---

## Papers

| # | Title | Authors | Venue | Year | Link | Why it matters to an FDE |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | The Curious Case of Neural Text Degeneration | Holtzman, Buys, Du, Forbes, Choi | ICLR 2020 | 2020 | [arXiv:1904.09751](https://arxiv.org/abs/1904.09751) | Explains why greedy and likelihood-maximizing decoding produce repetitive, template-shaped outputs. Load-bearing for diagnosing generic response collapse in any generation pipeline. |
| 2 | Scaling Laws for Reward Model Overoptimization | Gao, Schulman, Hilton | ICML 2023 | 2023 | [arXiv:2210.10760](https://arxiv.org/abs/2210.10760) | Quantifies how optimizing harder against a proxy reward eventually degrades the true objective. Reach for this whenever a client's fine-tuned model scores well in training but degrades on held-out evaluation. |
| 3 | SimPO: Simple Preference Optimization with a Reference-Free Reward | Meng, Xia, Chen | NeurIPS 2024 | 2024 | [arXiv:2405.14734](https://arxiv.org/abs/2405.14734) | The reference implementation for SimPO. Explains the length-normalized reward, the margin hyperparameter γ, and why pairs with high average log-probability from step zero contribute near-zero gradient. Required reading before tuning β and γ. |
| 4 | Direct Preference Optimization: Your Language Model is Secretly a Reward Model | Rafailov, Sharma, Mitchell, Manning, Ermon, Finn | NeurIPS 2023 | 2023 | [arXiv:2305.18290](https://arxiv.org/abs/2305.18290) | The DPO derivation. Needed alongside SimPO to understand what the reference model term does and why removing it changes the gradient landscape. |
| 5 | Preference Leakage: A Contamination Problem in LLM-as-a-Judge | Li, Su, Zhao, Ding, Zhang et al. | arXiv 2025 | 2025 | [arXiv:2502.01534](https://arxiv.org/abs/2502.01534) | Defines the three-leakage-vector taxonomy (same-model, same-family, prompt-template) and explains exactly what cross-family rotation does and does not fix. Reach for this before designing any LLM-as-judge evaluation pipeline. |
| 6 | Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena | Zheng, Chiang, Sheng et al. | NeurIPS 2023 | 2023 | [arXiv:2306.05685](https://arxiv.org/abs/2306.05685) | Section 3.3 documents self-preference, position, and verbosity bias empirically across multiple model families. The canonical reference for why cross-family rotation is necessary but not sufficient for evaluation integrity. |
| 7 | Efficient Guided Generation for Large Language Models | Willard, Louf | arXiv 2023 | 2023 | [arXiv:2307.09702](https://arxiv.org/abs/2307.09702) | Explains constrained decoding via regex and context-free grammars. Relevant whenever an LLM output field has a fixed schema — enum labels, structured JSON, boolean flags. Use to replace prompt-only constraints with boundary-code enforcement. |
| 8 | Let Me Speak Freely? A Study on the Impact of Format Restrictions on Performance of Large Language Models | Tam, Wu, Tsai, Lin, Lee, Chen | arXiv 2024 | 2024 | [arXiv:2408.02442](https://arxiv.org/abs/2408.02442) | Shows where format constraints help and where they hurt. Frames the tradeoff: constrain enum labels, leave reasoning fields free. Useful whenever you are deciding what to structure and what to leave as natural language. |
| 9 | An Empirical Investigation of Statistical Significance in NLP | Berg-Kirkpatrick, Burkett, Klein | EMNLP 2012 | 2012 | — | Shows that p-values from paired bootstrap depend on the number of discriminating items, not n alone. Reach for this before reporting bootstrap CIs on any NLP evaluation with a high baseline. |
| 10 | The Hitchhiker's Guide to Testing Statistical Significance in NLP | Dror, Baumer, Shlomov, Reichart | ACL 2018 | 2018 | [arXiv:1809.01448](https://arxiv.org/abs/1809.01448) | Section 4 covers ceiling effects and when bootstrap CIs are informative. Table 2 gives guidance on when to switch from paired bootstrap to permutation tests or continuous scoring. |

---

## Tools

| # | Tool | What it does | When to use it |
| --- | --- | --- | --- |
| 1 | `sentence-transformers` (`all-MiniLM-L6-v2`) | Encodes text into normalized embeddings; pairwise cosine similarity detects when outputs cluster together | Diagnosing response collapse: if average pairwise similarity across a generation batch exceeds ~0.90, the model is producing template-shaped outputs |
| 2 | `ablations/diagnose_per_pair_margins.py` | Per-pair margin diagnostic for SimPO-trained models: computes base margin, adapter margin, gradient strength bucket, and whether pairs flipped | Run before claiming any fine-tuning result — distinguishes pairs that actually contributed training signal from silent passengers at ceiling |
| 3 | `stratify_by_generation_model` (pattern, not a library) | Partitions held-out scores by the `generation_model` metadata field and compares means across model families | Detecting self-preference bias in LLM-as-judge pipelines: if Gemini-authored tasks score materially higher under Gemini evaluation, that gap is the empirical measure of self-preference |
| 4 | TRL `CPOTrainer` with SimPO loss | SimPO preference optimization without a reference model; configurable β and γ via `CPOConfig` | Narrow-domain preference fine-tuning where reference model overhead is not warranted and you want explicit margin control |

---

## Engineering Patterns

| # | Pattern | What problem it solves | Reference |
| --- | --- | --- | --- |
| 1 | Cross-family judge-generator rotation | Prevents preference leakage by ensuring the generation model and judge model are always from different families | Li et al. (2025); `generation/judge_filter.py` JUDGE_ROUTING table pattern |
| 2 | Evaluation-phase rotation (extension of above) | Cross-family rotation applied to the evaluation stage, not just data creation; closes self-preference path that survives standard rotation | Zheng et al. (2023) Section 3.3; pair_DAY_4/explainer.md |
| 3 | Keyword-first, LLM-fallback intent classification with tier logging | Handles 80% of cases deterministically at near-zero cost; reserves LLM calls for compositionally ambiguous intent; tier field makes the 80/20 claim measurable from logs | Willard & Louf (2023); pair_DAY_2/explainer.md |
| 4 | Scorer-gating on preference pair generation | Ensures training reward and held-out reward are aligned at the data level; prevents chosen-side template collapse | Gao et al. (2023); pair_DAY_3/explainer.md |
| 5 | Adversarial-slice evaluation (not random held-out) | For tasks where the base model is already near ceiling, a random held-out partition leaves one discriminating pair; an adversarial slice where base accuracy is 60–70% produces real signal | Berg-Kirkpatrick et al. (2012); pair_DAY_4/signoff.md |
