# Week 12 Synthesis

**Trainee:** Yonas Eshete
**Week:** 12 — Knowledge Gap Formulation
**Date:** 2026-05-08

---

## The Week in One Sentence

I spent four days finding the specific places where the language in my portfolio outran my actual understanding — and then closing those gaps with enough precision to make the corrections defensible under scrutiny.

---

## Eight Gaps Closed

### Gaps I Named (as asker)

#### Day 1 — Why low-rank decomposition is sufficient for narrow domain adaptation

My model card listed "LoRA rank 16, alpha 32" as bare numbers from a starter notebook. I could not explain why rank 16 was appropriate, whether rank 4 would have worked, or what a rank sweep would tell me. The gap was not "what does LoRA do?" — I knew the formula. The gap was why the formula is valid: what property of fine-tuning gradients makes low-rank decomposition sufficient in the first place. Amare's explainer gave me that mechanism. Fine-tuning a pretrained model does not learn new features — it reweights existing ones. The gradient updates for narrow domain adaptation concentrate along a few directions rather than spreading across the full parameter space, so the update is already low-rank before LoRA enforces it. For a 618-pair single-domain preference task, the intrinsic dimension is probably 4–8. Rank 16 was above that threshold, which is why it worked — but it was luck, not reasoning. I can now say why rank 16 was not wrong, what I would start with if I retrained, and what a sweep would reveal.

#### Day 2 — Why function-calling LLMs fabricate missing arguments instead of asking clarification questions

In my Week 10 Conversion Engine, the agent would fabricate an `order_id` and immediately call `cancel_pending_order` when the user had not provided one. I patched it with a SCAP postscript but did not understand why the problem existed. The gap was the mechanism: why does a function-calling model prefer hallucinated argument completion over clarification? The explainer I received traced this to training pressure toward action — RLHF data for function-calling models rewards tool execution over refusal, and the tool schema in context biases next-token probabilities toward argument-shaped continuations. Understanding the mechanism changed what fix I was looking for. A prompt patch fights the model's learned prior at inference time. Structural fixes — an explicit `ask_clarifying_question()` tool, required-argument validation before destructive calls, a planner routing step — work with the model's architecture rather than against it.

#### Day 3 — Which pairs actually contributed training signal during SimPO fine-tuning

My model card claimed "+2.27pp lift on held-out evaluation" and implied the adapter had improved judgment on adversarial probes. What I could not defend was the actual per-pair story: which pairs moved, which were silent from step zero, and whether the lift came from the cases the bench was built to stress. The diagnostic I ran after Charlie's morning-call sharpening showed 76 of 100 dev pairs were silent passengers (margin M ≥ 1.75 at step zero, contributing under 12% of max gradient). The adapter flipped 2 pairs — both non-adversarial easy edges. All 18 adversarial probes in the eval window were already correct at the base model level. The lift was real but it came from the wrong cases. My prediction before running the diagnostic was 60–80% of pairs in real-signal regime. The actual result was 9%. That inversion was the most important thing I learned all week.

#### Day 4 — Whether CI=[0.0%, 6.8%] at a 97.73% baseline is interpretable

My model card reported the bootstrap CI with a footnote saying the result was "not significant due to small n." Charlie's morning-call interrogation forced me to separate two distinct problems I had been treating as one: small n affects power; ceiling effects affect which test is appropriate at all. The specific question I committed to was whether the CI was interpretable given only one discriminating pair. Charlie's explainer showed the CI was mathematically correct but nearly uninformative: the 6.8% upper bound is the apparent improvement in the luckiest bootstrap resample where the critical pair is drawn three times. It is a resampling artifact, not an estimate of plausible true effect. More data on the same non-adversarial cases would not fix this. A different test on the same one-discriminating-pair data would not fix this either.

---

### Gaps I Researched (as explainer)

#### Day 1 — Why LLMs collapse to generic fallback responses in agent evaluation

My Day 1 partner's bench was designed to detect policy violations but the agent kept producing template-shaped outputs that scored well on surface features while failing the factual checks. The mechanism behind generic response collapse is in Holtzman et al. (2020): likelihood-maximizing decoding assigns high probability to the highest-frequency continuations in the model's training distribution — not the most contextually appropriate ones. For a sales agent, that means fluent, polite-sounding responses that avoid any specific commitment. The fix is not prompt engineering against individual template patterns; it is aligning what the training data rewards with what the held-out scorer checks. A proxy reward that does not match the evaluation objective will be exploited by the model regardless of how the templates are worded (Gao et al., 2022).

#### Day 2 — How LLM intent classification works at the token level, and why keyword pre-screening handles most cases

Mamaru's classifier used a prompt asking the model to return one of seven intent labels as JSON. He did not know whether the model was doing classification the way a trained classifier head does (picking from a fixed output layer) or something else. The mechanism is next-token generation under prompt pressure: the system prompt biases the probability distribution toward the label string continuations, but does not physically prevent invalid outputs. Keyword pre-screening handles 80% of cases because most sales replies express intent through explicit lexical markers — "not interested," "send me pricing," "tell me more." The LLM earns its cost only on the remaining 20% where intent depends on how phrases relate to each other rather than what individual words appear. That distinction between lexical and compositional intent is the load-bearing idea behind the 80/20 architecture.

#### Day 3 — Why a low chosen-side diversity ratio makes SimPO unable to generalize

Amir had 599 preference pairs but only 111 unique chosen strings, generated from a 16-template library. His trained model (pass rate 30%) underperformed both the prompted base (58%) and a regex heuristic (34%). The gap was why. The chosen side of preference pairs IS the implicit reward function — the model learns to approximate whatever patterns separate chosen from rejected. A 16-template chosen side teaches the model to produce template-surface markers ("to be upfront," "I want to be honest") rather than the factual constraints the held-out scorer checks. SimPO's gradient found the template separator early and optimized for it. The fix requires two changes together: LLM-generated chosen outputs for diversity, plus scorer-gating so every chosen output passes the evaluation before entering training. Diversity without gating produces misaligned data. Gating without diversity produces the same template collapse with a filter applied.

#### Day 4 — Whether cross-family rotation closes the self-preference bias path

Charlie had correctly implemented cross-family judge-generator rotation (JUDGE_ROUTING table in `judge_filter.py`). His question was whether rotation closed the self-preference path as well. It does not. Preference leakage (Li et al., 2025) is a data contamination problem — rotation fixes it. Self-preference bias (Zheng et al., 2023) is an inference-time problem: a judge scores outputs higher when they stylistically resemble its own generations, regardless of actual quality. Rotation governs which model judges which task at the data creation stage. It does not govern the evaluation phase. In Charlie's pipeline, Gemini both authors 90 bulk synthesis tasks and evaluates tone on those tasks via `_gemini_call()`. The same stylistic preferences are active at both ends of the pipe. The empirical test — stratify held-out scores by `generation_model` — was already implementable from Charlie's existing metadata and did not require any additional API calls.

---

## Most Surprising Thing

The Day 3 diagnostic inversion. Before running `diagnose_per_pair_margins.py`, I would have said the trained adapter improved performance on the adversarial probes, and that the +2.27pp held-out lift was probably driven by those cases. I expected 60–80% of pairs to be in the real-signal regime from step zero. The actual number was 9%. Seventy-six of one hundred dev pairs were silent passengers from the start of training. The two pairs that actually flipped were non-adversarial easy edges that had nothing to do with the adversarial probe set I had designed the bench around.

What made this genuinely surprising rather than just disappointing: the headline result (Delta A = 0.454, p = 0.001) is still real. The adapter substantially outperforms the base model. But the improvement is not coming from where I thought it was. The bench is at ceiling for the base model in pairwise log-prob mode on adversarial cases. The adapter is improving non-adversarial margin quality. Those are not the same thing as improving adversarial judgment.

The second thing that changed my thinking: the CI inversion from Day 4. I treated p = 0.372 as a small-n problem for months. Charlie's explainer showed the p-value is approximately P(absent) = (43/44)^44 ≈ 0.362 — the fraction of bootstrap iterations where the one discriminating pair was not drawn. The CI is not a range of plausible true effect sizes. It is the sampling variability of a single pair. The evaluation design itself, not the sample size, was the constraint.

---

## Canonical Reading List

Full annotated list in `canonical_list.md`. The papers and tools that changed how I think about production AI systems:

- **Holtzman et al. (ICLR 2020)** — why likelihood-maximizing decoding produces generic outputs; the diagnostic framing for response collapse
- **Gao, Schulman, Hilton (ICML 2023)** — reward model overoptimization; why proxy reward and true objective diverge under optimization pressure
- **Meng, Xia, Chen (NeurIPS 2024)** — SimPO mechanics; why the margin parameter γ creates a gradient dead zone for high-margin pairs
- **Li et al. (2025)** and **Zheng et al. (NeurIPS 2023)** — the two bias types that every LLM-as-judge pipeline needs separate mitigations for
- **Berg-Kirkpatrick et al. (EMNLP 2012)** and **Dror et al. (ACL 2018)** — when bootstrap CIs are informative and when the evaluation structure itself is the constraint
- **`ablations/diagnose_per_pair_margins.py`** — run this before claiming any fine-tuning result; it distinguishes signal from silence
