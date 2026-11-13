<!--
Example nested AGENTS.md for public API code. Copy it to <app/controllers/api/>AGENTS.md, replace every <placeholder>,
and delete every rule that doesn't apply to your API. Most rules hold for any API customers call from their own code;
the sections that start with "If" apply only when they match your API. Delete this comment.
-->

# Public API

This directory holds the controllers for the public API at `<api.example.com/v2>`. The request schemas are in
`<schemas/>`, the generated OpenAPI document is `<openapi/v2/openapi.json>`, and the API tests that generate it are
in `<spec/api/>` (read `<spec/api/AGENTS.md>` before changing one). Customers keep client libraries and their own
generated clients for years: treat every response field, parameter and error as a contract.

## One source of truth

- **One schema validates requests and generates the docs.** The schemas that validate live traffic are the files the
  docs generator reads, so the docs can't describe something the API doesn't do, and a schema change is a docs and
  client library change in the same PR. (enforced by: <a test that every route has a schema>)
- **Reject an invalid request in every environment.** If production validates more loosely than tests, a rejection in a
  test doesn't mean production rejects that request, and customers come to depend on what it lets through.
- **Regenerate the API document with `<the generate command>` and commit the whole result.** Stub everything random or
  local (IDs, tokens, times) so every run produces the same bytes. <If your generator can't assemble partial or
  parallel runs deterministically:> never merge partial runs.
  (enforced by: <a CI job that regenerates the document and fails on a difference>)

## Additive changes only

- **Add; never change or remove.** Additive: a new response field, a new optional parameter, a new endpoint, and a new
  enum value only when supported clients tolerate unknown values. Breaking, and needing a new API version: rejecting
  input the API accepts today, removing or renaming a field, changing a field's type or making it nullable, making a
  parameter required, removing an enum value, or changing what the same request does. A bug fix can break integrations
  too: check what supported clients rely on, documented or not, before calling a change safe.
- **Count real usage before deciding nothing breaks.** Run a bounded, read-only query in production. The accounts that
  rely on the old behavior keep it behind a per-account gate and get notice; everyone else gets the change.
- **A new parameter answers an unknown value with an error that lists the accepted values** (<422>). Never silently do
  what a default would: a silent fallback becomes an undocumented contract. Tightening an existing parameter that
  accepted anything is a breaking change: count usage first.
- **One way to send each request.** A value that could mean two things is an error, never a guess.
- **Make writes with effects that mustn't happen twice safe to retry.** <Our choice: every create and every
  non-repeatable write takes an idempotency key, recorded against what it created or changed in the same
  transaction.> Test a duplicate, two concurrent attempts and a retry after a failure; accepting the header proves
  nothing on its own.

## Responses and errors

- **Keep one shape for each kind of response.** <A resource returned as it is; a list as `{data, next_cursor}`; one
  error shape; 201 for creates; 404 for an unknown ID in the path, 422 for one in the body; 409 for a state conflict.>
- **Log only parameter values that can't hold customer data.** Log IDs, numbers, booleans and enum values; log other
  strings as their type and size.

## If you generate client libraries

- **Check what the generator really produces** before deciding a client can or can't do something, and extend the
  generator when the API needs something new instead of bending the API to suit it.
- **Give every schema object and enum a title.** <If your generator names classes after titles:> an untitled object
  gets a name made from an operation or status code, and adding a response can rename a model in every client.
- **Positional arguments never move.** <If your generator takes an operation's arguments in the document's order:> a
  new parameter goes after the existing ones, and new options on a write go in its JSON body, which is one argument
  however many properties it has. A moved argument still compiles in a customer's code and sends values as the wrong
  parameters. (enforced by: <a CI check that models argument order>)
- **Clients tolerate fields and enum values they don't know.** Leave response objects open in the document, and keep
  an unknown enum value as it came; a client that rejects new fields breaks on every additive change. Test it with an
  example that returns a field and an enum value no client has seen, in every client language.

## If you send webhooks

- **Webhooks speak the version that created them.** A webhook keeps the payload version of the API version that created
  it, so integrations on the old version never see a change.
