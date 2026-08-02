---
name: xiaoman-crm-webhook-subscriptions
description: Configure Xiaoman OKKI CRM message-push webhooks — register a
  callback URL, subscribe to module events (create/update/delete), test the
  callback, and rotate the encryption secret. Pro plan only.
api: openapi/xiaoman-openapi.yml
operations:
  - POST /v1/oauth2/access_token
  - POST /v1/webhook
  - POST /v1/webhook/test
  - POST /v1/subscribe
  - GET /v1/subscribe
  - POST /v1/webhook/regenerate-secret
generated: '2026-07-21'
method: generated
---

# Xiaoman OKKI CRM — webhook message-push setup

Message push is available on the **Pro** plan only (Smart plan unsupported).
The spec carries no operationIds — steps reference METHOD + path from
`openapi/xiaoman-openapi.yml`. Event catalog:
`asyncapi/xiaoman-crm-webhooks.yml`.

## Steps

1. **Get a webhook-scoped token** — `POST /v1/oauth2/access_token` with
   `scope=webhook` (add other module scopes as needed).
2. **Register the callback** — `POST /v1/webhook` (创建回调配置) with your
   HTTPS callback URL; `PUT /v1/webhook` updates it, `GET /v1/webhook` reads
   it, `DELETE /v1/webhook` removes it.
3. **Verify delivery** — `POST /v1/webhook/test` (测试回调接口) fires a test
   callback at your endpoint before going live.
4. **Subscribe to events** — `POST /v1/subscribe` (创建订阅配置) with
   `subscribe_type` (one module per subscription, e.g. `crm.order`) and
   `subscribe_configs[].biz_type` event names (e.g.
   `xiaoman.crm.order.create`). Multiple events of the same module may be
   subscribed together. Manage with `GET /v1/subscribe`,
   `GET|PUT|DELETE /v1/subscribe/{subscribe_type}`.
5. **Rotate secrets** — `POST /v1/webhook/regenerate-secret`
   (重新生成加密密钥) regenerates the payload-encryption secret; update your
   verifier immediately after rotation.

## Event payloads

Events deliver record identifiers per module (e.g. `crm.order` →
`order_id, order_no`; `crm.company` → `company_id, serial_id`) — fetch the
full record via the module's detail endpoint afterwards.
