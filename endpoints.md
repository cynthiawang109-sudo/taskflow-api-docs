# TaskFlow — endpoint reference (production API)

All paths are under `/v1` (see `api-conventions.md` for versioning,
the error envelope, and pagination). Authentication is bearer-token
based (see `taskflow-docs`, `authentication.md`). The **Demo** column
marks the endpoints the workspace's demo implementation (`demo-app`)
implements — a subset, over the reduced task shape described in
`data-model.md`. The demo's OpenAPI spec is `openapi.yaml` in this
repo.

| Method | Path | Purpose | Demo |
|---|---|---|---|
| GET | `/health` | Health contract (unversioned; see `api-conventions.md`) | ✓ |
| GET | `/v1/users/me` | The current user's profile | |
| GET | `/v1/projects` | List projects the caller owns or is assigned into (cursor-paginated) | |
| POST | `/v1/projects` | Create a project | |
| GET | `/v1/projects/{id}` | Fetch one project | |
| PATCH | `/v1/projects/{id}` | Rename a project | |
| POST | `/v1/projects/{id}/transfer` | Transfer ownership (audited, MFA-gated) | |
| DELETE | `/v1/projects/{id}` | Soft-delete a project | |
| GET | `/v1/tasks` | List tasks; filters: `project_id`, `status`; cursor-paginated | ✓ |
| POST | `/v1/tasks` | Create a task | ✓ |
| GET | `/v1/tasks/{id}` | Fetch one task | ✓ |
| PATCH | `/v1/tasks/{id}` | Update title, status, or assignee; `status` → `done` sets `completed_at` | |
| DELETE | `/v1/tasks/{id}` | Soft-delete a task | |
| GET | `/v1/tasks/{id}/comments` | List comments (cursor-paginated, oldest first) | ✓ |
| POST | `/v1/tasks/{id}/comments` | Add a comment | ✓ |
| GET | `/v1/tasks/{id}/comments/{comment_id}` | Fetch one comment | ✓ |

Notes:

- Comments are append-only: what looks like an edit in the UI posts a
  new revision row that supersedes the previous one (see
  `data-model.md`). There is deliberately no `PATCH` on comments. The demo
  implements all three comments operations, including the chain-head
  rule: superseding an already-superseded revision returns
  409 `conflict`.
- The demo's task endpoints accept the `status` filter and cursor
  pagination exactly as specified in `api-conventions.md`; project
  scoping is production-only.
