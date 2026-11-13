# Configuring the skills for your repository

The skills in `skills/` read your tracker, code host, CI, test and approval details from one section of your
repository's root `AGENTS.md`, called `## Agent workflow settings`. Copy the blank block below into it and fill in
what applies, with commands you've checked in your repository. Leave out what you don't use: a skill that needs a
missing setting asks you once and offers to add your answer to the section.

## What works out of the box, and what needs an adapter

The workflow instructions work with any tracker, CI provider and test runner. Two pieces are code, and they ship for
one tool each:

- **The CI waiter** (`skills/fix-ci/scripts/wait_for_ci`) watches a commit's GitHub Actions runs, including the
  required workflows you name; fix-ci then confirms the branch's required checks with the code host. For another
  provider, write a waiter with the same contract (wait until every run on the commit has finished and the required
  ones have run, exit non-zero if any failed, can't be read or the deadline passed) and check it on a real failing run
  before you rely on it.
- **The second-reviewer launcher** (`skills/second-opinion-review/scripts/start_review`) runs the Codex CLI. The
  "Second reviewer" CLI setting records your choice; for another CLI, change the launcher too.
- The pull request steps in fix-ci, pr-review and session-wrap-up use the GitHub CLI (`gh`) as their worked example. On
  another code host, fill in the "Code host" commands and check each one.

Copying the block installs nothing. A command it names must exist in your repository: save and test any example script
before you list it.

## Who approves what

1. The user's instruction in the current task wins.
2. Otherwise the "What the agent may do without asking" settings apply.
3. Invoking a skill approves the actions its description names (fix-ci: committing and pushing the branch it's
   fixing), unless the settings say that action always needs asking.

The agent asks for anything else, and doesn't ask again for something already approved.

The agent reads the "What the agent may do without asking" settings from the root AGENTS.md on the base branch (the
pull request's base, otherwise the repository's default branch), for example with `git show origin/main:AGENTS.md`,
not from the checkout. A branch, and especially someone else's pull request, can change its own copy, so a change to
those settings takes effect only once it's merged. A review reports such a change as a finding. The other settings
(commands, folders) come from the checkout as usual.

## The review folder for a branch

story-kickoff, pr-review and second-opinion-review keep their records for a branch in one shared folder:

```
<review folder root>/<head-owner>/<branch>/
```

- **Review folder root**: the setting below, by default `~/.agent-reviews/<owner>/<repo>/`, named after the base
  repository (the one the PR merges into) as the code host spells it, not after the local folder.
- **head-owner**: the owner of the repository the branch lives in: the base repository's owner for a branch in the
  same repository, the fork's owner for a branch in a fork.
- **branch**: the full branch name, with each `/` written as `%2F` (`feature/login` becomes `feature%2Flogin`), so two
  different branches never share a folder.

`review.md` records the identity (base repository, head repository, branch) and the folder's absolute path. Before
reading decisions from an existing folder, a skill checks that its recorded identity matches the branch it's working
on, and stops to ask if it doesn't.

## The blank block

```markdown
## Agent workflow settings

### Tracker
- Tool:
- Read a ticket:
- Search tickets:
- Create a ticket:
- Ticket IDs look like:
- Move a ticket to started:
- Comment on a ticket:

### Code host
- Tool:
- Find the pull request for a branch:
- Read review threads:
- Reply to and resolve a thread:
- Post a comment or review on a pull request:

### Branches
- Start work on a ticket:
- Remove a merged branch's checkout:

### Review records
- Review folder root (each branch's folder is `<root>/<head-owner>/<branch>/`, see above):
- Shared copy:

### CI
- Provider:
- Wait for the runs on a commit:
- Required workflows and checks:
- Failed jobs' logs:
- Jobs that commit their own fixes:
- Usual pipeline time:

### Tests
- Run one file:
- Local concurrency:
- Browser tests without a visible window:

### Local stack and QA
- Start the app:
- Test users and data:
- Themes and widths to check:
- Screenshots and reports go to:
- QA reports are published to:

### Second reviewer
- CLI:
- Run during self-review:
- Model and settings:
- Access:
- Also keep from the reviewer:

### Knowledge sources
- Read-only sources an investigation may use:

### What the agent may do without asking
- Commit locally:
- Push the branch and open a PR:
- Edit the PR description:
- Create tickets:
- Move a ticket's state or comment on a ticket:
- Remove a merged branch's checkout:
- Post review comments or reply to threads:
- Resolve review threads:
```

## A worked example

One team's filled-in block, for a repository on Linear, GitHub and GitHub Actions. Every command here is an example:
`scripts/start_ticket` and `scripts/remove_worktree` stand for scripts your repository would have to provide, and the
permissions are one team's choices, not defaults.

```markdown
## Agent workflow settings

### Tracker
- Tool: Linear
- Read a ticket: `linear issue view <id>`
- Search tickets: `linear issue list --search "<words>"`
- Create a ticket: `linear issue create --title ... --description ...`, in the Triage state, with a priority
- Ticket IDs look like: `ENG-123`. Commits and PR titles name only the PR's own ticket.
- Move a ticket to started: `linear issue update <id> --state "In Progress"`
- Comment on a ticket: `linear issue comment add <id> --body ...`

### Code host
- Tool: GitHub, with the `gh` CLI
- Find the pull request for a branch: `gh pr view --json number,url`
- Read review threads: `gh api graphql` (see skills/fix-ci)
- Reply to and resolve a thread: `gh api graphql` (see skills/fix-ci)
- Post a comment or review on a pull request: `gh pr comment <n> --repo <owner>/<repo> --body-file <file>`, or
  `gh pr review <n> --repo <owner>/<repo> --comment --body-file <file>`

### Branches
- Start work on a ticket: `scripts/start_ticket <id>` (creates the branch and a worktree with its own database)
- Remove a merged branch's checkout: `scripts/remove_worktree <path>`

### Review records
- Review folder root: `~/.agent-reviews/<owner>/<repo>/` (the default; each branch's folder is
  `<root>/<head-owner>/<branch>/`, see above)
- Shared copy: none

### CI
- Provider: GitHub Actions
- Wait for the runs on a commit: `skills/fix-ci/scripts/wait_for_ci --require CI` (copied into the repository and
  tested), then `gh pr checks <pr> --required`
- Required workflows and checks: the `CI` workflow; branch protection requires `CI / test` and `CI / lint`
- Failed jobs' logs: `gh run view <run-id> --log-failed`
- Jobs that commit their own fixes: the formatter job, the generated-types job
- Usual pipeline time: 8 minutes

### Tests
- Run one file: `bin/test path/to/file`
- Local concurrency: one integration or browser test file at a time; unit tests in parallel
- Browser tests without a visible window: `HEADLESS=1`

### Local stack and QA
- Start the app: `bin/dev` (it prints the URL)
- Test users and data: `bin/seed-qa`
- Themes and widths to check: light and dark; 1280, 768 and 375 pixels wide
- Screenshots and reports go to: `~/qa/<repo>/<branch>/`
- QA reports are published to: a PR comment with images

### Second reviewer
- CLI: `codex exec`
- Run during self-review: for critical fixes or changes needing a careful review; otherwise only when asked
- Model and settings: recorded in the review folder
- Access: sandboxed (the reviewer works in a separate checkout inside its output folder, and its sandbox writes
  only there and to the temp folder; the launcher fails if that checkout changed)
- Also keep from the reviewer: the first reviewer's review folder and session transcripts

### Knowledge sources
- Read-only sources an investigation may use: the support inbox's MCP server, the error tracker's MCP server

### What the agent may do without asking
- Commit locally: yes
- Push the branch and open a PR: ask, except when running fix-ci
- Edit the PR description: yes
- Create tickets: after the user approves the list
- Move a ticket's state or comment on a ticket: yes
- Remove a merged branch's checkout: yes, after the safety checks in session-wrap-up
- Post review comments or reply to threads: after the user approves the exact text
- Resolve review threads: only threads the agent's own fix addressed; ask before resolving one it dismissed
```

The skills never need credentials in this section. If a tool needs a token, give the agent a way to read it that
doesn't print it (an MCP server, an environment variable, a secret manager's CLI), and say which in the section.
