# Model and prompt specification: [system]

**Date:** [YYYY-MM-DD] · **Inputs:** [design note, guardrail plan, eval suite] · **Prompt versions:** [ ]

## Calls

| Call | Job | Volume / day | Latency target | Cost of an error | Tier | Effort |
|---|---|---|---|---|---|---|

## Call: [name]

- **Tier and effort:** [ ] · **Why:** [start cheap / start capable; eval evidence]
- **Model version pinned:** [ ]
- **System prompt (outline):**
  1. Role and task
  2. Context and why it matters
  3. Rules / steps
  4. Output format
- **User turn:** [per-request data, in tags; which are untrusted]
- **Examples:** [none / 3–5, what they cover]
- **Output contract:** [schema; validated where]
- **Escape hatch:** [what the model outputs when it can't do the task]
- **Refusal and truncation handling:** [ ]
- **Never in context:** [secrets, limits, rules]

## Context management (long sessions)

[Trimming, clearing, compaction, state held in code, subagent briefs]

## Caching plan

| Prefix part | Size (tokens) | Breakpoint? | Lifetime | Reuse (requests per cache period) |
|---|---|---|---|---|

## Cost and latency model

| Call | Uncached input | Cache write | Cache read | Output | Calls per case | Cost per case | p95 latency |
|---|---|---|---|---|---|---|---|
| **Total** | | | | | | | |

Monthly cost at expected volume: [ ]

## Tuning log

| # | Lever (free win / routing / effort / accuracy) | Change | Cost effect | Latency effect | Eval result |
|---|---|---|---|---|---|
