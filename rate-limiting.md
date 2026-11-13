# TaskFlow — rate limiting

Limits protect PostgreSQL from runaway clients; they are not a
billing mechanism. Counters live in Redis (see `taskflow-docs`,
`system-overview.md`) and are enforced in `api-service` before any
database work happens.

## Limits by credential class

| Credential | Limit |
|---|---|
| User session token | 600 requests / minute |
| API token (`tf_live_…`) | 300 requests / minute, per token |
| Unauthenticated, per IP | 60 requests / minute |
| Login and password-reset endpoints, per IP | 10 requests / minute |

## Responses

- Exceeding a limit returns `429` with code `rate_limited` (see
  `errors.md`) and a `Retry-After` header in seconds. Clients must
  honor it; retrying early extends nothing and fixes nothing.
- Every response carries `X-RateLimit-Limit` and
  `X-RateLimit-Remaining` so well-behaved clients can slow down
  before they hit the wall.

## Exemptions and changes

- `GET /health` is exempt: load balancers and the incident runbook
  key off it (see `api-conventions.md`), and it costs no database
  work.
- A limit change is a change to this file plus review — never a
  console tweak — so the documented numbers are always the enforced
  numbers.
