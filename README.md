# HLB Site Editor skill

A Claude skill for turning a design into WordPress Site Editor block markup:
patterns, template parts, templates and page layouts.

It exists because the default failure mode of "build this design in WordPress"
is a custom block wrapping hand written HTML, styled by a stylesheet, populated
by a filter. That renders correctly and is unusable by the client who has to
maintain it.

## The four rules

1. **Core blocks only.** No custom blocks, no `core/html`, no raw markup.
2. **SVG through `core/image` or `core/icon`.** Never inline `<svg>`.
3. **Never style with a custom CSS class.** Block supports, `theme.json`, or a
   registered style variation the client can see in the editor.
4. **Never hook a filter to a CSS class to produce content.** Block Bindings,
   block variations, Query Loop.

## Layout

```
SKILL.md                          the rules, the workflow, the decision flow
references/
├── reading-designs.md            Figma, screenshot or PDF to structure and tokens
├── core-blocks.md                design element to core block, availability by WP version
├── block-markup.md               serialization and the attribute to HTML mirroring rule
├── styling.md                    rule 3 in full
├── images-and-svg.md             rule 2 in full
├── dynamic-content.md            rule 4 in full
├── output-targets.md             pattern vs template part vs template
├── verification.md               checking the markup actually works
└── block-runner.md               optional CLI assembly/validation via block-runner
```

## Install

Pull the skill straight into the Claude skills directory:

```
npx giget@latest gh:jdbg/hlb-site-editor ~/.claude/skills/hlb-site-editor --force
```

For a single project instead of the whole machine, target the project skills
directory:

```
npx giget@latest gh:jdbg/hlb-site-editor .claude/skills/hlb-site-editor --force
```

`SKILL.md` has to sit at the root of `hlb-site-editor/` or Claude will not find
the skill, so keep the destination directory name as written. Re-running the
same command updates an existing install; `--force` is what lets it write over
the directory that is already there.

## Local development

Working on the skill itself, clone it and symlink the checkout rather than
copying it, so edits are live:

```
git clone git@github.com:jdbg/hlb-site-editor.git
cd hlb-site-editor
ln -s "$PWD" ~/.claude/skills/hlb-site-editor
```

To check that [block-runner.md](./references/block-runner.md)'s claims still
hold against the pinned block-runner version:

```
npm install
npm run blocks:demo
```

Re-run this after bumping the pinned `block-runner` version in
`package.json`.

## Scope

This skill covers design to blocks. It does not cover theme scaffolding, build
tooling or deploy, which belong to `project-setup`, nor `theme.json` generation
from a Figma design system, which belongs to `figma-to-wp-theme-json`.

## Facts and how they were checked

Block availability, binding sources, bindable attributes and the Icons API were
read from WordPress installs on disk (6.8.3, 6.9.4 and 7.1) rather than from
release notes. Markup idioms were taken from the patterns shipped with Twenty
Twenty-Four and Twenty Twenty-Five. Re-check the version gates in
`core-blocks.md` when a new WordPress ships.
