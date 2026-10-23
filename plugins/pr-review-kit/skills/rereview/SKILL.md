---
name: rereview
description: Re-review a PR or branch after the author pushed fixes. Checks every item from the previous review (fixed / partly fixed / still open / no longer relevant), reviews only what changed since, catches commits pushed mid-review, and produces an updated verdict plus ONE copy-paste markdown comment. Use when the user says the developer addressed comments, asks to "check the changes", "review again", or asks whether a new commit was taken into account.
argument-hint: "<branch | PR number> [--previous <path to earlier review file>]"
---

# Re-review: check the earlier items, then what's new

Target: `$ARGUMENTS` (if empty, use the current branch).

## 1. Load the previous round

1. Find the latest `.pr-reviews/<branch-slug>/r<N>.md`, or the file given with `--previous`.
2. If none exists, ask the user to paste their previous PR comment. If they have none, run the `review` skill instead.
3. Pull out every numbered finding with its severity, plus the head SHA it was reviewed at (`<old-sha>`).

## 2. Work out what changed

```bash
git fetch --all --prune
git log --oneline <old-sha>..<new-head>          # the new commits
git diff <old-sha>..<new-head> --stat
```

- **Merges from base**: when a new commit merges the base branch in, separate base's changes from the author's conflict resolution. Review the resolution and any adaptation the merge forced; don't re-review base. `git diff <base>...<new-head>` before and after the merge shows what the PR itself now changes.
- **Rebases or force-pushes**: if `<old-sha>` is no longer an ancestor, compare the PR's diffs instead: `git range-diff <base>...<old-sha> <base>...<new-head>`. Point out duplicate commits if the history now contains the same change twice (a squash-merge fixes it).
- If the branch moves while you're reviewing, check `git rev-parse origin/<branch>` again before writing up and cover the newest commits. State the SHA you reviewed.

## 3. Check every previous item against the code

Check the **code and tests**, not the commit messages. For each item, choose one:

- ✅ **Fixed**: say how, in one line, and name the test that covers it if there is one.
- 🟡 **Partly fixed**: what's left.
- ❌ **Still open**: unchanged, or the fix doesn't work. Give the scenario that still fails.
- ➖ **No longer relevant**: the code it applied to was removed or redesigned (say why).
- 🎫 **Ticketed**: the author linked an issue. Accept it for 🟠/🟡 items, not for 🔴.

Reproduce wherever the original finding was reproduced (tests, a throwaway DB, curl), using the `review` skill's verification method: a throwaway worktree and containers, cleaned up afterwards.

## 4. Review the new code

Review only the new changes, with the same rigour and severity scale as the `review` skill (`${CLAUDE_PLUGIN_ROOT}/skills/review/references/severity.md`). Fixes often introduce new problems: look for regressions in the fix itself and for edge cases the fix created (new statuses with no state machine, new batching boundaries, new config with no defaults).

## 5. Verdict and write-up

Save the review as `.pr-reviews/<branch-slug>/r<N+1>.md` in the same format as the `review` skill (`${CLAUDE_PLUGIN_ROOT}/skills/review/references/comment-template.md`), with the status table first in the comment:

````markdown
## Re-review at `<sha>`: <verdict>

<One line: what the new commits did.>

**Checks:** …

| # | Item | Status |
|---|------|--------|
| 1 | <short title> | ✅ Fixed: <how> |
| 2 | <short title> | ❌ Still open: <scenario> |

### Before merge: fix or open an issue
…

### New in this round
…

### Follow-ups (non-blocking; please open issues)
…
````

Keep the original numbers for earlier items, and continue the numbering for new ones.

In chat: one line saying who the comment is for, then the verdict, what changed since last time, the checks run, the comment block and the saved file path. If the user asked whether a specific commit was included, answer that first, in one sentence.
