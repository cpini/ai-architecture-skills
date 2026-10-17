---
name: ai-retrieval-design
description: Design how an AI system finds the knowledge it answers from - first deciding whether retrieval is needed at all (or whether filtering plus putting the whole source in context with caching is simpler), then the corpus and its owners, chunking with added context, hybrid dense and keyword search with rank fusion, reranking, permission and version filters, citations, refusal when nothing relevant is found, and evals that measure retrieval and answers separately. Use when building or fixing a knowledge assistant, RAG pipeline, document Q&A or search feature, when answers cite the wrong or outdated source, or when someone proposes a vector database. Works in any domain.
---

# AI retrieval design

Most bad answers from a knowledge assistant are retrieval failures: the right passage was never
in front of the model. So retrieval is designed and measured as its own system, separately from
the answer. And the first design question is whether you need retrieval at all: if the relevant
material fits comfortably in the context window, putting it all in (with caching) is simpler,
cheaper to build and usually more accurate.

## Inputs

The use-case card (what questions, who asks, how wrong is too wrong), the corpus (what exists,
who owns it, how it changes), permission rules, and a sample of real questions with the answers
an expert would give.

## Process

1. **Decide whether to retrieve.** Work out how much text a single question could need:
   - **Small enough to include whole** (as a rule of thumb, below roughly 200,000 tokens, or a
     few hundred pages, for the material any one question could touch; check current limits in
     `references/retrieval-facts.md`): filter to the relevant documents in code (by customer,
     product, version, permission) and include them whole, with prompt caching. No index.
   - **Too large**: retrieval. Often both apply: filter in code first, then retrieve within the
     filtered set.
   - The answer is already in a structured system (a price, a status, a balance): use a lookup
     tool, not retrieval.

2. **Profile the corpus.** For each source: owner, format, size, update frequency, versions,
   permissions, quality problems (duplicates, outdated pages, scanned images). Assign an owner
   for keeping it current; stale sources are the commonest cause of wrong answers that look
   right.

3. **Design the filters in code.** Before any similarity search, restrict candidates by the
   rules that must always hold: the user's permissions, tenant, product, effective date,
   document version, language. A filter applied by the model is not a filter.

4. **Choose the embedding model first, then chunking.** The embedder often matters more than
   chunk size. Then chunk along the document's structure (sections, clauses, questions), keep
   headings with their text, and **add context to each chunk** at indexing time: a short note
   saying which document and section it comes from and what it is about. Chunks without
   context ("The limit is £500") retrieve badly and mislead the model.

5. **Use hybrid search.** Combine dense (embedding) search with keyword search (BM25 or
   similar), which catches exact terms, codes and names that embeddings miss. Merge the result
   lists with reciprocal rank fusion. Then **rerank** a wider candidate set (around a hundred or
   more) down to the few passages the model will see, with a reranking model.

6. **Design the answer step.**
   - Passages are fenced as data, each with its source reference.
   - The model answers only from the passages and cites the reference for each claim.
   - If the passages don't answer the question, the model says so and offers a route (a
     person, a form), rather than answering from general knowledge.
   - Where exact wording matters (contracts, policies), quote the passage, then explain.

7. **Measure retrieval and answers separately.**
   - **Retrieval**: recall@k (is a correct passage in the top k?) on a labelled question set,
     measured for each search method alone and combined, so you know which part helps.
   - **Answers**: correctness, groundedness (every claim supported by a cited passage),
     citation accuracy, correct refusal on unanswerable questions.
   - Include unanswerable, ambiguous, out-of-scope and adversarial questions, and documents
     containing instructions (indirect injection).
   Hand these to `ai-eval-suite-design`.

8. **Plan freshness and monitoring.** How documents get re-indexed when they change, how
   removed documents disappear from the index, and how you'll detect answers drawn from stale
   sources. Sample live questions and re-score them with the offline graders.

## Adapting by system type

| System type | Retrieval role | Watch out for |
|---|---|---|
| Conversational agent with actions | Policy and product knowledge behind replies | Retrieved text treated as instructions; answers contradicting the action taken |
| Knowledge assistant (retrieval) | The core of the system | Everything in this skill |
| Document extraction | Rarely needed; sometimes reference data (supplier lists, codes) | Use a lookup, not similarity search, for reference data |
| Classification and routing | Similar past examples as few-shot context | Leaking labels of the item being classified |
| Content generation | Facts and approved claims to write from | Generating beyond the retrieved facts |
| Autonomous tool-using agent | Search over code, tickets or docs as a tool | Too much text returned per call; permission scope |
| Decision support | Similar past cases and their outcomes | Biased precedent; outdated cases |

## Output

A retrieval design using `templates/retrieval-design.md`: the retrieve-or-not decision with
sizes, corpus profile and owners, filters, indexing and chunking, search and reranking, answer
rules, eval metrics with targets, and the freshness plan.

## Quality bar

- The retrieve-or-not decision is made with numbers.
- Every rule that must always hold (permissions, version, tenant) is a filter in code.
- Chunks carry context; search is hybrid with reranking, or there's a written reason not.
- Answers cite sources and refuse when the sources don't answer.
- Recall is measured for each search method and combined; answers are measured separately.
- Every source has an owner and a re-indexing plan.

## Reference

- `references/retrieval-facts.md`: dated figures and techniques (contextual retrieval results,
  rank fusion formula, context limits). Check current documentation before relying on them.
- `references/worked-example.md`: the fictional Larkspur insurer, where policy wording needed
  no retrieval at all but the claims manual and past decisions did.
