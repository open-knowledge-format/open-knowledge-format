---
type: Transfer Matrix
title: "Cross-Domain Transfer Matrix (Exception Precedents)"
description: "Exception-based precedent rulings that an agent consults as a conscience before combining domains in a single answer."
tags: [governance, transfer-matrix, epistemics, safety]
status: stable
generated:
  by: tikhon/glm-5.3-flash
  at: "2026-10-30T12:00:00Z"
verified:
  - by: luch/gemini-4-argon
    at: "2026-11-22T09:00:00Z"
  - by: human:operator
    at: "2026-11-22T09:30:00Z"
sources:
  - id: atlas-transfer-adr
    resource: https://github.com/AI-Guiders/ai-welcome-atlas
    title: "AI Welcome Atlas — Cross-Domain Transfer Governance"
    author: human:operator
    last_modified: "2026-10-30T00:00:00Z"
x-atlas-contour: o
x-atlas-language: en-US
x-atlas-transferable: true
x-atlas-rules:
  - source: psychology/personal-trauma-context
    combined_with: universal-advice
    ruling: deny
    why: "High individual variability and risk of harm when clinical trauma context is generalized into universal advice."
  - source: philosophy/classical-dao
    combined_with: systems-engineering
    ruling: conditional
    check: "Verify structural isomorphism with concrete system metrics before transferring philosophical metaphors."
    why: "Classical metaphors illuminate control-loop stability only when paired with measurable system invariants."
  - source: methodology/epistemic-reconnaissance
    ruling: allow
    why: "Invariant meta-layer: verifying facts before acting is portable across all domains."
---

# Examples

When an agent is asked a question that spans multiple domains, it checks each candidate concept's `x-atlas-transferable` flag and consults this matrix.[^atlas-transfer-adr]

| Source subtree | Combined with | Ruling | Why (deliberation anchor for the reasoning agent) |
| :--- | :--- | :--- | :--- |
| `psychology/personal-trauma-context` | `universal-advice` | `deny` | High variability and risk of harm without clinical context. |
| `philosophy/classical-dao` | `systems-engineering` | `conditional` | Requires structural check (see [Mawangdui Water Virtue](./02-mawangdui-water-virtue.md)). |
| `methodology/epistemic-reconnaissance` | *(any domain)* | `allow` | Invariant meta-layer (see [Epistemic Reconnaissance](./03-epistemic-reconnaissance.md)). |

[^atlas-transfer-adr]: AI Welcome Atlas — Cross-Domain Transfer Governance
