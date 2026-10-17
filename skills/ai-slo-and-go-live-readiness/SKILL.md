---
name: ai-slo-and-go-live-readiness
description: Get an AI feature ready for production and keep it there - service level indicators that count refusals, malformed and wrong outputs as failures, SLOs set stricter than any SLA, error budgets and multi-window burn-rate alerts, a production gate with exit criteria (representative evals, monitoring, rollback path), a staged pilot plan, a hand-off pack (decision log, implementation guidance, runbook, acceptance criteria), and rules for telling drift from regression. Use before a pilot or go-live, when defining SLAs or SLOs for an LLM service, when setting up alerting and on-call, when handing a system to an operations team, or when deciding whether to widen a rollout. Works in any domain.
---

# AI SLOs and go-live readiness

An AI feature is ready for production when three things are true: you can tell when it's
failing (including failures that return HTTP 200), you have agreed how much failure is
acceptable and what happens when that's used up, and you can switch it off or roll it back in
minutes. Phase gates are passed on evidence against exit criteria, not on dates.

## Inputs

The exam question and metrics (`ai-use-case-framing`, `ai-eval-suite-design`), the guardrail
plan (kill switch, monitoring), the register entry, any contractual or customer-facing
commitments (SLAs), and the operations team who will run it.

## Process

1. **Define the indicators (SLIs).** For each user journey, define what a *good* event is.
   For AI features, good means more than "returned 200":
   - **Availability / success**: the request completed with a valid, usable output. Refusals,
     truncated outputs, schema-invalid outputs and fallbacks to an error message count as
     failures.
   - **Latency**: measured at a high percentile (p95 or p99), split into time to first token
     and time to complete for streamed responses, and time to outcome for asynchronous work.
   - **Quality**: from sampled grading of live traffic with the offline graders (accuracy,
     groundedness, policy compliance), reported over a window.
   Separate three kinds of failure in dashboards: infrastructure errors, refusals, and silent
   wrong answers.

2. **Set SLOs stricter than SLAs.** If there's an external or internal SLA, the SLO sits
   inside it (an SLA of 99.9% gets an SLO of 99.95%), so you act before the promise breaks.
   Set targets from a healthy baseline, not from hope.

3. **Derive the error budget.** Budget = 1 − SLO over the window. 99.9% over 30 days is about
   43 minutes of full outage, or the equivalent share of failed requests. Agree the policy when
   it's spent: freeze changes except P0 and security fixes until the budget recovers; an
   incident that consumes more than about 20% of the budget gets a postmortem with at least one
   P0 action. A purely external cause (a provider outage) doesn't trigger a freeze but still
   gets reviewed.

4. **Alert on burn rate, over two windows.** Burn rate = how fast the budget is being spent
   relative to the SLO (1 = exactly on budget). A common scheme:
   - **Page** when the burn rate exceeds about 14.4 over 1 hour (2% of a 30-day budget in an
     hour).
   - **Ticket** when it exceeds about 6 over 6 hours.
   - **Review** when it exceeds 1 over 3 days.
   Use a short confirmation window alongside each so alerts clear quickly when the problem
   stops. Quality SLOs alert on sampled-grade drops and on any critical failure code.

5. **Write the production gate** with `templates/readiness-pack.md`. Exit criteria typically
   include:
   - Evals representative of live traffic pass the release gate (golden set, pass^k, zero
     critical), on the exact versions being released.
   - Monitoring and SLO dashboards live; alerts tested by firing them.
   - **Rollback path** tested: kill switch to the fallback (often a human queue), previous
     prompt and model versions redeployable without a code change.
   - Runbook written and walked through by the on-call team.
   - Register entry complete; open RAID items accepted by named owners.
   - Compliance sign-offs (DPIA, disclosure) recorded.

6. **Plan the pilot in stages.** Start with 2–3 champion teams or a small traffic share, with
   exit criteria for each step (for example 5% → 25% → 100%), a minimum duration, and the
   metrics that must hold. Widen only on evidence. Keep a comparison group (human-handled or
   previous version) to measure the business outcome.

7. **Prepare the hand-off pack** for the team that will run it:
   - **Decision log**: the ADRs and policy decisions, with where each is encoded.
   - **Implementation guidance**: architecture, configuration, how to change prompts and
     thresholds safely (through the eval gate).
   - **Runbook**: alerts and what each means, diagnosis steps, the kill switch, rollback,
     escalation contacts, provider status checks.
   - **Acceptance criteria**: what the receiving team checked before accepting.

8. **Define drift versus regression.** **Regression**: you shipped a change and quality fell;
   roll back first, then fix through the eval loop. **Drift**: the world changed (new input
   types, seasonal patterns, new document versions, a provider model update) with no change
   from you; collect the new cases, add them to the evals, then adapt. Detect drift by
   re-scoring sampled live traffic with the offline graders and by watching input
   distributions. Iterate on evidence, prompt and retrieval before model changes.

9. **Plan retirement.** Pinned model versions have end-of-life dates: schedule migrations with
   eval runs well before them. Retiring the feature itself needs a plan too (data, users,
   fallback).

## Adapting by system type

| System type | Key SLIs beyond availability | Typical fallback |
|---|---|---|
| Conversational agent with actions | Reply latency (first token); action correctness from sampling | Hand off to a human queue |
| Knowledge assistant (retrieval) | Groundedness and refusal correctness from sampling; index freshness | "I can't answer that; here's who can" |
| Document extraction | Field accuracy from sampling; straight-through rate; queue age | Manual keying queue |
| Classification and routing | Cost-weighted error from sampling; time to route | Default queue with human triage |
| Content generation | Edit rate; approval rejections | Human drafting |
| Autonomous tool-using agent | Task success; cost per task; forbidden-action count (zero) | Pause the agent; human takes the task |
| Decision support | Recommendation availability; override rate | Decision without the recommendation |

## Output

A readiness pack using `templates/readiness-pack.md`: SLIs, SLOs and SLAs, error budget and
policy, burn-rate alerts, production gate with evidence against each exit criterion, pilot
stages, hand-off pack contents, and drift and regression procedures.

## Quality bar

- SLIs count refusals, malformed and truncated outputs as failures, and include a quality SLI.
- Latency is at a high percentile, split into first token and completion where streamed.
- SLOs are stricter than SLAs; the error budget has an agreed policy.
- Alerts use burn rates over at least two windows, and have been test-fired.
- The rollback path has been exercised, not just documented.
- Every gate decision cites evidence against written exit criteria.

## Reference

`references/worked-example.md`: the fictional Larkspur insurer's claims assistant SLOs, error
budget, alerts, production gate and the decision to widen the pilot from 5% to 25%, including a
drift event during a storm.
