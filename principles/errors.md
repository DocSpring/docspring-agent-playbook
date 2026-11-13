# Errors

### Dev and test code crashes fast.

In tests, scripts, build tasks, generators and CI code: no null-safe navigation on things that must exist, no rescue
or catch without re-raising, no fallback values, and no carrying on after a failed step. Raise with context (the
values and paths involved). Where null really is a valid value, handle it where it's visible: a guard clause, an
explicit default, a `fetch` with a default. Silent failures in tooling become broken code in production, because the
script that should have stopped the build reports success instead.

**How to enforce it:** Scope lint rules to the tooling directories. For example, in Ruby, a custom cop that bans `&.`
under `scripts/` and `spec/`, plus `Lint/SuppressedException`; in TypeScript, `no-empty` for catch blocks and a
`no-restricted-syntax` rule on `ChainExpression`; in Python, Ruff's `S110` and `BLE001`. Give an intended rescue a
comment saying why.

*Scope: Any team.*

### Production degrades gracefully, except when the output would be wrong.

Production code reports errors to the error tracker and keeps the page working. The exception: never produce an
artifact (a document, an invoice, a report, an export) with missing or wrong values. Fail the job with a message the
customer can act on. A broken widget is worse than a logged error, and a wrong document that someone signs, files or
pays is worse than no document.

**How to enforce it:** Test each failure path of an output-producing job and assert that it ends in an error state
with a message, never in a finished artifact with a blank where a value should be.

*Scope: Any team.*

### An unknown value gets an error, not a fallback.

When a new parameter takes a fixed set of values, reject any other value with an error that lists the ones it takes.
Don't silently do what a default would: silent fallbacks become contracts customers depend on without knowing it.
Tightening validation on an existing parameter is a breaking change if supported clients send other values today:
check usage first ([compatibility](compatibility.md)).

**How to enforce it:** Validate requests against a schema with closed enums and closed objects
(`additionalProperties: false`), in every environment.

*Scope: Teams with public APIs, and any internal interface with more than one caller.*

### Tell customer errors from your own errors.

Classify each failure. An error the customer caused (a URL they gave that won't download, a file over the size limit,
an account that's blocked) gets a stored error state and a message they can act on, and is neither retried nor
alerted on. An unexpected error re-raises, so the job system retries it and the error tracker alerts. A new error type
counts as yours until you've decided otherwise. Alerts about customer mistakes get ignored, and then the real alerts
get ignored with them.

*Scope: Any team.*

### Report only what someone would act on, and don't quiet alerts by lowering retries.

Don't lower retry counts or add report-once filters to make the error tracker quieter: fix the cause, or reclassify a
customer error as one. Where an upstream outage would fail every request at once, rate-limit reports per error kind,
and still log every occurrence.

*Scope: Any team.*

### Scheduled tasks fail loudly.

A scheduled task's monitor reads its exit status, so a task must raise when it fails: an error it only logs shows as a
passing run. A task killed for running out of memory raises nothing at all, so a loop over many records must release
each batch (and anything it loaded with it) before loading the next. A task with a cap per run reports when it stops
with work left, counted with a bounded query.

**How to enforce it:** Monitor every scheduled task from its exit status and a maximum duration (a killed run reports
no completion). Test that the task raises when one of its steps fails.

*Scope: Any team.*

### Retry for as long as the result is still useful, and check what a killed worker loses.

Set each job's retry deadline from who is waiting for it. When a person is waiting on the screen, retry an unexpected
error a few times within about a minute, then set the error state: a result hours later is no use to them. An export,
a sync or a reconciliation that only has to finish eventually can retry for hours, if repeating it is safe.

Find out what your queue does with a job whose worker is killed outright (out of memory, SIGKILL, a node going away).
Some job systems lose it, with no retry and no dead-letter entry; others redeliver it. Where yours can lose work that
must finish, add a separate check that comes back to it. Rename a job class only with an alias for the old name, or
jobs already queued under it are stranded.

**How to enforce it:** For example, in Sidekiq, a shared concern for quick retries whose exhausted-retries hook sets
the error state, and an alias constant in an initializer for each renamed job. Test the kill case once in staging: kill
a worker mid-job and see what happens to the job.

*Scope: Any team with background jobs.*
