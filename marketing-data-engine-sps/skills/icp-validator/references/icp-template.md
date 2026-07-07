# ICP definition template

Ask the user to fill this in (or help them fill it in) at the start of every icp-validator run. Save the completed version alongside the dataset so re-running validation later reuses the same definition rather than re-eliciting it.

```yaml
icp_name: ""                     # e.g., "Q3 Banking ABM - India"
industries:
  include: []                    # e.g., ["Banking"]
  exclude_adjacent: []           # e.g., ["NBFC", "Fintech"] — explicitly out unless stated otherwise
geography:
  countries: []
  states_or_regions: []          # optional, leave empty if country-level is sufficient
employee_count:
  min: null
  max: null
revenue:
  min: null
  max: null
  currency: ""
technology_stack:
  required_evidence: []          # e.g., ["Azure"]
funding_stage:
  applies: false
  stages: []                     # e.g., ["Series B", "Series C"]
growth_signals:
  weight: "bonus"                # "bonus" | "required"
  types: ["hiring", "funding", "expansion", "acquisition", "ipo"]
intent_signals:
  sources: []                    # e.g., ["Bombora", "G2", "6sense"]
  weight: "bonus"                # "bonus" | "required"
buying_committee:
  required_titles: []            # e.g., ["CIO", "CTO", "CISO", "IT Director", "Network Head"]
  min_contacts_per_account: 3
```

If the user only has a loose verbal description ("banks in India with 1000+ employees doing cloud adoption"), translate it into this structure and read it back to them before scoring — this avoids silently guessing at thresholds like what counts as "large" or "cloud adoption evidence."
