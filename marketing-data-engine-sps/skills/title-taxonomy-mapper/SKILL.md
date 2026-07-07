---
name: title-taxonomy-mapper
description: >
  This skill should be used when the user wants to normalize job titles,
  extract seniority levels, map titles to functions/departments, identify
  buying roles (champion/decision-maker/influencer/user), or filter out
  irrelevant titles (interns, students, former employees, freelancers).
  Trigger on phrases like "normalize these titles", "map seniority", "which
  of these are decision makers", "filter out irrelevant titles", or "check
  title match against our ICP". Corresponds to Level 4 (Job Title Validation)
  — the highest-value, most error-prone validation layer, so it gets its own
  dedicated skill rather than being folded into contact-validator.
metadata:
  version: "0.1.0"
---

Normalize and classify every contact's job title. This is where junior
researchers most often introduce noise — including "Engineer" or "Developer"
titles when the ICP calls for "Director" or "VP," or including titles that
should never have been in scope at all (interns, students, former employees).

## Step 1: Title normalization

Expand abbreviations and standardize format. Example: "Vice President Information Technology" -> `VP IT`. Use `references/title-taxonomy.md` for the standard abbreviation map (VP, SVP, EVP, Dir, Sr Dir, Mgr, etc.) and apply it consistently — do not leave some titles expanded and others abbreviated in the same dataset.

## Step 2: Seniority extraction

Map every normalized title to one seniority tier: `C-Level`, `EVP`, `SVP`, `VP`, `Senior Director`, `Director`, `Associate Director`, `Senior Manager`, `Manager`, `Individual Contributor`. Keep these as separate, ordered tiers — do not collapse "Director" and "Senior Director" into one bucket, since ICP buying-committee requirements often specify a minimum tier.

## Step 3: Function extraction

Map every title to a function: IT, Security, Operations, Finance, HR, Sales, Marketing, Product, Engineering, Legal, Procurement, Customer Success, or Other. Watch for titles that name a function different from the account's department field (e.g., someone titled "Finance Director" but tagged under an IT department in the source system) — flag this as a Level 9 consistency issue for `dataset-health-analyzer` to pick up, don't silently resolve it here.

## Step 4: Buying-role mapping

Map seniority + function + (if available) ICP context to a buying role: `Champion`, `Decision Maker`, `Influencer`, or `User`. This mapping is context-dependent on the ICP's buying committee definition (see `icp-validator`'s `references/icp-template.md` — `buying_committee.required_titles`), so cross-reference that list: titles matching required buying-committee roles map to Decision Maker or Champion; adjacent functional titles map to Influencer; individual contributors map to User.

## Step 5: Keyword blacklist (reject)

Reject contacts whose title contains, as a whole-word match: Intern, Student, Advisor (unless the ICP specifically targets advisors), Retired, Volunteer, Former (or "Ex-"), Freelancer, and Founder when the ICP is not targeting founders specifically. See `references/title-taxonomy.md` for the full blacklist and edge-case notes.

Flag code: `TITLE_BLACKLIST_REJECT`.

## Step 6: Keyword whitelist / ICP title match

Compare the normalized title and seniority tier against the ICP's `required_titles`/minimum seniority (from `icp-validator`). A title like "IT Engineer" when the ICP wants "Head of IT / VP IT / Director IT" is a clear reject on seniority grounds even though the function (IT) matches.

Flag code: `TITLE_ICP_SENIORITY_NOFIT`.

## Output

Populate `job_title_normalized`, `seniority`, `function`, `buying_role` on every contact record. Append flag codes to `validation_flags`. Report a distribution summary: seniority breakdown, function breakdown, and reject count with reasons — this is the input `dataset-health-analyzer` needs for its Level 8 persona-distribution and buying-committee-coverage checks.

## Additional resources

- `references/title-taxonomy.md` — abbreviation map, seniority tier definitions, function list, and the full blacklist/whitelist keyword sets.
