# planning/

This folder is the **async coordination substrate** for the workspace. When work needs to cross repo boundaries, the artifact that bridges it lives here — not inside any one child repo.

## What lives here

| File | Role |
|---|---|
| `HANDOFF_TEMPLATE.md` | Canonical template for cross-repo handoff docs |
| `backlog.md` | One-line cross-repo items not yet worth a full handoff |
| `team-structure-log.md` | Append-only observation log from team-dispatch sessions |
| `<YYYY-MM-DD>-<topic>-handoff.md` | Individual cross-repo handoffs |
| `<domain-class>/` | Subfolder per cross-cutting agent class (binding contracts, session manifests) |

## Filename convention

Handoff docs follow `YYYY-MM-DD-<topic>-handoff.md`. The date is when the handoff was **written**, not when the work happens. This keeps the directory sortable and makes "what's still open?" a `ls -lt | head` rather than a query.

Topic should be terse: `caddy-on-opnsense`, `dhcp-reservation-for-ha`, `observability-rollout`. Avoid generic words like "update" or "fix"; they don't disambiguate.

## Backlog vs. handoff doc

If it fits in one line, it belongs in `backlog.md`. If it has prerequisites, acceptance criteria, or an agent log, it needs its own dated handoff doc.

| Belongs in backlog | Belongs in a handoff doc |
|---|---|
| "Service A needs a DHCP reservation that service B should set up" | "Migrate database from X to Y across services A, B, C — full plan, prerequisites, validation steps" |
| "Service A's docs reference service B's old port number" | "New observability platform spans services A, B, C — architecture, rollout phases, fan-out work" |

Backlog items graduate to handoff docs when their scope grows past one line.

## Domain-class subfolders

If a cross-cutting agent class operates in a risky domain that needs a binding contract (see [`DESIGN.md` §7](../DESIGN.md#7-the-binding-contract-pattern)), it gets its own subfolder here:

```
planning/
├── <your-domain>/
│   ├── README.md           Binding contract: invariants, refusal patterns, attestation schema
│   ├── DESIGN_BRIEF.md     Template for spinning up future agents in the class
│   ├── SESSION_LOG.md      Append-only index of all sessions in the domain
│   └── sessions/           JSONL session manifests, append-only
```

The template ships with `domain-class-example/` as a worked reference. Rename and adapt when you have a real domain; delete if you don't need one yet.
