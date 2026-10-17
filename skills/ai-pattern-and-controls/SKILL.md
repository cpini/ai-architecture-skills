---
name: ai-pattern-and-controls
description: Design the target architecture for an AI feature. Splits the task into model work and deterministic decisions, picks the lowest rung on the escalation ladder (prompt, augmented model, workflow, agent) that does the job, assigns authority for every action that changes the world to code or model, decides what the model may see, and records the choices as ADRs. Use after discovery (a use-case card and/or current-state scan) and before build, when choosing between "agent" and "workflow", or when a review finds the model holding authority it shouldn't. Applies to any domain and any AI system type, from a single prompt to an autonomous agent.
---

# AI pattern and controls

The design principle: **the model decides what to say; code decides what is agreed.** Models
are good at open-ended language and judgement and unreliable at arithmetic, state tracking and
following hard rules every time. A good design gives each part of the task to whichever is
reliable for it, and gives the model the least authority and the least information it needs.

## Inputs

- The use-case card from `ai-use-case-framing` (exam question, error tolerances, task
  feasibility split).
- The triage note from `ai-feasibility-risk-triage`, if one exists (risk tier per action and
  the conditions the design must meet).
- The scan report from `ai-current-state-scan`, if something is already built (authority map,
  findings).
- The decision pack from `ai-options-and-tradeoff-analysis`, if options were compared.
- Constraints: runtime, vendors, latency and cost targets.

## Process

1. **Decompose the task into steps**, and label each one:
   - **Language**: understanding open-ended input, writing replies, summarising, classifying
     intent where the categories are fuzzy. Needs a model.
   - **Decision**: choosing an action or a value from known inputs with rules (amounts, limits,
     eligibility, routing on structured data). Belongs in code, even if a model *could* do it.
   - **Retrieval**: finding the facts the model needs. Code, sometimes with a model for ranking.
   For each decision step, write the rule in one or two sentences. If you can write the rule,
   it is code.

2. **Choose the pattern: the lowest rung that works.**
   1. Single prompt: one call, no tools.
   2. Augmented model: one call with retrieval or read-only tools.
   3. Fixed workflow: code routes between steps; the model handles the language steps.
   4. Agent: the model chooses its own tools and sequence in a loop.
   Go up a rung only when the task genuinely needs it: the path can't be known in advance and
   the cost of a wrong step is low. Binding actions with money, rights or other people's data
   almost never justify rung 4. Record why each lower rung was rejected.

3. **Pick the workflow pattern and decide on agents.** At rung 3, name the pattern (or the
   combination) that fits:
   - **Prompt chaining**: fixed sequence of steps, each checkable (extract → validate →
     explain).
   - **Routing**: a classifier (often a small, cheap model or plain rules) sends each input to
     a specialised path.
   - **Parallelisation**: independent subtasks run at once, either sectioning (different
     parts) or voting (same task several times for confidence). The subtasks are fixed in code.
   - **Orchestrator-workers**: a model decides the subtasks at runtime and delegates them. Use
     only when the subtasks can't be known in advance.
   - **Evaluator-optimizer**: one call drafts, another critiques against explicit criteria,
     in a loop. Needs criteria clear enough to judge.
   **Decompose at the coarsest split that works**: split a task only when each part runs
   reliably on its own, can be checked on its own, and keeping it merged would lose
   something (accuracy, parallelism, a different model tier). Default to fewer calls.
   **Multiple agents** cost far more tokens than a single call (an agent typically uses
   several times the tokens of a chat; a multi-agent system an order of magnitude more) and
   add four failure modes: spawning too many subagents, vague delegation, waiting on the
   slowest agent, and errors compounding across hand-offs. Subagents inherit no context, so
   every delegation needs a complete brief. Use multiple agents only for broad, parallel,
   read-heavy work (research across many sources) where the value justifies the cost.

4. **Design authority for every action that changes the world** (writes, sends, spends,
   commits, deletes). For each, decide:
   - **Who decides**: code (preferred for anything binding) or model.
   - **Limits** and **where they are enforced**: in code at the point of action, and again
     downstream where possible. A limit stated only in a prompt is not a control.
   - **Fail mode**: if config is missing or a check errors, fail closed (refuse and hand off).
   - **Cap**: at most one binding action per turn; step and token budgets per request.
   If the model must still call a binding tool, the tool executor re-checks the decision in
   code and refuses anything outside policy.

5. **Decide what the model sees.** The minimum context to do its language job:
   - No secrets, thresholds or limits it doesn't need. If it never sees the limit, it can't
     reveal it, and an injected instruction has nothing to override.
   - The decision, not the rule: "approved £412.50 after the £250 excess", not "the single-item
     limit is £1,500, deduct the excess and 20% wear and tear".
   - Untrusted input fenced as data, with platform notes in a format users can't imitate.
   - Only the tools it needs. If code makes the decision, take the decision tools away from
     the model entirely.

6. **Score the result**: risk tier (impact, reach, regulation, reversibility) and autonomy tier.
   Where a limit is enforced changes the tier: the same feature is Tier 4 with limits in the
   prompt and Tier 3 with them in code.

7. **Sketch the target architecture** as containers: what's new, what's kept, what each owns.
   Check it covers all four stages every request passes through: **input handling**
   (validate, normalise, enrich), **processing** (model calls with retrieval and tools),
   **output delivery** (format, validate in code, deliver) and **feedback** (capture outcomes
   and route them to evals, prompts or data). The three common gaps fail silently: skipped
   input validation, unchecked output, and no feedback loop. Logging alone is not feedback;
   the loop closes when a production outcome becomes an eval case or a change that is verified.
   Keep the existing shape where it works; list the changes in priority order with one line
   each on why.

8. **Write the ADRs.** One per significant decision, with the downside stated honestly. Use
   `templates/adr.md`. Typical set: who owns the decision (policy in code), what the model
   sees, how concurrency and idempotency work, disclosure, model tier.

9. **Define how you'll know it worked.** For each design decision, the eval or test that proves
   it (hand this to `ai-eval-suite-design`). A design that can't be tested isn't finished.

## Adapting by system type

| System type | Usual target rung | What code should own | What the model keeps |
|---|---|---|---|
| Conversational agent with actions | 3 (workflow) | Eligibility, amounts, limits, state, which action runs | Understanding requests, writing replies from the decision |
| Knowledge assistant (retrieval) | 2 (augmented) | Which documents are eligible (version, permissions, tenant), citation format, refusal when nothing relevant is found | Composing the answer from retrieved passages |
| Document extraction | 2–3 | Schema validation, cross-field checks (totals add up), confidence thresholds, posting | Reading messy documents into the schema |
| Classification and routing | 1–3 | Thresholds, the fallback route, hard rules that override the model (legal keywords, VIP accounts) | Classifying fuzzy text |
| Content generation | 1–3 | Facts injected from source data, banned and regulated claims, approval before publishing | Drafting and editing language |
| Autonomous tool-using agent | 4, bounded | Permissions, sandbox, step and cost budgets, approval before irreversible actions | Planning and choosing the next step |
| Decision support | 2–3 | The score or calculation, its explanation inputs, logging of the human's decision | Explaining the recommendation in plain language |

## Output

A design note using `templates/design-note.md`: task decomposition, pattern and workflow
pattern with rejected alternatives, authority table, model context, target architecture, ADRs and the tests
that prove each decision. Then hand the failure modes to `ai-guardrail-design`, the tools to
`ai-tool-and-mcp-design`, knowledge needs to `ai-retrieval-design` and each model call to
`ai-model-and-prompt-design`.

## Quality bar

- Every decision step is in code unless there is a written reason it can't be.
- Every binding action has a code-enforced limit and a fail-closed default.
- The model's context contains no value it must keep secret.
- The pattern is the lowest rung that works, with lower rungs explicitly rejected.
- Any split into multiple calls or agents passes the three decomposition tests.
- The architecture covers input handling, processing, output delivery and feedback.
- Each ADR names its downside.
- Each decision has a test or eval that proves it.

## Reference

`references/worked-example.md`: a fictional insurer's claims assistant redesigned from an
agent (rung 4) to a workflow (rung 3), with cover and settlement decided in code and the model
writing explanations, plus the ADRs and the eval evidence afterwards.
