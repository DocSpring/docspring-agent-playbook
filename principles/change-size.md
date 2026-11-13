# Change size and scope

### Make the smallest complete change.

The scope is complete: the change reaches every place it must (callers, sibling entry points, the other client or
evaluator that applies the same rule, tests, docs) and fixes the problems you find in the code the change touches, all
in one pull request, with no phases or follow-up tickets. The solution is small: use what the codebase already has, and
add no option, setting, compatibility path, backfill or repair task that the task didn't ask for. When stored data needs
repairing, ask the user. Don't build anything to keep an old release line maintainable (an old SDK major, an old embed
version, a legacy mode) unless the task asks for it. Edge cases in the code you add still get handled and tested.

Problems you find outside the code the change touches don't join the pull request on their own. Record each one with
what you saw and a recommended next step (fix now, a ticket, or leave it), and let the user decide.

Agents fail in both directions. They miss the second entry point that applies the same rule, so the bug survives on
one path, and they add options, shims and migrations nobody asked for, which someone then has to review, test and
maintain. Naming both failures in one rule stops either from passing as thoroughness.

Before you write new code, work down this list and stop at the first step that does the job:

1. Check whether the task needs code at all: a setting, a docs fix or deleting code may be enough.
2. Use what the codebase already has: the existing helper, component, per-account gate, scheduler or job hook.
3. Extend it generically, with a variant or parameter its other callers could use too.
4. Write new code, as little as the task needs.

We sharpened this rule after evaluating [Ponytail](https://github.com/DietrichGebert/ponytail) (MIT), whose lean
checklist covers similar ground.

**How to enforce it:** Before implementing, list every caller and sibling entry point of the code you're changing, and
check the plan reaches each one. In review, ask of each new option, setting or compatibility path which line of the
ticket required it.

**When this doesn't fit:** a change that has to roll out in steps for compatibility (expand, migrate readers, then
contract) is one change in several pull requests. Plan the steps up front and name them in the first pull request.

*Scope: Any team.*

### A review finding needs a trigger that can really happen.

A finding (yours, a subagent's or another review tool's) names an input that a real user, a file from the wild, an
integration or a concurrent job could produce. Hostile input counts as a real trigger when it reaches a security,
availability or data-integrity boundary: someone will construct it on purpose. Be thorough with code the change adds,
where nobody knows yet which edges matter, and with anything that could put a wrong or missing value in your output (a
generated document, an invoice, a signed file, an audit record) or lose or expose data: fix those even when the case is
rare. Don't fix by default an input someone constructed that no real system produces and no attacker gains from: list
it as "not fixing", with the reason, so the user sees it and can disagree. A real problem in code the change didn't
touch goes under "Found, not in this change", with a recommended next step, never under "Checked and fine". When a fix
would be bigger than the code it reviews, or the finding is about code an earlier review round added, put it to the
user as a decision.

Each speculative fix looks cheap on its own, and review rounds compound: a fix for one edge case gets reviewed and
grows its own edge cases. Every added line is code that has to be read, tested and kept working, and can itself be
wrong.

**How to enforce it:** Put the rule in your review prompt or review skill, and require each finding to state its
trigger. Record the outcome on the pull request ("14 findings: 12 fixed, 2 dismissed"), so dismissed findings are
visible rather than silently dropped.

*Scope: Any team.*

### Fix a flaky test you hit in the branch you're on.

A flaky test you hit gets fixed in the branch you're on, even when your change didn't cause it, without a ticket, when
the fix is small and you can show the cause. Rerunning the job gets the branch through CI meanwhile. When you can't
find the cause, or the fix is bigger than your change, report it with the evidence (the failing runs, the seed, the
log) and ask. Follow-up tickets for flakes rarely get done, and every flaky test teaches people to ignore red builds.

*Scope: Any team.*

### Ship a new or tightened detector with every real finding fixed.

When you add a linter rule, an N+1 query detector or stricter type checking, fix every real finding in the same pull
request. Suppress only false positives, each with a comment on the line saying why. A detector that ships with a pile
of suppressions and a follow-up ticket teaches everyone that suppressing is the normal response.

**How to enforce it:** Require a reason on every inline suppression: for example, ESLint's
`eslint-comments/require-description` and `@typescript-eslint/ban-ts-comment` with `allow-with-description`, or a CI
check that fails a suppression comment with nothing after it.

**When this doesn't fit:** the detector finds more than one pull request can safely change. Then fix one area at a
time, with the detector scoped to the areas already fixed, and widen its scope in each pull request.

*Scope: Any team.*

### When code mishandles a reasonable input, fix the code.

Rejecting a valid input moves the bug onto your users. The same goes for size limits: when a stored value is far over
its limit, the bug is usually in whatever wrote it (a whole tool output dumped into a column, base64 in a log line).
Fix the writer instead of raising the cap, and set each limit by who reads the value and what helps them, not by the
largest value already stored.

*Scope: Any team.*

### Extend the tool instead of bending the product to fit it.

Code generators, linters and SDK builders support what the product has needed so far. When the product needs something
new, extend the tool generically and keep product names out of it. Never change an API's shape to suit a generator or
one client language, and check what the tool really produces before deciding it can't do something. A shape chosen to
please a tool becomes a public contract you can't take back.

*Scope: Any team.*

### Write settled decisions down next to the code.

Record "settled", "tried and dropped" and "intended behavior, not a bug" in the AGENTS.md beside the code they're
about, with the reason in a sentence. For a rejected approach, also say what would justify revisiting it, and don't
propose it again unless you can say what has changed. Without the record, every new agent session rediscovers the same
"improvement" and reports the same "bug", and someone has to explain it again.

```markdown
- Settled: the import overwrites existing settings instead of merging them, because merged settings broke round
  trips. Don't report it as a bug.
- Tried and dropped: caching the rendered list, because bulk deletes left it serving stale counts. Worth revisiting if
  bulk deletes start invalidating the cache.
```

*Scope: Any team.*
