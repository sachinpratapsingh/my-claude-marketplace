# Marketing Data Engine (marketing-data-engine-sps)

A complete marketing data quality pipeline: ingest raw prospecting data, enrich the gaps, validate it through a 12-level / 100+ point framework, score it with a weighted Marketing Data Quality Score (MDQS), and route it to a CRM or marketing automation platform — never loading unvalidated data into production.

## Overview

Junior researchers mining data from Apollo, Lusha, or LinkedIn Sales Navigator typically only get email validation right. This plugin implements the full quality model: completeness, accuracy, relevance, consistency, and freshness — checked at the account, ICP-fit, contact, job-title, and employment level, then rolled up into a dataset-wide health report and a per-record MDQS tier (Gold / Silver / Review / Reject).

## Components (8 skills)

| Skill | Pipeline stage | Purpose |
|---|---|---|
| `data-ingest` | 1 | Load data from Excel/CSV, Airtable, HubSpot, or a stubbed connector (Salesforce, Demandbase, 6sense, RecoTap, BambooBox); normalize into the canonical schema |
| `data-enrich` | 2 | Fill missing fields via a LinkedIn Sales Navigator > Apollo > Lusha waterfall |
| `account-validator` | 3 | Level 1: company name, website, industry, employee/revenue cross-checks, HQ, status, hierarchy, ticker, LinkedIn consistency |
| `icp-validator` | 3 | Level 2: industry/geography/size/tech-stack/funding/growth/intent fit against a per-run ICP definition |
| `contact-validator` | 3 | Levels 3 & 5: name, email, phone, LinkedIn, duplicates, employment/job-change checks |
| `title-taxonomy-mapper` | 3 | Level 4: title normalization, seniority/function extraction, buying-role mapping, blacklist/whitelist filtering |
| `dataset-health-analyzer` | 4 | Levels 6-12: completeness, duplicates, marketing readiness, consistency, freshness, and final MDQS scoring |
| `crm-export-router` | 5 | Routes Gold/Silver/Review/Reject records to the right destination — live connector or stub export |

Skills are designed to run in sequence (ingest -> enrich -> [account/icp/contact/title validators] -> dataset health -> export), but each can also be invoked standalone against an already-partially-processed dataset.

## Setup

### Live MCP connectors (bundled in `.mcp.json`)

- **Airtable** — `https://mcp.airtable.com/mcp`
- **HubSpot** — `https://mcp.hubspot.com/anthropic`
- **Apollo.io** — `https://mcp.apollo.io/mcp`
- **Lusha** — `https://mcp.lusha.com/mcp/claude`

Connect each via Cowork's connector settings before running skills that depend on them. No API keys are hardcoded in this plugin — authentication is handled by each connector's own OAuth flow.

### Stubbed connectors (no live MCP server yet)

Salesforce, Demandbase, 6sense, RecoTap, LinkedIn Sales Navigator, and BambooBox have no MCP connector today. These are handled via documented manual-export workflows (see `skills/data-ingest/references/stub-connectors.md`) so that wiring in a real connector later is a config change, not a rewrite. BambooBox specifically ships with a full field-mapping contract ready for when its MCP server launches.

## Usage

Run skills conversationally, e.g.:

- "Load this spreadsheet of Apollo contacts" -> `data-ingest`
- "Enrich the missing emails and LinkedIn URLs" -> `data-enrich`
- "Validate these accounts" -> `account-validator`
- "Check these against our banking ICP" -> `icp-validator` (will ask for the ICP definition — see `skills/icp-validator/references/icp-template.md`)
- "Validate the contacts and check for duplicates" -> `contact-validator`
- "Map these titles and check seniority" -> `title-taxonomy-mapper`
- "Give me the full quality report and MDQS scores" -> `dataset-health-analyzer` (will ask for scoring weights — see `skills/dataset-health-analyzer/references/mdqs-config-template.md`)
- "Push the Gold records to HubSpot" -> `crm-export-router`

### Per-client configuration

This plugin is configured for **per-run client config**: `icp-validator` and `dataset-health-analyzer` will ask you to confirm the ICP definition and MDQS weights at the start of every run rather than reusing a global default. This is intentional for agency/multi-client use — save the completed config alongside each dataset if you want to reuse it on a later run for the same client.

## Customization

- Edit `skills/account-validator/references/industry-taxonomy.md` to match your own industry classification system.
- Edit `skills/title-taxonomy-mapper/references/title-taxonomy.md` to match your own seniority/function/buying-role definitions.
- Edit `skills/contact-validator/references/role-account-prefixes.md` to adjust role-account/personal-domain policy.
- Edit `skills/dataset-health-analyzer/references/mdqs-config-template.md` to change default MDQS weights or tier thresholds.
