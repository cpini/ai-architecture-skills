# Worked example: Larkspur handler copilot and claims MCP server

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

After the claims assistant's pilot, Larkspur started a second product: a copilot inside the
handler workbench that answers questions about a claim, summarises evidence, drafts letters and
adds notes. The customer-facing assistant (a workflow with no binding tools) and the
engineers' coding tools also needed claims data. A first prototype wrapped the claims API
automatically: **23 tools**, one per endpoint.

## What the prototype showed

| Symptom | Evidence |
|---|---|
| Wrong-tool calls | 14% of tasks: `get_claim_v2` vs `get_claim_summary` vs `get_claim_detail`, three tools with overlapping descriptions |
| Wrong-parameter calls | 9%: the policy number passed where the claim ID was expected (both strings) |
| Token overhead | 23 definitions ≈ 11,000 input tokens on every turn, before any conversation |
| Long returns | `get_claim_detail` returned 180 fields (≈ 6,000 tokens) to answer "what's the excess?" |
| A write the model shouldn't have | `update_reserve` was exposed because the endpoint existed |

## Mechanism choice

| Capability | Mechanism | Why |
|---|---|---|
| Claims and policy data for the copilot, the customer assistant and engineers' coding tools | **MCP server** (remote, Streamable HTTP) | Three hosts; one place for authorisation, audit and the tool contract |
| Drafting letters in Larkspur's house style | **Skill** | Procedure and templates, no live connection needed |
| Calculations (settlement, excess) | **Custom call to the existing engine**, inside the MCP server | Decisions stay in code, not in the model |
| Web search | Not provided | No task needs it; reduces injection surface |

## Tool set after redesign (7 tools)

| Tool | Effect | Replaces | Notes |
|---|---|---|---|
| `claims_get_case(case_id, view)` | Read | 7 tools | `view` is an enum: `summary`, `parties`, `items`, `payments`, `history`. Summary is ~400 tokens |
| `claims_search(customer_ref?, policy_number?, status?)` | Read | 3 tools | Returns at most 10 case summaries with IDs |
| `policy_get_clause(policy_number, topic)` | Read | 3 tools | Returns the clause text and reference from the wording version in force at the loss date |
| `claims_get_document(case_id, document_id, part)` | Read | 2 tools | Returns extracted fields with their source locations, or one page's text, fenced as untrusted |
| `claims_calculate_settlement(case_id)` | Read (calculation) | — | Calls the engine; the model never calculates |
| `claims_add_note(case_id, text)` | Write | 1 tool | Idempotency key; author recorded as "copilot, on behalf of [handler]" |
| `letters_create_draft(case_id, template, fields)` | Write (draft only) | 2 tools | Creates a draft the handler must send; never sends |

Removed (5): `update_reserve`, `close_case`, `reopen_case`, `issue_payment` and
`update_contact`. They stay handler actions in the workbench. Together with the 18 tools
consolidated above, that accounts for all 23.

**Description example**, as the model sees it:

> `claims_get_case`: Get information about one claim. Use `view: summary` first; it answers most
> questions (status, cause, amounts, excess, next action). Use other views only when the summary
> doesn't have what you need. Don't use this to search for claims; use `claims_search`. Returns
> labelled fields; amounts are in GBP.

## Execution rules

- Read tools run in parallel (a summary and a clause lookup together).
- `claims_add_note` and `letters_create_draft` run one at a time, after any reads, at most one
  of each per user request.
- Cap: 8 tool calls per request.

## Authorisation

| Component | Design |
|---|---|
| Copilot (host) | Handler signs in; host obtains a token for the claims MCP server through OAuth 2.1 with PKCE |
| MCP server | Validates the token's audience; reads the handler's team and permissions; calls the claims API with its own service credentials, filtered to cases the handler may see |
| Customer assistant | Separate client, read-only scopes, restricted to the customer's own case |
| Engineers' coding tools | Project-scoped server configuration, test environment only; production access denied by rule |
| Secrets | In the vault; never in prompts, configuration files or the repo |

No client token is passed to the claims API, so a token stolen from one host can't be replayed
against it.

## Observability

Every call is logged with the tool, the case ID, the result size, latency and error, under the
copilot's session ID and the handler's ID, and exported to Larkspur's tracing platform. A
dashboard tracks calls per task and wrong-tool rate from the eval runs.

## Results on the copilot eval set (120 tasks)

| Metric | 23 tools | 7 tools |
|---|---|---|
| Wrong-tool calls | 14% | 1.5% |
| Wrong-parameter calls | 9% | 2% |
| Tool definition tokens per turn | ≈ 11,000 | ≈ 2,100 |
| Median tokens per task | ≈ 24,000 | ≈ 7,500 |
| Task success | 78% | 93% |

The remaining wrong-parameter calls were policy numbers typed with spaces; the server now
normalises them.
