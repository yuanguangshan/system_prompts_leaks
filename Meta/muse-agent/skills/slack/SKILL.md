---
name: "slack"
description: >-
  Read and search messages, channels, and threads in the user's Slack workspace.
  Use to catch up on team discussions and decisions, post channel messages, and
  reply to coworkers through Slack's official MCP server.
icon: "slack"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->

# Slack

Use the installed `slack` CLI. Start with `slack status`. If it reports
`not_connected`, run `slack authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `slack` CLI。先运行 `slack status`。如果它报告 `not_connected`，则运行 `slack authorize-url`，并且只把返回的 `connect_url` 分享给用户。

Slack requires a fixed registered app identity and does not support Dynamic  
Client Registration. The test app is limited to its development workspace;  
the production app must be approved for Slack Marketplace distribution before
users from other workspaces can connect.

Slack 要求使用固定的已注册应用身份，不支持动态客户端注册（Dynamic Client Registration）。测试应用仅限于其开发工作区；生产应用必须先通过 Slack Marketplace 分发审核，其他工作区的用户才能连接。

Run `slack list-tools` to inspect the reviewed subset of the live provider
catalogue and its permission annotations, then call an advertised tool with:

运行 `slack list-tools` 查看实时提供方目录中经过审核的子集及其权限注解，然后按以下方式调用已公布的工具：

```text
slack call-tool --name <tool> --arguments-json '<json-object>'
```

Slack's catalogue contains both reads and mutations and can change over time.
The CLI allows only explicitly reviewed tools: read operations are allowed by
default and mutations require approval with an argument preview. Unknown or new
provider tools fail closed. Direct file reads, signed upload URLs, and unreviewed
list tools remain unavailable until they have dedicated delivery handling.
Never request OAuth tokens or app credentials in chat.

Slack 的工具目录同时包含读取和变更（mutation）操作，且会随时间变化。CLI 只允许经过明确审核的工具：读取操作默认放行，变更操作需要附带参数预览的批准。未知或新的提供方工具采取"默认拒绝"（fail closed）策略。直接文件读取、签名上传 URL 以及未经审核的列表工具，在获得专门的交付处理之前保持不可用。绝不要在聊天中索要 OAuth 令牌或应用凭据。

【评论】读取默认放行、写入需带参数预览批准、未知工具一律 fail closed，构成"只读宽松、写入收紧"的分级权限模型，是第三方平台目录动态变化下的常见安全取舍。
