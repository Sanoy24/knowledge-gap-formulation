# Day 2 — Sign-Off

**Asker:** Mamaru Yirga

**Gap closure status:** Closed

**What I understand now that I didn't before:**

Before reading this explainer, I assumed the model "classified" intent the way a trained classifier head does — picking from a fixed set of outputs. I now understand the actual mechanism: the LLM fallback in `classify_reply()` is next-token generation under prompt pressure, not a separate classification module. The seven label strings work because the system prompt biases the next-token probability distribution toward those continuations. But the prompt does not physically prevent invalid outputs — the model can still generate `"interested"`, `"Rejection"`, or JSON wrapped in a markdown fence, all of which would silently fail my original `json.loads()` call.

The second insight that closed the gap is the lexical vs compositional distinction. Keyword rules add value when intent is visible in a phrase — `"not interested"`, `"send me pricing"`, `"tell me more"`. The LLM adds value when intent depends on how phrases relate to each other — contrast, qualification, implied refusal. That distinction explains why 80% pre-screening is not a hack: most sales replies have explicit lexical intent, and a deterministic rule is faster, cheaper, and more predictable than an LLM call for those cases. The LLM earns its cost only on the ambiguous 20%.

The third thing I now understand is that the `"interested"` keyword in my positive list was a latent false-positive risk. The explainer's worked example made it concrete: `"I am not interested enough to continue"` would have matched `"interested"` and returned `POSITIVE_INTEREST` with 0.88 confidence. The rejection tier runs first, so `"not interested"` is caught, but the broader `"interested"` substring is not safe. Removing it and letting those cases fall to the LLM is the right call.

The grounding commits — enum repair, tier logging, and keyword removal — are all direct consequences of understanding the mechanism, not just following a recommendation.
