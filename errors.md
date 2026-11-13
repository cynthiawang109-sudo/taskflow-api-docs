# TaskFlow — error code catalog

For migration steps, see the [migration guide](migration-guide.md).

Every error response from the API uses the envelope from
`api-conventions.md`:
`{ "error": { "code": "...", "message": "..." } }`. Clients branch on
`code`; `message` is human-facing and may change. Codes are never
renamed or reused once shipped — new failures get new codes. The one
exception is `GET /health`: its `503` body follows the health
contract in `api-conventions.md`, not this envelope.

| Code | HTTP | Meaning |
|---|---|---|
| `unauthorized` | 401 | Missing, expired, or revoked credential |
| `forbidden` | 403 | Authenticated, but not allowed (for example, not the project owner) |
| `user_not_found` | 404 | No user with that ID |
| `project_not_found` | 404 | No (visible) project with that ID |
| `task_not_found` | 404 | No (visible) task with that ID |
| `comment_not_found` | 404 | No comment with that ID |
| `validation_error` | 422 | The request failed validation; the message never echoes submitted values |
| `invalid_status` | 422 | A task `status` value outside `todo` / `doing` / `done` |
| `email_already_registered` | 409 | Sign-up attempted with an email that already has an account |
| `conflict` | 409 | The resource's current state forbids the operation (for example, transferring a deleted project) |
| `rate_limited` | 429 | Too many requests; honor `Retry-After` (see `rate-limiting.md`) |
| `store_unavailable` | 503 | A hard dependency is unreachable mid-request (the demo's task store, in production the database); the same condition appears on `/health` as `degraded` |
| `internal_error` | 500 | Unexpected failure; the message stays generic and the detail lives in logs under the `request_id` |

Two habits keep this catalog trustworthy:

- **404 hides existence.** A task the caller may not see returns
  `task_not_found`, not `forbidden` — IDs are not a capability, and
  the API does not confirm what it will not show.
- **422 is for the request, 409 is for the world.** If fixing the
  payload fixes the call, it is a validation error; if the payload is
  fine but the resource has moved on, it is a conflict.
