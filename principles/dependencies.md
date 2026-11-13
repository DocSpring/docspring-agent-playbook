# Dependencies

### Ship permissive licenses, plus LGPL and MPL on their terms.

Our allow-list is permissive licenses (MIT, Apache, BSD) plus LGPL and MPL-2.0, whose obligations we meet. Customers
run our images, and we don't want GPL-style obligations or a license review on every release. That includes programs
we only
call as separate processes, which we'd still be distributing. MPL-2.0's copyleft covers only the MPL files themselves,
so publish any change you make to one. Ask before adding anything under another copyleft license, and check the
licenses of everything a new dependency pulls in. The same applies to machine learning model weights (a model page
without a license doesn't make the weights free) and to fonts and other assets a browser tool bundles, which also need
their license texts shipped. This is policy, not legal advice: what copyleft requires depends on how you combine and
distribute the code.

**How to enforce it:** A license checker in CI with an allow-list (for example, `license_finder`, `cargo deny`,
`pip-licenses` or `license-checker`), and a test that every bundled package's license text ships with the bundle.

**When this doesn't fit:** you never distribute your software (a hosted service only). Your license policy can then be
looser, but check it with whoever owns licensing in your organization.

*Scope: Teams that distribute software or images.*

### Build it or use open source; no paid libraries.

We don't take paid or commercially licensed libraries as dependencies: they add per-seat or per-install costs, license
keys in builds, and a vendor who can change terms. A paid product can be a reference for behavior, never a dependency.

**When this doesn't fit:** a commercial component is cheaper than the engineering it replaces and your procurement
process covers it. Then it's a purchasing decision for a person, not something an agent adds.

*Scope: Any team; a policy choice.*

### Pin downloaded artifacts by immutable reference and hash.

Model files, binaries and datasets you download at build time are pinned to a commit or release (never a branch) and
verified against a SHA-256. Never commit model weights to the repository.

*Scope: Any team.*

### Fix a security advisory in the pull request that hits it, when the fix is small.

When an audit tool fails your branch, upgrade the flagged dependency there if it's a patch or minor release, or a
package your change already touches: the next branch will hit the same failure. When the fix needs a major upgrade or
real code changes, record the advisory with a recommended next step and ask, rather than turning the branch into an
upgrade project. Ignore an advisory only when no fix exists or the user has chosen to defer it, with a comment saying
why and when to remove the ignore.

**How to enforce it:** Run the audit tool in CI, and fail on stale ignore entries as well as on new advisories.

*Scope: Any team.*

### Treat shared CI caches as shared state.

Anything a job writes to a shared cache reaches later jobs and other branches, including third-party code that arrived
with a dependency update. Run dependency-update branches on isolated runners or with separate caches, and never run a
shared clone's Git hooks.

*Scope: CI.*

### Fix bugs in your forks, and offer every fix upstream.

Carrying long-lived patches on a fork is fine. Build customizations on the upstream's extension points (configuration,
plugins, schema extensions, themes) instead of editing its components, so rebases stay small. When the upstream itself
needs a change, make the small, general change a maintainer would take: an option, a slot, a hook. Tag the old fork tip
before force-pushing a rebase, so older submodule pointers stay reachable. Mark local patches in vendored code with a
greppable comment and re-apply them on every update. Offering fixes upstream is a public post: it follows the rules in
[data-and-privacy.md](data-and-privacy.md).

*Scope: Any team that forks or vendors code.*
