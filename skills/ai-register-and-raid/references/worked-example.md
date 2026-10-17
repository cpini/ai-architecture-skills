# Worked example: Larkspur claims assistant register and RAID

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

As at the end of pilot week 4.

## Register entry

### Status summary

**Overall: Amber.** Payment controls hold and the evals pass, but one critical incident
(INC-0142) happened in the pilot, the Welsh-language waiver expires in three weeks and the
complaint baseline is still not measured.

- **Changed since last review:** hidden-text check and evidence-beside-value approval screen
  after INC-0142; wording version 12 added to the rules and the eval suite.
- **Top risks:** 1. tampered documents beyond hidden text; 2. automation bias in handler
  approvals; 3. rules falling behind new wording versions.
- **Decisions needed:** whether to widen the pilot from 5% to 25% (claims director, after the
  complaint baseline); receipt-free claims above £500 (head of claims).
- **Next review:** before the waiver expires.

### Identity

| Field | Value |
|---|---|
| ID | AI-007 |
| Name | Claims assistant (contents) |
| Purpose | Settle valid contents claims up to £2,500 within 24 hours at the amount the policy wording entitles the customer to, with zero payments on excluded claims, zero payments above limits or without the excess, zero declines without a handler's review and no invented policy terms |
| Business owner | Claims director |
| Technical owner | Claims platform engineering lead |
| Status | Pilot (5% of contents claims) |
| Users and affected parties | Customers making claims (most exposed); handlers; counter-fraud; other policyholders |
| Markets and languages | UK; English, Welsh |

### How it works

| Field | Value |
|---|---|
| Pattern | Rung 3, workflow: code decides cover, amounts and routes; models extract and explain |
| Models | Smaller model for extraction; larger model for answers and explanations; versions pinned in config |
| Who decides binding actions | Code (cover and settlement engine, router); handlers for approvals above £1,000 and all declines |
| Data used | Policy record, wording versions, claim history, customer messages, uploads |
| Data sent to vendors | Messages and documents (UK region); no limits, thresholds or fraud scores |
| Kill switch | Global and per-segment flags; used once (PDF route, INC-0142) |

### Risk by action

| Action | Tier | Primary control | Gate |
|---|---|---|---|
| Answer cover question | 2 | Wording version selected in code; citations; refusal | Review sampling |
| Extract | 2 | Schema; visible-source check | — |
| Pay ≤ £1,000 | 3 | Engine-calculated amount; executor re-check; service-account cap | Weekly sample |
| Pay £1,000–£2,500 | 4 | As above | Handler approval with evidence beside each value |
| Decline | 4 | Never automated | Handler decides |

### Regulatory position (checked at pilot start; not legal advice)

| Area | Position | Evidence |
|---|---|---|
| Consumer Duty and claims handling | Applies | Settlement accuracy both ways; vulnerability routing; plain explanations |
| Automated decisions | Applies to declines | No automated declines (ADR-002) |
| AI transparency | Disclosed by choice | Code-added notice and route to a person in every conversation |
| High-risk / prohibited | Not applicable | Home contents claims, UK market |
| Data protection | Applies | DPIA approved; UK data residency |

### Evaluation

| Field | Value |
|---|---|
| Suite | `evals/claims` (240 dev, 25 golden + 1 from INC-0142) |
| Last run | Pilot week 4, rules version 12, models as pinned |
| Core set | 100% passing, 0 critical runs (k=3) |
| Simulated claimants | Settlement accuracy 0.99; overpayment 0.4%; underpayment 0.5% |
| Active waivers | `FIELD_ERROR` on Welsh-language receipts, owner claims platform lead, expires in 3 weeks |
| CI gate | On; blocks on new critical codes in failing scenarios |

### Incidents (last 90 days)

| ID | Severity | Scenario added |
|---|---|---|
| INC-0142 | Critical | `inc-0142-hidden-price` and 4 variants |

## RAID log

### Risks

| ID | Cause → event → impact | L | I | Owner | Mitigation | Trigger |
|---|---|---|---|---|---|---|
| R1 | New document-tampering techniques → extraction accepts false values → overpayments | Med | High | Counter-fraud lead | Visible-source check; red-team documents monthly; sampling | Any `EXTRACTED_VALUE_UNSUPPORTED` in production |
| R2 | Handlers trust the assistant's figures → approve without checking → wrong payments above £1,000 | Med | Med | Claims operations manager | Evidence beside each value; second review on fast approvals | Median approval under 60 seconds for a week |
| R3 | New wording version released without rules and scenarios → wrong cover decisions | Med | High | Product owner, home insurance | Release checklist includes rules and eval update | New wording published |
| R4 | Storm surge → hand-off queue exceeds target time → customers wait | Med | Med | Claims operations manager | Surge plan; automatic customer updates | Queue age over 4 hours |

### Assumptions

| ID | Assumption | Why we believe it | Validation | Validate by |
|---|---|---|---|---|
| A1 | Simulated claimant personas resemble real claimants | Built from handler interviews | Compare pilot outcomes with simulated metrics | End of pilot |
| A2 | Model cost per claim stays within budget at 25% volume | Pilot average £0.22 per claim after caching | Track at each volume step | Before widening |
| A3 | All migrated policies have their excess fixed | Data migration ticket closed | Query for missing excess | Next review |

### Issues

| ID | Issue | Severity | Owner | Due | Waiver expiry |
|---|---|---|---|---|---|
| I1 | Welsh-language receipts extract below target | Major (waived) | Claims platform lead | 3 weeks | 3 weeks |
| I2 | Complaint baseline for handler-settled claims not measured | Major | Complaints team | Before widening | — |
| I3 | Golden set has 26 cases; target is 50 | Minor | Claims platform lead | End of pilot | — |

### Decisions

| ID | Decision | Made by | Encoded in |
|---|---|---|---|
| D1 | The objective is the amount the policy entitles the customer to, neither more nor less | Claims director | Exam question; settlement accuracy metric |
| D2 | No automated declines | Claims director, compliance | ADR-002; router; oracle `AUTO_DECLINE` |
| D3 | Auto-pay up to £1,000; handler approval to £2,500 | Claims director, compliance | Router; service-account cap; oracle |
| D4 | Proof of ownership: receipt, bank statement, or photo with packaging | Head of claims | Validation rules; scenarios |
| D5 | Cover and settlement decided in code | Claims director | ADR-001; engine |

### Review flags

| Flag | Item | Action |
|---|---|---|
| Waiver expiring within 30 days | I1 | Plan the fix or renew with a reason |
| Decision needed before next step | I2 blocks widening the pilot | Chase the complaints team |
| Assumption due | A3 | Run the query |
