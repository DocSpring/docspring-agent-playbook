# Data and privacy

### Use only the AI tools and data your organization has approved for the task.

Keep customers' and end users' personal and confidential data (form values, uploaded or generated documents, names,
emails, IP addresses, audit trails, webhook payloads, the payload columns of production tables) out of any session that
isn't approved for it. Select metadata and counts instead, use synthetic examples, and ask the person who owns the data
when you're unsure. An agent can't tell from inside a session what terms it runs under, so write down in your AGENTS.md
which tools are approved for which data, and have the agent ask the user to look at anything it isn't cleared to read.

*Scope: Teams that handle customer data.*

### Keep end users' personal data out of tickets, wiki pages, pull requests and commit messages.

End users are the people who use your customers' products: their names, emails, addresses, phone numbers, case numbers,
tokens and document contents. Refer to records by ID, or link the support conversation. Editing a ticket afterwards
doesn't remove the data: trackers and Git keep every version.

*Scope: Any team.*

### Fixtures are committed, so they hold synthetic data only.

Never put customer data in test fixtures, seeds or examples, even in a quick experiment that might get committed. Use
synthetic values, and sample files a person has checked for personal data.

*Scope: Any team.*

### Error reports and logs carry no customer data.

Strip request bodies, query strings, cookies and headers in the error SDK's `before_send` hook. Never report an error
whose message quotes a response body. Log job arguments only when they're IDs, numbers or booleans, and summarize other
strings as their type and size. The error SDK's state-store integration (Redux and the like) sends action types only,
never state. Never log tokens or the query string of a presigned URL. Redact secret-bearing path segments and query
parameters in every place that records URLs (request logs, error reports, analytics, proxies), and test the emitted
records. Naming a parameter `token` redacts nothing on its own.

**How to enforce it:** A test that raises an error through the SDK with a body, cookies and headers set, and fails if
the captured event contains any of them. It catches an SDK upgrade that starts collecting something new.

*Scope: Any team.*

### A tool that says it runs in the browser keeps documents in the browser.

If a page tells visitors their files stay on their device, nothing they open reaches your servers: no uploads,
analytics events or error context with document data, and the error SDK's network breadcrumbs limited to same-origin
requests. Where the tool does send something, the page says exactly what, and that stays true as the tool changes.

*Scope: Teams with in-browser tools.*

### Crop screenshots for public posts to the subject.

Leave out customer data, other accounts' names and anything else on screen that isn't the point of the image.

*Scope: Any team.*

### Publish only what's approved for a public audience.

Upstream issues, pull requests, forum posts, comments and blog drafts carry no secrets, personal data, or confidential
business or operational details. Use qualitative wording and synthetic figures when you don't have permission to use
real ones. Technical numbers from a reproduction (memory, versions, timings) are usually fine. Before an agent posts
anything public, a person approves the exact text.

**When this doesn't fit:** your organization publishes incident reports or metrics on purpose. Follow that policy for
those; the agent still doesn't decide on its own what's approved.

*Scope: Any team.*
