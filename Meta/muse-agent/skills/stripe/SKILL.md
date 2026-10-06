---
name: "stripe"
description: >-
  Manage a merchant's Stripe customers, products, coupons, promotion codes, payments,
  invoices, and recurring subscriptions. Use for billing-plan changes, payment links,
  refunds of duplicate card charges, disputes and chargebacks, account balances,
  payouts, and reconciling processing fees through Stripe's official MCP server.
icon: "connectorStripe"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Stripe / Stripe

Use the installed `stripe` CLI. Start with `stripe status`. If it reports
`not_connected`, run `stripe authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `stripe` CLI。先运行 `stripe status`。如果报告 `not_connected`，则运行 `stripe authorize-url`，并且只把返回的 `connect_url` 分享给用户。

OAuth uses Dynamic Client Registration and PKCE through authd. Stripe issues a
per-VM public client, so no shared credential enters Muse and credentials must
never be requested in chat.

OAuth 通过 authd 使用动态客户端注册（Dynamic Client Registration）与 PKCE。Stripe 按每个 VM 签发公共客户端，因此没有任何共享凭据进入 Muse，也绝不能在聊天中索要凭据。

【评论】per-VM 公共客户端加 PKCE 的设计使 Muse 侧不持有任何共享密钥，凭据暴露面收敛到用户本地环境。

Run `stripe list-tools` to inspect the live provider catalogue and schemas,
then call an advertised tool with:

运行 `stripe list-tools` 查看实时的提供方目录与 schema，然后用以下方式调用已公布的工具：

```text
stripe call-tool --name <tool> --arguments-json '<json-object>'
```

The OAuth connection identifies the merchant account. Never ask the user for
that connected account's `acct_...` ID. If another tool requires
`stripe_context`, first call the advertised `list_available_accounts_or_orgs`
tool with an empty arguments object and use a context it returns. If more than
one context matches the requested live or test mode, present their names and
ask the user which account to use; do not ask them to paste an account ID. Never
invent a context or call an account-scoped tool without one returned by Stripe.

OAuth 连接本身即标识商户账户。绝不向用户索要该已连接账户的 `acct_...` ID。如果其他工具需要 `stripe_context`，先用空的参数对象调用已公布的 `list_available_accounts_or_orgs` 工具，并使用它返回的某个 context。如果多个 context 都匹配所请求的 live 或 test 模式，列出它们的名称并让用户选择使用哪个账户；不要让用户粘贴账户 ID。绝不编造 context，也绝不在没有 Stripe 返回的 context 的情况下调用账户级工具。

`list-tools` exposes only reviewed Stripe tools and includes each tool's
`hatch_permission`, `hatch_action`, and `hatch_permission_label`. Unknown or
new provider tools remain unavailable until reviewed. For `stripe_api_write`,
the returned schema lists the reviewed `stripe_api_operation_id` values and
their operation-specific permission overrides; unlisted operation IDs are not
available. Read permissions follow the user's connector settings; Stripe API
writes and feedback require granular approval. Stripe may also require
confirmation through a provider URL for sensitive operations. Do not retry a
failed or timed-out write automatically because its side effect may have
completed.

`list-tools` 只暴露经过审查的 Stripe 工具，并包含每个工具的 `hatch_permission`、`hatch_action` 与 `hatch_permission_label`。未知或新的提供方工具在审查通过前保持不可用。对于 `stripe_api_write`，返回的 schema 列出了经过审查的 `stripe_api_operation_id` 值及其针对各操作的权限覆盖；未列出的操作 ID 不可用。读取权限遵循用户的连接器设置；Stripe API 写入与反馈需要细粒度审批。对敏感操作，Stripe 还可能要求通过提供方 URL 进行确认。不要自动重试失败或超时的写入，因为其副作用可能已经完成。

【评论】“不要自动重试写入”针对的是非幂等副作用：一次超时的写请求可能实际已生效，盲目重试会造成重复扣款或重复创建。
