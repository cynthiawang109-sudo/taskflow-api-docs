# taskflow-api-docs

API contracts for **TaskFlow** (fictional SaaS): the conventions every
TaskFlow service follows, and the data model behind them. If code disagrees
with this repo, that is a finding — raise it, don't silently pick a side.

Part of [ai-workspace-kit](https://github.com/cynthiawang109-sudo/ai-workspace-kit).
System-level docs live in the sibling repo `taskflow-docs`.

## Contents

| File | What it covers |
|---|---|
| `api-conventions.md` | Versioning, errors, pagination, health-check contract |
| `data-model.md` | Core tables and their invariants |
| `endpoints.md` | The production endpoint reference, with demo coverage marked |
| `errors.md` | The error code catalog and how to choose a code |
| `webhooks.md` | Events, payloads, signing, delivery semantics |
| `rate-limiting.md` | Limits per credential class, 429 behavior, exemptions |
| `openapi.yaml` | The demo implementation's OpenAPI spec (generated from demo-app, then curated); published to GitBook as the interactive API reference |
