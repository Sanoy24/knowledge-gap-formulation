# Evening call summary — Day 1

Written by Amare Kassa | Confirmed by Yonas Eshete

> **Note:** Two explainers were written for Day 1. Yonas wrote the response-collapse explainer for Amare's question. Amare wrote the LoRA explainer for Yonas's question. Each section below records the asker's feedback and the writer's revisions.

## Feedback on the LoRA explainer (Yonas → Amare)

Yonas flagged three things before the explainer could publish. First, no runnable code: the draft had pseudocode (`load_pretrained_model()`, `apply_lora()`) and the publication checklist requires something a reader can actually execute. Second, both citations (Hu et al. 2021, Li et al. 2018) were missing arxiv links. Third, the format read as a structured outline — bold inline headers, bullet-point lists, Title Case throughout — rather than a blog post.

What landed: the three-reason structure for why fine-tuning gradients are low-rank, the connection between intrinsic task dimensionality and rank selection, and the model card replacement text at the bottom that Yonas could paste directly.

I revised the explainer to add a runnable rank-sweep snippet showing r=4 vs r=8 vs r=16, added arxiv URLs for both papers, and rewrote the bullet-heavy sections as flowing prose. The expected-output block (`{4: 0.81, 8: 0.82, 16: 0.82}`) was kept to make the saturation argument concrete.

## Feedback on the response-collapse explainer (Amare → Yonas)

Yonas's explainer landed for me with no major revisions needed. The likelihood-maximization argument was the load-bearing piece I was missing, and the framing — that the judge inherits the same fluency bias as the model when trained on the same preference data — was the part that actually closed the gap. The collapse detector code was directly applicable to my `held_out_traces.jsonl` setup.

What I asked for in the call: a clearer ordering of fixes by cost, since I needed to know what to do first when I get back to the code. Yonas already had a "what to do, in cost order" section but I asked him to make the ordering explicit (detector first, judge prompt change second, contrastive negatives third — only if the first two don't resolve it). He tightened that paragraph. No other revisions requested.
