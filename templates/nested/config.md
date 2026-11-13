<!--
Example nested AGENTS.md for the config directory. Copy it to <config/>AGENTS.md, replace every <placeholder>, and
delete every rule that doesn't apply to your stack. The section that starts with "If" matters only if customers run your
software themselves. Delete this comment.
-->

# Configuration

This directory holds the app's settings, initializers and routes. Every environment variable the shipped code reads is
declared in `<config/environment_variables.yml>` and read through `<AppConfig>`. Read `<app/jobs/AGENTS.md>` before
giving a scheduled task a new variable, and `<internal_docs/generated-files.md>` before editing anything this
directory generates.

## Settings

- **Declare every environment variable in one typed file and read it through the config object, never raw `ENV`.**
  Each entry has its type, default, description, audience (internal, self-hosted, both) and old names. Scattered `ENV`
  reads drift: two parsers for one boolean, a default in one place and another elsewhere.
  (enforced by: <a lint rule banning raw `ENV` reads, and a spec that every variable read is declared>)
- **Classify each setting by what a bad value would do.** Every problem (an unknown variable with your prefix, a value
  that doesn't parse) raises in development, test and CI. In production, a setting that guards security or data
  integrity (an encryption key, a retention period, an allowed host) fails closed: it stops boot or the feature that
  uses it. An optional capability turns itself off. Only a setting with a documented, safe fallback warns and carries
  on, and the warning reaches someone.
- **Declare variables that only CI or another tool reads too,** marked as not read by the app. <If boot fails on an
  unknown prefixed variable:> an undeclared one that CI gives every job stops the app booting in those jobs.
- **Tests change settings through a helper, never by stubbing `ENV` or constants.**
  (enforced by: <a lint rule in the test directory>)
- **Start new variables with `<APP_>`,** unless other software reads them itself.
- **Read secrets only when the task needs one.** Never print a config, env or release object in full: select the fields
  you need.

## Renames and aliases

- **Renaming a variable adds the old name as an alias.** Deployed environments keep their old names through upgrades.
  Give a new setting a default that keeps an existing install working. The same holds for anything an install's config
  calls: a script, a command or a task keeps its old name as an alias when you rename it.
- **Never remove an alias** <if customers run your software themselves: they can upgrade from any old release>.
  **When this doesn't fit:** you run every deployment yourself. Then remove an alias once every environment you run has
  switched to the new name.

## If customers run your software themselves

- **An upgrade works with the install's existing config.** Customers upgrade by running a new image with their old
  environment and manifests, so use shims and fallbacks, never manual upgrade steps. Where a newer setting is missing,
  fall back to what an old install has, or turn the feature that needs it off instead of failing at boot.
- **Imports read archives that old releases wrote.** Handle missing keys; never rename or drop a key in the format.
- **No development or demo seed data in a customer's install.**

## External commands

- **Run every external command through one wrapper.** Never raw backticks or a bare process call. The wrapper sets:
  - a timeout, and kills the command's whole process group at the deadline;
  - a memory limit (except for JVM tools, which reserve more address space than they use and fail under one);
  - an empty environment plus an allow-list (`HOME`, `PATH`, `LANG`), so the app's secrets aren't handed to the
    command;
  - one thread for tools that would otherwise size themselves to every CPU.
  A variable set in the container image doesn't reach a command, so pass what it needs explicitly. These are resource
  and hygiene controls, not a security boundary: a command running as the app's user can still read its files and
  reach its network. Tools that parse untrusted files run in a sandbox or a separate service with no access to your
  secrets. (enforced by: <a lint rule banning other ways to start a process>)
- **Pass secrets to a command in files, never as arguments.** Arguments show up in process lists and error reports.
- **Never turn off TLS verification.** Check library defaults that skip certificate checks, and point libraries that
  ship their own CA bundle at the system's, so a CA the OS stops trusting stops being trusted everywhere.

## Routes

- **Redact secret-bearing path segments and query parameters in every place that records URLs** (request logs, error
  reports, analytics, proxies), and test the emitted records. Naming a parameter `token` redacts nothing on its own;
  consistent names only make the filters easier to write. (enforced by: <a spec that checks the logged URL for each
  such route>)
- **Check which host a route is served on before assuming a URL works there.** <If routes are split by host.>
