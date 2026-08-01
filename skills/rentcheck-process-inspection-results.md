---
name: Process RentCheck inspection results
description: Retrieve a completed inspection, review its features/photos, flag maintenance issues, and approve.
api: openapi/rentcheck-openapi-original.yml
generated: '2026-07-20'
method: generated
operations:
  - "GET /v2/inspections"
  - "GET /v1/inspections/{inspectionId}"
  - "GET /v2/inspections/{inspectionId}/features"
  - "POST /v2/inspections/{inspectionId}/features/{featureId}/flags"
  - "POST /v1/inspections/{inspectionId}/approve"
---

# Process RentCheck inspection results

Once a resident completes an inspection, use the RentCheck REST API to review the results,
flag maintenance issues, and approve. Operations are referenced by HTTP method + path
(verified against `openapi/rentcheck-openapi-original.yml`; the spec has no operationIds).

## Preconditions
- Bearer JWT access token plus `x-app-id` / `x-app-secret` headers on every request
  (see `authentication/rentcheck-authentication.yml`).

## Steps
1. **Find completed inspections.** `GET /v2/inspections` (paginated) to list inspections;
   filter to the ones you need and capture each `inspectionId`.
2. **Load one inspection.** `GET /v1/inspections/{inspectionId}` for full detail, including the
   `inspection_template` embed.
3. **Review features/photos.** `GET /v2/inspections/{inspectionId}/features` to walk the
   time-stamped photo features room by room.
4. **Flag maintenance issues.** `POST /v2/inspections/{inspectionId}/features/{featureId}/flags`
   to raise a maintenance flag on any feature that needs attention (these can drive work orders
   via integrations such as Latchel/AppFolio).
5. **Approve.** `POST /v1/inspections/{inspectionId}/approve` to sign off the inspection.

## Conventions & error handling
- Pagination: `page_size` (max 250) + zero-based `page_number`; envelope carries `data`,
  `count`, `total_results` (see `conventions/rentcheck-conventions.yml`).
- Errors are `{ "status": <int>, "error": "<string>" }` (see
  `errors/rentcheck-problem-types.yml`): handle `403` (feature/lease not owned by caller),
  `404` (inspection or feature not found), `409` (already reviewed/scanned), and `429`.
