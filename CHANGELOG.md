# Changelog

## 1.0.2

- The skills read "What the agent may do without asking" from the base branch's AGENTS.md, not the checkout, so a
  branch or someone else's pull request can't change what it's allowed to do; pr-review reports such a change as a
  finding.
- ui-qa-pass pins Playwright to an exact version and asks before installing it.

## 1.0.1

- story-kickoff: its description lost everything after "work on #42", which YAML read as a comment; the example is now "work on issue 42".
- The Claude plugin manifest sets a display name and a support link.

## 1.0.0

- First release: principles (with optional sections for particular product models, and an optional Rails and Postgres
  appendix), a root AGENTS.md template, ten nested templates, a settings block for the skills (`CONFIGURE.md`), guides
  for writing AGENTS.md files and skills, and six skills (story-kickoff, pr-review, second-opinion-review, fix-ci,
  ui-qa-pass, session-wrap-up) with worked adapters for GitHub, GitHub Actions and the Codex CLI.
