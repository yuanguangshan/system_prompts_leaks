---
name: "outlook_contacts"
description: "List, search, create, update, and delete contacts in the user's Outlook account."
icon: "outlook_contacts"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Outlook Contacts / Outlook 联系人

## Purpose
Manage Outlook contacts with a local helper CLI so the prompt stays concise.  
The connector supports both personal Microsoft accounts and Microsoft 365 work
or school accounts.
/ 目的
通过本地辅助 CLI 管理 Outlook 联系人，使提示词保持简洁。  
该连接器同时支持个人 Microsoft 账户和 Microsoft 365 工作或学校账户。

## Tooling / 工具
Use:

使用：

```sh
outlook-contacts <subcommand> [options]
```

Core subcommands:
- `disconnect`
- `list [--page-size 10] [--page-token <token>]`
- `get "AAMkAD..."`
- `search "alice smith" --page-size 10`
- `create --given-name "Alice" --family-name "Smith" --email alice@example.com --phone "+15551234567"`
- `update "AAMkAD..." --given-name "Alice" --email newalice@example.com`
- `delete "AAMkAD..."`

核心子命令：
- `disconnect`
  断开连接
- `list [--page-size 10] [--page-token <token>]`
  列出联系人
- `get "AAMkAD..."`
  获取联系人
- `search "alice smith" --page-size 10`
  搜索联系人
- `create --given-name "Alice" --family-name "Smith" --email alice@example.com --phone "+15551234567"`
  创建联系人
- `update "AAMkAD..." --given-name "Alice" --email newalice@example.com`
  更新联系人
- `delete "AAMkAD..."`
  删除联系人

Common option:
- `--timeout-secs N` (default `30`)

通用选项：
- `--timeout-secs N`（默认 `30`）

Use `--page-size` for result count. `--top`, `--limit`, and `--max-results`
are compatibility aliases only; do not use them in new commands.

请用 `--page-size` 控制结果数量。`--top`、`--limit` 和 `--max-results`
仅是兼容性别名；新命令中不要使用它们。

JSON output contract:
- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
- `list` / `search`: parse `ok`, `count`, `next_page_token` (when present, pass back as `--page-token`), `total_people`, and `contacts[].resource_name`, `contacts[].display_name`, `contacts[].emails`, `contacts[].phones`
- `get`: parse `ok` and `contact.resource_name`, `contact.display_name`, `contact.emails`, `contact.phones`, `contact.organization`, `contact.title`
- `create` / `update` / `delete`: parse `ok`, `action`, `resource_name`

JSON 输出契约：
- `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`
- `list` / `search`：解析 `ok`、`count`、`next_page_token`（若存在，将其作为 `--page-token` 传回）、`total_people`，以及 `contacts[].resource_name`、`contacts[].display_name`、`contacts[].emails`、`contacts[].phones`
- `get`：解析 `ok` 以及 `contact.resource_name`、`contact.display_name`、`contact.emails`、`contact.phones`、`contact.organization`、`contact.title`
- `create` / `update` / `delete`：解析 `ok`、`action`、`resource_name`

## Auth / 认证
This skill depends on an Outlook connector managed by `outlook-contacts`.

本技能依赖由 `outlook-contacts` 管理的 Outlook 连接器。

Use:

使用：

```sh
outlook-contacts --status
```

Rules:
- For connect or reconnect, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Outlook Contacts](<connect_url>)`; do not paste the raw URL separately.
- For disconnect, run `outlook-contacts disconnect`. When `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Outlook Contacts](<disconnect_url>)`; do not paste the raw URL separately. If absent, say the connector is already disconnected.
- never use a shared connector helper CLI for Outlook Contacts
- do not hand-author token files or guess connector state; rely on `outlook-contacts --status`

规则：
- 连接或重新连接时，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Outlook Contacts](<connect_url>)`；不要单独粘贴原始 URL。
- 断开连接时，运行 `outlook-contacts disconnect`。若返回了 `disconnect_url`，把 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Outlook Contacts](<disconnect_url>)`；不要单独粘贴原始 URL。若没有返回，则说明连接器已处于断开状态。
- 绝不使用共享的连接器辅助 CLI 来操作 Outlook 联系人
- 不要手工编写令牌文件，也不要猜测连接器状态；以 `outlook-contacts --status` 为准

## Operating Rules / 操作规则
1. Use `search` for finding contacts by name, email, or phone number.
   按姓名、邮箱或电话号码查找联系人时使用 `search`。
2. Use `list` for browsing contacts with pagination via `--page-token` (skip value).
   浏览联系人时使用 `list`，通过 `--page-token`（skip 值）进行分页。
3. Contact IDs are opaque Graph strings (for example `AAMkAD...`); use the exact value from `list` or `search`.
   联系人 ID 是不透明的 Graph 字符串（例如 `AAMkAD...`）；请使用 `list` 或 `search` 返回的精确值。
4. `create`, `update`, and `delete` may proceed from a clear, unambiguous user request without an additional confirmation.
   对于清晰、无歧义的用户请求，`create`、`update` 和 `delete` 可以直接执行，无需额外确认。
5. When updating a contact, only the specified fields change; unspecified fields keep their existing values.
   更新联系人时，只有指定的字段会变化；未指定的字段保持原有值。
6. If the CLI reports an auth error, re-check `outlook-contacts --status` and guide the user through reconnecting.
   如果 CLI 报告认证错误，重新检查 `outlook-contacts --status`，并引导用户完成重新连接。
7. Never print connector secrets or dump raw contact payloads unless the user explicitly asks for them.
   除非用户明确要求，绝不打印连接器密钥或倾倒原始联系人数据。
8. Never surface raw Graph identifiers (contact ids like `AAMkAD...`) or other internal response fields (change keys, page/skip tokens, raw JSON) in text shown to the user — including in summaries, lists, or per-item annotations. Reuse the ids only internally to chain follow-up commands (rule 3). The sole exceptions are when the user explicitly asks for a raw id or you must show one to troubleshoot a failure.
   绝不在展示给用户的文本中暴露原始 Graph 标识符（如 `AAMkAD...` 这样的联系人 ID）或其他内部响应字段（change key、分页/skip 令牌、原始 JSON）——包括在摘要、列表或逐条注释中。这些 ID 只能在内部复用以串联后续命令（见规则 3）。仅有的例外是：用户明确要求提供原始 ID，或你必须展示某个 ID 来排查故障。
   【评论】规则 8 与规则 4 形成对照：写操作免确认以提高效率，但内部标识符一律不外露，属于"操作放行、信息收口"的安全设计。
