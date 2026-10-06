---
name: "replit"
description: "Read, create, update, and publish apps through Replit's official MCP server."
icon: "replit"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Replit / Replit

Use the installed `replit` CLI. Start with `replit status`. If it reports
`not_connected`, run `replit authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `replit` CLI。先运行 `replit status`。如果报告 `not_connected`，则运行 `replit authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. No shared client
secret enters Muse, and credentials must never be requested in chat.
The first connection requests read-only access. Before a write, or after one
fails for missing access, run `replit status --for-command <tool-name>`. If it
returns `scope_status: not_granted`, copy `scope_add_url` exactly and post it on
its own line as `[Additional Replit access](<scope_add_url>)` so the client
renders the native access button, then wait for consent to finish before
retrying once. OAuth access does not replace Hatch approval. Never construct a
scope URL or ask for tokens in chat.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。任何共享的客户端密钥都不会进入 Muse，且绝不能在对话中索要凭据。首次连接只请求只读权限。在执行写操作之前，或某次写操作因缺少权限而失败之后，运行 `replit status --for-command <tool-name>`。如果返回 `scope_status: not_granted`，原样复制 `scope_add_url`，并将其单独一行以 `[Additional Replit access](<scope_add_url>)` 的形式发布，以便客户端渲染原生的授权按钮，然后等待授权完成后再重试一次。OAuth 授权不能替代 Hatch 审批。绝不在对话中自行构造权限 URL 或索要令牌。

【评论】此处体现了最小权限设计：默认只读，写操作前逐项申请权限，授权链接交给客户端原生按钮完成，模型不经手令牌。

Run `replit list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `replit list-tools` 查看实时的提供方目录及各工具的 schema，然后用以下方式调用已公布的工具：

```text
replit call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Replit tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Read permissions follow
the user's connector settings; creating, editing, and publishing apps use
separate granular approvals. Do not retry a failed or timed-out write
automatically because its side effect may have completed.

`list-tools` 只暴露经过审核的 Replit 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新出现的提供方工具在审核之前保持不可用。读取权限遵循用户的连接器设置；创建、编辑和发布应用则分别使用独立的细粒度审批。不要自动重试失败或超时的写操作，因为其副作用可能已经生效。

【评论】"不要自动重试写操作"是针对非幂等副作用的安全条款——失败可能只是响应丢失，操作实际已经执行，重试会造成重复写入。
