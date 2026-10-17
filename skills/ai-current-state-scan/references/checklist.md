# Current-state scan checklist

Work top to bottom. Sections 1–3 find most critical and high findings; do them in full.

## 1. Authority: what can the model decide?

- [ ] List every tool the model can call. For each: does it write, send, spend, commit, delete
      or only read?
- [ ] For each binding tool, who chooses the arguments (amounts, recipients, IDs)? The model,
      or code?
- [ ] Find the code that executes each tool. Does it validate arguments against policy before
      acting (limits, ranges, ownership, state)? Quote the check, or record that none exists.
- [ ] Follow the call downstream (API route, service, database rule). Is the limit enforced
      there? A missing agent-side check may be covered by the backend, or not.
- [ ] What happens if config is missing or malformed: does the limit default to something safe
      (fail closed) or permissive (fail open)?
- [ ] Is there a cap on binding actions per turn, iterations per message and tokens per
      message?
- [ ] Can the model act on free text (an amount typed in chat, a value read from a document)
      or only on validated, structured objects?

Search hints: `def _execute_tool`, `tool_name ==`, `if tool_name`, route handlers the tools
call, `amount`, `price`, `limit`, `threshold`, `max_`, `approve`, `delete`, `send`.

## 2. Prompt contents: what does the model know?

- [ ] Read the full system prompt as rendered, with a realistic config. Template variables hide
      what the model actually sees.
- [ ] Confidential values in the prompt: limits, thresholds, scoring rules, internal margins,
      other customers' data, API keys. Anything in the prompt can be extracted by a persistent user.
- [ ] Business rules expressed only as prompt instructions ("never pay more than X", "always
      escalate when Y"). Each is a probabilistic control; record whether code backs it up.
- [ ] Rules that contradict each other or the stated objective (for example "if unsure,
      approve" alongside "pay the correct amount").
- [ ] Arithmetic the model is asked to do (midpoints, percentages, rounding, schedules). Models
      get this wrong some of the time; it belongs in code.
- [ ] Identity and disclosure instructions ("never reveal you are an AI", "you ARE the
      account manager"). Check against transparency obligations (EU AI Act Article 50 for AI systems
      that interact with people; consumer protection law).
- [ ] Tool descriptions: they are prompt too. Check they match the intended policy.

## 3. Untrusted input: who can talk to the model?

- [ ] Which inputs come from people outside the organisation (customers, counterparties,
      uploaded documents, web pages, email)?
- [ ] Is untrusted text separated from instructions (tags, delimiters, a separate message
      role) and is the prompt told it is data, not instructions?
- [ ] Can untrusted text imitate platform markers the prompt trusts (for example a fake
      "[SYSTEM]" line, or a heading that looks like a verified field)? Are those markers stripped or neutralised?
- [ ] Do other record fields reach the prompt unfenced (descriptions written by users,
      AI-prefilled fields, profile data)?

## 4. Output controls

- [ ] Is anything checked before a reply is sent: moderation, grounding against source data,
      contact and payment details, promises outside the tool set, PII?
- [ ] Do those checks fail closed or open?
- [ ] Is an AI label added in code, or left to the model?
- [ ] Can the reply contradict the action taken (says "approved" when the case was only routed
      for review)?

## 5. State, concurrency and idempotency

- [ ] How is state derived: from the database, or reconstructed from message history? Look
      for counting and parsing logic (rounds, attempts, totals): off-by-one and wrong filters
      are common.
- [ ] Locks: in-process only, or distributed? Does the service scale beyond one instance?
- [ ] Idempotency on retries: a transaction, or a read followed by a write?
- [ ] Stale state: does the tool re-read live status (record closed, status already changed by a person)
      before acting?

## 6. Escalation and human-in-the-loop

- [ ] Is there a hand-off path, and what triggers it?
- [ ] Does the human actually find out (notification, queue, SLA), or does the case dead-end?
- [ ] What does the user hear while waiting?

## 7. Observability

- [ ] What is logged per action: inputs, tool, arguments, result, tokens, latency, model ID,
      prompt version?
- [ ] Are refusals, errors and guard trips distinguishable in the logs?
- [ ] Is personal data in the logs, and is retention defined?

## 8. Testing and evaluation

- [ ] Unit tests: what do they cover, and are model calls mocked?
- [ ] Is there any eval set, golden set, red-team set or regression gate in CI?
- [ ] Do tests assert behaviour that is actually wrong (tests that lock in a bad rule)?

## 9. Configuration

- [ ] Every config key the code reads, its default, and whether it is still used.
- [ ] Seed scripts and admin UIs that set keys the code ignores (misleading for operators).
- [ ] Model ID and parameters: pinned or floating?

## 10. Compliance and data

- [ ] AI disclosure to the people it talks to.
- [ ] Where conversation data goes (model provider, logs, analytics) and under which terms.
- [ ] Markets served, and which obligations apply there.
