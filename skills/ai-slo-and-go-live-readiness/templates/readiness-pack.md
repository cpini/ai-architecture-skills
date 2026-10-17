# Readiness pack: [system]

**Date:** [YYYY-MM-DD] · **Stage:** [pilot / widen to n% / general availability] · **Decision owner:** [name]

## Indicators, objectives and agreements

| Journey | SLI (what counts as good) | SLO | SLA (if any) | Window |
|---|---|---|---|---|

Failure kinds tracked separately: infrastructure errors · refusals · silent wrong answers

## Error budget

- Budget: [1 − SLO] over [window] = [minutes or requests]
- Policy when spent: [freeze rules]
- Postmortem trigger: [share of budget per incident]

## Alerts

| Alert | Condition (burn rate / window) | Confirmation window | Action | Test-fired on |
|---|---|---|---|---|
| Page | > 14.4 over 1 h | 5 min | | |
| Ticket | > 6 over 6 h | 30 min | | |
| Review | > 1 over 3 days | 6 h | | |
| Quality | [sampled grade below threshold / any critical code] | | | |

## Production gate

| Exit criterion | Evidence | Met? |
|---|---|---|
| Release evals pass on the exact versions | [run ID] | |
| Monitoring and SLO dashboards live | [link] | |
| Alerts test-fired | [date] | |
| Rollback exercised (kill switch, previous versions) | [date, time to recover] | |
| Runbook walked through by on-call | [date] | |
| Register complete; RAID items owned | [link] | |
| Compliance sign-offs | [DPIA, disclosure] | |

## Pilot stages

| Stage | Scope | Minimum duration | Exit criteria | Comparison group |
|---|---|---|---|---|

## Hand-off pack

- Decision log: [link]
- Implementation guidance: [link]
- Runbook: [link]
- Acceptance criteria and sign-off by receiving team: [ ]

## Drift and regression

- Regression procedure: [roll back, then fix through the eval loop]
- Drift detection: [live sampling and re-scoring; input distribution checks]
- Drift procedure: [collect cases, add to evals, adapt]

## Retirement and migrations

| Item | End-of-life date | Migration plan |
|---|---|---|
