<!--
Example nested AGENTS.md for app views and UI components. Copy it to <app/views/>AGENTS.md (and point
<app/components/>AGENTS.md at it), replace every <placeholder>, and delete every rule that doesn't apply to your stack.
The example design system section is ours: replace it with yours. Delete this comment.
-->

# App views and components

This directory holds the app's pages. Read `<internal_docs/design-system.md>` before you build a page or change how
one looks: it says which component to use for what, how list, settings and detail pages are laid out, and the token
tables. The components live in `<app/components/>`, and a reference page at `<the design system page's URL>` renders
every component with its guidelines and examples.

## Rules for every screen

- **Follow the design system.** Use its layouts, components, tokens and themes. When a page seems to need something
  new, change the system and use it everywhere; no page ships unstyled, internal tools included.
- **Design every state:** hover, focus, pressed, selected, disabled, empty, loading, error, and long or wrapping
  content. Each control shows the effective value, and separately whether the viewer can change it.
- **Nothing moves under the pointer.** Controls keep their place and size while their state changes; disable Previous
  and Next rather than hiding them.
- **Keep it accessible:** keyboard access to everything, visible focus, text contrast your design system meets (<WCAG
  AA>), and labels on every control.
- **Follow conventions.** A new screen looks like its nearest sibling in the app.
- **Do what was asked, and say so early.** No surprising defaults or silent substitutes; a request that can't be done
  as asked gets a clear error.

## Our design system, as an example

<These are the rules our own app follows. When this doesn't fit: you already have a design system, or your product
is meant to be expressive; follow yours and delete this section.>

1. **Calm, exact and easy to scan.** Neutral surfaces, one action color, and color that means state. No decoration that
   doesn't carry information.
2. **Content is as wide as it reads well.** Nothing stretches because the window is wide.
3. **Both themes are first-class.** Every color comes from a token, so dark mode needs no dark-only CSS.
4. **Optical alignment beats arithmetic.** Those nudges live in the theme with a comment, never in a view.

## Components and tokens

- **Use the components; never pass classes that restyle one.** A card title or button restyled in one view drifts from
  every other page. If a page needs something a component can't do, add a variant to the component, its guidelines and
  the theme. (enforced by: <a lint rule against style overrides on components>)
- **Use color tokens, never literal colors.** No hex values, inline color styles or a CSS framework's fixed color
  classes (`bg-white`, `text-red`): each one is a theme bug waiting to happen. Use the functional tokens
  (`--text`, `--surface`, `--danger-wash`), not the palette steps. (enforced by: <a stylelint rule or a grep in CI>)
- **A new component ships with its guidelines, examples and tests.** The reference page renders them, so keep them in
  step with the code.

## Scripts and the Content Security Policy

- **No inline event handlers.** A strict CSP blocks `onclick="..."` attributes, so they do nothing and each one sends a
  violation report. Attach handlers from a script file. (enforced by: <a CI check that rejects `on*` attributes>)
- **Inline scripts carry the request's nonce** (for example, in Rails, `javascript_tag nonce: true`). <If the test
  environment's CSP is looser:> tests won't notice a missing nonce, so check each one in review.
- **Serialize values into a script with an escaper that neutralizes `<`, `>` and `&`.** Rails' `to_json` does by
  default; `JSON.generate` and `JSON.stringify` don't. Never a value in quotes, which can end the script element or
  break on a backslash.
- **Translations that hold HTML escape their interpolated values.** Mark the translation as HTML by its key (for
  example, in Rails, a key ending in `_html`); never mark a translated string safe as a whole.
- <If you ship code that runs on customers' pages:> **it works under their strict CSPs.** No `eval`, and no
  `<style>` elements or `style` attributes in HTML strings.
  (enforced by: <a browser test that loads each embed under a strict policy>)

## Templates

- **Never run a formatter or a linter's fix mode that changes rendered whitespace.** Template formatters move spaces
  and newlines the page shows, and some of their fixes change what renders. Fix lint findings by hand; suppress a false
  positive with the reason on the line above.
- **Give partials explicit, typed locals** where your template language supports it, instead of reading instance
  variables.

## Checking what you changed

- **Look at every changed screen in each theme you support, at <desktop and phone width>.** Check each state the change
  touches (empty, loading, error, long text), and the narrowest width you support where long words can overflow.

## Writing

- **Write to "you", and lead with what the person came to do.** Don't over-explain; say what's needed where it's needed.
- **Sentence case everywhere:** titles, buttons, menu items, table headers, labels. No all-caps labels.
- **One name for each thing across the UI, the API and the docs, in the reader's words.** No internal terms in copy.
  A word like "Required" means one thing everywhere.
- **Never show a label without its value.** Something without a name says so ("Unnamed project", with its ID).
- **Errors say what happened and how to fix it.** No apologies, "Oops" or a bare "Something went wrong".
- **Keep safety caps out of copy until someone hits them.** Only the error at the cap mentions it.
- **Plain, short and calm:** active voice, no exclamation marks, no marketing adjectives inside the app, nothing
  alarming or pushy.
- **Directions match the interface.** Name where a setting really is; never point to something that isn't on the page.
