---
name: contact-validator
description: >
  This skill should be used when the user wants to validate individual
  contact/person data — name formatting, email syntax and deliverability,
  role/personal email detection, duplicate people or emails, phone format,
  LinkedIn profile match, or whether the contact is still employed where the
  record says. Trigger on phrases like "validate these contacts", "check for
  duplicate people", "is this email deliverable", "check role accounts", or
  "verify these people still work there". Corresponds to Levels 3-5 (Contact,
  Job Title context, and Employment Validation) of the framework, excluding
  title/seniority mapping which is handled by title-taxonomy-mapper.
metadata:
  version: "0.1.0"
---

Run Level 3 (Contact Validation) and Level 5 (Employment Validation) checks on
every contact record. Title-specific Level 4 checks (seniority, function,
buying-role mapping) live in the separate `title-taxonomy-mapper` skill —
run that skill alongside or after this one.

## Level 3: Contact validation

### Name

Fix capitalization (`john smith` -> `John Smith`). For multi-part names, apply a clear first/last split rule and flag ambiguous cases rather than guessing silently — e.g., "Mary Anne Lee" could split as first="Mary", last="Anne Lee" or first="Mary Anne", last="Lee". Default to treating the last space-separated token as the last name and note the alternate split as a flag (`CONTACT_NAME_AMBIGUOUS_SPLIT`) so a human can confirm for high-value records.

### Gender

Only populate `gender` if explicitly present in the source data. Never infer it from a first name — this is an explicit rule, not a style preference.

### Email syntax and deliverability

Check RFC-valid syntax first. Then check, where tooling allows: MX record existence, SMTP-level deliverability signal, disposable-domain detection, catch-all domain detection.

**Role account detection** — reject/flag addresses starting with `info@`, `sales@`, `support@`, `admin@`, `office@`, or similar generic prefixes (see `references/role-account-prefixes.md` for the full list). These are not usable as named-contact emails regardless of syntax validity.

**Personal email detection** — flag addresses on `gmail.com`, `yahoo.com`, `hotmail.com`, and similar consumer domains unless the user has explicitly said personal emails are allowed for this campaign (some SMB-focused ICPs do allow this — check before rejecting).

Flag codes: `CONTACT_EMAIL_INVALID_SYNTAX`, `CONTACT_EMAIL_UNDELIVERABLE`, `CONTACT_EMAIL_ROLE_ACCOUNT`, `CONTACT_EMAIL_PERSONAL`, `CONTACT_EMAIL_CATCHALL`.

### Duplicate detection

**Duplicate emails** — exact match across the dataset.

**Duplicate people with different emails** — fuzzy-match on name + account (e.g., "John Smith" with `john@acme.com` and `john.smith@acme.com` at the same account is very likely the same person). Use name similarity + shared domain + shared account_id as the matching signal; do not auto-merge, flag for review and link the duplicate `contact_id`s together.

Flag codes: `CONTACT_DUP_EMAIL_EXACT`, `CONTACT_DUP_PERSON_FUZZY`.

### Phone number

Check format (country code present, plausible length for the stated country). Flag phone numbers whose country code doesn't match the contact's stated location (see also the Level 9 consistency check in `dataset-health-analyzer`).

Flag code: `CONTACT_PHONE_FORMAT_INVALID`.

### LinkedIn URL

Confirm the URL exists and, where feasible, that it plausibly matches the named person (name in URL/profile matches record). `profile_photo_exists` is optional and only used as a soft confidence signal, never a rejection reason on its own.

Flag code: `CONTACT_LINKEDIN_MISMATCH`.

## Level 5: Employment validation

Apollo and similar sources are frequently stale. Cross-check `current_employer` (from the account record this contact is linked to) against the most recent enrichment source (ideally LinkedIn Sales Navigator, since it tends to be freshest — see `data-enrich`). If the enrichment source shows a different current employer than the account link, this is a job change: the contact record should be flagged and, in most cases, re-associated with the new employer's account rather than kept under the stale one.

Flag code: `CONTACT_EMPLOYER_STALE` (job change detected — re-link or reject depending on user preference).

## Output

Append flag codes to `validation_flags`. Set `contact_quality_status` to `pass`, `review`, or `reject` — role accounts, invalid syntax, and confirmed job changes to an unrelated company are reject-tier by default; everything else is review-tier. Report a flag-count summary and recommend `title-taxonomy-mapper` next.

## Additional resources

- `references/role-account-prefixes.md` — the full role-account and personal-domain reference lists.
