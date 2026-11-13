<!--
Example nested AGENTS.md for browser tests. Copy it to <spec/features/>AGENTS.md, replace every <placeholder>, and
delete every rule that doesn't apply to your stack. Examples marked "for example, in Capybara" have equivalents in
Playwright, Cypress and Selenium. Delete this comment.
-->

# Browser tests

This directory holds the tests that drive a real browser against the app. Read `<spec/AGENTS.md>` first: its rules
(local concurrency, fail first, same seed on main, synthetic fixtures) apply here too. Run one file with
`<HEADLESS=1 bin/test spec/features/path>`.

## Running them

- **Hide the browser unless someone is watching with you.** Set `<HEADLESS=1>`. A visible window steals focus from the
  user and can change timing.
- **Read the app's port from the test config, never a literal port.** It differs per checkout and per parallel process
  (for example, in Capybara, `Capybara.server_port`).
- **Make sure the tests run the code you just changed.** <Name any gap in this repository's test build, such as
  changes the automatic rebuild doesn't watch, the command that rebuilds (`<pnpm build:test>`), and how to check the
  bundle is current.>
- **Run a failing test on the parent commit before blaming your change.** Browser tests are the flakiest part of the
  suite. Treat CI as the authority.

## Console, waits and absence

- **Any console warning or error fails the test.** Fix the cause. For a known, harmless message, add it to
  `<the ignore list>` or the example's own allow-list, by its text. (enforced by: <a hook after each browser test>)
- **A check for something absent must not wait for it.** A query that waits for at least one match takes the whole
  wait time when there's none (for example, in Capybara, `expect(all('.row')).to be_empty`). Use the negative matcher
  (`have_no_css`, `not.toBeVisible`), which returns as soon as nothing matches.
- **Wait for what the page shows, never for a fixed time.** <List any synchronization quirks your app has, with the
  condition to wait for.>

## Environment differences

- **Keep the local browser at the version CI runs.** Pin both in one place and fail a test when they drift.
  (enforced by: <a spec comparing the local and CI browser versions>)
- **Linux browsers draw scrollbars that Mac browsers hide.** A layout check can pass locally and fail on CI. Measure
  the window as a fixed element sees it, and run layout tests in CI's image before pushing.
- **Start each example at the default window size.** A browser reused at another size fails the next example's
  screenshots. Reset the size before each example rather than restoring it after.

## The database

- **Know which connection the app server uses.** <If browser tests run in a transaction that rolls back:> the app
  server shares the test's connection. A test where another process reads the database, or that tests locks between
  connections, needs truncation instead (<`database_cleaner: :truncation`>), or it deadlocks until a timeout.

## Recorded HTTP

- **Outside requests are replayed from recordings, and CI only replays.** A new request without a recording fails on
  CI. Record it locally and commit the recording, after checking it holds no tokens or personal data.
  (enforced by: <your HTTP recorder in replay-only mode on CI>)

## Screenshots

- **Look at every changed screenshot before committing it.** Regenerate expected images with
  `<UPDATE_EXPECTED_IMAGES=1>` locally. A screenshot updated without looking makes the test pass and the bug permanent.
- **Screenshots that fail on CI are saved as artifacts.** Download and compare them before deciding the expected image
  is wrong.
