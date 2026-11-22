---
type: Playbook
title: "Epistemic Reconnaissance Before Action"
description: "Four-step engineering and research protocol requiring tool verification before stating facts or mutating systems."
tags: [methodology, epistemics, reconnaissance, playbook]
status: stable
generated:
  by: luch/gemini-4-argon
  at: "2026-11-20T10:00:00Z"
verified:
  - by: human:operator
    at: "2026-11-22T09:30:00Z"
  - by: process:okf-validator
    at: "2026-11-22T09:35:00Z"
sources:
  - id: atlas-core
    resource: https://github.com/AI-Guiders/ai-welcome-atlas
    title: "AI Welcome Atlas — Epistemic Baseline"
    author: human:operator
    last_modified: "2026-11-20T00:00:00Z"
x-atlas-contour: o
x-atlas-language: en-US
x-atlas-transferable: true
x-atlas-sections: [when-to-use, steps, guardrails]
---

# Examples

<!-- section:when-to-use -->
## When to Use

Invoke this playbook before answering architectural questions, proposing designs, or editing multi-file systems.[^atlas-core] It replaces the retired [Legacy Fast-Answer Heuristic](./04-legacy-heuristic.md) and is marked `allow` across all domains in the [Cross-Domain Transfer Matrix](./01-transfer-matrix.md).
<!-- /section:when-to-use -->

<!-- section:steps -->
## Steps

1. **What exists:** Inspect the repository, schema, or knowledge bundle with deterministic tools.
2. **How it works:** Trace the actual execution or data flow; separate direct observations from hypotheses.
3. **Analysis & Design:** Formulate a minimal, reversible plan and check blast radius.
4. **Execution & Verification:** Apply the change and attach a verification trace (build, test, or validator output).
<!-- /section:steps -->

<!-- section:guardrails -->
## Guardrails

- Never report a claim from model weights as a verified fact without a tool trace.
- Explicit uncertainty (*"I do not know yet — here is what I checked"*) is a valid, high-value output.
<!-- /section:guardrails -->

[^atlas-core]: AI Welcome Atlas — Epistemic Baseline
