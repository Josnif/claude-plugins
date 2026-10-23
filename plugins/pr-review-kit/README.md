# pr-review-kit

PR reviews that end in a **merge verdict** and **one markdown comment you can paste straight into the PR**, rather than a scatter of inline notes.

| Skill | Use it when | You get |
|---|---|---|
| `/pr-review-kit:review <branch\|PR#>` | A PR is ready for review | A verdict (approve / approve with follow-ups / request changes), a single PR comment grouped 🔴 / 🟠 / 🟡, and the build/lint/test results |
| `/pr-review-kit:rereview <branch\|PR#>` | The author pushed fixes | A status table for every earlier item (✅ / 🟡 / ❌ / ➖ / 🎫), a review of only the new code, and an updated verdict |
| `/pr-review-kit:followups <branch> [--combined] [--merged]` | Non-blocking items should become issues, or a PR merged before it was ready | Ready-to-file issues (title + body), and the items still unresolved after the merge |

Claude also picks these up without the slash command when you ask "can this branch be merged?" or "check the fixes on PR #42".

## What makes it different

- **Evidence first.** Findings are traced to a line and, where possible, proven by building and testing the head in a throwaway `git worktree`, or by reproducing against a throwaway Docker database. Anything it can't prove is dropped or downgraded.
- **Scoped to the PR.** Problems that already exist on the base branch go under "Out of scope / separate issue" and never block the PR.
- **Remembers rounds.** Every review is saved to `.pr-reviews/<branch>/r<N>.md` (excluded from git locally), so a re-review checks the exact items you raised last time.
- **Handles real-world git.** Merges from base, rebases, force-pushes, and commits pushed while the review is running.
- **Never touches your checkout or posts anything.** It doesn't commit, push, or create comments or issues; you copy and paste.

## Install

```text
/plugin marketplace add <github-user>/<this-repo>
/plugin install pr-review-kit@josnif-plugins
```

To try it locally from a clone:

```text
/plugin marketplace add /path/to/plugins
/plugin install pr-review-kit@josnif-plugins
```

## Requirements

- `git` (≥ 2.38 for conflict detection with `merge-tree --write-tree`)
- Optional: [`gh`](https://cli.github.com/), to read PR descriptions and diffs by number
- Optional: Docker, for runtime reproduction against a throwaway database

## Customising

- Severity rules: [`skills/review/references/severity.md`](skills/review/references/severity.md)
- Comment layout: [`skills/review/references/comment-template.md`](skills/review/references/comment-template.md)

Edit those two files to match your team's tone and bar for "blocking".
