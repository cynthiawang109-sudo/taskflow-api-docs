# TaskFlow — data model (core tables)

| Table | Key columns | Invariants |
|---|---|---|
| `users` | `id`, `email` (unique), `display_name` | Email is the only PII column in the system; stored lower-cased |
| `projects` | `id`, `owner_id → users`, `name` | A project always has exactly one owner; ownership transfer is an explicit audited action |
| `tasks` | `id`, `project_id → projects`, `title`, `status`, `assignee_id → users` (nullable) | `status` ∈ `todo / doing / done`; transitions to `done` set `completed_at` in the same transaction |
| `comments` | `id`, `task_id → tasks`, `author_id → users`, `body` | Comments are append-only; edits create a new revision row, never an UPDATE of `body` |

## Notes for reviewers

- Deletes are soft (`deleted_at`) on `projects` and `tasks`; hard deletes are
  a data-loss finding in review.
- Any new table holding user-generated content must state its PII story in
  this file before the migration is approved.

## Demo implementation note

The demo implementation (demo-app) stores a reduced `tasks` shape (`id`,
`title`, `status`) in a local SQLite file that stands in for this table;
the production store described above remains PostgreSQL. PII story for
the demo table: `title` is user-generated free text and is the only user
content stored — the demo is fictional-data-only, keeps the file on the
developer's machine, and has no email or other PII columns. The production
invariants above (project ownership, soft deletes, `completed_at`) are
not implemented by the demo store.
