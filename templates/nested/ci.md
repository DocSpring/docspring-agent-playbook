<!--
Example nested AGENTS.md for CI configuration. Copy it to <.github/> (or <.circleci/>, <.gitlab/>) AGENTS.md, replace
every <placeholder>, and delete every rule that doesn't apply. Examples use GitHub Actions; other providers have the
same traps. Delete this comment.
-->

# CI

This directory holds the CI configuration. <If it's generated:> every workflow file is generated from
`<.github/src/>` by `<scripts/ci/generate_config>`. Read `<internal_docs/generated-files.md>` for what else CI
regenerates and checks. Check a run with `<gh run watch <run-id> --exit-status>`.

## The configuration

- **Never edit a generated CI file; change its source and regenerate.** A hand edit is overwritten by the next
  regeneration, and the pipeline runs something nobody reviewed in the source.
  (enforced by: <a pre-commit hook and a CI job that regenerate and fail on a difference>)
- **When a check that a job's inputs are covered fails, add the missing inputs.** Don't loosen the check: a job that
  skips when its inputs haven't changed must know every file it depends on.
- **Record what was tried and dropped.** <List CI changes that were tried and reverted, each with the reason and what
  would justify revisiting it.> Don't propose one again unless you can say what has changed.

## What CI may change

- **CI never commits to main.** Autofix jobs (formatter, generated files) commit only on branches; on main and in
  scheduled runs they check and fail, so main only ever holds commits someone reviewed.
- **Pull CI's own commits before rebasing a pushed branch.** `git pull --rebase` first: a `--force-with-lease` after a
  fetch won't stop a push that drops them.

## Keep the tests honest

- **Keep one run that checks everything, with nothing cached.** <Name it: the nightly run, the release pipeline.> Don't
  skip jobs in it or move jobs out of it to save time or money; make tests faster or split slow files instead.
- **Test against what production has.** Install the same OS package upgrades the production image installs, and delete
  the files production leaves out before the tests run, so a test can't pass by reading a file production lacks.
  (enforced by: <a step that removes non-production files before tests>)
- **Watch the whole pipeline, not the PR's check list.** `gh pr checks` lists only jobs that have started, so "all
  checks pass" can mean "everything so far". Check the required workflows and any approval states in the CI provider
  itself: a green summary doesn't prove a required approval happened.

## Writing CI shell

- **Give tools that size themselves from the CPU count an explicit worker count.** Inside a container, the CPU count is
  often the host's, so a default like "half the CPUs" can start more workers than the container's memory holds, and
  the step is killed with no useful error.
- **Write `git --no-pager`.** Some CI providers (for example CircleCI) run steps in a pseudo-terminal, so a git command
  with long output opens a pager and waits until the step times out. A local run without a terminal won't show it: test
  under a pseudo-terminal with `script -q /dev/null <command>` on macOS or `script -qec '<command>' /dev/null` on
  Linux.
- **Pass large values on stdin.** Linux limits a single argument to 128 KB, so pipe large JSON to `jq` instead of
  `--argjson`.
- **Shallow checkouts need care.** Use `git fetch --deepen=<n>` to reach a merge base (`--deepen` adds commits past
  the current shallow boundary; `--depth` counts from the remote tip). For history without file contents,
  `git fetch --unshallow --filter=blob:none origin <sha>`, naming the commit.

## Supply chain

- **Pin actions to a full commit SHA, with the version in a comment,** and the same commit everywhere one is used.
  Tags can move. (enforced by: <a spec or linter that rejects unpinned actions>)
- **Pin base images by tag and multi-arch index digest** (`image:1.2.3@sha256:...`), so a moved tag doesn't change the
  image under you. Read the skipped releases' notes when bumping a language version: defaults can change.
- **Shared caches are shared state.** Anything a job writes to a shared cache reaches later jobs and other branches,
  including third-party code from dependencies. Run dependency update branches where they can't write to caches other
  branches read, and never run a shared clone's hooks or use its config.

## Dependency update PRs

- **Never disable a dependency because its update fails CI.** Fix the code, or leave the PR open. Disabling it also
  stops its security updates. To hold back routine updates, require dashboard approval, which security updates skip
  (for example, in Renovate, `dependencyDashboardApproval: true`).
- **Dependency PRs merge as they are once CI is green.** Never design a flow where a person pulls, re-signs or
  re-pushes them; let the bot sign its own commits if your policy requires signatures.
- **Fix a security advisory in the PR that hits it.** Ignore one only when no fix exists or the fix needs a major
  upgrade, with a comment saying why and when to remove the ignore.
