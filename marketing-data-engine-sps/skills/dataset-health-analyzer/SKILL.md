---
name: dataset-health-analyzer
description: >
  This skill should be used when the user wants an overall health check of a
  dataset rather than a single record — completeness/duplicate rates, buying
  committee coverage, persona and department distribution, marketing
  readiness, consistency issues, freshness, or a final Marketing Data Quality
  Score (MDQS) with Gold/Silver/Review/Reject classification. Trigger on
  phrases like "score this dataset", "how healthy is this list", "give me the
  MDQS", "what's our buying committee coverage", "run the final quality
  report", or "classify these records". Run this after account-validator,
  icp-validator, contact-validator, and title-taxonomy-mapper have all run —
  it aggregates their flags rather than re-validating individual fields.
metadata:
  version: "0.1.0"
---

Aggregate the outputs of the other validator skills into dataset-level
analytics (Levels 6-10), then compute a weighted Marketing Data Quality Score
(MDQS, Level 11) per record and an overall dataset health summary (Level 12).
This skill is per-client: MDQS weights and tier thresholds should be loaded or
confirmed at the start of every run rather than assumed, since different
clients weight quality dimensions differently. Use `references/mdqs-config-template.md`.

## Step 0: Load the scoring config for this run

Ask for or confirm the MDQS weights and thresholds before scoring (see `references/mdqs-config-template.md`). If the user has no preference, offer the framework default as a starting point but still confirm it explicitly for this run rather than silently applying it:

Default weights: Account Quality 20%, ICP Match 20%, Contact Quality 20%, Job Title & Persona Accuracy 20%, Freshness & Consistency 20%.
Default tiers: Gold 95-100 (auto-import), Silver 90-94 (import with monitoring), Review 80-89 (manual review required), Reject below 80 (send back for research).

## Level 6: Completeness

Check every mandatory field from the canonical schema (`data-ingest/references/schema.md`) is populated. Compute a per-record completeness percentage and an overall dataset completeness rate. Missing mandatory fields reduce the completeness sub-score directly.

## Level 7: Duplicate detection

Aggregate the duplicate flags already raised by `account-validator` (company-level) and `contact-validator` (person-level): exact duplicates, near-duplicates, alias companies, same-domain/phone/LinkedIn matches. Report the overall duplicate rate. Do not auto-merge — surface duplicate clusters for the user to resolve, since merge decisions (which record is "canonical") often need human judgment.

## Level 8: Marketing readiness

This goes beyond per-record quality — can marketing actually use this dataset as delivered?

- **Buying committee coverage**: for each account, check whether the required buying-committee titles (from the loaded ICP's `required_titles`) are present. Report the gap — e.g., "ICP wants CIO/CTO/CISO/IT Director/Network Head; this dataset only surfaced IT Managers for 40% of accounts."
- **Persona distribution**: report the actual seniority-tier mix (from `title-taxonomy-mapper`) against a target mix if the user has one (e.g., target 20% C-Level / 30% VP / 50% Director).
- **Department distribution**: report function mix.
- **Region distribution**: report geographic spread against ICP targets.
- **Company coverage**: flag accounts below the ICP's `min_contacts_per_account` threshold.
- **Title diversity, email coverage %, LinkedIn coverage %, phone coverage %**: straightforward completeness rates scoped to these specific fields, reported separately because marketing teams often care about them individually (e.g., phone coverage matters for outbound calling campaigns even if email coverage is fine).

## Level 9: Consistency checks

Cross-field logical checks that individual-record validators don't catch because they need cross-referencing:

- Title says a function (e.g., CIO) but department field says something contradictory (e.g., Sales).
- Account industry says one thing (e.g., Healthcare) but website content/domain suggests another (e.g., a banking domain).
- Contact's email domain doesn't match the linked account's company (e.g., contact tagged to Google but email is `@amazon.com`) — likely reject, not just a flag.
- Contact's phone country code doesn't match their stated location.

Flag code: `CONSISTENCY_CROSSFIELD_MISMATCH`.

## Level 10: Freshness

Check `last_updated` against a staleness threshold (default: flag records older than 12 months, reject records older than 24 months unless the user specifies otherwise for this run). Where available, also weigh LinkedIn activity recency, website liveness, and recent funding/employee-count updates as freshness signals.

## Level 11: MDQS scoring

For each record, compute weighted sub-scores using the confirmed config:

- **Account Quality** — derived from `account-validator` flags (fewer/lower-severity flags = higher score).
- **ICP Match** — derived from `icp-validator`'s fit verdict (strong_fit=full marks, partial_fit=partial, no_fit=zero).
- **Contact Quality** — derived from `contact-validator` flags.
- **Job Title & Persona Accuracy** — derived from `title-taxonomy-mapper` flags and ICP seniority fit.
- **Freshness & Consistency** — derived from Levels 9-10 above.

Sum weighted sub-scores into `mdqs_score` (0-100) and classify into `mdqs_tier` (Gold/Silver/Review/Reject) using the confirmed thresholds.

## Level 12: Dataset-level summary

Produce one consolidated report covering: coverage metrics (accounts mined vs. target, contacts per account min/avg/max, buying-committee role coverage, geographic/industry distribution vs. targets), title analysis (top titles by frequency, out-of-taxonomy titles, seniority/function distribution, rejected/ambiguous title %), account analysis (duplicate rate, multi-domain companies, subsidiary-to-parent mapping, out-of-ICP accounts, missing-enrichment accounts), contact quality (email/LinkedIn/phone coverage %, personal/role/invalid email %, duplicate contact %), and the KPI rollup (completeness, accuracy, freshness, relevance, duplicate rate, manual-review rate, auto-accept rate, rejection rate).

## Output

Write the scored dataset (all records with `mdqs_score`, `mdqs_tier`, and all upstream `validation_flags` intact) plus a manual-review queue (Review-tier and flagged-but-not-rejected records) as separate outputs. Recommend `crm-export-router` as the final step.

## Additional resources

- `references/mdqs-config-template.md` — the weights/thresholds template to confirm each run.
