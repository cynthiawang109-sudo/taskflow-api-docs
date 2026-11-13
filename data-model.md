# TaskFlow — data model (core tables)

| Table | Key columns | Invariants |
|---|---|---|
| `users` | `id`, `email` (unique), `display_name` | Email is the only PII column in the system; stored lower-cased |
| `projects` | `id`, `owner_id → users`, `name` | A project always has exactly one owner; ownership transfer is an explicit audited action |
| `tasks` | `id`, `project_id → projects`, `title`, `status`, `assignee_id → users` (nullable) | `status` ∈ `todo / doing / done`; transitions to `done` set `completed_at` in the same transaction |
| `comments` | `id`, `task_id → tasks`, `author_id → users`, `body` | Comments are append-only; edits create a new revision row, never an UPDATE of `body` |

## Table detail

### `users`

- Index: unique on `lower(email)` — uniqueness is case-insensitive
  because the column is stored lower-cased.
- `display_name` is a user-chosen public label shown on boards; the
  email remains the only PII column.

### `projects`

- Index: `(owner_id, id)` backs the project list.
- Soft delete sets `deleted_at`; deleted projects disappear from list
  endpoints, and their tasks go with them. Soft-deleted rows are
  retained — the application never hard-deletes them.

### `tasks`

- Indexes: `(project_id, status, id)` backs board paging (the cursor
  is the last seen `id`; see ADR-0002 in `taskflow-docs`), and
  `(assignee_id, status)` backs the "my tasks" view.
- Reopening a task (`done` → `doing`) clears `completed_at` in the
  same transaction that changes the status.
- List queries exclude rows with `deleted_at` set, by default and in
  every client we ship.

### `comments`

- Index: `(task_id, id)` — comments render oldest first.
- A revision row carries `supersedes_id` pointing at the row it
  replaces; readers render the head of each chain. The original text
  is never updated in place, which is what makes the append-only
  invariant auditable.

## Notes for reviewers

- Deletes are soft (`deleted_at`) on `projects` and `tasks`; hard deletes are
  a data-loss finding in review.
- Any new table holding user-generated content must state its PII story in
  this file before the migration is approved.

## Demo implementation note

The demo implementation (demo-app) stores a reduced `tasks` shape (`id`,
`title`, `status`) in a local SQLite file that stands in for this table;
the production store described above remains PostgreSQL. PII story for
the demo tables: `tasks.title` and `comments.body` are user-generated
free text, and they are the only user content stored — the demo is
fictional-data-only, keeps the file on the
developer's machine, and has no email or other PII columns. Comment
bodies are never echoed in error messages and never written to logs;
because comments are append-only, a comment's full revision history
is retained in the store, and the demo has no deletion path. The production
invariants above (project ownership, soft deletes, `completed_at`) are
not implemented by the demo store.

The demo also carries the worker's `digest_sends` ledger (`user_id`,
`week`, `status`, `attempts`, `last_error`, `sent_at`). PII story:
recipients are identified by user ID only — there is deliberately no
email column — and `last_error` holds only a sanitized error code (the
failure's exception class name), never the exception message, message
content, or an address.
