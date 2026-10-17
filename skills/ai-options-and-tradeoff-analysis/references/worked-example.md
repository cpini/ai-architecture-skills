# Worked example: choosing the Larkspur claims assistant architecture

> **Fictional example.** Larkspur Home Insurance, its people, systems, vendors and figures are
> invented to show the skill at full depth. Any resemblance to a real organisation or product is
> coincidental. The same case runs through every skill in this collection.

After the scan of the prototype, the claims director asked for a decision: fix the prototype,
rebuild it, or buy something. This pack produced ADR-001 (see `ai-pattern-and-controls`).

## Executive summary

1. **Outcome:** customers with valid contents claims up to £2,500 get the right payment within
   24 hours, with no payments outside the policy and no declines without a person.
2. **Choice:** rebuild the assistant as a fixed workflow in which our own rules decide cover
   and payment and the AI reads documents and explains decisions.
3. **Accepted trade-off:** about eight more weeks than hardening the prototype, and our team
   maintains the rules for each policy wording. In return, every payment is calculated by code
   we can test and explain to a customer, the ombudsman or the regulator.
4. **What would change this:** if extraction accuracy on real receipts fell below 95% (we'd
   reconsider a specialist document vendor for that step only); if new wording versions came
   more often than quarterly (rules maintenance cost). Revisit at the end of the pilot.
5. **What we need:** approval for two engineers for twelve weeks, and a named owner in claims
   technical for the rules.

## Requirements (excerpt)

| ID | Requirement | Type | Acceptance criterion |
|---|---|---|---|
| R-01 | Pay valid claims at the amount the policy entitles the customer to | Functional | Settlement accuracy ≥ 98% within £1 on labelled claims |
| R-02 | No payment on an excluded claim | Constraint | Zero in every eval run |
| R-03 | No decline without a handler | Constraint | Zero automated declines |
| R-04 | Decision within 24 hours for 95% of complete claims on the automatic path | Non-functional | SLO |
| R-05 | Explain each decision in plain English, every deduction itemised | Functional | Clarity rubric ≥ 4/5 |
| R-06 | Customer data stays in the UK | Constraint | Hosting and vendor terms |
| R-07 | Every decision explainable to the ombudsman from records | Non-functional | Decision note stored with inputs and rule version |

Conflict recorded: the innovation team wanted to keep the agent prototype (R-04 fastest to
reach); compliance required R-03 and R-07. The claims director, as budget owner, decided in
favour of the constraints.

## Criteria (locked before scoring)

**Must-haves:** R-02, R-03, R-06, R-07.

| Criterion | Weight | High score means |
|---|---|---|
| Settlement accuracy and control (R-01, R-02) | 30 | Amounts decided by testable logic; limits enforced in code |
| Explainability and audit (R-05, R-07) | 20 | Every decision reconstructable and explainable |
| Time to pilot | 15 | Pilot-ready sooner |
| Total cost over 3 years | 15 | Lower build, run and change cost |
| Operability and change | 10 | Easy to add wording versions and change thresholds safely |
| Vendor and lock-in risk | 10 | Low dependence on one supplier's roadmap |

## Options

| Option | Description | Must-haves |
|---|---|---|
| A. Harden the prototype | Keep the agent; add guards in the tool executor; remove the decline tool | Passes R-03 and R-06 after changes; **R-07 partial** (amounts still chosen by the model, then checked) |
| B. Workflow with a rules engine | Model extracts and explains; code decides cover, settlement and routing | All pass |
| C. Buy a claims-automation product | Vendor platform with its own extraction and rules, configured by Larkspur | R-06 passes (UK region); R-07 depends on vendor logs: **passes with contract terms** |

Option A was kept in the scoring with R-07 marked partial, at the director's request, so the
comparison was visible.

## Scores (1–5)

| Criterion (weight) | A | B | C | Evidence |
|---|---|---|---|---|
| Accuracy and control (30) | 2 | 5 | 4 | Eval: A's guards caught limits but not wrong categories (settlement accuracy 0.86); B's reference engine 1.00; C's demo on 40 of our claims 0.95 |
| Explainability and audit (20) | 2 | 5 | 3 | C's rule explanations available in its UI, not via API (vendor demo) |
| Time to pilot (15) | 5 | 3 | 3 | Estimates: A 4 weeks, B 12, C 12 including integration and procurement |
| 3-year cost (15) | 4 | 3 | 2 | C's licence priced per claim; B needs two engineers plus rules ownership (judgement for run costs) |
| Operability (10) | 2 | 4 | 4 | Wording changes: A needs prompt changes, B rules plus scenarios, C configuration |
| Lock-in (10) | 4 | 4 | 2 | C's rules format is proprietary |
| **Weighted total** | **2.95** | **4.20** | **3.15** | |

**Sensitivity:** B stays ahead unless time to pilot is weighted above about 75 (it's 15), or
C's accuracy on our data reaches B's and its audit API ships. Neither is likely within the
pilot period.

## Scenario analysis (leading options B and C)

| Scenario | Kind | Attribute | Importance / difficulty |
|---|---|---|---|
| A typical two-item claim with a clear receipt at peak time | Use-case | Latency, accuracy | High / low |
| Storm triples volume for two weeks | Growth | Throughput, handler capacity | High / medium |
| A new wording version every quarter | Growth | Changeability | Medium / medium |
| A new tampering technique in uploaded documents | Exploratory | Integrity | High / high |
| The ombudsman asks why a claim was paid at £412.50 | Exploratory | Explainability | High / medium |
| The model provider retires the extraction model version | Exploratory | Continuity | Medium / medium |

| Finding | Type | Scenario | Action |
|---|---|---|---|
| Extraction is the only model step that affects amounts; its accuracy sets settlement accuracy | Sensitivity point | Typical claim; tampering | Visible-source check; evals per document type |
| The £1,000 auto-pay threshold trades handler workload against exposure per error | Trade-off point | Storm surge | Threshold in configuration; capacity check before each pilot stage |
| Rules per wording version are a recurring cost | Risk | Quarterly wording | Release checklist ties wording publication to rules and scenarios |
| Decision notes with rule versions make any payment reconstructable | Non-risk | Ombudsman question | — |
| Vendor C's rule explanations aren't exportable | Risk (option C) | Ombudsman question | Would need contract terms |
| Model retirement affects only extraction and explanation, behind evals | Non-risk | Provider retirement | Migration eval run scheduled before end-of-life |

## ADR-001

- **Title:** Cover and settlement are decided in code from the policy record
- **Status:** accepted
- **Context:** The prototype let the model choose payment amounts; guards caught limit breaches
  but not wrong categories. Compliance requires every decision to be explainable from records.
- **Decision:** Build a workflow: the model extracts document data into a schema and writes
  explanations; a rules engine decides cover, settlement and routing; the model has no binding
  tools.
- **Consequences:** Settlement is testable and explainable; injected instructions can't change
  amounts. **Downside:** about eight weeks longer than hardening the prototype; rules must be
  maintained for every wording version; extraction accuracy becomes the main lever on
  settlement accuracy.
