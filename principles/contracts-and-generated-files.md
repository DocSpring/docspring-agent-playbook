# Contracts and generated files

### Never hand-edit a generated file: change its source and regenerate.

Keep a table of every generated file with the command that regenerates it and the check that catches drift, and name
the few that agents hit most in the root AGENTS.md as "edit X, then run Y". Add the sections that say where a file
isn't what it looks like:

- **Diffs you didn't make:** files a dev server or test boot rewrites on its own. Treat their diffs as generated output,
  not someone else's edits to keep or revert.
- **Labeled as generated, but edited by hand:** stale "generated" headers, and templates that carry generator markers
  but are the sources the generator reads.
- **Hand-edited, but checked by a tool:** allow-lists that a CI job prunes or validates.

Say which generated files are committed and why (schema dumps, OpenAPI documents and type stubs usually are, so
reviewers see the contract change). We never commit compiled bundles of our own code: CI builds them, so a bundle can't
drift from its source or hide a change from review.

**How to enforce it:** CI regenerates each committed generated file and fails on a difference.

**When this doesn't fit:** the way you distribute requires a committed build (a package installed from Git, a GitHub
Action). Then commit it from CI or check in CI that it matches a fresh build.

*Scope: Any team.*

### Regenerate from source with the supported command, and check the whole output.

Each generated contract (an OpenAPI document, say) has one supported command that produces it. Use that command, and
don't stitch together the output of partial or parallel runs unless the generator itself assembles them in a fixed
order: a partial run can silently drop whatever it didn't cover. Stub everything random or local (IDs, tokens,
timestamps, hostnames) so every run produces the same bytes.

**How to enforce it:** CI runs the supported command and fails when the committed file differs.

*Scope: Any team.*

### Keep one source of truth for each rule, and parity fixtures where there are two.

When the same rule runs in two places (browser and server, two renderers, two languages), keep one implementation where
you can, for example by compiling the browser code for the server's embedded JavaScript engine. Otherwise keep one
shared fixture file of cases that both test suites run, and write "change both together" next to each pair. Change a
rule's meaning in the engine, never in an adapter that feeds it. Two hand-written copies drift, and the drift shows up
as a customer seeing one result in a preview and another in the output.

**How to enforce it:** Both test suites load the same JSON fixture of cases and fail on any case they disagree on.

*Scope: Any team.*

### Validate requests with the schema that generates the docs.

The request schemas that validate live traffic are the same files the API document generator reads, so the docs can't
describe something the API doesn't do, and a schema change is visibly a docs and SDK change. If your SDKs validate
requests before sending them, build that validation from the server's schemas too, so a client never rejects what the
server accepts.

**How to enforce it:** A test that the generator reads the validating schemas, and, where SDKs validate, a conformance
suite that sends the same requests to the server and to each SDK and compares which ones are accepted.

*Scope: Teams with public APIs.*

### Declare each setting once and derive every list from it.

One declaration file lists each setting with its type and who sees it, and models, settings pages, the editor and the
API schema read their lists from it. A column or setting added in one place and forgotten in the others is a bug that
ships silently.

**How to enforce it:** A coverage check that fails when a new column or setting isn't classified in the declaration.

*Scope: Any team.*

### Write limits that have to agree down in one place.

When timeouts and sizes chain across systems (a request timeout, a job timeout, a queue visibility timeout, a watchdog,
a temp disk size), list them together with the inequalities that must hold, and say that changing one means checking the
others. Each limit looks reasonable alone; the bug is in the relation between them.

*Scope: Any team.*

### Give retries to one side.

When two systems could both retry the same work, pick one owner. Give each attempt's output its own storage key, so a
late attempt can't overwrite a newer result, and make the change safe to roll back on either side.
Two retrying sides multiply attempts and race each other's results.

*Scope: Any team with distributed jobs.*
