# Tweet thread - Day 2

## Thread

1/  
I looked at a real intent classifier in a sales agent today. It asks the LLM to return JSON with one of 7 labels. It looks like classification. Under the hood, it is next-token generation. The prompt makes valid labels likely. It does not make them guaranteed.

2/  
After the model emits `{"intent": "` it scores possible next tokens. Labels like `rejection` get high probability because the prompt and reply context point toward them. But the model can still produce `"interested"` instead of `"positive_interest"`. Valid JSON, invalid enum. That is the gap.

3/  
The repo also had keyword pre-screening before the LLM. That is not a hack. It is the right tool for lexical intent: replies where the answer is visible in a phrase like "not interested" or "send me pricing." The LLM earns its cost only on compositional intent — contrast, qualification, mixed signals.

4/  
I ran a local script against the actual repo. It extracted 7 valid labels, keyword list sizes, and sample matches. Concrete finding: `"can you send a link?"` falls to the LLM fallback because the keyword list has `"send me a link"`, not `"send a link"`. Tiny phrasing gaps matter in a substring gate.

5/  
The fix is not "write a better prompt." Constrain or validate the intent enum at the output boundary. Keep keyword rules for obvious replies. Let `reasoning` and `key_quote` stay free-form. Log which tier handled each reply. Measure the ambiguous bucket before fine-tuning anything.

6/  
Prompt instructions are useful. But production boundaries should carry the guarantee. For this classifier: the `intent` field must be one of the allowed enum values. Not "the model was asked nicely." Full explainer with worked examples and sources here: [Link]
