---
name: ai-use-case-framing
description: Frame a proposed AI use case during discovery. Turns a brief, stakeholder notes, interview transcripts or a feature description into a use-case card with stakeholders, requirements, the three AI qualifiers, a value pillar with a guard metric, and one measurable "exam question" that every later design and eval decision traces back to. Use at the start of any AI engagement, before architecture, or when someone describes an AI feature as a capability ("the assistant handles claims end to end") rather than an outcome. Works for any domain and any kind of AI system: conversational agents, knowledge assistants, document processing, classification and routing, content generation, autonomous agents and decision support.
---

# AI use-case framing

Discovery for an AI use case ends when you can write one sentence that says what outcome the
system serves, for whom, within which hard limits, measured how. That sentence is the **exam
question**. Everything later (pattern choice, guardrails, evals, the go-live gate) is checked
against it, so a vague exam question produces a vague system.

Most briefs describe a **capability** ("the assistant reads the evidence, decides whether the
claim is covered and pays the straightforward ones"). Your job is to turn it into an
**outcome** with measurable limits ("settle valid claims within 24 hours at the amount the
policy entitles the customer to, with zero payments on excluded claims…"). The reference
example shows that move end to end.

## Inputs

Work with whatever exists: a brief, meeting or interview notes, a product spec, a feature
description, a demo, an existing codebase's README. If the use case is already built, pair this
skill with `ai-current-state-scan`.

If the input is thin, don't invent detail. Draft what you can, mark every gap as an open
question, and say which gaps block the exam question.

## Process

1. **Restate the capability, then the outcome.** One sentence each. The outcome names who
   benefits and how you would know. If you can't write the outcome, the first open question is
   "what would make the sponsor call this a success?"

2. **Map stakeholders** into four types:
   - **Sponsor**: pays for it, owns the business result.
   - **End user**: uses or configures it.
   - **Affected party**: on the receiving end without choosing it (customers, counterparties,
     employees whose work changes, regulators). Name the most exposed affected party explicitly:
     they never opted in, and their needs usually set the hard limits.
   - **Operator**: runs and supports it (operations, support, the specialist who takes hand-offs).
   For each, one line on what they need from the system.

3. **Capture requirements** in three groups:
   - **Functional**: what the system must do, as verbs (answer, extract, decide, pay, route, escalate).
   - **Non-functional**: latency, cost per interaction, availability, languages, volume.
   - **Constraints**: data residency, runtime, approved models or vendors, limits that must
     never be invented by the model.

4. **Score the three AI qualifiers** (good / partial / poor, one line of evidence each):
   - **Data availability**: is the information the model needs structured, accessible and
     owned by this system?
   - **Task feasibility**: which parts genuinely need a model (open-ended language,
     judgement) and which are deterministic (arithmetic, rules, lookups)? Say so explicitly;
     it drives the pattern choice later.
   - **Error tolerance**: per error type, not overall. Zero, near zero, moderate. The zeros
     become hard gates.

5. **Pick the value pillar and a guard metric.** Use the client's own value framework if they
   have one; otherwise choose one of five pillars, each with its typical metrics:
   - **Efficiency**: time saved per task, tasks per hour, cycle time.
   - **Transformation**: work that wasn't possible before (every item reviewed instead of a
     sample; a service offered that couldn't be staffed).
   - **Productivity**: output per person, with the same people doing more valuable work (not
     headcount reduction).
   - **Cost**: spend per task or per case, including model and review costs.
   - **Performance**: the service level itself (accuracy, resolution, time to outcome,
     customer satisfaction).
   The commonest mistake is framing everything as a cost cut: it undersells transformation and
   performance cases and makes affected staff the opponents of the project. Name the primary
   metric and a **guard metric**
   that stops the system gaming it (conversion guarded by price realisation; deflection rate
   guarded by resolution rate; speed guarded by error rate).

6. **Write the exam question.** One sentence, in this shape:

   > [Do the job] for [who], [achieving the outcome], with [zero / bounded counts of each
   > zero-tolerance error], at [a performance level no worse than the current baseline] and
   > [a cost within budget].

   Every clause must be testable. Where a number is not yet known (baseline conversion, cost
   budget), keep the clause and mark the number as an open question rather than dropping it.

7. **Test the exam question** before you hand it over:
   - Could a "naive" system that games the primary metric pass it? (A claims assistant that
     pays every claim at the amount asked settles everything within 24 hours. If your question
     only says "settle claims within 24 hours", it passes. The question has to say "at the
     amount the policy entitles the customer to".)
   - Does every zero-tolerance error from step 4 appear as a zero in the question?
   - Could each clause become an eval with a pass/fail threshold? If not, rewrite it.
   - Does it describe an outcome, not a technology? No model names, no "agent".

8. **List open questions**, each with who can answer it and which part of the card it
   changes. Mark the ones that block the exam question.

## Output

Produce a use-case card using `templates/use-case-card.md`. Keep it to one page if you can:
the card is read by sponsors as well as engineers. Lead with the outcome and the exam question,
not the background.

## Adapting by system type

The card's shape stays the same; what fills it changes.

| System type | Typical capability brief | Outcome to write instead | Typical zero-tolerance errors | Primary metric → guard metric |
|---|---|---|---|---|
| Conversational agent with actions | "Handles refunds end to end" | Resolves eligible requests correctly within a time | Actions outside policy; false statements about the customer's entitlement | Resolution rate → correctness of each action |
| Knowledge assistant (retrieval) | "Answers staff questions from our documents" | Correct, sourced answers that let staff act without escalation | Invented policy; answers from outdated documents | Deflection → answer accuracy and citation rate |
| Document extraction | "Reads invoices and enters them" | Fields captured accurately with exceptions surfaced | Wrong amounts or account numbers posted silently | Straight-through rate → field accuracy |
| Classification and routing | "Triages incoming tickets" | Each item reaches the right queue in time | High-cost misroutes (urgent item sent to backlog) | Automation rate → cost-weighted error rate |
| Content generation | "Writes product descriptions" | Publishable copy that is accurate and on brand | False claims; regulated claims without approval | Volume or time saved → edit rate and accuracy |
| Autonomous tool-using agent | "Fixes failing builds" or "runs the research" | Tasks completed to an acceptance standard within budget | Destructive or irreversible actions; leaked secrets | Tasks completed → acceptance rate and rework |
| Decision support | "Recommends a credit limit" | Recommendations that improve human decisions | Unfair or unexplainable recommendations | Adoption → outcome quality and override analysis |

Then suggest the next steps: `ai-discovery-interview-kit` if gaps or undecided policy questions
remain, `ai-feasibility-risk-triage` before any design work, and `ai-current-state-scan` if
something is already built.

## Quality bar

- Outcome first, capability second. The card never reads as a feature description.
- The most exposed affected party is named, and at least one hard limit traces to them.
- Task feasibility separates model work from deterministic work.
- Every error type has its own tolerance; zeros are explicit.
- The exam question would fail a naive system that games the primary metric.
- Unknown numbers are open questions, not omitted clauses.
- No invented facts: anything not in the inputs is labelled as an assumption or a question.

## Reference

`references/worked-example.md` is a complete worked example: a claims assistant at a
fictional home insurer, framed from a capability brief. Read it before your first use, and
match its depth. Note how the exam question changed once the sponsor realised "settle fast" was
the wrong objective, and why the guard metric has to catch overpaying and underpaying.
