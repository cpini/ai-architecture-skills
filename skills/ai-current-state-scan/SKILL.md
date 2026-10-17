---
name: ai-current-state-scan
description: Scan an existing AI feature's code and documents during discovery and report its as-built architecture, where the model holds authority it shouldn't, which controls exist only in the prompt, and what is logged, tested and disclosed, with every finding cited to a file and line. Use when reviewing, auditing or inheriting an AI system (an LLM agent, chatbot, copilot or tool-using workflow), before redesigning one, or when asked "what's wrong with our AI feature" or "is this safe to ship". Works in any domain and for any kind of AI system: agents, chatbots, retrieval assistants, document pipelines, classifiers, generators and autonomous agents.
---

# AI current-state scan

Read an AI feature's code the way an architect would before signing it off. The question is
not "is the code tidy" but **"what can the model decide, and what stops it deciding wrongly?"**

The most valuable findings usually come from a handful of places: the system prompt, the tool
definitions, the code that executes tools, and the loop that calls the model (for a pipeline:
the prompt, the output schema and the code that acts on the output). A few hundred lines in a
handful of files typically reveal limits stated only in the prompt, a binding action whose
amount the model chooses with no check in code, untrusted documents pasted into the prompt,
a prompt rule that contradicts the stated objective, and an ordinary bug in the glue code.

## Ground rules

- **Read-only.** Don't run anything that calls a model, writes to a database or sends a
  message. Running the existing unit tests is fine if they are mocked.
- **Cite everything.** Every finding gives `path:line` and quotes the line or a short excerpt.
  A finding without evidence is a hypothesis: put it in open questions instead.
- **Say how you know.** Mark each finding *read in code*, *read in docs* or *inferred*. Docs
  describe intent; code describes behaviour. When they disagree, the code wins and the
  disagreement is itself a finding.
- **Trace downstream.** A missing check in the agent may exist in the backend API it calls.
  Before reporting "no check", follow the call to where the write happens.
- **Stay in scope.** This is a scan, not a fix. Recommend; don't change code unless asked.

## Process

1. **Orient (5 minutes).** Read the README, any architecture or capabilities docs, CLAUDE.md or
   AGENTS.md, and the dependency file. Note the runtime, the model provider and model ID, and
   how the feature is triggered (HTTP, event, queue, cron).

2. **Find the AI surface.** Search for the model SDK and prompts. Typical searches:
   - SDK calls: `anthropic`, `messages.create`, `openai`, `ChatCompletion`, `genai`,
     `generate_content`, `bedrock`, `langchain`, `llama_index`
   - Prompts: `system`, `prompt`, `persona`, `instructions`, `.md`/`.txt`/`.jinja` templates
   - Tools: `tools=`, `tool_choice`, `functions=`, `input_schema`, `@tool`
   - Loops: `for iteration`, `while`, `max_iterations`, `stop_reason`, `tool_use`
   List each model call with its file, model, tools offered and how its output is used.

3. **Work through the checklist** in `references/checklist.md`. The first three sections
   (authority, prompt contents, untrusted input) find most of the high-severity issues; do
   them in full. Do the rest proportionately.

4. **Build the authority map.** For every action the system can take that changes the world
   (writes, sends, spends, commits, deletes), record: who decides (model or code), what limits
   apply, **where each limit is enforced** (prompt, tool code, downstream API, database rule,
   nowhere) and whether it fails open or closed. A limit enforced only in the prompt is a
   probabilistic control on a deterministic requirement: report it.

5. **Look for logic bugs in the glue code.** Counters, thresholds, rounding, defaults when
   config is missing, state derived from message history, retry and idempotency paths. These
   are ordinary bugs with AI-sized consequences (a counter that includes the wrong events, or
   a missing config value that silently defaults to the most permissive setting).

6. **Assess the pattern as built.** Place it on the escalation ladder: (1) single prompt,
   (2) augmented model with retrieval or tools, (3) fixed workflow with model steps, (4) agent
   choosing its own tools in a loop. Note which parts of the task genuinely need the model and
   which are deterministic decisions it has been handed.

7. **Rate and rank findings.**
   - **Critical**: the model can take a binding or harmful action outside policy with no code
     control, or a zero-tolerance requirement is unenforced.
   - **High**: a control exists only in the prompt; confidential values are in the prompt; a
     compliance obligation is unmet.
   - **Medium**: logic bugs that degrade outcomes; missing logging, tests or monitoring.
   - **Low**: hygiene and maintainability.
   For each finding, name the eval or test that would confirm it (for example "an uploaded
   document containing an instruction: does any payment differ from the calculated amount?"). This hands discovery straight to evaluation.

8. **Write the report** with `templates/scan-report.md`. Lead with a one-paragraph verdict and
   the top five findings. Close with open questions that only a person can answer (deployment
   settings, live markets, who receives escalations and how fast).

## Adapting by system type

The authority question is the same everywhere; where authority hides differs.

| System type | Where to look first | Typical critical finding |
|---|---|---|
| Conversational agent with actions | Tool definitions and the tool executor | A binding tool whose amount or target the model chooses, with no check in code |
| Knowledge assistant (retrieval) | Indexing, retrieval filters, the answer prompt | Retrieval ignores document version, permissions or tenant, so answers come from the wrong source or leak another user's data |
| Document extraction | The output schema and the code that posts results | Extracted values written to a system of record with no validation or confidence threshold |
| Classification and routing | Label definitions, thresholds, the fallback route | Low-confidence items silently sent to a default queue; no route for the costly class |
| Content generation | Brand and claims rules, the publish step | Generated text published without review where claims are regulated |
| Autonomous tool-using agent | Tool permissions, sandbox, credentials, step and cost caps | Write or delete access broader than the task needs; secrets readable in the environment |
| Decision support | How recommendations are shown and logged | No record of human overrides, so nobody can tell whether the system helps |

## Output quality bar

- Verdict first: can this go to (or stay in) production, and what must change first.
- Every finding has severity, `path:line` evidence, consequence in business terms, and the
  confirming eval or test.
- The authority map covers every action that changes the world, and says where each limit
  is enforced.
- Strengths are recorded too (audit logging, idempotency, refusal handling). The team needs to
  know what to keep.
- Docs-versus-code disagreements are called out.
- No speculation presented as fact.

## Reference

- `references/checklist.md`: what to look for, section by section, with search hints.
- `references/worked-example.md`: a complete worked scan of a prototype claims assistant at a
  fictional insurer, with findings, evidence and the eval that confirms each. Match its depth
  and its habit of tying each finding to a test.
