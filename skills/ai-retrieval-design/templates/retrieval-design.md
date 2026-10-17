# Retrieval design: [system]

**Date:** [YYYY-MM-DD] · **Inputs:** [use-case card, question sample] · **Owner:** [name]

## Retrieve or not

| Question type | Material it could need | Size (tokens) | Decision (lookup / include whole with caching / retrieve) |
|---|---|---|---|

## Corpus

| Source | Owner | Format | Size | Updates | Versions | Permissions | Quality issues |
|---|---|---|---|---|---|---|---|

## Filters (applied in code before search)

| Filter | Source of truth | Why it must always hold |
|---|---|---|

## Indexing

- **Embedding model:** [ ] (why)
- **Chunking:** [structure-based / size; overlap]
- **Context added to each chunk:** [how generated; example]
- **Keyword index:** [BM25 / other]

## Search

- **Methods:** [dense, keyword]
- **Fusion:** [RRF, k = ]
- **Candidates → reranked → in prompt:** [ ] → [ ] → [ ]

## Answer rules

- Passages fenced as data with references: [format]
- Citation format: [ ]
- When nothing relevant is found: [response and route]
- Quoting rules: [ ]

## Evaluation

| Metric | Target | Dataset |
|---|---|---|
| Recall@[k], dense only | | |
| Recall@[k], keyword only | | |
| Recall@[k], fused + reranked | | |
| Answer correctness | | |
| Groundedness / citation accuracy | | |
| Correct refusal on unanswerable | | |

## Freshness and monitoring

- Re-indexing trigger: [ ]
- Removal of withdrawn documents: [ ]
- Live sampling and re-scoring: [ ]
