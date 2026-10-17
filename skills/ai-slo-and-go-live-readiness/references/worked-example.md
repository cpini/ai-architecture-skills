# Worked example: Larkspur claims assistant readiness

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

The claims assistant ran on 5% of contents claims for six weeks (including incident INC-0142;
see `ai-incident-to-scenario`). This pack supported the decision to widen it to 25%.

## Indicators, objectives and agreements

The claims operations team agreed an internal SLA with the digital team: the intake service is
available 99.5% of the time, and customers whose claims take the automatic path get a decision
within 24 hours.

| Journey | SLI (good event) | SLO | SLA | Window |
|---|---|---|---|---|
| Submit a claim | Upload accepted, extraction returned a schema-valid result or a deliberate "unreadable", within 60 s | 99.9% | 99.5% | 30 days |
| Get an explanation | Message delivered, validated against the decision note, not a refusal or fallback | 99.9% | — | 30 days |
| Explanation latency | Time to first token | p95 < 4 s | — | 7 days |
| Automatic decision | Complete claims on the automatic path decided within 24 h | 99% | 98% | 30 days |
| Settlement quality | Weekly sample of 200 auto-settled claims graded against entitlement: within £1 | ≥ 98%, zero critical | — | Weekly |

Refusals count as failures on "Get an explanation" even though the fallback message is
correct behaviour: they mean a customer got a generic message, and the trend matters.

## Error budget

- "Submit a claim": 0.1% of 30 days ≈ 43 minutes equivalent, or about 1 failed submission in
  1,000.
- Policy: if spent, only P0 and security fixes ship until it recovers; prompt and threshold
  changes wait.
- Any incident using more than 20% of a budget gets a postmortem with at least one P0 action.

## Alerts

| Alert | Condition | Action |
|---|---|---|
| Page | Burn rate > 14.4 over 1 h (confirmed over 5 min) | On-call engineer; check provider status, then kill switch to handler queue if not recovering |
| Ticket | Burn rate > 6 over 6 h (confirmed over 30 min) | Next working day |
| Review | Burn rate > 1 over 3 days | Weekly review |
| Quality | Weekly sample below 98%, or any critical code in production logs | Claims platform lead and claims director; kill switch for the affected segment if critical |

All four were test-fired in the staging environment before the pilot.

## Production gate for 25%

| Exit criterion | Evidence | Met? |
|---|---|---|
| Release evals pass on exact versions | Golden 26/26 pass^5; dev 100%; zero critical (rules v12, prompts v14) | Yes |
| Six weeks at 5% with quality SLO met every week | Weekly samples 98.5–99.5% | Yes |
| Incident INC-0142 closed with scenario and fix | Incident note; scenario passing | Yes |
| Complaint rate no higher than handler-settled claims | 0.9% vs 1.1% baseline (baseline measured in week 5) | Yes |
| Rollback exercised | Kill switch used for real on the PDF route during INC-0142; recovery 4 minutes. Prompt rollback rehearsed in staging | Yes |
| Runbook walked through | On-call team, week 4 | Yes |
| Handler capacity for approvals at 25% | Forecast 140 approvals a day; capacity confirmed by claims operations | Yes |
| Welsh-language extraction waiver | Fix shipped week 6; field accuracy 97.4% | Yes |

**Decision:** widen to 25% for at least four weeks, then review against the same criteria plus a
storm-period check.

## Pilot stages

| Stage | Scope | Minimum duration | Exit criteria | Comparison |
|---|---|---|---|---|
| 1 | 5% of contents claims, two regional teams as champions | 6 weeks | As above | Handler-settled claims in the same weeks |
| 2 | 25% | 4 weeks, including one surge period if possible | Same, plus approval queue age under 4 hours at peak | Same |
| 3 | 100% of eligible claims | — | Same at stage 2 scale; cost per claim within budget | Monthly sample of handler-settled claims kept for comparison |

## Hand-off pack

- **Decision log**: ADR-001 to ADR-005 and decisions D1–D5 from the register, each with where
  it's encoded.
- **Implementation guidance**: how to change thresholds (configuration plus eval run plus
  claims director approval), how to add a wording version (rules plus scenarios before
  release).
- **Runbook**: each alert, the diagnosis tree (provider error → our infrastructure → refusals →
  quality), the kill switch per segment, prompt and model rollback, contacts.
- **Acceptance**: the claims platform operations team signed off after a game day in which a
  simulated provider outage and a simulated bad prompt release were both handled within
  15 minutes.

## Drift versus regression, in practice

During stage 2 a storm brought a surge of water-damage claims. Extraction "unreadable" rates
tripled on photos of soaked receipts, and the automatic-decision SLI dipped to 97.6% for three
days. No change had shipped: **drift**. The team didn't roll back; they added 40 storm-damaged
receipt images to the labelled set, confirmed the router was correctly sending them to
handlers, and accepted the dip under the error budget. A month earlier, a prompt change that
shortened explanations dropped the clarity rubric score the same day it shipped: **regression**,
rolled back within the hour, then fixed through the eval loop.

## Migrations

The extraction model version has a published end-of-life date. The migration eval run is
scheduled eight weeks before it, including re-measuring cost per claim in case the new version
counts tokens differently.
