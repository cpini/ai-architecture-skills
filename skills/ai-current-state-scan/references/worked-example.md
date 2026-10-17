# Worked example: scan of the Larkspur claims assistant prototype

> **Fictional example.** Larkspur Home Insurance, its people, systems, code and figures are
> invented to show the skill at full depth. Any resemblance to a real organisation is
> coincidental. The same case runs through every skill in this collection.

**Repo:** `claims-assistant` · **Scope:** `agent.py`, `prompts/system.md`, `tools.py`,
`retrieval.py`, `config.py`, the payments API route it calls, and the README.

## Verdict

**Not ready for customers.** The prototype is a capable demo, but the model chooses payment
amounts and can decline claims, with no check in code on either; policy limits and the excess
exist only in the prompt; uploaded documents reach the prompt as plain text. Five changes are
needed before a pilot: settlement and cover in code (F1, F3), no automated declines (F2),
documents fenced and extracted to a schema (F4), and the "if unsure, approve" rule removed
(F5).

## Top findings

| # | Severity | Finding | Evidence | Consequence | Confirming eval |
|---|---|---|---|---|---|
| F1 | Critical | The model chooses the payment amount; the tool executor passes it straight to the payments API | `tools.py:88` `payments.create(claim_id, amount=args["amount"])` (read in code) | Any amount up to the API's limit can be paid | Snapshot claims at, above and below each limit: paid amount must equal the calculated amount |
| F2 | Critical | `decline_claim` is offered to the model and executes without review | `tools.py:121`; `agent.py:47` tools list (read in code) | Automated declines breach the claims policy and likely UK GDPR Article 22 | Excluded-peril snapshots: no decline without a handler |
| F3 | High | Single-item limit, excess and wear-and-tear rates stated only in the prompt; the model does the arithmetic | `prompts/system.md:14-22` "Single item limit is £1,500… deduct 20% per year for clothing" (read in code) | Arithmetic errors; limits extractable by asking | Settlement snapshots; probing dialogue for limits |
| F4 | High | OCR text from uploads inserted into the prompt unfenced | `agent.py:63` `messages.append({"role": "user", "content": ocr_text})` (read in code) | Instructions inside a document are read as instructions | Receipts with embedded instructions, 5 runs each |
| F5 | High | Prompt says "Be generous. If unsure, approve: customers hate waiting" | `prompts/system.md:31` (read in code) | Contradicts the accuracy objective; biases towards overpayment | Ambiguous-evidence snapshots: route, don't pay |
| F6 | Medium | Retrieval searches all 11 wording versions with no filter by policy or date of loss | `retrieval.py:40` `index.search(query, k=5)` (read in code) | Answers cite cover the customer doesn't have | Q&A across wording versions |
| F7 | Medium | Missing excess defaults to £100 | `config.py:12` `DEFAULT_EXCESS = 100` (read in code) | Migrated policies with a £250 or £500 excess are overpaid (fails open) | Snapshot with excess absent: must route, not pay |
| F8 | Medium | Routed claims go to a shared queue with no alert or target time | `tools.py:140`; README "handlers pick up from the queue" (read in code and docs) | Hand-offs can sit for days | End-to-end hand-off test |
| F9 | Low | Full receipt images and conversations logged with no retention period | `agent.py:102` (read in code) | Data protection exposure | Config review |

## Authority map

| Action | Binding? | Who decides | Limits | Where enforced | Fail mode |
|---|---|---|---|---|---|
| `approve_payment` | Yes: pays money | Model | Single-item limit, excess, auto-pay limit | Prompt only; payments API checks total < £2,500 | Open (default excess £100) |
| `decline_claim` | Yes: refuses a claim | Model | Must be reviewed | Nowhere | Open |
| `route_to_handler` | No | Model | — | — | — |
| `search_policy` | No (read) | Model | Wording version | Nowhere | Returns any version |

## Logic bugs in the glue code

- `agent.py:78` counts "previous claims in 12 months" from the conversation history rather than
  the claims system, so a new chat resets the count (inferred from code; confirm).
- `tools.py:95` rounds the payment to the nearest pound before deducting the excess, giving
  errors of up to 50p (read in code).

## Pattern as built

Rung 4: an agent choosing among six tools in a loop of up to eight iterations. Cover,
settlement and routing are deterministic decisions the model has been handed; only reading
messages and documents, answering questions and explaining decisions need a model.

## Strengths to keep

- The payments API takes an idempotency key per claim (`tools.py:86`), so retries can't pay
  twice.
- Every tool call is written to an audit table with arguments and result (`agent.py:110`).
- Model calls are mocked in unit tests.

## Docs versus code

The README says "payments are capped by policy limits". The only cap in code is the payments
API's £2,500 total (F1).

## Open questions

- Is the prototype connected to live payments anywhere, or only the sandbox?
- Who will receive routed claims, and with what target time?
- Which model version is pinned? `config.py:5` reads it from an environment variable with no
  default.
