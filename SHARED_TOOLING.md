# Shared Tooling Inventory

Descriptive (not prescriptive) inventory of agent, skill, and script overlap across child repos. The goal: surface consolidation candidates with effort/risk labels, so future decisions about "should this become a shared tool?" have a written-down starting point.

> **When to fill this in.** Once you have at least two child repos and notice the same patterns appearing in both. Before that, this doc is empty overhead.

## How to read this doc

Each row identifies a category of tooling that appears in multiple child repos, captures the variants per repo, and labels the consolidation prospect:

- **Effort** — rough estimate of consolidating: trivial / small / medium / large
- **Risk** — what could go wrong with consolidation: low / medium / high
- **Verdict** — keep separate / consolidation candidate / consolidate now / out of scope

Verdicts can change as the workspace evolves. The doc is descriptive — labeling something a "consolidation candidate" doesn't commit anyone to doing it.

## Inventory

<!-- TODO: replace this example row with your real inventory once you have one. -->

| Tooling category | Variants | Effort | Risk | Verdict | Notes |
|---|---|---|---|---|---|
| <!-- e.g. backup-log pattern --> | <!-- service-a uses script X; service-b uses Guardian-agent attestation Y --> | <!-- small --> | <!-- low --> | <!-- consolidation candidate --> | <!-- e.g. revisit once both have run 10+ backups --> |

## Categories to watch for

When auditing your child repos for shared-tooling candidates, common categories include:

- **Pre-commit hooks** — secrets detection, formatting, linting
- **Backup / snapshot patterns** — periodic exports, ZFS snapshots, database dumps
- **Health-check scripts** — service reachability, cert expiry, disk space
- **Documentation crawlers** — refresh scripts that pull docs from external sources
- **Guardian agents** — risk-zone classifiers, approval gates
- **Doc-sync agents** — promotion of docs from `planning/` to canonical locations
- **Backup-log attestations** — schema for "this operation was preceded by a backup"
- **CI workflows** — lint, typecheck, test patterns that repeat across repos

This list is illustrative, not exhaustive. Add categories as they appear in your workspace.

## When to actually consolidate

Recognize that consolidating shared tooling adds **its own** maintenance surface (a separate repo or shared package, a sync mechanism between children, version coupling). Don't consolidate until:

1. You've maintained the variant in 3+ places long enough to be confident the abstraction is correct
2. Drift between variants is causing real pain (a fix landed in one but not another, leading to a near-miss)
3. The consolidated form will be no more brittle than the variants

If any of those is no, "consolidation candidate" is the right verdict; "consolidate now" is premature.
