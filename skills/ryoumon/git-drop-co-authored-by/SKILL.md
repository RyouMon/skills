---
name: git-drop-co-authored-by
description: >-
  Removes lines starting with Co-Authored-By from Git commit messages via
  git filter-branch and sed /^Co-Authored-By/d. Always shows the execution plan
  and the exact list of commits to rewrite for user review first; runs history
  rewrite only after explicit user approval. Use when the user asks to delete
  signatures from commit messages, strip Co-Authored-By trailers, or clean
  commit footers.
disable-model-invocation: false
---

# Git: strip Co-Authored-By lines from commit messages

## When to use

Follow this skill when the user wants to **remove signature / trailer lines** (e.g. `Co-Authored-By: …`) from commit messages using **Git commands that rewrite history**. **Do not** run any history-rewriting command until the user has explicitly approved the shown plan and commit list.

## Rules (aligned with the repo plan)

- **Deletion rule**: delete any **full line** that starts with `Co-Authored-By` (`sed`: `/^Co-Authored-By/d`). No leading whitespace before `C`; if indented trailers must match, use `/^[[:space:]]*Co-Authored-By/d` instead and state that explicitly.
- **Effect**: rewritten commits get **new SHAs**; only the message changes, not the tree.

## Mandatory workflow (review before rewrite)

### 1. Read-only: fix the range and list every commit to rewrite

- If the user gave a range (e.g. `A^..B`, two SHAs, or `merge-base..HEAD`), use it; otherwise ask which **slice of history** to process (e.g. from commit X through `HEAD`, or unpushed commits relative to `origin/main`). If they say **“all commits”**, clarify whether that means **the entire current branch from root to `HEAD`** or **commits since some merge base**, so you do not scan the wrong history. List **every** commit in range (if huge, add `git rev-list --count`, first/last N SHAs, plus a scripted hit summary—but the user must still be able to audit).
- Use read-only Git to list each commit and whether the message **currently contains** a line starting with `Co-Authored-By` (e.g. `git log --format` showing subject and matching lines, or spot-check `git log -1 --format=%B` per commit).
- **Do not** run `git filter-branch`, `git filter-repo`, or history-rewriting `git rebase` in this phase.

### 2. Present “plan + commits to touch” for review

In one reply, use a **fixed structure** so the user can review quickly:

1. **Filter to apply**: one line stating `--msg-filter` + `sed '/^Co-Authored-By/d'` (or the agreed variant).
2. **Commits to rewrite**: table or list; each row at least **full SHA (or 40 chars)**, `git log -1 --oneline` title, and whether a `Co-Authored-By` line was detected (yes/no).
3. **Command draft** (copy-paste complete): optional `git branch backup/...`, `git filter-branch ... refs/heads/<branch> <rev-list>`, `refs/original` cleanup and `gc` if using filter-branch.
4. **Stop**: state clearly that you will **not** run any rewrite until they confirm (e.g. “Confirm whether to run the rewrite with the range and commands above.”).

### 3. Continue only after explicit approval

Only after the user **explicitly confirms** (e.g. “go ahead”, “confirmed”):

1. **(Required)** `git branch backup/<name>-before-strip-coauthor HEAD`
2. Run the **same** `git filter-branch` (or documented `git filter-repo`) command the user reviewed; `<rev-list>` must match what was shown.
3. If `filter-branch` was used: remove backup refs under `refs/original/` and run `git reflog expire` / `git gc --prune=now` as promised in the draft.
4. Spot-check: sample new SHAs with `git log -1 --format=%B` and confirm no line starts with `Co-Authored-By`.

This skill **does not** include push; `git push` / `--force-with-lease` after rewrite is the user’s choice unless they ask otherwise.

## Command template (filter-branch)

Replace `<branch>` and `<rev-list>` with the values already shown and approved (Git Bash or any shell where `sed` is on `PATH`):

```bash
git filter-branch -f --msg-filter \
  'sed "/^Co-Authored-By/d"' \
  refs/heads/<branch> <rev-list>
```

`<rev-list>` example: `abc123^..def456` (inclusive range semantics as agreed with the user).

## Notes

- If the range includes **already pushed** commits, a force-push and coordination may be needed—warn during the review step.
- The rule is broader than deleting a single email: **any** full line starting with `Co-Authored-By` is removed.
