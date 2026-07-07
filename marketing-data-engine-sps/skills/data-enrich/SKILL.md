---
name: data-enrich
description: >
  This skill should be used when the user wants to "enrich", "fill in missing
  fields", "find emails/phones/LinkedIn URLs", or "supplement" account or
  contact data before validation. Trigger on phrases like "enrich this list",
  "find missing emails", "fill the gaps with Apollo/Lusha", or "get LinkedIn
  URLs for these contacts". Runs after data-ingest and before the validator
  skills — enrichment should happen before validation so validators are
  scoring the most complete version of each record.
metadata:
  version: "0.1.0"
---

Fill missing fields on ingested account/contact records using a cost-aware
enrichment waterfall: LinkedIn Sales Navigator first, then Apollo, then Lusha.
Stop querying additional sources for a field as soon as one source fills it —
never call all three for a field that's already resolved.

## Enrichment waterfall

For each missing mandatory field (see `references/schema.md` in the
`data-ingest` skill for the mandatory list), attempt sources in this order:

1. **LinkedIn Sales Navigator** — no MCP connector exists yet. Ask the user to paste Sales Navigator export data or search results, or skip this source if they don't have Sales Navigator access. Document this gap rather than silently skipping to Apollo — the user should know tier 1 wasn't available for this run.
2. **Apollo** (`apollo` MCP tools: `apollo_enrich_organization`, `apollo_enrich_person`, `apollo_bulk_enrich_people`, `apollo_search_people`, `apollo_search_organizations`) — use for company firmographics and person-level title/email/phone/LinkedIn enrichment.
3. **Lusha** (`lusha` MCP tools: `contacts_search`, `companies_search`, `prospecting_contact_enrich`, `prospecting_company_enrich`) — use as the fallback for whatever Apollo couldn't resolve.

## Step 1: Identify enrichment targets

Scan the dataset for records missing mandatory fields (see schema). Group by what's missing — company-level gaps (industry, employee count, HQ) get resolved via organization enrichment; contact-level gaps (email, phone, title, LinkedIn) via person enrichment. Report the gap counts to the user before running (e.g., "212 of 800 contacts are missing LinkedIn URLs, 340 are missing verified email").

Batch-enrich where the connector supports it (`apollo_bulk_enrich_people`) rather than looping one record at a time.

## Step 2: Run the waterfall

For each gap:

1. Try LinkedIn Sales Navigator (manual/pasted data — see above).
2. If still missing, call the relevant Apollo tool.
3. If still missing, call the relevant Lusha tool.
4. If still missing after all three, leave the field null — do not fabricate data.

Record the source that filled each field in `enrichment_source` (e.g., `email: apollo`, `linkedin_url: lusha`). This matters later: `dataset-health-analyzer`'s accuracy scoring should weight fields with cross-source agreement higher than single-source enrichment.

## Step 3: Cross-check, don't just fill

When more than one source returns a value for the same field (e.g., employee count from both Apollo and Lusha), keep both values rather than silently picking one — `account-validator`'s Level 1 employee-size discrepancy check needs both to flag mismatches. Only collapse to a single value when sources agree.

## Step 4: Report and hand off

Summarize: how many fields were filled, broken down by source and by field type, and how many gaps remain unresolved after the full waterfall. Flag cost-relevant totals (Apollo/Lusha call counts) since these are typically metered APIs. Recommend running `account-validator` and `contact-validator` next.

## Additional resources

- `references/field-source-map.md` — which enrichment source is authoritative for which field, and known reliability caveats per source.
