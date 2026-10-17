# Eval plan: [feature]

**Date:** [YYYY-MM-DD] · **Exam question:** [quote it]

## Metrics

| Exam-question clause | Metric | Target | Grader | Type (hard gate / threshold / business) |
|---|---|---|---|---|

Guard metrics: [primary → guard, and how the guard stops gaming]

## Oracle

| State | Allowed actions | Failure code if violated |
|---|---|---|

## Failure taxonomy

| Code | Severity (critical / major / soft) | Meaning | Graded by |
|---|---|---|---|

## Graders

- **Action:** [oracle]
- **Text:** [checks]
- **Outcome:** [metrics]
- **LLM judge:** [criteria, model family, calibration plan, soft only]

## Scenario matrix

| Kind | Families | Dimensions crossed | Count | Runs (k) |
|---|---|---|---|---|
| Snapshot | | | | |
| Scripted | | | | |
| Simulated | personas × goal bands × configs | | | |

Golden set: [size, provenance, who owns it]

## Grader proof

| Agent | Expected result |
|---|---|
| Reference policy | Passes almost everything |
| Naive (games the primary metric) | Fails on the guard metric |

## Gate

- Pull request: [what blocks]
- Nightly: [what runs]
- Release: [thresholds]
- Waivers: [file, owner, expiry rule]

## Mapping to findings and controls

| Scan finding or control | Scenario(s) |
|---|---|
