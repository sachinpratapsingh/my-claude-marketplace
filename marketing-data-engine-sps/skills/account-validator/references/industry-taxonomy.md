# Industry taxonomy and common misclassification patterns

This is a starting taxonomy. Treat it as editable — if the user has a house taxonomy (e.g., matching their CRM's picklist), replace this file's categories with theirs at the start of a project rather than forcing their data into this default list.

## Top-level categories (default taxonomy)

- Banking
- NBFC (Non-Banking Financial Company)
- Fintech
- Insurance
- IT Services
- Software / SaaS
- Technology Hardware
- Telecommunications
- Healthcare Providers
- Healthcare Technology
- Pharmaceuticals
- Manufacturing
- Retail
- E-commerce
- Food & Beverage
- Food Delivery / Quick Commerce
- Logistics & Supply Chain
- Energy & Utilities
- Government / Public Sector
- Education
- Media & Entertainment
- Professional Services / Consulting
- Real Estate & Construction
- Automotive
- Travel & Hospitality

## Common misclassification patterns to check for

- **IT Services vs Software**: A company that builds and sells its own software product is "Software / SaaS." A company that provides staffing, custom development, or consulting services to other companies' IT needs is "IT Services." These are frequently swapped by junior researchers (example: TCS is IT Services, not Software).
- **Banking vs NBFC vs Fintech**: A bank has a banking license and takes deposits. An NBFC provides lending/financial services without a full banking license. A fintech is typically a technology company enabling financial services (often a vendor to banks/NBFCs, not one itself). Do not treat these as interchangeable unless the ICP explicitly includes all three.
- **Conglomerates with multiple business lines**: Tag the account by the business line actually being targeted, not the parent conglomerate's dominant industry. Example: a conglomerate best known for retail that also runs a food delivery arm should be tagged by whichever line the contact/account actually belongs to, not defaulted to the parent's primary industry.
- **Healthcare Providers vs Healthcare Technology**: A hospital chain is a Provider. A company selling software/IT to hospitals is Healthcare Technology. Different ICP fit even though both say "healthcare."
