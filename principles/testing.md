# Testing

### A regression test must fail before the fix.

Run the new test against the old code and see it fail for the reason you expect. When you select a test by line
number, check the output names the example you meant: some runners pick the nearest example at or above the line,
which may be the one above. To test an N+1 fix, create at least two records, each with its own associated record
(query cache hits don't count), then remove the preload and check the test fails.

*Scope: Any team.*

### Prove a failure isn't yours by rerunning it on main with the same seed.

Rerun the failing test with the same random seed on main, or on your branch's parent commit, before blaming or
dismissing your change. Treat CI as the authority when local and CI results disagree.

*Scope: Any team.*

### Make the suite enforce what production cares about.

Have tests fail on the problems that hurt in production: N+1 queries raise, an error-tracker report fails the test
unless it's tagged as expected, a browser console warning fails a browser test, every admin form renders without
unbounded queries. Before the tests run, CI deletes the files production images don't have, so code that reads one
fails in CI instead of after the deploy.

**How to enforce it:** For example, in Rails: Prosopite or Bullet set to raise, a stubbed error-tracker client that
fails the example, and a console-log check in the browser test teardown.

*Scope: Any team; the tools vary by stack.*

### Test behavior, not incidental wording or counts.

Don't assert incidental wording or catalog sizes: a test that counts the entries in a list breaks on every new entry
without catching a bug. Check the entries that matter (the first one, one that must be there, one that must not).
Assert exact text, absence or a count when that is the behavior being protected (personal data removed, a control
gone, a page size).

*Scope: Any team.*

### Stub responses; expect calls only when the call is the behavior.

Use a stub (`allow(...).to receive` in RSpec) to supply a response, and a message expectation (`expect(...).to receive`)
only when the call itself is what the test checks. Expectations on incidental calls make tests fail on harmless
refactors.

*Scope: Ruby; the same split exists in most mocking libraries.*

### Regenerate stored expectations locally and review every changed file.

Snapshots, expected images and expected documents are updated deliberately, with a flag (`UPDATE_SNAPSHOTS=1`), and
every changed file gets looked at before it's committed. CI never updates them, and nobody updates one just to make a
test pass.

*Scope: Any team.*

### Make tests safe to run in parallel processes.

Give each process its own database, cache database, storage bucket, ports, and log and result files. Write shared files
through a temp file and a rename, and take a lock around a read-modify-write. A server a test starts takes a port the
OS assigns. Never kill processes by name, only your own children, and never flush a shared cache server. Examples don't
depend on order, and every process defines the same examples (don't build the list from the time, random data or the
process number).

*Scope: Any team.*

### Know the traps that make tests pass on one machine and fail on another.

- A just-saved timestamp keeps nanoseconds on Linux while Postgres stores microseconds, so compare values read back
  from the database.
- Do date arithmetic and comparisons in one time zone: converting to UTC between them gives another day during the
  hours when the dates differ, so the test fails only then.
- CI may restrict `PATH`, so give full paths to commands a test runs.
- Linux browsers draw scrollbars that macOS hides, which shifts layouts and screenshots.
- A check that something is absent must not wait for it to appear: use the negative matcher.
- A test that starts a subprocess cleans up its children and their resources, even when it fails.
- Recorded HTTP interactions are replay-only on CI.

*Scope: Any team; the examples come from Rails and Capybara.*

### Restore configuration by deleting the override.

Reassigning the old value leaves an override in place, which changes later tests depending on their random order.
Remove the override (for example, `SomeController.config.delete(:setting)` in Rails) so the default applies again.

*Scope: Rails; the idea applies to any framework with layered configuration.*

### Keep one complete run of the whole suite.

Some documented check, on a schedule or on every merge, runs the whole suite with nothing cached, against the same OS
packages and file set that production has. Don't make it skip jobs to save time or money: it's what catches the
failures that caching and test selection hide. Choose its schedule from the risk and the capacity you have. Locally,
run the checks the change needs with the concurrency your repository supports, and write that down in your AGENTS.md.

*Scope: CI.*

### When a coverage or contract test fails, add the missing input.

A test that checks every route, column or file is covered fails because something new isn't. Add what's missing; don't
loosen the test.

*Scope: Any team.*

### Leave files alone during a long local test run.

Frameworks that reload code locally pick up files you edit mid-run, and later, unrelated tests fail in confusing ways.
Rerun with the same seed, without editing, before chasing those failures. A background run writing to a file buffers
its output, so judge its progress by CPU and memory.

*Scope: Rails and other frameworks that reload code in tests; the idea applies widely.*
