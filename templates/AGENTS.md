<!--
Root AGENTS.md template. Replace every <placeholder>, keep only the rules you'd defend in a review, and keep it short:
agents read all of it on every task, so each line should be something an agent would get wrong without it. Rules for
one directory go in a nested AGENTS.md next to that code (see templates/nested/). The reasons behind each rule live in
principles/: link to them instead of copying them here. Delete this comment.
-->

# <Project name>

<One paragraph: what the product does and what lives where in this repository. For example: "Acme bills
subscriptions. This monorepo holds the Rails app (`app/`, `lib/`, `config/`), the React client (`client/`), the Go
webhook relay (`relay/`) and the public docs site (`docs/`).">

## Read the nested AGENTS.md before you work in a directory

Not every coding agent loads nested AGENTS.md files on its own. Before you change files under one of these directories,
read its AGENTS.md. Add any new nested AGENTS.md to this list in the same PR.

- `<db/>`: <safe migrations, column sizes, bounded queries>
- `<spec/>`: <running tests, what the suite enforces, fixtures>
- `<app/api/>`: <request schemas, what an API change ships with>

## Where knowledge lives

- **AGENTS.md files**: rules and pitfalls for the code next to them. When you learn a durable rule about this repo (a
  footgun, a convention someone corrected you on), add it to the nearest AGENTS.md in the same PR. Keep one copy of the
  rules, here: if a tool needs a `CLAUDE.md`, make it a symlink to AGENTS.md or a one-line `@AGENTS.md` import, never a
  second copy ([why][writing-agents-md]).
- **`<internal_docs/>`**: longer references and runbooks whose commands change with the code. Update them in the PR that
  changes the code they describe.
- **<Wiki>**: policies, the reasons behind decisions, product and customer context.
- **Pull requests**: the record of each change, with its description, review and QA results.
- **<Tracker>**: open work. Ticket IDs look like `<ENG-123>`.
- Don't leave anything the team needs only in a temp directory or in a personal AI tool's memory.

## Hard rules

- **Generated files:** never hand-edit one; change its source and regenerate. The full list with commands is in
  `<internal_docs/generated-files.md>`. The ones you'll meet most: `<db/schema.sql>` (write a migration and run it),
  `<openapi.json>` (<regenerate with ...>). <Files the dev server or test boot rewrites on their own: treat their diffs
  as generated output.>
- **Data in AI sessions:** use only the AI tools and data your organization has approved for the task. Keep customers'
  and end users' personal and confidential data out of sessions that aren't approved for it: select metadata and
  counts, use synthetic examples, and ask <the person who owns the data> when you're unsure.
- **End-user data in tickets and notes:** never copy it into tickets, wiki pages, PR descriptions or commit messages.
  Refer to records by ID or by a link to the support conversation. Trackers keep every version of a description.
- **Public posts** (upstream issues and PRs, forum posts, blog drafts): publish only what's approved for a public
  audience: no secrets, personal data, or confidential business or operational details. Use synthetic figures when you
  don't have permission for real ones.
- **Production access:** every command against production goes through <your audited access path>. Bound every query,
  even a read-only one (an indexed column, batches, a statement timeout). Never print config, environment or release
  objects in full. Never add a step to the deploy pipeline without explicit approval.
- <If your policy requires signed commits:> **Every commit is signed.** Agents that work unattended get their own
  credential, which can be revoked on its own. If you can't sign, leave the work uncommitted and say so. Never skip the
  pre-push hook.

## Git and pull requests

- Give each concurrent task its own checkout (`<scripts/start_ticket <id>>`, or `git worktree add`), and isolate any
  mutable services they share (databases, ports, buckets).
- Never use `git stash` in a worktree: every worktree of a repository shares one stash, so another session's
  `git stash pop` can take your changes. Set changes aside with a local WIP commit and `git reset HEAD~1` later.
- Commit messages, PR titles and PR bodies name only the PR's own ticket, or your tracker's integration attaches the PR
  to every ticket it finds.
- Never push to `main`: every change goes on a branch and into a PR.
- <Our choice:> bring a branch up to date by rebasing onto `main` and pushing with `--force-with-lease`, and let CI test
  the result before anything ships. Before rebasing a pushed branch, pull commits CI bots added with
  `git pull --rebase`. <When this doesn't fit: release branches or artifact promotion; test the merge result or the
  artifact you promote.>
- Every PR gets an AI code review before it merges, except PRs that change only dev tooling, developer docs or CI
  speed. Record it in one line of the PR description, kept current:
  `**AI review:** <tool> (3 rounds), last run on abc1234. 9 findings: 8 fixed, 1 dismissed.`
- Keep the PR description current: when later commits change what it describes, rewrite it. Read the current body
  first. Post QA results as a PR comment: date, commit, what passed, what wasn't tested.

## Commands

- Tests: `<bin/test path/to/file>`. Run the files you touched; <the full suite runs on CI>.
- Lint: `<bin/lint [file]>`.
- Quote heredoc delimiters (`<<'EOF'`) and write PR bodies to a file for `--body-file`. An unquoted heredoc runs every
  backticked command in its text.

### Standing approval for agents in ask-first mode

Agents that ask before running commands have standing approval to run these in this repository. The list doesn't
restrict agents with more permissive settings. It doesn't cover changing git state, deploying, destructive operations,
or writing to the tracker, GitHub or other services.

- Tests and lint: `<bin/test [path]>`, `<bin/lint [path]>`. Builds and type checks: `<...>`.
- Git, read-only forms only: `git status`, `git diff`, `git log`, `git show`, `git blame`, `git grep`, `git ls-files`,
  `git rev-parse`, `git merge-base`, `git branch --list`, `git worktree list`. Not `git fetch`, `pull`, `checkout`,
  `switch`, `add`, `commit`, `stash`, `reset`, `clean` or `push`.
- GitHub, read-only: `gh pr list`, `gh pr view`, `gh pr diff`, `gh pr checks`.
- <Name any command that looks read-only but writes, so nobody adds it here: for example a CI helper that reruns
  failed jobs.>

## Code rules

<Pick the rules you'd defend in a review and delete the rest. Each one links to the page with its reason and when it
doesn't fit; keep the link so the next reader can find out why.>

- **Make the smallest complete change:** reach every place the change must, fix what you find in the code you touch,
  and add nothing the ticket didn't ask for. ([change-size])
- **A review finding needs a trigger that can really happen.** Pre-existing problems in untouched code are reported,
  not fixed by default. ([change-size])
- **Dev and test code crashes fast; production degrades gracefully, except that it never produces <a document,
  invoice, report> with wrong or missing values.** ([errors])
- **When code mishandles a reasonable input, fix the code. Don't reject the input.** ([change-size])
- **Ship features without a feature flag, and roll back a bad release.** Ask before adding one. <Delete this rule if
  rollback can't undo what your changes do (data writes, calls to other systems) or your deploys are slow or rare.>
  ([change-management])
- **Never break an existing customer's integration.** Count who relies on a behavior before changing it.
  ([compatibility])
- **Bound regular expressions and resources on input a user sizes; run user-written patterns with RE2; parse untrusted
  files inside a sandbox that keeps an exploit away from your secrets.** ([untrusted-input])
- **Only dependencies under licenses you can ship.** <MIT, Apache, BSD; ask before anything else.> ([dependencies])
- **A new or tightened detector ships with every real finding fixed.** Suppress only false positives, with the reason.
  ([change-size])

[change-size]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/change-size.md
[errors]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/errors.md
[change-management]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/change-management.md
[compatibility]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/compatibility.md
[untrusted-input]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/untrusted-input.md
[dependencies]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/principles/dependencies.md
[writing-agents-md]: https://github.com/DocSpring/docspring-agent-playbook/blob/main/guides/writing-agents-md.md

## Facts agents often get wrong

<Domain facts that contradict what an agent would guess from names or from typical apps. Name the code that proves
each one. For example:>

- <Amounts are integers in the currency's smallest unit (`amount_cents`), never floats (`app/models/money.rb`).>
- <Soft-deleted rows are hidden by a default scope. Admin and export queries call `unscoped` on purpose.>

## Agent workflow settings

<The skills in docspring-agent-playbook read your tracker, CI, test and approval details from here. Copy the blank block
from CONFIGURE.md and fill it in.>
