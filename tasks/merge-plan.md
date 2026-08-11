# Implementation Plan: MiniGit Merge (BLRID-2)

## Overview

Deliver merge of a source branch into HEAD: FF or two-parent merge commit; missing ref errors; conflict abort+report; CLI + Flask; tests. Spec: [docs/specs/merge-feature.md](../docs/specs/merge-feature.md). Ticket: BLRID-2.

## Architecture Decisions

- Additive **second parent** on commits (keep single-parent reads working)
- Merge logic in `Operations`; thin CLI/Flask
- Conflict = path changed on both sides vs ancestor → abort, no textual merge
- TDD per acceptance path

## Dependency graph

```
two-parent storage (schema + Commit)
    → Operations.merge (+ conflict/FF/ancestor)
        → tests/test_merge.py
        → CLI merge
        → Flask merge UI
```

## Tasks

### Phase A — Foundation
1. Two-parent commit storage + tests for old/new commits
2. Ancestor / FF / diverge helpers + missing ref errors

### Phase B — Merge core
3. `Operations.merge` clean diverged + FF + already-up-to-date + conflict abort (TDD)
4. CLI `minigit merge`
5. Flask merge UI

### Checkpoint
- `make check` green; AC paths covered

### Phase C — Close
6. Checker review (separate pass) + VERDICT

## Risks

| Risk | Mitigation |
|------|------------|
| Schema breaks old commits | Additive field; migration/read null as single parent |
| Naive conflict detection | Spec path-level rules; tests |

## Open Questions

None if spec assumptions accepted.
