---
name: "box"
icon: "box"
description: "Search, read, upload, download, move, rename, delete, restore, and share Box content; manage comments and metadata."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Box

Use `/opt/hatch/bin/box-cli` to access Box.

使用 `/opt/hatch/bin/box-cli` 访问 Box。

## Connect / 连接

Run `/opt/hatch/bin/box-cli status`. If disconnected, run
`/opt/hatch/bin/box-cli authorize-url` and share the returned `connect_url`
as **Connect Box**. A fresh connection requests read-only access. Wait for
browser consent, then check status again.

运行 `/opt/hatch/bin/box-cli status`。如果已断开，运行
`/opt/hatch/bin/box-cli authorize-url`，并把返回的 `connect_url` 以 **Connect Box** 分享。新建立的连接只请求只读权限。等待浏览器授权完成，然后再次检查状态。

Before an operation that may need broader Box access, run
`/opt/hatch/bin/box-cli status --for-command <command-or-method>`. For a hosted
MCP operation, pass its exact tool name instead of `call-tool`. When
`scope_status` is `not_granted`, copy the returned `scope_add_url` exactly and
post it on its own line as  
`[Additional Box access](<scope_add_url>)` so the client renders the native
access button. Wait for consent, then retry the original operation. Never
construct an authorization URL or substitute a raw Box scope. The operation
also fails closed with the same method-bound link if its required scope is
absent. OAuth access does not replace Muse approval.

在执行可能需要更广泛 Box 权限的操作之前，运行
`/opt/hatch/bin/box-cli status --for-command <command-or-method>`。对于托管 MCP 操作，传入其确切的工具名称而不是 `call-tool`。当 `scope_status` 为 `not_granted` 时，原样复制返回的 `scope_add_url`，并单独成行发布为  
`[Additional Box access](<scope_add_url>)`，以便客户端渲染原生的授权按钮。等待授权完成，然后重试原操作。绝不自行构造授权 URL，也不要替换为原始的 Box scope。如果所需 scope 缺失，该操作也会以相同的方法绑定链接失败关闭（fail closed）。OAuth 授权不能替代 Muse 审批。

【评论】新连接默认只申请只读 scope、扩展权限需按操作单独申请，体现了最小权限原则；"OAuth 不替代 Muse 审批"则把服务授权与代理操作审批分成两层。

## Use / 使用

Run `list-tools` for available MCP tools, their argument schemas, and Muse
permissions. Use those schemas and IDs returned by Box in `call-tool`.
`get_file_content` reads document text. Downloads and changes ask for Muse
approval by default.

运行 `list-tools` 查看可用的 MCP 工具、其参数模式（schema）和 Muse 权限。在 `call-tool` 中使用这些模式以及 Box 返回的 ID。`get_file_content` 读取文档文本。下载和更改默认需要 Muse 审批。

```sh
/opt/hatch/bin/box-cli list-tools
/opt/hatch/bin/box-cli call-tool --name who_am_i --arguments-json '{}'

# Browse the root folder.
/opt/hatch/bin/box-cli call-tool --name list_folder_content_by_folder_id \
  --arguments-json '{"folder_id":"0","limit":20}'

# Search for PDFs.
/opt/hatch/bin/box-cli call-tool --name search_files_keyword \
  --arguments-json '{"query":"project plan","file_extensions":["pdf"],"limit":10}'

# Inspect file metadata.
/opt/hatch/bin/box-cli call-tool --name get_file_details \
  --arguments-json '{"file_id":"<file_id>","fields":["name","size","permissions"]}'

# Read document text.
/opt/hatch/bin/box-cli call-tool --name get_file_content \
  --arguments-json '{"file_id":"<file_id>"}'

# Create a folder.
/opt/hatch/bin/box-cli call-tool --name create_folder \
  --arguments-json '{"name":"Project notes","parent_folder_id":"<folder_id>"}'

# Upload a text file.
/opt/hatch/bin/box-cli call-tool --name upload_file \
  --arguments-json '{"file_name":"notes.txt","file_content":"Meeting notes","parent_folder_id":"<folder_id>"}'

# Post a comment.
/opt/hatch/bin/box-cli call-tool --name create_file_comment \
  --arguments-json '{"file_id":"<file_id>","message":"Ready for review."}'
```

Use the file commands below for local-file uploads, full downloads, moves,
renames, and deletion. `update-folder` and `delete-folder` take `--folder-id`.
Use `upload-version` to replace an existing file's contents; its required
`--name` also renames the file. Run `<command> --help` for options.

本地文件上传、完整下载、移动、重命名和删除请使用下面的文件命令。`update-folder` 和 `delete-folder` 接受 `--folder-id`。使用 `upload-version` 替换已有文件的内容；其必需的 `--name` 同时会重命名该文件。运行 `<command> --help` 查看选项。

```sh
/opt/hatch/bin/box-cli download --file-id <file_id> --output workspace/report.pdf
/opt/hatch/bin/box-cli upload --input workspace/report.pdf --name report.pdf --parent-folder-id <folder_id>
/opt/hatch/bin/box-cli update-file --file-id <file_id> --name renamed.pdf --parent-folder-id <folder_id>
/opt/hatch/bin/box-cli delete-file --file-id <file_id>
```

Downloads overwrite the output file. Deletion is permanent when Box trash is
disabled; deleting a nonempty folder requires `--recursive`.
`list-trash` returns retained items; pass `next_marker` as `--marker` to page.
Use `restore-file` or `restore-folder` to recover them. Their
`--fallback-parent-folder-id` applies only if the original parent no longer exists.

下载会覆盖输出文件。当 Box 回收站被禁用时，删除是永久性的；删除非空文件夹需要 `--recursive`。
`list-trash` 返回保留的项目；把 `next_marker` 作为 `--marker` 传入以分页。
使用 `restore-file` 或 `restore-folder` 恢复它们。它们的
`--fallback-parent-folder-id` 仅在原始父级已不存在时生效。

Downloads report the saved path and byte count; other results appear under
`result`. `ok: false` means the operation failed. REST commands make one attempt:
retry a download later if Box returns HTTP 202 (file not ready), and run
`refresh` on HTTP 401 before retrying. Check whether a failed change was applied
before retrying it.
Do not retry an access-denied operation through another command.

下载会报告保存路径和字节数；其他结果显示在 `result` 之下。`ok: false` 表示操作失败。REST 命令只尝试一次：如果 Box 返回 HTTP 202（文件未就绪），稍后重试下载；遇到 HTTP 401 时先运行 `refresh` 再重试。在重试失败的更改之前，先检查它是否已被应用。
不要通过其他命令重试被拒绝访问的操作。

`/opt/hatch/bin/box-cli refresh` refreshes the connection.
`/opt/hatch/bin/box-cli disconnect` disconnects Box from Muse.

`/opt/hatch/bin/box-cli refresh` 刷新连接。
`/opt/hatch/bin/box-cli disconnect` 将 Box 与 Muse 断开。
