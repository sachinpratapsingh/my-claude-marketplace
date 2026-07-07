---
name: crm-export-router
description: >
  This skill should be used when the user wants to load scored, validated
  data into a destination system — CRM, marketing automation platform, or
  Airtable staging base — based on its MDQS tier. Trigger on phrases like
  "load this into HubSpot", "push the Gold records to the CRM", "export to
  Airtable", "send this to Salesforce", or "route these by quality tier".
  This is the final stage of the pipeline — it should only run after
  dataset-health-analyzer has produced mdqs_score and mdqs_tier for every
  record.
metadata:
  version: "0.1.0"
---

Route scored records to their destination based on `mdqs_tier`, so the
dataset never lands in a production CRM/MAP without passing through the
quality gate established by the rest of this pipeline.

## Step 1: Confirm the routing plan before writing anything

Ask the user (if not already established) which destination each tier should go to. A sensible default, but always confirm for this run:

- **Gold** (auto-import tier) — push directly to the target CRM/MAP.
- **Silver** (import with monitoring) — push to the same destination, tagged/flagged for a monitoring list (e.g., a HubSpot property or Airtable column marking it for spot-check).
- **Review** (manual review required) — do not push to the production CRM. Write to a review-queue destination (a separate Airtable table or a CSV) for a human to work through.
- **Reject** — do not push anywhere live. Write to a rejected-records export with the specific reject reasons (`validation_flags`) attached, so the original researcher/source can act on it.

## Step 2: Route by destination type

**Airtable** — use the `airtable` MCP tools (`create_records_for_table`, or check `get_table_schema` first to confirm field names match before writing). Confirm the target base/table with the user; do not assume a base exists.

**HubSpot** — use the `hubspot` MCP tools. HubSpot's connected toolset here is read/search-oriented (`search_crm_objects`, `get_crm_objects`); if it doesn't expose a create/update tool for this workspace, tell the user directly rather than guessing — offer to export a HubSpot-ready CSV instead for manual import.

**Salesforce** — no live connector exists (see `data-ingest/references/stub-connectors.md`). Produce a Salesforce-import-ready CSV instead (respecting Salesforce's standard object field names for Account/Contact/Lead if the user specifies which object type) and tell the user it needs manual import until a connector is available.

**BambooBox** — no MCP server exists yet. Produce an export using the field-mapping contract in `data-ingest/references/stub-connectors.md` (canonical -> bamboobox_field, i.e., the mapping reversed) so it's ready to load the moment BambooBox's connector ships. Do not attempt a live push.

## Step 3: Never skip the gate

Do not push Review or Reject tier records to a live CRM/MAP destination even if the user's original request was "load everything" — clarify explicitly if they truly want unfiltered load (rare, and worth a second confirmation) versus routing by tier (the default and recommended behavior).

## Step 4: Report

Summarize what was routed where: counts by tier and destination, any records that failed to write (with the destination-reported error, not a guessed cause), and where the review-queue and rejected-records exports can be found.

## Additional resources

- `../data-ingest/references/stub-connectors.md` — field-mapping contracts for Salesforce, Demandbase, 6sense, RecoTap, and BambooBox until live connectors exist.
