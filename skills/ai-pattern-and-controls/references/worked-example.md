# Worked example: redesigning the Larkspur claims assistant

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

Inputs: the use-case card, the triage note (conditions C1–C8) and the scan report (findings
F1–F9) for the prototype.

## Task decomposition

| Step | Type | Owner | Rule (for decisions) |
|---|---|---|---|
| Read the customer's messages | Language | Model | — |
| Extract receipt and photo details into the claim schema | Language | Model | — |
| Validate extracted fields | Decision | Code | Every value has a source location; totals add up; dates before the loss date |
| Answer cover questions | Retrieval + language | Code selects the wording version in force at the loss date and includes it whole (it fits; see `ai-retrieval-design`); model answers with citations | — |
| Check cover | Decision | Code | Policy in force at loss date; peril covered; no exclusion applies |
| Calculate settlement | Decision | Code | min(item value after wear and tear, single-item limit) summed, minus excess |
| Route | Decision | Code | ≤ £1,000, fraud score low, all fields valid → auto-pay; ≤ £2,500 → handler approval; any proposed decline → handler; fraud flag → counter-fraud; vulnerability signal → vulnerable-customers team |
| Pay | Action | Code | Only the calculated amount, only on the auto-pay route |
| Explain the decision | Language | Model, from a decision note | — |

Every decision step could be written as a rule, so every decision step went to code.

## Pattern

**Rung 3: fixed workflow with model steps**, replacing the prototype's rung-4 agent.

| Rung | Rejected because |
|---|---|
| 1. Single prompt | Needs the policy record, the right wording and documents |
| 2. Augmented model | Would still leave cover and payment decisions with the model |
| 4. Agent | The path is known in advance; payments and declines are too consequential for the model to choose the sequence |

## Workflow pattern and decomposition

**Prompt chaining plus routing.** The chain is extract → validate → cover → settle → explain;
the router (plain rules, no model) chooses the path after settlement.

| Split | Runs alone? | Checkable alone? | What merging would lose | Kept? |
|---|---|---|---|---|
| Extraction separate from explanation | Yes | Yes: field accuracy against labels | A smaller, cheaper model for extraction; a clean audit of what was read | Yes |
| Cover separate from settlement | Yes | Yes | Nothing: both are code in one engine | Merged into one engine |
| One extraction call per document vs one per claim | Yes | Yes | Per-document calls run in parallel and fail independently | Per document (parallelisation by sectioning) |

**Rejected:** orchestrator-workers (the subtasks are known in advance); evaluator-optimizer for
explanations (code checks against the decision note are more reliable than a critic model);
multi-agent (no broad parallel research; it would multiply cost for no gain).

## Four stages

| Stage | Design |
|---|---|
| Input handling | Upload checks, hidden-text detection, extraction into a schema, validation |
| Processing | Cover and settlement engine; explanation call |
| Output delivery | Amount and grounding checks before any message; payment executor re-check |
| Feedback | Handler edits and overturned decisions become eval cases weekly; complaints tagged by failure code |

The prototype had no feedback stage: handler corrections were stored as notes nobody read.

## Authority

| Action | Who decides | Limits | Where enforced | Fail mode |
|---|---|---|---|---|
| Pay | Code (settlement calculator and router) | Single-item limit, excess, auto-pay limit | Calculator; payment executor re-checks; payments API caps at £1,000 for the assistant's service account | Closed: any missing value routes to a handler |
| Decline | Nobody automatically | Handler review always | The model has no decline tool; the router can only propose | Closed |
| Route | Code | Thresholds above | Router | Closed: unknown → handler |
| Answer | Model | Only from the right wording version | Version selection in code; grounding check on output | Closed: "I'll ask a handler" |

## What the model sees

- **Extraction call:** the document, fenced as data, and the schema. No limits, no policy.
- **Answer call:** the customer's question and the full text of their wording version (cached),
  with clause references.
- **Explanation call:** a decision note, for example "[Decision: pay £412.50. Coat £600, less
  40% wear and tear = £360; television £302.50; total £662.50 less £250 excess.]", and the
  customer's messages. Not the rules, the limits, the fraud score or the thresholds.
- **Tools:** none that bind. The model writes; code acts.

## Risk and autonomy tier

The prototype, with limits in the prompt and a model-chosen amount, was tier 4. The redesign,
with limits in code, a capped service account and no automated declines, is tier 3 for auto-pay
up to £1,000 and tier 4 (approval before action) above it.

## Target architecture

| Container | New or kept | Owns |
|---|---|---|
| Intake service | New | Messages, uploads, extraction calls, validation |
| Cover and settlement engine | New | Rules from the policy record; calculation; decision note |
| Router | New | Routes and thresholds |
| Explanation service | Changed from the agent | Model call that writes the customer message from the decision note |
| Wording service | Changed (vector index removed) | Returns the wording version in force at the loss date |
| Payments API | Kept | Idempotent payments; new per-service cap |
| Handler workbench | Changed | Approval screen showing the evidence beside each value |

## ADRs

| ADR | Decision | Downside stated |
|---|---|---|
| ADR-001 | Cover and settlement are decided in code from the policy record | Rules must be maintained for each wording version |
| ADR-002 | No automated declines; the router proposes and a handler decides | Slower declines; handler workload |
| ADR-003 | Documents are fenced data; extraction outputs a schema in which every value cites its source location | Some messy receipts fall out to handlers |
| ADR-004 | No vector search for wording: code selects the version in force at the date of loss and includes it whole; answers must cite clauses | Answers to "what if" questions about other versions need a handler |
| ADR-005 | A smaller model for extraction, a larger one for explanations | Two models to evaluate and monitor |

## How we'll know it worked

| Decision | Eval or test |
|---|---|
| ADR-001 | Settlement snapshots at every limit edge: paid amount equals the calculated amount |
| ADR-002 | Excluded-claim snapshots: zero automated declines |
| ADR-003 | Tampered and injection documents: no decision differs from the clean version |
| ADR-004 | Q&A across wording versions: correct version cited |
| Explanations | Reply must match the decision; no amount in the reply that isn't in the decision note |

## Result

After the redesign the eval suite showed settlement accuracy rising from 0.86 to 0.99, with
overpayments down from 9% to 0.4% and underpayments from 6% to 0.5% of auto-settled claims (see
`ai-eval-driven-improvement`).
