# Title taxonomy reference

Editable per client — if the user's org uses different seniority labels or a different function list, update this file at project start rather than forcing their titles into these defaults.

## Abbreviation map

| Full form | Abbreviation |
|---|---|
| Chief Executive Officer | CEO |
| Chief Information Officer | CIO |
| Chief Technology Officer | CTO |
| Chief Information Security Officer | CISO |
| Chief Financial Officer | CFO |
| Chief Marketing Officer | CMO |
| Chief Operating Officer | COO |
| Executive Vice President | EVP |
| Senior Vice President | SVP |
| Vice President | VP |
| Senior Director | Sr Dir |
| Associate Director | Assoc Dir |
| Director | Dir |
| Senior Manager | Sr Mgr |
| Manager | Mgr |
| Information Technology | IT |

## Seniority tiers (ordered, highest first)

1. C-Level (CEO, CIO, CTO, CISO, CFO, CMO, COO, and equivalents)
2. EVP
3. SVP
4. VP
5. Senior Director
6. Director
7. Associate Director
8. Senior Manager
9. Manager
10. Individual Contributor (Engineer, Analyst, Specialist, Consultant, Associate, Coordinator, and similar non-management titles)

## Function list

IT, Security, Operations, Finance, HR, Sales, Marketing, Product, Engineering, Legal, Procurement, Customer Success, Other

## Buying role definitions

- **Champion** — actively advocates for the purchase internally; typically Director-level or above in the target function.
- **Decision Maker** — holds budget authority or final sign-off; typically VP-level or above, or a named role in the ICP's `required_titles`.
- **Influencer** — shapes the decision but doesn't hold budget authority; often a peer function or a senior IC (e.g., a principal engineer evaluating a technical fit).
- **User** — will use the product/service day-to-day; rarely a buying-committee blocker or accelerant on their own.

## Blacklist keywords (reject, whole-word match)

Intern, Student, Advisor, Retired, Volunteer, Former, Ex- (as a prefix, e.g., "Ex-CTO"), Freelancer, Founder (unless ICP targets founders)

**Edge case notes:**
- "Advisor" should not reject a title like "Trusted Advisor" if that is an actual internal job title at a consulting firm — check context before rejecting; when in doubt, flag for review rather than hard-reject.
- "Founder" should only reject if the ICP is targeting operational buying-committee roles, not founder-led company outreach — confirm with the user which mode applies for this project.

## Whitelist / seniority-fit examples

| ICP wants | Received | Verdict |
|---|---|---|
| Head of IT / VP IT / Director IT | IT Engineer | Reject — seniority too low despite function match |
| Director IT / Above | Senior Manager IT | Reject — one tier below minimum |
| CISO / Security Director | VP Security | Accept — meets or exceeds minimum tier in the right function |
