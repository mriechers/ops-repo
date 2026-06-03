# metarepo-pattern

A template for scaffolding a **coordination layer** over N independent sibling git repos — designed for agent-driven workflows where work routinely crosses repo boundaries.

## What this is

This is not a monorepo. It is a thin parent layer that sits *above* a collection of independent git repos (each with its own remote, CI, and contributors), giving agents an unambiguous place to:

- Track in-flight cross-repo work (`ROADMAP.md`)
- Write cross-repo handoffs without polluting any single child (`planning/`)
- Define shared safety contracts, cross-cutting platforms, and team-dispatch conventions (`CLAUDE.md`)
- House cross-cutting agents and skills that operate across child repos (`.claude/`)

If you've ever had an AI agent helpfully drop a Home Assistant fix into your firewall config repo because they share a working directory, this is the pattern that prevents it.

## Why you'd use it

The failure mode this addresses: **cross-repo dependencies have no shared substrate.** Without one, agents either (a) silently scatter work across the wrong repos, (b) duplicate effort because no one knows what was already scoped, or (c) require the human operator to be the bus.

This template gives you the bus.

## Tooling assumption

The worked example uses **Claude Code** conventions (`.claude/agents/`, `.claude/skills/`, the `Skill` tool, slash commands). The structural pattern is portable — the planning/, ROADMAP, handoff-doc, and session-manifest pieces work with any agent harness. Substitute the `.claude/` directory for your harness's equivalent.

## How to use this template

1. Click **"Use this template"** on GitHub → create your new repo
2. Clone it locally as a sibling to where your child repos live (or where they'll live)
3. Follow [RECIPE.md](RECIPE.md) — step-by-step instantiation guide
4. Read [DESIGN.md](DESIGN.md) for the why and the slot-by-slot rationale

## Designed by living example

Patterns crystallized from `mriechers/homelab` — a coordination layer over five sibling repos (Home Assistant, OPNsense, Proxmox, and two third-party theme forks). The [appendix in DESIGN.md](DESIGN.md#9-appendix-worked-example-homelab) maps each slot in this template to its filled-in version there.

## License

MIT. Use freely; attribution appreciated but not required.
