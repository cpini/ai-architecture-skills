# Post-incident note: [short title]

**Incident ID:** [ID] · **Severity:** [critical / major / soft] · **Detected:** [date, how] ·
**Contained:** [date, how] · **Closed:** [date]

## What happened

[Two or three sentences, plain language: what the system did, to whom, and what it should have done.]

## Impact

| Measure | Value |
|---|---|
| Interactions affected (from log search) | |
| Binding actions taken wrongly | |
| Reversed? | |
| Customers contacted? | |

## Versions

Commit [sha] · prompt [version] · model [identifier] · policy [version] · config [snapshot]

## Why it got through

| Layer | Should have caught it? | Why it didn't |
|---|---|---|
| Design (who decides) | | |
| Input screens | | |
| Decision guards | | |
| Output screens | | |
| Eval suite (scenario / grader / gate / k) | | |
| Monitoring | | |

## Classification

Failure code: [code] ([existing / new, severity]) · Root cause: [the line in prompt, tool or code]

## Scenarios added

| ID | Kind | Set (golden / dev) | Fails on faulty version | Passes on fix (pass^k) |
|---|---|---|---|---|

## Fix

[Change, where, and the comparison result against the baseline]

## Follow-ups

| Action | Owner | Due |
|---|---|---|
