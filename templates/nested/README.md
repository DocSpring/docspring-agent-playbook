# Nested AGENTS.md examples

A root `AGENTS.md` holds the rules every change must follow. A nested `AGENTS.md` holds the rules for one directory:
the pitfalls of the code next to it, stated where the agent edits. These files are examples to start from.

| File                 | Copy to (for example)           | Covers                                                  |
| -------------------- | ------------------------------- | ------------------------------------------------------- |
| `database.md`        | `db/AGENTS.md`                  | Safe migrations, column sizes, poolers, bounded queries |
| `tests.md`           | `spec/AGENTS.md`                | Running tests, proving a failure, what the suite checks |
| `browser-tests.md`   | `spec/features/AGENTS.md`       | Ports, console errors, waits, recordings, screenshots   |
| `public-api.md`      | `app/controllers/api/AGENTS.md` | Schemas, additive changes, retries, generated clients   |
| `background-jobs.md` | `app/jobs/AGENTS.md`            | Retries, failure classes, scheduled tasks, arguments    |
| `ui.md`              | `app/views/AGENTS.md`           | Screen rules, an example design system, CSP, UI writing |
| `config.md`          | `config/AGENTS.md`              | Typed settings, aliases, self-hosted upgrades, commands |
| `ci.md`              | `.github/AGENTS.md`             | Generated config, test parity, shell traps, pinning     |
| `docs.md`            | `docs/AGENTS.md`                | What's public, writing rules, forked docs tooling       |
| `vendor.md`          | `vendor/AGENTS.md`              | Patch markers, paired versions, advisories, licenses    |

## Pick directories where agents keep making the same mistake

Don't write a nested file for every directory. Write one where you've corrected an agent more than once, or where a
mistake costs more than a failed test, such as migrations or the public API. When you correct an agent, put the rule
in the nearest nested file in the same PR, so the next session doesn't repeat the mistake.

## Cut them down

These files are examples to cut down. Delete every rule that doesn't apply to your stack, and every rule your agents
already follow without being told. A section whose heading starts with "If" (if you generate client libraries, if
customers run your software themselves) applies only to products like that. A rule marked "our choice" or followed by
"When this doesn't fit" is a policy we chose: keep it, change it or delete it deliberately. Shorter files work better:
agents give each line less attention as a file grows, and context files that restate what agents already know add cost
without helping. A good rule is one an agent would get wrong without it.

Replace each `<placeholder>`, including the `(enforced by: ...)` notes. Where a check enforces a rule, name it: the
agent then knows the failure is expected and fixes the cause instead of loosening the check. Where nothing enforces it,
delete the note.

## Keep them next to the code

- Put each file in the directory it governs, so it sits beside the code it describes and changes in the same PRs.
- Start each file with one paragraph saying what the directory is and which files to read first.
- Under headings, write short bullets: the rule in bold on one line, then the reason, then the check that enforces it.
- Link to a sibling file (`see spec/api/AGENTS.md`) rather than repeating its rules. Long procedures go in a runbook
  that the nested file links to.

## Add each one to the root file's index in the same PR

Not every coding agent loads nested AGENTS.md files on its own. The root `AGENTS.md` lists every nested file with the
topics it covers (see `templates/AGENTS.md`, "Read the nested AGENTS.md before you work in a directory"), and tells
agents to read the one for a directory before changing files in it. Add a line there in the same PR that adds a
nested file, or agents that don't load nested files never see it.

The topic list doubles as a search index: an agent can tell from the root alone which file holds the rule it needs.
