# Merge feature (BLRID-2)

## Problem Statement
How Might We let MiniGit users merge one branch into their current branch with clear success and failure behavior (FF, two-parent merge, missing-ref errors, conflict abort)?

## Recommended Direction
Implement `Operations.merge(source)` plus CLI `minigit merge` and Flask UI. Prefer fast-forward when possible; otherwise create a merge commit with two parents. On path conflicts, abort and report without updating refs. Extend commit storage additively for a second parent while keeping existing single-parent commits working.

## Key Assumptions to Validate
- [ ] Additive two-parent storage is acceptable (column or parents table)
- [ ] “Unrelated histories” (no common ancestor) should error clearly
- [ ] Source may be branch name first; commit-hash source is nice-to-have if refs already resolve

## MVP Scope
- Resolve source ref; error if missing
- Detect already-up-to-date / FF / diverged
- FF update tip; diverged clean merge commit (2 parents)
- Conflict abort + report paths
- CLI + Flask
- Tests for the above; `make check`

## Not Doing (and Why)
- Interactive / textual 3-way merge editor — out of ticket scope
- Exotic Git merge strategies — educational clarity first
- Deploy beyond `make check`
