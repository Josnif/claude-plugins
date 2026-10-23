# Handoff: pr-review-kit

_Started 2026-10-23._

## Why this exists

I reviewed many PRs with Claude, and every review follows the same pattern. This plugin captures it:

1. "Review branch X / PR #N and give your verdict."
2. "Give me the issues as **one comment**.
3. "Are these issues real and relevant?" This means proving them (build/tests in a worktree, Docker DB repro) and moving unrelated problems to separate GitHub issues.
4. "The developer pushed fixes; check them." A re-review against the earlier items, including commits pushed mid-review and merges from the base branch.
5. "It merged by mistake; anything still to review?" Find what was unresolved at merge, plus clashes created by two PRs merging together.
6. Context matters: "we're pre-production and moving fast" keeps real blockers but moves polish to follow-ups.

The built-in `/code-review` finds bugs but gives no verdict, writes no single pasteable comment, keeps no memory of earlier rounds, and doesn't filter for relevance. That gap is why this plugin exists.

## What's built (v0.1.0, untested)

- `plugins/pr-review-kit/skills/review`: first review, with `references/severity.md` and `references/comment-template.md`
- `plugins/pr-review-kit/skills/rereview`: round N+1 against `.pr-reviews/<branch>/r<N>.md`
- `plugins/pr-review-kit/skills/followups`: issue text; `--combined` and `--merged` modes
- Reviews are stored in `.pr-reviews/` in the **reviewed** repo, excluded locally via `.git/info/exclude`

## Next steps

1. **Install locally and dogfood**: `/plugin marketplace add ~/Projects/plugins`, then `/plugin install pr-review-kit@josnif-plugins`. Run it on a real branch in one of the user's projects, and compare the output with earlier hand-made reviews.
2. Check that `${CLAUDE_PLUGIN_ROOT}` resolves inside SKILL.md bodies (the rereview skill uses it to reach the review skill's references). If it doesn't, copy the reference files into each skill.
3. Decide whether to add a `--comment` mode that posts via `gh` after confirmation, or keep it copy-paste only (current choice).
4. Consider a small agent for the verification step (worktree build/test plus Docker repro), so the main context stays clean on large PRs.
5. Create the GitHub repo, then replace `<github-user>/<this-repo>` in both READMEs. The user commits and pushes themselves.
