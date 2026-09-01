# Core block map

The vocabulary. If a design element is not on this page, it is a composition of
things that are.

## Check availability first

The core block set grows every release. Before using one of the newer blocks,
confirm the target site actually has it:

```
ls <site-root>/wp-includes/blocks/
```

That directory is the authoritative list of core blocks for that install. Do not
rely on a block existing because a newer WordPress has it.

Version gates below come from the `@since` tags in core's own files, except the
two marked as inferred, which were established by diffing installs on disk:

| Block | Availability |
| --- | --- |
| `core/details` | 6.3 |
| `core/accordion` and its children | 6.9 |
| `core/terms-query`, `core/term-template`, `core/term-name`, `core/term-count` | 6.9, inferred from install diff |
| `core/post-time-to-read`, `core/post-comments-count`, `core/post-comments-link` | 6.9 |
| `core/math` | 6.9, inferred from install diff |
| `core/icon` | 7.0, the icon registry behind it is 7.1 |
| `core/breadcrumbs` | 7.0 |
| `core/tabs`, `core/tab-list`, `core/tab-panel`, `core/tab-panels` | 7.1 |
| `core/playlist` | 7.1 |

When a block is not available on the target, the fallback is listed in the map
below. Never fill the gap with a custom block or `core/html`.

## Structure and layout

| Design element | Block | Notes |
| --- | --- | --- |
| Section band | `core/group` | `tagName` set to `section`, `header`, `footer`, `main`, `aside` |
| Vertical stack | `core/group` | `layout.type: flex`, `orientation: vertical` |
| Horizontal row | `core/group` | `layout.type: flex`, wraps by default |
| Centered content column | `core/group` | `layout.type: constrained` |
| Card grid, equal cards | `core/group` | `layout.type: grid` with `minimumColumnWidth` or `columnCount` |
| Two or three unequal columns | `core/columns` and `core/column` | `width` per column, only for genuinely unequal tracks |
| Image with text beside it | `core/media-text` | Two parts only, has its own stacking behaviour |
| Full bleed image with text on it | `core/cover` | Overlay, dim ratio, focal point built in |
| Divider line | `core/separator` | Not an empty group with a border |
| Fixed empty space | `core/spacer` | Last resort, prefer `blockGap` and padding |

`core/columns` is not a grid. A row of equal cards is a `group` with
`layout.type: grid`. Reach for `columns` when the tracks have different widths
and the design means them to be different.

## Content

| Design element | Block |
| --- | --- |
| Heading | `core/heading` with the right `level` |
| Body copy | `core/paragraph` |
| Bulleted or numbered list | `core/list` and `core/list-item` |
| Pull quote or testimonial | `core/quote` or `core/pullquote` |
| Table | `core/table` |
| Code sample | `core/code` |
| Preformatted block of text | `core/preformatted` |
| Button, single | `core/buttons` wrapping one `core/button` |
| Button group | `core/buttons` with `layout.type: flex` |
| Icon | `core/icon`, fallback `core/image` with an SVG |
| Logo | `core/site-logo` for the site's own, `core/image` otherwise |
| Social links row | `core/social-links` and `core/social-link` |

A single button still goes inside `core/buttons`. A lone `core/button` is
invalid markup.

## Interactive

| Design element | Block | Fallback |
| --- | --- | --- |
| Expand and collapse group | `core/accordion` with `core/accordion-item` | `core/details` repeated |
| One disclosure | `core/details` | none needed |
| Tabbed panels | `core/tabs` with `core/tab-list` and `core/tab-panels` | `core/accordion`, then agree the change with the designer |
| Site navigation | `core/navigation` | none, always this block |
| Search field | `core/search` | none |
| Slider or carousel | there is no core block | Redesign as a grid, a gallery or an accordion, or get explicit approval before anything custom |

If the design has a carousel, say so plainly rather than building one. A grid
that wraps is usually a better answer and always a cheaper one.

## Media

| Design element | Block |
| --- | --- |
| Single image | `core/image` |
| Image set or masonry | `core/gallery` |
| Video | `core/video`, or `core/embed` for a hosted one |
| Audio | `core/audio` |
| Background image behind content | `core/cover`, or `style.background` on a `core/group` |

## Dynamic and site data

| Design element | Block |
| --- | --- |
| List of posts | `core/query` with `core/post-template` |
| Post title, date, excerpt, image, terms | `core/post-title`, `core/post-date`, `core/post-excerpt`, `core/post-featured-image`, `core/post-terms` |
| Post body in a template | `core/post-content` |
| List of categories or tags | `core/terms-query` with `core/term-template`, or `core/categories` |
| Archive title | `core/query-title` |
| Breadcrumbs | `core/breadcrumbs` |
| Site title, tagline, logo | `core/site-title`, `core/site-tagline`, `core/site-logo` |
| Comments | `core/comments` and its children |
| A reused section | `core/template-part`, or `core/pattern` for a synced insert |

Anything showing a field that is not on this list is a Block Bindings job on an
ordinary core block. See [dynamic-content.md](./dynamic-content.md).

## Blocks that are never the answer

| Block | Why | Instead |
| --- | --- | --- |
| `core/html` | Unstyleable, uneditable, invisible to the Site Editor | Find the real block |
| `core/shortcode` | Same, plus it hides content from search and the editor | The block the shortcode wraps |
| `core/freeform` | Classic editor leftover | Real blocks |
| `core/legacy-widget` | Widget era leftover | Real blocks |
| `core/missing` | A block that failed to load | Fix the markup |
