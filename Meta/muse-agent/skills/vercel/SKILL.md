---
name: "vercel"
description: >-
  Inspect and manage the user's Vercel websites, web apps, projects, and
  deployments. Use to read build and runtime logs, investigate production errors
  and failed deployments, check deployment status, and deploy projects through
  Vercel's official MCP server.
icon: "vercel"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Vercel / Vercel

Use the installed `vercel` CLI. Start with `vercel status`. If it reports
`not_connected`, run `vercel authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `vercel` CLI。先运行 `vercel status`。如果它报告 `not_connected`，则运行 `vercel authorize-url`，并只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. Vercel declares
the client public and issues no client secret, so no shared credential enters
Muse and credentials must never be requested in chat. Vercel exposes only
identity and session OAuth scopes for this MCP server, so read-only-by-default
behavior is enforced by the connector permissions below rather than a narrower
provider scope.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）和 PKCE。Vercel 将该客户端声明为公共客户端且不签发客户端密钥，因此没有任何共享凭据进入 Muse，也绝不能在聊天中索要凭据。Vercel 为此 MCP 服务器只暴露身份和会话 OAuth 作用域，所以"默认只读"的行为由下文的连接器权限强制保证，而非依赖更窄的提供方作用域。
【评论】公共客户端无密钥方案意味着安全性不依赖机密保管，而由本地权限层兜底，这是对提供方作用域粒度不足的补偿设计。

Run `vercel list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `vercel list-tools` 查看实时的提供方目录与模式（schema），然后用以下方式调用已通告的工具：

```text
vercel call-tool --name <tool> --arguments-json '<json-object>'
```

The reviewed catalogue follows Vercel's current public MCP tool documentation,
including `create_deployment`. The former `deploy_to_vercel` name and other
recently replaced names remain accepted only when the live server still
advertises them during rollout. Always use the live schema returned by
`list-tools`; for example, current `create_deployment` arguments place the
deployment definition under `requestBody`.

已评审的目录遵循 Vercel 当前的公开 MCP 工具文档，包括 `create_deployment`。旧的 `deploy_to_vercel` 名称以及其他近期被替换的名称，仅在灰度期间实时服务器仍在通告时才被接受。始终使用 `list-tools` 返回的实时模式；例如，当前 `create_deployment` 的参数把部署定义放在 `requestBody` 之下。

`list-tools` exposes only reviewed Vercel tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. Ordinary reads follow
the user's connector settings. Decrypted secrets, deployment or sandbox file
contents, deployments, purchases, credential creation, security changes,
sandbox execution, and other mutations use separate granular permissions that
ask by default. Vercel advertises identity/session OAuth scopes only, so there
is no provider-side incremental scope to request for an individual tool.

`list-tools` 只暴露经过评审的 Vercel 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 和 `hatch_permission_label`。未知或新的提供方工具在评审通过前保持不可用。普通读取遵循用户的连接器设置。解密密钥、部署或沙箱文件内容、部署、购买、凭据创建、安全变更、沙箱执行及其他变更类操作使用单独的细粒度权限，默认都会询问。Vercel 只通告身份/会话 OAuth 作用域，因此不存在可针对单个工具请求的提供方侧增量作用域。

Purchase tools can create immediate, non-refundable charges. Obtain a quote
when the live tool contract provides one, show it to the user, and never infer
confirmation. Do not retry any failed or timed-out write automatically because
its side effect may have completed.

购买类工具可能产生立即生效且不可退款的费用。当实时工具契约提供报价时，先获取报价并展示给用户，绝不要自行推断已获确认。不要自动重试任何失败或超时的写操作，因为其副作用可能已经完成。
【评论】"不自动重试写操作"针对的是非幂等接口：超时并不等于未执行，重试可能造成重复扣费或重复部署。
