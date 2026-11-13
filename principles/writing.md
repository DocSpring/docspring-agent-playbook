# Writing

Rules for product copy, docs and anything customers read.

### Write to "you", and lead with what the reader came to do.

"Your project has no members yet." Give the quickest way to do the task and say what's needed where it's needed, not a
list of everything that's missing. Plain and short: active voice, sentence case for titles, buttons, labels and table
headers, no all-caps labels, no exclamation marks, and no marketing adjectives inside the product.

*Scope: Any team with product UI.*

### Use one name for each thing across the UI, the API and the docs.

The "Invite members" button calls `invite_members`. Use the reader's words: the name the industry uses rather than one
you made up, and no internal terms in public copy. A word like "Required" means one thing everywhere it appears.
Customers who see two names assume two things.

*Scope: Any team.*

### Errors say what happened and how to fix it.

No apologies, "Oops" or a bare "Something went wrong". Stay calm: state the action and how urgent it really is.

*Scope: Any team with product UI.*

### Never show a label without its value.

Something without a name says so ("Untitled project", with its ID). An empty label next to a blank looks like a bug.

*Scope: Any team with product UI.*

### Keep safety caps out of copy until someone reaches one.

A hidden safeguard that almost nobody hits (a maximum number of items per list) appears only in the error shown on
reaching it, and in reference docs if it fits there. Mentioning it everywhere makes a safeguard read as a limitation.

*Scope: Any team with product UI.*

### Directions match the interface, and changes are described as they are.

Name where a setting really is, and don't point to something that isn't on the page. A changelog entry says exactly
what changed for the customer.

*Scope: Any team.*

### Document only what exists, as it works now.

Check the code and the API document before writing a sentence about them. Guides describe current behavior, not its
history. Be direct and factual: no "Benefits" sections. Never hard-code a version number that will change; use a
placeholder. Keep related options in the main guide, and keep endpoint descriptions short with the details in guides.

*Scope: Any team.*

### Everything in the public docs directory is public.

Never put internal notes, agent files or customer details there, even temporarily: it gets built and published.

**How to enforce it:** A CI check that fails on agent files (`AGENTS.md`, `CLAUDE.md`) or an internal-notes path inside
the public docs source.

*Scope: Any team with public docs.*

### Alt text says what an image shows.

Describe the content ("The settings page with the webhook URL filled in"), not "Screenshot". Serve modern image formats
with fallbacks, check large SVGs for embedded raster images, and compare compressed images at 2x before committing.

*Scope: Any team.*

## If you write marketing pages or email customers

### Word claims exactly, and only about what's live.

Use the precise word for a compliance status ("compliant" is not "certified"), and name only the certifications you
hold. Never advertise a feature that isn't built, and check every feature an email mentions is already live. Read
prices from the price catalog instead of typing them into a page. Back every comparison claim with a source, admit
where a competitor is cheaper, and update a "reviewed on" date only after re-checking.

*Scope: Teams with marketing pages.*

### Agents draft customer-facing text in a pull request, and a person approves it there.

The pull request is the approval record. Run a pass that removes machine-sounding phrasing before asking for review. An
unknown template variable raises instead of sending a blank, and an email's text makes sense with images blocked.

*Scope: Teams that send email to customers.*

### Release notes are for the record; announcements are a separate choice.

Keep release notes for every change users or operators need to understand, small fixes, compatibility changes and
security patches included. Decide separately which changes get announced, by the audience and the channel: an
announcement interrupts people, so it goes out only for news most of them would want (a new feature, something they
must act on, a legal change). A fix one customer reported gets a reply to that customer. Open with what changed, not
background. When unsure whether to announce, leave it out and ask.

*Scope: Teams with release notes or product announcements.*
