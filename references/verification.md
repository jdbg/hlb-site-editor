# Verification

Hand written block markup fails silently. The page renders, the editor shows a
warning, and nobody notices until the client tries to edit the section. Verify
before handing anything over.

## What can go wrong

| Symptom | Cause |
| --- | --- |
| "This block contains unexpected or invalid content" | Attributes and saved HTML disagree, see the mirroring rule in [block-markup.md](./block-markup.md) |
| The block shows as a grey placeholder | A block name that does not exist on this install |
| The pattern is not in the inserter | Missing `Title` or `Slug` header, or a category that was never registered |
| The section has no styling | A class was invented instead of using block supports |
| The layout is right but colors are missing | A preset slug that does not exist in `theme.json` |
| Everything stacks in one column | `layout` missing on the group, so it fell back to flow |
| Content is missing entirely | A binding on an attribute with no content role, or a source that returned null |
| An image is broken on staging but fine locally | An upload URL or attachment ID hardcoded into a pattern |

## The checks, cheapest first

### 1. Parse the markup

Confirms the block comments are well formed and every block name resolves. Runs
without a browser.

```
hlbake wp <project> eval 'print_r( array_column( parse_blocks( file_get_contents( "wp-content/themes/acme/patterns/services-grid.php" ) ), "blockName" ) );'
```

A `core/missing` or an unexpected `null` in that list means a broken comment or
a block that does not exist here. This does not check inner HTML.

Note that `file_get_contents` on a pattern file returns the PHP source
unexecuted, so `esc_html_e( … )` calls appear as literal text. That is fine, the
block comments are what is being parsed.

This check, and check 2 below, can be automated with `block-runner validate`
when it is installed in the project. It only understands block markup, not
PHP, so it has to run on the raw markup before pattern-file translation
wrapping is added, or directly on a template part or template. See
[block-runner.md](./block-runner.md) for the exact command and that caveat.

### 2. Confirm the pattern registered

```
hlbake wp <project> eval 'print_r( array_keys( WP_Block_Patterns_Registry::get_instance()->get_all_registered() ) );'
```

The slug should be in the list. If it is not, the file headers are wrong.

### 3. Confirm the blocks exist on this install

```
ls wp-includes/blocks/ | grep -x icon
```

Newer blocks are the usual culprit. See the availability table in
[core-blocks.md](./core-blocks.md).

### 4. Insert it in the editor

The only check that catches invalid inner HTML. Insert the pattern, then:

- No yellow validation warning on any block.
- Every block is selectable and shows its own sidebar controls.
- Every styled property appears in the sidebar, not just on screen. A background
  that renders but shows no color selected means a class is doing the work,
  which breaks rule 3.
- The list view shows a sensible tree with semantic wrappers, no unexplained
  nested groups.

### 5. Compare against the design

Frontend, at a narrow width and a wide one. Grid and flex layouts should reflow
without a media query. Check the type scale and the spacing against the presets,
not against the mockup's pixels.

For an automated pass, the `wp-playwright` skill drives a Playground instance and
can screenshot the rendered page.

### 6. Lint

```
npm run lint
```

Pattern files are PHP and are covered by PHPCS, which will catch an unescaped
string or a missing text domain.

## Getting known good markup

When a block's saved HTML is uncertain, do not guess. Two reliable sources:

1. **Build it once in the editor.** Insert the block, configure it, select the
   parent, copy. The clipboard holds exactly the markup WordPress considers
   valid. Paste it into the file and adapt the attributes.
2. **Read a shipped pattern.** `wp-content/themes/twentytwentyfive/patterns/`
   has around ninety files covering nearly every core block, and it ships with
   WordPress.

This matters most for `core/cover`, `core/navigation`, `core/query`,
`core/accordion` and `core/tabs`, which all save internal structure that is easy
to get subtly wrong.

## Before handing over

- [ ] Parses, registers, inserts with no validation warning
- [ ] Every visual property is visible in the block sidebar
- [ ] No stylesheet was added for this design
- [ ] No `render_block` filter was added for this design
- [ ] No `core/html`, no inline `<svg>`, no custom block
- [ ] Every image is a media library upload, nothing committed to the theme
- [ ] Alt text is set on the attachments, not written per block
- [ ] Presets added to `theme.json` are called out to the designer
- [ ] Anything the design asked for that core cannot do is stated plainly rather
      than worked around
