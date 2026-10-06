---
name: "google_drive"
description: "Work with the user's Google Drive: files, folders, uploads, downloads, and sharing."
icon: "google_drive"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Google Drive / Google Drive

Everything runs through `hatch_gws_cli drive ...`. Raw API calls are space-separated (`<resource> <method>`) and take a `--params` JSON object for query and path values, plus a `--json` body when creating or editing. Run `hatch_gws_cli drive <command> --help` for a command's flags, and `hatch_gws_cli schema drive.files.list` (and the like) for a raw method's `--params` and `--json` shape. Check the shape before an unfamiliar method.

一切都通过 `hatch_gws_cli drive ...` 运行。原始 API 调用以空格分隔（`<resource> <method>`），接受用于查询和路径取值的 `--params` JSON 对象，创建或编辑时再加 `--json` 请求体。运行 `hatch_gws_cli drive <command> --help` 查看命令标志，运行 `hatch_gws_cli schema drive.files.list` 等类似命令查看原始方法的 `--params` 与 `--json` 结构。使用不熟悉的方法前先查其结构。

## Connecting / 连接

Drive needs a one-time connect before commands return data. Run `hatch_gws_cli drive status`. If it comes back not connected with a `connect_url`, post that exact URL as `[Connect Google Drive](<connect_url>)`, then stop and wait for the user to tap it. If no URL is returned, report that connection is unavailable and stop; never invent one. Connecting is the user's step, so never open the sign-in, drive a browser to it, point the user to Settings or a Google account page, or ask for credentials. If status comes back unavailable, say Google Drive is not available on this device and stop.

Drive 需要先完成一次性连接，命令才会返回数据。运行 `hatch_gws_cli drive status`。若返回未连接且带 `connect_url`，将该 URL 原样以 `[Connect Google Drive](<connect_url>)` 形式发布，然后停止并等待用户点击。若未返回 URL，报告连接不可用并停止；绝不编造 URL。连接是用户自己的步骤，因此绝不打开登录页、用浏览器导航过去、把用户指向设置或 Google 账户页面，也不索要凭据。若 status 返回不可用，说明 Google Drive 在此设备上不可用并停止。

Disconnect with `hatch_gws_cli drive disconnect`. Post `[Disconnect Google Drive](<disconnect_url>)` only when the command returns that URL. If status remains connected after the link was posted, re-post the same link and let the user finish in the browser; never send them to a Google account, security, or third-party-access page, and never loop on status. If no link is returned, rerun status once and report its state without inventing a link. If a later command reports an auth error, rerun status: post its connect link and wait when disconnected, retry the command once when connected, or report that Drive is unavailable when there is no usable link. Auth flows only through status and disconnect. Never hand-author credential files or run raw `gws auth`.

用 `hatch_gws_cli drive disconnect` 断开连接。仅当命令返回该 URL 时才发布 `[Disconnect Google Drive](<disconnect_url>)`。若发布链接后 status 仍显示已连接，重新发布同一链接，让用户在浏览器中完成；绝不把用户送往 Google 账户、安全或第三方访问页面，也绝不在 status 上循环。若未返回链接，重跑一次 status 并如实报告其状态，不编造链接。若后续命令报告认证错误，重跑 status：未连接时发布其连接链接并等待；已连接时将该命令重试一次；没有可用链接时报告 Drive 不可用。认证只经由 status 与 disconnect 进行。绝不手写凭据文件，也不运行原始的 `gws auth`。

Most people have one Google account, and that is the default: run commands with no account flag. If the user names one of several linked Drives ("my work Drive"), list them with `hatch_gws_cli drive accounts`, match the user's words to exactly one `display_name`, and pass its `account_id` as `--account <account_id>`. If there is no exact match, ask which account rather than guessing or silently falling back to the default.

多数人只有一个 Google 账户，这也是默认情况：运行命令时不带账户标志。若用户在多个已关联的 Drive 中指名其一（"我的工作 Drive"），用 `hatch_gws_cli drive accounts` 列出它们，将用户的话与唯一一个 `display_name` 精确匹配，并把其 `account_id` 作为 `--account <account_id>` 传入。若无精确匹配，询问是哪个账户，而不是猜测或悄悄回退到默认账户。

## Common flows / 常用流程

### Find and read / 查找与读取

- See what is in Drive: `drive files list --params '{"pageSize":20}'`. To browse inside one folder, query by its id: `drive files list --params "{\"q\":\"'<folder_id>' in parents and trashed=false\",\"pageSize\":50}"`.
  查看 Drive 中有什么：`drive files list --params '{"pageSize":20}'`。浏览某个文件夹内部时按其 id 查询：`drive files list --params "{\"q\":\"'<folder_id>' in parents and trashed=false\",\"pageSize\":50}"`。
- Search for a file: `drive files list --params "{\"q\":\"name contains 'budget' and trashed=false\",\"pageSize\":20}"`. Drive query operands use single quotes; use `fullText contains 'text'` to search inside content.
  搜索文件：`drive files list --params "{\"q\":\"name contains 'budget' and trashed=false\",\"pageSize\":20}"`。Drive 查询操作数使用单引号；搜索内容内部用 `fullText contains 'text'`。
- Get one file's details: `drive files get --params '{"fileId":"<id>","fields":"id,name,mimeType,parents,webViewLink,modifiedTime,owners"}'`.
  获取单个文件的详情：`drive files get --params '{"fileId":"<id>","fields":"id,name,mimeType,parents,webViewLink,modifiedTime,owners"}'`。

### Upload and download / 上传与下载

- Upload a local file the user gave you: `drive +upload ...` (run `drive +upload --help` for its flags). Confirm the local path exists first, and drop it into a folder by passing that folder's id. `+upload` always creates a new file. Every upload path must be absolute and inside the home directory. Relative and `/tmp` paths fail.
  上传用户提供的本地文件：`drive +upload ...`（其标志见 `drive +upload --help`）。先确认本地路径存在，并通过传入目标文件夹的 id 把文件放入该文件夹。`+upload` 总是创建新文件。上传路径必须是绝对路径且位于主目录内。相对路径和 `/tmp` 路径会失败。
- Replace an existing file's content, keeping its id and link: `drive files update --params '{"fileId":"<id>"}' --upload <absolute_path>`. Uploading an Office file into a Google-native Doc or Slides file converts it in place. The Docs and Slides skills own when to do that. Never upload into a Google Sheet. Google replaces the full contents, so the upload discards the user's other tabs. The Sheets skill formats through the Sheets API instead.
  替换既有文件的内容并保留其 id 和链接：`drive files update --params '{"fileId":"<id>"}' --upload <absolute_path>`。把 Office 文件上传进 Google 原生的 Doc 或 Slides 文件会原地转换。何时转换由 Docs 与 Slides 技能决定。绝不向 Google Sheet 上传。Google 会替换全部内容，因此这种上传会丢弃用户的其他工作表标签页。Sheets 技能改用 Sheets API 进行格式化。
- Download a stored binary file to a local path: `drive files get --params '{"fileId":"<id>","alt":"media"}' --output <path>`. The output path is the user's choice, so ask or reuse one they named.
  将存储的二进制文件下载到本地路径：`drive files get --params '{"fileId":"<id>","alt":"media"}' --output <path>`。输出路径由用户决定，因此要询问或复用用户指定的路径。

### Create and organize / 创建与整理

- New folder: `drive files create --params '{"ignoreDefaultVisibility":true}' --json '{"name":"Q3 Docs","mimeType":"application/vnd.google-apps.folder","parents":["<parent_folder_id>"]}'`. Use `"root"` as the parent for the top level. This opts out of domain-wide default visibility; the folder still inherits its parent's sharing.
  新建文件夹：`drive files create --params '{"ignoreDefaultVisibility":true}' --json '{"name":"Q3 Docs","mimeType":"application/vnd.google-apps.folder","parents":["<parent_folder_id>"]}'`。顶层以 `"root"` 作为父级。这样可退出域级默认可见性；文件夹仍继承其父级的共享设置。
- Rename: `drive files update --params '{"fileId":"<id>"}' --json '{"name":"Q3 Budget"}'`.
  重命名：`drive files update --params '{"fileId":"<id>"}' --json '{"name":"Q3 Budget"}'`。
- Move: `drive files update --params '{"fileId":"<id>","addParents":"<dest_folder_id>","removeParents":"<current_folder_id>"}'`. Read the current parent from a `files get` first so you remove the right one. Moves use sharing approval because the destination can grant access.
  移动：`drive files update --params '{"fileId":"<id>","addParents":"<dest_folder_id>","removeParents":"<current_folder_id>"}'`。先通过 `files get` 读取当前父级，确保移除的是正确的那个。移动使用共享审批，因为目标位置可能授予访问权限。
- Copy into My Drive: `drive files copy --params '{"fileId":"<id>","ignoreDefaultVisibility":true}' --json '{"name":"Copy of Q3 Budget","parents":["root"]}'`. Use the requested destination folder instead when specified. Omitting the destination can inherit the source's parent and uses shared-create approval.
  复制到 My Drive：`drive files copy --params '{"fileId":"<id>","ignoreDefaultVisibility":true}' --json '{"name":"Copy of Q3 Budget","parents":["root"]}'`。若用户指定了目标文件夹，则改用之。省略目标可能继承源文件的父级，并触发共享创建审批。
- Share: `drive permissions create --params '{"fileId":"<id>"}' --json '{"type":"user","role":"reader","emailAddress":"alex@example.com"}'`. Use `writer` only when the user asks for edit access.
  共享：`drive permissions create --params '{"fileId":"<id>"}' --json '{"type":"user","role":"reader","emailAddress":"alex@example.com"}'`。仅当用户要求编辑权限时才用 `writer`。

### Remove / 移除

- Prefer trashing, which the user can undo: `drive files update --params '{"fileId":"<id>"}' --json '{"trashed":true}'`. Restore with `{"trashed":false}`.
  优先移入回收站，用户可以撤销：`drive files update --params '{"fileId":"<id>"}' --json '{"trashed":true}'`。用 `{"trashed":false}` 恢复。
- Permanent delete cannot be undone: `drive files delete --params '{"fileId":"<id>"}'`. Only use it when the user clearly wants the item gone for good. Say plainly that it cannot be recovered; for a folder, also say that permanently deleting it removes the user's owned files and folders inside it.
  永久删除无法撤销：`drive files delete --params '{"fileId":"<id>"}'`。仅在用户明确要求彻底删除时使用。明确说明无法恢复；对文件夹，还要说明永久删除会连带移除其中用户拥有的文件和文件夹。

## Rules / 规则

- Follow the tool's approval flow. Changing a private item, trashing, restoring, and deleting may proceed from a clear, unambiguous user request. Moves and permission changes use sharing approval. Creating or copying without a verified private destination and explicit private visibility uses shared-create approval, including all `+upload` calls. My Drive placement alone does not rule out domain-wide default visibility. Confirm before creating in a shared folder or modifying an already shared item because other people can see the changes.
  遵循工具的审批流程。修改私有条目、移入回收站、恢复和删除，在用户清晰明确的要求下即可执行。移动与权限变更使用共享审批。在未经核实的私有目标位置和明确私有可见性的情况下进行创建或复制，使用共享创建审批，包括所有 `+upload` 调用。仅凭放在 My Drive 并不能排除域级默认可见性。在共享文件夹中创建或修改已共享条目之前先确认，因为其他人能看到这些更改。
- Prefer the reversible path. Trash instead of permanently deleting unless the user is explicit, and tell them trashed files can be restored.
  优先选择可逆路径。除非用户明确要求，否则移入回收站而非永久删除，并告知用户回收站中的文件可以恢复。
- Use only the file and folder ids returned by an earlier command. Never invent or rewrite an id.
  只使用先前命令返回的文件和文件夹 id。绝不编造或改写 id。
- Verify a local path before you upload from it or download to it. Never invent a path.
  在以其为源上传或为目标下载之前，先核实本地路径。绝不编造路径。
- Talk to the user in plain language only. Never show raw commands or JSON. Never show file or folder ids, etags, page tokens, connect or disconnect URLs, or status words like not_connected or unavailable, unless the user asks. Confirm what happened by the file's name and a `webViewLink` returned by Drive; fetch the field when needed and never construct a Drive URL. Keep ids only in your working context to chain the next command.
  对用户只使用平实语言。绝不显示原始命令或 JSON。除非用户要求，绝不显示文件或文件夹 id、etag、页面令牌、连接或断开 URL，以及 not_connected、unavailable 之类的状态词。用文件名和 Drive 返回的 `webViewLink` 确认发生了什么；需要时获取该字段，绝不自行构造 Drive URL。id 只保留在你的工作上下文中，用于串起下一条命令。
- Never print tokens, secrets, or credential material. Redact them if they appear in tool output.
  绝不打印令牌、机密或凭据材料。若它们出现在工具输出中，予以遮蔽。
- Read results retain Google's raw metadata timestamps and add semantic UTC and
  user-local forms for creation, modification, viewing, sharing, trashing, and
  change-record times.
  读取结果保留 Google 的原始元数据时间戳，并为创建、修改、查看、共享、回收站及变更记录时间补充语义化的 UTC 和用户本地时间形式。

## Limits / 限制

- Reading, editing, or exporting the contents of a Google Doc, Sheet, or Slides file is the job of those skills, not this one. Never run `drive files export` or otherwise pull a Google-native file's contents through Drive, even as a fallback when another skill's connection fails; say that opening its contents needs the Docs, Sheets, or Slides skill and stop. Drive handles the file itself: finding it, its details, and moving, sharing, or removing it. The binary download recipe above (`alt=media`) only fetches stored binary files, never Google-native ones.
  读取、编辑或导出 Google Doc、Sheet 或 Slides 文件的内容是那些技能的职责，不是本技能的。绝不运行 `drive files export`，也不通过 Drive 以其他方式获取 Google 原生文件的内容，即使在其他技能连接失败时作为回退也不行；说明打开其内容需要 Docs、Sheets 或 Slides 技能，然后停止。Drive 处理文件本身：查找文件、其详情，以及移动、共享或移除。上文的二进制下载方法（`alt=media`）只获取存储型二进制文件，绝不适用于 Google 原生文件。
- Comments, shared drives, and approval requests are not first-class flows here. If a task needs one, check its shape with `hatch_gws_cli schema drive.<resource>.<method>`; treat any write as shared and confirm before it, because there is no seeded, tested private-write recipe for those surfaces.
  评论、共享云端硬盘和审批请求不是本文档的一等流程。若任务需要用到，先用 `hatch_gws_cli schema drive.<resource>.<method>` 查其结构；把任何写入都当作共享写入，并在执行前确认，因为这些面没有经过预置和测试的私有写入配方。
