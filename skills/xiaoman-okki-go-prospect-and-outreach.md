---
name: xiaoman-okki-go-prospect-and-outreach
description: Prospect B2B companies and run cold outreach with the OKKI Go API —
  search companies free, unlock selected companies, pull decision-maker
  emails, send batch or personalized EDM, and track delivery. Paid actions
  (unlock, send) require explicit user confirmation.
api: openapi/xiaoman-okki-go-openapi.yml
operations:
  - searchCompaniesAdvanced
  - unlockCompany
  - getCompanyProfileEmails
  - sendBatchEmails
  - listEmailTasks
  - getEmailTask
  - getCreditBalance
generated: '2026-07-21'
method: generated
---

# OKKI Go — prospect and outreach

Distilled from the provider's own published Agent Skill
(`skills/okki-go/SKILL.md`, npm `@okki-global/okki-go`) against
`openapi/xiaoman-okki-go-openapi.yml`. Auth:
`Authorization: ApiKey sk-...` (keys from https://go.okki.ai). Shared rate
limit 60 req/min; errors are RFC 7807 (see errors/xiaoman-problem-types.yml).

## Steps

1. **Check balance** — `getCreditBalance` (free). Usable points =
   monthlyPoints + addonPoints; monthly is consumed first.
2. **Search companies** — `searchCompaniesAdvanced` (free). At least one of
   `companyTypeKeywords` / `productKeywords` / `industryKeywords`; country
   fields are filters only. Keep round 1 minimal (one keyword field +
   geography); do not invent unsupported filters. Paginate with `from`/`size`
   (max 50).
3. **Confirm, then unlock** — `unlockCompany` with `domain` + `countryCode`
   from search results. Costs 1 point on first unlock; free re-unlock for 30
   days. The returned `companyHashId` is the ONLY valid key for follow-up
   lookups — never reuse the free-search `id`. 404 = no match (not charged);
   402 = buy credits.
4. **Pull contacts** — `getCompanyProfileEmails` (free) per unlocked company;
   filter by `keyword`, paginate `page`/`pageSize` (max 100). Empty `emails`
   means no contacts available.
5. **Confirm, then send** — `sendBatchEmails` (one template, ≤100 recipients,
   1 EDM quota each; variables written bare in the template) or
   `sendPersonalizedEmails` for individual bodies. 403 = plan has no EDM
   permission; 402 refunds and aborts; 502 refunds deducted quota.
6. **Track** — record `task_id`; poll `listEmailTasks` /
   `getEmailTask` for per-mail status (`sent`/`failed`, `callbackReceivedAt`
   confirms delivery, `failureReason` explains failures).

## Guardrails (from the provider's skill)

- Never perform paid actions (unlock, send) without explicit confirmation.
- Do not substitute public web search for OKKI company results; recover
  inside the API (retry once, simplify keywords, paginate).
- `POST /api/v1/contacts/search` is retired (410) — use unlock +
  profileEmails.
