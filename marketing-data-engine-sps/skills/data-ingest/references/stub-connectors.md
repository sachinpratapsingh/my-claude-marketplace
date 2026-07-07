# Stub connectors

These source/destination systems do not have a live MCP connector available today. Treat them as documented interface contracts: the field mapping and workflow are defined now so that wiring in a real connector later is a config change, not a rewrite of the ingest or export skills.

## Salesforce

No direct Salesforce MCP connector is currently available in the registry.

Workaround: ask the user to export the relevant report/view as CSV, or check whether they have Zapier, Supermetrics, or another connected middleware tool that can bridge to Salesforce — search the connector registry for these if the user wants a live pull. Tag every record `source_system: salesforce` regardless of how it physically arrived, so lineage survives the eventual switch to a live connector.

## Demandbase, 6sense, RecoTap (ABM platforms)

No MCP connector exists for these today. Workaround: ask the user to export account/contact/intent lists as CSV/Excel from the platform's UI. Preserve any ABM-specific fields these platforms provide (intent score, tier, engagement minutes, buying stage) as custom fields alongside the canonical schema — they are valuable for the `dataset-health-analyzer` growth/intent-signal checks even though they aren't part of the core schema.

## BambooBox (proprietary tool)

BambooBox's MCP server is not yet built. Reserve the source_system value `bamboobox` and the field-mapping contract below so integration is a drop-in once the server ships:

```
bamboobox_field -> canonical_field
company_name    -> company_name_raw
company_domain  -> domain
industry_tag    -> industry_raw
hq_location     -> hq_country / hq_city (split on ingest)
contact_name    -> first_name_raw / last_name_raw (split)
contact_email   -> email
contact_title   -> job_title_raw
li_url          -> linkedin_url
```

Until the server exists, ask the user for a BambooBox data export (CSV/JSON) and map it using the table above. When the BambooBox MCP server becomes available, replace the manual export step with a direct `list_records` / equivalent call — the rest of the ingest skill (Step 3 normalization, Step 4 handoff) does not need to change.

## LinkedIn Sales Navigator

No MCP connector exists for Sales Navigator itself (used for enrichment, not ingest — see the `data-enrich` skill). The same "no live connector, use manual export/paste, preserve field lineage" pattern applies there.
