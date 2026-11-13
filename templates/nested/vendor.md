<!--
Example nested AGENTS.md for vendored third-party code. Copy it to <vendor/>AGENTS.md, replace every <placeholder>,
and delete every rule that doesn't apply. List each vendored library under its own heading with its version, source,
license, and how to update it. Delete this comment.
-->

# Vendored code

This directory holds third-party libraries and assets copied into the repo: <browser bundles, CSS frameworks, forked
packages>. Each one below says where it came from, its version and license, and how to update it. Read the root
AGENTS.md's license rules before adding anything here.

## Rules for every vendored file

- **Never edit a vendored file in place without a greppable patch marker.** Mark every local change with a comment
  (`<OURS:>` or your company's name) saying what changed and why. An unmarked edit is lost on the next update, and
  nobody knows to re-apply it. (enforced by: <a CI check that vendored files match their recorded checksums, or carry
  a marker>)
- **Re-apply every marked patch when you update, and check whether upstream has fixed it.** Search for the marker
  before replacing the files. Drop a patch upstream has absorbed.
- **Record the version, source and checksum.** Download from the official release, verify its checksum against the
  publisher's, and keep the license header. Strip source map comments that point at files you don't ship.
- **Vendor a package only when you must change it** (to lift a version pin that blocks a security fix, or to carry a
  patch). Otherwise depend on it through the package manager.
- **Fix bugs you find in a fork, and offer each fix upstream.** Long-lived patches are fine; carry them in the fork and
  make each one the small, general change a maintainer would take.
- **Linters and type checkers skip this directory.** Don't reformat vendored files: it hides the real diff on update.

## Paired versions

- **Keep a vendored copy and the bundled package of the same library at one version.** Two copies of a library that
  share global state on the page fight over it, and the one that loads last can replace the other's state with
  something incompatible. Update both in one PR, and check they work together.
  (enforced by: <a spec comparing the two versions>)
- **A build step that strips unused CSS can't see classes built at runtime** (`"alert-" + tone`) or added by another
  library's script. Add those to its safelist.

## Security advisories

- **Fix a failing advisory check in the PR that hits it when the fix is small:** a patch or minor upgrade, or a
  package the change already touches. Otherwise record it with a recommended next step and ask.
- **If another package's version pin blocks the fixed release,** propose vendoring that package with the pin lifted;
  don't do it unasked.
- **Ignore an advisory only when no fixed release exists or the fix needs a major upgrade you can't make yet,** with a
  comment saying why and when to remove the ignore. (enforced by: <your advisory scanner in CI>)

## Licenses

- **Our policy: permissive licenses (MIT, Apache 2.0, BSD), plus LGPL and MPL 2.0 on their terms** (publish your
  changes to MPL files). <If customers run your builds:> everything in them is distributed, and we don't want
  GPL-style obligations or a license review on every release. No GPL or AGPL, even as a separate program you
  ship. Ask before any other copyleft license. **When this doesn't fit:** you never distribute your software; then your
  policy can be looser, but check it with whoever owns licensing.
- **Check the licenses of what a dependency pulls in,** not only its own. Ship license copies for everything a browser
  bundle includes.
- **Our choice: no paid libraries.** A paid product can be a reference for behavior, never a dependency.

## Libraries

### <Library name>

- `<path>` is <library> <version> (<license>), from <official source>. <What it's for.>
- **Update:** <the command or steps; which other files must change with it>.
- **Local patches:** <none, or each marked change and why>.
