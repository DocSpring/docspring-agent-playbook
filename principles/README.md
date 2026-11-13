# Principles

Engineering rules we give coding agents, each with the reason behind it and, where a tool can check it, how to enforce
it. They're opinionated: each says what we do and why. Where a rule is a policy choice that won't suit every team, a
"**When this doesn't fit:**" line names the condition and what to do instead. Copy the ones that fit your team into
your AGENTS.md files ([guide](../guides/writing-agents-md.md)) and leave out the rest. Each rule ends with a scope line:
"Any team", a kind of team ("Teams with public APIs") or a stack ("Rails", "Postgres").

## The two rules everything else leans on

1. **Make the smallest complete change.** The change reaches every place it must (callers, sibling entry points, tests,
   docs) and fixes the problems you find in the code it touches, in the same PR, and adds nothing the task didn't ask
   for: no options, compatibility paths, backfills or follow-up phases.
2. **A review finding needs a trigger that can really happen.** Fix what a real user, file, integration or concurrent
   job could hit (adversarial input counts when it reaches a security, availability or data-integrity boundary), and
   anything that could put a wrong value in output or lose or expose data. List constructed inputs as "not fixing",
   with the reason, and problems in untouched code as found but not in this change, with a recommended next step.

Both are in full in [change-size.md](change-size.md).

## Pages

- [change-size.md](change-size.md): how far a change reaches and how little code it adds, review findings, fixing what
  you find, detectors, settled decisions.
- [errors.md](errors.md): crash fast in tooling, degrade gracefully in production, never produce wrong output, customer
  errors versus yours, alerts, scheduled tasks and jobs.
- [compatibility.md](compatibility.md): public APIs, SDKs, webhooks, embeds and self-hosted editions: what counts as
  breaking and how to change things without breaking anyone.
- [data-and-privacy.md](data-and-privacy.md): approved AI tools and data, tickets, fixtures, logs and public posts.
- [change-management.md](change-management.md): signed commits, never pushing to main, AI review records, worktrees,
  rebasing, pull request records, feature flags.
- [contracts-and-generated-files.md](contracts-and-generated-files.md): generated files, one source of truth, schemas
  that validate and document, limits and retries across systems.
- [untrusted-input.md](untrusted-input.md): regex safety, sandboxing parsers for uploaded files, decoded sizes, prompt
  injection, telemetry, TLS, CSP, subprocesses.
- [dependencies.md](dependencies.md): licenses, paid libraries, pinning, security advisories, shared CI caches, forks
  and upstream fixes.
- [testing.md](testing.md): failing first, parallel safety, what the suite enforces, cross-platform traps, the complete
  run.
- [writing.md](writing.md): UI copy, docs, claims, customer-facing drafts, announcements.
- [design.md](design.md): design principles for product UI and using a component library.

## Optional appendices

- [runtime-pitfalls.md](runtime-pitfalls.md): Rails and Postgres notes (temp files, connection poolers, migrations,
  serialized attributes, callbacks, typed config), plus a few for containers and CI. Not part of the core: copy only
  the entries that match your stack and versions.
