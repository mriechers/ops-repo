# Recipe: instantiate this template for your workspace

A practical, step-by-step guide to going from "I clicked Use this template" to "my agents are using the coordination layer for real work." Plan for a 60–90 minute first pass; you can defer optional slots until you actually need them.

## Before you start

Make sure you have:

- A directory where your child repos already live (or will) as peer folders — e.g. `~/Code/myorg/`
- At least two child repos identified that share a concern
- A clear sense of *the failure mode* you're trying to prevent. ("Agents have been dropping work into the wrong repo." "I keep forgetting which workstreams are in flight." "I want a shared safety contract for X.") If you can't articulate this in one sentence, you may not need this pattern yet.

## Step 1 — Clone the template

On GitHub: click **"Use this template"** → create a new repo (private or public, your call).

Locally, clone it into your workspace as a peer to your child repos:

```bash
cd ~/Code/myorg/
git clone git@github.com:yourname/your-metarepo-name.git
cd your-metarepo-name
```

## Step 2 — Fill the `CLAUDE.md` slots

Open `CLAUDE.md` and fill in:

1. **Project name** at the top
2. **Sibling repos table** — one row per child repo with subproject name, purpose, remote, path to that child's `CLAUDE.md`
3. **(Optional) Hardware or services inventory** — if your child repos manage physical devices, cloud resources, or production services. Delete this section if not applicable.
4. **Why this layer exists** — the one-paragraph failure-mode statement (the sentence you wrote in "Before you start" goes here, expanded)
5. **Rules for agents** — five inherited rules ship verbatim; tweak rule 5 (public vs. private parent) based on whether this metarepo is public
6. **Cross-cutting platforms** — fill in if you have shared infrastructure (observability, auth, deploy pipeline, etc.); delete if you don't yet
7. **Team structure** — inherited verbatim; modify when your team-structure-log gives you data to revise it

A reasonable first version of CLAUDE.md fits in a single screen. Resist the urge to over-document.

## Step 3 — Seed the `ROADMAP.md`

Open `ROADMAP.md`. The three sections (In flight / On deck / Recently completed) ship with one example bullet each. Replace the examples with:

- **In flight:** what are you actively working on across these repos? One bullet per workstream. Add a phase marker if useful.
- **On deck:** what's scoped but not started? Note the blocker.
- **Recently completed:** what shipped in the last 90 days? (If you're brand new, leave this empty and let it fill organically.)

If you can't fill any section, delete the example bullet — an empty section is better than a misleading placeholder.

## Step 3.5 — Enable AGENTS.md and the commit-msg hook

Two small chores that the template ships placeholders for:

**AGENTS.md** — the template ships `AGENTS.md` as a symlink to `CLAUDE.md` so non-Claude harnesses (Codex, Gemini CLI) discover the same coordination doc. Verify it points at the right place:

```bash
ls -la AGENTS.md   # should show: AGENTS.md -> CLAUDE.md
```

If it didn't survive the template-clone (some GitHub flows lose symlinks), recreate it:

```bash
ln -s CLAUDE.md AGENTS.md
```

**`.githooks/commit-msg`** — the template ships a placeholder. If your workspace has a shared commit-msg hook (e.g. a `the-lodge/conventions/git-hooks/commit-msg` that enforces a workspace commit convention), edit `.githooks/commit-msg` per the instructions in that file to delegate to it. Then:

```bash
chmod +x .githooks/commit-msg
git config core.hooksPath .githooks
```

If you don't have a shared hook, you can leave the placeholder or delete the `.githooks/` directory entirely.

## Step 4 — Add your first child repo to the workspace

Open `example.code-workspace`, rename it (`<your-org>.code-workspace`), and add folder entries:

```json
{
  "folders": [
    { "path": "." },
    { "path": "../my-service-a" },
    { "path": "../my-service-b" }
  ]
}
```

Add the child repos to `.gitignore` so they don't show up as untracked under the parent:

```gitignore
# Child repo working copies
my-service-a/
my-service-b/
```

## Step 5 — Decide on cross-cutting agents

Look at `.claude/agents/example-domain-agent.md` and `.claude/skills/example-scan/`. These ship as **worked examples** of the cross-cutting agent pattern — *not* as agents you keep.

Two paths:

**Path A — you have a cross-cutting domain.** (Bulk file ops, deploy orchestration, shared API operations.) Use the example as a starting point:
1. Copy `.claude/agents/example-domain-agent.md` → `.claude/agents/<your-agent-name>.md`
2. Copy `.claude/skills/example-scan/` → `.claude/skills/<your-domain>-scan/` (and similarly for classify/plan/execute as needed)
3. Adapt the domain-specific logic
4. Copy `planning/domain-class-example/` → `planning/<your-domain>/` (see Step 6)
5. Delete the example files (`example-domain-agent.md`, `example-scan/`, `domain-class-example/`)

**Path B — you don't yet.** Delete `.claude/agents/example-domain-agent.md`, `.claude/skills/example-scan/`, and `planning/domain-class-example/`. You can re-add the pattern later when you have a concrete need.

## Step 6 — (If applicable) Set up the binding-contract folder

If your cross-cutting agent operates in a risky domain that needs an audit trail (file ops, paid APIs, deploys), keep the `planning/<your-domain>/` folder you copied in Step 5:

1. Rename `planning/<your-domain>/README.md`'s headings to match your domain (replace "domain class" with the actual class name)
2. Define your **attestation fields** in the session-header schema — the safety invariants every session must declare (e.g. for file ops: `snapshot_coverage`, `validated_via`, `rollback_plan`, `approval_state`)
3. Define your **refusal patterns** — the conditions under which agents must refuse to execute (e.g. missing attestation fields, paths outside allowed prefixes, unresolved conflict-review rows)
4. Adapt `DESIGN_BRIEF.md` for spinning up future agents in the same class
5. Leave `SESSION_LOG.md` and `sessions/` empty — they fill as you run real sessions

## Step 7 — Write your first handoff

When your first cross-repo task lands, this is where the pattern earns its keep.

1. Copy `planning/HANDOFF_TEMPLATE.md` → `planning/<YYYY-MM-DD>-<topic>-handoff.md` (use today's date)
2. Fill the Context, Originating project, Receiving project sections
3. Write the "What needs to happen" checklist for the receiving side
4. Commit it to the parent metarepo
5. When the receiving project's session picks it up, they update the agent log and check off items

If your first handoff doesn't feel like it earned its weight (the work was trivial enough to do inline), that's fine — but if you keep skipping it, the coordination layer isn't load-bearing for you yet. Either find a real cross-repo workstream or shelf the pattern.

## Step 8 — Set up the team-structure-log

`planning/team-structure-log.md` ships empty with one example entry. The first time you dispatch a team (subagents per repo), append a 3–5 line observation: who was on the team, what worked, what didn't.

After ~10 team-dispatch sessions, re-read the log and ask: does the team structure in CLAUDE.md still match how teams actually work? Revise CLAUDE.md if not.

## Step 9 — (Optional) Add safety contract and shared tooling inventory

Skip these on day one. Add them when:

- **`SAFETY_CONTRACT.md`:** you've had a near-miss (or worse), or you're about to do risky work where you'd want a shared discipline written down. The template ships with table headers and 3–5 generic rules; expand to match your domain.
- **`SHARED_TOOLING.md`:** you notice the same agent / skill / script pattern appearing in 2+ child repos and you're tempted to consolidate. The doc is descriptive (label the overlap; note effort/risk) before it becomes prescriptive (actually consolidate).

## When to delete the example files

The template ships with several `example-*` files as **teaching artifacts**. Delete them once you have your own equivalents — they should never live alongside real ones:

| Example file | Delete after | Replacement |
|---|---|---|
| `.claude/agents/example-domain-agent.md` | Step 5 | Your own cross-cutting agent (or no replacement if you took Path B) |
| `.claude/skills/example-scan/` | Step 5 | Your own pipeline skills |
| `planning/domain-class-example/` | Step 6 | Your renamed binding-contract folder |
| `example.code-workspace` | Step 4 | `<your-org>.code-workspace` |

The example bullets in `CLAUDE.md`, `ROADMAP.md`, `SAFETY_CONTRACT.md` are inline placeholders — overwrite them rather than deleting separate files.

## Verification: did the pattern actually take?

After ~3 weeks of use, ask yourself:

- **Is the ROADMAP current?** If it's drifted by more than 2 weeks, your wrap-up ritual broke down. Fix the ritual, not the doc.
- **Are agents writing handoffs?** Or are they still scattering work into the wrong repos? If still scattering, the rules in CLAUDE.md may need to be stricter or the agents may need re-briefing.
- **Are session manifests being written?** If you set up a binding contract but no sessions appear under `planning/<domain>/sessions/`, the contract isn't actually load-bearing — either the agents aren't using it, or the work isn't happening, or the discipline is being skipped.
- **Did the team-structure-log get appended?** If you've dispatched teams but the log is empty, the observation discipline didn't take. The log is the only thing that lets the team pattern evolve.

If any of these are no, the answer is usually: tighten the convention or accept that you don't need the pattern yet. Don't add more docs to fix it.

---

*End of recipe. See [DESIGN.md](DESIGN.md) for the why and rationale.*
