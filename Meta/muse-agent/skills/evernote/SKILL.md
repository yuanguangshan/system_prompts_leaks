---
name: "evernote"
description: "Read and create notes through Evernote's official MCP server."
icon: "evernote"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->

# Evernote

Use the installed `evernote` CLI. Start with `evernote status`. If it reports
`not_connected`, run `evernote authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `evernote` CLI。先运行 `evernote status`。如果它报告 `not_connected`，则运行 `evernote authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. No shared client
secret enters Muse, and credentials must never be requested in chat.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。共享客户端密钥不会进入 Muse，且绝不能在聊天中索要凭据。

Run `evernote list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `evernote list-tools` 查看实时的提供方工具目录及其模式，然后按以下方式调用已公布的工具：

```text
evernote call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Evernote tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Read permissions follow
the user's connector settings; creating notes requires the corresponding
granular approval. Do not retry a failed or timed-out write automatically
because its side effect may have completed.

`list-tools` 只暴露经过审核的 Evernote 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新的提供方工具在审核通过之前保持不可用。读取权限遵循用户的连接器设置；创建笔记需要相应的细粒度批准。不要自动重试失败或超时的写入操作，因为其副作用可能已经完成。

【评论】"未知工具须经审核才可用"体现了对 MCP 工具目录的白名单式治理；"写入失败不自动重试"则是针对非幂等副作用（可能已实际完成）的防重复操作条款。
