---
name: icp-validator
description: >
  This skill should be used when the user wants to check whether accounts or
  contacts actually match a defined Ideal Customer Profile (ICP) — industry
  fit, geography, employee/revenue bands, tech stack evidence, funding stage,
  growth signals, or intent signals. Trigger on phrases like "check ICP fit",
  "does this match our target profile", "filter to accounts in our ICP", or
  "validate against the target list". Corresponds to Level 2 (ICP Validation).
  This skill is per-client: always load or ask for the specific ICP definition
  before scoring, since thresholds differ by client/campaign.
metadata:
  version: "0.1.0"
---

Score every account (and its contacts, where relevant) against a specific
ICP definition. Unlike account-validator, this skill's pass/fail thresholds
are never hardcoded — a different client or campaign means a different ICP,
so always establish the current ICP definition before scoring.

## Step 0: Load the ICP definition (always ask, every run)

Before scoring, get the ICP definition for this run. Ask the user for it if not already provided in the conversation — do not reuse an ICP from a previous run without confirming it still applies. Use `references/icp-template.md` as the structure to request or fill in:

- Target industries (and explicitly, which adjacent industries are OUT — e.g., "Banks, not NBFC or Fintech unless stated")
- Target geography (country/state/region)
- Employee count band
- Revenue band
- Required technology stack evidence (e.g., "must show Azure usage")
- Funding stage requirements (if targeting startups)
- Growth signal preferences (hiring, funding, expansion, acquisition, IPO)
- Intent signal sources to weight, if available (Bombora, G2, 6sense) — optional enrichment, not required for a pass

If the user has no formal ICP written down, help them build one quickly using `references/icp-template.md` rather than guessing at defaults — an unstated ICP produces meaningless validation results.

## Step 1: Score each account against the loaded ICP

For each dimension, mark match / partial / no-match / insufficient-data:

1. **Industry match** — must fall within the ICP's target industries, using the taxonomy from `account-validator`. Adjacent-but-excluded industries (e.g., NBFC when only Banks are wanted) are a no-match, not a partial.
2. **Geography** — country, then state/region if specified. A right-country-wrong-region hit is a partial, not a full match, unless the ICP is country-level only.
3. **Employee count** — must fall inside the stated band. Significantly under the floor (e.g., wanted 1000+, got 250) is a hard no-match, not a rounding issue.
4. **Revenue band** — same logic as employee count, using the account's validated revenue from account-validator.
5. **Technology stack evidence** — if the ICP requires evidence of a specific technology (e.g., Azure), look for it in enrichment data, job postings, or public signals. Mark insufficient-data rather than no-match if there's simply no evidence either way.
6. **Funding stage** — only applies if the ICP specifies a startup/funding-stage target.
7. **Growth signals** — hiring, funding, expansion, acquisition, IPO activity. Optional but valuable — note presence/absence, don't penalize absence unless the ICP explicitly requires a growth signal.
8. **Intent signals** — Bombora/G2/6sense data if available. Treat as bonus signal, not a validation gate, unless the ICP explicitly requires it.

## Step 2: Assign an ICP fit verdict

Combine dimension results into `icp_fit_status`: `strong_fit` (industry + geography + employee/revenue all match), `partial_fit` (one secondary dimension off, but core three match), or `no_fit` (any of industry/geography/employee-band fails). Record the specific failing dimension(s) in `validation_flags` using codes like `ICP_INDUSTRY_NOFIT`, `ICP_GEO_NOFIT`, `ICP_SIZE_NOFIT`, `ICP_REVENUE_NOFIT`.

## Step 3: Report

Summarize fit distribution (strong/partial/no-fit counts) and the most common failing dimension — this tells the user where the mining process is systematically drifting from the ICP (e.g., "68% of no-fits failed on employee count, meaning the source list is skewing toward smaller companies than targeted"). Recommend `contact-validator` and `title-taxonomy-mapper` next for the surviving accounts' contacts.

## Additional resources

- `references/icp-template.md` — structured template to capture or elicit an ICP definition when the user doesn't have one ready.
