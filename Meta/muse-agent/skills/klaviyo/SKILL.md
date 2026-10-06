---
name: "klaviyo"
description: >-
  Read and manage the user's Klaviyo email and SMS marketing campaigns, newsletters,
  flows, subscriber lists, and audience segments. Use to compare campaign
  performance and revenue, find recent customers, plan audience targeting, and
  manage marketing content and subscriptions through Klaviyo's official MCP server.
icon: "klaviyo"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Klaviyo

Use the installed `klaviyo` CLI. Connecting requests access to the full
supported catalog. Writes, sends, and deletions require Hatch approval.
Settings group the catalog into eight capabilities: reading, marketing
content, campaign delivery, flows, audiences and subscriptions, catalogs and
coupons, tracking and integrations, and deletion. Editing content does not
grant permission to send campaigns or delete data. Each approval still
previews the specific operation and inputs. Campaign creation and campaign/message
edits show the submitted audience, sender, subject, and message when present.
Campaign send/cancel and flow status/action approvals also show available
Klaviyo metadata; lookup failures
retain the original inputs. Audience IDs and recipient estimates are shown
when available, not inferred audience names or guaranteed delivery counts.

使用已安装的 `klaviyo` CLI。建立连接时请求访问完整受支持目录。写入、发送和删除操作需要 Hatch 批准。设置把目录划分为八项能力：读取、营销内容、广告系列投放、flows（自动化流）、受众与订阅、目录与优惠券、追踪与集成，以及删除。编辑内容不等于获得发送广告系列或删除数据的权限。每次批准仍会预览具体操作及其输入。创建广告系列以及编辑广告系列/消息时，若存在已提交的受众、发件人、主题和消息内容，则会予以展示。广告系列发送/取消以及 flow 状态/操作的批准还会展示可用的 Klaviyo 元数据；查找失败时保留原始输入。在可用时展示受众 ID 和收件人估算值，而不是推断的受众名称或有保证的送达数量。

【评论】将"编辑内容"与"发送/删除"拆分为不同权限，并把异步发送作业的状态回读作为报告前提，属于对高风险营销动作的分级授权设计。

Run `klaviyo list-tools` for the compact reviewed tool catalog. These names and
their Hatch permissions are available before connection. Run `klaviyo
list-tools --name <tool-name>` for one reviewed tool's live input schema; the
provider's output schemas are intentionally omitted to keep discovery bounded.
Do not guess tool names or argument schemas.

运行 `klaviyo list-tools` 获取精简的已审查工具目录。这些名称及其 Hatch 权限在连接之前即可获得。运行 `klaviyo
list-tools --name <tool-name>` 可获取某个已审查工具的实时输入 schema；提供方的输出 schema 被有意省略，以使发现过程保持有界。不要猜测工具名称或参数 schema。

If `klaviyo status` reports `auth_status: not_connected`, run
`klaviyo authorize-url` and share only the returned `connect_url`. Klaviyo
requires an Owner, Admin, or Manager role. Do not construct OAuth URLs or
request tokens in chat.

如果 `klaviyo status` 报告 `auth_status: not_connected`，运行 `klaviyo authorize-url` 并只分享返回的 `connect_url`。Klaviyo 要求账户具备 Owner、Admin 或 Manager 角色。不要在对话中手工构造 OAuth URL 或请求令牌。

```text
klaviyo call-tool --help
klaviyo status
klaviyo list-tools [--name <tool-name>]
klaviyo call-tool --name <tool-name> --arguments-json '<JSON object>'
klaviyo account-details
klaviyo list-campaigns --channel <email|sms|mobile-push> [--page-cursor <cursor>]
klaviyo list-flows [--page-cursor <cursor>] [--page-size <1-100>]
klaviyo list-metrics [--page-cursor <cursor>]
```

The catalog covers stable remote campaign, flow, audience, subscription,
template, image, catalog, event, metric, reporting, coupon, tag, webhook, form,
review, push-token, and profile-deletion tools. Beta and local-only tools are
excluded. Klaviyo's MCP supports sending campaigns; Hatch's explicit allowlist
controls which tools can be called here. If `list-tools --name` reports that a
reviewed tool is unavailable, check the connection before concluding it is
unsupported. Do not invent a schema or bypass the CLI with a direct API call.

该目录涵盖稳定的远程工具：广告系列、flow、受众、订阅、模板、图片、目录、事件、指标、报告、优惠券、标签、webhook、表单、评论、推送令牌以及资料删除工具。Beta 工具和仅限本地使用的工具不在其中。Klaviyo 的 MCP 支持发送广告系列；Hatch 的显式允许列表控制哪些工具可在此处调用。如果 `list-tools --name` 报告某个已审查工具不可用，先检查连接状态，再断定它不受支持。不要凭空编造 schema，也不要绕过 CLI 直接发起 API 调用。

- Campaigns: read back the audience, message content, and schedule before
  `send_campaign`. Sending and cancellation share a campaign-delivery
  permission, separate from content editing. A successful request starts an
  asynchronous job; check `get_campaign_send_job` and
  `get_campaign` before reporting the outcome. `cancel_campaign_send` can
  cancel or revert a send to draft where Klaviyo permits it; read back status.
  Cancellation cannot recall delivered mail. A created draft is not sent.
  广告系列：在 `send_campaign` 之前回读受众、消息内容和排期。发送与取消共用一项广告系列投放权限，独立于内容编辑。请求成功后启动的是异步作业；在报告结果之前，先检查 `get_campaign_send_job` 和 `get_campaign`。在 Klaviyo 允许的情况下，`cancel_campaign_send` 可以取消发送或将发送回退为草稿；之后要回读状态。取消无法撤回已送达的邮件。创建的草稿不会被发送。
- Flows: `create_flow` uses an encoded flow definition; preserve returned IDs.
  `update_flow` changes the flow status AND all its actions. Read `get_flow`,
  `get_flow_action`, and `get_flow_message` before and after changes. Making a
  flow or action live can start delivering messages.
  Flows：`create_flow` 使用编码后的 flow 定义；保留返回的 ID。`update_flow` 会更改 flow 状态及其全部操作。在修改前后都要读取 `get_flow`、`get_flow_action` 和 `get_flow_message`。将某个 flow 或操作置为启用状态可能开始发送消息。
- Audiences: list membership does not grant or revoke marketing consent; use
  subscription tools for consent changes. Adding list members, subscribing
  profiles, or creating events can trigger flows. Check the relevant flows
  before those changes.
  受众：加入列表成员身份不会授予或撤销营销许可；许可变更应使用订阅工具。添加列表成员、订阅资料或创建事件都可能触发 flows。在这些变更之前，先检查相关 flows。
- Bulk changes, merges, and deletions: confirm the target set and consequences.
  Profile deletion is permanent. Preserve returned job IDs and use the matching
  job-status read before claiming completion. After any uncertain write or
  deletion result, inspect state before retrying.
  批量变更、合并与删除：确认目标集合及其后果。资料删除是永久性的。保留返回的作业 ID，并在宣称完成之前用对应的作业状态读取进行确认。任何写入或删除结果不明确时，先检查状态再重试。

`assign_template_to_campaign_message` returns a new, message-owned template
copy whose ID differs from the source. Do not retry because of that substitution;
verify with `get_campaign` using `include=campaign-messages`. Preserve returned
Klaviyo UI links when present.

`assign_template_to_campaign_message` 返回一个新的、归属于该消息的模板副本，其 ID 与源模板不同。不要因为这个替换而重试；应使用 `get_campaign` 并带上 `include=campaign-messages` 进行验证。返回的 Klaviyo UI 链接在存在时予以保留。
