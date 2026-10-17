# AI architecture Skills for Claude

Fifteen Claude skills that take an AI use case from first conversation to production
operation: framing the outcome, interviewing stakeholders, triaging feasibility and risk,
scanning what's already built, comparing options, choosing the pattern and who decides what,
designing tools, retrieval, prompts and guardrails, building an eval suite and improving the
system against it, getting ready for go-live, turning incidents into regression tests, and
keeping a living register and RAID log.

They are domain-neutral and work for any kind of AI system:

| System type | Examples |
|---|---|
| Conversational agent with actions | Refunds, bookings, account changes, claims |
| Knowledge assistant (retrieval) | Policy and HR Q&A, support knowledge bases, research over documents |
| Document extraction | Invoices, claims evidence, contracts, forms |
| Classification and routing | Ticket triage, email routing, risk flags |
| Content generation | Product copy, reports, marketing, correspondence |
| Autonomous tool-using agent | Coding agents, IT operations, research agents |
| Decision support | Credit, pricing, clinical or operational recommendations a person decides on |

Each skill includes a table showing what changes for each system type.

They share one design principle: **the model decides what to say; code decides what is
agreed.** Language and judgement go to the model. Amounts, limits, rules and anything
unacceptable even once go to code, and evals prove the split holds.

Every skill has:

- `SKILL.md`: when to use it, the process step by step, how it adapts by system type, the
  output, and a quality bar;
- `references/`: a complete worked example, plus checklists or question banks where useful;
- `templates/`: the output format, ready to fill in.

## The skills

| Phase | Skill | What it produces |
|---|---|---|
| Discovery | [`ai-use-case-framing`](skills/ai-use-case-framing) | Use-case card: stakeholders, requirements, AI qualifiers, value pillar and guard metric, and the measurable "exam question" |
| Discovery | [`ai-discovery-interview-kit`](skills/ai-discovery-interview-kit) | Interview guides, a question bank, a boundary walk that finds undecided policy questions, and a synthesis that updates the card |
| Discovery | [`ai-feasibility-risk-triage`](skills/ai-feasibility-risk-triage) | Triage note: steps labelled model or code, feasibility ratings, risk tier per action, regulatory screen, cheapest experiments, verdict with conditions |
| Discovery | [`ai-current-state-scan`](skills/ai-current-state-scan) | Scan report for an existing AI feature: verdict, authority map, findings cited to `path:line`, and the eval that confirms each |
| Design | [`ai-options-and-tradeoff-analysis`](skills/ai-options-and-tradeoff-analysis) | Decision pack: requirements with IDs, criteria locked before scoring, weighted matrix with evidence, scenario analysis (risks, sensitivity and trade-off points), ADR, one-page executive summary |
| Design | [`ai-pattern-and-controls`](skills/ai-pattern-and-controls) | Design note: task split into model and code work, pattern on the escalation ladder, workflow pattern and decomposition, authority for every binding action, what the model sees, ADRs |
| Design | [`ai-tool-and-mcp-design`](skills/ai-tool-and-mcp-design) | Tool catalogue: mechanism per capability (built-in, custom, MCP, Skill), tool contracts, bloat fixes, execution rules, authorisation, observability |
| Design | [`ai-retrieval-design`](skills/ai-retrieval-design) | Retrieval design: whether to retrieve at all, corpus and owners, filters in code, contextual chunks, hybrid search and reranking, answer rules, recall and answer metrics |
| Design | [`ai-model-and-prompt-design`](skills/ai-model-and-prompt-design) | Model and prompt specification: tier and effort per call, prompt layout, output contracts, context and caching plan, cost and latency model, tuning in the right order |
| Design | [`ai-guardrail-design`](skills/ai-guardrail-design) | Guardrail plan: failure modes with a deterministic primary control each, defence in depth, gate patterns by action, a hand-off that can't dead-end |
| Evaluate | [`ai-eval-suite-design`](skills/ai-eval-suite-design) | Eval plan and harness: metrics with guard metrics, oracle, failure taxonomy, scenario matrix, grader proof, CI gate |
| Evaluate | [`ai-eval-driven-improvement`](skills/ai-eval-driven-improvement) | Fix log: triage of every failure, root cause traced to the line, one measured change at a time, prompt-or-code decisions |
| Operate | [`ai-slo-and-go-live-readiness`](skills/ai-slo-and-go-live-readiness) | Readiness pack: SLIs that count wrong outputs as failures, SLOs and error budgets, burn-rate alerts, production gate, pilot stages, hand-off pack, drift versus regression |
| Operate | [`ai-incident-to-scenario`](skills/ai-incident-to-scenario) | Post-incident note and new regression scenarios that fail on the faulty version and pass on the fix |
| Operate | [`ai-register-and-raid`](skills/ai-register-and-raid) | Register entry and RAID log with a status summary, kept in the repo and reviewed for expiring waivers, unowned risks and stale evals |

## How they chain

```
Discovery   use-case card ─► interview synthesis ─► triage note ─► current-state scan (if built)
                                                        │
Design      decision pack ─► design note + ADRs ─► tool catalogue · retrieval design ·
                                                    model and prompt spec · guardrail plan
                                                        │
Evaluate    eval plan ─► fix log
                                                        │
Operate     readiness pack ─► incident notes ─► register + RAID (updated throughout)
```

Each skill names the output it expects from the one before, and works on its own if that
output doesn't exist yet (it drafts what it needs and marks the gaps).

## The worked example

All fifteen examples follow one **fictional** case: a claims assistant for an invented UK insurer. The assistant reads customers' messages, photos and receipts, answers
cover questions from the policy wording, and settles small contents claims. One product
combines four system types (extraction, retrieval, routing and an agent that pays money), so
the examples show each skill handling more than one kind of system. The case follows a
prototype through discovery, an options decision, redesign, evaluation, a pilot incident, the
decision to widen the pilot, and a second product (a handler copilot with a shared MCP server).
The UK insurer, its people, systems and figures are invented; any resemblance to a real organisation is coincidental.

## Coverage of the Claude's Architectural domains

The skills were written to cover Claude's architectural domains and are real working tools.

| Domain | Skills |
|---|---|
| Solution design and architecture | `ai-use-case-framing`, `ai-feasibility-risk-triage`, `ai-current-state-scan`, `ai-options-and-tradeoff-analysis`, `ai-pattern-and-controls` |
| Claude models, prompting and context | `ai-model-and-prompt-design` |
| Integration | `ai-tool-and-mcp-design`, `ai-retrieval-design` |
| Evaluation, testing and optimisation | `ai-eval-suite-design`, `ai-eval-driven-improvement`, `ai-incident-to-scenario` (feedback loop), `ai-model-and-prompt-design` (cost and latency) |
| Governance, safety and risk | `ai-guardrail-design`, `ai-current-state-scan` (controls audit), `ai-feasibility-risk-triage` (regulatory screen), `ai-incident-to-scenario`, `ai-register-and-raid` |
| Stakeholder communication and lifecycle | `ai-discovery-interview-kit`, `ai-options-and-tradeoff-analysis`, `ai-slo-and-go-live-readiness`, `ai-register-and-raid` |
| Developer productivity | Partly: `ai-tool-and-mcp-design` covers MCP configuration for coding tools |

Platform-specific details (model tiers, caching, MCP authorisation, retrieval figures) live in
dated `references/*-facts.md` files so the skills themselves don't go stale. Check them against
current documentation.

## Installing

**Claude Code:** copy or symlink skill folders into `~/.claude/skills/` (for you) or a
project's `.claude/skills/` (shared with that repo):

```bash
git clone <this repo> && cd <repo>
mkdir -p ~/.claude/skills
cp -R skills/ai-* ~/.claude/skills/
```

**Claude apps:** add a skill from the skills settings by uploading its folder as a ZIP, or
ask Claude to propose it to your account.

Claude picks a skill up from its `description` when a request matches, or you can ask for one
by name ("use ai-feasibility-risk-triage on this card").

## About the author

Written by Cristian Pini, architect, engineer and technology leader.

<a href="https://www.credly.com/badges/5c9b3813-caf7-43e1-8fba-bbdf6b2afd4d"><img src="https://images.credly.com/images/7583afbc-4aab-4c7d-8af4-d9af72786ab3/linkedin_thumb_blob" alt="Claude Certified Architect – Professional badge" width="120"></a>

The certification is the author's; the skills are independent work and not endorsed by
Anthropic.

## Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
