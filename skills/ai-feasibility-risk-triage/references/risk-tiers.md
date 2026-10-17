# Risk tiers and regulatory screen

## Tiers (per action)

| Tier | Typical profile | Minimum controls |
|---|---|---|
| 1 | Reversible, low impact, one interaction; or a suggestion a human acts on | Logging; review sampling; basic evals |
| 2 | Reversible at some cost, or binding within tight limits someone set | Tier 1, plus a code guard on the limits, an output check, and eval coverage of the limits |
| 3 | Binding and hard to reverse, affects money, rights or reputation, or reachable by untrusted input | Tier 2, plus the decision owned by code (not the model), fail-closed guards, a kill switch, a blocking eval gate in CI, and monitoring with alerts |
| 4 | Irreversible or high value, large blast radius, or significant effects on a person | Tier 3, plus human approval before the action, or don't automate the action |

Raise the tier by one when untrusted input reaches the step that decides the action, or when
the affected party never opted in. Lower it by one only when a human reviews every instance
before it takes effect.

## Regulatory and policy screen

Mark each *applies / may apply / doesn't apply*, with the reason and the date checked. These
are prompts for the right conversation with counsel, not conclusions; verify the current status
of every rule, since several are being phased in or amended.

| Area | What to check |
|---|---|
| AI transparency | Do people interacting with the system need to be told it's an AI (for example EU AI Act Article 50 for systems that interact with people; platform or app-store policies)? Is the label added by code? |
| Prohibited and high-risk uses | Does the use fall into a prohibited practice or a high-risk category (for example EU AI Act Annex III: employment, credit, essential services, education, law enforcement)? |
| Automated decisions about people | Does the system make decisions with legal or similarly significant effects on a person without human involvement (for example UK and EU GDPR Article 22)? Is there a route to human review? |
| Data protection | Lawful basis; whether personal data goes to the model vendor; vendor data-processing terms and retention; data residency; whether a DPIA is needed |
| Consumer protection | Could replies mislead (false scarcity, "best price" claims, invented product facts, pressure tactics)? Unfair commercial practice rules (for example UK DMCC Act 2024, EU Unfair Commercial Practices Directive) |
| Sector rules | Financial services (for example FCA Consumer Duty), health, legal, employment, children's data |
| Contract and vendor | Model provider usage policies and terms; customer contracts; promises in the product's own terms of service |
| Internal policy | The organisation's AI policy, risk appetite, approved-vendor list |
