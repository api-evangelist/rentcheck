---
name: Schedule a RentCheck inspection
description: Authenticate, pick an inspection template, create an inspection for a unit/resident, and send a reminder.
api: openapi/rentcheck-openapi-original.yml
generated: '2026-07-20'
method: generated
operations:
  - "POST /v1/oAuth2/emailPasswordAuth"
  - "POST /v1/oAuth2/token"
  - "GET /v1/inspection-templates"
  - "POST /v2/inspections"
  - "POST /v1/inspections/reminders"
---

# Schedule a RentCheck inspection

Use the RentCheck REST API (`https://prod-public-api.getrentcheck.com`) to schedule a
resident-led inspection. The spec declares no operationIds, so operations are referenced by
HTTP method + path (verified against `openapi/rentcheck-openapi-original.yml`).

## Preconditions
- You have an `x-app-id` and `x-app-secret` from the RentCheck API integration page
  (`app.getrentcheck.com/account/integrations/rentcheck-api`).
- Every request sends three headers: `Authorization: Bearer <jwt>`, `x-app-id`, `x-app-secret`
  (see `authentication/rentcheck-authentication.yml`).

## Steps
1. **Authenticate.** `POST /v1/oAuth2/emailPasswordAuth` (or `POST /v1/oAuth2/token`) to obtain
   a Bearer JWT access token. Refresh with `POST /v1/oAuth2/refreshToken` when it expires.
2. **Choose a template.** `GET /v1/inspection-templates` to list available inspection templates;
   capture the template id you want (move-in, move-out, routine, maintenance).
3. **Create the inspection.** `POST /v2/inspections` with the unit/resident references and
   `inspection_template_id` (preferred; `inspection_type` is deprecated as of 2024-03-07).
4. **Nudge the resident.** `POST /v1/inspections/reminders` to send reminders so the resident
   completes the guided inspection from their phone.

## Conventions & error handling
- Paginate list calls with `page_size` (max 250) and zero-based `page_number`; read
  `count` / `total_results` from the envelope (see `conventions/rentcheck-conventions.yml`).
- Errors return `{ "status": <int>, "error": "<string>" }` (not RFC 9457) — see
  `errors/rentcheck-problem-types.yml`. Handle `401` (bad token / app creds), `404`
  (unit or template not found), and `429` (rate limit: 8/s, 256/min, 1024/10-min).
- There is no general Idempotency-Key contract; do not blindly retry `POST /v2/inspections`.
