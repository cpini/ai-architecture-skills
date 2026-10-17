---
name: ai-guardrail-design
description: Design guardrails and human-in-the-loop controls for an AI feature. Builds a failure-mode inventory, gives every high-impact failure one deterministic primary control, layers defence in depth (access, input, decision, output, monitoring, kill switch), chooses a gate pattern for each action from reversibility, blast radius and regulation, and designs a hand-off that actually reaches a human. Use after the pattern is chosen, when hardening an existing AI system, or when asked how to make an agent safe, prevent prompt injection, or decide where humans approve. Domain-neutral: works for agents, retrieval assistants, document pipelines, classifiers, generators and autonomous agents.
---

# AI guardrail design

Most AI systems arrive with controls that are **probabilistic**: instructions in the prompt and
the model's judgement. Those are useful but they fail some of the time, and some failures can't
be allowed to happen at all. The rule of this skill: **every high-impact failure mode gets one
deterministic primary control**, with probabilistic controls as extra layers, never the only
one.

## Inputs

- The design note from `ai-pattern-and-controls` (authority table, what the model sees).
- The use-case card's error tolerances (zeros become hard controls).
- The scan report's findings, if the system exists.

## Process

1. **Build the failure-mode inventory.** Go through the standard list and keep every mode that
   applies, with how it would show up in *this* system (a concrete example, not a definition):
   - Direct prompt injection (user types instructions or fake system messages)
   - Indirect injection (instructions inside documents, web pages, record fields)
   - Disclosure of confidential values or instructions
   - Probing (mapping limits through repeated small attempts, or across accounts)
   - Hallucination (invented facts, numbers, policies)
   - Unauthorised commitments (promises outside the tool set)
   - Jailbreak or persona abuse (offensive output in the organisation's name)
   - Channel steering and data leakage (contact or payment details, links, other users' data)
   - Stale state (acting on data that changed)
   - Concurrency races (two workers, or a human and the agent, acting at once)
   - Ambiguous inputs (amounts without currency, values only in free text, unreadable documents)
   - Runaway loops (repeated tool calls, cost blow-ups)
   - Distribution shift (new languages, product types, document layouts, currencies)
   - Automation bias (humans stop checking what the system does)
   - Vulnerable or distressed users (the system misses a person who needs a human)

2. **For each mode, fill the control table** (`templates/guardrail-plan.md`): how it shows up,
   what controls exist today, the **primary control** (deterministic where the impact is high),
   any secondary controls, and the test that proves the control works. Typical primary
   controls:
   - Limits checked in code at the point of action, failing closed
   - Secrets kept out of the model's context entirely
   - Untrusted text fenced as data, with platform markers users can't imitate
   - Detectors (regex or classifiers) for contact details, payment handles, PII, on input and
     output
   - Grounding checks: every number and claim in a reply must appear in the source record
   - Binding actions only on validated structured objects, never on free text
   - Conditional transactions and leases for state changes
   - Step, token and binding-action caps per request

3. **Layer defence in depth** and check every layer has something in it:
   - **Access**: the service account can read and write only what the feature needs.
   - **Input screens**: detectors, abuse screen (fail closed), intent routing.
   - **Decision**: policy in code, guards in the tool executor.
   - **Output screens**: grounding, commitments, moderation, the reply matches the action, a
     code-added AI label.
   - **Monitoring**: metrics from the audit log, alerts on any limit hit or guard trip.
   - **Kill switch**: a per-tenant and a global flag checked in code before any action,
     switchable without a deploy.

4. **Choose a gate pattern for each action.** Options, from lightest to heaviest:
   - **No blocking gate, digest afterwards**: inside limits the owner set; a daily digest
     counters automation bias.
   - **Review sampling**: reversible, low stakes, high volume.
   - **Approval before action**: large blast radius per action or hard to reverse.
   - **Escalation threshold**: the system hands the case to a human (outside policy, no
     grounded answer, safety flag).
   Decide with three signals: reversibility (a one-way door?), blast radius (per action and
   across all users), and regulation. One strong signal justifies a gate.

5. **Design the hand-off so it can't dead-end.** Four parts, all required:
   - A notification to the human, with the conversation and the system's proposed decision.
   - An honest holding reply to the user.
   - A response timer, and what happens when it expires.
   - A review screen where approve, edit and reject are equally easy.

6. **Hand the tests to evaluation.** Every control has a named test: unit tests for code
   controls, adversarial scenarios for injection and disclosure, output graders for screens.
   Pass the list to `ai-eval-suite-design`.

## Adapting by system type

| System type | Failure modes that dominate | Typical primary control |
|---|---|---|
| Conversational agent with actions | Injection, unauthorised commitments, actions outside policy | Policy decided in code; tool-executor guards; reply must match the action |
| Knowledge assistant (retrieval) | Hallucination, stale or wrong-version sources, permission leaks | Retrieval filtered by permissions and version in code; answer only from retrieved passages with citations; refuse when nothing is found |
| Document extraction | Indirect injection in documents, hidden or altered text, misread values | Schema and cross-field validation; values must be traceable to a location in the source; low confidence goes to a person |
| Classification and routing | High-cost misroutes, drift | Cost-weighted thresholds; rule overrides for critical classes; drift monitoring |
| Content generation | False or regulated claims, brand and IP issues | Facts injected from source data; claims checker; approval before publishing |
| Autonomous tool-using agent | Destructive actions, secret leakage, runaway cost | Least-privilege credentials, sandbox, budgets, approval before irreversible steps |
| Decision support | Automation bias, unfair outcomes | Show the evidence, not just the answer; log overrides; fairness monitoring |

## Output

A guardrail plan using `templates/guardrail-plan.md`: failure-mode and control table, defence
layers, gate patterns by action, hand-off design, and the test for each control.

## Quality bar

- Every high-impact failure mode has a deterministic primary control.
- No zero-tolerance requirement relies on the prompt alone.
- Every control fails closed unless there's a written reason.
- Every action has a gate pattern with its reason.
- The hand-off has all four parts.
- Every control has a test.

## Reference

`references/worked-example.md`: the failure-mode table, defence layers, gate patterns and
hand-off design for a fictional insurer's claims assistant, including indirect injection
through uploaded receipts and the reason declines always go to a person.
