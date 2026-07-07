---
name: account-validator
description: >
  This skill should be used when the user wants to validate, clean, or check
  company/account-level data for accuracy — company name normalization,
  website/domain checks, industry classification, employee size and revenue
  cross-checks, headquarters, company status, or corporate hierarchy. Trigger
  on phrases like "validate these accounts", "check company data", "clean up
  company names", "is this company still active", or "check the parent
  company mapping". Corresponds to Level 1 (Account Validation) of the data
  quality framework.
metadata:
  version: "0.1.0"
---

Run Level 1 (Account Validation) checks against every account record. This
validates the company itself, independent of any specific contact or ICP.
Attach findings as entries in each record's `validation_flags` array using the
issue codes below, and produce a normalized `company_name_normalized` field —
do not silently overwrite `company_name_raw`.

## 1. Company name normalization

Detect and fix: spelling errors, duplicate spelling variants, legal-suffix inconsistency (Pvt Ltd vs Private Limited, Inc. vs Incorporated), extra whitespace, stray punctuation, ALL-CAPS or all-lowercase input.

Normalize to a single canonical form: strip/standardize legal suffixes, fix casing to proper title case for the brand name, collapse whitespace. Example: `MICROSOFT`, `Microsoft Corporation`, `Microsoft India Pvt Ltd` should all resolve toward a normalized name with the legal suffix separated out, so `company_name_normalized = "Microsoft"` and the legal entity detail is preserved separately if needed for the India-specific record.

Flag code: `ACCT_NAME_NORMALIZED` (informational, not a rejection).

## 2. Website validation

Check: domain resolves, HTTPS works, no redirect to an unrelated domain, not a parked page, not expired.

Reject patterns: `company.co`, `company.in.net`, numeric-suffixed domains (`company123.xyz`), or any domain that doesn't match the expected TLD pattern for a real corporate site. If a live web check isn't feasible in the current environment, flag the record for manual verification instead of guessing.

Flag code: `ACCT_WEBSITE_INVALID` (reject-tier), `ACCT_WEBSITE_UNVERIFIED` (review-tier).

## 3. Industry validation

Junior researchers frequently misassign industry. Cross-check `industry_raw` against a predefined taxonomy (see `references/industry-taxonomy.md`) rather than accepting free text. Watch for classic ambiguity cases: an IT services company tagged "Software", a conglomerate tagged by only one of its business lines (e.g., a company that does both fintech and food delivery).

Flag code: `ACCT_INDUSTRY_MISMATCH`.

## 4. Employee size cross-check

Compare `employee_count_apollo`, `employee_count_linkedin`, and `employee_count_website` (populated during enrichment). If they disagree by more than 30%, flag for review rather than picking one silently. Example: Apollo says 500, LinkedIn says 8,500 — this is a hard flag, not a rounding difference.

Flag code: `ACCT_EMPLOYEE_DISCREPANCY`.

## 5. Revenue validation

Same cross-source discrepancy logic as employee size. Apply a wider tolerance band (revenue estimates are noisier than headcount) — flag only when sources disagree by more than 50%, or when revenue is wildly inconsistent with employee count for the stated industry.

Flag code: `ACCT_REVENUE_DISCREPANCY`.

## 6. Headquarters validation

Verify `hq_country`, `hq_city`, `hq_region` are internally consistent (city belongs to the stated region/country) and consistent with the website's TLD or LinkedIn's listed HQ where available.

Flag code: `ACCT_HQ_INCONSISTENT`.

## 7. Company status

Reject or flag records for companies that are: closed, merged, acquired (old brand name no longer independently addressable), stealth-mode, or dormant. Check `ownership_type` (public vs private) matches what's expected — a company flagged private with a `stock_ticker` populated is a contradiction.

Flag code: `ACCT_STATUS_REJECT` (hard reject), `ACCT_OWNERSHIP_MISMATCH`.

## 8. Parent company / subsidiary hierarchy

Detect and record hierarchy relationships (e.g., Instagram -> Meta) — useful for ABM roll-up reporting so accounts aren't double-counted. Detect subsidiary variants of the same brand (Microsoft India vs Microsoft US) and link them to a shared `parent_company` rather than treating them as unrelated accounts, while still preserving them as separate account records if the user's ICP tracks at the subsidiary level.

Flag code: `ACCT_HIERARCHY_DETECTED` (informational).

## 9. Stock ticker validation

If `ownership_type = public`, `stock_ticker` must be populated and should resolve to a real, currently-traded ticker. If public but no ticker, flag for enrichment. If a ticker is present but the company is marked private, that's an ownership_type error (see #7).

Flag code: `ACCT_TICKER_MISSING`.

## 10. LinkedIn company URL / website consistency

Confirm `linkedin_company_url` points to the correct company (name match) and that the LinkedIn-listed website matches `website`/`domain`. A mismatch (e.g., website is `acme.com` but LinkedIn lists `acmetech.com`) is a real signal of either a wrong LinkedIn match or a company rebrand — flag rather than assume.

Flag code: `ACCT_LINKEDIN_WEBSITE_MISMATCH`.

## Output

For each account, append all applicable flag codes to `validation_flags` and set an interim `account_quality_status` of `pass`, `review`, or `reject` based on whether any hard-reject flags fired (`ACCT_WEBSITE_INVALID`, `ACCT_STATUS_REJECT`). Report a summary table of flag counts by code to the user. Recommend `icp-validator` as the next step.

## Additional resources

- `references/industry-taxonomy.md` — the approved industry classification list and common misclassification patterns.
