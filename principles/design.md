# Design

Principles for product UI. The first rules apply to any product. The last section is our own design system, as an
example of the decisions a design system's guide should make.

### Follow your design system, and change the system rather than one page.

The same thing looks the same on every page. When a page seems to need something new, change the system (the theme,
the components and the guide) and use it everywhere, rather than styling one page. A redesign covers every variant of
a page, and no page ships unstyled, internal tools included.

*Scope: Teams with product UI.*

### Use the components, and add a variant instead of restyling one.

If a page needs something a component doesn't do, add a variant to the component, its guidelines and the theme,
rather than overriding its look from one view. Each component ships with guidelines, examples and tests, and a
reference page renders them, so agents and people can see what exists before building another spinner, modal or menu.

**How to enforce it:** A lint rule or test that flags style overrides passed into components from views.

*Scope: Teams with a component library.*

### Every color comes from a token, and every pair meets its contrast minimum.

A fixed color (a white background class, a hex value, a white icon) breaks the next theme you add, and an unchecked
pair fails people with low vision. Every theme you support is first-class: it gets the same review as the default.

**How to enforce it:** Generate every platform's color variables from one token file, fail the build when a color
pair drops below its contrast minimum, and lint stylesheets for literal colors (for example, Stylelint's
`color-no-hex`).

*Scope: Teams with product UI.*

### Design every state.

Hover, focus, pressed, selected, disabled, empty, loading, error, and long or wrapping content. Everything reachable by
mouse is reachable by keyboard, with a visible focus. An undesigned state is the one a customer screenshots in a
support ticket.

*Scope: Teams with product UI.*

### Nothing moves under the pointer.

Controls keep their place and size while their state changes: a selected tab doesn't get bolder, Previous and Next are
disabled rather than hidden, and loading placeholders have the shape of the rows they stand for.

*Scope: Teams with product UI.*

### Follow conventions.

A new screen looks like its nearest sibling in the product, and behavior follows what people already know from
browsers and the tools they use every day. Take ideas from other products without copying their look.

*Scope: Teams with product UI.*

### Do what was asked, and say so early.

No surprising defaults or silent substitutes: an action makes what the person chose, and a request that can't be done as
asked gets a clear error. Tell people what you know (this step takes a minute) before the slow work starts.

*Scope: Any team.*

### Never run formatters that change rendered whitespace on templates.

HTML and template formatters, and linters' autofix modes, move whitespace that the page shows. Lint templates, and fix
findings by hand.

*Scope: Teams with server-rendered templates (ERB, Jinja, Blade, Handlebars).*

## Our design system, as an example

These are our taste, written down so agents stop guessing. Copy the ones you agree with, or use them as a list of the
questions your own guide should answer.

**When this doesn't fit:** you already have a design system or a brand that says otherwise. Follow it.

### Keep the interface calm, exact and easy to scan.

Neutral surfaces, one action color, and color that means state. No decoration that doesn't carry information:
gradients, colored card stripes, illustrations and emoji compete with the content people came to read.

### Make content as wide as it reads well.

Pages sit in a centered container, tables size their columns to their content, and forms stay narrow. Nothing stretches
just because the window is wide.

### Light and dark themes, both first-class.

We ship both, and dark mode works from the tokens alone, without dark-only CSS.

### Selected beats hover.

A selected item keeps its selected look under the pointer, and everything clickable shows the pointer cursor.

### Align optically, and keep the nudges in the theme.

Icons, dots and checkboxes are placed by eye, and those adjustments live in the theme with a comment, so views never
nudge anything themselves.

### Views never pass classes that change how a component looks.

Our lint rule rejects them outright, so the only way to a new look is a new variant.
