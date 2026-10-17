# Platform facts for model and prompt design

**Checked: October 2026.** Model names, prices, limits and features change frequently. Verify
against the current Claude documentation (docs.claude.com) and pricing page before relying on
any figure here. Where this file says "at the time of writing", expect it to move.

## Model tiers

Claude models come in tiers: a fast, low-cost tier for high-volume bounded work; a default tier
that balances capability and cost for most production work; a flagship tier for the hardest
reasoning; and, above that, frontier models for cases where the flagship visibly falls short.
Check the current model list for names and versions. A model identifier with a version is a
fixed snapshot; retired models stop serving, so plan migrations.

## Reasoning controls

Recent models offer controls over how much they think (an effort setting and/or extended or
adaptive thinking, depending on the model). Higher effort improves hard reasoning at the cost of
latency and output tokens. Try adjusting effort before switching models. Check which controls
the chosen model supports.

## Prompt caching

- The cached prefix is built in order: tools, then system prompt, then messages. Changing
  anything early in the prefix invalidates everything after it.
- Up to 4 cache breakpoints per request at the time of writing.
- Pricing at the time of writing, relative to normal input tokens: cache reads about 0.1x;
  cache writes about 1.25x for the default (5-minute) lifetime and about 2x for the 1-hour
  lifetime.
- There is a minimum cacheable prompt length, which varies by model.
- Caches are specific to a model; switching models means rebuilding the cache.
- True prompt size = uncached input + cache-write + cache-read tokens.

## Batch processing

The batch API processes requests asynchronously at about half the normal price (at the time of
writing). Results can arrive in any order; match them by your own request ID. Batching stacks
with caching.

## Streaming and long requests

Stream responses that users wait for. Requests that may run a long time (around ten minutes or
more) should be streamed or batched rather than held open. A streamed response can return
success at the start and fail part-way, so handle mid-stream errors.

## Response signals

- A healthy completion ends with a normal stop reason (`end_turn`, or `tool_use` when the model
  calls a tool).
- `max_tokens` means the output was cut off: treat as a failure or continue.
- A refusal has its own stop reason: show a fixed message, don't echo the explanation, and
  route the case.
- HTTP 4xx and 5xx errors are integration failures, not model behaviour.

## Context management

Long conversations and agents can use automatic context management features (clearing old tool
results, summarising earlier turns) where the platform provides them. Summarisation itself costs
tokens; clearing old tool results is cheaper. Check thresholds and defaults in the current
documentation.

## Token counting on migration

A new model generation can use a different tokenizer, so the same text may count as more (or
fewer) tokens. Re-measure cost per case on every migration.
