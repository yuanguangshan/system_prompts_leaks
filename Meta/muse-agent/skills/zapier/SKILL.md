---
name: "zapier"
description: "Connect Muse to actions across apps through Zapier's official MCP server."
icon: "zapier"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->

# Zapier

Use the installed `zapier` CLI. Start with `zapier status`. If it reports
`not_connected`, run `zapier authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `zapier` CLI。先运行 `zapier status`。如果它报告 `not_connected`，则运行 `zapier authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. No shared client
secret enters Muse, and credentials must never be requested in chat. Zapier's
MCP authorization exposes identity scopes rather than separate read and write
scopes, so read-only-by-default behavior is enforced by connector permissions.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。共享客户端密钥不会进入 Muse，且绝不能在聊天中索要凭据。Zapier 的 MCP 授权只暴露身份（identity）作用域，而不区分读取和写入作用域，因此"默认只读"的行为由连接器权限来强制执行。

Run `zapier list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `zapier list-tools` 查看实时的提供方工具目录及其模式，然后按以下方式调用已公布的工具：

```text
zapier call-tool --name <tool> --arguments-json '<json-object>'
```

`list-tools` exposes only reviewed Zapier agentic-mode tools and includes each
tool's `hatch_permission`, `hatch_action`, and `hatch_permission_label`.
Unknown or dynamic managed-mode tools remain unavailable until reviewed. Read
permissions follow the user's connector settings; executing write actions and
changing configuration use granular approvals. Do not retry a failed or
timed-out write automatically because its side effect may have completed.

`list-tools` 只暴露经过审核的 Zapier 代理模式（agentic-mode）工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或动态的托管模式（managed-mode）工具在审核通过之前保持不可用。读取权限遵循用户的连接器设置；执行写入操作和更改配置需要细粒度批准。不要自动重试失败或超时的写入操作，因为其副作用可能已经完成。

【评论】由于 Zapier 的授权模型不提供读写分离的作用域，"默认只读"只能下沉到连接器权限层实现，这一点与作用域天然分离的服务不同，属于值得注意的权限设计权衡。
