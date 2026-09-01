# Styling

**Rule 3: never style with a custom CSS class.**

No `"className":"hero-banner"` paired with a `.hero-banner { … }` rule in a
stylesheet. Not in the theme's CSS, not in Additional CSS, not in a `<style>`
tag, not in a block's `style` support used as a smuggling route for arbitrary
CSS.

## Why

A custom class is invisible to the Site Editor. The client opens the section,
sees a plain group, changes the background, and nothing happens because a
stylesheet is overriding it. The design becomes uneditable by the person it was
built for, and the next developer has to read CSS to find out what a block
looks like. Everything in this reference is a way to keep the look *in the
block* or *in `theme.json`*, where the editor can see it.

## The ladder

Work down it. Stop at the first rung that fits.

### 1. One instance, unique look: block supports

Set it on the block. It serializes into the block's own attributes and shows up
in the editor sidebar exactly where the client expects.

```html
<!-- wp:group {"style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}},"border":{"radius":"12px"}},"backgroundColor":"base-2","layout":{"type":"constrained"}} -->
<div class="wp-block-group has-base-2-background-color has-background" style="border-radius:12px;padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
```

Supports available on most layout and content blocks: `color` (text,
background, gradient, link), `typography` (size, family, weight, line height,
letter spacing, transform, decoration), `spacing` (padding, margin, blockGap),
`__experimentalBorder` (color, radius, style, width), `dimensions` (min height,
aspect ratio), `shadow`, `layout`, `position` (sticky).

If the control is not in the sidebar, it is off in `theme.json`. Turn it on
under `settings.blocks.<block>` rather than routing around it with CSS.

### 2. Every instance of a block: `theme.json` block styles

The design says every quote is indented and italic. That is not a class on each
quote, it is one entry in `theme.json`.

```json
{
  "styles": {
    "blocks": {
      "core/quote": {
        "typography": { "fontStyle": "italic" },
        "spacing": { "padding": { "left": "var:preset|spacing|40" } }
      }
    }
  }
}
```

Same for elements: `styles.elements.button`, `.link`, `.heading`, `.h1` through
`.h6`, `.caption`. A design where all buttons are pill shaped is one
`elements.button.border.radius` value, not a class.

### 3. A named look the client should be able to pick: a style variation

The escape hatch when a look genuinely repeats and needs a name. It is
allowed because it appears in the editor's Styles panel: the client can apply
it, swap it and remove it. A bare CSS class cannot do any of that.

A block style variation lives as a JSON file in the theme's `styles/`
directory:

```json
{
  "$schema": "https://schemas.wp.org/trunk/theme.json",
  "version": 3,
  "title": "Card",
  "slug": "card",
  "blockTypes": ["core/group"],
  "styles": {
    "color": { "background": "var:preset|color|base" },
    "border": { "radius": "8px" },
    "spacing": { "padding": "var:preset|spacing|40" },
    "shadow": "var:preset|shadow|natural"
  }
}
```

Applied in markup by its generated class, which is the one `className` value
that is legitimate:

```html
<!-- wp:group {"className":"is-style-card","layout":{"type":"flex","orientation":"vertical"}} -->
<div class="wp-block-group is-style-card">
```

`blockTypes` may name several blocks, so one variation can cover
`core/group`, `core/column` and `core/cover` at once. Applied to a section
wrapper it also styles the blocks nested inside it through `styles.blocks`
inside the variation.

### 4. The whole site or a whole alternate look: a global style variation

Also a JSON file in `styles/`, but with no `blockTypes` and no `slug`. It
overrides `settings` and `styles` wholesale and appears under Styles, Browse
styles. This is for a dark mode or an alternate brand, not for a section.

## Missing tokens

When the design uses a value that is not in `theme.json`, the answer is always
to add the preset, never to write the raw value.

```json
{
  "settings": {
    "color": { "palette": [{ "slug": "surface", "color": "#f4f2ee", "name": "Surface" }] },
    "spacing": { "spacingSizes": [{ "slug": "70", "size": "6rem", "name": "70" }] },
    "typography": { "fontSizes": [{ "slug": "display", "size": "clamp(2.5rem, 5vw, 4rem)", "name": "Display" }] }
  }
}
```

Adding a preset is a design system decision, so say so when doing it. Six font
sizes in the design against four in the theme means two presets get added and
the type scale grew, which the designer should know.

Fluid type belongs in the preset as `clamp()` or via
`settings.typography.fluid`, not in a media query in a stylesheet.

## Responsive behaviour

There are no breakpoints to write for most layouts. Reach for these in order:

1. `layout.type: grid` with `minimumColumnWidth`, which reflows on its own.
2. `layout.type: flex` which wraps by default.
3. `core/columns` stacks below the mobile breakpoint automatically unless
   `isStackedOnMobile` is set to `false`.
4. Fluid presets: `clamp()` in a font size or spacing preset.

If after all four something still needs a media query, that is a `theme.json`
level concern, handled by overriding preset custom properties at a breakpoint in
the theme's own stylesheet. It is never a class attached to one section.

## Forbidden, with the replacement

| Do not | Do |
| --- | --- |
| `"className":"hero"` plus `.hero {}` | Block supports on that group |
| `!important` anywhere | Fix it in `theme.json` where the cascade starts |
| `<style>` in a pattern | Block supports, or a style variation |
| Additional CSS in the Customizer | `theme.json` |
| A wrapper `div` that exists only to hold a class | Set the style on the block that is already there |
| `"style":{"css":"…"}` for arbitrary rules on one block | A style variation if it repeats, supports if it does not |
| Inline `style` attribute written by hand | The matching block support, which writes it for you |
| A utility class system | Presets, which already are one |
