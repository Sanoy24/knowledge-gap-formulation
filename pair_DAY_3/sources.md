# Sources — Day 3

**For:** [`explainer.md`](explainer.md) (Yonas Eshete writing for Amir Ahmedin)

## Canonical sources cited

### 1. Meng, Xia, Chen — _SimPO: Simple Preference Optimization with a Reference-Free Reward_

NeurIPS 2024. arXiv: [2405.14734](https://arxiv.org/abs/2405.14734).

This is the algorithm Amir's run used. The explainer cites it for the loss formulation `L = −log σ(β · (1/|y_w|·log π(y_w|x) − 1/|y_l|·log π(y_l|x)) − γ)` and for the reference-free framing that drops DPO's `log(π/π_ref)` term. Load-bearing because the central argument hinges on what the loss is actually optimizing: with no reference anchor, the gradient direction is determined entirely by what separates chosen from rejected in the training data. When that separator is template-surface markers, the policy walks toward template-surface markers.

### 2. Rafailov, Sharma, Mitchell, Manning, Ermon, Finn — _Direct Preference Optimization: Your Language Model is Secretly a Reward Model_

NeurIPS 2023. arXiv: [2305.18290](https://arxiv.org/abs/2305.18290).

The contrast paper. DPO's `log(π/π_ref)` term anchors the policy to the base model's distribution, which provides implicit regularization against drifting too far off-distribution during training. The explainer cites it for the reference-model contrast: if Amir's failure mode were pure mode collapse without an upstream data problem, re-introducing DPO's reference term (or an equivalent KL penalty) would help. The explainer argues this is downstream of the data fix, not a substitute for it.

### 3. Gao, Schulman, Hilton — _Scaling Laws for Reward Model Overoptimization_

ICML 2023. arXiv: [2210.10760](https://arxiv.org/abs/2210.10760).

Load-bearing for the central mechanism in the explainer. The paper studies what happens when a policy is optimized against a reward signal that is misaligned with the true objective: the policy exploits the reward's blind spots rather than learning the underlying task. The explainer reuses this framing for preference data, treating the chosen-side strings as the implicit reward function. A narrow chosen-side (111 unique strings, 16 templates) is a narrow reward, and Gao et al.'s prediction is that optimization against a narrow reward produces a narrow policy. That is exactly the pattern Amir's numbers show: 30% trained < 34% heuristic < 58% prompted base.
