# Severity guide

Classify by **consequence if merged as is**, not by how hard it is to fix.

## 🔴 Blocking (must fix before merge)

- Wrong results that users or money depend on: totals, balances, fees, permissions.
- Data loss or corruption, or a migration that fails or is unsafe on existing data.
- Security: unauthenticated write routes, trusting client-supplied identity, reads or writes across tenants or accounts.
- The build, existing tests, or the deploy path breaks.
- The PR does not do what its description says on the main path.

## 🟠 Should fix (fix now, or open a ticket and link it in the PR)

- Correct today but breaks under a realistic next step: reassignment, retries, scale past a page or batch limit, concurrent requests.
- Contract changes with callers that weren't updated or confirmed.
- Missing operational pieces: a Dockerfile or build config, env var docs, timeouts.
- Changes to code another team owns, without their sign-off.
- Removed TODOs or guards whose underlying problem still exists.

## 🟡 Minor / follow-up (non-blocking; fine as issues)

- Inconsistent metrics definitions, naming, comments that have gone stale.
- Missing tests for non-critical paths.
- Observability: errors that are only logged.

## Out of scope (separate issue, not this PR)

- Problems that exist on the base branch and that this PR neither touches nor makes worse.

## Relevance test

Before keeping a finding, answer all three:

1. Did this PR introduce it, or make it worse? (If not → Out of scope.)
2. Can I point to the line and the scenario that triggers it? (If not → drop it.)
3. Would the author agree it's a problem once they read the scenario? (If unsure → lower the severity, or phrase it as a question.)
