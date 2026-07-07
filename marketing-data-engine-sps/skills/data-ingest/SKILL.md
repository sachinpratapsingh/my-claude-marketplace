---
name: data-ingest
description: >
  This skill should be used when the user wants to "load", "import", "pull in",
  or "ingest" marketing or prospecting data from an Excel/CSV file, Airtable,
  HubSpot, Salesforce, or an ABM tool (Demandbase, 6sense, RecoTap, BambooBox)
  into the data quality pipeline. Trigger on phrases like "load this file",
  "pull contacts from Airtable", "import from HubSpot", "bring in the
  Demandbase list", or "start the data engine with this spreadsheet". This is
  always the first stage of the pipeline — run it before enrichment or
  validation skills.
metadata:
  version: "0.1.0"
---

Ingest raw account and contact data from any source and normalize it into the
canonical schema defined in `references/schema.md`. Every other skill in this
plugin (enrichment, validation, scoring, export) reads and writes records in
that schema, so ingestion is the point where source-specific quirks get
resolved once instead of repeatedly downstream.

## Step 1: Identify the source

Ask the user (if not already stated) which source the data is coming from:

- **Excel/CSV file** — a file the user uploads or points to by path.
- **Airtable** — use the `airtable` MCP tools (`list_bases`, `list_tables_for_base`, `get_table_schema`, `list_records_for_table`).
- **HubSpot** — use the `hubspot` MCP tools (`get_properties`, `search_crm_objects`, `get_crm_objects`).
- **Salesforce, Demandbase, 6sense, RecoTap, BambooBox** — no live MCP connector exists yet for these. Follow `references/stub-connectors.md`: ask the user to export a CSV/Excel from the tool instead, and log the source system name on every record so lineage is preserved for when a live connector ships.

## Step 2: Load and inspect

For a file: read it (use the xlsx skill if it's a multi-sheet workbook with formatting, otherwise read the CSV directly). For a connector: pull a small sample first (10-20 records) to confirm the field mapping before pulling the full set — connector schemas vary by workspace/instance and should never be assumed.

Print the detected column headers / field names back to the user and map them to the canonical schema fields in `references/schema.md`. Flag any source column that doesn't have an obvious canonical match and ask the user how to handle it (map it, keep as a custom field, or drop it).

## Step 3: Normalize into canonical records

Produce two record sets: `accounts` and `contacts`, linked by `account_id`. Rules:

- Every record gets a `source_system` field (`excel`, `airtable`, `hubspot`, `salesforce`, `demandbase`, `6sense`, `recotap`, `bamboobox`) and a `source_record_id` so it can be traced back.
- Every record gets an `ingested_at` timestamp (use the current date).
- Do not invent values for missing fields — leave them null. Completeness scoring downstream depends on knowing what's genuinely missing versus present.
- Do not deduplicate or validate at this stage — that belongs to `account-validator`, `contact-validator`, and `dataset-health-analyzer`. Ingest's only job is faithful normalization.

## Step 4: Hand off

Save the normalized dataset (as a working CSV/JSON pair — `accounts_raw.csv` and `contacts_raw.csv`) and tell the user what came in: record counts, source breakdown, and any fields that couldn't be mapped. Recommend the next step: `data-enrich` if coverage looks thin (many nulls in email/phone/LinkedIn/title), or straight to `account-validator` if the data already looks complete.

## Additional resources

- `references/schema.md` — the full canonical account/contact schema every downstream skill expects.
- `references/stub-connectors.md` — how to handle Salesforce, Demandbase, 6sense, RecoTap, and BambooBox until live MCP connectors exist.
