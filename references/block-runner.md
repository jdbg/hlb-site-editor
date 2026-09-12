# block-runner

**Optional.** [block-runner](https://www.npmjs.com/package/block-runner)
(`github.com/humanmade/block-runner`) is a third party npm CLI that assembles
or converts a block tree deterministically and validates it against headless
Gutenberg. It is not a dependency of this skill package. Install it per
project with `npm install --save-dev block-runner` and pin an exact version —
it is pre-1.0, and commands and flags can move between releases. Everything
below was checked against `block-runner@0.9.4`.

## Where it fits

| Workflow step | Command | What it replaces or speeds up |
| --- | --- | --- |
| 5. Write the markup | `assemble` | Submit the block tree already decomposed in step 3 as an intent tree; block-runner serializes and validates the result instead of hand typing block comments |
| 7. Verify | `validate` | A fast automated pass before the manual editor check |
| 2. Check `theme.json` first | `context` | Pulls existing theme token presets from a live, WP-CLI-reachable site into a Wesper manifest, for feeding `--context` into `assemble`/`validate` |

`assemble <jsonOrStdin>` takes an intent tree as its argument, not markup.
The intent tree's shape is block-runner's own contract — read it from
`block-runner assemble --help` or the package's own docs rather than from
this file, so this doc does not drift out of sync with a pre-1.0 schema.

```
npx block-runner assemble intent.json --json
```

`validate <globOrStdin>` checks existing block markup against headless
Gutenberg:

```
npx block-runner validate "parts/*.html" --json
```

## The pattern-file gotcha

`validate` only understands block markup, not PHP. Template parts
(`parts/*.html`) and templates (`templates/*.html`) are plain block markup
with no PHP and no translation (see [output-targets.md](./output-targets.md)),
so `validate` runs on those files exactly as they are.

Patterns (`patterns/*.php`) are different: per
[block-markup.md](./block-markup.md#in-a-pattern-file), visible text and URLs
get `esc_html_e()` / `esc_attr_e()` / `esc_url()` wrapping as the *last*
authoring step, after the raw block markup is written. `validate` has to run
on that raw markup before the translation wrapping is added — never on the
finished `.php` file.

This was checked against a real fixture, not assumed. Validating
`wp-content/themes/twentytwentyfive/patterns/banner-about-book.php` directly
reports 332 of 1574 blocks as invalid, all for the same reason:

```
Expected token of type `Comment` (…), instead saw `StartTag`
(…tagName: 'php'…)
```

block-runner's HTML parser reads `<?php esc_html_e( 'About the book', … ); ?>`
as a broken opening tag called `php`. Validating the same theme's
`parts/*.html` files (no PHP, nothing to misparse) reports 7 of 7 blocks
valid. Same tool, same theme — the difference is entirely the PHP wrapping.

## Never

| Do not | Because |
| --- | --- |
| Treat a passing `validate` as proof the block exists on the client's WP version | It validates against block-runner's pinned Gutenberg, not the target site. Still do [verification.md](./verification.md) check 3 |
| Treat a passing `validate` as a substitute for inserting the pattern in the real editor | Still the only check that catches genuinely invalid inner HTML the way WordPress itself parses it |
| Use `block-runner author` or `block-runner plugin` to generate a registered block | Generates a custom block — collides with rule 1 (core blocks only) unless a custom block has already been explicitly approved, see `wp-block-development` |
| Run `block-runner skill --install` in a project that uses hlb-site-editor | It installs a second, competing agent skill centered on that same custom-block generation. Use block-runner as a CLI dependency only |
| Run `validate` against a finished `patterns/*.php` file | The interleaved PHP is misparsed as broken HTML tags, producing false invalid-block reports |
