# Question — Day 2

**Asker:** Yonas Eshete
**Topic:** Agent and tool-use internals — *The mechanics of "Premature Execution Bias" and why models suppress clarification questions.*
**Date:** 2026-05-06

---

## The Question

While evaluating my Week 10 Conversion Engine against the τ²-Bench retail tasks, I documented a severe failure mode in my probe library (Probes P023, P024, and P025): **premature tool execution**. 

When a prospect asked "cancel my last order", the agent lacked the required `order_id`. Instead of simply emitting text to ask "What is your order ID?", the agent fabricated an ID just so it could immediately fire the `cancel_pending_order` tool. I had to build a heavy SCAP (Signal Confidence Aware Phrasing) postscript injected into `eval/run_heldout.py` just to force the model to "Ask-not-assert."

**What is happening at the token level during function-calling that makes generating a conversational clarification question so mathematically unlikely compared to forcing a tool call?**

Is this purely an artifact of how tool-use is rewarded during RLHF — where "stalling" is heavily penalized and executing a tool is treated as task success? When the attention heads process the structured JSON tool schemas in the system prompt, does the probability mass shift so heavily toward the `<tool_call>` control token that the model's normal conversational reasoning is suppressed? 

## Artifact Connection

Knowing this mechanism would let me completely refactor the scaffolding in my agent. Specifically, it would allow me to delete the 20-line `SCAP_POSTSCRIPT` string from `eval/run_heldout.py` (which currently tries to "fight" the model using prompt engineering) and instead inject an `ask_clarifying_question(question_text)` tool directly into the agent's context. 

If I can understand the exact token probabilities that drive "Premature Execution Bias," I can confidently replace fragile prompt instructions with a structural tool interface, guaranteeing a higher pass rate on the τ²-Bench without hallucinating parameters. More broadly, every FDE engagement involving an agent requires deciding how to constrain destructive tools — knowing how to "absorb" rather than "fight" the model's tool bias is a universally applicable pattern.
