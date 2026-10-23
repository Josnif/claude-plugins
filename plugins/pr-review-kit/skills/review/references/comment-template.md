# Review file template

Saved to `.pr-reviews/<branch-slug>/r<N>.md`. Only the fenced block under "PR comment" gets pasted.

---

# Review r<N>: <branch> → <base> (PR #<n>, if known)

- Head: `<sha>` · Base: `<base>@<sha>` · Date: <YYYY-MM-DD>
- Verdict: <✅ Approve | ✅ Approve, with follow-ups | 🔴 Request changes>

## Notes for the reviewer (not for pasting)

- Ran: build ✅/❌ · lint (<errors> errors, <warnings> warnings) · tests (<suite>: <pass>/<total>, …)
- Couldn't run: <what> because <why>
- Uncertain: <anything you'd want a human to double-check>

## PR comment

````markdown
## Review: <Approve | Approve, with follow-ups | Request changes>

<1–2 sentences: what's good, specifically. Then what stands between this and merge.>

**Checks:** build ✅ · lint ✅ · tests ✅ <suite> <n>/<n>, … · not run: <what> (<why>)

### 🔴 Blocking

**1. <Impact in one line>**
`path/to/file.ts:123`
<What happens, the concrete scenario, why it matters.>
Fix: <what to do>. Test: <the test that would have caught it>.

### 🟠 Should fix before merge (or open a ticket and link it here)

**2. <Impact>**
`path:line`: <scenario> → <fix>

### 🟡 Minor / non-blocking

- **3. <Impact>** (`path:line`): <one or two lines>

### Process

- <branch is N commits behind base / needs sign-off from X / PR size>

Happy to re-review once <#1> is fixed and <#2> is fixed or ticketed.
````

Leave out any empty section. For an approval, drop "Blocking" and title the follow-ups "Follow-ups (non-blocking; please open issues)".
