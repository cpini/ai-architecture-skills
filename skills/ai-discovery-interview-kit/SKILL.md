---
name: ai-discovery-interview-kit
description: Plan, run and synthesise discovery interviews and workshops for an AI use case. Picks who to talk to from the stakeholder map, builds a guide per interview from a question bank, asks for real examples instead of opinions, hunts for undecided policy questions at every boundary the system will face, and synthesises notes into facts, decisions, conflicts and open questions that update the use-case card. Use when preparing discovery sessions, when a brief is thin, when stakeholders disagree about what the AI should do, or before writing evals that need someone to decide what "correct" means. Works in any domain and for any kind of AI system.
---

# AI discovery interview kit

AI systems make decisions at boundaries nobody thought about: the request just over the limit,
the customer who says "final", the document with no date, the instruction that conflicts with
policy, the message in another language. If discovery doesn't ask what should happen there, the model decides, and
the first time anyone notices is an incident or a failing eval.

This kit has two jobs. The ordinary one: gather facts, baselines and examples for the use-case
card. The important one: **find the policy questions nobody has decided** and get them decided
by the person who owns them, before build.

## Inputs

A brief or draft use-case card, the stakeholder map (from `ai-use-case-framing`), and any
existing artefacts: process docs, sample conversations, dashboards, the current system.

## Process

1. **Choose interviewees** from the stakeholder types. Cover at least:
   - **Sponsor**: outcome, value, appetite for risk, what failure would embarrass them.
   - **Process owner or end user**: how the work is done today, the decisions in it, the
     numbers.
   - **Operator**: who handles exceptions and hand-offs, how fast, with what tools.
   - **Affected-party proxy**: customer service, complaints, user research; the people who
     hear from those who never opted in.
   - **Data and system owners**: what exists, where, its quality, who may use it.
   - **Risk, legal or compliance**: transparency, data protection, sector rules.
   - **Engineering**: runtime, integration points, what's already built.
   Pair or group interviews only when people won't contradict a senior colleague in the room.

2. **Build a guide per interview** from `references/question-bank.md`. Pick the modules for
   that role, 8–12 questions for 45 minutes, with the boundary questions (module F) for anyone
   who owns a decision. Use `templates/interview-guide.md`.

3. **Ask for examples, not opinions.** "Walk me through the last claim you paid without a
   receipt" beats "how do you handle missing receipts?". Ask for artefacts: real conversations,
   tickets, spreadsheets. Collect 10–20 real cases across normal, hard and bad; they become
   eval seeds and the first golden set (anonymised).

4. **Hunt for policy questions.** For every decision the system will make, walk its
   boundaries and ask "what should happen when…":
   - just above, at, and just below each limit;
   - when the other party says it's final, repeats, or escalates;
   - when instructions conflict (customer vs owner vs platform policy);
   - when data is missing, stale or contradictory;
   - in another language, currency or market;
   - outside hours, or when the human it hands off to doesn't respond;
   - when the system could do better than "acceptable" (is any valid answer fine, or is
     there an objective to maximise?).
   For each, record the answer and who gave it. "It depends" or two different answers from
   two people means **undecided**: name the owner who must decide.

5. **Get the numbers.** Every exam-question clause with a number needs a baseline: today's
   reply time, conversion, error rate, cost, volume. Record the source, or mark it unknown and
   who can pull it.

6. **Observe if you can.** Thirty minutes watching the work catches what interviews miss:
   workarounds, the spreadsheet beside the system, the decisions people make without noticing.

7. **Synthesise within 24 hours** with `templates/synthesis.md`: facts (with source),
   quotes, examples collected, decisions made, undecided policy questions with owners,
   conflicts between stakeholders, and changes to the use-case card.

8. **Play it back.** A 30-minute session with the sponsor and decision owners: confirm the
   facts, decide the open policy questions, or set a date to. Record each decision where the
   build will see it (an ADR, then the eval oracle).

## Adapting by system type

The boundary walk changes with what the system decides.

| System type | Boundaries to walk | Examples to collect |
|---|---|---|
| Conversational agent with actions | Limits on each action; final or repeated requests; complaints; hand-off timing | Real conversations, including bad ones |
| Knowledge assistant (retrieval) | Questions the documents don't answer; conflicting documents; superseded versions; who may see what | Real questions with the answer an expert would give and its source |
| Document extraction | Missing, unreadable or contradictory fields; new layouts; handwritten notes | Documents across layouts and quality, with correct values |
| Classification and routing | Items that fit two classes; urgent items; what a wrong route costs per class | Items with agreed labels, including disputed ones |
| Content generation | Claims that need approval; tone limits; facts that must come from data | Approved and rejected drafts, with reasons |
| Autonomous tool-using agent | Actions that need approval; when to stop; budget limits | Past tasks with acceptance criteria |
| Decision support | When people overrule the recommendation and why; fairness concerns | Past decisions with outcomes |

## Output

Interview guides, a synthesis note, an updated use-case card, a list of decisions and
undecided questions (which feed `ai-register-and-raid`), and a folder of anonymised real
examples for `ai-eval-suite-design`.

## Quality bar

- Every stakeholder type is covered, including someone who speaks for the affected party.
- Every decision the system makes has had its boundaries walked with its owner.
- Undecided questions each have a named owner and a date, not "TBC".
- Facts carry a source; numbers carry a source or are marked unknown.
- At least 10 real examples collected, or the gap is recorded as a risk.
- Conflicts between stakeholders are written down, not smoothed over.

## Reference

- `references/question-bank.md`: questions by module, with which roles to ask.
- `references/worked-example.md`: the discovery plan for a fictional insurer's claims
  assistant: who to interview, the boundary walk on the settlement decision, and the policy
  questions it surfaced before build.
