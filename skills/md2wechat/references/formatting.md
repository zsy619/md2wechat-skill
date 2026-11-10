# Formatting, theme, layout, and brand

Read this file when the user asks to format an article without a chosen theme or modules, or when theme, layout-module, or Brand Profile rules are needed.

## Formatting Protocol

When the user asks to format an article and has not chosen a theme or modules:

1. Read the article and optional Brand Profile.
2. Use discovery output as facts.
3. Choose a compatible theme and a small set of modules from the article's content goal.
4. Keep the source Markdown read-only.
5. Create a temporary formatted Markdown artifact, for example `/tmp/md2wechat-format/<run-id>/article.formatted.md`.
6. Insert only layout modules whose required fields can be filled correctly.
7. Run `md2wechat layout validate --file <formatted.md> --json`.
8. Pass the formatted Markdown artifact to `convert`.

Saving generated Markdown next to the source file requires explicit user confirmation and must not overwrite the source.

## Theme Selection

- Read `type` and `selectable` from `themes list --json`.
- API mode can use only `type: api` and `selectable: true` themes.
- AI mode can use only `type: ai` and `selectable: true` themes.
- Do not use collection descriptors such as non-selectable theme groups as concrete themes.
- If Brand Profile names a theme, verify it through CLI discovery before using it.
- If a requested theme is invalid or mode-incompatible, stop that path and choose a valid theme or ask the user.

## Layout Modules

Advanced layout modules render only in API mode. AI mode (`--mode ai`) does not parse `:::module` syntax, so advanced layout cards will not render there.

Use this decision frame:

- `attention`: help readers decide whether the article is worth reading.
- `readability`: make mobile reading easier.
- `memorability`: make one judgment, quote, metric, or brand anchor stick.
- `conversion`: help readers save, follow, inquire, share, or buy.

Use CLI discovery as the source of truth for layout syntax instead of memorizing or guessing `body_format` values:

- Inspect the opener, body schema, canonical executable example, and structurally distinct variants with `layout show <name> --json`. Reuse the canonical witness.
- Use `layout render` for structured fields and `--body-file` (or `--body-file -` for stdin) for complex bodies, then validate the generated Markdown.
- Default discovery returns recommended modules. Use `layout list --lifecycle compatibility --json` only for old-content migration. Local validation proves syntax acceptance only; production support is a release-conformance fact.
- `gallery` is recommended. For `expand`, pass header fields, a standalone `---`, and Markdown body through `--body-file`. `cover-reveal` and `expand` default to complete static output; use `first-layer` only when the user explicitly requests an interaction candidate, and verify the target API before relying on it.

Default module discipline:

- Do not pile on modules.
- Use at most one hero, one verdict, and one cta unless the user explicitly asks for more.
- Skip modules when the article does not provide enough content to fill them honestly.

## Brand Profile

Brand Profile lives at `~/.config/md2wechat/brand.md`.

- It is free-form Markdown, not YAML and not a fixed schema.
- The CLI does not parse it.
- Read it as context for voice, theme preferences, module preferences, CTA preferences, and forbidden expressions.
- Treat quantity preferences as soft constraints.
- Verify any named theme or module through CLI discovery.
- If Brand Profile does not exist, do not block the task. You may mention once that system defaults will be used.
- Create or edit Brand Profile only when the user explicitly asks.

