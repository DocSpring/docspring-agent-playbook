<!--
Example nested AGENTS.md for background jobs. Copy it to <app/jobs/>AGENTS.md, replace every <placeholder>, and delete
every rule that doesn't apply to your stack. Examples marked "for example, in Sidekiq" have equivalents in Celery,
BullMQ, Oban and the rest. Delete this comment.
-->

# Background jobs

This directory holds the background jobs and the scheduled tasks that enqueue or run them. Jobs run on
<your job system>; scheduled tasks run from <cron, your platform's scheduler>, and the list of them with how each one's
failures show is in `<internal_docs/scheduled-tasks.md>`. Read `<config/AGENTS.md>` before a job reads a new setting.

## Retries and failures

- **Retries are deliberate; don't lower them to quiet the error tracker.** The job system's default retry count suits
  most jobs. Don't reduce it or add report-once filters to silence alerts. Report only what someone would act on, and
  rate-limit reports per error kind where one upstream outage fails every job at once, but log every occurrence.
- **Classify each failure as the customer's or ours.** An error the customer caused (a URL they gave that won't
  download, a document too large) sets the record's error state with a message they can act on, and is not retried or
  re-raised. Anything unexpected re-raises, so the job system retries and alerts. A new error type counts as ours until
  someone decides otherwise. Alerts on customer mistakes get ignored, and then real alerts get ignored too.
- **Set each job's retry deadline from how long its result stays useful.** For work a person is waiting on (a document,
  an export), retry an unexpected error a few times within <about a minute>, then set an error state that says the
  fault is on your side: a result that turns up hours later is no use to them. Work that only has to finish eventually
  (a reconciliation, a sync) can retry for longer, if it's safe to repeat.
- **Know what your queue does with a killed worker's jobs.** <If it can lose them (for example, Sidekiq's default fetch
  when a worker is killed outright):> anything that must finish needs a separate check that comes back to it. A queue
  that redelivers unacknowledged jobs needs no watchdog, but its jobs must be safe to run twice.
- **Never produce output with missing or wrong values.** When a job can't make a correct document or record, fail it
  with a message the customer can act on.

## Scheduled tasks

- **A scheduled task raises when it fails.** Its monitor reads the exit status, so an error it only logs or reports
  shows as a passing run. (enforced by: <your scheduled task monitor>)
- **Release each batch before loading the next.** Don't keep handled records, or what they loaded, until the loop ends:
  memory grows with every batch, and a task killed for memory raises nothing and reports nothing.
- **A task with a cap per run reports when it stops with work left.** Count what's left with a bounded query on an
  indexed column (see `<db/AGENTS.md>`).
- **Give a scheduled task only the environment variables it needs.** Least privilege: a task that doesn't need a secret
  doesn't get it. A missing variable often fails silently (a "configured?" check skips the work and the run still
  passes), so read the task's code for what it needs: the queue's URL to enqueue jobs, the mail provider's key to send
  mail. A task that only reads uses a read-only database user with a statement timeout.
  (enforced by: <a spec that pins each task's variable list>)
- **Add work to an existing scheduled task when it fits.** Prefer a runner that runs each step even when an earlier one
  failed, then fails the run.

## Queues and names

- **Add a new queue everywhere the workers read queues from.** <If queues are listed in two places, such as a constants
  file and the worker config:> a queue missing from the worker config is never polled, and nothing errors.
- **Rename a job class only with an alias for the old name.** Jobs already queued under the old name have nowhere to go.
  Other systems that enqueue a job by name (a bridge, another service) break on any rename.
- **Share logic between jobs the way this repo already does** (<a module, a base class>). Don't copy it between twin
  jobs.

## Job arguments in error reports

- **Error reports carry job arguments only when they can't hold customer data.** Send IDs, numbers, booleans and record
  references; summarize other strings as their type and size, and let the job ID find the log lines. When a string
  argument names what the job does (an event or class name), allow that one position or key explicitly.
  (enforced by: <your error tracker's before-send hook and its spec>)
- <If your N+1 detector can watch jobs:> **tests run each job inline and scan it for N+1 queries.**
  (enforced by: <your N+1 detector's job hook>)
