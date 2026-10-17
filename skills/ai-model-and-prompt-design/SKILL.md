---
name: ai-model-and-prompt-design
description: Specify each model call in an AI system - which model tier and reasoning effort, what goes in the system prompt versus the user turn, how the prompt is layered, examples, structured output, context management for long sessions, a caching plan, and a cost and latency model - then tune cost and latency in the right order (route to cheaper tiers, then reduce reasoning, and trade accuracy away last; caching, batching and streaming are free wins). Use when writing or reviewing prompts for production, choosing a model, cutting cost or latency, planning a model migration, or when a system works in demos but is too slow or expensive at volume. Works in any domain.
---

# AI model and prompt design

A production system is usually several model calls, each with a different job. Each call gets
its own specification: the cheapest model that passes its evals, a prompt laid out so the model
knows its role, its rules and the output contract, and a context and caching plan that keeps
cost and latency predictable. Prompts are versioned like code and changed one at a time against
the eval suite.

Model names, prices and limits change often. This skill uses tiers (fast, default, flagship);
current models and figures are in `references/platform-facts.md`, which is dated and should be
checked against the provider's documentation.

## Inputs

The design note (which steps are model work), the guardrail plan (what the model must never
see), the eval suite (to compare choices), volumes and latency targets, and the cost budget.

## Process

1. **List the model calls.** One row per call: its job, input, output, volume per day, latency
   target and cost of an error. Calls with different jobs get different specifications even if
   they use the same model.

2. **Choose the tier and effort for each call.**
   - **Start cheap, then upgrade** for high-volume, well-bounded, latency-sensitive work
     (classification, extraction, routing): begin with the fast tier and move up only if evals
     fail.
   - **Start capable, then optimise** for ambiguous, multi-step or high-cost-of-error work
     (judgement, long reasoning, customer-facing explanations of consequential decisions):
     begin with the default or flagship tier, then try cheaper settings once the evals pass.
   - **Reasoning effort** (or extended thinking) is a dial within a tier: try changing effort
     before changing model.
   - Pin model versions; plan migrations (re-run the full eval suite and re-check token counts,
     since a new model generation may count the same text as more tokens).

3. **Lay out the prompt.**
   - **System prompt**: role, standing rules, the output contract, and anything identical for
     every request. **User turn**: the per-request data.
   - **Order**: role and task; context and why it matters; numbered steps or rules; then the
     output format. Wrap distinct inputs in clearly named tags (`<policy_wording>`,
     `<customer_message>`), and say which inputs are untrusted data.
   - **Examples**: zero-shot by default; when the output format or judgement is hard to
     describe, 3–5 diverse, consistent examples in their own tags.
   - **Structured output**: when code consumes the output, use a schema (a tool definition or
     structured-output feature) and validate it in code.
   - **Escape hatch**: tell the model what to output when it can't do the task ("insufficient
     evidence", "not covered by the wording") so it doesn't guess.
   - **No secrets, limits or rules the model doesn't need** (see `ai-pattern-and-controls`).

4. **Handle refusals and errors.** Detect a refusal from the response metadata (the stop
   reason), show a fixed message and route the case, never echo the model's explanation to the
   user. Treat truncation (output hit the maximum) as a failure, not an answer.

5. **Plan context for long sessions.** For conversations or agents that run long: trim or clear
   old tool results, summarise earlier turns (compaction) when near a threshold, keep durable
   facts in a structured state the code owns rather than in the transcript, and give subagents
   a complete brief because they inherit nothing.

6. **Plan caching.** Put stable content first so it can be cached: tools, then the system
   prompt, then long reference material, then the conversation, then the new input. Mark cache
   breakpoints after the stable parts. Choose the cache lifetime by how often the prefix is
   reused. Measure true prompt size as input plus cache-write plus cache-read tokens.

7. **Build the cost and latency model.** For each call: input tokens (cached and uncached),
   output tokens, calls per case, price per token, so you can state cost per case and per month
   at expected volume. Latency is driven mostly by output tokens and by reasoning; stream
   responses users wait for; use batch processing for work that can wait.

8. **Tune in the right order** when cost or latency misses its target:
   1. **Free wins first**: caching, batching non-urgent work, streaming, shorter outputs,
      trimming context and tool results. These don't change accuracy.
   2. **Route to a cheaper tier** for the calls or cases that don't need the bigger one (a
      cheap classifier sending easy cases to the fast tier).
   3. **Reduce reasoning effort** where evals show it isn't needed.
   4. **Trade accuracy** only last, and only with the owner's agreement against the
      exam question.
   Let the binding constraint decide: if one target can't move (a latency limit, a hard
   accuracy gate), optimise everything else around it.

9. **Version and test.** Every prompt and model setting has a version; every change goes
   through the eval suite, one change at a time, and the winner re-runs the full suite before
   release (see `ai-eval-driven-improvement`).

## Adapting by system type

| System type | Typical tier | Prompt focus | Cost and latency levers |
|---|---|---|---|
| Conversational agent with actions | Default for replies; fast for routing | Decision note in, reply out; reply must match the action | Stream replies; cache system prompt and reference data |
| Knowledge assistant (retrieval) | Default | Answer only from passages, cite, refuse | Cache large documents; fewer, better passages |
| Document extraction | Fast, upgrade on evals | Schema, examples of hard layouts, "unreadable" escape hatch | Batch backlogs; one call per document in parallel |
| Classification and routing | Fast | Label definitions, examples of borderline cases | Batch; cache instructions; very short outputs |
| Content generation | Default or flagship | Facts block, style guide, banned claims | Batch non-urgent drafts; cache style guide |
| Autonomous tool-using agent | Flagship for planning, cheaper for sub-steps | Task brief, tools, stopping rule | Budgets; clear old tool results; delegate sub-steps to cheaper tiers |
| Decision support | Default | Explain the score from its inputs | Cache case context; short explanations |

## Output

A model and prompt specification using `templates/model-prompt-spec.md`: one section per call
(tier, effort, prompt layout, examples, output contract, escape hatch, refusal handling),
the context and caching plan, the cost and latency model, and the tuning decisions with eval
evidence.

## Quality bar

- Every call has a tier chosen by evidence from the evals, not habit.
- System prompt holds standing rules; user turn holds per-request data; untrusted input is
  tagged.
- Every call consumed by code has a schema validated in code, and an escape hatch.
- Refusals and truncation are detected and handled.
- Cost per case and latency are modelled before launch and measured after.
- Cost cuts follow the order: free wins, routing, effort, accuracy last.

## Reference

- `references/platform-facts.md`: dated platform details (tiers, effort, caching prices and
  rules, batch, streaming, refusal signals). Check current documentation before relying on them.
- `references/worked-example.md`: the three model calls in the fictional Larkspur insurer's
  claims assistant, with prompts, caching plan, cost model and the tuning that cut cost per
  claim without touching accuracy.
