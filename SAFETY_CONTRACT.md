# Safety Contract

Shared safety rules that every child repo in this workspace inherits. Each child repo may **mechanize** these rules differently (one might enforce a rule via a pre-commit hook, another via a Guardian agent), but the rules themselves are universal.

> **When to keep this doc.** Only if your child repos manage things where mistakes are expensive: production infrastructure, customer data, irreversible storage operations, paid third-party services. If you're coordinating low-stakes work (docs, small services with cheap rollback), delete this file — it's overhead.

## Rules

<!-- TODO: tighten or expand to match your domain. The five rules below are a starting point that applies to most coordination layers. -->

1. **Backup before destructive operations.** Any operation that mutates state in a way that can't be undone via `git revert` must have a backup or snapshot taken first. If the operation is large, the backup must be verified (size + checksum) before the operation runs.
2. **Validate state changes.** After a state change, verify the new state matches the expected state before considering the operation complete. Don't trust that "the command returned 0" means "it worked" — observe the resulting state.
3. **Plan rollback.** Before executing a state change, the agent must have a documented rollback procedure. Either inline in the session log, or in the handoff doc that scheduled the change.
4. **Stage destructive work.** For multi-step destructive operations, stage the work so that each step is reviewable independently. Don't bundle "delete N files" into one approval — emit a manifest, get approval, then execute row-by-row.
5. **No secrets in handoff docs or session manifests.** Agents must redact secrets (API tokens, passwords, private keys) from any artifact that lands in this metarepo or its child repos. If a value must be referenced, use a placeholder + a pointer to its canonical storage (1Password, vault, secrets manager).

## Mechanization

Each child repo mechanizes the rules above using whatever tooling fits its domain. The table below tracks which child uses which mechanism.

<!-- TODO: fill this in once you have ≥2 child repos with concrete mechanizations. The columns are illustrative; adapt to your domain. -->

| Rule | `my-service-a` | `my-service-b` | Notes |
|---|---|---|---|
| Backup before destructive | <!-- e.g. pre-commit hook --> | <!-- e.g. Guardian agent gate --> | <!-- where they legitimately diverge --> |
| Validate state changes | | | |
| Plan rollback | | | |
| Stage destructive work | | | |
| No secrets | | | |

## Fire-drill ritual

<!-- TODO: optional but recommended once the contract is in active use. -->

Quarterly (or whenever the contract changes), run a fire-drill: pick one rule and one child repo, deliberately try to violate the rule, observe whether the mechanization catches it. Log the result. Adjust the mechanization if it didn't catch the violation. The goal is to test the contract, not hypothesize it.

| Date | Rule tested | Repo | Caught? | Notes |
|---|---|---|---|---|
| <!-- YYYY-MM-DD --> | <!-- which rule --> | <!-- which repo --> | <!-- yes/no --> | <!-- what we learned --> |

## When to add a new rule

A new rule earns its place when:

1. A near-miss or incident exposed a gap the existing rules didn't cover
2. You catch yourself writing the same ad-hoc safety check into multiple sessions
3. A new child repo joins the workspace with a class of risk the existing rules don't address

Don't add rules speculatively. Rule rot (rules nobody enforces because they don't match real work) is worse than missing rules.
