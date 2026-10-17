# Tool catalogue: [system]

**Date:** [YYYY-MM-DD] · **Inputs:** [design note] · **Hosts:** [applications, copilots, coding tools]

## Mechanism per capability

| Capability | Task step | Mechanism (built-in / custom / MCP / Skill) | Why |
|---|---|---|---|

## Tools

### `[namespace_verb_object]`

- **Description (as the model sees it):** [what it does; when to use; when not to; what it returns]
- **Parameters:**

  | Name | Type | Required | Notes (enum, units, format) |
  |---|---|---|---|

- **Returns:** [fields, format, size limit, pagination]
- **Errors:** [messages the model can act on]
- **Effect:** [read-only / state-changing] · **Idempotency:** [key] · **Code-side check:** [ ]
- **Deferred?** [loaded every turn / on demand]

## Execution rules

- Parallel: [read-only tools]
- Sequential: [state-changing tools]
- Caps: [steps per request, calls per tool]

## Authorisation

| Component | Identity | Scopes | Token handling | User permissions enforced where |
|---|---|---|---|---|

## Observability

[What is logged per call; trace and session IDs; where it's exported]

## Tests

| Test | Expectation | Metric |
|---|---|---|
| Right tool for each task | | Wrong-tool rate |
| Right parameters | | Wrong-parameter rate |
| Tool error recovery | | |
| Permission boundary | | Zero out-of-scope reads |

## Token budget

| Item | Tokens per request |
|---|---|
| Tool definitions (always loaded) | |
| Typical tool results per task | |
