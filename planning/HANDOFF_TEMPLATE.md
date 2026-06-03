# Handoff: <topic>

> **How to use this template:** copy to `planning/YYYY-MM-DD-<topic>-handoff.md` (use today's date), fill the sections below, commit. Delete this instructional block from your copy.

## Context

<!-- One paragraph: why this handoff exists. What problem or need prompted it. The intended outcome. A reader who's never seen this workspace should understand what's at stake from this paragraph alone. -->

## Originating project

**Repo:** `<my-service-a>`
**Author:** <agent or human name>
**Date:** YYYY-MM-DD

<!-- What this project did or needs to do that requires the receiving project's cooperation. -->

## Receiving project

**Repo:** `<my-service-b>`
**Owner:** <who picks this up>

<!-- Who needs to act on this handoff. If unknown, write "TBD — needs assignment." -->

## What was done

<!-- Work already completed (typically on the originating side). Be specific: file paths, commit refs, test results.

Example:
- Added new schema field `foo` in `my-service-a/db/schema.sql` (commit abc123)
- Deployed to staging, validated against existing flows
- Wrote integration test in `my-service-a/tests/test_foo.py`
-->

## What needs to happen

<!-- Checklist for the receiving side. Each item should be concrete and verifiable. -->

- [ ] <specific action>
- [ ] <specific action>
- [ ] <specific action>

## Acceptance criteria

<!-- How will we know this handoff is done? Be testable.

Example:
- `my-service-b` can read records with the new `foo` field
- Integration test `tests/test_foo_cross_service.py` passes
- No regressions in existing flows (verify by running `make test` in both repos)
-->

## Open questions

<!-- Things we don't know yet. Don't paper over uncertainty.

Example:
- Should `foo` be nullable or required? (defaults to nullable; receiving project may want to override)
- Is there a migration path for existing records?
-->

## Agent log

<!-- Append-only timestamped entries as the work progresses. Latest at the bottom.

Example:
- 2026-MM-DD — handoff written; originating work complete (Mark + Claude session)
- 2026-MM-DD — receiving project began work; clarified question 1 inline (defaulting to nullable)
- 2026-MM-DD — Phase 1 of receiving work landed; remaining items: 2, 3
- 2026-MM-DD — closed; acceptance criteria met
-->
