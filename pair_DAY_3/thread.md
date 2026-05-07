# Tweet Thread — Day 3

## 1/6

You trained a SimPO judge on 600 preference pairs. Reward accuracy hit 1.0 at step 100. Eval loss came down clean.

On held-out, your trained model loses to a regex script (30% vs 34%) and to its own untrained backbone by 28pp.

What broke? Not the trainer. The data.

## 2/6

Preference optimization (SimPO, DPO, ORPO, KTO, IPO) all push the policy to assign higher probability to chosen than rejected. The signal it sees is "this string chosen, that string rejected."

Whatever surface feature separates the two clusters becomes the gradient direction.

## 3/6

My partner's 599 chosen outputs collapsed to 111 unique strings, all generated from a 16-template library. Strongest separator: phrasing markers ("to be upfront" vs "Guaranteed!").

The model learned the markers. The held-out scorer checks bench honesty, timezone, banned phrases. Different objective.

## 4/6

This is reward overoptimization (Gao, Schulman, Hilton, ICML 2023) showing up in a different form.

Their paper studies it with explicit reward models. Same dynamic appears when your "reward model" IS your chosen-side data: a narrow reward produces a narrow policy.

## 5/6

The fix is two changes in your data pipeline, run together:

1. Generate chosen outputs via real LLM call, not template lookup
2. Validate every chosen against the held-out scorer; drop failures

Diversity without alignment fails. Alignment without diversity is template-collapse. You need both.

## 6/6

Diversity threshold from two real datasets I checked: target unique-chosen-to-total ratio ≥ 0.9. Danger floor 0.5. Below 0.5, half your pairs share chosen strings and the gradient overweights duplicates.

Full blog with diagnostic snippet + canonical sources: [\[BLOG_LINK\]](https://yonasmekonnen.substack.com/p/your-chosen-side-training-data-is)
