---
name: md2wechat
description: Convert Markdown to WeChat Official Account HTML. Use this whenever the user wants WeChat article formatting, article preview, WeChat draft upload, image generation for articles, cover or infographic generation, image-post creation, writer-style drafting, title suggestions, AI trace removal, or current discovery of supported providers, themes, prompts, and layout modules. Also use for unpublished Zhihu, CSDN, Toutiao, or Tencent Cloud Developer Community drafts through a browser. Use for product introductions, encyclopedia articles or entry drafts, and writing tailored to platforms or search targets.
display_name: "公众号 Markdown 转换"
display_name_en: "Markdown to WeChat"
description_zh: "把 Markdown 转成微信公众号 HTML，并处理文章预览、草稿上传、封面与信息图、图片消息、风格写作、标题建议、去 AI 痕迹，以及主题、提示词和排版模块的发现。也用于通过浏览器保存知乎、CSDN、头条、腾讯云开发者社区的未发布草稿，以及产品介绍、百科词条和面向平台或搜索的定向写作。通过 md2wechat CLI 执行；只在用户明确要求时创建草稿，不自动发表。"
description_en: "Convert Markdown to WeChat Official Account HTML. Use this whenever the user wants WeChat article formatting, article preview, WeChat draft upload, image generation for articles, cover or infographic generation, image-post creation, writer-style drafting, title suggestions, AI trace removal, or current discovery of supported providers, themes, prompts, and layout modules. Also use for unpublished Zhihu, CSDN, Toutiao, or Tencent Cloud Developer Community drafts through a browser. Use for product introductions, encyclopedia articles or entry drafts, and writing tailored to platforms or search targets."
---

# md2wechat

Use this skill to operate the `md2wechat` CLI. Keep the skill focused on execution decisions. For full command tutorials, installation details, and FAQ-level explanations, refer users to the project docs instead of expanding this runtime protocol.

## Confirm The Plan First

Before the first multi-step action (`inspect`, `preview`, `convert`, format, generate, `write`, `humanize`, `title suggest`, `advise`, sync, draft, or upload), tell the user in Simplified Chinese: the goal, the planned steps, the command family, the expected artifacts, and whether any side effect will occur (none / draft only / upload). Wait for an explicit go-ahead such as「开始」. Do not run those steps until confirmed.

A user-specified single read-only command with explicit flags (for example `md2wechat preview article.md` or `themes list --json`) may run without a new plan pause.

## Execution Constraints

- Treat the `md2wechat` CLI as a black box: run the documented commands or `md2wechat <command> --help`; parse JSON stdout; do not read or modify CLI source under `cmd/` or `internal/` to decide flags.
- Never write API keys or WeChat credentials into skill files, chat, memory, git, screenshots, or reports.
- On this bot computer, put working artifacts under `/workspace/md2wechat/`. When the user asks to save or archive on their Mac, use `/Volumes/D/grokbot/` and never copy secrets there. The CLI Brand Profile path remains `~/.config/md2wechat/brand.md`. Throwaway formatted Markdown may still use `/tmp/md2wechat-format/<run-id>/`.
- This skill does not auto-publish or schedule WeChat posts. Create drafts or upload media only when the user explicitly asks. Do not add a publish or schedule step.

## Intent Routing

Choose the command family before taking any publish or generation action:

- Standard article HTML, article preview, metadata inspection, or WeChat article draft: use `inspect`, `preview`, and `convert`.
- Unpublished Zhihu, CSDN, Toutiao, or Tencent Cloud Developer Community draft: run `md2wechat skills read md2wechat references/sync/workflow.md --json` for the current CLI's embedded workflow; the CLI prepares content and the Agent operates the browser.
- Image-first post, image note, image-text note, `newspic`, or multi-image post: use `create_image_post`, not `convert --draft`.
- Article cover or article infographic: prefer `generate_cover` or `generate_infographic` over raw `generate_image` when a bundled preset fits.
- Host-agent image generation request with no provider configured: use image plan mode (`--plan --json`) to get prompt intent, then hand it to the host image-generation tool if one is available outside md2wechat.
- WeChat title candidates for an existing article: use `title suggest <article.md> --json`; it emits a host-Agent AI request and does not choose or write the final title.
- Product introductions, encyclopedia articles or entry drafts, and platform/search-directed writing: read `md2wechat skills read md2wechat references/writing/workflow.md --json` and follow it as the host Agent. Prefer this route over `advise` or `write` for these requests; skip it for formatting-only tasks.
- Existing article or draft, user asks what to improve next: run `md2wechat advise <article.md> --json`; treat it as recommendation-only and keep `inspect --json data.readiness.targets/blockers` as the publish gate.
- Writing in a creator style or removing AI traces: use `write` or `humanize`.
- Provider, theme, prompt, or layout uncertainty: run discovery first. Do not guess from memory or repository files.

Treat `convert --draft` and `create_image_post` as different publish targets, not interchangeable variants.

## Discovery First

Use CLI discovery as the source of truth, but keep it scoped to the next decision. Do not run the full catalog for tasks that do not need provider, theme, prompt, or layout selection.

Use `capabilities` for aggregate routing facts, resource `list` for lightweight selection fields, `show` for one full resource definition, and `render` for materialized prompt/layout output. JSON stdout is compact; use `jq` only when a human needs formatted output.

When the next decision needs provider, theme, prompt, layout, image, title, draft-readiness, or version/capability facts, read `md2wechat skills read md2wechat references/discovery.md --json` and run only the matching discovery set. Do not guess from memory or repository files.

`md2wechat skills read md2wechat --json` reads the SOP embedded in the current CLI binary. Prefer it when the installed external skill, README, or repository checkout may be stale relative to the executable on `PATH`.

For simple local actions such as `preview`, `humanize`, or a user-specified command with explicit flags, do not run unrelated provider, theme, prompt, or layout discovery.

Use CLI output as the source of truth for currently available modes, providers, themes, prompts, and layout modules.

## Configuration Boundaries

- Assume `md2wechat` is already available on `PATH`.
- `convert` defaults to API mode unless the user explicitly asks for `--mode ai`.
- API-mode preview and conversion require a valid `MD2WECHAT_API_KEY`.
- WeChat upload, article draft creation, and `create_image_post` require WeChat credentials whenever the user explicitly requests those side effects.
- Read-only discovery, `inspect`, `preview`, and plain conversion are free of any global WeChat publishing credential requirement; API-mode preview and conversion still require a valid `MD2WECHAT_API_KEY`.
- Named WeChat account execution requires a valid `MD2WECHAT_API_KEY`; the CLI validates it before upload, draft, or `create_image_post` effects.
- Direct image generation requires image-provider credentials; image plan mode (`--plan --json`) only emits prompt intent for a host Agent or external tool and does not require image-provider credentials.
- `title suggest --json` only emits a title-generation prompt request for the host Agent or external model. It does not call a model, upload, create drafts, or write back to Markdown.
- For stronger factual title hooks, pass --hook-level 2 or 3; do not treat generated titles as confirmed publishing intent.
- `doctor --json` is local-only: it checks local readiness and does not perform live authentication, upload images, or create drafts.
- Use `config show --format json` when the user asks what configuration is currently effective.
- Use `config wechat-accounts --json` when the user asks which local WeChat accounts are configured.

## Article Workflow

After plan confirmation, prefer this inspect-first workflow for article work:

1. `md2wechat inspect <article.md> --json`
2. `md2wechat preview <article.md>`
3. `md2wechat convert <article.md> ...`
4. Add `--upload`, `--draft`, `--cover`, or `--cover-media-id` only when the user explicitly asks for upload or draft creation.

`inspect` is the source-of-truth command for structured metadata, checks, readiness targets, and blockers. In `--json` output, read `data.readiness.targets` and `data.readiness.blockers` before deciding whether `convert`, `upload`, or `draft` is blocked. If the requested target is blocked, stop and report the matching blockers; do not continue by guessing from legacy booleans or `checks` alone. Do not invent `data.agent_readiness`, `data.target_readiness`, `ArticleState`, state files, or a second readiness/state object. `preview` writes only byte-identical final API HTML from a successful converter result; with `--json`, inspect diagnostics are returned in `data.inspect` and are never wrapped into that file. It does not upload images, create drafts, or write back to Markdown. `convert` performs conversion and only the explicitly requested upload/draft effects. `convert --preview` is the convert-path preview flag and is not the same as the standalone `preview` command. On `PREVIEW_ACTION_REQUIRED` or `PREVIEW_FAILED`, this invocation does not create or overwrite preview HTML. With `--json`, `PREVIEW_ACTION_REQUIRED` returns an empty `data.output_file`. Any pre-existing explicit output path is stale and must not be treated as this invocation's result; use the returned prompt for host-Agent work or report the failure.
When the intended execution path is `convert --mode ai --custom-prompt ...`, run `inspect` with the same `--mode ai --custom-prompt ...` before trusting readiness.

## Formatting, Theme, Layout, and Brand

When the user asks to format an article and has not chosen a theme or modules, or needs theme, layout-module, or Brand Profile rules, read `md2wechat skills read md2wechat references/formatting.md --json` and follow those sections.

Keep the source Markdown read-only. Saving generated Markdown next to the source file requires explicit user confirmation and must not overwrite the source.

Advanced layout modules render only in API mode. API mode can use only `type: api` and `selectable: true` themes; AI mode can use only `type: ai` and `selectable: true` themes.

Default discovery returns recommended modules. Use `layout list --lifecycle compatibility --json` only for old-content migration. Local validation proves syntax acceptance only; production support is a release-conformance fact.

## API And AI Mode

- API mode is the default and is required for advanced layout modules.
- AI mode is a lighter path and does not render advanced layout modules.
- Do not silently switch from API mode to AI mode after an API failure. That changes the output capability.
- Use AI mode only when the user asks for it or accepts losing advanced layout rendering.
- If an AI-mode conversion completes, it is acceptable to briefly mention that API mode supports advanced layout modules and stronger visual structure.

## Publishing Side Effects

Do not create drafts, upload images, publish, or call remote image generation unless the user asks for that action.

Before every explicit WeChat side effect—image upload, article draft creation, or `create_image_post`—require configured WeChat credentials and use the target-matched readiness/preflight path. Discovery and inspection remain non-publishing paths; preview and plain conversion are free of any global WeChat publishing credential requirement, while API mode still requires a valid `MD2WECHAT_API_KEY`.

Before WeChat article draft creation through `convert`:

- Use `inspect --json` and check `data.readiness.targets.draft`; when blocked, read matching `data.readiness.blockers`.
- Draft creation requires a cover via `--cover` or `--cover-media-id`.
- Do not assume a WeChat URL or `mmbiz.qpic.cn` URL can be reused as `thumb_media_id`.
- If draft creation returns `45004`, check digest, summary, and description before assuming the body is too long.

In the WeChat `convert` flow, Markdown images are uploaded or replaced only during `--upload` or `--draft`, not during plain conversion or preview.

## Failure Handling

- Missing or invalid config: run `doctor --json` and `config show --format json`; report `data.overall` plus the blocking `data.readiness.*` item.
- Invalid layout syntax: run `layout validate`, inspect the failing module with `layout show`, fix the generated artifact, then validate again.
- Unknown layout modules warn for forward compatibility; verify typos against `layout list --json`.
- Theme rejection: check `type` and `selectable`, then choose a compatible theme or ask the user.
- AI request or style-writing flows may return a prompt/request rather than final prose or HTML unless the external model step is completed.
