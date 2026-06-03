---
name: example-scan
description: Worked example of the scan stage of a domain agent's four-mode pipeline. Walks a source substrate within allowlisted prefixes and emits a candidates.jsonl plus discovery.md. Read-only — never mutates. Triggers on "scan <source> for <domain> candidates", "what's accumulated in <source>", or as step 1 of the example-domain-agent pipeline. Delete this skill once you've adapted it for your real domain.
---

# example-scan

A worked example of the **scan** stage. Read-only walk of source substrate within allowlisted prefixes, producing the input file that the classify stage consumes.

> **Where this fits.** Stage 1 of the four-stage pipeline: `scan → classify → plan → execute`. Stages 2–4 are separate skills (not shipped in this template — you write them per your domain). See `planning/domain-class-example/README.md` for the full contract this skill inherits.

## When to use

- Operator says "scan &lt;source-prefix&gt; for &lt;domain&gt; candidates"
- Operator says "what's accumulated in &lt;source-prefix&gt;"
- As step 1 of the example-domain-agent's plan mode

## Inputs

- **Source prefixes** — one or more paths within the allowlist defined in `planning/<domain-class>/README.md`. Refuse any prefix outside the allowlist.
- **Domain-specific filters** (optional) — e.g. extension list, manifest markers, name patterns

## Outputs

Two files per scan, named with a shared session-id prefix:

1. **`planning/<domain-class>/pending/<session-id>-candidates.jsonl`** — one JSON record per candidate item:

   ```json
   {
     "record_type": "candidate",
     "source_path": "/source/prefix/path/to/item",
     "size_bytes": 1234567,
     "mtime": "2026-MM-DDTHH:MM:SSZ",
     "scan_signals": {
       "extension": ".ext",
       "matched_pattern": "name-pattern",
       "...": "..."
     }
   }
   ```

   Hash computation is **deferred to the classify stage**, because hashing every file at scan time can be expensive and many candidates will be filtered out by classification.

2. **`planning/<domain-class>/pending/<session-id>-discovery.md`** — human-readable summary: file counts by extension, per-prefix distribution, suspected anomalies, anything operator-visible that helps decide whether to proceed to classify.

## Refusal patterns

This skill refuses to proceed when:

- A configured source prefix is outside the allowlist in `planning/<domain-class>/README.md`
- The source substrate is unreachable (network mount down, API endpoint 5xx) — refuses rather than emitting a partial scan
- The output directory `planning/<domain-class>/pending/` is read-only or missing

Refusals exit non-zero with a clear error message. Silent partial scans are the failure mode this skill prevents.

## Idempotency

This skill is **fully idempotent** — running it twice produces equivalent outputs (modulo the session-id and a re-scan of mtime-changed items). No state is mutated, no cache is poisoned, no manifests are touched. Re-run freely.

## What this skill does NOT do

- Classify items (that's the classify stage's job; this skill only emits raw candidates)
- Hash file contents (deferred to classify; hashing every file at scan time is wasteful when classification will filter most)
- Compare against canonical inventory (that's the plan stage's job)
- Mutate anything (this skill is read-only by construction)

## Adapting for your domain

When copying this skill to your domain:

1. Rename `example-scan` → `<your-domain>-scan` (folder name + frontmatter `name`)
2. Update the trigger phrases in the frontmatter `description`
3. Define your source prefixes and scan signals
4. Decide what goes in `scan_signals` vs. what's deferred to classify
5. Write the discovery.md template for your domain (what's interesting to see at scan time)
6. Delete the "example" framing from the body

The structural pattern (read-only, refuses on allowlist violation, emits two files, idempotent) is inherited unchanged.
