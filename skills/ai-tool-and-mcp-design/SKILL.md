---
name: ai-tool-and-mcp-design
description: Design how an AI system connects to other systems - the tool set a model sees, each tool's name, description, parameters and return shape, the choice between built-in tools, custom tools, MCP servers and Skills, authorisation and token handling, and which calls run in parallel or in sequence. Diagnoses tool bloat (wrong-tool and wrong-parameter calls, token overhead) and fixes it by consolidating, namespacing, trimming returns and deferring rare tools. Use when giving a model tools, wrapping an API for an agent, building or reviewing an MCP server, when an agent picks the wrong tool, or when tool definitions are eating the context window. Works in any domain.
---

# AI tool and MCP design

Tools are prompt. Every tool definition is loaded into the model's context on every turn, and
the model chooses between tools by reading their names and descriptions. A tool set designed
like an API reference (one tool per endpoint, every field returned) gives the model too many
choices and too much to read, and it picks wrong. A tool set designed for the task gives it a
few clear choices and only the data it needs.

Tool design is also authority design. A tool the model can call is an action the model can
take, so the rules from `ai-pattern-and-controls` apply: binding actions are decided by code,
and the tool executor re-checks every call.

## Inputs

The design note (task decomposition, authority table, what the model sees), the systems to
connect, who else needs the same integration, and the deployment surfaces (API, chat apps,
coding tools).

## Process

1. **List what the model needs to do, not what the APIs offer.** Start from the task steps in
   the design note. For each, write the question the model must answer or the action it must
   request. Only these become tools.

2. **Choose the integration mechanism** for each capability:
   - **Built-in tools** (web search, code execution, file handling): use when they fit; they
     can't reach private systems.
   - **Custom tools** defined in your application: right for a single application with its
     own logic.
   - **An MCP server**: worth its overhead when the same integration serves several hosts or
     teams (an assistant, an internal copilot, coding tools). It gives one place to own
     authorisation, logging and the tool contract.
   - **A Skill**: packages instructions, procedures and resources; not a live connection to
     a system.
   Record the choice and why in an ADR.

3. **Design each tool** with `templates/tool-catalogue.md`:
   - **Name**: a verb and an object, namespaced when there are several systems
     (`claims_get_case`, `policy_get_clause`).
   - **Description**: what it does, when to use it, when *not* to use it, and what it returns,
     in plain language. This is the text the model decides from.
   - **Parameters**: as few as possible, typed, with enums for closed sets, IDs rather than
     free text, and units and currency explicit.
   - **Return shape**: only the fields the model needs, in a compact, consistent format;
     human-readable labels instead of internal codes; paginated or truncated with a note when
     large. Errors returned as clear text the model can act on ("Case not found: check the
     case ID"), not stack traces.
   - **Effect**: read-only or state-changing; if state-changing, idempotency key and the
     code-side check that runs before it acts.

4. **Keep the set small.** Fix bloat with four moves, in this order:
   - **Consolidate**: one tool with an `action` or `view` parameter instead of many near-
     duplicates.
   - **Namespace**: prefixes that make the right tool obvious across systems.
   - **Return high-signal fields only**: trim responses to what the next decision needs.
   - **Defer rare tools**: load them on demand (a tool search or a second stage) rather than
     on every turn.
   Signs of bloat: the model calls the wrong tool or passes wrong parameters, tool definitions
   take a large share of input tokens, or transcripts show it reading long responses to find
   one field.

5. **Set execution rules.** Read-only calls can run in parallel; state-changing calls run one
   at a time, in order, each confirmed before the next. Cap steps and calls per request. Give
   every state-changing call an idempotency key.

6. **Design authorisation.** Follow `references/platform-facts.md` for the current protocol
   details. The principles:
   - **Least privilege**: each tool, server and service account can do only what the task
     needs; separate read and write scopes.
   - **The user's permissions, not the server's**: the integration enforces what the person on
     whose behalf it acts may see and do.
   - **Tokens bound to their audience**: a token issued for one server is rejected by others;
     never pass a client's token through to an upstream API (the confused-deputy problem).
   - **Secrets out of prompts and repos**: credentials come from a vault or the environment,
     never from the model's context.
   - **Deny beats allow**: explicit deny rules for dangerous operations.

7. **Make it observable.** Log every call with tool, parameters (redacted where needed),
   result size, latency, error and the request or session ID, so traces show the path through
   tools. Export through the organisation's standard telemetry.

8. **Test the tool set.** Scenarios that check the model picks the right tool and parameters
   for each task, ignores tools it shouldn't use, recovers from tool errors, and can't reach
   data outside the user's permissions. Track wrong-tool and wrong-parameter rates as metrics
   in the eval suite.

## Adapting by system type

| System type | Typical tools | Main design risk |
|---|---|---|
| Conversational agent with actions | Lookups plus a few state-changing actions | Binding tools without code checks; replies drifting from tool results |
| Knowledge assistant (retrieval) | Search, fetch document, maybe filters | Search tools returning too much text; permissions not applied to results |
| Document extraction | Usually none: a structured-output schema is the "tool" | Treating document content as instructions |
| Classification and routing | None, or one lookup for context | Over-engineering: a single call usually suffices |
| Content generation | Fact lookup from product or account data | The model inventing facts a lookup should supply |
| Autonomous tool-using agent | File, shell, browser, ticketing, deployment tools | Broad permissions; destructive actions; tool bloat across many systems |
| Decision support | Data lookups, calculators | Letting the model do calculations a tool should return |

## Output

A tool catalogue using `templates/tool-catalogue.md`: mechanism choice per capability, each
tool's contract, execution rules, authorisation design, observability and tests. ADRs for the
mechanism and authorisation choices.

## Quality bar

- Every tool maps to a task step; nothing exists because an endpoint did.
- Descriptions say when to use the tool, when not to, and what it returns.
- Returns are trimmed to what the model needs; errors are actionable text.
- State-changing tools are sequential, idempotent and re-checked in code.
- Authorisation is least-privilege, per user, audience-bound, with no token passthrough.
- Wrong-tool and wrong-parameter rates are measured.

## Reference

- `references/platform-facts.md`: dated protocol and platform details (MCP transports and
  authorisation, token handling, tool loading). Check the current documentation before relying
  on them.
- `references/worked-example.md`: the fictional Larkspur insurer's handler copilot and shared
  claims MCP server, cut from 23 auto-generated tools to 7.
