# Domain-Class Binding Contract — Example

This folder is a **worked example** of the binding-contract pattern (see [`DESIGN.md` §7](../../DESIGN.md#7-the-binding-contract-pattern)). It's domain-agnostic on purpose — rename and adapt it for your specific class of risky multi-step work, or delete the folder if you don't yet have one.

> **Common domain classes that need a binding contract:**
> - Bulk file operations on shared storage (homelab's reference impl)
> - Mass deploys across services
> - Paid third-party API calls with cost or rate-limit risk
> - Database schema migrations across services
> - Mass data exports or imports between systems

## What this folder establishes

Three things bound together:

1. **An agent class definition** — a reusable shape (typically a four-mode pipeline: scan → classify → plan → execute) that any agent in this domain follows
2. **A safety contract** — the invariants every session must honor, expressed as attestation fields on a session-header record
3. **A coordination protocol** — append-only session manifests + a session log that together form a queryable audit trail

## Invariants

Every agent operating in this domain class must honor:

<!-- TODO: tighten these to your domain. The list below is illustrative. -->

1. **Source/target allowlists** — operations are restricted to hard-coded path or resource prefixes. Agents refuse operations outside these prefixes.
2. **Read-only stages are idempotent** — `scan`, `classify`, and `plan` stages can re-run without side effects. Only `execute` mutates state.
3. **Execute is approval-gated** — the `execute` stage requires `--approved <manifest>` (or an equivalent gate) referencing a manifest produced by a prior `plan` run.
4. **Per-row verification** — each mutating step verifies its result before moving to the next row. No fire-and-forget batches.
5. **Append-only manifests** — manifest rows are never edited in place. Corrections append a new row with `operation: superseded` referencing the row being replaced.

## Refusal patterns

Agents in this class must **refuse to proceed** (not warn-and-continue) when:

- Session-header attestation fields are missing or set to null
- Source or target paths are outside the allowlisted prefixes
- The approved manifest contains any rows in a `conflict-review` or unresolved state
- The agent detects content that would violate a safety rule (e.g. secrets in files about to be copied to a less-restricted location)

Refusals are loud — they print the reason and exit non-zero. Silent partial-compliance is the failure mode this pattern is designed to prevent.

## Attestation schema

Every session manifest's first line is a `record_type: "session-header"` JSON object carrying the safety attestation fields.

<!-- TODO: define your domain's required attestations. The fields below are illustrative — for the homelab reference impl, they map to the 8 parent safety rules. -->

| Field | Type | Meaning |
|---|---|---|
| `session_id` | string | timestamp-slug, unique |
| `agent` | string | which agent ran this session |
| `agent_class` | string | which domain class (this folder's name) |
| `invoker` | string | human or upstream agent who invoked |
| `started_at` | ISO-8601 string | session start time |
| `operation_summary` | string | one-line "what this session does" |
| `source_prefixes` | array of strings | allowed source paths/resources |
| `target_prefixes` | array of strings | allowed target paths/resources |
| `snapshot_coverage` | string | how the source state was snapshotted before mutation |
| `validated_via` | string | how each operation's result is verified |
| `rollback_plan` | string | how this session's effects can be reversed |
| `approval_state` | string | `dry-run` \| `pending-approval` \| `approved-via-<token>` |
| `secret_scope_check` | string | what's been done to prevent secret leakage |

A missing or null required field is a non-compliance signal — the `execute` stage refuses to run; the audit query flags the session.

## Per-record schema

Lines 2…N of the manifest are `record_type: "file"` (or `record_type: "<your-domain-item>"`) records, one per item the session touched:

```json
{
  "record_type": "file",
  "source_path": "/path/to/source/item",
  "target_path": "/path/to/destination/item",
  "operation": "copied | moved | skipped-dup | conflict-review | quarantined | superseded",
  "result": "ok | error | pending",
  "sha256": "abc123...",
  "size_bytes": 1234567,
  "...domain-specific fields..."
}
```

## Compliance audit

Because the manifests are JSONL with structured fields, audit queries are one-liners.

```bash
# Census: what's been done across this domain in total?
jq -c 'select(.record_type == "file")' planning/<your-domain>/sessions/*.jsonl | wc -l

# Compliance: sessions missing required attestation fields
jq -c 'select(.record_type == "session-header" and (.snapshot_coverage == null or .approval_state == null))' planning/<your-domain>/sessions/*.jsonl

# Provenance: which session moved this specific file?
jq -c 'select(.record_type == "file" and .source_path == "/path/of/interest")' planning/<your-domain>/sessions/*.jsonl
```

These queries are cheap. They run in seconds over hundreds of sessions. The structure is designed so that compliance can be verified without re-walking the underlying substrate (8 TB of files, thousands of API calls, etc.).

## Spinning up a new agent in this class

See [`DESIGN_BRIEF.md`](DESIGN_BRIEF.md) — a template that constrains creative work to the **domain-specific axes** while inheriting all the invariants, refusal patterns, and schemas defined above.
