# Design Brief — new agent in this domain class

Use this template when spinning up a new agent in this binding contract's class. The template **constrains creative work to the domain-specific axes** below. Everything else — pipeline shape, session schema, attestation fields, approval gates, refusal patterns — is inherited from [`README.md`](README.md) without modification.

> **The rule.** If your design needs to deviate from any inherited invariant, that's a flag-to-operator item, not a silent redesign. Document the deviation and the reason here; don't bury it in the agent file.

## 1. Agent identity

- **Name:** <agent-name> (snake-case-with-dashes, e.g. `file-archivist`, `deploy-orchestrator`)
- **Class:** <inherited from this folder's name>
- **One-line role:** <what does it do for the workspace>
- **Scope:** <which substrate it operates on; which it explicitly refuses>

## 2. Domain inputs (the creative axis)

These are the four things you must define to instantiate a new agent in this class. Everything else inherits.

### 2.1 Content domain

What single class of items does this agent handle?

<!-- Examples:
- file-archivist for movies/TV: handles video files only; music deferred to a separate agent
- deploy-orchestrator for backend services: handles HTTP services only; batch jobs out of scope
- billing-api-caller for refund operations: handles refund calls only; not subscription changes
-->

**Narrow domains classify better.** Resist the temptation to handle "everything that's similar." Two narrow agents beat one wide one.

### 2.2 Target / sink

Where does the agent's mutations land? What's the target substrate and its layout?

<!-- Examples:
- file-archivist: target is /shared-storage/library/, layout is Type/Subtype/Title (Year).ext
- deploy-orchestrator: target is the Kubernetes cluster, namespace per service
- billing-api-caller: target is Stripe API, endpoint /v1/refunds
-->

**Include the layout doc location.** Agents need to be able to reference the canonical layout in their reasoning.

### 2.3 Classification taxonomy

How does the agent categorize the items it scans? What categories exist, what parsing strategy does each need, what quality tiers (if any) apply?

<!-- Examples:
- file-archivist: { movie, tv-episode, music-track, unknown } × { quality: lossless | high | medium | low }
- deploy-orchestrator: { greenfield-service, rolling-update, hotfix, rollback }
- billing-api-caller: { full-refund, partial-refund, dispute, void }
-->

**Parsing notes per category.** What signals are used to assign each category (filename patterns? metadata fields? content inspection)? What's the fallback when parsing fails?

### 2.4 Dedup / deconfliction rules

How does the agent decide whether two items are the "same" item, and which one wins when there's a conflict?

<!-- Examples:
- file-archivist: dedup by SHA-256; lossless > lossy; higher resolution > lower
- deploy-orchestrator: dedup by service+version+environment; the most recent successful deploy wins
- billing-api-caller: dedup by (charge_id, refund_amount); idempotent via idempotency-key
-->

**Tie-break rules matter.** When two items tie, what happens? Conflict-review row in the manifest? Pick deterministically? Refuse?

## 3. Inherited (do not redefine)

These come from the binding contract in [`README.md`](README.md):

- Four-mode pipeline (scan → classify → plan → execute)
- Session-header attestation schema (all required fields)
- Per-record manifest schema
- Append-only invariant
- Source/target allowlist enforcement
- Approval-gated execute
- Per-row verification
- Refusal patterns

If you find yourself wanting to change any of these, **stop and flag it** — either the contract needs amendment (a separate decision), or your agent might not fit this class.

## 4. Skills

The pipeline maps to four skills (or whatever number your class uses). Per-skill responsibilities:

<!-- Adapt to your class's pipeline shape. The four-stage shape below is the default. -->

| Skill | Mutates? | Output |
|---|---|---|
| `<agent-name>-scan` | no | `candidates.jsonl` — one row per item discovered |
| `<agent-name>-classify` | no | `classified.jsonl` — enriched with taxonomy assignment, confidence, dedup key |
| `<agent-name>-plan` | no | `manifest.jsonl` — `{source, target, action}` rows for review |
| `<agent-name>-execute` | yes (gated) | session manifest in `sessions/<session-id>.jsonl` |

Skills should be **idempotent for read-only stages** and **strictly approval-gated for the execute stage**.

## 5. Open questions

<!-- Append items the designer can't resolve unilaterally. -->

- <question for the operator>
- <question for the operator>

## 6. Acceptance criteria for this design

Before this brief is "done" and skills implementation begins:

- [ ] Operator has reviewed the four domain inputs (§2)
- [ ] Operator has approved any deviations from inherited invariants (§3)
- [ ] Open questions (§5) are resolved or explicitly punted
- [ ] Skills outline (§4) names every skill and its mutation status
