# Sources - Day 2

## Question

Amare's question asks why `agent/llm/reply_agent.py` can usually classify a reply into one of seven intent labels, what is happening at the token level when the LLM does that, and why keyword pre-screening handles most cases before the LLM fallback runs.

## Source 1

**Title:** Efficient Guided Generation for Large Language Models  
**Authors:** Brandon T. Willard and Remi Louf  
**Year:** 2023  
**Type:** arXiv preprint  
**Link:** https://arxiv.org/abs/2307.09702  

**What it teaches:**  
This paper explains guided generation: constraining LLM output with regular expressions or context-free grammars by treating generation as transitions through valid states. The relevant mechanism for this explainer is constrained decoding, where invalid next-token continuations can be masked so the model can only produce outputs that satisfy the allowed structure.

**How it is used in the explainer:**  
It supports the section on replacing prompt-only JSON with a constrained label boundary. For Amare's classifier, the useful application is small: constrain the `intent` field to one of seven enum values instead of merely asking the model to choose one.

**What I am not claiming from it:**  
I am not claiming this paper was tested on Amare's repo or that it measured his exact intent classifier. I use it only for the decoding mechanism behind grammar/enum-constrained generation.

## Source 2

**Title:** Let Me Speak Freely? A Study on the Impact of Format Restrictions on Performance of Large Language Models  
**Authors:** Zhi Rui Tam, Cheng-Kuang Wu, Yi-Lin Tsai, Chieh-Yen Lin, Hung-yi Lee, and Yun-Nung Chen  
**Year:** 2024  
**Type:** arXiv preprint  
**Link:** https://arxiv.org/abs/2408.02442  

**What it teaches:**  
This paper studies how forcing LLMs into structured output formats can affect performance. The relevant point for this explainer is the tradeoff: format constraints can be useful for fixed-output fields, but may interfere with tasks that benefit from free-form reasoning.

**How it is used in the explainer:**  
It supports the recommendation to constrain the `intent` label while keeping fields like `reasoning` and `key_quote` freer. The label is a fixed enum; the explanation fields are natural-language outputs.

**What I am not claiming from it:**  
I am not claiming the paper proves constrained decoding always improves classification. I use it only to justify being selective about what gets constrained.

## Local artifact inspected

**File:** `signal-driven-sales-conversion-engine/agent/llm/reply_agent.py`  

**Why it matters:**  
This is the implementation Amare's question points to. The explainer uses it to ground the analysis in the actual classifier shape:

- `ReplyIntent` defines seven labels.
- `_CLASSIFY_SYSTEM` asks the LLM to return JSON with one intent.
- `classify_reply()` first checks keyword lists.
- The fallback LLM path parses the response with `json.loads()`.
- Invalid JSON or invalid enum values fall through to `UNCLEAR`.

**How it is used in the explainer:**  
The local code is the basis for the practical recommendation: keep keyword pre-screening for obvious lexical cases, constrain or validate the LLM fallback label, and log which tier handled each reply.
