# Images and SVG

**Rule 2: an SVG reaches the page through `core/image` or `core/icon`. Never as
inline `<svg>` markup.**

An inline `<svg>` inside a `core/html` block, or hand written into a pattern, is
not selectable in the editor, carries no block supports, cannot be recolored
from the sidebar and cannot be swapped by the client. It is the same problem as
a custom CSS class, in a different costume.

## Media belongs in the media library

Every image, SVG, logo and illustration from the design is **uploaded**. None of
them are committed into the theme or a plugin as a file under `assets/`.

An asset in the repo cannot be swapped by the client, cannot be replaced without
a deploy, has no alt text of its own, gets no responsive sizes generated, and
puts binaries in a repo that CI has to carry. The same file in the media library
is editable, replaceable, reusable across the site, and already has everything
WordPress does with attachments.

```
hlbake wp <project> media import ./design-exports/hero.webp \
	--title="Hero" --alt="Team at the studio" --post_name=acme-hero --porcelain
```

`--porcelain` returns the attachment ID. `--post_name` fixes the slug, which is
what makes an asset resolvable across local, staging and production where the
IDs and URLs differ.

Set `--alt` at import time. Alt text lives on the attachment, so it is written
once and every block that uses the image inherits it.

The only things that stay as files in the repo are covered under
[custom icons](#custom-icons-from-the-design) below, plus font files when the
project does not use the Font Library. Design imagery never is.

## Which block

| The SVG is | Block |
| --- | --- |
| A logo, illustration, or any uploaded artwork | `core/image` |
| An interface icon: arrow, chevron, check, social glyph, feature bullet | `core/icon` |
| The site's own logo in a header or footer | `core/site-logo` |
| A row of social platform glyphs | `core/social-links` |
| Decoration behind content | `core/cover`, or a `background` on a group |

## `core/image` with an SVG

Upload the SVG to the media library and reference it like any other image.

```html
<!-- wp:image {"id":42,"width":"180px","sizeSlug":"full","linkDestination":"none"} -->
<figure class="wp-block-image size-full is-resized">
	<img src="https://example.test/wp-content/uploads/2026/01/logo.svg" alt="Acme" class="wp-image-42" style="width:180px"/>
</figure>
<!-- /wp:image -->
```

### Referencing an upload from a pattern

A pattern file cannot hardcode an upload URL, because the domain and the
attachment ID differ between local, staging and production. Two correct answers,
in order of preference.

**Ship the pattern without the image.** The client inserts it and picks their
own. This is right for most patterns, and it is what an inserter preview is for.

**Resolve the attachment by slug at render time.** For a specific asset the
design genuinely requires, such as a logo:

```php
<?php
$acme_logo = get_page_by_path( 'acme-logo', OBJECT, 'attachment' );
$acme_id   = $acme_logo ? (int) $acme_logo->ID : 0;
$acme_src  = $acme_id ? wp_get_attachment_image_url( $acme_id, 'full' ) : '';
$acme_alt  = $acme_id ? get_post_meta( $acme_id, '_wp_attachment_image_alt', true ) : '';

if ( $acme_src ) :
	?>
<!-- wp:image {"id":<?php echo $acme_id; ?>,"width":"180px","sizeSlug":"full"} -->
<figure class="wp-block-image size-full is-resized">
	<img src="<?php echo esc_url( $acme_src ); ?>" alt="<?php echo esc_attr( $acme_alt ); ?>" class="wp-image-<?php echo $acme_id; ?>" style="width:180px"/>
</figure>
<!-- /wp:image -->
	<?php
endif;
```

The slug is the one from `--post_name` at import. The guard matters: a pattern
that emits `wp-image-0` and an empty `src` produces a broken block in the
inserter on any environment where the media was not imported.

WordPress does not accept SVG uploads out of the box, and enabling them with a
bare `upload_mimes` filter is a stored XSS hole. The house baseline includes the
Safe SVG plugin, which sanitizes on upload. If SVG uploads are refused on the
target site, that plugin is missing. Do not work around it with a filter.

Sizing an SVG is a block support, `width` or `style.dimensions`, never a CSS
rule.

## `core/icon`

Available from WordPress 7.0. The icon registry that backs it landed in 7.1. It
is a dynamic block, so the pattern markup is only the self closing comment:

```html
<!-- wp:icon {"icon":"core/arrow-right","style":{"dimensions":{"width":"24px"}}} /-->
```

Attributes:

| Attribute | Purpose |
| --- | --- |
| `icon` | Namespaced icon name, `collection/name`, for example `core/info` |
| `flipHorizontal`, `flipVertical` | Mirror the glyph |
| `rotation` | Degrees |
| `style.dimensions.width` | Size, this is how you size an icon |
| `textColor` or `style.color.text` | The glyph color, the SVG fills with `currentColor` |
| `backgroundColor`, border, padding | Turns the icon into a chip or a circle without any CSS |

An icon in a circular accent colored badge, entirely in block attributes:

```html
<!-- wp:icon {"icon":"core/check","textColor":"base","backgroundColor":"accent-1","style":{"dimensions":{"width":"20px"},"spacing":{"padding":"var:preset|spacing|20"},"border":{"radius":"100px"}}} /-->
```

### Custom icons from the design

When the designer supplies icons that are not in the core collection, register
them rather than pasting SVG into the page. Registration belongs in the
project's `<prefix>-core` mu-plugin, not the theme, so the icons survive a theme
change.

**This is the one exception to the uploads rule, and it is deliberate.** The
Icons API takes either SVG markup or a file path. It has no attachment backed
source, so a registered icon is necessarily a file in the plugin. That is
acceptable because an icon is interface furniture rather than content: it is
part of the design system, the client is not expected to replace it, and putting
it in the registry is what makes it appear in the icon picker.

If a glyph really is content the client should be able to swap, it is not an
icon. Upload it and use `core/image`.

```php
add_action( 'init', function (): void {
	wp_register_icon_collection( 'acme', [
		'label' => __( 'Acme', 'acme-core' ),
	] );

	wp_register_icon( 'acme/leaf', [
		'label'     => __( 'Leaf', 'acme-core' ),
		'file_path' => plugin_dir_path( __FILE__ ) . 'icons/leaf.svg',
	] );
} );
```

Then in markup: `<!-- wp:icon {"icon":"acme/leaf"} /-->`. The icon now appears
in the block's picker, so the client can use it anywhere.

`wp_register_icon()` takes either `file_path` or `content`. Prefer `file_path`
so the SVG stays a file that can be reviewed and diffed. The `core` collection
is reserved, always register under a project collection.

Registering icons needs 7.1. The block itself works from 7.0 with the core
collection. Below 7.0 the fallback is `core/image` with an uploaded SVG. Note
the substitution when handing the work over, because icons then lose the
recolor from `currentColor` behaviour.

## Raster images

```html
<!-- wp:image {"aspectRatio":"16/9","scale":"cover","sizeSlug":"large"} -->
<figure class="wp-block-image size-large">
	<img src="…" alt="…" style="aspect-ratio:16/9;object-fit:cover"/>
</figure>
<!-- /wp:image -->
```

- `aspectRatio` plus `scale` is how a design's fixed image box is reproduced.
  Never a CSS `height` on a class.
- `sizeSlug` picks the registered image size. `full` for artwork that must not
  be resampled, such as an SVG or a logo.
- Set `alt` on every image. Decorative images take `alt=""`, not a missing
  attribute.
- `linkDestination` when the image links. A `core/image` wrapped in a hand
  written `<a>` is invalid markup.

## Images with text on them

`core/cover`, not an image with an absolutely positioned overlay.

Cover gives the overlay color, the dim ratio, the focal point, a minimum height
and the `isDark` handling, all from the sidebar. Its saved markup is intricate,
so get it by inserting the block once and copying rather than writing it from
memory. See the shipped examples in
`wp-content/themes/twentytwentyfive/patterns/`.

For a decorative background behind a normal section, `core/group` supports a
background image directly:

```html
<!-- wp:group {"align":"full","style":{"background":{"backgroundImage":{"url":"…","id":42,"source":"file"},"backgroundSize":"cover","backgroundPosition":"50% 50%"}},"layout":{"type":"constrained"}} -->
```

Use cover when text sits on the image and needs contrast handling. Use the group
background when the image is decoration.

## Never

| Do not | Do |
| --- | --- |
| `<svg>` inline in a pattern or an HTML block | `core/icon`, or `core/image` with an uploaded SVG |
| An icon font | `core/icon` |
| A CSS `background-image` on a custom class | Group background support, or `core/cover` |
| `upload_mimes` to allow SVG | Safe SVG, which sanitizes |
| A hand written `<a>` around an image | `linkDestination` and `href` on `core/image` |
| A CSS rule to size an SVG | `width`, or `style.dimensions.width` |
| Commit design imagery to `assets/` in the theme or a plugin | Import it to the media library |
| Hardcode an upload URL in a pattern | Resolve the attachment by slug, or ship the pattern without the image |
| Alt text written per block | `--alt` on the attachment, inherited everywhere |
