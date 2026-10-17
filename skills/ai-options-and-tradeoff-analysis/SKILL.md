---
name: ai-options-and-tradeoff-analysis
description: Compare architecture options for an AI system and explain the choice to decision-makers - requirements with IDs and acceptance criteria traced to the options, a weighted decision matrix whose criteria and weights are locked before anyone scores the options, quality attributes stated in customer terms, ATAM-style scenario analysis (use-case, growth and exploratory scenarios) that finds risks, non-risks, sensitivity points and trade-off points, and a one-page executive summary that leads with the business outcome, then the choice, then the accepted downside. Use when choosing between build and buy, models, vendors, patterns or platforms, when stakeholders disagree on an approach, or when a decision needs an audit trail. Works in any domain.
---

# AI options and trade-off analysis

Architecture decisions go wrong in two predictable ways: the criteria are chosen after people
have a favourite (so the matrix confirms it), and the trade-offs are hidden from the people who
will live with them. This skill fixes both. Criteria and weights are locked before options are
scored; scenarios expose where each option is sensitive; and the summary states the downside
being accepted, in business terms.

## Inputs

The use-case card (exam question, requirements), triage note (risk tiers, conditions), discovery
synthesis (stakeholder views and conflicts), and the candidate options (or a request to find
them).

## Process

1. **Write the requirements as a list with IDs.** Each has a stable ID, the outcome it serves,
   an acceptance criterion (which later seeds an eval or test) and its type: functional,
   non-functional (these become SLOs) or constraint. Keep an assumption log beside it. When
   stakeholders' requirements conflict and can't be reconciled, the budget owner decides, and
   the decision is recorded.

2. **State quality attributes in customer terms**: "95% of customers see a decision within 24
   hours", not "low latency"; "no customer is paid more than their entitlement", not
   "accurate". Use high percentiles for latency.

3. **Lock the criteria and weights before seeing the options.** Group criteria in tiers:
   - **Must-haves** (pass/fail): hard constraints and zero-tolerance requirements. An option
     that fails one is out, however well it scores elsewhere.
   - **Weighted criteria**: the rest, with weights agreed by the decision owner and recorded
     with the date. Typical criteria: fit to the exam question, risk and control, time to
     value, total cost (build, run, change), operability, vendor and lock-in risk, team
     capability.
   Changing a weight after scoring needs a written reason.

4. **Describe the options** at the same level of detail, including "do nothing" or "improve
   what exists" where relevant. Two to four options is usual.

5. **Score with evidence.** Each score cites its evidence (a spike result, an eval run, a
   vendor demo on your own data, a reference call). Mark scores that are judgement rather than
   evidence. Compute totals, then check sensitivity: would a reasonable change in one weight
   or one uncertain score change the winner? If so, say so.

6. **Run scenario analysis (ATAM-style)** on the leading options. Write scenarios of three
   kinds, each tied to a quality attribute:
   - **Use-case**: normal operation ("a customer submits a typical claim at peak").
   - **Growth**: expected change ("volume triples during a storm"; "a new product line").
   - **Exploratory**: stress and surprise ("a new document-tampering technique"; "the model
     provider deprecates the version"; "the regulator asks why a claim was refused").
   Prioritise the scenarios that are both important and hard. For each, walk through how the
   option handles it and record:
   - **Risks**: decisions that may cause problems.
   - **Non-risks**: decisions that are sound (worth saying, so they aren't reopened).
   - **Sensitivity points**: parameters a quality attribute depends on heavily (a threshold, a
     model tier, a vendor's roadmap).
   - **Trade-off points**: decisions that affect two attributes in opposite directions.

7. **Write the recommendation and its ADR.** The ADR (title, status, context, decision,
   consequences) records the downside honestly. Accepted ADRs are not edited; a later change
   supersedes them.

8. **Write the executive summary** on one page, in this order:
   1. **The business outcome** the decision serves, with its measure.
   2. **The choice**, in one sentence of plain language.
   3. **The accepted trade-off**: what we give up or risk, and why it's worth it.
   4. **What would change the decision** (the sensitivity points), and when it will be revisited.
   5. **What we need from the reader**: approval, budget, a person, a policy decision.
   No architecture diagrams or jargon on this page; link to the full analysis.

## Adapting by system type

| System type | Options usually compared | Criteria that tend to decide |
|---|---|---|
| Conversational agent with actions | Agent vs workflow; build vs a vendor platform | Control over binding actions; auditability |
| Knowledge assistant (retrieval) | Full context with caching vs retrieval; vendor search vs own pipeline | Answer accuracy on your questions; permission handling; freshness |
| Document extraction | Model extraction vs specialist document-AI vendor vs hybrid | Field accuracy on your documents; tamper handling; cost per page |
| Classification and routing | Rules vs small model vs large model; build vs ticketing-tool features | Cost-weighted error; latency; explainability |
| Content generation | Model tiers; in-house vs agency tool | Brand and claims compliance; review effort |
| Autonomous tool-using agent | Off-the-shelf agent vs custom; sandbox options | Permission control; cost per task; failure recovery |
| Decision support | Model vs scorecard vs status quo | Outcome lift; fairness; explainability to regulators |

## Output

A decision pack using `templates/decision-pack.md`: requirements with IDs and traceability,
locked criteria and weights, scored matrix with evidence and a sensitivity check, scenario
analysis findings, the ADR, and the one-page executive summary.

## Quality bar

- Requirements have IDs and acceptance criteria; every one is traced to how each option meets
  it.
- Criteria and weights are dated before scoring; must-haves are pass/fail.
- Every score cites evidence or is marked as judgement; sensitivity is checked.
- Scenarios cover use-case, growth and exploratory kinds; findings are classified as risks,
  non-risks, sensitivity points and trade-off points.
- The ADR and the executive summary both state the downside.
- The executive summary fits one page and leads with the outcome.

## Reference

`references/worked-example.md`: the fictional Larkspur insurer choosing between hardening its
agent prototype, a rules-engine workflow, and buying a claims-automation product.
