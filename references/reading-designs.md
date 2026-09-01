# Reading designs

Before a single block comment is written, the design has to be reduced to a
structure and a token set. Building block by block from the top of a mockup
produces markup that looks right at one width and collapses at every other.

## Order of extraction

1. **Sections.** Horizontal bands, top to bottom. Each becomes one outer
   `core/group`. Name them out loud: hero, logo strip, feature grid, testimonial,
   CTA, footer.
2. **Section width behaviour.** For each band, is the background full width? Is
   the content constrained, wide, or full? That decides `align` on the wrapper
   and the layout type inside it.
3. **The grid inside each section.** Columns, rows, a card grid, or a single
   stack. Count the tracks and note the gap.
4. **The type scale.** Every distinct size in the design, collected into a set.
   Compare that set against `theme.json` `settings.typography.fontSizes`.
5. **The spacing rhythm.** Distinct paddings and gaps, collected into a set.
   Compare against `settings.spacing.spacingSizes`.
6. **The color roles.** Not the hexes, the roles. Background, surface, text,
   muted text, accent, border. Compare against `settings.color.palette`.
7. **The repeats.** Anything appearing more than once is a candidate for a
   pattern, a template part or a style variation.
8. **The assets.** Every image, logo and illustration the design uses. Export
   them from the source, then import them to the media library. They are
   uploads, never files committed into the theme. See
   [images-and-svg.md](./images-and-svg.md).

## Source specific notes

### Figma

Prefer the structured data over the picture. `get_variable_defs` gives the real
token values, `get_metadata` gives the layer tree and the auto layout settings.
An auto layout frame maps directly:

| Figma | Block |
| --- | --- |
| Auto layout, vertical | `group` with `layout.type: flex`, `orientation: vertical` |
| Auto layout, horizontal | `group` with `layout.type: flex` |
| Auto layout, wrap | `group` with `layout.type: flex`, `flexWrap: wrap` |
| Grid or an evenly spread card set | `group` with `layout.type: grid` |
| Frame padding | `style.spacing.padding` on the group |
| Item spacing / gap | `style.spacing.blockGap` |
| Fill container | stretch or `flexBasis` on the child |
| Hug contents | the default, nothing to set |

If the file has variables, the design system is already tokenised. Route the
token work to the `figma-to-wp-theme-json` skill rather than transcribing hexes
into block markup.

### Screenshot or image

There is no layer tree, so measurement is estimation. Do not invent precision.

- Read proportions, not pixels. "Two thirds and one third" beats "812px".
- Snap every measurement to the nearest existing preset. A 22px gap in a
  screenshot is the theme's `spacing|30`, not a new `22px` value.
- Ask about anything the image cannot show: hover states, what the section does
  on mobile, whether a list is static or comes from posts.

### PDF or a written brief

Treat headings in the document as the section list. Everything else follows the
same order above.

## Mapping to presets

Never carry a raw value from the design into the markup. The pipeline is:

```
design value  →  nearest existing preset  →  var:preset|<type>|<slug>
```

If nothing is near enough, the preset is missing. Add it to `theme.json` and
then reference it. Two rules keep this honest:

- A design with six font sizes and a theme with four means adding two presets,
  not writing two inline `fontSize` values.
- A one off value that appears exactly once and belongs to no scale is a sign
  the design is drifting from its own system. Flag it rather than encoding it.

## Output of this step

Before writing markup, be able to state:

- the ordered list of sections and what each one is called
- for each section: alignment, layout type, track count, gap preset, padding preset
- the list of presets to add to `theme.json`, if any
- which sections are patterns, which are template parts, which are page content
- which parts are dynamic and what feeds them
- the list of assets to export and import, with the alt text for each
