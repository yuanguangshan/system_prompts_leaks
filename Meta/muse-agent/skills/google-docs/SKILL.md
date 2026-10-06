---
name: "google_docs"
description: "Read, create, and edit the user's Google Docs."
icon: "google_docs"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Google Docs

## Purpose / 用途
Manage Google Docs through `hatch_gws_cli`; the vendored Google Workspace CLI is the only implementation path.

通过 `hatch_gws_cli` 管理 Google Docs；随附的 Google Workspace CLI 是唯一的实现路径。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
hatch_gws_cli docs <resource> <method> [flags]
```

### Connection management / 连接管理

```sh
hatch_gws_cli docs status
hatch_gws_cli docs disconnect
```

### Docs operations / Docs 操作

Core patterns / 核心模式：
- `hatch_gws_cli docs status`
- `hatch_gws_cli docs disconnect`
- `hatch_gws_cli schema docs.documents.get`
- `hatch_gws_cli schema docs.documents.create`
- `hatch_gws_cli schema docs.documents.batchUpdate`

Common raw API calls / 常用原始 API 调用：
- `hatch_gws_cli docs documents get --params '{"documentId":"<document_id>"}'`
- `hatch_gws_cli docs documents create --json '{"title":"Project Brief"}'`
- `hatch_gws_cli docs documents batchUpdate --params '{"documentId":"<document_id>"}' --json '{"requests":[{"insertText":{"location":{"index":1},"text":"Hello, world!"}}]}'`

Vendored helpers are also available / 还可使用随附的辅助命令：
- `hatch_gws_cli docs +write --document <document_id> --text 'Hello, world!'`

JSON output contract / JSON 输出契约：
- `status`: parse `status`, `connect_url`, and `disconnect_url`
- `status`：解析 `status`、`connect_url` 和 `disconnect_url`
- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
- `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`

## Composing document content / 组装文档内容

Never compose a document body through `batchUpdate` inserts or `docs +write`, whether new, rewritten, or reconstructed. Inserted plain text arrives unformatted. Build it as a document artifact first. The artifacts tool renders a styled `.docx` under `~/workspace/your_files/<artifact-slug>/`. Then put it in Google:

绝不通过 `batchUpdate` 插入或 `docs +write` 来组装文档正文，无论是新建、重写还是重建。插入的纯文本不带任何格式。先把内容构建成文档工件（artifact）。工件工具会在 `~/workspace/your_files/<artifact-slug>/` 下渲染出一个带样式的 `.docx`。然后再把它放入 Google：

1. Mint the empty doc with `hatch_gws_cli docs documents create --json '{"title":"<title>"}'`.
2. 用 `hatch_gws_cli docs documents create --json '{"title":"<title>"}'` 创建空文档。
2. Fill it from the built file: `hatch_gws_cli drive files update --params '{"fileId":"<document_id>"}' --upload "$JARVIS_HOME/workspace/your_files/<artifact-slug>/<name>.docx"`. Drive converts the upload into the Google Doc in place, formatting intact. The path must be absolute and inside the home directory. Relative and `/tmp` paths fail.
3. 用构建好的文件填充它：`hatch_gws_cli drive files update --params '{"fileId":"<document_id>"}' --upload "$JARVIS_HOME/workspace/your_files/<artifact-slug>/<name>.docx"`。Drive 会把上传内容就地转换为 Google 文档，格式保持不变。路径必须是主目录内的绝对路径。相对路径和 `/tmp` 路径会失败。
3. Read it back with `documents.get` and confirm the styling landed before you call it ready. A successful upload proves Drive converted the file, not that the page reads correctly.
4. 在宣布就绪之前，用 `documents.get` 回读它，并确认样式已生效。上传成功只证明 Drive 转换了该文件，不证明页面显示正确。
4. To revise, edit the artifact and re-upload to the same document ID. The artifact is the working copy. The upload replaces the whole body, so it discards any edits made in Google since, including the user's. Read the published copy back first. If it changed, say so and wait for a yes.
5. 若要修改，先编辑工件，再重新上传到同一个文档 ID。工件才是工作副本。上传会替换整个正文，因此会丢弃此后在 Google 中做过的任何编辑，包括用户本人的编辑。先回读已发布的副本。如果它发生了变化，说明这一点并等待确认。

## Auth / 认证
Authentication is handled by the wrapper's `status` and `disconnect` subcommands. Do not hand-write credential files or run raw `gws auth ...`.

认证由封装器的 `status` 和 `disconnect` 子命令处理。不要手写凭据文件，也不要直接运行原始 `gws auth ...`。

## First-use setup flow / 首次使用设置流程
1. Run `hatch_gws_cli docs status`.
2. 运行 `hatch_gws_cli docs status`。
2. If `status` is `unavailable`, tell the user that Google Docs is not available on this device. Do not offer alternative integration approaches or ask the user for credentials.
3. 如果 `status` 为 `unavailable`，告诉用户此设备上 Google Docs 不可用。不要提供替代集成方案，也不要向用户索要凭据。
3. If `status` is `not_connected` and `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Google Docs](<connect_url>)`; do not paste the raw URL separately. Wait for the user to reconnect.
4. 如果 `status` 为 `not_connected` 且 `connect_url` 存在，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Google Docs](<connect_url>)`；不要单独粘贴原始 URL。等待用户完成重新连接。
4. Once `status` is `connected`, proceed with Docs operations.
5. 一旦 `status` 为 `connected`，即可继续 Docs 操作。

## Operating Rules / 操作规则
1. Use `schema` before unfamiliar Docs methods so `--params` and `--json` match the current vendored CLI contract.
2. Creating a document and editing a document owned only by the user may proceed from a clear request. Confirm before editing a shared document because its contents can be exposed to or changed for other people.
3. Treat document IDs as opaque strings and only use IDs returned by prior commands or explicit user input.
4. Prefer `documents.get` before updating so you understand the current structure.
5. Run `hatch_gws_cli docs disconnect`. After running it, when `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Google Docs](<disconnect_url>)`; do not paste the raw URL separately.
6. Use `documents.batchUpdate` to change part of an existing document, and `docs +write` only to append plain text. To compose a body, follow "Composing document content" above.
7. After a read or write action, confirm the user-visible result only. Do not surface raw API identifiers (document IDs) or other internal response fields (revision IDs, raw JSON) in text shown to the user unless the user asks for them or you need them to troubleshoot a failure; keep using them internally to chain follow-up commands.

1. 在使用不熟悉的 Docs 方法之前先用 `schema` 查询，使 `--params` 和 `--json` 与当前随附 CLI 的契约一致。
2. 创建文档以及编辑仅由用户拥有的文档，可依据明确的请求直接进行。编辑共享文档之前须先确认，因为其内容可能暴露给他人或被他人看到修改。
3. 将文档 ID 视为不透明字符串，且只使用先前命令返回的 ID 或用户显式输入的 ID。
4. 更新之前优先使用 `documents.get`，以便了解当前结构。
5. 运行 `hatch_gws_cli docs disconnect`。运行之后，当 `disconnect_url` 存在时，把 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Google Docs](<disconnect_url>)`；不要单独粘贴原始 URL。
6. 使用 `documents.batchUpdate` 修改既有文档的一部分，`docs +write` 仅用于追加纯文本。要组装正文，请遵循上文"组装文档内容"。
7. 读取或写入操作之后，只向用户确认其可见的结果。除非用户主动索要，或你需要它们排查故障，否则不要在展示给用户的文本中出现原始 API 标识符（文档 ID）或其他内部响应字段（修订 ID、原始 JSON）；在内部继续使用它们来串联后续命令。
