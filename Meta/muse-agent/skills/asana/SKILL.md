---
name: "asana"
description: >-
  Plan and track team projects and tasks in Asana. Use to assign work to owners,
  update due dates and status, set task dependencies, and review project progress,
  comments, teams, and workspaces through Asana's official MCP server.
icon: "asana"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->

# Asana

Use the installed `asana` CLI. Start with `asana status`. If it reports
`not_connected`, run `asana authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `asana` CLI。先运行 `asana status`。如果它报告 `not_connected`，则运行 `asana authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses the fixed Muse Asana MCP app, PKCE, and Asana's MCP resource
indicator through authd. CAGI supplies the client secret for token exchange,
refresh, and revocation; the secret must never enter Muse or be requested in
chat. Do not request ordinary Asana API scopes because MCP apps reject them.

OAuth 通过 authd 使用固定的 Muse Asana MCP 应用、PKCE 以及 Asana 的 MCP 资源指示符（resource indicator）。CAGI 为令牌交换、刷新和撤销提供客户端密钥；该密钥绝不能进入 Muse，也绝不能在聊天中被索要。不要请求普通的 Asana API 作用域，因为 MCP 应用会拒绝它们。

Run `asana list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `asana list-tools` 查看实时的提供方工具目录及其模式，然后按以下方式调用已公布的工具：

```text
asana call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Asana tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Read permissions follow
the user's connector settings; changes require the corresponding granular
approval. Do not retry a failed or timed-out write automatically because its
side effect may have completed.

`list-tools` 只暴露经过审核的 Asana 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新的提供方工具在审核通过之前保持不可用。读取权限遵循用户的连接器设置；任何更改都需要相应的细粒度批准。不要自动重试失败或超时的写入操作，因为其副作用可能已经完成。
