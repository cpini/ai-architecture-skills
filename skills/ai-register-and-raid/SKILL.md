---
name: ai-register-and-raid
description: Create and maintain the living record for an AI system - a register entry (purpose, owners, model and versions, pattern, risk tier per action, data, regulatory position, controls, eval status, waivers, incidents, review dates) and a RAID log of risks, assumptions, issues and decisions - built from the outputs of the other architecture skills and reviewed on a cadence that flags expiring waivers, unowned risks, stale evals and decisions nobody encoded. Use when an AI feature moves toward pilot or go-live, for a periodic AI governance review, when asked "what's the status of this agent?" or "what are the open risks?", or after an incident or a significant change. Works in any domain and for any kind of AI system.
---

# AI register and RAID

Every other skill produces a document that is true on the day it's written. The register and
RAID log are the one place that stays true: what the system is, who owns it, how risky each of
its actions is, what's protecting it, how it's performing on its evals, and what's still open.
Governance reviews, go-live gates and new team members start here.

Keep both in the repository next to the code, as Markdown or YAML, so they change in the same
pull requests as the system they describe.

## Inputs

Whatever exists of: the use-case card, triage note, discovery synthesis, scan report, design
note and ADRs, guardrail plan, eval plan and latest results, fix log, waivers file, and
post-incident notes.

## Process

1. **Fill the register entry** using `templates/register-entry.md`. Every field is required;
   a blank field is itself an issue for the RAID log ("no named business owner", "cost per
   conversation unmeasured").

2. **Extract the RAID log** from the inputs with these rules:
   - **Risk** (might happen): each failure mode whose primary control is still probabilistic
     (prompt-only); each "may apply" regulatory line; each known trade-off in the policy;
     each unknown from triage not yet retired. Write as *cause → event → impact*, with
     likelihood, impact, owner, mitigation and the trigger that would make it an issue.
   - **Assumption** (believed, not proven): every fact the design relies on that nobody has
     checked against real data, with how and by when it will be validated.
   - **Issue** (happening now): unfixed scan findings, failing eval codes, every active
     waiver (with its expiry), missing components the design assumes exist.
   - **Decision** (settled): every policy decision and ADR, with who made it, when, and where
     it's encoded (oracle, policy code, prompt). A decision not encoded anywhere is an issue.

3. **Review on a cadence** (monthly while in pilot, quarterly when stable, and after every
   incident or significant change). Flag automatically:
   - waivers expiring within 30 days, or already expired;
   - risks or issues without an owner, or past their due date;
   - assumptions past their validation date;
   - eval results older than the last change to the prompt, model, policy or tools;
   - decisions recorded but not encoded;
   - a risk tier that no longer matches the controls in place;
   - model or vendor versions approaching deprecation.

4. **Write the status summary** at the top: overall RAG with the reason, what changed since
   the last review, the top three risks, decisions needed and from whom, and the next review
   date.

## Adapting by system type

The register fields are the same; these entries need extra care.

| System type | Register fields to expand | Typical RAID items |
|---|---|---|
| Conversational agent with actions | Risk tier per action; kill switch; AI disclosure | Actions outside policy; hand-off capacity |
| Knowledge assistant (retrieval) | Document sources, owners and refresh schedule; permission model | Stale documents; answers from superseded versions |
| Document extraction | Field-level accuracy; confidence thresholds; downstream systems written to | New layouts; tampered documents |
| Classification and routing | Class definitions and owners; thresholds; cost per error | Drift; a class with too few examples |
| Content generation | Approval workflow; claims rules; IP position | Regulated claims; brand complaints |
| Autonomous tool-using agent | Permissions and credentials; sandbox; budgets | Privilege creep; cost overruns |
| Decision support | Override rate; fairness monitoring; outcome tracking | Automation bias; unfair outcomes |

## Output

`register-entry.md` and `raid-log.md` for the system (see `templates/`), updated in place, with
the status summary at the top of the register entry. If the organisation keeps a central AI
register, the entry is written so it can be copied there unchanged.

## Quality bar

- Every register field filled, or the gap logged as an issue.
- Every risk written as cause → event → impact, with an owner and a trigger.
- Every waiver has an owner and an expiry, and appears as an issue.
- Every decision says where it's encoded.
- Eval status carries a date and the versions it was run against.
- The status summary can be read in one minute by someone who has never seen the system.

## Reference

`references/worked-example.md`: the register entry and RAID log for a fictional insurer's
claims assistant during its pilot, including a waiver counting down, assumptions waiting on
production data and decisions traced to where they're encoded.
