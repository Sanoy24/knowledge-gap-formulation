# Question - Day 2

**Asker:** Yonas Eshete  
**Topic:** Agent and tool-use internals - the mechanics of premature execution bias and suppressed clarification questions  
**Date:** 2026-05-06

---

## The Question

While evaluating my Week 10 Conversion Engine against tau2-Bench retail-style tasks, I documented a severe failure mode in my probe library, especially probes P023, P024, and P025: **premature tool execution**.

When a user asked to "cancel my last order", the agent lacked the required `order_id`. Instead of asking "What is your order ID?", it fabricated an ID and immediately called `cancel_pending_order`.

I patched this with a heavy SCAP (Signal Confidence Aware Phrasing) postscript in `eval/run_heldout.py` to force the model to ask, not assert, when required evidence is missing. The patch improved behavior, but it feels like I am fighting the model with prompt text instead of designing a safer tool boundary.

**Why does a function-calling LLM prefer fabricating missing required arguments and executing a tool over asking the user a clarifying question?**

I want to understand whether this comes from tool-use training, benchmark pressure toward action, the presence of tool schemas in context, or the way the agent interface represents clarification. The answer should explain the mechanism well enough to decide whether my fix should be prompt-based, validation-based, or structural.

---

## Artifact Connection

Closing this gap changes how I improve my Week 10 Conversion Engine.

Right now, `eval/run_heldout.py` relies on a long `SCAP_POSTSCRIPT` prompt to reduce hallucinated tool arguments. If I understand the mechanism behind premature execution bias, I can replace or simplify that fragile prompt patch with a safer interface pattern, such as:

- an explicit `ask_clarifying_question(question_text)` tool
- required-argument validation before destructive tools execute
- a planner/check step that routes missing arguments to clarification instead of tool execution

This matters beyond this benchmark. In FDE work, agents often sit in front of real business tools: CRMs, calendars, billing systems, support workflows, and internal databases. A safe agent must know when it lacks required information and must ask before acting.
