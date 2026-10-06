---
name: "linear"
description: >-
  Read and manage software issues, tickets, projects, and planning data in Linear.
  Use to review sprints and cycles, identify ticket rollover and release blockers,
  and update issue status, assignees, priorities, and milestones through Linear's
  official MCP server.
icon: "linear"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Linear / Linear

Use the installed `linear` CLI. Start with `linear status`. If it reports
`not_connected`, run `linear authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `linear` CLI。先运行 `linear status`。如果它报告 `not_connected`，则运行 `linear authorize-url`，并只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. No shared client
secret enters Muse, and credentials must never be requested in chat.
The first connection requests read-only access. Before a write, or after one
fails for missing access, run `linear status --for-command <permission>`. For
Linear's `save_*` tools, use the matching `*.create` permission when the
arguments omit `id`, and the matching `*.manage` permission when they include
`id`. If the status command
returns `scope_status: not_granted`, copy `scope_add_url` exactly and post it on
its own line as `[Additional Linear access](<scope_add_url>)` so the client
renders the native access button, then wait for consent to finish before
retrying once. OAuth access does not replace Hatch approval. Never construct a
scope URL or ask for tokens in chat.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。没有任何共享客户端密钥进入 Muse，也绝不能在聊天中索要凭据。首次连接只请求只读访问。在执行写操作之前，或某次写操作因缺少权限而失败之后，运行 `linear status --for-command <permission>`。对于 Linear 的 `save_*` 工具，参数中不含 `id` 时使用对应的 `*.create` 权限，含 `id` 时使用对应的 `*.manage` 权限。如果 status 命令返回 `scope_status: not_granted`，则原样复制 `scope_add_url`，并单独成行以 `[Additional Linear access](<scope_add_url>)` 的形式发出，让客户端渲染原生的访问按钮，然后等用户同意完成后再重试一次。OAuth 授权不能替代 Hatch 审批。绝不自行构造作用域 URL，也绝不在聊天中索要令牌。
【评论】默认只读、写操作再逐步申请授权，叠加独立于 OAuth 的 Hatch 审批层，是双层的最小权限设计。

Run `linear list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `linear list-tools` 查看实时的提供方目录与模式，然后用以下方式调用已通告的工具：

```text
linear call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Linear tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. For a
polymorphic `save_*` tool, the permission fields show the `id`-based selector
used at execution. Linear hides write tools from its live catalogue while the
OAuth grant is read-only; do not describe an all-read `tools` list as lack of
write support. In that state, `list-tools` returns manifest-derived
`additional_access` entries separately from the provider-advertised tools. When
the user requests one of those capabilities, share its `scope_add_url` using
the exact Additional Linear access link format above, wait for consent, then
rerun `linear list-tools` to obtain the provider's live write-tool schema.
Unknown or newly advertised provider tools remain unavailable until reviewed.
Read permissions follow the user's connector settings; changes require the
corresponding granular approval. Do not retry a failed or timed-out write
automatically because its side effect may have completed.

`list-tools` 只暴露经过评审的 Linear 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。对于多态的 `save_*` 工具，权限字段显示的是执行时使用的基于 `id` 的选择器。当 OAuth 授权仍为只读时，Linear 会在其实时目录中隐藏写工具；不要把一个全为读取的 `tools` 列表描述成"不支持写入"。在该状态下，`list-tools` 会把来自清单（manifest）的 `additional_access` 条目与提供方通告的工具分开返回。当用户请求其中某项能力时，按上文的确切 Additional Linear access 链接格式分享其 `scope_add_url`，等待用户同意，然后重新运行 `linear list-tools` 以获取提供方实时的写工具模式。未知或新通告的提供方工具在评审通过前保持不可用。读取权限遵循用户的连接器设置；变更类操作需要对应的细粒度批准。不要自动重试失败或超时的写操作，因为其副作用可能已经完成。
