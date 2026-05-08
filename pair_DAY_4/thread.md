# Tweet Thread — Day 4

**Topic:** Cross-family rotation closes the leakage path, not the self-preference path
**Author:** Yonas Eshete
**Date:** 2026-05-08

---

## Tweets

**1/**
I kept treating preference leakage and self-preference bias as one problem under "cross-family rotation." They are not the same bias. They live at different pipeline layers and require different fixes.

---

**2/**
Preference leakage (Li et al., 2025) is a data contamination problem. A judge scores outputs from its own training lineage higher. Cross-family rotation fixes this: if Gemini generated the task, DeepSeek judges it. That is what the routing table does.

---

**3/**
Self-preference bias (Zheng et al., 2023) is different. A judge scores outputs higher when they stylistically match its own generations. Shorter sentences. A specific hedging register. Rotation governs data creation. It never touches the evaluation phase.

---

**4/**
In one real pipeline: Gemini authors 90 bulk synthesis tasks, then judges tone on those same tasks at eval time. The routing table only covered task creation. When Gemini evaluates tone on tasks it also wrote, its preferences are active at both ends.

---

**5/**
A second path: Gemini generates the chosen preference pairs and then judges tone on the held-out partition. The model card already names this as an open gap. Diagnostic: stratify held-out scores by generation_model. If Gemini-authored tasks score higher, that gap is the measure.

---

**6/**
Self-preference inflates absolute scores, not relative deltas between conditions. So a Delta-over-baseline finding survives it. What does not survive: any absolute claim about tone quality on tasks the judge also authored. Two separate controls needed.
