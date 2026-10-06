---
name: "google_slides"
description: "Read, create, and edit the user's Google Slides presentations."
icon: "google_slides"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Google Slides

## Purpose / 用途
Manage Google Slides through `hatch_gws_cli`; the vendored Google Workspace CLI is the only implementation path.

通过 `hatch_gws_cli` 管理 Google Slides；随附的 Google Workspace CLI 是唯一的实现路径。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
hatch_gws_cli slides <resource> <method> [flags]
```

### Connection management / 连接管理

```sh
hatch_gws_cli slides status
hatch_gws_cli slides disconnect
```

### Slides operations / Slides 操作

Core patterns / 核心模式：
- `hatch_gws_cli slides status`
- `hatch_gws_cli slides disconnect`
- `hatch_gws_cli schema slides.presentations.get`
- `hatch_gws_cli schema slides.presentations.create`
- `hatch_gws_cli schema slides.presentations.batchUpdate`

Common raw API calls / 常用原始 API 调用：
- `hatch_gws_cli slides presentations get --params '{"presentationId":"<presentation_id>"}'`
- `hatch_gws_cli slides presentations create --json '{"title":"Quarterly Review"}'`
- `hatch_gws_cli slides presentations batchUpdate --params '{"presentationId":"<presentation_id>"}' --json '{"requests":[{"createSlide":{"slideLayoutReference":{"predefinedLayout":"TITLE_AND_BODY"}}}]}'`

JSON output contract / JSON 输出契约：
- `status`: parse `status`, `connect_url`, and `disconnect_url`
- `status`：解析 `status`、`connect_url` 和 `disconnect_url`
- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
- `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`

## Composing presentation content / 组装演示文稿内容

Never compose a deck through `presentations create` plus `batchUpdate` element inserts, whether new or rebuilt. Build it as a presentation artifact first. The artifacts tool writes a `.pptx` under `~/workspace/your_files/<artifact-slug>/`. Then put it in Google.

绝不通过 `presentations create` 加 `batchUpdate` 元素插入的方式来组装演示文稿，无论是新建还是重建。先把内容构建成演示文稿工件（artifact）。工件工具会在 `~/workspace/your_files/<artifact-slug>/` 下写出一个 `.pptx`。然后再把它放入 Google。

1. Mint the empty presentation with `hatch_gws_cli slides presentations create --json '{"title":"<title>"}'`.
2. 用 `hatch_gws_cli slides presentations create --json '{"title":"<title>"}'` 创建空演示文稿。
2. Fill it from the built file: `hatch_gws_cli drive files update --params '{"fileId":"<presentation_id>"}' --upload "$JARVIS_HOME/workspace/your_files/<artifact-slug>/<name>.pptx"`. Drive converts the upload into the Google Slides deck in place. The path must be absolute and inside the home directory. Relative and `/tmp` paths fail.
3. 用构建好的文件填充它：`hatch_gws_cli drive files update --params '{"fileId":"<presentation_id>"}' --upload "$JARVIS_HOME/workspace/your_files/<artifact-slug>/<name>.pptx"`。Drive 会把上传内容就地转换为 Google Slides 演示文稿。路径必须是主目录内的绝对路径。相对路径和 `/tmp` 路径会失败。
3. Read it back with `presentations.get` before you call it ready, and confirm the page count matches the artifact. Say in your reply that the slides are pictures, so the text cannot be edited in Slides.
4. 在宣布就绪之前，用 `presentations.get` 回读它，并确认页数与工件一致。在回复中说明这些幻灯片是图片形式，因此文本无法在 Slides 中编辑。
4. To revise, edit the artifact and re-upload to the same presentation ID. The artifact is the working copy. The upload replaces every slide, so it discards any edits made in Google since, including the user's. Read the published copy back first. If it changed, say so and wait for a yes.
5. 若要修改，先编辑工件，再重新上传到同一个演示文稿 ID。工件才是工作副本。上传会替换每一张幻灯片，因此会丢弃此后在 Google 中做过的任何编辑，包括用户本人的编辑。先回读已发布的副本。如果它发生了变化，说明这一点并等待确认。

## Auth / 认证
Authentication is handled by the wrapper's `status` and `disconnect` subcommands. Do not hand-write credential files or run raw `gws auth ...`.

认证由封装器的 `status` 和 `disconnect` 子命令处理。不要手写凭据文件，也不要直接运行原始 `gws auth ...`。

## First-use setup flow / 首次使用设置流程
1. Run `hatch_gws_cli slides status`.
2. 运行 `hatch_gws_cli slides status`。
2. If `status` is `unavailable`, tell the user that Google Slides is not available on this device. Do not offer alternative integration approaches or ask the user for credentials.
3. 如果 `status` 为 `unavailable`，告诉用户此设备上 Google Slides 不可用。不要提供替代集成方案，也不要向用户索要凭据。
3. If `status` is `not_connected` and `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Google Slides](<connect_url>)`; do not paste the raw URL separately. Wait for the user to reconnect.
4. 如果 `status` 为 `not_connected` 且 `connect_url` 存在，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Google Slides](<connect_url>)`；不要单独粘贴原始 URL。等待用户完成重新连接。
4. Once `status` is `connected`, proceed with Slides operations.
5. 一旦 `status` 为 `connected`，即可继续 Slides 操作。

## Operating Rules / 操作规则
1. Use `schema` before unfamiliar Slides methods so `--params` and `--json` match the current vendored CLI contract.
2. Creating a presentation and editing a presentation owned only by the user may proceed from a clear request. Confirm before editing a shared presentation because its contents can be exposed to or changed for other people.
3. Treat presentation IDs, page object IDs, and element IDs as opaque strings.
4. Run `hatch_gws_cli slides disconnect`. After running it, when `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Google Slides](<disconnect_url>)`; do not paste the raw URL separately.
5. Use `presentations.get` before modifying an existing deck so you understand the current page and object structure.
6. After a read or write action, confirm the user-visible result only. Do not surface raw API identifiers (presentation, page object, and element IDs) or other internal response fields (revision IDs, raw JSON) in text shown to the user unless the user asks for them or you need them to troubleshoot a failure; keep using them internally to chain follow-up commands.

1. 在使用不熟悉的 Slides 方法之前先用 `schema` 查询，使 `--params` 和 `--json` 与当前随附 CLI 的契约一致。
2. 创建演示文稿以及编辑仅由用户拥有的演示文稿，可依据明确的请求直接进行。编辑共享演示文稿之前须先确认，因为其内容可能暴露给他人或被他人看到修改。
3. 将演示文稿 ID、页面对象 ID 和元素 ID 视为不透明字符串。
4. 运行 `hatch_gws_cli slides disconnect`。运行之后，当 `disconnect_url` 存在时，把 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Google Slides](<disconnect_url>)`；不要单独粘贴原始 URL。
5. 在修改既有演示文稿之前使用 `presentations.get`，以便了解当前的页面与对象结构。
6. 读取或写入操作之后，只向用户确认其可见的结果。除非用户主动索要，或你需要它们排查故障，否则不要在展示给用户的文本中出现原始 API 标识符（演示文稿、页面对象和元素 ID）或其他内部响应字段（修订 ID、原始 JSON）；在内部继续使用它们来串联后续命令。
