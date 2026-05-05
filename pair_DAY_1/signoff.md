# Sign-off — Day 1

**Asker:** Yonas Eshete

Gap closure: closed.

Before this I knew the LoRA formula — ΔW = B·A, scaled by alpha/rank but I couldn't explain why that approximation was valid in the first place. Rank 16 was in my model card because the Unsloth notebook had rank 16. That's it. What Amare's explainer gave me is the actual mechanism: fine-tuning a pretrained model doesn't learn new features, it reweights existing ones. The task is narrow, so the gradients concentrate along a few directions instead of spreading across the whole parameter space. The update is already low-rank before LoRA even enforces it. For a 618-pair single-domain preference task, the intrinsic dimension is probably 4–8. Rank 16 was above that threshold, so it worked but that's luck, not reasoning. I can now say why rank 16 wasn't wrong, what I'd actually start with if I retrained, and what a rank sweep would tell me. I couldn't say any of that yesterday.
