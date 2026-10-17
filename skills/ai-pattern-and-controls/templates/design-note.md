# Design note: [feature]

**Date:** [YYYY-MM-DD] · **Inputs:** [use-case card, scan report] · **Status:** draft / agreed

## Summary

[Two sentences: the pattern chosen and the one principle that drives it.]

## Task decomposition

| Step | Type (language / decision / retrieval) | Owner (model / code) | Rule, if a decision |
|---|---|---|---|

## Pattern

- **Chosen:** rung [1–4], [name]
- **Rejected:** [each lower rung, and why it doesn't work] · [higher rungs, and why not needed]
- **Workflow pattern(s):** [chaining / routing / parallelisation / orchestrator-workers / evaluator-optimizer]
- **Decomposition tests:** [for each split: runs alone? checkable alone? what merging would lose]
- **Multi-agent:** [not used, because … / used for …, cost estimate …]

## Authority

| Action | Changes the world? | Decided by | Limits | Enforced where | Fail mode | Cap |
|---|---|---|---|---|---|---|

## What the model sees

- **Gets:** [ ]
- **Never gets:** [secrets, thresholds, rules]
- **Tools:** [list]
- **Untrusted input handling:** [fencing, marker neutralisation]

## Risk and autonomy

- Risk tier: [ ] because [ ]
- Autonomy tier: [ ] (as built: [ ])

## Target architecture

[Containers, new and kept. Then the changes in priority order:]

1. [Change] — [why]

## Four stages

| Stage | Design | Gap check |
|---|---|---|
| Input handling | | Validation? |
| Processing | | |
| Output delivery | | Checked in code before delivery? |
| Feedback | | Outcomes routed to evals, prompts or data? |

## ADRs

- ADR-[n]: [title] — [link or inline]

## How we'll know it worked

| Decision | Eval or test |
|---|---|
