---
name: ai-eval-driven-improvement
description: Improve an AI feature one measured change at a time using its eval suite. Reads transcripts not just scores, triages every failure (agent bug, grader false positive, wrong expectation, policy question), traces each cluster to the prompt line, tool description or code that causes it, makes one change, re-runs the affected scenarios and the full regression set against the last baseline, and keeps a fix log. Use after the first eval run of a real system, when results surprise you, when deciding whether to fix something in the prompt or in code, or when asked "how do we make the evals pass". Domain-neutral and works for any AI system type.
---

# AI eval-driven improvement

Evals tell you *that* something fails. Improvement comes from working out *why*, changing one
thing, and proving the change helped without breaking anything else. The loop is simple; the
discipline is in not skipping steps.

## The loop

1. **Run and save a baseline.** Same scenarios, same settings, before any change. Every later
   run is compared against the latest baseline, never against memory.

2. **Read transcripts, not just scores.** For every failing run, read the input, what the
   system did, any values it chose and the output it produced. Scores tell you where to look; transcripts tell
   you what happened. Always also skim a few *passing* runs: graders can pass bad behaviour.

3. **Triage every failure into one of four buckets:**
   - **Agent bug**: the system did something wrong. Fix it, or waive it with an owner and
     an expiry date if it must wait.
   - **Grader false positive**: the system was right and the grader was wrong. Fix the grader
     and add the exact reply as a regression test for the grader. Expect several of these in
     the first runs; regex graders meet real language.
   - **Wrong expectation**: the scenario expected something the policy doesn't require. Fix the
     scenario.
   - **Policy question**: nobody had decided what should happen. Ask the owner, record the
     decision (ADR), encode it in the oracle.
   After changing any grader, **re-score stored transcripts** (no model calls) and re-score the
   baseline too, so comparisons stay fair.

4. **Group agent bugs by root cause, then trace each to its source.** Several failure codes
   often come from one line. Open the prompt, the tool descriptions (they are prompt too) and
   the code, and find the line responsible. Quote it.

5. **Decide prompt or code.** Rule of thumb from the evidence:
   - The failure is about **words** (tone, language, what the reply claims) → prompt.
   - The failure is **arithmetic, state or a hard rule** (amounts, counts, limits, sequences)
     → code. A prompt fix may help, but if the failure persists or passes only some runs, move
     the decision into code. Don't keep tuning the prompt for arithmetic.
   - The failure is **unacceptable even once** → code, regardless (a guard that refuses the
     action).
   Prefer a pair: the prompt steers, a code guard enforces.

6. **Change one thing.** One root cause per change, even if it touches a few files. Add unit
   tests for any code change. Several changes at once make it impossible to tell which helped
   or which broke something.

7. **Measure in two steps:**
   - **Targeted**: re-run only the affected scenario families, with k ≥ 3 if the change is in
     the prompt (model behaviour varies between runs).
   - **Regression**: the full comparison set against the baseline through the gate. A change
     that fixes its target but breaks something else is not done.

8. **Read the comparison carefully.** Watch for:
   - "Regressions" that are grader false positives triggered by *better* wording.
   - New critical codes on scenarios that were already failing for another reason: the gate
     must block these, or a worse failure hides behind a known one.
   - Side effects on neighbouring behaviour (cutting overpayments can increase
     underpayments; stricter refusals can increase escalations).

9. **Save the new baseline and log the fix**: change, where, scenarios passing, critical runs,
   key metric. Then pick the next root cause.

10. **Calibrate periodically.** Build a labelling sheet from transcripts; a human labels 20–50
    turns; measure agreement with the graders per failure code. Codes below ~85% agreement get
    fixed before they're trusted to block merges.

## Adapting by system type

The loop is identical; the typical root causes and fixes differ.

| System type | Common root causes | Usually fixed in |
|---|---|---|
| Conversational agent with actions | Prompt rules that contradict the objective; arithmetic or state in the prompt; reply drifting from the action | Code for decisions and state; prompt for wording |
| Knowledge assistant (retrieval) | Retrieval misses (chunking, filters, query), answers beyond the passages, wrong document version | Retrieval pipeline and filters first; prompt for grounding and refusal |
| Document extraction | Ambiguous schema, unseen layouts, values without a source location, OCR noise | Schema and validation in code; examples in the prompt; routing low confidence to people |
| Classification and routing | Overlapping label definitions, imbalanced classes, thresholds not tied to error cost | Label definitions (and relabelling), thresholds in code |
| Content generation | Facts not supplied, style rules vague, claims rules only in the prompt | Inject facts from data; checker in code; prompt for style |
| Autonomous tool-using agent | Poor tool descriptions, missing feedback from tools, no stopping rule | Tool design and permissions; budgets in code; prompt for planning |
| Decision support | Miscalibrated scores, explanations that don't match the score | Model or rules recalibration; explanation built from the same inputs |

## Ordering the fixes

- Critical before major; safety before value.
- Fix what other fixes depend on first (counting and state bugs before tuning thresholds).
- When three or more fixes are all about arithmetic or state, stop patching and redesign:
  move the decision into code (see `ai-pattern-and-controls`).

## Output

A fix log using `templates/fix-log.md`, updated after every change, plus the triage sheet for
each run. Saved baselines in version control (`baselines/v0`, `v1`, …).

## Quality bar

- Every failure is triaged; none is fixed without reading its transcript.
- Every grader change has a regression test built from the real reply that exposed it.
- One root cause per change; every change measured on targeted and full sets.
- Prompt-versus-code decided on evidence, with guards for anything unacceptable even once.
- The fix log shows the metric moving at each step.

## Reference

`references/worked-example.md`: a fictional insurer's claims assistant taken from 52% to 100%
of core scenarios passing in measured changes, including grader false positives, an injection
surprise found mid-way, and the point where patching stopped and cover and settlement moved
into code.
