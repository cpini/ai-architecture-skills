---
name: ai-eval-suite-design
description: Design and build an evaluation suite for an AI feature that becomes its regression gate. Turns the exam question into metrics with targets, defines an oracle of allowed outputs per input or state, a failure taxonomy with severities, deterministic graders first and a calibrated LLM judge last, a scenario matrix (labelled datasets, single-decision snapshots, scripted adversarial cases, simulated users with hidden goals), pass^k on repeated runs, reference and naive agents to prove the graders, and the CI gate rules. Use when someone asks how to evaluate, test or regression-test an LLM agent, chatbot or workflow, before go-live, or when a feature has an audit log but no evals. Works for agents, chatbots, retrieval assistants, extraction and classification pipelines, generators and autonomous agents in any domain.
---

# AI eval-suite design

An eval suite answers one question before every change ships: **is this version at least as
safe and at least as good as the one we have?** It has to measure the outcome the business
wants, not a proxy that a bad system can game, and it has to be cheap and repeatable enough to
run on every pull request.

## Inputs

- The exam question and error tolerances (`ai-use-case-framing`).
- The controls and their tests (`ai-guardrail-design`) and design decisions
  (`ai-pattern-and-controls`).
- The scan findings (`ai-current-state-scan`): each finding should map to at least one
  scenario.

## Process

1. **Turn every clause of the exam question into a metric** with a target and a grader type:
   - **Hard gate**: zero-tolerance errors. Must be zero in every run.
   - **Threshold**: quality measures with a minimum (or a maximum drop from baseline).
   - **Business outcome**: measured live in a pilot, not in the suite.
   Add a **guard metric** for each primary metric so the system can't game it (claims settled
   quickly guarded by settlement accuracy; deflection guarded by answer accuracy).

2. **Define the oracle: what is allowed for each input or state, not one right answer.**
   Write the policy as a table of states (inputs, history) to allowed outputs or actions.
   Different valid strategies should pass; unsafe or value-losing ones should fail. Implement
   it as code. Where there is one right answer (an extracted field, a class label, a calculated
   amount), the oracle is the labelled answer plus a tolerance.

3. **Define the failure taxonomy.** One code per way of failing, each with a severity:
   - **Critical**: breaks a hard rule. Fails the run and blocks release.
   - **Major**: loses value or breaks policy. Fails the run.
   - **Soft**: worth tracking, doesn't fail a run.
   The same codes go in the graders, the dashboards, the production logs and the waiver list,
   so a failure means the same thing everywhere.

4. **Grade with code first.** Most of what matters (actions, amounts, fields, labels, state
   transitions, what a reply mentions) is checkable exactly:
   - **Action graders**: the oracle, applied to every decision.
   - **Field and label graders**: exact or tolerance match per field; confusion matrix per
     class, weighted by the cost of each error.
   - **Grounding graders**: every claim and number in an answer traces to a retrieved passage
     or source record; citations point to the right document and version.
   - **Text graders**: regex and parsing for disclosures, amounts not decided, contact
     details, promises, a missing AI label, and whether the reply matches the action taken.
   - **Outcome graders**: how the whole case ended, against what was possible.
   - **LLM judge**, last and only for what code can't see (tone, language, pressure tactics,
     invented facts in prose). Use a different model family from the system under test,
     calibrate it on 50+ human-labelled examples to at least 85% agreement, and let it raise
     only soft flags. It never overrides code graders on hard rules.

5. **Build the scenario matrix** from these kinds (use the ones that fit the system):
   - **Labelled dataset**: real inputs with human-labelled answers (documents with gold
     fields, tickets with gold classes, questions with gold passages). The core of any
     extraction, classification or retrieval eval. Sample for coverage of rare, costly cases,
     not just volume.
   - **Snapshot**: a single decision point (history plus one new input). Fast, precise, covers
     every policy edge: at, just above and just below each threshold.
   - **Scripted**: fixed multi-turn cases for adversarial families: direct and indirect
     injection (including inside documents), fake system messages, probing for secrets,
     requests to move off the channel, emotional pressure, other languages, ambiguous amounts.
   - **Simulated**: a simulated user with a persona and a **hidden truth or goal** (the real
     loss behind a claim, the budget a customer has in mind, the actual cause of a fault) plays out
     a full conversation. This is what measures outcomes against what was achievable. Use a
     seeded rule-based simulator for pull requests (deterministic, free) and an LLM simulator
     nightly (realistic). Code keeps the simulator consistent with its hidden truth.
   Cross personas, hidden-truth bands, configurations and input types so every cell is covered.
   Generate the matrix with a script, so changes are reviewable.

6. **Split dev and golden.** The dev set is what you iterate on and is the regression suite.
   The golden set is held out, never tuned against, and used at release. A script checks no
   golden case duplicates a dev case. Every golden case records where it came from.

7. **Run each scenario k times and use pass^k**: a scenario passes only if every run passes.
   One bad decision in five conversations is a real incident rate, not noise.

8. **Prove the graders before trusting them.** Run two bundled agents through the suite:
   - A **reference** implementation of the policy, which should pass almost everything.
   - A **naive** system that games the primary metric (approves everything, answers
     everything, routes everything to the default queue), which should fail badly on the guard
     metric.
   If the naive agent passes, your metrics are gameable. Fix them before testing the real
   system.

9. **Connect the real system** through an adapter that runs the production code with side
   effects recorded, not executed (a recording API client; no database writes, no messages
   sent).

10. **Define the gate** (`compare.py`-style, comparing a run with a committed baseline). Block
    a merge when:
    - a scenario that passed every run in the baseline no longer does;
    - critical failures rise, or an already-failing scenario gains a new critical code;
    - a threshold metric drops by more than the tolerance.
    Already-failing scenarios that fail the same way are reported, not blocking, so the gate
    measures change rather than perfection. Known failures go in a waivers file with an owner
    and an expiry date.

11. **Write the workflow**: local loop, pull request gate, nightly, release gate, production
    sampling with the same failure codes, and every incident becoming a scenario before its
    fix (`ai-incident-to-scenario`).

## Adapting by system type

| System type | Core scenario kinds | Key metrics and graders | Naive system that must fail |
|---|---|---|---|
| Conversational agent with actions | Snapshots, scripted adversarial, simulated users | Oracle on every action; reply-matches-action; outcome vs achievable | Approves or agrees to everything within limits |
| Knowledge assistant (retrieval) | Labelled question set with gold passages; adversarial and unanswerable questions | Retrieval recall@k; answer correctness; grounding and citation; correct refusal rate | Always answers confidently, never refuses |
| Document extraction | Labelled documents across layouts, quality and languages; tampered documents | Field accuracy per field; exceptions caught; hidden-text and injection cases | Copies the largest number on the page into the amount field |
| Classification and routing | Labelled items with costly classes oversampled | Cost-weighted confusion matrix; recall on critical classes | Sends everything to the most common class |
| Content generation | Briefs with source facts; banned-claim traps | Factual consistency with source; banned or regulated claims; rubric judged for style | Copies the brief, or writes fluent text with invented facts |
| Autonomous tool-using agent | Task suites in a sandbox with acceptance tests; traps for destructive actions | Task success; cost and steps; zero forbidden actions | Declares success without doing the work |
| Decision support | Historical cases with known outcomes | Outcome lift versus baseline; calibration; fairness across groups | Always recommends the most common decision |

## Output

An eval plan using `templates/eval-plan.md`, and, if asked to build, a harness with: data
models, oracle, graders, simulators (where used), scenario generator, runner, compare, regrade
and a labelling sheet for judge calibration.

## Quality bar

- Every exam-question clause has a metric and a grader; every zero is a hard gate.
- A naive agent that games the primary metric fails the suite.
- The oracle allows any valid strategy, not one scripted answer.
- Code grades everything code can grade; the judge is calibrated and soft-only.
- Every scan finding and every guardrail has at least one scenario.
- pass^k on repeated runs; seeded simulators for the gate.
- Dev and golden are split and checked.

## Reference

`references/worked-example.md`: the eval suite for a fictional insurer's claims assistant,
which mixes four system types in one product (extraction, retrieval Q&A, routing and a
conversational agent with a payment action): metrics, oracle, failure codes, the scenario
matrix with simulated claimants, the naive-versus-reference proof and the gate rules.
