# Platform facts for tool and MCP design

**Checked: October 2026.** Protocols and platform behaviour change. Verify against the current
Model Context Protocol specification (modelcontextprotocol.io) and the Claude documentation
(docs.claude.com) before relying on any detail here.

## How tools reach the model

- Every tool definition (name, description, input schema) is sent with every request and counts
  as input tokens, alongside a system prompt the platform adds for tool use. Large tool sets
  can cost tens or hundreds of thousands of tokens per request.
- Tool definitions sit at the start of the cached prefix (tools, then system prompt, then
  messages), so changing a tool definition invalidates the cache for everything after it.
- Deferring rarely used tools (loading them only when searched for or needed) reduces the
  per-turn cost and the number of choices.
- The model can request several tool calls in one turn; your code decides whether to run them
  in parallel. Run read-only calls in parallel and state-changing calls in sequence.

## MCP transports and authorisation

- Two standard transports: **stdio** (a local process; credentials come from the environment,
  no OAuth) and **Streamable HTTP** (a remote server; authorisation based on OAuth 2.1).
- For remote servers, clients use the OAuth authorisation code flow with PKCE. Tokens are
  requested for a specific server (a resource indicator), sent in the `Authorization` header,
  never in the query string.
- Servers must validate that a token was issued for them (audience) and must not pass a
  client's token through to upstream APIs; they call upstream systems with their own
  credentials and enforce the user's permissions themselves.
- MCP servers can expose tools, resources and prompts. Tools are model-invoked; resources are
  data the host application chooses to include.

## Choosing a mechanism

| Mechanism | Use when | Limits |
|---|---|---|
| Built-in tools | The provider's tool fits (search, code execution, files) | Can't reach private systems |
| Custom tools | One application, its own logic | Re-implemented per host |
| MCP server | The same integration serves several hosts or teams | Overhead to build, secure and run |
| Skills | Packaging procedures, instructions and resources | Not a live connection to a system |

## Coding-tool configuration (if the integration is also used by developers)

- MCP servers can be configured per user, per project (committed and shared) or locally
  (private). Project-scoped servers still need each developer's approval on first use.
- Permission rules and hooks enforce; instructions in project context files only persuade.
