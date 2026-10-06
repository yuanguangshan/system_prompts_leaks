---
name: "dropbox"
description: >-
  Search, read, upload, organize, and share files and folders in the user's
  Dropbox cloud storage. Use to upload or download files, collect client uploads
  with file requests, inspect or create shared links, and create, copy, move, or
  delete content through Dropbox's official APIs.
icon: "dropbox"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Dropbox / Dropbox

Use the installed `dropbox` CLI. Start with `dropbox status`. If it reports
`not_connected`, run `dropbox authorize-url` and share only the returned
`connect_url` with the user. After connecting, run `dropbox list-tools` to get
the live input schemas for the reviewed Dropbox MCP catalogue.

使用已安装的 `dropbox` CLI。先运行 `dropbox status`。如果报告 `not_connected`，则运行 `dropbox authorize-url`，并且只把返回的 `connect_url` 分享给用户。连接之后，运行 `dropbox list-tools` 获取经过审查的 Dropbox MCP 目录的实时输入 schema。

The first connection requests read-only OAuth access. Before a write, or after
one fails for missing access, run `dropbox status --for-command <tool-name>`.
If it returns `scope_status: not_granted`, copy `scope_add_url` exactly and post
it on its own line as `[Additional Dropbox access](<scope_add_url>)` so the
client renders the native access button, then wait for consent to finish before
retrying once. OAuth access does not replace Hatch approval. Never construct a
scope URL or ask for tokens in chat.

首次连接只请求只读 OAuth 权限。在执行写入之前，或某次写入因权限不足而失败之后，运行 `dropbox status --for-command <tool-name>`。如果返回 `scope_status: not_granted`，则原样复制 `scope_add_url`，并把它单独成行地发布为 `[Additional Dropbox access](<scope_add_url>)`，以便客户端渲染原生的授权按钮，然后等待授权完成后再重试一次。OAuth 授权不能替代 Hatch 审批。绝不自行构造 scope URL，也绝不在聊天中索要令牌。

【评论】要求以固定格式的 Markdown 链接触发客户端原生授权按钮，属于为代理输出预定义的受控 UI 通道，避免自由文本链接被挪作他用。

```text
dropbox list-tools
dropbox call-tool --name <tool-name> --arguments-json '<JSON object>'
dropbox call-tool --name <tool-name> --arguments-json '<JSON object>' --output <path>
dropbox create-file --path <dropbox-path> --input <local-path>
```

The reviewed catalogue supports listing, searching, reading, and downloading
files; file and account metadata; inspecting and creating shared links and file
requests; and creating, copying, moving, deleting, or sharing content. Follow the
schema returned by
`dropbox list-tools` exactly. Never call a tool that is absent from that list.

经过审查的目录支持：列出、搜索、读取和下载文件；文件与账户元数据；查看和创建共享链接与文件请求（file requests）；以及创建、复制、移动、删除或共享内容。严格遵循 `dropbox list-tools` 返回的 schema。绝不调用不在该列表中的工具。

Use `create-file` to create or replace a Dropbox file from a local file. It
accepts text and binary files up to 150 MiB.

使用 `create-file` 从本地文件创建或替换 Dropbox 文件。它接受最大 150 MiB 的文本与二进制文件。

File requests collect uploads from other people; they do not upload a local file
from this VM. The CLI does not expose revision history or version restore.

文件请求（file requests）用于收集他人的上传；它不能从本 VM 上传本地文件。该 CLI 不提供修订历史或版本恢复功能。

Use `--output` when the selected tool returns binary content. Never request or
expose temporary download links, OAuth tokens, or Dropbox app credentials in
chat. Confirm destructive intent before deleting, moving, or overwriting
content.

当所选工具返回二进制内容时使用 `--output`。绝不在聊天中请求或暴露临时下载链接、OAuth 令牌或 Dropbox 应用凭据。在删除、移动或覆盖内容之前，先确认破坏性操作的意图。
