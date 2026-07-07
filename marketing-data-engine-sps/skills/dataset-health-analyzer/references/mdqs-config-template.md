# MDQS scoring config template

Confirm this with the user at the start of every dataset-health-analyzer run — do not silently reuse a config from a previous client/project.

```yaml
mdqs_config_name: ""              # e.g., "Acme Corp - Q3 ABM run"
weights:
  account_quality: 20             # must sum to 100
  icp_match: 20
  contact_quality: 20
  job_title_persona_accuracy: 20
  freshness_consistency: 20
tiers:
  gold_min: 95                    # auto-import
  silver_min: 90                  # import with monitoring
  review_min: 80                  # manual review required
  # below review_min = reject, send back for research
freshness:
  flag_after_months: 12
  reject_after_months: 24
personal_email_policy: "flag"     # "flag" | "allow" | "reject"
duplicate_merge_policy: "surface_only"   # this skill never auto-merges; always surface for human resolution
```

## Notes on customizing weights

Agencies running this across multiple clients should expect to fill this out per client rather than using one global config — a bank ABM campaign might weight ICP Match higher (structured, narrow target), while a broad SMB campaign might weight Contact Quality (email/phone deliverability) higher since volume matters more than precision. Always ask which dimension matters most for this specific dataset before defaulting to the even 20/20/20/20/20 split.
