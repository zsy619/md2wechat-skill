# Discovery command sets

Read this file when the next decision needs provider, theme, prompt, layout, image, title, draft-readiness, or version/capability facts. Run only the matching set. Do not guess from memory or repository files.

Run the smallest useful discovery set:

- Article formatting with no theme or modules chosen:
  ```bash
  md2wechat themes list --json
  md2wechat layout list --json
  ```

- A named theme, provider, prompt, or layout module:
  ```bash
  md2wechat themes show <name> --json
  md2wechat providers show <name> --json
  md2wechat prompts show <name> --kind <kind> --json
  md2wechat layout show <name> --json
  ```

- Image generation or image-preset selection:
  ```bash
  md2wechat providers list --json
  md2wechat prompts list --kind image --json
  ```

- Subject-reference (image-to-image) capability before using `--subject-reference`:
  ```bash
  md2wechat providers show minimax --json
  ```
  Read `supports_subject_reference` on the provider and on each entry of `supported_models`. Only the `minimax` provider and its `image-01` model accept `--subject-reference`, and the reference must be a publicly reachable `http(s)` portrait image URL; inline data URLs and local paths are rejected. Unsupported provider/model combinations fail immediately with `CONFIG_INVALID`, so do not retry them as generation failures.

- Title suggestion prompt selection:
  ```bash
  md2wechat prompts list --kind title --json
  md2wechat prompts show wechat-title-expert --kind title --json
  ```

- Draft, upload, API local-readiness, or configuration troubleshooting:
  ```bash
  md2wechat doctor --json
  md2wechat config show --format json
  md2wechat config wechat-accounts --json
  ```
  `doctor` readiness is local configuration attemptability. `config wechat-accounts` is local-only and never prints WeChat secrets. Use `inspect --json` for article-specific target readiness.

- Unknown CLI version, changed behavior, or capability uncertainty:
  ```bash
  md2wechat version --json
  md2wechat capabilities --json
  md2wechat skills list --json
  md2wechat skills read md2wechat --json
  ```

Inspect specific resources only when the task needs them:

```bash
md2wechat providers show <name> --json
md2wechat themes show <name> --json
md2wechat prompts show <name> --kind <kind> --json
md2wechat layout show <name> --json
```
