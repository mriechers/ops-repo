# <!-- TODO: Your Workspace Name -->

<!-- One-paragraph statement of what this is. The template ships with a generic version; tighten it to your project. -->

This directory is **not a project** — it is a coordination layer over multiple independent repos. Each child below is its own git repo with its own remote, CLAUDE.md, and planning/. Treat them as siblings, not subdirectories of a monorepo.

**Start here:** [`ROADMAP.md`](./ROADMAP.md) is the rolling index of in-flight, on-deck, and recently-completed work across all repos. Read it before proposing new work to avoid duplicating something already scoped.

## Sibling repos

<!-- TODO: list your child repos here. Replace the example rows. -->

| Subproject | Purpose | Remote | CLAUDE.md |
|---|---|---|---|
| `my-service-a/` | <!-- one-line description --> | `git@github.com:yourorg/my-service-a.git` | `my-service-a/CLAUDE.md` |
| `my-service-b/` | <!-- one-line description --> | `git@github.com:yourorg/my-service-b.git` | `my-service-b/CLAUDE.md` |

<!-- If you have third-party forks or other non-owned repos, separate them into a second table:

### Third-party / forks

| Subproject | Upstream | Purpose |
|---|---|---|
| `vendor-tool/` | upstream/vendor-tool | <!-- why we have this --> |

-->

## Services / hardware inventory

<!-- TODO: optional. Keep this section only if your child repos manage physical or cloud infrastructure where agents need to know names, addresses, roles, and adjacencies.

For each major component, capture:
- Hardware (or service tier) — what it runs on
- Role — what it does for the system
- Location / address — where it lives, network address, hostname
- Adjacent components — what it talks to or depends on
- Pointer to authoritative doc — typically `child-repo/docs/device-inventory.md` or similar

Delete this section if your child repos are pure software and there's no meaningful infrastructure to inventory. -->

## Why this layer exists

<!-- TODO: tighten the generic version below to match your specific failure mode. -->

Agents working on one project have historically dropped files into sibling projects' directories (e.g., an agent fixing service A writes a note into service B's repo). This causes confusion, mis-attributed work, and merge surprises. This layer gives agents an unambiguous place to put cross-project work.

## Rules for agents

1. **Stay in your project.** An agent invoked to work on `my-service-a` edits files only inside `my-service-a/`. Same for siblings. Never write into a sibling's directory.
2. **Cross-project work lives here, at the parent level.** If your work for project A requires a coordinated change in project B, write a handoff doc in `planning/` here — never inside the sibling project.
3. **Handoff filename pattern:** `planning/YYYY-MM-DD-{topic}-handoff.md` (see `planning/HANDOFF_TEMPLATE.md`).
4. **Quick cross-repo items** that don't yet justify a full handoff doc go in `planning/backlog.md` as one-liners.
5. **The parent repo is a coordination overlay, not a monorepo.** <!-- TODO: tweak based on whether this metarepo is public or private. If private: "Child repos are gitignored at the parent level and never tracked here; their content always lives in their own remotes. There is no CI on the parent — cross-repo handoff docs land direct-to-`main` rather than via PR." If public, you may want a PR workflow for handoff docs. -->

## Cross-cutting platforms

<!-- TODO: fill in if you have shared infrastructure that any agent should reuse rather than re-propose. Delete this section if not yet applicable. -->

Before proposing a new tool to solve a cross-cutting concern (observability, alerting, secrets, monitoring, auth), check whether the workspace already has an in-flight platform that absorbs it. Re-proposing tool X when tool Y is already spec'd wastes effort and accretes duplicates.

| Concern | Canonical home | Status |
|---|---|---|
| <!-- e.g. Observability --> | <!-- e.g. planning/observability-platform/ (Prometheus + Loki + Grafana on obs-host) --> | <!-- e.g. In flight as of 2026-MM-DD --> |
| <!-- e.g. Auth / SSO --> | <!-- e.g. Authelia on auth-host, planning/auth-stack-handoff.md --> | <!-- e.g. Live --> |

If a new need genuinely doesn't fit any of these, propose a fresh handoff doc rather than introducing a parallel platform. Pushback from this list is fine — but it should be a deliberate fork, not an accident.

## Team structure

For trivial, single-repo asks, work as a single agent. For anything cross-project, dispatch a **team**: the main session thread is the team leader; it dispatches subagents and integrates their work.

**When to dispatch a team:**
- Work spans 2+ child repos
- 3+ discrete deliverables
- Blast radius isn't obvious from the prompt

Single-file cross-repo asks (e.g. one config reservation that touches two services) don't need a team — write a `planning/YYYY-MM-DD-{topic}-handoff.md` instead.

**Team composition:** one **Lead** per relevant child repo, expanded as needed. Each Lead stays inside its assigned repo (rule 1 still applies). Cross-Lead coordination goes through `planning/` handoff docs — never through direct cross-repo edits.

**Safety hand-off:** if a child repo has a Guardian agent (a child-repo agent that gates risky changes inside that repo), the Lead must invoke the Guardian *before* proposing a state-change crossing a safety zone documented in that child's CLAUDE.md. This makes safety a prime concern from the start of any larger project rather than a check tacked on at the end.

**Iteration:** observations on what worked and what didn't go in `planning/team-structure-log.md`. After each team-dispatch session, append 3–5 lines.

## Workspace file

Open `<your-org>.code-workspace` (or whatever you renamed `example.code-workspace` to) to load all sibling repos as a multi-root workspace.

## Conventions inherited

<!-- TODO: optional. If you adopted this layer from another pattern source (e.g. another workspace, an internal pattern doc), name it here so future agents know where the conventions originated.

Example: "This layer mirrors the planning/handoff conventions from the metarepo-pattern template (https://github.com/mriechers/metarepo-pattern)."

Delete this section if it doesn't apply. -->
