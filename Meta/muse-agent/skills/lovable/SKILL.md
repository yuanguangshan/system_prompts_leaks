---
name: "lovable"
description: >-
  Read, create, edit, and deploy web apps and projects in Lovable. Use to turn a
  product brief into a working website or prototype, inspect existing project
  code, update pages and features, and publish changes through Lovable's official
  MCP server.
icon: "lovable"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Lovable / Lovable

Use the installed `lovable` CLI. Start with `lovable status`. If it reports
`not_connected`, run `lovable authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `lovable` CLI。先运行 `lovable status`。如果报告 `not_connected`，则运行 `lovable authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Lovable's fixed Muse client and PKCE through authd. CAGI supplies the
client secret for token exchange and refresh; it never enters Muse and must
never be requested in chat.
The first connection requests read-only access. Before a write, or after one
fails for missing access, run `lovable status --for-command <tool-name>`. If it
returns `scope_status: not_granted`, copy `scope_add_url` exactly and post it on
its own line as `[Additional Lovable access](<scope_add_url>)` so the client
renders the native access button, then wait for consent to finish before
retrying once. OAuth access does not replace Hatch approval. Never construct a
scope URL or ask for tokens in chat.

OAuth 通过 authd 使用 Lovable 的固定 Muse 客户端和 PKCE。客户端密钥由 CAGI 提供，用于令牌交换和刷新；它绝不进入 Muse，也绝不能在对话中索要。首次连接只请求只读权限。在执行写操作之前，或某次写操作因缺少权限而失败之后，运行 `lovable status --for-command <tool-name>`。如果返回 `scope_status: not_granted`，原样复制 `scope_add_url`，并将其单独一行以 `[Additional Lovable access](<scope_add_url>)` 的形式发布，以便客户端渲染原生的授权按钮，然后等待授权完成后再重试一次。OAuth 授权不能替代 Hatch 审批。绝不在对话中自行构造权限 URL 或索要令牌。

【评论】与 replit 技能不同，此处客户端密钥由独立服务 CAGI 托管、仅用于令牌交换与刷新，模型运行时环境（Muse）完全接触不到密钥，属于密钥隔离设计。

Run `lovable list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `lovable list-tools` 查看实时的提供方目录及各工具的 schema，然后用以下方式调用已公布的工具：

```text
lovable call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Lovable tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Read permissions follow
the user's connector settings; edits, publishes, and database operations use
separate granular approvals. Do not retry a failed or timed-out write
automatically because its side effect may have completed.

`list-tools` 只暴露经过审核的 Lovable 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新出现的提供方工具在审核之前保持不可用。读取权限遵循用户的连接器设置；编辑、发布和数据库操作分别使用独立的细粒度审批。不要自动重试失败或超时的写操作，因为其副作用可能已经生效。
