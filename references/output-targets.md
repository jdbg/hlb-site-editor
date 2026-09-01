# Where the output goes

Four possible homes for the markup. Picking the wrong one is the difference
between a client who can edit their site and one who files a ticket.

| The design is | Goes in | Editable by the client |
| --- | --- | --- |
| A reusable section they will insert and edit | A pattern, `patterns/*.php` | Yes, each copy independently |
| A region repeated on every page: header, footer | A template part, `parts/*.html` | Yes, in one place for the whole site |
| The frame of a page type: single, archive, 404 | A template, `templates/*.html` | Yes, per template |
| The content of one specific page | Page content in the editor, usually built from patterns | Yes |

Ask "does this repeat, and should editing one copy change the others?" A header
must change everywhere at once, so it is a template part. A hero section that
each landing page tweaks is a pattern.

## Patterns

`wp-content/themes/<theme-slug>/patterns/<slug>.php`. Auto registered by
WordPress, no PHP registration call needed.

```php
<?php
/**
 * Title: Services grid
 * Slug: acme/services-grid
 * Categories: acme-sections, featured
 * Description: Three service cards with an icon, heading and short copy.
 * Keywords: services, features, grid
 * Viewport Width: 1400
 *
 * @package Acme
 */

?>
<!-- wp:group {"tagName":"section","align":"full","layout":{"type":"constrained"}} -->
<section class="wp-block-group alignfull">
	<!-- … -->
</section>
<!-- /wp:group -->
```

Header fields, exactly as core reads them:

| Header | Purpose |
| --- | --- |
| `Title` | Required. Shown in the inserter |
| `Slug` | Required. `<theme-slug>/<pattern-slug>` |
| `Description` | Screen reader text and the inserter tooltip |
| `Categories` | Comma separated category slugs |
| `Keywords` | Extra search terms |
| `Viewport Width` | Preview width in the inserter, 1400 for full width sections |
| `Block Types` | Blocks this pattern can be used as a transform target for |
| `Post Types` | Restrict the pattern to certain post types |
| `Template Types` | Offer the pattern as a starting point for those templates |
| `Inserter` | `no` hides it from the inserter, for patterns only used inside templates |

A custom category has to be registered before patterns can use it. That is
theme presentation, so it stays in the theme:

```php
add_action( 'init', function (): void {
	register_block_pattern_category( 'acme-sections', [
		'label'       => __( 'Acme sections', 'acme' ),
		'description' => __( 'Full width page sections.', 'acme' ),
	] );
} );
```

Core registers these categories already, and one of them is usually the right
answer: `about`, `audio`, `banner`, `buttons`, `call-to-action`, `columns`,
`contact`, `featured`, `footer`, `gallery`, `header`, `media`, `navigation`,
`portfolio`, `posts`, `query`, `services`, `team`, `testimonials`, `text`,
`videos`. Prefer one of these over inventing a category with one pattern in it,
and check the slug against
`wp-includes/block-patterns.php` before using it: a category that was never
registered makes the pattern vanish from the inserter with no error.

Everything visible in a pattern file is translated and escaped, see
[block-markup.md](./block-markup.md).

## Template parts

`wp-content/themes/<theme-slug>/parts/<name>.html`. Plain block markup, no PHP,
no headers, so no translation. Text inside a part is site content the client
edits, not theme strings.

Registered in `theme.json` so it gets a proper name and area in the editor:

```json
{
  "templateParts": [
    { "name": "header", "title": "Header", "area": "header" },
    { "name": "footer", "title": "Footer", "area": "footer" },
    { "name": "sidebar", "title": "Sidebar", "area": "uncategorized" }
  ]
}
```

Areas are `header`, `footer` and `uncategorized`. The area drives the editor's
grouping and the default `tagName`.

Used from a template:

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->
```

## Templates

`wp-content/themes/<theme-slug>/templates/<name>.html`. The file name is the
template hierarchy slot.

| File | Used for |
| --- | --- |
| `index.html` | Required fallback |
| `front-page.html` | The site front page |
| `home.html` | The posts index |
| `page.html` | Pages |
| `single.html` | Single posts |
| `archive.html` | Archives |
| `search.html` | Search results |
| `404.html` | Not found |
| `singular.html` | Both single and page, if you do not need them apart |
| `page-<slug>.html` | One specific page |
| `single-<post-type>.html` | One post type |
| `archive-<post-type>.html` | One post type's archive |

The standard skeleton:

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
	<!-- wp:post-content {"layout":{"type":"constrained"}} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
```

A template with a name that is not in the hierarchy needs registering in
`theme.json` so the client can assign it:

```json
{
  "customTemplates": [
    { "name": "landing-page", "title": "Landing page", "postTypes": ["page"] }
  ]
}
```

## Laying out a page

When the request is "lay out this page from the design", the answer is usually
not a template. It is:

1. Each section of the design becomes a pattern in `patterns/`.
2. The page uses the ordinary `page.html` template.
3. The patterns get inserted into that page's content.

That keeps every section editable and reusable, and it keeps the page out of the
theme where a client edit would be overwritten on the next deploy.

Two ways to get the patterns onto the page:

- Hand the client the pattern names and let them insert them. Preferred, it is
  their page.
- Insert them yourself with WP-CLI when the page is being built as part of the
  handover.

If the design is genuinely a one off page frame rather than content, for example
a coming soon screen, then it is `page-<slug>.html` plus a pattern, and the
template references the pattern with `<!-- wp:pattern {"slug":"acme/coming-soon"} /-->`.

## Never

| Do not | Do |
| --- | --- |
| Put page content in a template | Patterns inserted into the page |
| Duplicate a header into every template | One template part |
| Create a template per page | `page.html`, plus patterns, plus `customTemplates` only where the frame really differs |
| Register patterns with `register_block_pattern()` in a block theme | Drop the file in `patterns/` |
| Translate strings in a `parts/*.html` file | Those are content, not theme strings |
