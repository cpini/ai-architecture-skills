---
name: ai-feasibility-risk-triage
description: Triage a framed AI use case for feasibility and risk before anyone designs or builds it. Splits the job into steps, rates feasibility (data, task fit, integration, economics, operability), tiers the risk of every action the system can take by impact, reversibility, blast radius and autonomy, screens regulation and policy, ranks the biggest unknowns with the cheapest experiment to retire each, and ends in a verdict with named conditions. Use after a use-case card exists, when someone asks "should we build this?", "is this safe to automate?" or "how risky is this agent?", or before committing budget to a pilot. Works in any domain and for any kind of AI system.
---

# AI feasibility and risk triage

The use-case card says what outcome the system serves. Triage answers two questions before
design starts: **can it work here**, and **what is the worst it can do**. The answer is never
just "go" or "no". It is usually "go, on these conditions", and the conditions become design
inputs, guardrails and eval gates.

The most useful finding is often not "the model can't do this" but "the model shouldn't decide
this". Arithmetic, limits and rules that a model *could* handle still belong in code, and
triage is the cheapest place to say so.

## Inputs

A use-case card (from `ai-use-case-framing`) or equivalent. If something is already built, the
scan report from `ai-current-state-scan` too. If the card is missing, draft its exam question
first; triage without an outcome measures the wrong things.

## Process

1. **Split the job into steps** and label each one:
   - **Language**: understanding or writing open-ended text. Needs a model.
   - **Judgement**: weighing ambiguous evidence where no rule exists. Probably needs a model.
   - **Decision**: choosing an amount, a yes/no, a route, where a rule can be written.
     Belongs in code even if a model could do it.
   - **Retrieval**: finding facts in records or documents.
   - **Action**: anything that changes the world (sends, pays, agrees, deletes). Note who it
     binds.

2. **Rate feasibility** green / amber / red, one line of evidence each:
   - **Data**: does the information exist, can the system reach it, is its quality known, does
     the organisation have the right to send it to a model vendor?
   - **Task fit**: is there evidence (a spike on 20–50 real examples, not a demo) that a model
     does the language and judgement steps well enough? Are decision steps being given to a
     model that a rule could own?
   - **Integration**: systems of record, write access, latency budget, events or triggers, and
     anything the design assumes exists but doesn't (a notification, an approval screen).
   - **Economics**: rough cost per interaction (tokens × price × turns) × volume, against the
     value per interaction. Note when the budget is unknown.
   - **Operability**: who supports it, who receives hand-offs and how fast, whether "correct"
     can be defined well enough to build an eval.

3. **Tier the risk of every action.** For each action from step 1, score:
   - **Impact** if wrong: financial, legal, safety, reputational, to whom.
   - **Reversibility**: undone freely, undone at a cost, or not at all.
   - **Blast radius**: one interaction, one customer, many.
   - **Autonomy**: suggests, acts after approval, acts alone.
   - **Exposure**: does untrusted input (customer text, documents, web pages) reach the step
     that decides this action?

   Then assign a tier using `references/risk-tiers.md`. The tier sets the minimum controls.
   Tier the action, not the system: one agent can hold tier-1 and tier-4 actions.

4. **Screen regulation and policy** with the checklist in `references/risk-tiers.md`: AI
   transparency, prohibited and high-risk uses, automated decisions about people, data
   protection, consumer protection, sector rules, vendor and contract terms. For each, say
   *applies / may apply / doesn't apply* and why. This is a screen, not legal advice: anything
   marked "may apply" goes to counsel with the facts, and rules change, so record the date
   checked and verify current status.

5. **Rank the unknowns.** List what you don't know, scored by how likely it is to be wrong
   and how expensive being wrong would be. For the top three, propose the **cheapest
   experiment** that retires it, with a success criterion and a timebox:
   - a prompt spike on 20–50 real examples,
   - a handful of adversarial scripted conversations,
   - simulated users with hidden goals,
   - shadow mode on live traffic (the system drafts, a human acts),
   - a data pull to check quality or baselines.

6. **Give a verdict** and its conditions:
   - **Go**: feasible, risks within appetite with ordinary controls.
   - **Go with conditions**: each condition named, owned and testable (most cases).
   - **Reshape**: feasible only with lower autonomy, narrower scope, or a decision moved to
     code or a human.
   - **Not yet**: an unknown must be retired first (run the experiment, then re-triage).
   - **No**: risk outside appetite at any autonomy level, or no value case.

## Adapting by system type

| System type | Feasibility question that matters most | Actions that usually set the tier |
|---|---|---|
| Conversational agent with actions | Can the decisions be written as rules, leaving the model only language? | Payments, refunds, commitments, account changes |
| Knowledge assistant (retrieval) | Are the documents current, owned, permissioned and good enough to answer from? | Answers people act on (policy, legal, medical, safety) |
| Document extraction | Is there a labelled sample across real layouts and quality, and a check for each field? | Posting values to systems of record or payments |
| Classification and routing | Are the classes agreed and is the cost of each error known? | Routes that delay urgent or regulated items |
| Content generation | Are the facts available as data, and who approves what is published? | Publishing externally; regulated claims |
| Autonomous tool-using agent | Can it run in a sandbox with least privilege and a budget? | Deleting, deploying, spending, emailing outside the organisation |
| Decision support | Is there outcome data to show it helps, and can a person meaningfully disagree? | Recommendations with significant effects on people (credit, hiring, care) |

## Output

A triage note using `templates/triage-note.md`. Lead with the verdict and conditions; sponsors
read the first half page. The conditions feed `ai-options-and-tradeoff-analysis` (as must-haves),
`ai-pattern-and-controls` and
`ai-guardrail-design`; the unknowns feed the first eval scenarios; the regulatory screen and
risk tiers feed `ai-register-and-raid`.

## Quality bar

- Every step labelled; every decision step that could be a rule is flagged for code.
- Every action tiered on its own, with evidence for each score.
- Every amber or red feasibility rating has an experiment or a condition attached.
- Regulatory lines say applies / may apply / doesn't apply, with the date checked.
- The verdict's conditions are testable, so an eval or a review can prove each one.
- No invented facts: unknown numbers are unknowns, not estimates dressed as findings.

## Reference

- `references/risk-tiers.md`: tier definitions, minimum controls per tier, and the
  regulatory screen.
- `references/worked-example.md`: a triage of a fictional insurer's claims assistant, with
  each step labelled model or code, a risk tier per action (including why declines can never
  be automated), a regulatory screen and a verdict with conditions.
