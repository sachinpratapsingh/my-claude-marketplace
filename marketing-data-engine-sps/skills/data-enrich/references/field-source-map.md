# Field-to-source authority map

| Field | Best source | Fallback | Caveat |
|---|---|---|---|
| Job title (current) | LinkedIn Sales Navigator | Apollo | Sales Nav is most current; Apollo/Lusha data can lag 3-6 months behind a job change |
| Employer (current) | LinkedIn Sales Navigator | Apollo | Same staleness caveat — cross-check against Level 5 employment validation |
| Email | Apollo | Lusha | Apollo has stronger deliverability/catch-all detection; verify with contact-validator regardless of source |
| Direct phone | Lusha | Apollo | Lusha has stronger direct-dial coverage for enterprise contacts |
| LinkedIn URL | LinkedIn Sales Navigator | Apollo/Lusha | Always verify the URL resolves to the same person (name + current title match) |
| Company employee count | Apollo | Lusha | Both can lag; treat LinkedIn company page as tie-breaker for large discrepancies |
| Company revenue | Apollo | Lusha | Revenue estimates vary significantly by source for private companies — flag large deltas rather than trusting either blindly |
| Industry | Apollo | — | Apollo's industry taxonomy is broader than most CRMs; always re-map through title-taxonomy-mapper's industry equivalent in account-validator |
| Funding stage / growth signals | Apollo | — | Useful for ICP validation on startup-stage targets; not always populated for larger private companies |

## General rule

Never treat an enrichment source as ground truth on its own. Every enriched value still passes through `account-validator` or `contact-validator` — enrichment fills gaps, it does not replace validation.
