---
name: hlb-site-editor
description: "Turn a design into WordPress Site Editor block markup. Use when the input is a design artifact: build a pattern, template part, template, page layout or section from a Figma file, screenshot, mockup, PDF or written design description, or lay out a page in the Site Editor from one. Not for pattern work with no design to translate, use wp-patterns for that. Enforces core blocks only, SVG through core/image or core/icon, styling through block supports and theme.json instead of custom CSS classes, and Block Bindings or block variations instead of render_block filters keyed off a class."
---

# HLB Site Editor

Translating a design into WordPress block markup: patterns, template parts,
templates and page layouts, built out of core blocks and styled in the Site
Editor.

## The four rules

These are not preferences. Work that breaks one of them gets rewritten.

### 1. Core blocks only

No custom blocks. No `core/html`. No raw markup smuggled into a pattern file.
Every element in the design maps onto a block WordPress already ships. If it
looks like nothing maps, the mapping is being read too literally, see
[core-blocks.md](./references/core-blocks.md).

The editor is the deliverable as much as the frontend is. A client who opens a
custom block or an HTML block in the Site Editor cannot edit it the way they
edit everything else.

### 2. SVG goes through `core/image` or `core/icon`

An inline `<svg>` in an HTML block is not editable, not styleable from the
editor, and not replaceable by the client. Logos and illustrations become
`core/image` pointing at an uploaded SVG. Interface icons become `core/icon`.

Images and SVGs from the design are **media library uploads**, not files
committed into the theme or a plugin. Import them, set alt text on the
attachment, and reference them from there. The only files that stay in the repo
are icons registered through the Icons API, which has no upload backed source.
See [images-and-svg.md](./references/images-and-svg.md).

### 3. Never style with custom CSS classes

No `className` invented for this design plus a rule in a stylesheet. Everything
visual comes from block supports serialized into the block markup, from
`theme.json`, or from a registered style variation that shows up in the editor
UI. If a look cannot be produced that way, the token is missing from
`theme.json`, so add it there. See [styling.md](./references/styling.md).

### 4. Never hook a filter to a CSS class to produce content

No `render_block` filter that looks for `.my-section` and injects markup into
it. Dynamic values are Block Bindings. Repeated preconfigured structures are
block variations. Lists of posts or terms are Query Loop. Reused chunks are
template parts or patterns. See
[dynamic-content.md](./references/dynamic-content.md).

## Workflow

1. **Read the design.** Extract sections, the grid, the type scale, the
   spacing rhythm and the color roles before writing any markup.
   [reading-designs.md](./references/reading-designs.md)
2. **Check `theme.json` first.** Every color, font size and spacing step the
   design uses must exist as a preset. Add the missing ones now. A design built
   against presets that do not exist ends up with hardcoded values.
3. **Decompose into blocks, outermost first.** Section wrapper, then layout,
   then content. [core-blocks.md](./references/core-blocks.md)
4. **Decide where the output lives.** Pattern, template part or template.
   [output-targets.md](./references/output-targets.md)
5. **Write the markup.** Attributes and the saved HTML have to agree exactly.
   [block-markup.md](./references/block-markup.md)
6. **Wire dynamic parts.** Bindings, variations, Query Loop.
   [dynamic-content.md](./references/dynamic-content.md)
7. **Verify.** Insert it, confirm no block validation errors, compare against
   the design. [verification.md](./references/verification.md)

## Decision flow

```
An element in the design
├── Text?                     → heading / paragraph / list / quote
├── Picture or logo?          → image, or cover when text sits on top
├── Interface icon?           → icon  (WP 7.0+, else image with an SVG)
├── Call to action?           → buttons > button
├── Row, stack or grid?       → group with the matching layout type
├── Side by side media+text?  → media-text, or columns when more than 2 parts
├── Expand and collapse?      → accordion  (WP 6.9+), details below that
├── Tabbed panels?            → tabs  (WP 7.1+)
├── Repeating list of posts?  → query > post-template
├── Repeats across pages?     → template part, or a pattern
├── Value comes from data?    → block bindings on a core block
└── Nothing fits?             → re-read the design, it almost always does
```

## Never

| Do not | Do instead |
| --- | --- |
| Register a custom block for a layout | Compose core blocks |
| Use `core/html` | Find the block that produces that element |
| Inline `<svg>` in markup | `core/image` with an SVG, or `core/icon` |
| Commit design imagery to the theme's `assets/` | Import it to the media library |
| Hardcode an upload URL in a pattern | Resolve the attachment by slug, or leave the image for the client |
| `className` plus a stylesheet rule | Block supports, `theme.json`, or a registered style variation |
| `!important` | Fix the specificity at the `theme.json` level |
| Hardcode `#hex`, `24px`, `1.5rem` | Presets: `var:preset|color|...`, `var:preset|spacing|...` |
| `render_block` filter matched on a class | Block Bindings, block variations, Query Loop |
| A shortcode inside a pattern | The equivalent core block |
| `core/spacer` to fake margins | `blockGap`, padding or margin on the parent |
| Nested `core/columns` to build a grid | `group` with `layout.type: grid` |

## References

| File | Covers |
| --- | --- |
| [reading-designs.md](./references/reading-designs.md) | Getting structure and tokens out of Figma, a screenshot or a PDF |
| [core-blocks.md](./references/core-blocks.md) | Design element to core block map, availability by WP version |
| [block-markup.md](./references/block-markup.md) | Serialization, attribute and class mirroring, layout types, composites |
| [styling.md](./references/styling.md) | Rule 3 in full: block supports, `theme.json`, style variations |
| [images-and-svg.md](./references/images-and-svg.md) | Rule 2 in full: image, icon, cover, backgrounds |
| [dynamic-content.md](./references/dynamic-content.md) | Rule 4 in full: bindings, variations, Query Loop |
| [output-targets.md](./references/output-targets.md) | Pattern versus template part versus template, file headers, registration |
| [verification.md](./references/verification.md) | Validating markup, checking against the design |

## Related skills

The boundary: **this skill when there is a design to translate**, whether that
is a Figma file, an image or a written description. When there is no design and
the task is pattern authoring, registration or review on its own terms, that is
`wp-patterns`.

- `figma-to-wp-theme-json` when the design system itself has to become `theme.json`
- `wp-patterns` for pattern work with no design input
- `wp-block-themes` for block theme and Site Editor reference beyond layout work
- `wp-interactivity-api` when a section genuinely needs client side behaviour
- `wp-block-development` only if a custom block has been explicitly approved

## Checklist

- [ ] Every block is a core block, no `core/html`, no custom block
- [ ] No inline `<svg>` anywhere in the markup
- [ ] No invented `className`, no stylesheet written for this layout
- [ ] No `render_block` filter added for this design
- [ ] Every color, size and space is a preset reference, no raw values
- [ ] Attributes in the block comment match the classes and inline styles in the HTML
- [ ] Headings descend in order and the section uses semantic `tagName`
- [ ] Every image is a media library upload, none committed to the theme
- [ ] Images have real `alt` set on the attachment, decorative ones have `alt=""`
- [ ] Strings in a pattern file are translated and escaped
- [ ] Inserted in the editor with no block validation warning
