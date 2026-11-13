# DocSpring Agent Playbook

Engineering principles, AGENTS.md templates and workflow skills that other teams copy and adapt. Everything here is
public and meant to work for any stack, tracker and CI provider.

## Writing rules

- Plain, direct English, US spelling. Address the agent as "you" and the person as "the user". Imperative, present
  tense, standing instructions rather than history.
- Each rule states the rule first and the reason in the same bullet. Explain the failure mode (what goes wrong and
  why); don't describe incidents from any company's history, and no company names, ticket numbers, dates, hostnames
  or figures from a business.
- Rules are opinionated. Where a rule is a policy choice that won't fit every team, add one line starting "**When this
  doesn't fit:**" with the condition and what to do instead. Rules for one kind of product go under a heading that
  starts with "If" ("If you generate client libraries").
- No em dashes or en dashes. Avoid the usual AI tells: "not X but Y" reveals, forced lists of three, one-line closers
  that restate the paragraph, marketing adjectives.
- Name a tool, tracker or CI provider only as a labeled example ("for example, GitHub Actions"). Skills read those
  details from the adopting repository's `## Agent workflow settings` section (`CONFIGURE.md`).
- Rules too specific to one product don't belong here. Neither does anything that describes how to get past a
  particular system's defences.

## Skills

- Each skill is `skills/<name>/SKILL.md` with `name` (matching the folder), `description` (under 1,024 characters:
  what it does, "Use when ..." in the user's words, "Not for ..." when there's a likely mix-up) and
  `metadata.version`.
- Keep SKILL.md short with the most important rules first. Put long checklists in `references/` and say exactly when
  to read each one.
- Subagent definitions live in the skill's `agents/` folder.
- Bump the skill's `metadata.version` and the plugin `version` in `.claude-plugin/` and `.codex-plugin/` in the PR that
  changes a skill, and add a line to `CHANGELOG.md`.

## Files

- Keep one copy of the rules, in this file. If a tool needs a `CLAUDE.md`, make it a symlink to AGENTS.md or a
  one-line `@AGENTS.md` import, never a second copy
  ([guides/writing-agents-md.md](guides/writing-agents-md.md#keep-one-copy-of-the-rules)).
- `templates/` and `principles/` must stay consistent: a rule changed in one changes in the other in the same PR. The
  reason for a rule lives once, in `principles/`; templates state the rule and link there.
