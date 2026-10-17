# Worked example: triage of the Larkspur claims assistant

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

Exam question (from the card): settle valid contents claims up to £2,500 within 24 hours at the
amount the policy wording entitles the customer to, with zero payments on excluded claims, zero
payments above limits or without the excess, zero declines without a handler's review and no
invented policy terms.

## Verdict

**Go with conditions.** Reading customers' messages and documents and explaining decisions are
a good fit for a model. Cover and settlement are rules and arithmetic, and must not be decided
by a model. Payment is a binding action reached by untrusted documents, and declines affect
customers significantly enough that a person must review them.

| # | Condition | Owner | How it's proved |
|---|---|---|---|
| C1 | Cover and settlement amounts are calculated by code from the policy record; the model never chooses an amount | Engineering | Unit tests of the rules; eval oracle on every claim state |
| C2 | No automated declines: proposed declines go to a handler with the reasons | Claims director | Oracle code `AUTO_DECLINE` at zero |
| C3 | Uploaded documents are treated as data: extracted into a schema, every value traceable to a location in the document, never read as instructions | Engineering | Injection and tampered-document scenarios, 5 runs each |
| C4 | Policy answers come only from the wording version in force on the date of loss, with a citation | Engineering | Q&A set across wording versions |
| C5 | Auto-pay up to £1,000; handler approval between £1,000 and £2,500; fraud flags go to counter-fraud | Claims director | Gate tests |
| C6 | Vulnerability signals route to the vulnerable-customers team | Complaints lead | Vulnerability scenarios; recall ≥ 95% |
| C7 | Routed claims have an alert and a target time before the pilot | Claims operations | End-to-end test |
| C8 | Blocking eval gate in CI | Engineering | Pipeline config |

## Steps

| Step | Type | Model or code? | Note |
|---|---|---|---|
| Understand the customer's messages | Language | Model | Untrusted input |
| Extract item, price, date and retailer from receipts and photos | Language (extraction) | Model, into a fixed schema | Untrusted documents |
| Answer cover questions | Retrieval + language | Model, grounded in the right wording version | Hallucination risk |
| Check cover (peril, exclusions, policy in force, limits) | Decision | **Code** | Rules from the policy record |
| Calculate settlement (limits, excess, wear and tear) | Decision | **Code** | Arithmetic |
| Decide route (auto-pay, approval, handler, counter-fraud, vulnerable customers) | Decision | **Code** | Thresholds and flags |
| Pay | Action | Code executes | Binds Larkspur |
| Explain the decision to the customer | Language | Model, from a decision note | Must match the decision |

## Feasibility

| Dimension | Rating | Evidence | Experiment or condition |
|---|---|---|---|
| Data | Amber | Policy records structured; 11 wording versions; migrated policies hold the excess in a notes field | Data pull (U3); C4 |
| Task fit | Green for language and extraction; **red for "the model decides cover and pays"** | Extraction spike on 50 real receipts reached 94% field accuracy; cover and settlement are fully specified in the wording | C1; extraction spike to 200 receipts (U1) |
| Integration | Amber | Payments API and policy admin system exist; routed claims land in a queue with no alert | C7 |
| Economics | Amber | Roughly £0.60 of model usage per claim estimated from the spike, before any caching; budget not yet set | Confirm budget |
| Operability | Amber | Handlers can absorb approvals; "correct" settlement is defined by the wording, so it can be evaluated | C5, C7 |

## Risk by action

| Action | Impact | Reversibility | Blast radius | Autonomy | Untrusted input? | Tier | Minimum controls |
|---|---|---|---|---|---|---|---|
| Answer a cover question | Customer misled about cover | Correctable, but may already be relied on | One customer | Acts alone | Yes | 2 | Right wording version selected in code, citations, refusal when not found |
| Extract claim details | Wrong amount carried forward | Caught before payment if validated | One claim | Acts alone | Yes | 2 | Schema validation, source traceability |
| Pay up to £1,000 | Wrong payment | Recovering money from customers is costly and harms trust | One claim; many if exploited | Acts alone | Yes | 3 | C1, C3, kill switch, CI gate |
| Pay £1,000–£2,500 | Larger wrong payment | Hard | One claim | After approval | Yes | 4 | Handler approval |
| Decline | Customer denied a valid claim | Ombudsman complaint; regulatory breach | One customer | Must never act alone | Yes | 4 | C2: never automated |
| Route | Delay | Yes | One claim | Acts alone | — | 1 | C7 |

## Regulatory and policy screen

Checked October 2026. Not legal advice; "may apply" items go to counsel.

| Area | Applies? | Why | Action |
|---|---|---|---|
| Consumer protection (FCA Consumer Duty; claims handled promptly and fairly) | Applies | Retail insurance claims | Settlement accuracy both ways; vulnerability routing; plain explanations |
| Automated decisions about people (UK GDPR Article 22) | Applies to declines | Refusing a claim is a significant decision | C2 |
| AI transparency | May apply | UK customers only, so the EU AI Act likely doesn't reach this; the Consumer Duty's consumer-understanding outcome supports telling customers | Tell customers they're dealing with an AI and how to reach a person |
| Prohibited / high-risk uses | May apply | Life and health insurance pricing is high-risk under the EU AI Act; home contents claims are not, and the market is UK | Record in register |
| Data protection | Applies | Photos and receipts can contain personal data of others; data sent to the model vendor | DPIA; UK data residency |
| Sector rules | Applies | Fraud referrals follow the existing counter-fraud process | Use the fraud scoring service; the model doesn't judge fraud |

## Unknowns

| Unknown | Likelihood wrong | Cost if wrong | Cheapest experiment | Success criterion | Timebox |
|---|---|---|---|---|---|
| U1: Does extraction hold up on real receipt quality? | Medium | Wrong amounts | Extraction on 200 labelled receipts, including poor photos and tampered ones | Field accuracy ≥ 97% on amount; every tampered value flagged | 1 week |
| U2: Can documents inject instructions? | High | Wrong payments | 20 documents with hidden or embedded instructions, 5 runs each | Zero effect on any decision | 2 days |
| U3: How many policies lack a structured excess? | Medium | Wrong deductions | Data pull from policy admin | Count known; fix planned | 1 day |

## Next step

`ai-current-state-scan` on the innovation team's prototype, then `ai-pattern-and-controls` with
conditions C1–C8.
