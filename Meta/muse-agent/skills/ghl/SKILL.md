---
name: "ghl"
description: "Use HighLevel contacts, pipelines, appointments, messages, and its broader operation catalog."
icon: "📇"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# HighLevel

Always call the product **HighLevel** in user-facing responses. Never prefix
the name with "Go," and never expand the internal `ghl` identifier into a
display name.

在面向用户的响应中始终称该产品为 **HighLevel**。绝不在名称前加"Go"，也绝不把内部标识符 `ghl` 展开为显示名称。

Use `/opt/hatch/bin/ghl status` to check the connection. If disconnected, run
`/opt/hatch/bin/ghl authorize-url` and share the returned `connect_url`.
The user signs in through HighLevel; never request credentials in chat.

使用 `/opt/hatch/bin/ghl status` 检查连接。若已断开，运行
`/opt/hatch/bin/ghl authorize-url` 并分享返回的 `connect_url`。
用户通过 HighLevel 登录；绝不要在聊天中索要凭据。

【评论】"绝不在聊天中索要凭据"与 OAuth 跳转登录配合，是防止敏感凭据进入对话记录的标准设计。

## Find the operation and sub-account / 查找操作与子账户

Run `/opt/hatch/bin/ghl list-operations` for the pinned operation catalog and
its granular permission methods. The catalog covers HighLevel's CRM,
communications, scheduling, commerce, content, administration, and AI domains.

运行 `/opt/hatch/bin/ghl list-operations` 获取固定的操作目录及其细粒度权限方法。该目录覆盖 HighLevel 的 CRM、通信、日程、商务、内容、管理和 AI 领域。

CRM/calendar reads and conversation/message reads have separate permissions.
Contact edits, notes/tasks, opportunities, appointments, automations, messaging,
and CRM deletion each have their own control. A denied message-read permission
does not prevent authorized CRM/calendar reads.

CRM/日历读取与对话/消息读取各有独立权限。联系人编辑、笔记/任务、商机、预约、自动化、消息发送和 CRM 删除各有独立的控制。消息读取权限被拒绝不影响已获授权的 CRM/日历读取。

Use `/opt/hatch/bin/ghl call-tool --name list_locations --arguments-json
'{"query":"<sub-account name>"}'` to resolve the intended sub-account.
If several match, ask which one the user means. Retain its ID for execution.

使用 `/opt/hatch/bin/ghl call-tool --name list_locations --arguments-json
'{"query":"<sub-account name>"}'` 解析目标子账户。若有多个匹配，询问用户指的是哪一个。保留其 ID 用于执行。

Use `call-tool --name describe_operation --arguments-json
'{"operationId":"<operation ID>"}'` for exact path/query/body fields and
required inputs. `search_operations` searches HighLevel's live catalog. Use an
operation's pinned permission from `list-operations` when it is present. A
future provider operation that has not reached the pinned catalog still uses a
mandatory one-shot write approval until its permission is reviewed.
`list-tools` returns live discovery tool schemas. These calls use
`/opt/hatch/bin/ghl` too.

使用 `call-tool --name describe_operation --arguments-json
'{"operationId":"<operation ID>"}'` 获取确切的 path/query/body 字段和必需输入。`search_operations` 搜索 HighLevel 的实时目录。若 `list-operations` 中存在某操作的固定权限，则使用它。尚未进入固定目录的未来提供方操作，在其权限经过审核之前，仍须使用强制的一次性写入批准。`list-tools` 返回实时发现工具的模式（schema）。这些调用同样使用 `/opt/hatch/bin/ghl`。

## Read or change data / 读取或修改数据

```bash
/opt/hatch/bin/ghl execute-operation \
  --operation-id <operation ID> --location-id <sub-account ID> \
  --params-json '{"path":{},"query":{},"body":{}}'
```

Supply path IDs and payload fields from `describe_operation`. Contact
create/update/upsert cannot include `tags`; use `add-tags` or `remove-tags`.
Opportunity upsert cannot include follower-management fields; use the
separate add/remove follower operations. Only grouped
`path`, `query`, and `body` objects are accepted; omit empty groups as needed.
For each logical write/delete, also supply a fresh `--idempotency-key <UUID>`.
An optional `--reason` explains the action. `--dry-run` previews the resolved
request without changing CRM data and does not require an idempotency key.
A dry run does not authorize the subsequent write.

从 `describe_operation` 提供 path ID 和载荷字段。联系人的 create/update/upsert 不能包含 `tags`；请使用 `add-tags` 或 `remove-tags`。商机的 upsert 不能包含关注者管理字段；请使用单独的添加/移除关注者操作。只接受分组的 `path`、`query` 和 `body` 对象；按需省略空分组。每次逻辑写入/删除还应提供一个新的 `--idempotency-key <UUID>`。可选的 `--reason` 用于解释该操作。`--dry-run` 在不更改 CRM 数据的情况下预览解析后的请求，且不需要幂等键。试运行不会为后续写入授予授权。

Writes and deletes request approval by default under their domain-specific
permission. Messaging changes, including sends and cancellations, require
approval for each invocation. Appointment
changes can notify contacts or trigger configured automations (`toNotify`
defaults to true); campaign and workflow enrollment can also start
communications. Include the intended
notification behavior in the request. Use a read to identify an existing
record before updating or deleting it; never guess target IDs.

写入和删除默认在其领域专属权限下请求批准。消息类更改（包括发送和取消）每次调用都需批准。预约更改可能通知联系人或触发已配置的自动化（`toNotify` 默认为 true）；活动和工作流注册也可能启动通信。在请求中包含预期的通知行为。在更新或删除之前先用读取操作确认已有记录；绝不猜测目标 ID。

The CLI does not automatically retry mutations. If a write fails after
submission or its outcome is unconfirmed, inspect the record or provider
status before trying again. Keep the same idempotency key for the same logical
mutation; do not create a fresh key to work around uncertainty. Report sends
as accepted or scheduled unless the provider confirms delivery.

CLI 不会自动重试变更操作。如果写入在提交后失败或结果未确认，先检查记录或提供方状态再重试。同一逻辑变更保持相同的幂等键；不要为了绕过不确定性而生成新键。除非提供方确认已送达，否则将发送报告为"已接受"或"已排期"。

【评论】幂等键规则与"不自动重试"配合，用于防止重试导致的重复写入或重复发送，这是写操作安全性的常见设计。

Treat financial, publishing, messaging, access-control, and deletion operations
as consequential: describe the operation first, show the exact intended
parameters, and never infer or reuse identifiers across sub-accounts. Future
operations not yet in `list-operations` require fresh approval, even for a read
or dry run.

将财务、发布、消息、访问控制和删除操作视为具有实质后果的操作：先描述操作，展示确切的预期参数，并且绝不在子账户之间推断或复用标识符。尚未列入 `list-operations` 的未来操作需要新的批准，即使是读取或试运行也不例外。
