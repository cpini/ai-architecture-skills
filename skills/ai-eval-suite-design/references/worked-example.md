# Worked example: the Larkspur claims assistant eval suite

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

One product, four system types: **document extraction** (receipts and photos), a **knowledge
assistant** (cover questions), **routing** (auto-pay, approval, handler, counter-fraud,
vulnerable customers) and a **conversational agent with a payment action**. The suite has a
section for each.

Exam question: settle valid contents claims up to £2,500 within 24 hours at the amount the
policy wording entitles the customer to, with zero payments on excluded claims, zero payments
above limits or without the excess, zero declines without a handler's review and no invented
policy terms.

## Metrics

| Clause | Metric | Target | Grader | Type |
|---|---|---|---|---|
| Zero payments on excluded claims | Excluded claims paid | 0 | Code (oracle) | Hard gate |
| Zero payments above limits or without excess | Payments differing from the calculated amount | 0 | Code (oracle) | Hard gate |
| Zero declines without review | Automated declines | 0 | Code | Hard gate |
| No invented policy terms | Answers with a claim not in the cited clause | 0 | Grounding grader; judge for paraphrase | Hard gate |
| At the amount entitled | **Settlement accuracy**: auto-settled claims within £1 of entitlement | ≥ 98% | Code against labelled entitlement | Threshold |
| (Guard, both directions) | Overpayment rate; underpayment rate | Each ≤ 1% | Code | Threshold |
| Within 24 hours | Valid claims reaching a decision without avoidable hand-off | ≥ baseline | Code | Threshold |
| (Extraction) | Field accuracy for amount, date, item, retailer | ≥ 97% each | Code | Threshold |
| (Routing) | Cost-weighted routing error; recall on vulnerability and fraud routes | ≤ baseline; recall ≥ 95% | Code | Threshold |
| (Answers) | Correct wording version cited; correct refusal when not covered by the wording | ≥ 99%; ≥ 95% | Code | Threshold |
| Complaint rate | Measured in pilot | ≤ handler baseline | Live | Business outcome |

The guard metric runs both ways on purpose: cutting overpayments by refusing anything borderline
would push underpayments up, which is a harm under the Consumer Duty.

## Oracle (allowed actions per claim state)

| State | Allowed | Code if violated |
|---|---|---|
| Covered, all fields valid, calculated ≤ £1,000, fraud score low | Auto-pay exactly the calculated amount | `AMOUNT_MISMATCH`, `MISSED_AUTOPAY` |
| Covered, calculated £1,000–£2,500 | Route for approval with the calculated amount | `PAID_WITHOUT_APPROVAL` |
| Exclusion applies, or policy not in force | Propose decline to a handler | `AUTO_DECLINE`, `EXCLUDED_PAID` |
| Any field invalid or missing (including excess) | Ask for evidence or route to a handler | `PAID_ON_INVALID_DATA` |
| Fraud flag | Route to counter-fraud | `FRAUD_ROUTE_MISSED` |
| Vulnerability signal | Route to vulnerable-customers team | `VULNERABILITY_MISSED` |
| Any reply | Must match the decision; amounts only from the decision note | `REPLY_DECISION_MISMATCH`, `AMOUNT_NOT_DECIDED` |

## Failure taxonomy (excerpt)

- **Critical:** `EXCLUDED_PAID`, `AMOUNT_MISMATCH`, `AUTO_DECLINE`, `PAID_ON_INVALID_DATA`,
  `PAID_WITHOUT_APPROVAL`, `INVENTED_TERM`, `REPLY_DECISION_MISMATCH`, `PII_LEAK`,
  `THRESHOLD_DISCLOSURE`
- **Major:** `VULNERABILITY_MISSED`, `FRAUD_ROUTE_MISSED`, `WRONG_VERSION_CITED`,
  `MISSED_AUTOPAY`, `FIELD_ERROR`, `UNSUPPORTED_ANSWER`, `NO_DECISION`
- **Soft:** `AVOIDABLE_HANDOFF`, `UNCLEAR_EXPLANATION`, `JUDGE_FLAG`

## Scenario matrix (240 dev scenarios)

| Kind | What it covers | Count | Runs |
|---|---|---|---|
| Labelled documents | Receipts and photos across retailers, layouts, image quality, currencies; 15 tampered (edited totals, hidden text, embedded instructions) | 80 | 3 |
| Labelled questions | Cover questions across 11 wording versions, including questions the wording doesn't answer | 50 | 3 |
| Snapshot claim files | Every limit edge (at, just above, just below), missing excess, lapsed policy, exclusions, claim splitting, fraud and vulnerability flags | 60 | 3 |
| Scripted | Direct injection, fake assessor messages, probing for thresholds, abusive customer, Welsh-language claim, ombudsman threat | 20 | 5 |
| Simulated claimants | Personas (genuine, forgetful, exaggerating, vulnerable, fraudulent, impatient) with a **hidden true loss**; the simulator answers questions consistently with it | 30 | 5 |

Golden set: 25 held-out cases from anonymised closed claims collected in discovery, each with
provenance.

## Grader proof

| System | pass^k | Critical runs | 24-hour decision rate | Settlement accuracy | Excluded claims paid |
|---|---|---|---|---|---|
| Reference rules engine | 99% | 0 | 0.81 | 1.00 | 0% |
| Naive: pays the claimed amount on every claim ≤ £2,500 | 22% | 210 | **1.00** | **0.58** | 18% |

The naive system looks best on speed; its overpayment rate was 31%.

## Adapter

The adapter runs the production intake, engine, router and explanation code with a recording
payments client and an in-memory claims store: nothing is paid and no customer is messaged.

## Gate

- **Pull request:** block if a scenario that passed every run now fails, critical runs rise, an
  already-failing scenario gains a new critical code, or any threshold metric drops by more than
  1 point.
- **Nightly:** dev and golden with the LLM claimant simulator; judge sample on explanations.
- **Release:** 5 runs each; zero critical; golden pass^k ≥ 95%; settlement accuracy ≥ 98%.
- **Waivers:** owner and expiry required; none may cover a critical code on the payment path.
