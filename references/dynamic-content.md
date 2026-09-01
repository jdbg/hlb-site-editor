# Dynamic content

**Rule 4: never hook a filter to a CSS class to produce content.**

The anti pattern, in full:

```php
// Never do this.
add_filter( 'render_block', function ( $content, $block ) {
	if ( str_contains( $block['attrs']['className'] ?? '', 'team-grid' ) ) {
		return $content . build_team_cards();
	}
	return $content;
}, 10, 2 );
```

It is invisible in the editor, so the client sees an empty group and deletes it.
It breaks the moment someone renames the class. It cannot be previewed, styled
or reordered. And every case it is reached for already has a first class answer.

## What to use instead

| The content is | Use |
| --- | --- |
| A field value on the current post or term | Block Bindings |
| A list of posts, pages or a custom post type | `core/query` with `core/post-template` |
| A list of terms | `core/terms-query` with `core/term-template` |
| A preconfigured block the client inserts | A block variation |
| A section reused across templates | `core/template-part` |
| A layout the client inserts and then edits | A pattern |
| A layout inserted in many places that must stay identical | A synced pattern, `core/block` |
| A pattern with a few editable slots | Pattern overrides |

## Block Bindings

Bind a core block's attribute to a value resolved at render time. The block stays
a normal core block: still selectable, still styleable, still in the editor.

```html
<!-- wp:paragraph {"metadata":{"bindings":{"content":{"source":"core/post-meta","args":{"key":"acme_role"}}}}} -->
<p></p>
<!-- /wp:paragraph -->
```

The inner HTML is left empty on purpose. The source fills it.

### Core sources

| Source | Since | Args | Returns |
| --- | --- | --- | --- |
| `core/post-meta` | 6.5 | `key` | A registered meta value on the current post |
| `core/pattern-overrides` | 6.5 | none | The per instance value of a pattern slot |
| `core/post-data` | 6.9 | `field`: `date`, `modified`, `link` | Data about the current post |
| `core/term-data` | 6.9 | `field`: `id`, `name`, `link`, `slug`, `description`, `parent`, `count` | Data about the current term |

### Which attributes can be bound

Only attributes marked with a content role:

| Block | Bindable |
| --- | --- |
| `core/paragraph` | `content` |
| `core/heading` | `content` |
| `core/image` | `url`, `alt`, `caption`, `title`, `href`, `id` |
| `core/button` | `url`, `text`, `title`, `linkTarget`, `rel` |
| `core/quote` | `value`, `citation` |
| `core/cover` | `url` |
| `core/icon` | `icon` |

Binding to anything else silently does nothing.

### `core/post-meta` needs registered meta

The meta field must be registered with `show_in_rest` true and a single value,
otherwise the editor cannot read it. Registration belongs in the project's
`<prefix>-core` mu-plugin, never in the theme.

```php
add_action( 'init', function (): void {
	register_post_meta( 'acme_person', 'acme_role', [
		'type'              => 'string',
		'single'            => true,
		'show_in_rest'      => true,
		'sanitize_callback' => 'sanitize_text_field',
		'auth_callback'     => fn (): bool => current_user_can( 'edit_posts' ),
	] );
} );
```

With that in place the bound paragraph is editable directly in the editor, and
typing into it writes the meta field.

### A custom source

When the value is computed rather than stored, register a source. Still in the
core mu-plugin, still on `init`.

```php
add_action( 'init', function (): void {
	register_block_bindings_source( 'acme/reading-time', [
		'label'              => __( 'Reading time', 'acme-core' ),
		'get_value_callback' => function ( array $args, $block ): ?string {
			$post_id = $block->context['postId'] ?? null;
			if ( ! $post_id ) {
				return null;
			}
			$words = str_word_count( wp_strip_all_tags( get_post_field( 'post_content', $post_id ) ) );
			return sprintf(
				/* translators: %d: minutes */
				__( '%d min read', 'acme-core' ),
				max( 1, (int) ceil( $words / 200 ) )
			);
		},
		'uses_context'       => [ 'postId' ],
	] );
} );
```

A source returns a value. It never returns markup. Anything that wants to return
markup is a layout problem being solved in the wrong place.

## Block variations

A variation is a core block with preset attributes, a name and an icon, showing
up in the inserter as its own entry. This is the answer to "the client needs to
be able to drop in a styled testimonial card".

```js
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation( 'core/group', {
	name: 'acme/testimonial',
	title: 'Testimonial',
	description: 'A quote with an author line.',
	icon: 'format-quote',
	attributes: {
		className: 'is-style-card',
		layout: { type: 'flex', orientation: 'vertical' },
	},
	innerBlocks: [
		[ 'core/quote', {} ],
		[ 'core/paragraph', { fontSize: 'small' } ],
	],
	scope: [ 'inserter' ],
	isActive: ( attrs ) => attrs.className?.includes( 'is-style-card' ),
} );
```

Notes:

- `innerBlocks` is what makes a variation a structure rather than just a preset.
- `isActive` is required, or the editor cannot tell the variation apart from the
  plain block and the sidebar shows the wrong name.
- The output is still `core/group`. Nothing custom is registered, nothing breaks
  if the JS fails to load.
- Variations are also how you preconfigure `core/query` for a specific post
  type, or `core/embed` for a provider.

## Query Loop

Any repeating list of posts is `core/query`, never a filter and never a
hand built list.

```html
<!-- wp:query {"queryId":0,"query":{"postType":"acme_person","perPage":9,"offset":0,"order":"asc","orderBy":"title","inherit":false},"layout":{"type":"default"}} -->
<div class="wp-block-query">
	<!-- wp:post-template {"style":{"spacing":{"blockGap":"var:preset|spacing|40"}},"layout":{"type":"grid","minimumColumnWidth":"18rem"}} -->
		<!-- wp:post-featured-image {"aspectRatio":"1","isLink":true} /-->
		<!-- wp:post-title {"level":3,"isLink":true,"fontSize":"medium"} /-->
		<!-- wp:paragraph {"metadata":{"bindings":{"content":{"source":"core/post-meta","args":{"key":"acme_role"}}}},"fontSize":"small"} -->
		<p class="has-small-font-size"></p>
		<!-- /wp:paragraph -->
	<!-- /wp:post-template -->
</div>
<!-- /wp:query -->
```

`inherit: true` inside a template makes the loop follow the main query, which is
what an archive template wants. `inherit: false` with an explicit `postType` is
what a curated section on a page wants.

Inside `core/post-template`, bindings resolve against each post in turn, so a
custom field per card needs no PHP at all.

## Pattern overrides

For a pattern the client inserts repeatedly where only some slots change, mark
those slots instead of building a variation for each case.

```html
<!-- wp:heading {"metadata":{"name":"Card title","bindings":{"content":{"source":"core/pattern-overrides"}}}} -->
<h3 class="wp-block-heading">Default title</h3>
<!-- /wp:heading -->
```

Inserted as a synced pattern, each instance can change that heading while the
rest stays locked to the original.

## When a filter is legitimate

`render_block` is not banned outright. It is banned as a content injection
mechanism keyed off a class. It remains correct for cross cutting concerns that
have no block level expression, such as adding a `loading` attribute to images
site wide. Even then, target the specific block, `render_block_core/image`, use
`WP_HTML_Tag_Processor` rather than string surgery, and never make a design's
content depend on it.
