---
name: ai-incident-to-scenario
description: Turn an AI incident, complaint, flagged conversation, guard trip, red-team finding or surprising eval transcript into a permanent regression scenario. Contains first, captures the full evidence (transcript, state, versions), classifies it against the failure taxonomy (adding a code if none fits), minimises and anonymises it, writes it as a snapshot, scripted or simulated scenario with variants, proves it reproduces on the failing version, fixes it through the eval loop, and closes with a post-incident note. Use in operation whenever something goes wrong in production, when monitoring or a user flags a conversation, or when an eval run turns up a failure nobody predicted. Works in any domain and for any kind of AI system.
---

# AI incident to scenario

Every incident is a free test case. Production finds failures no one thought to write, and the
only way to stop them coming back is to turn each one into a scenario that runs on every
change. An incident is closed when the scenario exists, fails on the version that caused it,
and passes on the fix, not when the ticket is marked done.

The same workflow applies to a surprising eval transcript: if an eval run shows something no
scenario was written for, treat it as an incident.

## Inputs

The incident report or flagged conversation, access to logs and the audit trail, the failure
taxonomy and scenario format from the eval suite (`ai-eval-suite-design`), and the guardrail
plan (`ai-guardrail-design`).

## Process

1. **Contain first, then investigate.** Classify severity against the taxonomy. For a
   critical failure (a binding action outside limits, a data leak, harmful output),
   use the kill switch or per-tenant flag now, and reverse the action if it can be reversed.
   Don't wait for the scenario to exist.

2. **Capture the evidence** while it still exists:
   - the full conversation, every turn, exactly as the system saw it (including system
     notes and tool results);
   - the state at each turn: configuration, limits, records the system read;
   - versions: commit, prompt version, model identifier, policy version;
   - every action taken and its audit record;
   - how it was detected and how long it ran.

3. **Find the blast radius.** Search the logs for the same pattern (same failure code, same
   triggering phrase, same state) over the period the faulty version ran. One incident
   reported often means several unreported.

4. **Classify.** Map it to a failure code. If no code fits, the taxonomy has a gap: add a code
   with a severity and a grader. Then ask why the eval suite missed it:
   - **No scenario** covered this situation → this skill fixes that.
   - **A scenario existed but the grader passed it** → grader gap; fix the grader with this
     transcript as its regression test.
   - **The gate let it through** (for example the scenario was already failing for another
     reason) → fix the gate rule.
   - **The failure is probabilistic** and the scenario ran too few times → raise k for that
     family.

5. **Minimise and anonymise.** Cut to the fewest turns that still trigger the failure. Replace
   names, contact details, item identifiers and anything personal; keep the language features
   that triggered it (the phrasing, the language, the formatting trick). Record provenance:
   incident ID, date, severity, "anonymised".

6. **Write the scenario** in the suite's format:
   - **Snapshot** if one decision state reproduces it;
   - **Scripted** if it needs the conversation history;
   - **Labelled item** if it's a single input with a right answer (a document, a ticket, a
     question);
   - **Simulated persona** if it's a behaviour pattern (a user tactic) rather than a fixed
     script.
   Add **2–4 variants** (paraphrase, another language or currency, a different record) so the
   fix generalises instead of memorising one transcript. Put the original in the golden set and
   the variants in the dev set, or keep the variants held out to check generalisation.

7. **Prove it reproduces.** Run the scenario against the version that failed, with k ≥ 5 if
   the failure involves the model. It must fail at least once. If it never fails, the scenario
   is missing whatever triggered the incident: go back to step 5. Record the failure rate; a
   1-in-5 failure is still a failure.

8. **Fix through the eval loop** (`ai-eval-driven-improvement`): trace to the line, decide
   prompt or code (anything unacceptable even once goes in code), change one thing, then
   prove the scenario passes on every run and the full regression set has no regressions.

9. **Close with a post-incident note** using `templates/incident-note.md`: what happened,
   impact, why each layer of defence missed it, which layer now catches it, scenario IDs, and
   follow-ups. Update the guardrail plan and `ai-register-and-raid`.

## Adapting by system type

| System type | Evidence to capture | Usual scenario form |
|---|---|---|
| Conversational agent with actions | Full conversation, state at each turn, actions and audit records | Scripted dialogue or snapshot |
| Knowledge assistant (retrieval) | The question, retrieved passages with document versions, the answer | Labelled question with gold passage |
| Document extraction | The original file (not just extracted text), extracted values, confidence | Labelled document, plus tampered variants |
| Classification and routing | The item, predicted and correct class, score, where it ended up | Labelled item; check the class threshold |
| Content generation | Brief, source facts, draft, published version | Brief with source facts and a claims check |
| Autonomous tool-using agent | Full trace of steps, tool calls, permissions used, environment state | Sandbox task with a trap for the forbidden action |
| Decision support | Inputs, recommendation, explanation, human decision, outcome | Historical case with a known outcome |

## Output

A post-incident note, new scenarios with provenance (and any new failure code or grader),
evidence that they fail before and pass after, and updates to the guardrail plan and register.

## Quality bar

- Contained before investigated, for anything critical.
- Evidence captured with versions, so the failure can be replayed exactly.
- Blast radius searched, not assumed to be one.
- The reason the eval suite missed it is named and fixed (scenario, grader, gate or k).
- The scenario is minimal, anonymised, has provenance and variants, and demonstrably fails on
  the faulty version.
- The fix is proved on pass^k plus the full regression set.

## Reference

- `templates/scenario-snippet.yaml`: scenario format with provenance fields.
- `references/worked-example.md`: an overpayment at a fictional insurer, caused by hidden text
  in an uploaded receipt and approved by a hurried handler, taken through the workflow,
  including the taxonomy gap and the approval-screen change it led to.
