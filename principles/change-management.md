# Change management

### Never push to main.

Every change, however small (docs, agent rules, CI config), goes on a branch and into a pull request. CI jobs never
commit to main either: autofix jobs only check there.

**How to enforce it:** Branch protection on main, without admin bypass for routine work. Remember that agents push with
a person's credentials, so a person who can bypass protection gives their agents the same power: say it in AGENTS.md.

*Scope: Any team.*

### Test the exact revision you deploy.

Whatever your release flow, the revision CI tested is the revision that ships: not the branch before it was combined
with main, and not a merge result nobody ran.

What we do: we deploy the pull request's branch (staging, then production), and CI merges it into main only after the
deploy succeeds, so main is always what's in production. Branches take turns on the one shared staging environment
through a queue, and release jobs refuse a branch that is behind main.

**When this doesn't fit:** you merge before deploying, use release branches or promote a built artifact between
environments. Then test the merge result or the artifact you promote, and record who may merge and who may deploy.

*Scope: Any team.*

### Rebase onto main; never merge main into a branch.

Rebase (`git fetch origin && git rebase origin/main`), push with `--force-with-lease`, and let CI test the result
before anything ships. Rebasing combines your changes with main just as a merge does; the point is that the combined
result is the branch CI tests, with a linear history that shows exactly what it tested. Before rebasing a pushed
branch, pull the commits CI bots added (a formatter, generated files) with `git pull --rebase`: `--force-with-lease`
after a fetch won't stop a push that drops them.

**How to enforce it:** Release jobs refuse a branch that is behind main.

**When this doesn't fit:** release branches, where fixes are merged or cherry-picked between lines. Then test each
line's result before it ships.

*Scope: Any team.*

### Every pull request gets an AI code review, recorded in one line.

Run a code-review tool with the whole repository as context, and fix each finding or say why it doesn't apply. Record
the reviews in a single line of the pull request description, kept current as rounds run, instead of a comment per
round:

```markdown
**AI review:** <tool> (4 rounds) and <second tool> (2), last run on abc1234. 14 findings: 12 fixed, 2 dismissed.
```

The full notes stay in the reviewer's own files. Pull requests that change only dev tooling, developer docs or CI speed
can skip it.

*Scope: Any team.*

### Read authorization from outside the change.

Lists that decide who may approve, merge or sign live in the repository, and CI reads them from the default branch,
never from the branch under test. Otherwise a pull request can approve itself by editing the list. The same goes for
any check a change can switch off: an intended exception goes through your normal review and leaves a record, never
through something the change itself contains.

*Scope: Any team.*

### If your policy requires signed commits, give agents their own credential.

An agent that commits gets its own signing key, separate from any person's, so its commits are attributed to it and the
key can be revoked on its own. Never commit unsigned to get past a missing key, and never skip the pre-push hook: if
the agent can't sign, it leaves the work uncommitted and says so. Merge pull requests with merge commits if your host's
"rebase and merge" re-creates commits without their signatures.

**How to enforce it:** A CI check that verifies every commit on the branch against the list of allowed signers (read
from the default branch), and the same check in a pre-push hook.

*Scope: Teams whose change-management policy requires signed commits.*

### Dependency update pull requests deploy as they are.

Let the update bot sign its own commits (if you sign) so its pull requests can deploy once they pass; never design a
flow where a person re-signs or re-pushes them. Never disable a dependency's updates because one fails CI: fix the
code, or leave the pull request open, since disabling also stops its security updates. Hold back routine updates with
an approval step that security updates skip.

*Scope: Any team.*

### Give each parallel task its own checkout and its own mutable services.

Agents working in parallel each get a branch in their own worktree or checkout, and anything mutable they would
otherwise share (development databases, cache databases, ports, storage buckets) gets one copy per checkout. Create and
remove them with a script, so freeing a checkout frees its services too, and derive nothing from the directory name:
list the worktrees instead. If you clone development databases, clone from a copy that only holds migrations that are
on main, so one branch's tables don't leak into another's schema dump.

**When this doesn't fit:** stateless tools, or containers and remote environments that already isolate each task. Then
use your normal checkout workflow.

*Scope: Any team that runs agents in parallel.*

### Never use `git stash` in a worktree.

Every worktree of a repository shares one stash list, so another session's `git stash pop` can take your changes into
its worktree, or yours can take theirs. To set changes aside, make a local WIP commit
(`git add -A && git commit -m WIP`, later `git reset HEAD~1`), which keeps staged, untracked and binary changes too. To
run a test against the old code, copy your file aside and restore the old one with `git show HEAD:<path> > <path>`.

*Scope: Any team that runs agents in parallel.*

### Name only the pull request's own ticket in commits and pull requests.

Tracker integrations attach the pull request to every ticket ID they find and move those tickets through the workflow.
Write other tickets in a form the integration doesn't parse ("ticket 1234" instead of the ID format). A pull request
that fixes several tickets owns all of them: each commit names its own ticket, and the body lists them all.

*Scope: Any team using a tracker integration.*

### Keep the pull request description current.

When later commits change what the description says, rewrite it, after reading the current body. Write bodies to a file
and pass it with `--body-file`, so the shell doesn't run anything inside them.

*Scope: Any team.*

### Post QA results as a pull request comment.

Give the date, the commit or release, what passed and what wasn't tested. No secrets or customer data. The pull request
is the record of the change, and a QA result anywhere else is lost.

*Scope: Any team.*

### Change docs, skills and release notes in the pull request that changes the behavior.

API docs, agent skills, runbooks, the list of places that describe what the product does, and release notes for
customers who upgrade on their own all change with the code. Write release-note bullets ready to send.

When docs and behavior disagree, find the intended contract. Fix wrong docs straight away. Build the missing behavior
only when the task's scope covers it; otherwise describe the current limitation accurately and record the gap.

*Scope: Any team.*

### Ship new features without a feature flag.

A feature goes live with the deploy that ships it, and a bad release is rolled back. That only works when rollback is
quick and safe, so keep it that way. Ask before adding a flag: each one is a second code path to test and a cleanup
nobody schedules. An environment variable that points at optional infrastructure (a queue, a separate service), so an
install without it keeps working, is configuration, not a rollout flag.

**When this doesn't fit:** the change writes data or calls external systems that a rollback can't undo, or your deploys
are slow or rare. Then use a flag with an owner, tests for both states, and a removal date.

*Scope: Teams that deploy continuously.*
