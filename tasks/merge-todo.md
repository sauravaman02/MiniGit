# Tasks: MiniGit Merge (BLRID-2)

Spec: [docs/specs/merge-feature.md](../docs/specs/merge-feature.md)  
Plan: [tasks/merge-plan.md](merge-plan.md)

- [ ] Task 1: Two-parent commit storage
  - Acceptance: DB + Commit support second parent; existing single-parent commits still read/hash correctly
  - Verify: focused tests + `make check`
  - Files: `src/backend/sqlite_client.py`, `src/components/commit.py`, `tests/test_commit.py` (and/or merge tests)
  - Scope: M

- [ ] Task 2: History relationship helpers
  - Acceptance: resolve source ref (error if missing); detect already-up-to-date, FF, diverged, no common ancestor
  - Verify: unit tests with `tmp_path` repos
  - Files: `src/frontend/operations.py`, tests
  - Scope: M
  - Depends: Task 1 (for later merge commit write; helpers may land with Task 3)

- [ ] Task 3: `Operations.merge` + conflict abort (TDD)
  - Acceptance: FF; two-parent clean merge; conflict abort with path report and tip unchanged; already-up-to-date message
  - Verify: `tests/test_merge.py`; `make check`
  - Files: `src/frontend/operations.py`, `tests/test_merge.py`
  - Scope: M
  - Depends: Task 1

- [ ] Task 4: CLI `minigit merge <source>`
  - Acceptance: success prints tip; failures non-zero + clear errors
  - Verify: CLI tests or manual + unit coverage of cmd path
  - Files: `src/cli.py`, tests
  - Scope: S
  - Depends: Task 3

- [ ] Task 5: Flask merge UI
  - Acceptance: choose source branch, merge into current, show success/error
  - Verify: Flask test client and/or manual; `make check`
  - Files: `src/app.py`, `src/templates/*`
  - Scope: M
  - Depends: Task 3

## Checkpoint after Tasks 1–5

- [ ] All AC from BLRID-2 covered by tests
- [ ] `make check` green
- [ ] Ready for checker pass + outer-loop VERDICT
