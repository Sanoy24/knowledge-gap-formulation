# Day 2 — Grounding Commit

**Artifact edited:** `agent/llm/reply_agent.py` — `classify_reply()` function and `ClassifiedReply` model

**Commit link:** https://github.com/mamee13/signal-driven-sales-conversion-engine/commit/94b31a9e84bd54604d9f64d70bc59681339195c1

**What changed and why:**

The explainer identified three concrete problems in the intent classifier and recommended three fixes. All three are now applied.

**Change 1 — Enum repair + markdown fence stripping in the LLM fallback.**
The original `json.loads(response.content)` call would silently fall to `unclear` on any parse failure, including recoverable ones like `"Rejection"` (wrong case) or JSON wrapped in a markdown fence. The fix strips markdown fences before parsing, then attempts case-normalisation before failing closed to `unclear`. This is the structural fix the explainer called "constrain the intent field" — the prompt alone does not prevent invalid labels; the boundary code must handle them. The explainer's worked example showed `{"intent": "Rejection"}` parses as valid JSON but fails the enum, which is exactly the case this repair handles.

**Change 2 — Tier logging via `classification_tier` field on `ClassifiedReply`.**
Every classification now records `"keyword"` or `"llm"` in a new `classification_tier` field. The explainer's recommendation was explicit: "Every classification should record whether it came from keywords or LLM fallback. Over time, that gives you real evidence for the 80/20 claim instead of relying on intuition." Without this field, the 80% pre-screening claim in the Week 11 model card is an estimate. With it, the claim becomes measurable from logs.

**Change 3 — Removed `"interested"` from `positive_keywords`.**
The explainer flagged this keyword as too broad: it appears in negative sentences like "I am not interested enough to continue." The rejection tier runs first, so `"not interested"` is caught correctly, but a reply like "I am not interested enough to justify the switch right now" would have matched `"interested"` and returned `POSITIVE_INTEREST` with 0.88 confidence — a false positive. Removing it sends these ambiguous cases to the LLM fallback, which is the right tier for compositional intent.
