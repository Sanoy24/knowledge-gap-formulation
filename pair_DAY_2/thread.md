# Tweet thread - Day 2

## Thread

1/  
I looked at a small but important agent bug today:

An LLM intent classifier returns one of 7 labels from a prompt.

It looks like classification.

But under the hood, it is still next-token generation.

2/  
The classifier lives in `reply_agent.py`.

It asks the model to return JSON like:

`{"intent": "rejection"}`

The prompt lists 7 allowed labels.

That list makes valid labels likely.

It does not make them guaranteed.

3/  
The mechanism:

After the model emits:

`{"intent": "`

it scores possible next tokens.

Labels like `rejection` become likely because the prompt and reply context point toward them.

That is not a classifier head. It is generation under label pressure.

4/  
Important detail:

Some labels may be multiple tokens.

So the model may not choose `positive_interest` in one step.

It may generate the label piece by piece.

That matters because invalid continuations can still happen unless the decoder or validator blocks them.

5/  
Prompt-only JSON has a weak boundary.

This is valid JSON:

`{"intent": "interested"}`

But it is not a valid enum if the allowed label is:

`positive_interest`

So JSON validity and task validity are different things.

6/  
The repo also had keyword pre-screening before the LLM.

That is not a hack.

It is the right tool for obvious replies:

`not interested` -> rejection  
`pricing` -> capacity_question  
`tell me more` -> positive_interest

7/  
The useful split is:

Lexical intent: the answer is visible in a phrase.

Compositional intent: the answer depends on sentence structure.

Keywords handle lexical intent.

The LLM earns its cost on compositional intent.

8/  
Example:

`Interesting, but we just signed with another vendor.`

A keyword rule may over-focus on `interesting`.

The real signal comes after `but`.

That is where the LLM can help.

It can reason over contrast, qualification, and mixed signals.

9/  
I checked the actual repo with a small local script.

It extracted:

- 7 valid intent labels
- keyword list sizes
- sample keyword matches
- JSON outputs that parse but fail enum validation

No fake LLM trace. Just the real code boundary.

10/  
One concrete finding:

`can you send a link?`

falls to the LLM fallback.

Why?

The keyword list has:

`send me a link`

not:

`send a link`

Tiny phrasing differences matter in a substring gate.

11/  
The fix is not just "write a better prompt."

Better boundary:

1. Keep keyword rules for obvious replies.
2. Constrain or validate the intent enum.
3. Let `reasoning` and `key_quote` stay free-form.
4. Log which tier handled each reply.

12/  
This also explains why the classifier does not obviously need fine-tuning yet.

If keywords handle the easy 80%, do not train a model to relearn those.

Measure the ambiguous fallback bucket first.

Train only if that bucket is large and recurring.

13/  
My takeaway:

Prompt instructions are useful.

But production boundaries should carry the guarantee.

For this classifier, the guarantee should be:

the `intent` field is one of the allowed enum values.

Not "the model was asked nicely."
