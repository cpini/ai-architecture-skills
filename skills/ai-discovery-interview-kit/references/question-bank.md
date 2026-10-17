# Question bank

Pick modules by role. Rephrase into the interviewee's language; never read these out as a
list. Prefer "tell me about the last time…" over "how do you…".

| Module | Ask |
|---|---|
| A. Outcome and value | Sponsor, process owner |
| B. Current process and baseline | Process owner, end user, operator |
| C. Decisions and authority | Sponsor, process owner, operator |
| D. Errors and tolerance | Sponsor, process owner, risk |
| E. Affected parties | Affected-party proxy, sponsor, risk |
| F. Boundaries and edge cases | Anyone who owns a decision |
| G. Data | Data owner, engineering |
| H. Integration and runtime | Engineering |
| I. Risk, regulation and policy | Risk, legal, compliance |
| J. Operations and hand-off | Operator, end user |
| K. Success and go-live | Sponsor, process owner |

## A. Outcome and value
- What would make you call this a success in six months? How would you know?
- Which number moves if it works? What is it today?
- What could it do that would make it look successful but actually be bad? (Finds the guard metric.)
- What happens if we don't build it?

## B. Current process and baseline
- Walk me through the last time this happened, start to finish.
- How long does it take, how often, how many people, what does it cost?
- Where do people use workarounds: spreadsheets, notes, memory?
- Can you share five real examples: a typical one, a hard one, one that went wrong?

## C. Decisions and authority
- What decisions are made in this process? Who is allowed to make each one?
- Which of them could be written as a rule? Which need judgement?
- What limits does each decision-maker have (amounts, discounts, exceptions)? Who sets them?
- Which actions bind the organisation or someone else? Can they be undone?

## D. Errors and tolerance
- Which mistake would be worst? Which one happens today and nobody minds?
- For each kind of mistake: zero tolerance, rare is acceptable, or fine if corrected?
- Has a mistake here ever become a complaint, a refund, a regulator question, a headline?

## E. Affected parties
- Who is on the receiving end without choosing to use this?
- What do they need to know about it? Would they expect to be told it's an AI?
- What do they complain about today?

## F. Boundaries and edge cases
For each decision, walk the edges:
- Just above, at, and just below the limit: what should happen?
- When the other side says "final", repeats a request, or threatens to complain or leave?
- When instructions conflict: the customer asks for one thing, policy says another?
- When information is missing, stale, or contradictory?
- In another language, currency, market or channel?
- Out of hours, or when the person it hands off to doesn't reply?
- Is any acceptable answer fine, or is there something to maximise (price, satisfaction, speed)?
- Should the system say no, hand off to a person, or keep going?
- After a "no", can the other party try again?

## G. Data
- Where does each piece of information live? Who owns it? How fresh is it?
- What's its quality? Known gaps or errors?
- May it be sent to an external model provider? Any residency or retention limits?
- Which fields are actually used, and which exist but nobody fills in?

## H. Integration and runtime
- What triggers the work? What does the system read and write?
- What exists today, and what does the design assume exists (notifications, approval screens)?
- Latency and availability expectations? Volume and peaks?
- How are changes tested and deployed? Is there a staging environment and test data?

## I. Risk, regulation and policy
- Which markets and customer types are in scope?
- Transparency, data protection, consumer protection or sector rules you already apply here?
- Internal AI policy, approved vendors, risk appetite?
- Who signs off go-live from a risk point of view?

## J. Operations and hand-off
- When the system can't handle something, who takes over? How quickly can they respond?
- How will they be notified? What will they need to see to decide quickly?
- Who watches it day to day? What would make them switch it off?

## K. Success and go-live
- What evidence would you need to let it run without approving every action?
- Pilot: who, how many, how long?
- What would make you roll it back?
