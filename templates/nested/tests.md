<!--
Example nested AGENTS.md for a test directory. Copy it to <spec/ or test/>AGENTS.md, replace every <placeholder>, and
delete every rule that doesn't apply to your stack. Examples marked "for example, in RSpec" have equivalents in pytest,
Jest, Go's testing package and the rest. Delete this comment.
-->

# Tests

This directory holds the test suite. Run a file with `<bin/test path/to/file>`. Browser tests have their own rules in
`<spec/features/AGENTS.md>`, and API tests in `<spec/api/AGENTS.md>`. Read `<spec/support/>` before writing a helper:
the one you need may exist.

## Running tests

- **Run tests locally at the concurrency this repo supports:** <one file at a time / up to N processes>. Past that,
  app boots starve each other and fail in ways that have nothing to do with your change. Send bigger runs to <CI>.
- **Make sure the local services are running first.** A test waiting on a socket to <the database, the cache, the object
  store> looks deadlocked. Check them with <a health check command>.
- **Leave app files alone during a long local run.** <If the test environment reloads code locally:> editing a file
  mid-run makes later, unrelated tests fail. Rerun with the same seed before you chase those failures.
- **Judge a background run by its CPU and memory.** Output written to a file is buffered and may appear only at the end.
- **Keep assertion failures readable.** A failing matcher that prints a whole large object can take minutes and look
  like a hang. <Name the matchers to avoid in your runner, if any.>

## Proving a failure

- **A regression test must fail first.** Run it without the fix and watch it fail for the reason you expect. Check that
  the output names the example you meant: selecting a test by line number can run the example above it.
- **To test an N+1 fix, create at least two records, each with its own associated record.** Query cache hits don't
  count. Then remove the preload and check the test fails.
- **Prove a failure isn't yours by rerunning it with the same seed on main.** Locally a flaky test fails the first time;
  CI may retry it. Treat CI as the authority.
- **Fix a flaky test you hit, in the branch you're on, when the fix is small and you can show the cause.** Even when
  your change didn't cause it: rerunning the job gets the branch through, but follow-up tickets for flakes don't get
  done. When the fix isn't small, report the flake with its evidence and ask.

## What the suite enforces

These checks fail tests on purpose. When one fails, fix the cause; don't loosen the check.

- **N+1 queries raise.** Fix them with eager loading or batched lookups. Pausing the detector is only for a polling loop
  that reloads one record, with a comment saying why.
  (enforced by: <your N+1 detector, for example, in Rails, Prosopite or Bullet>)
- **A report to the error tracker fails the test** unless the test is tagged as expecting one
  (<`:allow_error_reports`>). An unexpected report in a test is an unexpected report in production.
  (enforced by: <a hook that fails on any error tracker capture>)
- **Browser console warnings and errors fail a browser test.** Fix the cause; allow a known, harmless message by name.
  (enforced by: <a hook after each browser test>)
- **When a coverage or contract test fails, add the missing input.** It's telling you something new isn't covered.

## Writing tests

- **Test behavior, not incidental wording or catalog sizes.** A test that counts a catalog's entries breaks on every new
  entry without catching a bug. Assert exact text, absence or a count when that is the behavior being protected
  (personal data removed, a control gone, a page size).
- **Stub with `allow`; use `expect(...).to receive` only when the call is the behavior under test** (for example, in
  RSpec). An expectation on an incidental call breaks on every harmless refactor.
- **Regenerate stored expectations locally and review every changed file.** Snapshots, expected images, golden files:
  update them with `<UPDATE_EXPECTATIONS=1 bin/test path>`, deliberately, never to make a test pass. CI never updates
  them.
- **Restore config by deleting the override, not by reassigning the old value.** Reassigning leaves an override behind
  that changes later tests, depending on the random order.
- **Fixtures are committed, so they hold synthetic values only.** Never put customer or end-user data in a fixture or a
  recording, even in a quick experiment, and check any real-world file for personal data before adding it.

## Several test processes at once

- **Give each process its own state.** Its own database, cache database, buckets, ports, logs and result files: add the
  process number (<`TEST_ENV_NUMBER`>) to the name of any file each process writes.
- **Write shared files safely.** Write through a temp file and rename it into place, and lock around read-modify-write.
- **Let the OS pick ports.** A server a test starts listens on port 0, never a number searched for from a fixed start.
- **Never kill processes by name or flush a shared cache server.** Other processes, and other checkouts, use them.
- **Don't depend on order.** A slow file's examples can be split between processes, so no example relies on another
  having run, and every process must define the same examples (don't build the list from the time or random data).

## Cross-platform traps

- **Compare timestamps after reloading.** Linux clocks keep nanoseconds, but Postgres stores microseconds, so a freshly
  saved value never equals its reloaded copy on CI, while a Mac's microsecond clock passes locally.
- **Do date arithmetic and comparisons in one time zone.** Converting to UTC between them gives another day or month
  during the hours when UTC's date is ahead, so the test fails only in the evening.
- **Give full paths to commands a test runs.** <If CI restricts `PATH`:> a bare `bash` passes locally and fails on CI.
- **Clean up every process a test starts.** A child left running (by a debugger, a file watcher or a forked worker)
  keeps the run from exiting. Wait for or kill each child the test started, and only those.
