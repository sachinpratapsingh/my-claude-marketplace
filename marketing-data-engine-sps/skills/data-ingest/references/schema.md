# Canonical data schema

Every skill in this plugin reads and writes records that conform to this schema. Fields marked "mandatory" feed the Level 6 completeness check and the MDQS completeness sub-score — do not silently drop them during any transformation.

## Account record

| Field | Type | Mandatory | Notes |
|---|---|---|---|
| account_id | string | yes | Stable internal ID assigned at ingest |
| company_name_raw | string | yes | As received from source |
| company_name_normalized | string | yes | After Level 1 normalization (legal suffix stripped/standardized, casing fixed) |
| website | url | yes | |
| domain | string | yes | Derived from website |
| industry_raw | string | no | As received |
| industry_normalized | string | yes | Mapped to the approved taxonomy |
| employee_count_apollo | int | no | |
| employee_count_linkedin | int | no | |
| employee_count_website | int | no | |
| revenue | number | no | |
| revenue_currency | string | no | ISO code |
| hq_country | string | yes | |
| hq_city | string | no | |
| hq_region | string | no | |
| company_status | enum | yes | active / closed / merged / acquired / stealth / dormant |
| ownership_type | enum | no | public / private |
| parent_company | string | no | For ABM hierarchy rollups |
| subsidiaries | array | no | |
| stock_ticker | string | no | Required if ownership_type = public |
| linkedin_company_url | url | yes | |
| source_system | enum | yes | excel / airtable / hubspot / salesforce / demandbase / 6sense / recotap / bamboobox |
| source_record_id | string | yes | |
| ingested_at | date | yes | |
| last_updated | date | no | Freshness signal — from source system if available |

## Contact record

| Field | Type | Mandatory | Notes |
|---|---|---|---|
| contact_id | string | yes | |
| account_id | string | yes | FK to account |
| first_name_raw | string | yes | |
| last_name_raw | string | yes | |
| first_name_normalized | string | yes | Proper-case, correctly split from full name |
| last_name_normalized | string | yes | |
| gender | enum | no | Only populate if explicitly provided by source. Never infer. |
| email | string | yes | |
| email_type | enum | yes (post-validation) | corporate / personal / role / disposable / catch-all |
| email_deliverability_status | enum | yes (post-validation) | valid / invalid / unknown / risky |
| phone | string | no | |
| phone_country_code | string | no | |
| linkedin_url | url | yes | |
| job_title_raw | string | yes | |
| job_title_normalized | string | yes (post-validation) | See title-taxonomy-mapper |
| seniority | enum | yes (post-validation) | C-Level / VP / Director / Senior Director / Associate Director / Manager / IC |
| function | string | yes (post-validation) | IT / Security / Finance / HR / Operations / Sales / Marketing / etc. |
| department | string | no | |
| buying_role | enum | no | Champion / Decision Maker / Influencer / User |
| current_employer | string | no | For employment/job-change verification |
| employment_start_date | date | no | |
| profile_photo_exists | boolean | no | Confidence signal only |
| source_system | enum | yes | Same enum as account |
| source_record_id | string | yes | |
| ingested_at | date | yes | |
| last_updated | date | no | |

## Fields added by downstream skills (do not populate at ingest)

| Field | Added by |
|---|---|
| confidence_score, completeness_score, accuracy_score, relevance_score, freshness_score, consistency_score | dataset-health-analyzer |
| mdqs_score, mdqs_tier (Gold/Silver/Review/Reject) | dataset-health-analyzer |
| validation_flags (array of issue codes) | account-validator, icp-validator, contact-validator, title-taxonomy-mapper |
| enrichment_source (per field: linkedin_sales_nav / apollo / lusha / original) | data-enrich |
