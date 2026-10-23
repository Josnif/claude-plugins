---
name: followups
description: Turn the non-blocking, ticketed and out-of-scope items from a saved PR review into ready-to-file GitHub issues (title + markdown body), either one issue per item or one combined tracking issue. Also checks a PR that already merged for review items that never got fixed. Use when the user asks for issues to create from a review, a "general issue" covering several branches, or what is still open after a merge.
argument-hint: "<branch | review file> [--combined] [--merged]"
---

# Follow-up issues from a review

Input: `$ARGUMENTS`. This is a branch name (use its latest `.pr-reviews/<branch-slug>/r<N>.md`) or a path to a review file. If nothing is saved, ask the user to paste the review comment.

## 1. Collect the items

Take every item that is:
- 🟠 or 🟡 and not marked ✅ Fixed in the latest round,
- 🎫 Ticketed, but no issue link exists yet,
- in the "Out of scope" section.

With `--merged`, or when the PR is already merged (`git branch -r --merged <base>`, `gh pr view --json state`), first check each **open** item, including 🔴 ones, against the current base branch. Drop the ones that were fixed before merge, and list the ones that weren't at the top as **unresolved at merge**.

Also look for problems that only appear **because of the merge**: two PRs that each look fine alone but clash on the base branch. Typical examples are the same logic computed in two places, migrations that overlap, or one route that another PR changed underneath it.

## 2. Write the issues

Default: one issue per item. With `--combined`, or when the user wants one "general issue", write a single tracking issue with a task list grouped by area, so it can be split up later.

Per issue, use this format:

```markdown
**<Title: the impact, in under 70 characters>**

**Where:** `path:line` (from PR #<n> / branch `<branch>`)
**What happens:** <the scenario, in 1–3 sentences>
**Expected:** <the correct behaviour>
**Suggested fix:** <one or two lines; leave it out if it's a product decision>
**Labels:** <bug | security | tech-debt | product-decision> · <severity>
```

Mark product decisions as such (for example, which definition a metric should use) rather than calling them bugs.

## 3. Output

- Save to `.pr-reviews/<branch-slug>/issues.md`.
- In chat: one line saying who it's for, then the issues as one copy-paste block per issue (or one block for `--combined`).
- If `gh` is available, offer the `gh issue create --title … --body-file …` commands, but **never run them without explicit confirmation**. Creating issues is public and visible to the team.
