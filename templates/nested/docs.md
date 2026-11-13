<!--
Example nested AGENTS.md for a public docs site. Copy it to <docs/>AGENTS.md, replace every <placeholder>, and delete
every rule that doesn't apply. Keep this file outside the directory that gets published. Delete this comment.
-->

# Docs site

This directory holds the public docs at `<example.com/docs>`, built with <your docs framework>. The pages are in
`<src/content/docs/>`; how the site builds is in `<README.md>`. Build and check it with `<task docs:build>` and
`<task docs:lint>`.

## Everything in the content directory is public

- **Never put internal notes, drafts or agent files under `<src/content/docs/>`.** Everything there is published,
  including an AGENTS.md. <If CI mirrors the directory to a public repo:> the mirror is one-way: never push to it or
  merge pull requests there; apply outside contributions here.
- **Docs change in the same PR as the code they describe,** and deploy with it. A docs PR that trails the code leaves
  customers reading about behavior that doesn't exist yet, or no longer does.
- **Check every claim against the code before writing it.** Read the code or the API document before describing an
  endpoint, parameter or option.

## Writing rules

- **Document only what exists.** No planned features, no "coming soon".
- **Describe how things work now.** Guides aren't history: don't explain how older versions behaved. Each version's
  changelog entry still says what changed.
- **Be direct and factual.** No "Benefits" sections or marketing language.
- **Never hard-code a version.** Read versions from <a helper or build-time placeholder>, so pages don't go stale on
  each release. (enforced by: <a lint rule for version-like strings>)
- **Keep related options in the feature's main guide** instead of adding a page per small option.
- **Keep API endpoint descriptions short and put the details in guides.** The reference links to the guide.
- **Use the API's names exactly** (`template_id`, not "the template's identifier"), and one name for each thing across
  the docs, the UI and the API.
- **Link to the API reference in one format** (for example `/api#operation/<operationId>`), so the build can check
  every such link. (enforced by: <a build step that validates those links against the API document>)

## When the docs and the code disagree

- **Find the intended contract first.** Fix wrong docs straight away (a wrong default, argument order or field name).
  Build the missing behavior only when the task's scope covers it; otherwise describe the current limitation
  accurately and record the gap.

## Forked or vendored docs tooling

- **Customize a fork through its extension points.** Use configuration, plugins, schema extensions and theme files
  rather than editing the upstream's components, so each rebase stays small. When the upstream itself needs a change,
  make the small, general change a maintainer would take (an option, a slot, a hook) and offer it upstream.
- **Tag the fork's old tip before force-pushing a rebase.** Older submodule pointers on main must stay reachable.
- **Start agent sessions from the repo root.** <If a vendored submodule ships its own CLAUDE.md:> Claude Code started
  inside it reads only CLAUDE.md files there, not the AGENTS.md files in that directory and above it.
- **When a tooling upgrade changes generated anchors, update every link to them in the same PR.**
