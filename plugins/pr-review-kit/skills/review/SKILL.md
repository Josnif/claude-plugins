---
name: review
description: Review a pull request or branch for merge and produce a verdict (approve / approve with follow-ups / request changes) plus ONE copy-paste markdown comment for the PR, grouped by severity. Use when the user asks to review a PR or branch, asks "can this be merged?", or wants issues they can paste as a single PR comment.
argument-hint: "<branch | PR number | PR URL> [--base <branch>] [--audience <who reads the comment>]"
---

# PR review: verdict + one pasteable comment

The reviewer is usually a lead or teammate who will paste your comment into the PR. They want:

- a **clear merge verdict**,
- **one** markdown comment covering everything (never one comment per file),
- only **real, relevant** issues: evidence over opinion, scoped to this PR,
- wording that a developer can act on straight away.

Target: `$ARGUMENTS` (if empty, review the current branch against its base).

## 1. Establish scope before reading code

1. Resolve the target:
   - A PR number or URL: if `gh` is available (`gh --version`), use `gh pr view <n> --json title,body,baseRefName,headRefName,commits,files,url` and `gh pr diff <n>`. If `gh` is missing, say so in one line, ask for the branch name only if you cannot find it, then use git.
   - A branch: `git fetch --all --prune`, then use `origin/<branch>` when it exists, so you review what was pushed rather than a stale local copy.
2. Find the base. Use `--base`, else the PR's base, else the repo default branch (`git symbolic-ref refs/remotes/origin/HEAD`). Diff from the merge base: `git diff <base>...<head>`.
3. Record the head SHA you are reviewing. Every verdict is "at `<sha>`".
4. Size it up with `git diff --stat <base>...<head>` and `git log --oneline <base>..<head>`. Note if the branch is behind base (`git rev-list --count <head>..<base>`) or has conflicts (`git merge-tree --write-tree <base> <head>`, git ≥ 2.38).
5. Read the PR description, plus any plan, spec or ticket it links to or that the user pasted. **The stated intent defines "in scope".** If there is no description, infer the intent from the commits and say that you did.

## 2. Review

Read the diff with the surrounding code, not just the hunks. For each changed area, check:

- **Correctness**: logic, edge cases, error paths, concurrency, idempotency, time zones, money and units.
- **Data and migrations**: destructive changes, backfills, ordering against migrations already on base, re-runnability, whether production data could be lost.
- **Security and tenancy**: auth on new routes, client-supplied identity fields, cross-tenant or cross-account reads and writes, secrets.
- **Contracts**: API and response shape changes, callers in other services or the frontend, removed fields that old clients still send.
- **Ops**: config and env vars, Dockerfiles and build files for new services, timeouts and retries, deploy order.
- **Tests**: do the tests actually exercise the risky paths?
- **Drive-by changes**: edits outside the PR's stated purpose, especially in areas another team owns.

Read `references/severity.md` before classifying anything.

## 3. Verify: prove it before you claim it

A finding goes in the comment only once you have verified it. Verify in this order, and stop when you have proof:

1. **Trace it in code**: the exact file and line, and the call path that reaches it.
2. **Build, lint and test the head** in a throwaway worktree, never in the user's checkout:
   ```bash
   git worktree add --detach "$TMPDIR/pr-review-<sha>" <head>
   ```
   Detect the toolchain from lockfiles and manifests (pnpm/npm/yarn/bun, cargo, go, uv/pip/poetry, maven/gradle, make). Mirror what CI does (read `.github/workflows/*`, for example a `build` before `test` in monorepos). Run installs offline or with a frozen lockfile where possible.
3. **Reproduce at runtime** when a claim depends on data or HTTP behaviour: a throwaway database container (`docker run --rm` postgres/mysql/redis on a free port), run the migrations, start the service, hit it with curl. Check first that Docker is running (`docker info`). If it isn't, say so; don't ask the user to start it unless runtime proof is essential.
4. If a test fails, decide whether it is **environmental** (missing DB or env var, packages not built) or **real**. Compare against base before blaming the PR.
5. Remove the worktree and any containers you started.

Then **drop or downgrade** anything you could not substantiate. Pre-existing problems that this PR does not touch or make worse go under "Out of scope", not into the PR's findings.

## 4. Decide the verdict

- **✅ Approve**: nothing blocking or should-fix.
- **✅ Approve, with follow-ups**: no blockers; the remaining items can safely be issues.
- **🔴 Request changes**: at least one blocker, or should-fix items with no plan.

Weigh the user's context. If they say the project is pre-production or moving fast, keep real data-loss, security and correctness blockers but move polish to follow-ups. Say that you did.

## 5. Write it up

1. Save the review to `.pr-reviews/<branch-slug>/r1.md` in the repo root (use `r2.md`, … if earlier rounds exist). Make sure `.pr-reviews/` is listed in `.git/info/exclude` so it is never committed. The `rereview` skill reads this file later.
2. The file has two parts, using `references/comment-template.md`:
   - **Notes for the reviewer** (not pasted): what was run, what couldn't be run and why, and anything uncertain.
   - **The PR comment**: inside one fenced ` ````markdown ` block, so the user can copy it in one go.
3. In chat, reply with:
   - one line saying who the comment is written for (default: "the PR author and other reviewers"),
   - the verdict with a one-sentence reason,
   - checks run (build / lint / tests, with counts) and anything you could not run,
   - the full comment block,
   - the path to the saved review file.

### Comment rules

- Every finding names a `path:line`, says what goes wrong for whom, and says what to do. Give a test idea for anything blocking.
- Lead with the impact, not the mechanism. Plain words, short sentences, no hedging stacks.
- Number findings continuously across sections, so people can say "see #3".
- Start with one or two genuine strengths, specific rather than flattery.
- Never paste secrets, tokens or personal data into the comment.
- Process notes (PR too big, branch behind base, needs sign-off from another owner) go in their own small section and never block on their own.
