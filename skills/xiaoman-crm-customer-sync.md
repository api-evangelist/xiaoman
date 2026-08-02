---
name: xiaoman-crm-customer-sync
description: Sync customers between an external system and Xiaoman OKKI CRM —
  authenticate with a module-scoped OAuth2 token, page through customers,
  dedupe, and upsert customer records with contacts.
api: openapi/xiaoman-openapi.yml
operations:
  - POST /v1/oauth2/access_token
  - GET /v1/company/list
  - GET /v1/company/info
  - GET /v1/company/query
  - POST /v1/company/pushCompanyAndCustomers
generated: '2026-07-21'
method: generated
---

# Xiaoman OKKI CRM — customer sync

Ground rules from the provider docs (open.xiaoman.cn): base URL
`https://api-sandbox.xiaoman.cn`, UTF-8, 60s connect/response timeouts.
The published spec carries no operationIds — steps reference METHOD + path
from `openapi/xiaoman-openapi.yml`.

## Steps

1. **Get a token** — `POST /v1/oauth2/access_token` with
   `grant_type=client_credentials`, `client_id`, `client_secret`, and
   `scope=company` (space-separate extra module scopes if needed). Cache the
   Bearer token: it lives 8 hours (`expires_in: 28800`); client_credentials
   returns no refresh_token, so re-request before expiry. Calling an API whose
   module scope you did not request fails with `no permission <module> scopes`.
2. **Page through customers** — `GET /v1/company/list` with
   `start_index` (page number, default 1) and `count` (page size). Use
   `time_type` + `start_time`/`end_time` (YYYY-MM-DD) for incremental syncs
   (time_type 1 = updated, 2 = created, 3 = filed); `removed=1` fetches
   deleted records; `all` selects private (0) / all (1) / public-pool (2)
   customers.
3. **Check for duplicates before creating** — `GET /v1/company/query`
   (客户查重) with the candidate identifiers.
4. **Upsert** — `POST /v1/company/pushCompanyAndCustomers`
   (客户（含联系人）新增/编辑) creates or edits a customer including contacts;
   consult the per-operation error-code table (异常码) on
   https://open.xiaoman.cn/api-3478272 for remediation. Field ids come from
   the data dictionary `GET /v1/company/fields` (客户数据字典).
5. **Read back details** — `GET /v1/company/info` by id to verify.

## Conventions

- Envelope: `{code, message, now, data}` — treat non-success `code` per the
  operation's 异常码 table (see errors/xiaoman-problem-types.yml).
- No idempotency keys: the upsert endpoint is keyed on record id — always
  dedupe (step 3) before create to avoid duplicates.
