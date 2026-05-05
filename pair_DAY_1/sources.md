# Sources — Day 1

**Explainer writer:** Yonas Eshete
**Partner's gap:** Why LLMs collapse to generic fallback responses in sales agent evaluation

---

## Canonical Sources

1. **"The Curious Case of Neural Text Degeneration"**
    - Ari Holtzman, Jan Buys, Li Du, Maxwell Forbes, Yejin Choi — ICLR 2020
    - [arxiv.org/abs/1904.09751](https://arxiv.org/abs/1904.09751)
    - Why cited: grounds the claim that greedy/likelihood-maximizing decoding produces repetitive, high-probability templates rather than specific, task-grounded outputs — the core mechanism behind generic response collapse.

2. **"Scaling Laws for Reward Model Overoptimization"**
    - Leo Gao, John Schulman, Jacob Hilton — 2022
    - [arxiv.org/abs/2210.10760](https://arxiv.org/abs/2210.10760)
    - Why cited: grounds the claim that optimizing harder against a proxy reward signal eventually degrades true objective performance — the mechanism explaining why a fine-tuned judge trained on fluency-biased preference data learns to pass generic responses.

---

## Tool Used Hands-On

- **Tool:** `sentence-transformers` library, model `all-MiniLM-L6-v2`
- **What was done:** Built and tested the `detect_response_collapse` function in the explainer — encodes a batch of model outputs into normalized embeddings, computes the pairwise cosine similarity matrix, and returns the average similarity and a collapse flag. Verified the function produces expected output on a small set of paraphrased vs. distinct responses.
- **Output or evidence:** Runnable code included directly in `explainer.md`. Install: `pip install sentence-transformers numpy`. No external dependencies beyond those two.

---

## Follow-on Reading

- Li et al. (2022), "Contrastive Decoding: Open-ended Text Generation as Optimization" — a decoding strategy that explicitly down-weights tokens assigned high probability regardless of context, attacking the generic response problem at inference time without retraining.
- Stiennon et al. (2020), "Learning to summarize with human feedback" — the paper that first clearly showed how RLHF reward models can be gamed, predating Gao et al.'s scaling analysis.
