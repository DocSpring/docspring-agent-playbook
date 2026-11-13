<!--
Example nested AGENTS.md for a database directory. Copy it to <db/>AGENTS.md, replace every <placeholder>, and delete
every rule that doesn't apply to your stack. Examples marked "for example, in Rails" have equivalents in other
migration tools (Django, Alembic, Flyway, Prisma). Delete this comment.
-->

# Database and migrations

This directory holds the schema dump (`<db/structure.sql>`) and the migrations that change it. Migrations run
<when: from the deploy pipeline, before the new release goes live>. Read `<app/models/AGENTS.md>` before you change a
model's validations, and the production access rules in the root AGENTS.md before you run any query against
production. The schema dump is generated: change it by writing a migration and running it.

## Writing a migration

- **Every migration must be safe for the code that's deployed now.** Migrations run before the new release goes live,
  so the old code runs against the new schema for a while. Add a column before the code that writes it ships, and stop
  reading a column before anything renames or retires it.
- **Name the file with the real UTC time.** Use `date -u +%Y%m%d%H%M%S` (or your generator, which does the same),
  never a round `YYYYMMDD000000`. Two branches that pick the same version can't both merge, and most migration tools
  refuse to load duplicate versions, so every test run fails at boot once both land.
  (enforced by: <a CI check for duplicate migration versions>)
- **Use the safe pattern your migration linter suggests.** Wrap an operation in the linter's escape hatch (for example,
  in Rails, strong_migrations' `safety_assured`) only after you've checked it's safe, with a comment saying why. An
  escape without a reason teaches the next agent that escapes are routine.
  (enforced by: <your migration safety linter>)
- **Build indexes concurrently and idempotently.** In Postgres, build outside the migration's transaction with
  `CREATE INDEX CONCURRENTLY IF NOT EXISTS` (for example, in Rails, `disable_ddl_transaction!` and
  `add_index ..., algorithm: :concurrently, if_not_exists: true`). A plain build blocks writes to the table until it
  finishes. An interrupted concurrent build leaves an invalid index behind, and `IF NOT EXISTS` alone hides it: the
  retry, up or down, skips the build and reports success. Before retrying, drop the invalid index
  (`DROP INDEX CONCURRENTLY IF EXISTS ...`, or check `pg_index.indisvalid`).
- **Classify a new column wherever the code keeps lists of columns.** Exports, change tracking for webhooks, copy and
  lock lists: a column missing from one is silently left out of it.
  (enforced by: <a coverage spec that fails on an unclassified column>)

## Retiring a column

- **Retire a column; don't drop it.** Code from the previous release still reads it while a deploy runs, and
  <if customers run your software themselves:> installs can upgrade across many releases at once, so no release can
  assume the one that stopped using it has run. To retire a column:
  1. Stop using it and tell the ORM to ignore it (for example, in Rails, `self.ignored_columns += [...]`), with a
     comment saying what replaced it.
  2. Clear its data if it holds anything sensitive.
  3. Don't plan a follow-up migration that drops it.

  **When this doesn't fit:** you run every deployment yourself and declare a support window. Then use
  expand-and-contract: add the new column, move every reader and writer to it, and drop the old one once no supported
  release reads it.

## Column sizes

- **Give every string, text and JSON column a maximum size, with a CHECK constraint to match.** An unbounded column
  lets one runaway writer (a tool's whole output, base64 in a log) fill rows with megabytes. Put the cap in the model's
  validation and the constraint in the same migration. (enforced by: <a spec that lists columns without a cap>)
- **Make the constraint read the stored size, never the decoded value.** Postgres checks every constraint on every
  update of the row, so use `octet_length(col)` for text and `pg_column_size(col)` for JSON and jsonb, not
  `char_length(col)` or `col::text`, which decode or cast the whole value on each write.
- **Size a cap by who reads the value, not by the largest value already stored.** A stored value far over its cap is a
  bug in whatever wrote it: fix the writer (keep the first few errors and "and N more", send tool output to the error
  tracker) instead of raising the cap.
- **Writes that skip validations meet only the constraint.** Bulk updates and inserts (for example, in Rails,
  `update_all`, `update_columns`, `insert_all`) skip the model's cap, so cut or check the value first.

## Connections and production queries

- **Use `SET LOCAL` inside a transaction, never a session `SET`.** <If requests go through a connection pooler in
  transaction mode, such as PgBouncer's,> each transaction can get a different server connection, and a session `SET`
  stays on that connection and changes whatever runs there next, another customer's request included. Tests run inside
  a transaction, so `SET LOCAL` takes effect in a test even when the code under test opens none, while production's
  autocommit ignores it with only a warning. Assert that a transaction is open where the code relies on it.
- **Bound every production query, even a read-only one.** Select by an indexed column (IDs, one account's rows), work
  in batches and set `SET LOCAL statement_timeout`. A filter on an unindexed column reads the whole table whatever the
  batch size, and a read-only scan of a large table can take production down.
- **Page a batch job on the filtered column's index.** Batch iterators page by primary key (for example, in Rails,
  `find_each` and `in_batches`). On a filtered relation over a large table, Postgres can run each batch as a
  primary key scan that checks every row, so the last batch reads the rest of the table even when nothing is left to
  match. Count leftover work on an indexed column, with a `LIMIT` and an `ORDER BY` in that index's order: with a
  `LIMIT` and no `ORDER BY`, the planner can choose to scan the table.

## Before committing the schema dump

- **Check that the dump's diff holds only your migration's changes.** A local database that has run another branch's
  migrations dumps that branch's tables and columns too. If that happened, rebuild your database from main's schema
  and migrate again. (enforced by: <a CI job that rebuilds the dump from the migrations and diffs it>)
- **Change undeployed migrations in place.** While a branch's migrations haven't run anywhere outside development,
  edit them instead of adding fix-up migrations, then rebuild your local database. A chain of fix-ups on one branch is
  noise every later reader has to follow.
