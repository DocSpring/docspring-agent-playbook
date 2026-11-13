# Optional appendix: Rails and Postgres notes

Traps we've written down for our own stack, mostly Ruby, Rails and Postgres, plus a few for containers and CI. They
aren't part of the core principles: copy only the entries that match your framework, its version and how you deploy,
and check each one against your versions before you rely on it. The general rules behind them (bound the work, own the
resources you clean up, keep migrations safe for deployed code) are in the core pages.

### Keep a reference to a temp file until whatever reads it is done.

Garbage collection deletes an unreferenced `Tempfile`'s file, so passing only its path to a subprocess, a later method
or a file download fails now and then with "No such file or directory". Hold the object until the reader finishes, then
close and unlink it; for a download, send the file's contents. Python's `NamedTemporaryFile` deletes on close and on
collection in the same way.

*Scope: Ruby, Python.*

### Smoke tests delete only what they created.

Give a smoke test its own disposable directories and clean up only paths it made. A test that runs both in image builds
and on developers' machines sees whatever the environment points it at (a config or cache directory from `XDG_*` or
`HOME`), and deleting that removes a developer's real files.

*Scope: Any team.*

### Use `SET LOCAL` behind a transaction-mode connection pooler.

Behind a transaction-pooling proxy (PgBouncer in transaction mode), each transaction can get a different server
connection, and a session-level `SET` stays on its server connection and changes whatever runs there next, even another
customer's request. Use `SET LOCAL` inside a transaction. Specs usually run inside a transaction, so `SET LOCAL` takes
effect there even where production's autocommit ignores it with only a warning: assert that a transaction is open.

*Scope: Postgres.*

### Bound every production query, even a read-only one.

Select by an indexed column, work in batches by primary key, and set a statement timeout (`SET LOCAL
statement_timeout`). A filter on unindexed columns reads the whole table whatever the batch size, and batch iterators
on a filtered relation can scan to the end of the table: page on the filtered column's index. A read-only scan of a
large table can take a production database down as surely as a write.

*Scope: Postgres; the idea applies to any database.*

### Every migration is safe for the code that's already deployed.

Migrations usually run before the new release goes live, so the old code runs against the new schema for a while. Build
indexes concurrently and idempotently, but drop an invalid index before retrying an interrupted build: an interrupted
concurrent build leaves an invalid index behind, and `IF NOT EXISTS` on the retry skips it and reports success. We never
drop a column: stop using it, mark it ignored in the model with a comment, scrub sensitive data, and don't plan the
drop, because customers' self-hosted installs can skip many releases. Give every string, text and JSON column a maximum
size, with a CHECK constraint that reads the stored length (`octet_length` for text, `pg_column_size` for JSON) so it
stays cheap on every update. Name migration files with the real UTC time, as the generator does: two branches that both
pick a round timestamp can't both be merged.

**How to enforce it:** A migration-safety linter (for example, strong_migrations in Rails), with its "safety assured"
escape allowed only with a comment saying why, and a coverage check that fails a new column without a size limit.

**When this doesn't fit:** you run every deployment yourself and declare a support window. Then use expand and
contract: add the new column, move readers and writers to it, and drop the old one once no supported version reads it.

*Scope: Rails and Postgres; the ideas apply to any migration tool.*

### Assign a new object to a serialized attribute instead of mutating it.

An in-place change to a serialized or encrypted attribute (`record.data["key"] = value`) may not be detected and
saved, and reassigning the same mutated object can save without marking it changed, so audits and callbacks miss it.
Assign a new object (`self.data = data.merge("key" => value)`). Old rows can deserialize to legacy types: read through
a normalizing accessor and test with a legacy row.

*Scope: Rails.*

### Decide what changed in `after_save`, and publish after the commit.

In an after-commit hook, `saved_changes` describes only the last save, so it misses a change that another save
followed (some libraries save again in their own after-commit hooks). Decide what changed in `after_save` and publish
the event after the commit. Writes that skip callbacks
(`update_columns`, bulk updates) must publish explicitly.

*Scope: Rails.*

### Turn audit logging off per block on the current thread, never globally.

Audit libraries often offer a global switch, and flipping it turns off auditing for every thread in the process, so
other requests go unaudited while it's off.

**How to enforce it:** A custom lint rule that bans the global switch.

*Scope: Rails, and any audit library with a global toggle.*

### Don't let unrelated validations block a verified payment event.

Older records may hold data that fails validations added since (a profile field that's now required, say). That must
never stop a card change or a payment webhook from being recorded. Give billing updates their own path that verifies the
event's signature, checks it belongs to the account, and validates the financial fields and the state transition, and
skips only the unrelated validations it was built to skip. Have the webhook handler re-raise unexpected errors so the
payment provider retries.

*Scope: Teams that take payments.*

### Read settings through one typed config, never raw environment variables.

Declare every environment variable the shipped code reads, with its type, default, description, audience and old
names, and read it through the config object. Problems raise in development, test and CI. In production, classify each
setting by what a bad value would do: refuse to boot on anything that affects security or data integrity (keys,
authorization, retention), turn off the one optional feature whose setting is bad, and only warn and carry on where
the fallback is safe and the warning reaches someone. Specs change settings through a helper, never by stubbing `ENV`.
Scheduled tasks and one-off commands get only the variables they name, and a missing one usually fails silently, so
read the task's code for what it needs.

**How to enforce it:** A lint rule that bans direct `ENV` reads outside the config, and a spec that every variable the
code reads is declared.

*Scope: Any team.*

### Give tools an explicit worker count in containers.

Containers report the host's CPU count, so a tool that sizes itself from it starts too many workers and gets killed
for memory, often with no exit code or message. Set the worker count explicitly in CI and in production images.

*Scope: CI, containers.*

### Pin images and CI actions to immutable references.

Pin base images by tag and multi-architecture index digest. Pin CI actions to a commit, with the version in a comment,
and use the same commit everywhere. When you bump a language runtime's version, read the notes of every release you
skip: defaults such as garbage collection or memory settings can change.

**How to enforce it:** A test or lint step that fails an unpinned action or image, or one action pinned to two
different commits.

*Scope: Any team.*

### Test both sides of every exception to a protective callback.

When a change skips or narrows a callback that protects something (authentication, authorization, CSRF, rate limits,
quotas), test the requests that should skip it and the requests that shouldn't, for every action and entry point the
callback covers, including requests that meet only part of the exception's condition. Callback options combine in
ways that don't read the way they look, and a test of the allowed path alone passes either way.

*Scope: Rails, and any framework with before-action filters or middleware.*

### Admin forms never load whole tables.

Default admin forms render each association as a select of every row. On a big table that loads the table on every
render and can exhaust the web processes' memory. Use ID inputs or bounded collections.

**How to enforce it:** A spec that renders every admin form against a database with enough rows and fails on an
unbounded query.

*Scope: Rails admin frameworks (ActiveAdmin, Administrate) and similar tools elsewhere.*
