# Retrieval facts and techniques

**Checked: October 2026.** Figures come from published research and platform documentation at
that date. Context limits, prices and model capabilities change: verify against current
documentation (docs.claude.com) before relying on them.

## When to skip retrieval

- If the material a question could need is below roughly 200,000 tokens (about 500 pages), it is
  usually better to include it in the prompt and use prompt caching than to build a retrieval
  pipeline. Check the current context window of the model you use.
- Prompt caching makes repeated large prefixes cheap: cache reads cost a fraction of normal
  input tokens (about a tenth at the time of writing), and cache writes cost somewhat more than
  normal input. See `ai-model-and-prompt-design`.

## Contextual retrieval

Anthropic's published "Contextual Retrieval" results (2024): prepending a short, generated
context (roughly 50–100 tokens explaining where the chunk sits in its document) to each chunk
before embedding and keyword indexing reduced the top-20 retrieval failure rate (1 − recall@20)
by about:

| Technique | Reduction in retrieval failures |
|---|---|
| Contextual embeddings | ~35% |
| Contextual embeddings + contextual BM25 | ~49% |
| Both, plus a reranker | ~67% |

Generating the context for every chunk is a one-off indexing cost, which caching the source
document makes much cheaper.

## Reciprocal rank fusion (RRF)

Merges ranked lists from different search methods. Each document scores, for every list it
appears in:

  score = Σ 1 / (k + rank), with k usually 60

Example: rank 2 in dense search and rank 5 in keyword search scores 1/62 + 1/65 ≈ 0.032; rank 1
in dense search only scores 1/61 ≈ 0.016. Documents found by both methods rise.

## Reranking

Retrieve a wide candidate set (often around 150) with fast search, then use a reranking model
(a cross-encoder, or a model scoring each candidate against the question) to pick the top ~20 or
fewer for the prompt. Rerankers add latency and cost per query; measure the gain.

## Measurement

- **Recall@k**: share of questions where at least one correct passage is in the top k.
- Measure each search method alone and fused, so you know what each contributes.
- Answer metrics (correctness, groundedness, citation accuracy, refusal) are separate from
  retrieval metrics: a good answer from bad retrieval is luck.
