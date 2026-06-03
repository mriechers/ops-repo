# The Metarepo Coordination Pattern

A design doc for scaffolding a parent layer that supervises N independent sibling git repos, with built-in patterns for agent communication, async cross-repo handoffs, and cross-cutting domain agents.

> **Tooling note.** This doc speaks Claude Code (`.claude/agents/`, `.claude/skills/`, the `Skill` tool). The structural and communicational layers are harness-agnostic; substitute your own agent layer if you use Codex, Gemini CLI, or another framework. The operational layer is where Claude Code-specific assumptions concentrate.

---

## 1. Context & motivation

When a single working directory contains multiple independent git repos (siblings, not submodules), agents working in that environment routinely cause three failures:

1. **Scatter.** An agent asked to fix repo A writes a note, config, or test fixture into repo B because they share a working directory. The mis-attributed change rides B's next commit and silently breaks B's invariants — or worse, accumulates as untracked drift across both.
2. **Duplication.** Without a shared index of what's already scoped, two agents (or the same agent on two different days) re-propose the same workstream. Sometimes they even land conflicting designs because neither knew the other existed.
3. **Operator-as-bus.** With no shared substrate, the human operator becomes the only thing connecting work across repos. They forward the context, paste the relevant files, restate the constraints. This doesn't scale past a handful of repos.

This pattern fixes those failures with three layers:

| Layer | Solves | Artifacts |
|---|---|---|
| **Structural** | scatter — agents know which directory their work belongs in | `CLAUDE.md`, sibling-repo table, rules-for-agents |
| **Communicational** | duplication — there's one place to look for "is this already in flight" | `ROADMAP.md`, `planning/` handoffs, backlog, team-structure-log |
| **Operational** | operator-as-bus — agents communicate via documents, not chat | session manifests, cross-cutting agents, the binding-contract pattern |

## 2. When to use this pattern

**Use it when:**
- You have 2+ independent git repos that share concerns (infrastructure, deployment surface, security posture, shared data)
- Work routinely crosses repo boundaries — a feature in repo A requires a coordinated change in repo B
- You're driving the work with AI agents (or a mix of agents and humans), and the agents need a shared substrate
- You have at least one **cross-cutting concern** — observability, safety, file ops, deploys — that touches every child repo

**Don't use it when:**
- You're working in a single repo (the parent layer is overhead)
- You have a true monorepo with one CI pipeline and one release cadence (use the monorepo's existing structure)
- Your repos are connected via git submodules and you're already happy with that (this pattern is the orthogonal alternative — coordination overlay vs. nested checkout)
- You only have 2 repos with a clean, well-documented API between them (the overhead isn't worth it; just write that API doc)

**Scaling notes.** This pattern is well-tested in the 3–7 child repo range. Past ~15, the supervisor agent's context starts to fragment and the ROADMAP becomes a wall of text. The "single supervising agent dispatching per-repo Leads" pattern probably wants to become "domain-clustered supervisors" past that point — out of scope here.

## 3. The pattern at a glance

```
                       ┌─────────────────────────────────────┐
                       │  Metarepo (this template)           │
                       │                                     │
                       │  Structural:                        │
                       │    CLAUDE.md  ROADMAP.md            │
                       │    SAFETY_CONTRACT.md               │
                       │                                     │
                       │  Communicational:                   │
                       │    planning/                        │
                       │      HANDOFF_TEMPLATE.md            │
                       │      backlog.md                     │
                       │      team-structure-log.md          │
                       │      <date>-<topic>-handoff.md      │
                       │                                     │
                       │  Operational:                       │
                       │    .claude/agents/  (cross-cutting) │
                       │    .claude/skills/                  │
                       │    planning/<domain-class>/         │
                       │      (binding contract +            │
                       │       session manifests)            │
                       └────┬────────────┬────────────┬──────┘
                            │            │            │
                  ┌─────────▼────┐ ┌─────▼────┐ ┌─────▼────┐
                  │  child-repo  │ │ child-repo │ │ child-repo│
                  │  (own git    │ │ (own git   │ │ (own git  │
                  │  remote, own │ │ remote,    │ │ remote,   │
                  │  CLAUDE.md,  │ │ own        │ │ own       │
                  │  own CI,     │ │ Guardian   │ │ Guardian  │
                  │  own         │ │ agent)     │ │ agent)    │
                  │  Guardian)   │ │            │ │           │
                  └──────────────┘ └────────────┘ └───────────┘
```

The parent layer is a coordination overlay. Each child is a fully independent repo with its own remote, CI, CLAUDE.md, and (optionally) a Guardian agent that gates risky in-repo work.

## 4. Core slots

### 4.1 `CLAUDE.md` — the coordination manifest

This is the most-read document in the repo. It tells every agent that opens this directory:

- **Where they are** (a coordination layer, not a monorepo)
- **What the siblings are** (table: subproject / purpose / remote)
- **What the rules are** (stay in your project; cross-project work lives here; handoff filename pattern; etc.)
- **What the cross-cutting platforms are** (so they don't re-propose Uptime Kuma when you're already running Grafana)
- **When and how to dispatch a team** (single agent vs. team-of-Leads)
- **What conventions are inherited** from another pattern source (if applicable)

Required sections (in this order, for skim-readability):

1. One-paragraph "this is not a project, it's a coordination layer"
2. Sibling repos table
3. (Optional) Hardware or services inventory — if your child repos manage physical or cloud infrastructure
4. **Why this layer exists** (the failure mode statement, in your words)
5. **Rules for agents** (numbered, 5–7 rules)
6. **Cross-cutting platforms** (table of single-stack homes for shared concerns)
7. **Team structure** (when to dispatch, composition, safety hand-off via Guardians)
8. (Optional) Conventions inherited

### 4.2 `ROADMAP.md` — the rolling cross-repo index

A curated list, not a state machine. Three sections:

- **In flight** — work with motion in the last ~30 days. One bullet per workstream. Carries: phase marker, authoritative doc location, next gate (the signal that unblocks it), and which child repos it fans out to.
- **On deck** — scoped but not started. Blocked, gated, or awaiting an external precondition. Links to the blocking item.
- **Recently completed** — last ~90 days. Kept for "where did that thing go" lookups. After 90 days, move to per-repo historical record or just `git log`.

The ROADMAP is *not* the source of truth for any workstream — it points to authoritative planning docs (in `planning/` here, or in child repos' own `planning/`). It's an index, kept short enough to skim in under a minute.

**Update rhythm:** at the end of every session, the agent moves bullets between sections and updates the next-gate marker. Drift is the failure mode; a `/wrap-up` skill (or equivalent ritual) is what keeps it honest.

### 4.3 `planning/` — async coordination substrate

| File / folder | Role |
|---|---|
| `README.md` | Explains what this folder is for |
| `HANDOFF_TEMPLATE.md` | Canonical template for cross-repo handoffs |
| `backlog.md` | One-line cross-repo items not yet worth a full handoff doc |
| `team-structure-log.md` | Append-only observations from team-dispatch sessions |
| `<YYYY-MM-DD>-<topic>-handoff.md` | Individual cross-repo handoffs |
| `<domain-class>/` | Subfolder per cross-cutting agent class (see §7) |

**Filename convention:** dated handoffs use `YYYY-MM-DD-<topic>-handoff.md`. The date is when the handoff was *written*, not when the work happens. This keeps the directory sortable and makes "what handoffs are still open?" a `ls -lt | head` rather than a query.

**Backlog vs. handoff:** if it fits in one line ("HA needs a DHCP reservation that OPNsense should set up"), it goes in `backlog.md`. If it has context, prerequisites, acceptance criteria, or an agent log, it gets its own handoff doc.

### 4.4 `.claude/agents/` — cross-cutting agents

**Important:** agents at the parent level are only for **cross-cutting concerns** — work that genuinely spans child repos and can't be owned by any one of them. Examples:

- A `file-archivist` that operates on shared storage (used by every child that produces or consumes that storage)
- An `infra-deployer` that orchestrates deploys across services owned by different child repos
- A `coordination-secretary` that maintains the ROADMAP, backlog, and team-structure-log

Agents that work inside a single child repo live in *that child's* `.claude/agents/`, not here. The rule of thumb: if removing a child repo from the workspace would obsolete the agent, the agent belongs in that child.

### 4.5 `.claude/skills/` — composable pipeline skills

Cross-cutting agents typically operate via **multi-stage pipelines**. The pattern that works well:

```
scan        →   classify     →   plan         →   execute
(read-only)     (read-only)      (read-only)      (gated mutation)
```

- **scan**: walk the input substrate, emit `candidates.jsonl` (one row per item)
- **classify**: enrich each candidate with domain metadata (type, target, dedup key, confidence) → `classified.jsonl`
- **plan**: cross-check against canonical state, build a `manifest.jsonl` of `{source, target, action}` rows for human review
- **execute**: hard-gated by `--approved <manifest>`; performs mutations one row at a time; appends to session log

This shape generalizes far beyond file ops. The same scan/classify/plan/execute pattern works for: bulk API operations (scan inventory → classify by action needed → plan calls → execute with rate limiting); infrastructure migrations (scan resources → classify target environments → plan steps → execute via Terraform); content moderation (scan submissions → classify by risk → plan actions → execute with override gate).

The skills compose: each one's output is the next one's input. Plan stages are idempotent and re-runnable; only execute mutates.

### 4.6 Workspace file (`example.code-workspace`)

A multi-root VS Code workspace that loads the parent + every child as a peer folder. Gives unified search, navigation, and tool access across all siblings. Replace `example.code-workspace` with `<your-org>.code-workspace` once you've added real children.

### 4.7 Optional adjuncts

- **`SAFETY_CONTRACT.md`** — shared safety rules + table of how each child mechanizes them differently + a fire-drill ritual. Keep this only if your child repos manage things where mistakes are expensive (production infra, customer data, irreversible storage).
- **`SHARED_TOOLING.md`** — inventory of agent/script/skill overlap across child repos. Descriptive, not prescriptive — labels consolidation candidates with effort/risk notes. Useful once you have 3+ children and notice the same patterns appearing.
- **`SNAPSHOT.md`** — live state inventory (current IPs, current versions, current hosts). Each child repo keeps its part fresh. Useful if you need a single document for an outside collaborator or a recovery scenario.

Don't ship these on day one. Add them when the absence is causing pain.

## 5. The agent hierarchy

```
Supervising agent (main session thread)
│
├── per-repo Lead (dispatched for child A)
│       │
│       └── invokes child A's Guardian on risky ops
│
├── per-repo Lead (dispatched for child B)
│       │
│       └── invokes child B's Guardian on risky ops
│
└── cross-cutting agent (e.g. file-archivist, at parent level)
        │
        └── operates across all children's domain
            via shared substrate (e.g. shared storage)
```

**Supervising agent** = the main session thread. It reads `CLAUDE.md`, picks the right collaborator pattern (solo vs. team), and integrates the results.

**Per-repo Lead** = a dispatched subagent scope-locked to one child repo. It works inside the child's directory, follows the child's CLAUDE.md, makes commits to the child's branch. Cross-repo coordination happens through handoff docs, not direct edits.

**Cross-cutting agents** = parent-level agents (archivists and similar). Distinct from Leads — they don't own a child repo; they operate across the domain that all children share.

**Per-repo Guardian** = a child-repo agent that gates high-risk operations in that child. The Lead invokes the Guardian before executing irreversible changes.

**Known gap:** the parent layer can't easily *discover* the Guardian agents that live in child repos. They're registered in each child's CLAUDE.md but invisible to the parent's `.claude/agents/` listing. In practice this means a parent-level agent that needs Guardian review for a child must either (a) be explicitly briefed on the Guardian's name, (b) defer to a child-repo session, or (c) escalate to the human operator. See §8.

## 6. Communication patterns

### Async handoff docs

The canonical artifact for cross-repo work. One file per handoff, named `planning/<YYYY-MM-DD>-<topic>-handoff.md`. Structure (see `planning/HANDOFF_TEMPLATE.md`):

- **Context** — one paragraph: why this exists
- **Originating project** — who's writing this and why
- **Receiving project** — who needs to act
- **What was done** — work already completed (often: the originating side)
- **What needs to happen** — checklist for the receiving side
- **Acceptance** — how do we know it's done
- **Open questions** — explicit unknowns
- **Agent log** — append-only timestamped entries

Handoffs are **async**: the receiver picks them up at their own pace. They're not Slack messages; they're more like email + tracking ticket combined.

### Session manifests

For risky, repeatable work, every session emits a manifest. Conventionally JSONL with:

- **Line 1** = a session-header record carrying attestations (snapshot coverage, validation method, approval state, rollback plan, etc.)
- **Lines 2…N** = per-item records (one per file moved, API call made, deploy step run, etc.)

Manifests live at `planning/<domain-class>/sessions/<session-id>.jsonl` and are append-only. They give you:

- **Census via jq** — answer "what work has been done across this domain?" with `jq` over all session manifests
- **Compliance audit** — answer "which sessions are missing a rollback plan?" by filtering session-header records
- **Provenance** — answer "which session moved this file / created this resource?" without git archaeology

### ROADMAP-as-state

Not a state machine, not a kanban board — just a curated index. The convention: every session that finishes a unit of work updates the relevant ROADMAP bullet (move between sections, update next-gate). If the ROADMAP drifts from reality, that's a signal the wrap-up ritual broke down.

### Team-structure-log

Append-only observation log. After each team-dispatch session, the supervisor appends 3–5 lines: who was on the team, what worked, what didn't, what to try next. After ~10 sessions, the log informs revisions to the team-structure section of `CLAUDE.md`. This is meta-pattern hygiene — the team pattern itself evolves, and the log is how you keep that evolution visible instead of forgetting why you did it the way you did.

## 7. The binding-contract pattern

This is the strongest, most generalizable piece of the pattern, and it deserves its own treatment.

**Problem:** some domains have multi-step risky operations (bulk file ops, paid API calls, infrastructure deploys, mass data migrations). You want every agent that touches that domain to:

- Follow the same safety discipline (validate, rollback plan, approval gate, audit trail)
- Refuse to execute non-compliant operations
- Produce machine-queryable evidence of compliance

You don't want each agent to re-derive that discipline. You want them to **inherit a contract**.

**Solution:** establish a *binding contract* for the domain class in `planning/<domain-class>/`. The contract is binding in two senses: agents are bound by it (refuse to proceed without compliance), and the contract binds together the substrate (session manifests, attestations, refusal patterns, design brief for spinning up new agents in the class).

### Contract structure

A binding contract folder typically contains:

- **`README.md`** — the contract itself: invariants, refusal patterns, attestation schema, what gets logged where
- **`DESIGN_BRIEF.md`** — a template for spinning up a new agent in the class, constraining creative work to the *domain-specific* axes (what to operate on, how to classify, dedup rules) and inheriting *invariant structure* (pipeline shape, session schema, approval gates)
- **`SESSION_LOG.md`** — append-only index of all sessions in the domain, with safety attestation codes per row
- **`sessions/`** — JSONL manifests, one per session, append-only

### What makes it binding (vs. just documentation)

1. **Enforcement is structural.** Agents read the contract and refuse to execute without compliance. Session headers missing attestation fields are detectably non-compliant — the execute step reads the header and errors if any required field is unset. Refusals come early, not late.
2. **The substrate is reusable.** The session-header schema (attestations + metadata) applies to any agent doing risky multi-step work, not just one domain. The JSONL structure is domain-agnostic.
3. **Compliance is auditable.** Append-only manifests form a queryable audit trail. A jq query lists all non-compliant sessions. No silent partial-compliance.
4. **Deviation is visible.** The design brief constrains newcomers to specific creative axes and inherits everything else. Deviations are flagged as operator decisions, not silent redesigns.

### When to add a binding contract

You don't need one on day one. Add it when:

- You notice you're writing the same safety checklist into multiple agents
- A risky domain operation went wrong and you wished you had an audit trail
- You're about to add a second agent in a domain that already has one

The template ships with `planning/domain-class-example/` to show the shape. Rename it to your domain (e.g. `planning/file-ops/`, `planning/deploys/`, `planning/billing-api/`) when you're ready, or delete it if you don't need one.

## 8. Known gaps & honest limitations

**Child-repo Guardians aren't discoverable from the parent.** Guardian agents live in child-repo `.claude/agents/` directories, so the parent's agent registry doesn't see them. When a parent-level agent needs Guardian review, it has to either be briefed explicitly, defer to a child-repo session, or escalate to the operator. Not solved; documented honestly.

**No CI on the parent layer.** The conventions in `CLAUDE.md` are enforced socially, not mechanically. If an agent ignores the "stay in your project" rule, nothing catches it except the next reader. This is a feature for low-stakes coordination layers but a bug if you grow past ~10 children with multiple human contributors.

**Manual ROADMAP maintenance.** The ROADMAP drifts if no one updates it. The `/wrap-up` ritual (or equivalent — a slash command, a Friday habit, a Friday-afternoon-reminder bot) is how you keep it honest. Without that, the ROADMAP becomes outdated in 2–3 weeks and the pattern breaks down.

**Scaling past ~15 child repos.** The supervising-agent-with-dispatched-Leads pattern starts to fragment past ~15 children. The supervisor's context can't hold the full sibling-repo table cleanly. A future evolution (out of scope here) would cluster children by domain and introduce per-domain supervisors. If you're approaching this size, plan that evolution before you hit it.

**Cross-cutting agents can rot.** A `file-archivist` that nobody invokes for 6 months is a stale agent — its mental model of the substrate may have drifted from reality. The `SESSION_LOG.md` is a leading indicator: if no rows have been added in months, audit the agent before invoking it next.

## 9. Appendix: worked example (homelab)

The pattern crystallized from `mriechers/homelab`, a coordination layer over five sibling repos:

| Slot in template | Filled-in homelab equivalent |
|---|---|
| `CLAUDE.md` | Five named sections: sibling repos table (5 repos), hardware inventory (3 sites with full device specs), why-this-layer-exists, 6 numbered rules for agents, cross-cutting platforms table (observability, TLS, DNS, NAS file-ops), team structure with Guardian hand-off |
| `ROADMAP.md` | Three sections (in flight / on deck / recently completed) with phase markers and next-gate signals; ~6 active workstreams as of 2026-06-03 |
| `SAFETY_CONTRACT.md` | 8 shared safety rules, mechanization table per child, quarterly fire-drill ritual |
| `SHARED_TOOLING.md` | Inventory of guardians, mad scientists, backup-log patterns, doc-crawl tools, knowledge-sync directions — labeled with consolidation effort/risk |
| `planning/HANDOFF_TEMPLATE.md` | Canonical handoff template; 30+ dated handoffs from 2026-05-26 onward live alongside it |
| `planning/<domain-class>/` | `planning/nas-file-operations/` — binding contract for any agent doing bulk file ops on the NAS; reference impl is `media-archivist`, with `rom-archivist` and `dev-archivist` following the same shape |
| `.claude/agents/` | Three cross-cutting archivists: `media-archivist`, `rom-archivist`, `dev-archivist` |
| `.claude/skills/` | Twelve skills — three parallel four-stage pipelines (scan / classify / plan / execute) for media, ROM, and dev domains |
| Workspace file | `homelab.code-workspace` loading all five sibling repos as peer folders |

The homelab repo is the lived test of every claim in this design doc. Where this doc says "known gap," it's because the homelab hit that gap. Where it says "the binding contract pattern works for any risky domain," it's because the same shape works for media, ROMs, and dev-folder cleanup with only the domain-specific axes changing.

If you want to see what a populated, in-flight version of this pattern looks like, that's where to look.

---

*End of design doc. See [RECIPE.md](RECIPE.md) for step-by-step instantiation.*
