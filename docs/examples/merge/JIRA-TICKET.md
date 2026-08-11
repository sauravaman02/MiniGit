# Jira ticket body (paste into BLRID description)

Use the block below as the issue **Description** (and mirror Acceptance Criteria in the AC field if your project has one).

---

## Summary (suggested title)

`feat: MiniGit merge — merge one branch into another`

## Description

### Background

MiniGit today supports init, branch, checkout, commit history, and diffs, but **not** merging one branch into another. We need a **Merge** feature so a user can integrate commits from a source branch into a target branch (typically the current HEAD / checked-out branch).

### Goal

Implement `merge` so that:

1. The user selects a **source branch** (or ref) to merge **into** the current branch (HEAD).
2. MiniGit computes how the histories relate (common ancestor / divergence).
3. If the merge is possible, MiniGit records the result as history on the current branch:
   - **Fast-forward** when the current branch has not diverged (HEAD is an ancestor of the source): move the current branch tip to the source tip (no new merge commit required).
   - **True merge** when histories have diverged: create a **new commit** on the current branch that combines both sides and records **two parents** (current HEAD + source tip), representing “merge source into current.”
4. If a required commit/ref **does not exist**, or merge cannot proceed safely, **fail with a clear error** (do not silently succeed).

### Non-goals (this ticket)

- No interactive conflict resolution / textual 3-way file merge editor.
- On overlapping content conflicts: **abort and report** which paths conflict; leave repository state unchanged (or clearly documented safe abort).
- Not a full Git-compatible merge of every edge case; educational clarity over exotic Git options (`octopus`, `ours` strategy, etc.).

### User-facing behavior

**CLI**

```text
minigit merge <source-branch>
```

- Merges `<source-branch>` into the **current** branch (HEAD).
- Success: print new commit hash (or FF result) and updated branch tip.
- Failure: non-zero exit + clear message (unknown ref, missing commit, conflict abort, etc.).

**Web UI (Flask)**

- From a repo view, allow choosing a source branch and triggering merge into the current branch.
- Show success (new tip / merge commit) or error message (same cases as CLI).

### Core concepts (for the agent)

| Term | Meaning in this ticket |
|------|-------------------------|
| Current branch | The branch HEAD points at (merge **target**) |
| Source branch | The branch being merged **in** |
| Divergence | Histories share an ancestor but each has commits the other lacks |
| Fast-forward | Target has no unique commits; tip can move to source tip |
| Merge commit | New commit with **two parents**: `(target_tip, source_tip)` |
| Abort | Do not update refs; report why |

### Functional requirements

1. **Resolve refs**
   - Accept a branch name (and, if already supported elsewhere, a commit hash).
   - If the source ref or any commit needed for the merge **does not exist** → **error** (e.g. `ValueError` / CLI message: ref or commit not found).

2. **Find relationship**
   - Determine common ancestor (or equivalent) between current HEAD commit and source tip.
   - Detect: already up to date | fast-forward | diverged | unrelated / no ancestor (define and error clearly).

3. **Fast-forward path**
   - When allowed, update current branch ref to source tip.
   - No merge commit.

4. **Diverged path**
   - Build the merged tree from both sides relative to the ancestor.
   - If the same path changed differently on both sides → **abort and report** conflicting paths.
   - If clean: create merge commit with message (default ok, e.g. `Merge branch '<source>'`), author from env/user, **two parents**, store it, point current branch at it.

5. **Idempotent / safe messaging**
   - If source is already an ancestor of HEAD (“already up to date”), report that; do not create a useless commit.

6. **Persistence**
   - Extend storage/model as needed so merge commits can record **two parents** (today MiniGit may only store one `parent_hash` — plan and implement an additive change; existing single-parent commits must keep working).

7. **Validation**
   - Follow `AGENTS.md`: hash format, ref name rules, parameterized SQL, layer boundaries (`operations` orchestrates; CLI/app stay thin).

### Acceptance criteria

- [ ] `minigit merge <branch>` merges source into current branch (FF or two-parent merge commit).
- [ ] Missing / unknown source ref or missing commit → **clear error**, no partial ref update.
- [ ] Diverged histories with path conflicts → **abort and report** conflicting paths; branch tip unchanged.
- [ ] Diverged histories without conflicts → new merge commit with **two parents**; current branch updated.
- [ ] Fast-forward case works and does not create an unnecessary merge commit.
- [ ] “Already up to date” is detected and reported.
- [ ] Flask UI can trigger the same merge and show success or error.
- [ ] Tests under `tests/` cover: FF, two-parent merge, missing ref, conflict abort, already-up-to-date (use `tmp_path`).
- [ ] `make check` passes.
- [ ] No secrets; follow MiniGit architecture rules in `AGENTS.md`.

### Implementation hints (agent)

- Prefer implementing in `frontend/operations.py` first, then CLI + Flask.
- Use TDD: failing tests for each acceptance path before code.
- Discover existing commit/ref APIs in `sqlite_client.py` / `Operations` before inventing parallel storage.
- Keep functions small; document schema change if adding a second parent field/table.

### Out of scope / ask first

- Changing conflict policy to auto-merge file text
- New third-party dependencies
- Deploy / CI beyond existing `make check`

### Definition of done

AC checked, tests green, CLI + UI paths demoable, and (if run via AI SDLC) Jira phase labels/comments updated by the factory.

---

## Short summary line (for Jira Summary field)

MiniGit: merge source branch into current branch (FF or two-parent merge commit; error if ref/commit missing; abort+report on conflict; CLI + Flask)
