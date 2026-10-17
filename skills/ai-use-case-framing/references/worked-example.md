# Worked example: Larkspur claims assistant

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

Context: Larkspur is a UK home insurer with about 400,000 policies and a claims team of 60
handlers. Contents claims under £2,500 make up most of the volume and take a handler an average
of six working days to settle, mostly waiting in a queue. The claims director brought this
brief:

> "We want an AI assistant that handles contents claims end to end. It reads the customer's
> photos and receipts, decides whether the claim is covered, works out the payment and pays the
> straightforward ones without a handler."

That is a capability. The card below turns it into an outcome.

---

## Outcome

- **As described (capability):** an assistant that reads evidence, decides cover, calculates
  and pays straightforward contents claims.
- **Outcome it serves:** customers with valid small contents claims are paid the right amount
  quickly, and handlers spend their time on the claims that need judgement.

## Exam question

> Settle valid contents claims up to £2,500 within 24 hours at the amount the policy wording
> entitles the customer to, with zero payments on claims the policy excludes, zero payments
> above policy limits or without the excess deducted, zero declines without a handler's review
> and no invented policy terms, at a complaint rate no higher than handler-settled claims and a
> cost per claim within budget.

Numbers still to set: cost budget per claim; the complaint baseline for handler-settled
contents claims.

### How the exam question changed (the lesson to copy)

The first draft said "**settle contents claims up to £2,500 automatically within 24 hours**".
Step 7 caught it: a naive assistant that pays every claim at the amount the customer asks for
settles 100% within 24 hours and passes. The rewrite added "valid" and "at the amount the policy
wording entitles the customer to".

The second correction came from the compliance lead: under the FCA's Consumer Duty, **paying
too little is a harm too**, not a saving. So the guard metric couldn't be "overpayment" alone.
It became settlement accuracy, which counts underpayments and overpayments.

## Stakeholders

| Type | Who | What they need from the assistant |
|---|---|---|
| Sponsor | Claims director | Faster settlement, lower handling cost, no rise in complaints or leakage |
| End user | Customers making a claim | A quick, fair, understandable decision; a person when they want one |
| Affected party | Other policyholders | Premiums not inflated by overpayments or fraud |
| Affected party | The regulator and the Financial Ombudsman Service | Claims handled promptly and fairly; decisions explainable; vulnerable customers identified |
| Operator | Claims handlers and the counter-fraud team | Clean case files, clear hand-offs, nothing approved they'd have refused |

**Most exposed affected party: the customer making the claim.** Many claim after a burglary or
a flood, some are vulnerable, and few can tell whether the amount offered is right. Their needs
set three hard limits: no declines without a person, no invented policy terms, and a route to a
human at any point.

## Requirements

- **Functional:** answer cover questions from the customer's own policy wording; collect and
  extract claim details from messages, photos and receipts; check cover; calculate the
  settlement; pay within an auto-pay limit; route other claims to handlers or counter-fraud;
  explain every decision in plain English.
- **Non-functional:** decision within 24 hours of a complete claim; available around the clock;
  English and Welsh; peak volume after storms at five times normal.
- **Constraints:** policy limits, excesses and wear-and-tear rules come from the policy
  administration system, never from the model; payments only through the existing payments
  API; customer data stays in the UK.

## AI qualifiers

| Qualifier | Rating | Evidence |
|---|---|---|
| Data availability | Partial | Policy records and wording are structured, but wording exists in eleven versions and receipts arrive as photos of varying quality |
| Task feasibility | High for parts | Model work: reading messages, extracting receipt details, explaining decisions, answering cover questions. Deterministic work: **cover rules and settlement arithmetic need no model at all** |
| Error tolerance | Per type | See below |

| Error type | Tolerance |
|---|---|
| Paying an excluded claim | Zero |
| Paying above a limit, or without deducting the excess | Zero |
| Declining without a handler's review | Zero |
| Inventing a policy term | Zero |
| Extraction error caught before payment | Moderate |
| Tone | Moderate |

The task-feasibility line is the most important on the card: "settlement arithmetic needs no
model" later justified putting cover and settlement in code.

## Value

- **Pillar:** Performance (the service level customers get: time to a fair settlement), with
  efficiency second (handler time moved to complex claims). The first draft said "cost: fewer
  handlers"; the sponsor rejected it because the business case was faster, fairer settlement,
  and because handlers would review the assistant's work.
- **Primary metric:** share of valid contents claims settled within 24 hours.
- **Guard metric:** settlement accuracy, the share of auto-settled claims paid within £1 of
  the amount the policy entitles the customer to, reported separately for overpayments and
  underpayments.

## Open questions

| Question | Who can answer | Changes | Blocks exam question? |
|---|---|---|---|
| What is the cost budget per claim? | Claims director | Exam question number, model choice | Yes (number only) |
| What is today's complaint rate for handler-settled contents claims? | Complaints team | Exam question number | Yes (number only) |
| Can the assistant pay without a receipt, using other proof of ownership? | Head of claims | Requirements, eval oracle | No |
| What should the auto-pay limit be? | Claims director with compliance | Gate design | No |

## Next step

A prototype already existed (built by the innovation team), so the next steps were
`ai-discovery-interview-kit` for the open policy questions, `ai-feasibility-risk-triage`, and
`ai-current-state-scan` on the prototype.
