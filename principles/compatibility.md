# Compatibility

For teams whose customers build on what they ship. The first rules apply to any such team; the sections after them
apply only if you ship that kind of thing.

### Decide what you support, then never break it by accident.

Write down what customers can rely on: which API versions, client releases, embed versions and self-hosted releases
you support, and for how long. A change is breaking if it changes behavior a supported client or integration uses,
documented or not: rejecting input accepted today, removing or renaming something, changing a type or a default, or
changing what the same request does. A bug fix can still break an integration that worked around the bug, so judge by
what supported clients do, not by whether the old behavior was intended.

**How to enforce it:** Test your supported clients (the oldest and newest of each line) against the server in CI.

*Scope: Teams with public APIs or integrations.*

### Count who relies on a behavior before you break it, and gate only those accounts.

When a change must stop something customers rely on and it can't go into a new API version (a link that kept working,
an output format, a value an unversioned endpoint accepted), count in production who relies on it, with a bounded,
read-only query. Those accounts keep the old behavior behind a per-account gate and get notice long enough to change
their integration (for example, three months), with a reminder and a ticket for the second notice and for removing the
gate. Everyone else gets the change with the deploy.

A fix to output that is visibly wrong (a value in the wrong cell, a number cut off) ships to everyone without a count
or a gate: that's our choice, because customers want the correct value and a gate would keep the wrong one. If an
integration might read the wrong output (a machine-readable field, or extraction that reads a value by its position),
it is a behavior change: count first.

**How to enforce it:** Read the gate through one predicate, where a missing gate means the new rule applies, so new
accounts and other installs need nothing.

*Scope: Teams with public APIs or integrations.*

### Renames are total, and old names become aliases.

When a customer-facing concept is renamed, every name a customer can see changes: copy, docs, URL paths, query
parameters, field names, environment variables, scripts and commands. The old names keep working as aliases for as
long as any supported client, integration or install might send them. Customers see one consistent name, and nothing
they built stops working. The code and database can keep the old name.

We never remove an alias, because customers run old self-hosted releases and old SDKs for years and we can't see
them all.

**When this doesn't fit:** you can see every caller and you've declared a support window. Then remove an alias once
no supported version sends it and your logs show nobody has used it for the whole window.

*Scope: Teams with public APIs or self-hosted editions.*

### Published artifacts are immutable.

A published package version, a sent email's images, a released changelog URL, a published model file, an image tag:
never change or delete one. Write a new one. Someone has already pinned, cached or linked to it.

*Scope: Any team.*

## If you ship a public API

### Public APIs take additive changes only.

Additive: a new response field, a new optional request parameter, a new endpoint, and a new enum value if your
published clients accept values they don't know (see the SDK section; if old clients reject them, a new enum value is
breaking for them). Everything else in the breaking list above needs a new API version. Correcting the schema to
match what the API already returns doesn't change the wire behavior, but it can change generated client signatures (a
field that becomes nullable): treat it as a client change and version the SDKs accordingly. Customers keep SDKs and
their own generated clients for years, so check production usage before deciding nothing breaks.

**How to enforce it:** Run an OpenAPI diff tool (for example, oasdiff) in CI against the published document and fail
on breaking changes.

*Scope: Teams with public APIs.*

### Writes that customers retry are safe to retry.

Decide for each write what happens when a client sends it twice: after a timeout, from a retrying job, or twice at
once. Writes that leave the same state when repeated (setting values, deleting) are safe as they are. Writes that
would create or change something again (creating a record, adding items, publishing a version, charging a card) need
a way to recognize the repeat.

What we do: every such write takes an idempotency key. A create records the key against the record it created, and a
write that changes state links the key to the record it changed, in the same transaction, so a slow request retried
by the client can't take effect twice. Payment and ledger writes use deterministic IDs for the same reason: a repeated
webhook can't credit an account twice.

**How to enforce it:** Test each non-repeatable write with a duplicate request, two concurrent requests with the same
key, and a retry after a failure, and assert one effect. Accepting the header proves nothing on its own.

*Scope: Teams with public APIs.*

## If you generate SDKs

### Client libraries tolerate response fields and enum values they don't know.

Published clients accept new fields (open objects) and new enum values (open enums with an "unknown" case). A client
that rejects an unknown field turns every additive server change into an outage for the customers on that release.
Keep your own API tests strict instead: closed schemas in the test suite fail on an undocumented field, while the
published document stays open.

**How to enforce it:** In each client language's tests, include an example response with a field and an enum value the
client has never seen, and assert it parses.

*Scope: Teams with SDKs.*

### Generated signatures never change under customers' code.

Many generators take arguments in the order the API document lists them and name models after schema titles, so a
parameter inserted in front, or a renamed schema, still compiles in a customer's code and sends their values as the
wrong parameters or breaks their imports. Check what your generator really does with each kind of change, and write
down the conventions it needs (with the generators we use: new parameters after existing ones, optional headers last,
new options on a write in its JSON body, which is one argument however many properties it has).

**How to enforce it:** A CI check that compares each operation's generated signature and model names with the
released SDK's and fails a branch that changes an existing one. An intended change goes into a new major version
through your normal review.

*Scope: Teams with generated SDKs.*

## If you ship code that runs on customers' pages

### Released embed versions are frozen.

Every change to the bytes of code that runs on customers' pages (embeds, loaders, widgets) ships under a new version
number, including dependency and security updates, and a released version's file is never overwritten. Protocol
strings and option names that published loaders send are kept forever: map new names in the loader. Public CSS custom
properties and class names are API, and so are third-party globals you bundle.

Whether a release gets an announcement is a separate decision: a dependency update customers won't notice can ship
quietly, under its new version, and wait for the next release worth announcing. Urgent security fixes follow your
security policy.

*Scope: Teams with embeds or SDKs that run in customers' pages.*

## If you send webhooks

### Webhooks speak the API version that created them.

A webhook keeps the payload version of the API used to create it. Integrations built against the old version never see
a change, and new payloads go only to webhooks created on the new version.

*Scope: Teams with webhooks.*

## If customers run your software themselves (self-hosted)

### Self-hosted upgrades work with the old configuration and data.

An upgrade must work with an install's existing environment and manifests: add shims and fallbacks, never manual
upgrade steps, because nobody reads the upgrade notes until something breaks. Installs can jump many releases at once,
so imports keep reading archives written by old releases (handle missing keys; never rename or drop keys), and
database columns are never dropped: a column that looks unused may still be read by a release an install is about to
upgrade from, or rolled back to.

**When this doesn't fit:** you run every deployment yourself and declare a support window. Then use expand and
contract: add the new column, move readers and writers to it, and drop the old one once no supported version reads it.

*Scope: Teams with self-hosted editions.*
