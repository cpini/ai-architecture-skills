# Worked example: retrieval at Larkspur

> **Fictional example.** Larkspur Home Insurance, its people, systems and figures are invented
> to show the skill at full depth. Any resemblance to a real organisation is coincidental. The
> same case runs through every skill in this collection.

Two Larkspur products needed knowledge: the customer-facing claims assistant (cover questions
from the policy wording) and the handler copilot (questions about claims handling practice).
The same skill gave opposite answers.

## Retrieve or not

| Question type | Material it could need | Size | Decision |
|---|---|---|---|
| Customer: "Am I covered for…?" | The customer's own policy wording, in the version in force at the loss date | One version ≈ 38 pages ≈ 24,000 tokens | **No retrieval.** Filter to the right version in code; include the whole wording; cache it |
| Customer: "What's my excess?" | One field on the policy record | — | **Lookup** from the policy system |
| Handler: "How do we handle a claim for items bought abroad?" | Claims manual (≈ 1,900 pages), technical bulletins (≈ 400), and 30,000 past decision notes | Several million tokens | **Retrieval**, filtered by product and date |

The prototype had embedded all eleven wording versions into one vector index and searched it
for every question. That was the source of answers citing cover the customer didn't have (scan
finding F6). The fix wasn't better retrieval; it was **no retrieval**: once the version is known,
the whole document fits. Caching meant the wording for each version was paid for in full only
once per cache period, not once per question.

## Handler copilot: corpus

| Source | Owner | Updates | Versions | Permissions | Quality issues |
|---|---|---|---|---|---|
| Claims manual | Claims technical team | Monthly | Superseded sections kept | All handlers | Scanned appendices; tables as images |
| Technical bulletins | Claims technical team | Weekly | Dated; some withdrawn | All handlers | Withdrawn bulletins still on the intranet |
| Decision notes | Handlers | Daily | — | Handler's own team, plus complex-claims team | Abbreviations; customer data in free text |

Withdrawn bulletins were the biggest risk: a third of "wrong" answers in the first prototype came
from bulletins that had been replaced.

## Filters (in code)

| Filter | Source of truth | Why |
|---|---|---|
| Status = current (excludes withdrawn and superseded) | Document metadata, maintained by the claims technical team | Stale guidance is the commonest wrong answer |
| Product line | Case record | Home and landlord guidance differ |
| Team permissions for decision notes | Identity provider | Notes contain customer details |
| Customer data redacted in decision-note chunks | Indexing pipeline | Data minimisation |

## Indexing and search

- **Embedding model chosen first**, comparing three on 200 labelled handler questions; the best
  one beat the others by 9 points of recall@20 before any chunking changes.
- **Chunking** by manual section and by bulletin, keeping headings; decision notes as one chunk
  each.
- **Context added to each chunk**, for example: "From the Claims Manual, section 7.3 'Items
  purchased overseas', current since March 2026. Covers currency conversion and proof of
  purchase for contents claims."
- **Hybrid search**: dense plus BM25 (BM25 finds bulletin numbers and clause references like
  "CB-2026-14" that embeddings miss), merged with RRF (k = 60).
- **Reranking**: 150 candidates → 12 passages in the prompt.

## Answer rules

- Passages fenced, each tagged with its reference and date.
- Every statement cites a reference, e.g. "[Manual 7.3]".
- If the passages don't cover it: "I can't find guidance on this in the manual or current
  bulletins. The complex-claims team can advise." No answers from general knowledge.
- For customer-facing wording questions, quote the clause, then explain it in plain English.

## Evaluation (handler question set: 200 labelled questions, 30 unanswerable)

| Configuration | Recall@20 |
|---|---|
| Dense only, plain chunks | 0.71 |
| Dense only, contextual chunks | 0.79 |
| Dense + BM25, contextual, RRF | 0.86 |
| … + reranker (top 12 in prompt) | 0.92 (recall@12) |

| Answer metric | Target | Result |
|---|---|---|
| Correctness (senior handler graded) | ≥ 90% | 91% |
| Every claim supported by a cited passage | ≥ 98% | 98.5% |
| Correct refusal on unanswerable questions | ≥ 90% | 93% |
| Answers citing withdrawn guidance | 0 | 0 |

For the customer assistant, the wording Q&A set measures correct-version citation (99%) and
correct refusal when the wording doesn't answer (96%); there is no retrieval metric because
there is no retrieval.

## Freshness

- Re-index on publish events from the document management system; withdrawn documents removed
  from the index the same day.
- Weekly job compares index contents with the document system and alerts on mismatches.
- 50 live handler questions a week re-scored with the offline graders.
