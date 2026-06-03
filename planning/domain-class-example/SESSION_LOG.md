# Session Log — `<domain-class>`

Append-only index of every session in this domain class. Newest at the top.

Each row distills the corresponding session manifest's session-header record into a human-skimmable form. The full manifest (with per-record rows and complete attestation fields) lives at `sessions/<session-id>.jsonl`.

## Safety attestation codes

The Safety column uses compact codes for the attestation state. Common codes:

- `BACKUP` — pre-operation backup verified
- `RO` — read-only operation (no mutations)
- `PLAN` — plan-only stage (no execute)
- `VERIFIED` — per-row verification ran on all rows
- `ROLLBACK` — rollback procedure documented in manifest
- `APPROVED:<token>` — operator-approved via the named token
- `—` — field intentionally not applicable
- `MISSING` — field expected but absent (non-compliance signal)

A row with `MISSING` in the Safety column is a compliance failure — investigate before invoking the agent again.

## Log

| Session ID | Agent | Operation | Source | Target | Items | Size | Safety | Status | Notes |
|---|---|---|---|---|---|---|---|---|---|
| <!-- e.g. 2026-MM-DDTHHmm-example-agent-scan-prod --> | <!-- example-agent --> | <!-- scan --> | <!-- /source/prefix --> | <!-- — --> | <!-- 1234 --> | <!-- — --> | <!-- RO --> | <!-- complete --> | <!-- first run --> |

<!-- TODO: replace the example row above with your real first session, or leave the row as-is until you have one. -->
