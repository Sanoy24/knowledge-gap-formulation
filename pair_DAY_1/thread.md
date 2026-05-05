# Tweet / LinkedIn Thread — Day 1

By Yonas Eshete | Post-evening-call revision

---

**[Tweet 1 — Hook, stands alone]**

Your LLM sales agent gives the same answer to everything.

Weak-evidence → "30-minute call?"
Staffing overclaim → "30-minute call?"
Tone drift → "30-minute call?"

Same output. Every probe. Judge marks it passed.

Here's why and how to catch it 🧵

---

**[Tweet 2 — The mechanism]**

LLMs maximize log-likelihood. "Schedule a call?" fits every professional context — it's the statistical mode.

Greedy decoding always lands there.

Holtzman et al. (2020): optimize for likelihood, get templates that fit everywhere.
arxiv.org/abs/1904.09751

---

**[Tweet 3 — Why RLHF doesn't fix it]**

RLHF makes it worse.

Human raters score fluent, professional responses higher than ones that engage with adversarial constraints.

The model learned: generic = high reward.

Nothing in preference training penalizes ignoring the specific failure category.

---

**[Tweet 4 — Detection]**

Measure it first. If outputs cluster in embedding space across different probes, you have collapse.

avg_sim = (emb @ emb.T).sum() / (n\*(n-1))

avg_pairwise_similarity > 0.90 = confirmed.

Full runnable detector in the blog.

---

**[Tweet 5 — The judge problem + fix]**

The judge has the same bias. Trained on fluent-first preference data, it passes generic responses.

Fix: add a specificity criterion.

"Does this address the specific constraint? Score 0 if it could appear for any input."

One prompt change.
arxiv.org/abs/2210.10760

---

**[Tweet 6 — Link + summary]**

Full explainer:

→ why likelihood maximization collapses to the same output
→ why RLHF doesn't fix it
→ collapse detector (runnable code)
→ judge specificity fix (zero cost)
→ contrastive negatives (root cause fix)

blog link: https://open.substack.com/pub/yonasmekonnen/p/why-your-sales-agent-gives-the-same
