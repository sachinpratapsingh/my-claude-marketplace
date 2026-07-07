# Role account prefixes and personal email domains

## Role account prefixes (reject as named-contact emails)

info, sales, support, admin, office, contact, hello, help, enquiries, inquiries, marketing, hr, careers, jobs, press, media, billing, accounts, noreply, no-reply, webmaster, postmaster, abuse, security, privacy, legal, feedback, service, team, general

Match case-insensitively against the local part before the `@`. A prefix match like `sales@acme.com` is a reject regardless of what follows.

## Personal/consumer email domains (flag unless explicitly allowed)

gmail.com, yahoo.com, hotmail.com, outlook.com, live.com, aol.com, icloud.com, protonmail.com, mail.com, yandex.com, gmx.com, zoho.com (note: zoho.com is also a legitimate business tool but the free consumer mail tier uses the same domain — verify context before flagging)

Ask the user, once per project, whether personal emails are acceptable for this campaign (common for SMB/prosumer ICPs, rare for enterprise ABM). Default to flagging as review-tier, not auto-reject, since some legitimate small businesses only use personal email domains.

## Disposable/temporary email domains (always reject)

mailinator.com, guerrillamail.com, tempmail.com, 10minutemail.com, throwawaymail.com, yopmail.com — and any domain matching common disposable-email patterns. These should always be a hard reject regardless of campaign type.
