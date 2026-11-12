# TaskFlow — API conventions

These conventions apply to every TaskFlow HTTP service. They exist so that
clients, reviewers, and tooling can rely on one predictable shape.

## Versioning

- URL-prefixed major versions: `/v1/...`. A breaking change ships as a new
  major version; the old one keeps working for at least one release cycle.

## Errors

- Errors are JSON: `{ "error": { "code": "task_not_found", "message": "..." } }`.
- `code` is a stable machine-readable string; `message` is human-facing and
  may change. Clients must branch on `code`, never on `message`.
- HTTP status mirrors the error class (404 unknown resource, 409 conflict,
  422 validation) — never return 200 with an error body.

## Pagination

- Cursor-based for all list endpoints: `?cursor=<opaque>&limit=<n>`
  (default 50, max 200). Responses include `next_cursor` (null at the end).
- Offset pagination is not used: it skips or repeats rows under concurrent
  writes.

## Health checks

**Every service must expose `GET /health`** with this exact contract:

- `200` and `{ "status": "ok" }` when the service can serve traffic.
- `503` and `{ "status": "degraded", "checks": { "<dependency>": "down" } }`
  when a hard dependency (database, queue) is unreachable.
- The endpoint must not require authentication and must not leak version,
  host, or dependency details beyond the check names.
- Load balancers and the incident runbook key off this contract; a service
  whose `/health` deviates is a review finding, not a style nit.
