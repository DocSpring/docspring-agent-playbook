# Writing AGENTS.md files

How to lay out AGENTS.md files so coding agents find the rules they need and follow them. The
[root template](../templates/AGENTS.md) and the [nested examples](../templates/nested/) put this into practice.

## Write only what an agent would get wrong without it

Agents already know your languages and frameworks. They don't know your rules: which tool does what here, which
behavior other systems depend on, which "improvement" you already tried and dropped. Write those, and leave out the
rest.

Length has a cost. A study of repository context files
([Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988)) found that they usually didn't raise task success while
adding over 20% to inference cost on average, and that repository overviews in particular didn't help. Agents did
follow the instructions in them, and the files helped most where they stated practices an agent wouldn't guess. So
skip the architecture tour and the list of technologies, and keep the non-obvious rules.

What goes in AGENTS.md is read on every task, and that's its strength. In
[Vercel's evals](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals), a compressed docs index in
AGENTS.md passed every task, while the same docs as a skill passed no more than the baseline with no docs at all,
because the agent didn't load the skill in most cases. Knowledge an agent needs whenever it touches some code belongs
in AGENTS.md, as rules or as an index pointing at files to read. Keep skills for workflows a user starts on purpose
([writing skills](writing-skills.md)).

## One root file, and nested files next to the code

- **The root AGENTS.md** holds what every change must follow, in a fixed order: one paragraph saying what the
  repository is and what lives where, the nested-file index, where knowledge lives, hard rules, the Git and pull
  request workflow, commands, standing approvals, code rules, and facts agents often get wrong. Keep each rule to a line
  or two and link to where its reason is written down.
- **A nested AGENTS.md** goes in each directory with pitfalls of its own: the database, the tests, the public API, the
  background jobs, the CI config, each client app. Start it with one paragraph saying what the directory is and which
  other files to read first, then short bullets under headings.
- **State each rule where agents edit, once.** A nested file links to its siblings ("see `spec/api/AGENTS.md`") instead
  of repeating them, and long procedures go in a docs directory such as `internal_docs/`, linked from the nested file.
  When a rule must appear in two places, copy its conditions and exceptions with it, or the copies drift apart.

## Index the nested files in the root

Not every agent loads nested AGENTS.md files on its own, so the root says to read them and lists them, one line each,
with the topics each file covers. The topic list doubles as a search index: an agent can tell from the root alone
which file holds the rule it needs.

```markdown
## Read the nested AGENTS.md before you work in a directory

Not every coding agent loads nested AGENTS.md files on its own. Before you change files under one of these directories,
read its AGENTS.md. Add any new nested AGENTS.md to this list in the same PR.

- `db/`: the schema file, safe migrations, retiring columns, adding a column
- `spec/`: running tests, what the suite enforces, fixtures
- `app/controllers/api/`: request schemas, validation, what an API change ships with
```

## Map where knowledge lives

List every store your team uses and what goes in each, so agents and people put a fact in one place and look for it
there:

| Store | What goes there |
| --- | --- |
| AGENTS.md files | Rules and pitfalls for the code next to them. A rule you learn goes in the nearest one, in the same pull request. |
| A docs directory (`internal_docs/`) | Longer references and runbooks whose commands change with the code, updated in the same pull requests |
| The wiki | Policies, the reasons behind decisions, product and customer context |
| Pull requests | The record of each change: description, review, QA results |
| The tracker | Open work |
| A search tool, if you have one | Searches all of the above at once: use it before grepping widely or asking |

Add one rule to the map: don't leave anything the team needs only in a temp directory or in a personal AI tool's
memory. A rule saved only in one person's agent memory never reaches their teammates' agents.

## List the generated files, including the misleading ones

Name the few generated files agents hit most in the root, each as "edit X, then run Y", and keep the full table (path,
regenerate command, what checks it) in a docs file. That file also needs the sections where a file isn't what it looks
like:

- **Diffs you didn't make:** files the dev server or test boot rewrites on its own. Agents otherwise revert them as
  someone else's edits, or commit them as their own.
- **Labeled as generated, but edited by hand:** stale "generated" headers, and templates that carry generator markers
  but are the sources the generator reads. Agents otherwise refuse to edit the one file they should.
- **Hand-edited, but checked by a tool:** allow-lists that a CI job prunes or validates.

The rule itself is in [contracts-and-generated-files.md](../principles/contracts-and-generated-files.md).

## Give standing approvals, and name the look-alikes that write

Agents in ask-before-running mode stop for every command. A standing approval list lets them run the safe ones without
asking. Scope it carefully in both directions: say it doesn't restrict agents with more permissive settings, and that
it doesn't cover changing Git state, deploying, destructive operations or writing to other services. List read-only
forms explicitly, and name the commands that look read-only but write.

```markdown
### Standing approval for agents in ask-first mode

You may run these without asking. Every other command keeps your normal permissions.

- Tests and lint: `bin/test [path]`, `bin/lint [path]`.
- Git, read-only forms only: `git status`, `git diff`, `git log`, `git show`, `git blame`, `git branch --list`.
  This does not approve `git fetch`, `git pull`, `git checkout`, `git stash`, `git reset`, `git push` or
  `git branch -d`.
- CI status, read-only: `bin/ci status`. This does not approve `bin/ci rerun-failed` (it reruns jobs) or
  `bin/ci update-screenshots` (it overwrites fixtures).
```

## Correct the facts agents often get wrong

Keep a short section of domain facts that contradict what an agent would assume from names or from typical apps. Each
fact names the code that proves it and the file with details.

```markdown
## Facts agents often get wrong

- Invoices are immutable once sent. A correction is a credit note that references the original
  (`app/models/credit_note.rb`), never an edit to the invoice.
- Amounts are integers in the currency's smallest unit (`amount_cents`), never floats (`app/models/money.rb`).
- Soft-deleted rows are hidden by a default scope. Admin and export queries call `unscoped` on purpose.
```

## Write settled decisions down

Record "tried and dropped", "settled, don't propose X" and "intended behavior, not a bug" next to the code, with the
reason and what would justify revisiting it. Every new session otherwise rediscovers the same improvement and reports
the same bug. An agent doesn't propose a rejected approach again unless it can say what has changed. See
[change-size.md](../principles/change-size.md).

## How to write a rule

- **Shape:** the rule first, in bold, as one line; then the reason, as the failure mode it prevents; then what enforces
  it. Imperative and present tense.
- **Standing instructions, not history.** "Render a test document after every change", never "In March we added
  test renders". Leave out dates and version numbers that will change.
- **Name the enforcement.** Say which lint rule, test or CI check catches the rule. An agent that knows a failure is
  expected fixes the code; one that doesn't tends to loosen the check. Say it outright where it matters: "update the
  snapshot deliberately, never to make the test pass", "add the missing input rather than loosening the spec".
- **Give one default way, not a menu.** "Use `bin/test`" works; "you can use `bin/test`, `rake spec` or `rspec`" gets
  you all three.
- **Put the gotchas in the main file.** Agents often skip reference files. Keep the most important rules first, since
  some clients keep only the start of a long file after compacting a conversation, and say exactly when to read each
  linked file ("Read `docs/deploy.md` before running a deploy command").

```markdown
- **Never run an unbounded regex repeat over input a user sizes.** Backtracking engines keep state for each repeat, so
  one pattern can use gigabytes of memory on a value of a few megabytes. Cut the string first, bound the repeat or
  scan with a loop. `spec/lib/regex_memory_spec.rb` measures a child process's memory.
```

## Keep one copy of the rules

Keep your rules in AGENTS.md, and make every agent you use load that one file. Claude Code (v2.1.277 and later) reads
AGENTS.md files itself, but if a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` exists in the working directory
or above it, it reads only the CLAUDE.md files in those directories, and nothing tells you. A subdirectory's AGENTS.md
still loads when that subdirectory has no CLAUDE.md of its own. Your `~/.claude/CLAUDE.md`, an organization's managed
CLAUDE.md and `.claude/rules/` files load alongside AGENTS.md either way.

If you need a CLAUDE.md (an older Claude Code, a teammate's personal file), make it a symlink to AGENTS.md or a
one-line `@AGENTS.md` import, never a second copy of the rules. Someone who wants personal instructions on top can set
Project instructions (in `/config`) to `claude-md-and-agents-md`, which reads both. After changing the setup, check
`/memory` shows the rules loaded. Watch for vendored code and submodules that ship their own CLAUDE.md, and start
sessions from the repository root. The details are in
[Claude Code's memory docs](https://code.claude.com/docs/en/memory).

## Tell agents to quote heredoc delimiters

Add this to the commands section: quote heredoc delimiters (`<<'EOF'`), use one heredoc per command, and write pull
request bodies to a file passed with `--body-file`. An unquoted heredoc runs every backticked command in its text, and
notes about your codebase are full of backticked commands.

## Keep the files current

- When an agent learns a durable rule (a footgun, a convention someone corrected), it adds it to the nearest AGENTS.md
  in the same pull request.
- When code changes what a rule describes, the rule changes in the same pull request.
- Add each new nested file to the root index in the pull request that creates it.
- Reread a file now and then and delete what no longer earns its place. A rule nobody would get wrong is cost without
  benefit.
