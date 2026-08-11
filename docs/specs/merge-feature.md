# Spec: MiniGit Merge (BLRID-2)

## Objective

Implement branch merge in MiniGit so a user can merge a **source branch** into the **current branch** (HEAD): fast-forward when possible; otherwise a **two-parent merge commit**. Missing refs/commits and content conflicts fail clearly. Surfaces: **CLI + Flask**.

**Ticket:** [BLRID-2](https://redhat.atlassian.net/browse/BLRID-2)  
**Users:** MiniGit CLI and web UI users learning Git merge semantics.

### Acceptance criteria (from ticket)

- [ ] `minigit merge <branch>` merges source into current (FF or two-parent commit)
- [ ] Missing/unknown source → clear error; no partial ref update
- [ ] Path conflicts → abort and report; tip unchanged
- [ ] Clean diverged merge → two-parent commit; tip updated
- [ ] FF works without unnecessary merge commit
- [ ] Already up to date detected and reported
- [ ] Flask UI triggers same behavior
- [ ] Tests: FF, two-parent, missing ref, conflict abort, already-up-to-date (`tmp_path`)
- [ ] `make check` passes
- [ ] Follow `AGENTS.md`

## ASSUMPTIONS

1. Target is always current HEAD branch (not arbitrary target branch flag in v1).
2. Conflict = same path changed vs ancestor on both sides differently; abort, no textual merge.
3. No common ancestor → error (unrelated histories).
4. Two-parent storage is additive; single-parent commits remain valid.
5. Default merge message: `Merge branch '<source>'`.

## Tech Stack

Existing MiniGit: Python, SQLite, Flask, pytest. No new dependencies.

## Commands

```bash
make setup | make lint | make typecheck | make test | make check | make fmt
# CLI (after implement)
python src/cli.py merge <source-branch>
```

## Project Structure

```
src/components/commit.py      # may gain second parent in hash/model
src/backend/sqlite_client.py  # store/read second parent
src/frontend/operations.py    # merge()
src/cli.py                    # merge subcommand
src/app.py + templates/       # merge UI
tests/test_merge.py           # new
```

## Code Style

Follow `AGENTS.md`: type hints, parameterized SQL, hash `^[0-9a-f]{64}$`, refs `^[A-Za-z0-9_.\-/]+$`, layer import rules, functions &lt; 50 lines.

Example:

```python
def merge(self, source_ref: str, author: str | None = None, message: str | None = None) -> str:
    """Merge source_ref into HEAD. Return new tip hash (FF or merge commit)."""
    ...
```

## Testing Strategy

| Case | Level |
|------|-------|
| Missing ref | unit/integration `tmp_path` |
| Already up to date | integration |
| Fast-forward | integration |
| Two-parent clean merge | integration |
| Conflict abort | integration |
| CLI / Flask | CLI invoke / Flask test client as practical |

No network in tests.

## Boundaries

**Always:** validate refs/hashes; abort conflicts without tip move; `make check` before done; TDD for AC paths.

**Ask first:** new dependencies; changing conflict policy to auto-merge text; unrelated-histories allow-with-flag.

**Never:** commit secrets; execute user input as code; f-string SQL.

## Success Criteria

All ticket AC checked; CLI and UI demoable; evidence posted on BLRID-2 via SDLC labels.

## Open Questions

None blocking MVP if assumptions above hold.
