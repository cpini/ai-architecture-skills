# Current-state scan: [system / feature]

**Repo / commit:** [repo@sha] · **Date:** [YYYY-MM-DD] · **Scope:** [files and services read]

## Verdict

[One paragraph: can this go to (or stay in) production? What must change first? Name the risk
tier you'd assign and why.]

**Top findings**

1. **[Finding]** ([severity]): [one sentence with consequence]. `path:line`
2. …
3. …
4. …
5. …

## As built

[How a request flows: trigger → context assembly → model call(s) → tools → writes → logging.
Model and ID. Pattern on the escalation ladder: prompt / augmented model / workflow / agent.]

**What works well already:** [strengths to keep]

## Authority map

| Action | Changes the world? | Who decides | Limits | Where enforced | Fails open or closed |
|---|---|---|---|---|---|
| [e.g. approve_payment] | Yes: pays money | Model | Single-item limit | Prompt only | n/a (no check) |

## Findings

| # | Severity | Finding | Evidence | How known | Consequence | Confirming eval or test |
|---|---|---|---|---|---|---|
| 1 | Critical | | `path:line` "quote" | Read in code | | |

## Docs versus code

| Docs say | Code does | Evidence |
|---|---|---|

## Open questions

| Question | Who can answer | Changes which finding |
|---|---|---|

## Recommended next steps

1. [Evals to run first, tied to findings]
2. [Changes to make first, tied to findings]
