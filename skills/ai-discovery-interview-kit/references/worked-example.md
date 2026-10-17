# Worked example: discovery for the Larkspur claims assistant

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

Use case (from the card): an assistant that settles contents claims up to £2,500, answering
cover questions, extracting details from receipts and photos, checking cover, calculating and
paying within a limit, and routing the rest.

## Who to talk to

| Session | Role | Why | Modules |
|---|---|---|---|
| 1 | Claims director (sponsor) | Owns the outcome and the auto-pay appetite | A, C, D, K |
| 2 | Three experienced claims handlers | They make the settlement decisions today | B, C, F, J |
| 3 | Counter-fraud lead | Knows what fraudulent contents claims look like | D, F, I |
| 4 | Complaints and vulnerable-customers lead | Speaks for customers who never chose to talk to an AI | E, D, J |
| 5 | Policy administration data owner | Where limits, excesses and wording versions live | G |
| 6 | Compliance (Consumer Duty, claims handling rules) | Fair value, automated decisions, explainability | I, K |
| 7 | Engineering lead for the prototype | What exists and what the design assumes exists | H, J |

The handlers were interviewed without their team manager present; the first session with a
manager in the room produced only the official process.

## Boundary walk on the settlement decision (session 2)

| Boundary | Question | Answer | Decided? |
|---|---|---|---|
| Single-item limit | "A laptop costs £1,800 and the single-item limit is £1,500. What do you pay?" | £1,500 minus the excess, and tell the customer why | Yes (policy wording) |
| Just under auto-pay | "Two claims from the same customer, £990 each, a week apart?" | Treat as one incident if the cause is the same; flag for review | Yes |
| No receipt | "A customer has photos of the item but no receipt. Pay?" | Handlers disagreed: two accept bank statements or photos with the box; one never pays without a receipt | **No: head of claims to decide** |
| Wear and tear | "A two-year-old coat. What deduction?" | 20% a year for clothing, capped at 60%; nothing for electronics | Yes (policy wording) |
| Customer disputes | "The customer says the deduction is unfair. Then what?" | A handler reviews it; the customer must always be able to reach a person | Yes |
| Decline | "Theft from an unlocked shed, and the policy excludes it. Can the assistant decline?" | Compliance: no automated declines; a handler reviews every proposed decline | Yes |
| Lapsed policy | "The policy lapsed the day before the loss?" | Handler review; check whether a renewal payment was in flight | Yes |
| Vulnerability | "The customer mentions a bereavement or money worries?" | Route to the vulnerable-customers team, whatever the claim value | Yes |
| Ombudsman | "The customer says they'll go to the ombudsman?" | Hand to a handler; never argue | Yes |
| Objective | "Is any amount the customer accepts fine?" | No: the right amount under the policy, neither more nor less | Yes (became the guard metric) |

## Questions for the other sessions

| Session | Question | What it surfaced |
|---|---|---|
| Counter-fraud | "What gives a fraudulent receipt away?" | Edited totals, fonts that don't match, text that's invisible on screen; the assistant shouldn't judge fraud itself but pass documents to the fraud scoring service |
| Complaints | "What do customers complain about on contents claims?" | Not understanding deductions; waiting without updates |
| Data owner | "Which wording version applies to a claim?" | The version in force on the date of loss, which isn't always the current one |
| Data owner | "Is the excess always on the policy record?" | Not for policies migrated in 2019, which hold it in a notes field |
| Engineering | "When a claim is routed to a handler, who is told?" | It lands in a shared queue with no alert and no target time |

## Examples collected

Twenty anonymised closed claims: eight straightforward, five over a limit, three without
receipts, two declined, one later found fraudulent, one involving a vulnerable customer. They
became the first golden-set cases.

## Synthesis extract

| Undecided question | Why it matters | Owner | Decide by |
|---|---|---|---|
| Pay without a receipt? | Otherwise the assistant applies whichever rule its prompt implies, inconsistently | Head of claims | Before design |
| Auto-pay limit: £500, £1,000 or £2,500? | Sets the gate between automatic payment and handler approval | Claims director with compliance | Before design |
| Target time for routed claims | A hand-off with no target can dead-end | Claims operations manager | Before pilot |

| Conflict | View A | View B | Who resolves |
|---|---|---|---|
| Receipts | "Never pay without one" (senior handler) | "Bank statements are fine" (two handlers, and the published guidance) | Head of claims |

Decisions taken at the playback: proof of ownership may be a receipt, a bank or card statement,
or a photo showing the item and its packaging; auto-pay up to £1,000, with handler approval
between £1,000 and £2,500. Both were recorded as decisions and later encoded in the cover
rules.
