# Sign-off - Day 2

**Asker:** Mamaru Yirga  
**Explainer:** Yonas Eshete  
**Question:** Token-level classification mechanics in `reply_agent.py`

---

## Gap closure judgment

The explainer closes the gap I asked about.

My question was not asking for generic classifier best practices. I wanted to understand what happens when my prompt-only LLM classifier appears to choose one of seven labels, and why keyword pre-screening handles most cases before the LLM fallback runs.

The explanation answered that directly. The core mechanism is now clear: the LLM path is next-token generation under label pressure, not a separate classification head. The prompt makes the seven labels likely, but it does not guarantee that the output will be a valid enum. That distinction explains why prompt-only JSON is weaker than constrained decoding or explicit enum validation.

## What changed in my understanding

Before this explainer, I treated the LLM fallback as if it were "doing classification" in a special way. After the explainer, I understand that it is still generating tokens. The label list in the prompt shapes the probability distribution, but the boundary is only reliable if the output is constrained or validated.

The keyword layer also makes more sense now. It is not a crude shortcut; it is the correct first tier for lexical intent. Replies like "not interested" or "send me pricing" do not need an LLM. The LLM is useful for compositional cases: mixed signals, contrastive phrasing, implied rejection, or replies where the surface keywords are misleading.

## Why this changes the artifact

The explainer gives me a concrete direction for improving `agent/llm/reply_agent.py`:

- keep keyword pre-screening for obvious lexical replies
- constrain or validate the `intent` field against the `ReplyIntent` enum
- leave `reasoning` and `key_quote` as freer natural-language fields
- log keyword-tier versus LLM-fallback performance before considering a fine-tuned classifier

The hands-on check against the actual repo made the issue visible. It showed that valid JSON such as `{"intent": "interested"}` can still fail the enum boundary, and that small phrasing differences can route a reply from keyword matching to LLM fallback.

## Final sign-off

I consider the gap closed. The explainer gives me both the mechanism and the implementation direction: do not rely on prompt-only JSON for the intent label; make the label boundary explicit through constrained decoding, structured output support, or enum validation.
