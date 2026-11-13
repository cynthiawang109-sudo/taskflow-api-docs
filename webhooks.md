# TaskFlow — webhooks

Projects can subscribe an HTTPS endpoint to TaskFlow events. Delivery
is a `worker` job and follows the same retry policy as every other
job: exponential backoff, dead-letter after 5 attempts (see
`taskflow-docs`, `architecture.md`).

## Events

| Event | Fires when |
|---|---|
| `project.created` | A project is created |
| `project.deleted` | A project is soft-deleted |
| `task.created` | A task is created |
| `task.updated` | A task's title or assignee changes |
| `task.completed` | A task transitions to `done` |
| `comment.created` | A comment (or a comment revision) is posted |

## Payload

```json
{
  "id": "evt_9f2c…",
  "type": "task.completed",
  "occurred_at": "2026-11-12T14:03:11Z",
  "data": { "project_id": 12, "task_id": 841, "actor_id": 5 }
}
```

Payloads carry IDs and changed fields only — never email addresses or
other PII (see `taskflow-docs`, `security.md`). A receiver that needs
more fetches the resource through the API with its own credentials.

## Signing

Each subscription has its own secret. Every delivery sends
`X-TaskFlow-Signature: sha256=<hmac>` — an HMAC-SHA256 of the raw
request body. Receivers must verify the signature before parsing, and
reject deliveries whose `occurred_at` is more than 5 minutes old.

## Delivery semantics

- The endpoint must answer within 10 seconds; any 2xx counts as
  delivered, everything else as a failed attempt.
- Deliveries of one event to one endpoint are at-least-once: after a
  retry or an operator re-queue, a receiver may see the same event
  twice and must dedupe on the event `id`.
- A dead-lettered delivery stays in the dead-letter list until an
  operator re-queues or discards it (see `taskflow-docs`,
  `incident-runbook.md` — this is why the queue is never purged
  during an incident).
