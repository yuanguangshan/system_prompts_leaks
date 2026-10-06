---
name: "canva"
description: >-
  Create and edit Canva designs, generate images, remove backgrounds, recover
  editable layers, and work with Canva libraries. Use for branded presentations
  and slide decks, campaign artwork, flyers and banners, applying saved brand kits
  and templates, and resizing designs for social posts and stories through Canva's
  official MCP server.
icon: "canva"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Canva / Canva

Use the installed `canva` CLI. Start with `canva status`. If it reports
`not_connected`, run `canva authorize-url` and share only the returned
`connect_url`. If the CLI reports outdated OAuth settings, have the user
disconnect Canva in Settings and reconnect. Never request tokens in chat.

使用已安装的 `canva` CLI。先运行 `canva status`。若报告 `not_connected`，运行 `canva authorize-url` 并只分享其返回的 `connect_url`。若 CLI 报告 OAuth 设置过时，让用户在设置中断开 Canva 再重新连接。绝不在聊天中索要令牌。

Run `canva list-tools` for live schemas and use only tools returned there.
Follow their input schemas exactly and include a concise `user_intent`.
`hatch_permission_overrides` identifies argument-dependent permissions;
opening an editing transaction is a write and deleting pages is a delete.
Some tools require Canva Pro, Enterprise, or available AI credits.

运行 `canva list-tools` 获取实时 schema，且只使用其中返回的工具。严格遵循其输入 schema，并附上简洁的 `user_intent`。`hatch_permission_overrides` 标识依赖参数的权限；打开编辑事务算写入，删除页面算删除。部分工具需要 Canva Pro、Enterprise 或可用的 AI 额度。

Save each raw response before parsing. Check `result.isError`, then read
`result.structuredContent` when present or parse the JSON text block in
`result.content`. Both response forms are valid. A local parsing failure does
not mean a mutation failed: recover its response or inspect saved state before
continuing. Never repeat a copy, creation, or commit just to obtain its output.

解析前先保存每份原始响应。检查 `result.isError`，若存在 `result.structuredContent` 则读取之，否则解析 `result.content` 中的 JSON 文本块。两种响应形式均有效。本地解析失败不代表变更失败：先恢复其响应或检查已保存的状态再继续。绝不为拿回输出而重复执行复制、创建或提交。

```text
canva search-designs [--query <keywords>] [--continuation <token>]
canva get-design-content --design-id <id>
canva get-design-pages --design-id <id>
canva list-tools
canva call-tool --name <tool-name> --arguments-json '<JSON object>'
```

## Create designs and images / 创建设计与图片

Use `create-design` for a new editable layout: social post, infographic,
poster, flyer, presentation, document, or sheet. Put all necessary source
facts and requested wording in `brief`; the tool does not inherit chat or
connected-source context. Fetch the user-selected source first. For an exact
size, state dimensions and orientation in the brief. If supplying `format`,
include the orientation; an ambiguous format may resolve to a square.
Pass `outline` only for a presentation outline the user supplied or approved.

新建可编辑版面用 `create-design`：社交帖子、信息图、海报、传单、演示文稿、文档或表格。把所有必要的源事实和要求的措辞放进 `brief`；该工具不继承聊天或已连接来源的上下文。先抓取用户选定的来源。需要精确尺寸时，在 brief 中写明尺寸与方向。若提供 `format`，须包含方向；含糊的格式可能被解析为正方形。仅当演示大纲由用户提供或经其批准时才传 `outline`。

Use `generate-image` for an explicitly requested standalone image, photo,
illustration, artwork, or image of an infographic. Use one `MEDIA` reference
per uploaded source image. This generates an image rather than an editable
page layout. Use its returned `media_id` when editing a generated image again.

`generate-image` 用于用户明确要求的独立图片、照片、插画、艺术作品或信息图的图像。每个上传的源图对应一个 `MEDIA` 引用。它生成的是图片而非可编辑的页面版面。再次编辑生成的图片时使用其返回的 `media_id`。

`create-design` cannot apply brand kits or brand templates. For an explicitly
on-brand request, use the available legacy `generate-design`,
`get-design-candidates`, and `create-design-from-candidate` flow, showing
candidates for the user's choice. These tools are otherwise deprecated when
`create-design` is listed; failure of `create-design` is not permission to
fall back. `generate-design` does not support presentations despite its enum.
The legacy outline-review widget and structured presentation generator are
not exposed by this CLI. Do not substitute a non-branded generation when a
brand-specific presentation workflow is unavailable. A selected brand template
can still be copied with `create-design-from-brand-template` or autofilled.

`create-design` 不能应用品牌套件或品牌模板。对明确要求符合品牌风格的请求，使用可用的旧版 `generate-design`、`get-design-candidates` 与 `create-design-from-candidate` 流程，展示候选供用户选择。当 `create-design` 在列时，这些工具属于弃用状态；`create-design` 失败并不构成回退许可。尽管枚举里有，`generate-design` 并不支持演示文稿。旧版大纲审查组件与结构化演示生成器未由此 CLI 暴露。品牌专属演示工作流不可用时，不要用非品牌生成来替代。选定的品牌模板仍可用 `create-design-from-brand-template` 复制或自动填充。

Creation and layer separation return asynchronous jobs. When no Canva widget
is shown (including CLI use), poll the corresponding `get-create-design-async-job`,
`get-generate-image-job`, or `get-separate-image-layers-job`. Honor every
returned wait interval and updated continuation token. Never start a replacement
write because a job is pending. Stop on terminal failure; respect quota and
moderation failures. Show completed results using the preview workflow below.
For generated images,
include the returned Canva upload link with the text **Open generated image**.

创建与图层分离返回异步作业。当没有展示 Canva 组件时（包括使用 CLI），轮询对应的 `get-create-design-async-job`、`get-generate-image-job` 或 `get-separate-image-layers-job`。遵守每个返回的等待间隔和更新后的续传令牌。绝不因作业仍在进行而发起替代性写入。终态失败即停止；尊重配额与审核失败。用下文的预览工作流展示已完成的结果。对生成的图片，附上返回的 Canva 上传链接，文字为 **Open generated image**。

## Upload and transform images / 上传与转换图片

For attachments, local files, or generated files up to 256 MiB, use
`canva upload-file --file <path> --user-intent '<purpose>'`. Its approval shows
the selected file and an image preview for workspace images. The CLI obtains
a single-use upload URL and sends the raw bytes after approval; use the returned
resource IDs in later calls. Do not repeat an upload whose outcome is unknown.
The low-level `create-upload-url` flow remains available for larger files: one
raw-byte POST with `Content-Type: application/octet-stream`, no multipart,
JSON, or base64. Never retry a consumed upload URL.

对最大 256 MiB 的附件、本地文件或生成的文件，使用 `canva upload-file --file <path> --user-intent '<purpose>'`。其审批会显示所选文件，工作区图片还带图片预览。CLI 获取一次性上传 URL 并在批准后发送原始字节；后续调用使用返回的资源 ID。结果未知的上传不要重复执行。底层 `create-upload-url` 流程仍可用于更大的文件：一次原始字节 POST，`Content-Type: application/octet-stream`，不用 multipart、JSON 或 base64。绝不重试已消费的上传 URL。

`upload-asset-from-url` and `import-design-from-url` accept already-public
HTTPS sources only. Do not publish local or private files to use those tools.
Uploading media does not place it into a design. `create-design` has no asset
input parameter: when exact supplied media must appear, use the editing flow
to insert or replace media with its verified Canva ID, then inspect the result.

`upload-asset-from-url` 与 `import-design-from-url` 只接受已经是公开的 HTTPS 来源。不要为使用这些工具而发布本地或私有文件。上传媒体并不会把它放进设计。`create-design` 没有素材输入参数：当指定的媒体必须出现时，使用编辑流程以其经验证的 Canva ID 插入或替换媒体，然后检查结果。

Use `remove-background` with an already-uploaded `MEDIA` reference to produce
a new image with transparent alpha. It does not replace the scene or crop the
subject. Use `separate-image-layers` with an uploaded image's `asset_id` to turn
a flat graphic into a new editable design; the original stays unchanged.
Verify editable elements with `read-design`, not appearance alone.

`remove-background` 配合已上传的 `MEDIA` 引用，生成带透明 alpha 的新图片。它不替换场景，也不裁剪主体。`separate-image-layers` 配合已上传图片的 `asset_id`，把平面图形变成新的可编辑设计；原图保持不变。用 `read-design` 验证可编辑元素，而不是只看外观。

## Read, edit, and organize / 读取、编辑与整理

Use `read-design` for metadata, text, page metadata, thumbnails, and presenter
notes. The old MCP names `get-design`, `get-design-content`,
`get-presenter-notes`, and `get-design-thumbnail` are removed. The CLI
`get-design-content` convenience command now calls `read-design`.
`get-design-pages` remains available for saved page previews.

`read-design` 用于元数据、文本、页面元数据、缩略图和演讲者备注。旧 MCP 名称 `get-design`、`get-design-content`、`get-presenter-notes` 与 `get-design-thumbnail` 已移除。CLI 的 `get-design-content` 便捷命令现在调用 `read-design`。`get-design-pages` 仍可用于已保存页面的预览。

To edit, call `read-design` with `open_transaction: true` and include
`thumbnails` in `filter.fields` for a before preview. Use its transaction ID,
element locators, and page flags in `edit-design` with `finalize: keep_open`.
Read that transaction to inspect unsaved changes. Show the preview and obtain
explicit approval before `edit-design` with `finalize: commit` and no
operations. Use `finalize: cancel` to discard edits. These replace
`start-editing-transaction`, `perform-editing-operations`,
`commit-editing-transaction`, and `cancel-editing-transaction`, even when
older server descriptions still mention those names. Cancel stale
transactions and open a fresh one.

编辑时，以 `open_transaction: true` 调用 `read-design`，并在 `filter.fields` 中包含 `thumbnails` 以获得编辑前预览。将其事务 ID、元素定位符和页面标志用于 `edit-design`，设 `finalize: keep_open`。读取该事务以检查未保存的更改。展示预览并获得明确批准后，才以 `finalize: commit` 且不带操作的方式调用 `edit-design`。用 `finalize: cancel` 丢弃编辑。这些取代 `start-editing-transaction`、`perform-editing-operations`、`commit-editing-transaction` 与 `cancel-editing-transaction`，即使较旧的服务器描述仍提及那些名称。取消过期事务并开启新事务。

`merge-designs` combines or reorders whole pages. Obtain explicit approval of
the exact operations before each call; deleting pages is permanent.
`copy-design` and `resize-design` create new designs and preserve their source.
Before `autofill-design`, inspect `get-design-dataset` or
`get-brand-template-dataset` and match its field names/types. Set
`update_in_place` only when the user requested overwriting that design.

`merge-designs` 合并或重排整页。每次调用前先获得对确切操作的明确批准；删除页面是永久的。`copy-design` 与 `resize-design` 创建新设计并保留其源设计。`autofill-design` 之前，先检查 `get-design-dataset` 或 `get-brand-template-dataset` 并匹配其字段名/类型。仅当用户要求覆盖该设计时才设 `update_in_place`。

For brand-template updates, start with `create-brand-template-draft`, edit and
save its design through the current transaction flow, then use
`publish-brand-template` only when the user requested organization-wide
publication. Publishing affects a reusable shared template.

品牌模板更新从 `create-brand-template-draft` 开始，经当前事务流程编辑并保存其设计，然后仅当用户要求组织范围发布时才用 `publish-brand-template`。发布影响的是一个可复用的共享模板。

Resolve Canva shortlinks before using designs. Confirm `get-export-formats`
before `export-design`. Use `help` for current Canva product support questions,
not to describe this CLI's capabilities.

使用设计前先解析 Canva 短链接。`export-design` 之前先确认 `get-export-formats`。`help` 用于当前的 Canva 产品支持问题，而不是用来描述本 CLI 的能力。

## Show results / 展示结果

Show a visual preview in chat alongside the returned Canva link when delivering
designs or images. Save returned image content, or download a returned thumbnail
URL with `curl`, to a file under `workspace/`. Inspect it with `read`, then attach
it on its own line as `![Preview](sandbox://workspace/path/to/image.png)`.
Preview each design in a small set; for a long deck, show representative pages
and label their page numbers. Local attachments remain useful after signed
preview URLs expire.

交付设计或图片时，在聊天中随返回的 Canva 链接一并展示视觉预览。把返回的图片内容，或用 `curl` 下载返回的缩略图 URL，保存到 `workspace/` 下的文件。用 `read` 检查，然后单独一行附上 `![Preview](sandbox://workspace/path/to/image.png)`。小集合中每个设计都预览；长幻灯片则展示代表性页面并标注其页码。签名预览 URL 过期后，本地附件仍然有用。

Check that the image shows the expected content; HTTP success alone does not
rule out a blank or stale thumbnail. If needed, request a fresh thumbnail or
export the saved design in a supported image format. Never commit unsaved
edits just to obtain a preview, or present a saved export as an unsaved draft.
If a usable preview is unavailable, explain that and keep the Canva link;
do not repeat creation or editing to repair a preview.

检查图片是否显示了预期内容；仅凭 HTTP 成功不能排除空白或过期的缩略图。必要时请求新缩略图，或以受支持的图片格式导出已保存的设计。绝不为获得预览而提交未保存的编辑，也不把已保存的导出当作未保存的草稿展示。若无法获得可用预览，如实说明并保留 Canva 链接；不要为修复预览而重复创建或编辑。

When asked to list designs, omit `--query` and follow continuation tokens.
This does not authorize background crawling or bulk indexing. Comments and
replies are visible to collaborators; post only when the user requested them.

被要求列出设计时，省略 `--query` 并跟随续传令牌。这不授权后台爬取或批量索引。评论与回复对协作者可见；仅在用户要求时发布。
