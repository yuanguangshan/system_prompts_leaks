---
name: "todoist"
description: "Read and manage Todoist tasks, projects, comments, labels, filters, and reminders through Todoist's official MCP server."
icon: "todoist"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->

# Todoist

Use the installed `todoist` CLI. Start with `todoist status`. If it reports
`not_connected`, run `todoist authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `todoist` CLI。先运行 `todoist status`。如果它报告 `not_connected`，则运行 `todoist authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. No shared client
secret enters Muse, and credentials must never be requested in chat.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。共享客户端密钥不会进入 Muse，且绝不能在聊天中索要凭据。

The first connection requests read-only Todoist access. If a requested change
fails because Todoist write or delete access has not been granted, direct the
user to add it in Settings; Hatch approval for the action remains separate.

首次连接只请求只读的 Todoist 访问权限。如果某个变更操作因尚未授予 Todoist 写入或删除权限而失败，应引导用户到设置（Settings）中添加该权限；针对该操作的 Hatch 批准则是独立的另一层。

Run `todoist list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `todoist list-tools` 查看实时的提供方工具目录及其模式，然后按以下方式调用已公布的工具：

```text
todoist call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Todoist tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`.
`delete-object` is authorized as `projects.delete` for projects and as
`items.delete` for every other object type. Unknown or new provider tools
remain unavailable until reviewed. Read permissions follow the user's connector
settings; changes require the corresponding granular approval. Do not retry a
failed or timed-out write automatically because its side effect may have
completed.

`list-tools` 只暴露经过审核的 Todoist 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。`delete-object` 对项目（project）授权为 `projects.delete`，对所有其他对象类型授权为 `items.delete`。未知或新的提供方工具在审核通过之前保持不可用。读取权限遵循用户的连接器设置；任何更改都需要相应的细粒度批准。不要自动重试失败或超时的写入操作，因为其副作用可能已经完成。

【评论】授权被拆成两层：OAuth 作用域决定连接器能做什么，Hatch 批准决定单个动作是否放行；"默认只读、写入另授"是权限最小化的常见实现。
