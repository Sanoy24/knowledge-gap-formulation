# Grounding Commit — Day 1

**Artifact edited:** `model_card.md` in `Sanoy24/tenacious-bench` on GitHub

**Commit:** `37b46c2055b73834f951442a70118d28a0ee5a4b`
**Commit URL:** [37b46c2](https://github.com/Sanoy24/tenacious-bench/commit/37b46c2055b73834f951442a70118d28a0ee5a4b)

**What changed and why:**

The Technical Specifications section previously listed "LoRA Rank: 16, LoRA Alpha: 32" as bare numbers with no justification values copied from Unsloth starter notebook defaults. The update adds a one-paragraph explanation grounded in the mechanism Amare's explainer revealed: fine-tuning a pretrained model does not learn new features, it reweights existing ones, and the resulting gradient updates are intrinsically low-rank. For a 618-pair narrow-domain preference task distinguishing a small number of latent factors (persuasion quality, tone alignment, constraint violations), the effective adaptation signal likely lies in the rank 4–8 range. Rank 16 is now described as a conservative upper bound that ensures sufficient capacity without constraining optimization — not an optimal choice, but a defensible one. A rank sweep would likely confirm the extra capacity went unused.
