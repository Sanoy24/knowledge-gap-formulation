# How your intent classifier actually picks one of seven strings

Written by Yonas Eshete | For Amare Kassa's question

You asked two things.

First: when your `classify_reply()` path falls through to the LLM, what is happening at the token level when the model chooses one of the seven intent labels from `_CLASSIFY_SYSTEM`?

Second: why does the keyword pre-screening layer handle most replies before the LLM runs, and what does that say about when the LLM actually adds value?

The short answer:

The current prompt does not make the model "classify" in the way a small classifier head would. It makes the model generate text. The seven labels work because the prompt strongly biases the next-token distribution toward those label strings. But the prompt alone does not guarantee valid labels. Only constrained decoding, structured output support, or post-generation validation can do that.

## The mechanism: next-token prediction under label pressure

When `classify_reply()` reaches the LLM fallback, it sends two messages:

- a system prompt listing seven valid labels
- a user message containing the cleaned reply body

The model then generates output autoregressively, one token at a time.

By the time it has generated something like:

```json
{"intent": "
```

it has reached the label decision point. Internally, the final layer produces logits: raw scores over the model's vocabulary. Softmax turns those scores into probabilities:

```text
P(token_i) = exp(logit_i) / sum(exp(logit_j))
```

If the reply says:

```text
not interested, please remove me from your list
```

then tokens that continue toward `rejection` should receive much higher probability than tokens that continue toward `positive_interest` or `scheduling_request`, because the prompt has named the allowed labels and the reply contains a very strong rejection phrase.

One important detail: the model may not emit the full label as one token. Depending on the tokenizer, a label like `positive_interest` may be split into multiple tokens. So the model is not always choosing the whole string in one step. It is walking token by token through the label.

That means the plain-language mechanism is:

**LLM classification here is next-token generation constrained by prompt pressure, not a separate classification module.**

The prompt narrows the likely continuations. It does not physically prevent invalid continuations.

That distinction matters because your current implementation asks for JSON and then parses the result. If the model outputs `"interested"`, `"Rejection"`, or fenced markdown around the JSON, your parser can still fail. The prompt makes valid output likely. It does not make valid output guaranteed.

## Why keywords handle most of the work

Your keyword layer is not a hack. It is a cheap deterministic classifier for replies where intent is visible at the surface level.

For example:

```text
"not interested, please stop emailing me"
```

The intent is carried by two phrases: `not interested` and `stop emailing`. The rest of the message does not add much. Your keyword layer can classify that as `rejection` without paying for an LLM call.

The same is true for many common sales replies:

```text
"send me pricing"        -> capacity_question
"send me a link?"        -> scheduling_request
"remove me"              -> rejection
"tell me more"           -> positive_interest
```

If the reported 80 percent pre-screening rate holds across logged replies, it means most replies in this workflow are lexically separable. The intent is available from a small phrase, not from subtle sentence structure.

That also explains why the LLM is not automatically more valuable on those examples. A large model would likely reach the same answer, but with higher latency and cost.

I would avoid claiming that specific attention heads definitely focus on those exact phrases unless we inspect attention traces. The safer statement is behavioral: phrases like "not interested" and "remove me" are highly predictive of rejection in ordinary language, and your keyword rules capture that signal directly.

## When the LLM actually earns its cost

The LLM adds value when intent depends on relationships between phrases, not just the presence of a phrase.

Here are the cases where keywords become brittle:

```text
"Interesting, though I am not sure this is the right time."
```

This contains `interesting`, but the actual intent is closer to an objection or hesitation.

```text
"I appreciate the outreach, but we just signed a three-year deal with Toptal."
```

This has no obvious rejection keyword from your list, but the meaning is rejection or objection.

```text
"We are looking to scale the team, but how much does this cost?"
```

This contains buying interest and a capacity/pricing question. The correct label depends on which part of the reply should drive the next action.

In these cases, the LLM helps because it can represent sentence-level structure: contrast, qualification, implied refusal, and competing signals. Your keyword layer matches substrings. The LLM can use the surrounding language to decide which signal dominates.

So the answer to your second question is:

**Keyword rules add value when intent is lexical. The LLM adds value when intent is compositional.**

Lexical means the intent is mostly in a phrase. Compositional means the intent depends on how phrases relate to each other.

## The fragile part: prompt-only JSON

Your current fallback asks the model for JSON, then parses it with `json.loads()`.

That is the fragile boundary. Prompt-only JSON can fail in two different ways:

1. The format is invalid.
2. The format is valid, but the label is outside your enum.

Examples:

```json
{"intent": "interested"}
```

This is valid JSON, but `interested` is not one of your seven labels.

```json
{"intent": "Rejection"}
```

This is valid JSON, but the capitalization does not match the enum.

````text
```json
{"intent": "rejection"}
```
````

This may be semantically correct, but `json.loads()` fails unless you strip the markdown fence first.

The fix is not "write a scarier prompt." The structural fix is to constrain or validate the output at the boundary.

## Hands-on check against your actual file

To avoid making this purely theoretical, I ran a small local Python check against:

```text
signal-driven-sales-conversion-engine/agent/llm/reply_agent.py
```

The script parsed the file with Python's `ast` module, extracted the real `ReplyIntent` enum values and keyword lists, then ran a few sample replies through the same keyword-list order used by `classify_reply()`. It also checked whether three possible LLM JSON outputs are valid JSON and valid enum labels.

This is the output:

```text
VALID_LABELS= ['positive_interest', 'scheduling_request', 'objection', 'rejection', 'capacity_question', 'wrong_signal', 'unclear']
KEYWORD_LIST_SIZES= {'rejection_keywords': 12, 'positive_keywords': 11, 'scheduling_keywords': 9, 'capacity_keywords': 10, 'wrong_signal_keywords': 14, 'sms_keywords': 4}
{"reply": "not interested, please stop emailing me", "tier_result": "rejection", "matched_keyword": "not interested"}
{"reply": "send me pricing", "tier_result": "capacity_question", "matched_keyword": "pricing"}
{"reply": "can you send a link?", "tier_result": "LLM fallback", "matched_keyword": null}
{"reply": "tell me more", "tier_result": "positive_interest", "matched_keyword": "tell me more"}
{"reply": "Interesting, though I am not sure this is the right time.", "tier_result": "LLM fallback", "matched_keyword": null}
{"raw": "{\"intent\": \"interested\"}", "json_valid": true, "enum_valid": false}
{"raw": "{\"intent\": \"Rejection\"}", "json_valid": true, "enum_valid": false}
{"raw": "{\"intent\": \"rejection\"}", "json_valid": true, "enum_valid": true}
```

This makes two things visible.

First, the keyword tier really is a deterministic lexical gate. It catches exact phrases such as `not interested`, `pricing`, and `tell me more`. It misses nearby phrasings if the exact substring is absent. For example, `can you send a link?` falls to the LLM because the actual list contains `send me a link`, not `send a link`.

Second, valid JSON is not the same as a valid intent. `{"intent": "interested"}` parses, but it is not in `ReplyIntent`. That is the concrete boundary failure constrained decoding or enum validation is meant to fix.

## Worked example: unconstrained vs constrained label choice

This is a small worked example to make the mechanism visible. It is not a live trace from DeepSeek; it shows the decoding logic.

Assume the valid labels are:

```text
positive_interest
scheduling_request
objection
rejection
capacity_question
wrong_signal
unclear
```

The reply is:

```text
"Interesting, but we just signed with another vendor."
```

An unconstrained generation path might produce:

```json
{"intent": "interested"}
```

The model reached for a natural word, but that word is not in the enum.

With constrained label decoding, the decoder tracks the valid label strings. At the label position, invalid continuations are blocked:

```text
Allowed:
positive_interest
scheduling_request
objection
rejection
capacity_question
wrong_signal
unclear

Blocked:
interested
maybe_interested
vendor_selected
Rejection
```

For multi-token labels, this happens step by step. If the model starts with `pos`, only continuations that can still complete `positive_interest` remain valid. If it starts with `obj`, only continuations toward `objection` remain valid.

That is the core idea behind constrained decoding: do not merely ask the model to obey the enum; make invalid enum outputs unavailable during generation.

## What constrained decoding changes

Willard and Louf (2023) describe guided generation as a way to restrict generation with a grammar or finite-state machine. In your case, the grammar can be tiny: one of seven labels.

Conceptually:

```python
from enum import Enum

class ReplyIntent(str, Enum):
    positive_interest = "positive_interest"
    scheduling_request = "scheduling_request"
    objection = "objection"
    rejection = "rejection"
    capacity_question = "capacity_question"
    wrong_signal = "wrong_signal"
    unclear = "unclear"

valid_labels = [intent.value for intent in ReplyIntent]
```

The decoder should only allow completions from `valid_labels` at the intent field. Whether you implement that with a local constrained decoding library, provider-native structured outputs, or strict post-generation validation depends on the model provider and runtime.

The important design point is provider-independent:

**The intent label should be constrained. The explanatory fields can stay freer.**

Your `ClassifiedReply` model also asks for `confidence`, `key_quote`, and `reasoning`. Those fields are useful, but they are not the decision boundary. If you strictly constrain the entire output, you may make the reasoning less natural. A practical pattern is:

1. Generate or extract the intent using a constrained enum.
2. Generate `key_quote` and `reasoning` as ordinary text.
3. Validate the final object with `ClassifiedReply`.

Tam et al. (2024) discuss the tradeoff: format restrictions can hurt tasks that need free-form reasoning, but they can help when the desired output is a fixed class label. Your intent label is exactly the kind of field that benefits from constraint.

## Why this does not require a fine-tuned model yet

This mechanism also explains why your intent classifier does not obviously need a fine-tuned model right now.

The easy cases are already handled by deterministic rules. Fine-tuning a model to relearn `remove me -> rejection` would not add much. Your code already knows that.

The hard cases are the ambiguous 20 percent: contrastive replies, mixed signals, implied rejection, and edge cases where surface keywords mislead. Before training anything, the next step should be measurement:

- keyword hit rate
- keyword false-positive rate
- LLM fallback rate
- LLM fallback accuracy
- invalid JSON rate
- invalid enum rate

If the keyword false-positive rate is low and the LLM fallback is accurate, document the two-tier design and skip fine-tuning. If the ambiguous bucket is large and recurring, then a small trained classifier or fine-tuned encoder may become worth it.

Until then, the strongest improvement is not a custom model. It is a cleaner boundary: deterministic rules for obvious replies, constrained labels for LLM fallback, and logging that shows when each tier is earning its place.

## What I would change in your repo

Your current two-tier architecture is directionally right. I would keep it, but make three changes.

**1. Constrain the intent field.**  
Use provider-supported structured output if available, or a local constrained decoding approach if your runtime supports it. If neither is available, keep `json.loads()` but add explicit enum repair only for safe cases, then fail closed to `unclear`.

**2. Log tier performance.**  
Every classification should record whether it came from keywords or LLM fallback. Over time, that gives you real evidence for the 80/20 claim instead of relying on intuition.

**3. Review broad keywords.**  
Some keywords are riskier than others. For example, `interested` can appear in a negative sentence like "I am not interested enough to continue." Your rejection rules currently run before positive rules, which helps, but tier logging will show whether any broad positive keywords are causing false positives.

The final answer to your gap is:

**The model outputs one of the seven labels because the prompt makes those label strings high-probability next-token continuations. Keyword rules handle most cases because most replies have explicit lexical intent. The LLM adds value only when intent depends on sentence structure. To make the classifier robust, constrain the label output instead of relying on prompt-only JSON.**

## Sources

1. Willard, B. T. and Louf, R. (2023). "Efficient Guided Generation for Large Language Models." arXiv:2307.09702. Used here for the constrained decoding mechanism: restrict generation with a grammar or finite-state machine so invalid outputs are not available at decode time.

2. Tam, Z. R. et al. (2024). "Let Me Speak Freely? A Study on the Impact of Format Restrictions on Performance of Large Language Models." arXiv:2408.02442. Used here for the tradeoff: strict output formats can help fixed-label outputs while constraining free-form reasoning can hurt more complex generation.
