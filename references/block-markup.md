# Block markup

Block markup is HTML with the block's own attributes carried in a comment above
it. The comment and the HTML are two views of the same thing and **they have to
agree**. When they disagree the editor shows "this block contains unexpected or
invalid content" and the client loses the block.

## Syntax

```html
<!-- wp:group {"align":"full","layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull"><!-- inner blocks --></div>
<!-- /wp:group -->
```

Self closing, for blocks with no inner HTML of their own:

```html
<!-- wp:post-content {"layout":{"type":"constrained"}} /-->
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->
```

Attribute rules:

- Valid JSON. Double quotes, no trailing comma, no comments, no single quotes.
- Only non default values appear. Never write `{"level":2}` on a heading, 2 is
  the default and the editor will not write it back.
- Key order does not matter to the parser, but keeping `align`, `style`,
  `<color keys>`, `layout` in that order matches what the editor emits and makes
  diffs readable.

## The mirroring rule

Every attribute that produces a class or an inline style must appear in the
saved HTML too. This is the single most common way hand written markup breaks.

| Attribute | Class in the HTML | Inline style in the HTML |
| --- | --- | --- |
| `"align":"full"` | `alignfull` | none |
| `"align":"wide"` | `alignwide` | none |
| `"backgroundColor":"accent-2"` | `has-accent-2-background-color has-background` | none |
| `"textColor":"contrast"` | `has-contrast-color has-text-color` | none |
| `"fontSize":"x-large"` | `has-x-large-font-size` | none |
| `"fontFamily":"system"` | `has-system-font-family` | none |
| `"style":{"spacing":{"padding":...}}` | none | `padding-top:...` etc |
| `"style":{"spacing":{"margin":...}}` | none | `margin-top:...` etc |
| `"style":{"border":{"radius":"16px"}}` | none | `border-radius:16px` |
| `"style":{"color":{"background":"#111"}}` | `has-background` | `background-color:#111` |
| `"style":{"spacing":{"blockGap":...}}` | none | **none**, layout CSS handles it |
| `"textAlign":"center"` | `has-text-align-center` | none |
| `"verticalAlignment":"center"` | `is-vertically-aligned-center` | none |
| `"width":"56%"` on a column | none | `flex-basis:56%` |
| `"className":"is-style-x"` | `is-style-x` | none |

Worked example, exactly as the editor writes it:

```html
<!-- wp:group {"align":"full","style":{"spacing":{"margin":{"top":"0","bottom":"0"},"padding":{"top":"var:preset|spacing|50","bottom":"var:preset|spacing|50","left":"var:preset|spacing|50","right":"var:preset|spacing|50"}}},"backgroundColor":"accent-2","layout":{"type":"constrained"}} -->
<div class="wp-block-group alignfull has-accent-2-background-color has-background" style="margin-top:0;margin-bottom:0;padding-top:var(--wp--preset--spacing--50);padding-right:var(--wp--preset--spacing--50);padding-bottom:var(--wp--preset--spacing--50);padding-left:var(--wp--preset--spacing--50)">
	<!-- inner blocks -->
</div>
<!-- /wp:group -->
```

Two things to notice: the attribute uses `var:preset|spacing|50`, the inline
style uses the resolved `var(--wp--preset--spacing--50)`. And the inline style
runs top, right, bottom, left regardless of the key order in the JSON.

`blockGap` is the exception. It stays in the attributes and never appears in the
inline style, because WordPress emits it as a layout rule instead.

## Preset references

In **attributes**: `var:preset|color|primary`, `var:preset|spacing|50`,
`var:preset|font-size|large`.

In the **inline style**: `var(--wp--preset--color--primary)`,
`var(--wp--preset--spacing--50)`, `var(--wp--preset--font-size--large)`.

Where a block has a dedicated attribute for the preset, use it instead of
`style`. `"backgroundColor":"accent-2"` is correct,
`"style":{"color":{"background":"var:preset|color|accent-2"}}` is not: it emits
a hardcoded value and loses the editor's color picker selection.

## Layout types

`layout` on a `core/group` decides everything about how children arrange.

```json
{"layout":{"type":"constrained"}}
{"layout":{"type":"constrained","contentSize":"640px"}}
{"layout":{"type":"default"}}
{"layout":{"type":"flex"}}
{"layout":{"type":"flex","orientation":"vertical"}}
{"layout":{"type":"flex","justifyContent":"space-between","flexWrap":"nowrap"}}
{"layout":{"type":"grid","minimumColumnWidth":"16rem"}}
{"layout":{"type":"grid","minimumColumnWidth":null,"columnCount":3}}
```

| Type | Behaviour |
| --- | --- |
| `constrained` | Children are centered to `contentSize`, `alignwide` and `alignfull` children escape it |
| `default` | Plain flow, no width constraint |
| `flex` | Row by default, `orientation: vertical` for a stack, wraps unless `flexWrap: nowrap` |
| `grid` | Auto fitting tracks with `minimumColumnWidth`, or fixed tracks with `columnCount` |

`minimumColumnWidth` gives a responsive grid with no media query. Prefer it over
`columnCount` unless the design demands an exact track count at every width.

## Semantic tags

`core/group` accepts `tagName`. Use it. The default `div` is only right for a
wrapper with no meaning.

```html
<!-- wp:group {"tagName":"section","layout":{"type":"constrained"}} -->
<section class="wp-block-group">…</section>
<!-- /wp:group -->
```

`header`, `main`, `footer`, `section`, `article`, `aside`, `nav`. In a template,
the content region is `main`, and `core/template-part` takes `tagName` too.

## Common composites

### Section wrapper

The shape almost every band takes: full width background, constrained content.

```html
<!-- wp:group {"tagName":"section","align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"backgroundColor":"base-2","layout":{"type":"constrained"}} -->
<section class="wp-block-group alignfull has-base-2-background-color has-background" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
	<!-- content -->
</section>
<!-- /wp:group -->
```

### Card grid

```html
<!-- wp:group {"align":"wide","style":{"spacing":{"blockGap":"var:preset|spacing|40"}},"layout":{"type":"grid","minimumColumnWidth":"18rem"}} -->
<div class="wp-block-group alignwide">
	<!-- wp:group {"style":{"spacing":{"padding":"var:preset|spacing|40"},"border":{"radius":"8px"}},"backgroundColor":"base","layout":{"type":"flex","orientation":"vertical"}} -->
	<div class="wp-block-group has-base-background-color has-background" style="border-radius:8px;padding:var(--wp--preset--spacing--40)">
		<!-- card contents -->
	</div>
	<!-- /wp:group -->
</div>
<!-- /wp:group -->
```

### Two unequal columns

```html
<!-- wp:columns {"align":"wide","style":{"spacing":{"blockGap":{"top":"var:preset|spacing|50","left":"var:preset|spacing|50"}}}} -->
<div class="wp-block-columns alignwide">
	<!-- wp:column {"width":"56%"} -->
	<div class="wp-block-column" style="flex-basis:56%"><!-- … --></div>
	<!-- /wp:column -->

	<!-- wp:column {"verticalAlignment":"center"} -->
	<div class="wp-block-column is-vertically-aligned-center"><!-- … --></div>
	<!-- /wp:column -->
</div>
<!-- /wp:columns -->
```

### Buttons

```html
<!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
<div class="wp-block-buttons">
	<!-- wp:button -->
	<div class="wp-block-button"><a class="wp-block-button__link wp-element-button">Learn more</a></div>
	<!-- /wp:button -->
</div>
<!-- /wp:buttons -->
```

`wp-element-button` on the `a` is not optional. It is what connects the button
to the `elements.button` styles in `theme.json`.

### Heading and paragraph

```html
<!-- wp:heading {"textAlign":"center","fontSize":"xx-large"} -->
<h2 class="wp-block-heading has-text-align-center has-xx-large-font-size">Heading</h2>
<!-- /wp:heading -->

<!-- wp:paragraph {"align":"center","fontSize":"medium"} -->
<p class="has-text-align-center has-medium-font-size">Body copy.</p>
<!-- /wp:paragraph -->
```

`core/heading` carries `wp-block-heading`, `core/paragraph` carries no block
class of its own. Heading alignment is `textAlign`, paragraph alignment is
`align`. These inconsistencies are real, copy them exactly.

## Blocks whose markup is easy to get wrong

`core/cover`, `core/navigation`, `core/query`, `core/accordion` and `core/tabs`
save a fair amount of internal structure. Do not reconstruct that from memory.
Get known good markup one of two ways:

1. Build it once in the editor, select the block, copy, and paste into the file.
2. Read a shipped example. `wp-content/themes/twentytwentyfive/patterns/` has 90
   or so files covering nearly every core block, and it ships with WordPress.

Then adapt attributes. That is much safer than inventing inner HTML.

## In a pattern file

Pattern files are PHP, so text and URLs are translated and escaped:

```php
<!-- wp:heading -->
<h2 class="wp-block-heading"><?php esc_html_e( 'Our services', 'theme-slug' ); ?></h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p><?php esc_html_e( 'What we do and who we do it for.', 'theme-slug' ); ?></p>
<!-- /wp:paragraph -->
```

The text domain is the theme slug from `style.css`. Use `esc_html_e()` for
visible text, `esc_attr_e()` for attributes, `esc_url()` for URLs.

Imagery is the exception to "write it into the file". Design images are media
library uploads, not theme assets, so a pattern either ships without the image
or resolves the attachment at render time. See
[images-and-svg.md](./images-and-svg.md).
