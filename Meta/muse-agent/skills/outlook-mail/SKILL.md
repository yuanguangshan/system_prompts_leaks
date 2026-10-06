---
name: "outlook_mail"
description: "Read, search, send, reply to, and delete messages in the user's Outlook Mail."
icon: "outlook_mail"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Outlook Mail / Outlook 邮件

## Purpose / 用途
Manage Outlook Mail messages with the `outlook-mail` companion CLI. The
connector supports both personal Microsoft accounts and Microsoft 365 work or
school accounts.

使用 `outlook-mail` 配套 CLI 管理 Outlook 邮件消息。该连接器同时支持个人 Microsoft 账户和 Microsoft 365 工作或学校账户。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
outlook-mail <subcommand> [options]
```

Core subcommands:

核心子命令：

- `disconnect`
- `list [--page-size 10] [--unread] [--page-token 20] [--folder sentitems]`
- `get "<MESSAGE_ID>"`
- `search "<query>" [--folder sentitems] [--page-size 10]`
- `send --to alice@example.com [--to bob@example.com] [--cc charlie@example.com] --subject "Hello" --body "Hi ..." [--attachment /path/to/file.pdf]`
- `reply "<MESSAGE_ID>" --body "Thanks for the update!" [--reply-all]`
- `delete "<MESSAGE_ID>"` (moves the message to Deleted Items; recoverable, not a permanent delete)
  `delete "<MESSAGE_ID>"`（将消息移入"已删除项目"（Deleted Items）文件夹；可恢复，不是永久删除）
- `mark-read "<MESSAGE_ID>"`
- `mark-unread "<MESSAGE_ID>"`

Use `--page-size` for result count. `--top`, `--limit`, and `--max-results`
are compatibility aliases only; do not use them in new commands. Use
`get "<MESSAGE_ID>"` for a single message; `read`, `get --id`, and
`search --query` are compatibility forms only.

使用 `--page-size` 指定结果数量。`--top`、`--limit` 和 `--max-results` 只是兼容性别名，不要在新命令中使用它们。获取单条消息用 `get "<MESSAGE_ID>"`；`read`、`get --id` 和 `search --query` 只是兼容形式。

JSON output contract:

JSON 输出契约：

- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
  `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`
- `list` / `search`: parse `ok`, `count`, `next_page_token` (when present, pass back as `--page-token`), `total_messages`, `retrieved_at`, and `messages[]` with `id`, `subject`, `from`, `to`, `date`, `message_received_at`, `preview`, `is_read`, and `has_attachments`
  `list` / `search`：解析 `ok`、`count`、`next_page_token`（若存在，作为 `--page-token` 传回）、`total_messages`、`retrieved_at`，以及带有 `id`、`subject`、`from`、`to`、`date`、`message_received_at`、`preview`、`is_read`、`has_attachments` 的 `messages[]`
- `get`: parse `ok`, `retrieved_at`, and `message` with `id`, `subject`, `from`, `to`, `cc`, `date`, `message_received_at`, `body`, `body_type`, `is_read`, and `has_attachments`
  `get`：解析 `ok`、`retrieved_at`，以及带有 `id`、`subject`、`from`、`to`、`cc`、`date`、`message_received_at`、`body`、`body_type`、`is_read`、`has_attachments` 的 `message`
- `send`: parse `ok` and `action`
  `send`：解析 `ok` 和 `action`
- `reply`: parse `ok`, `action`, and `message_id`
  `reply`：解析 `ok`、`action` 和 `message_id`
- `delete`: parse `ok`, `action` (`trashed`), and `message_id`; the moved message gets a **new** id, so `message_id` is not the id you passed in — use the returned one for any follow-up command
  `delete`：解析 `ok`、`action`（`trashed`）和 `message_id`；被移动的消息会获得一个**新的** id，因此 `message_id` 不是你传入的那个 id——后续命令请使用返回的 id
- `mark-read` / `mark-unread`: parse `ok`, `action`, and `message_id`
  `mark-read` / `mark-unread`：解析 `ok`、`action` 和 `message_id`

## Auth / 认证
Use `outlook-mail --status` for connector state and link management. The binary handles its callback target internally.

使用 `outlook-mail --status` 管理连接器状态和关联。该二进制程序在内部处理其回调目标。

- If the user wants to connect or reconnect, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Outlook Mail](<connect_url>)`; do not paste the raw URL separately.
  如果用户想要连接或重新连接，将 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Outlook Mail](<connect_url>)`；不要单独粘贴原始 URL。
- If the user wants to disconnect, run `outlook-mail disconnect`. When `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Outlook Mail](<disconnect_url>)`; do not paste the raw URL separately. Otherwise say it is already disconnected.
  如果用户想要断开连接，运行 `outlook-mail disconnect`。若存在 `disconnect_url`，将 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Outlook Mail](<disconnect_url>)`；不要单独粘贴原始 URL。否则告知用户已经断开连接。
- Keep status and linking inside `outlook-mail --status`; do not use a shared connector helper CLI.
  状态查询和关联保持在 `outlook-mail --status` 内进行；不要使用共享的连接器辅助 CLI。
- Never print tokens, cookies, or connector secrets.
  绝不打印 token、cookie 或连接器机密。

## Operating Rules / 操作规则
1. Use `search` for targeted lookup, `list` for browsing, and `get` only when you need the full body of a specific message.
   定向查找用 `search`，浏览用 `list`，只有需要某条消息的完整正文时才用 `get`。
2. Message IDs are opaque Graph values. Reuse the exact `id` returned by `list` or `search`.
   消息 ID 是不透明的 Graph 值。复用 `list` 或 `search` 返回的确切 `id`。
3. Before `send`, confirm recipients, subject, and body in the current thread.
   在 `send` 之前，在当前会话中确认收件人、主题和正文。
4. Before `reply`, confirm the reply body and whether the user wants `--reply-all`. Use `--reply-all` only when the user explicitly wants everyone included.
   在 `reply` 之前，确认回复正文以及用户是否需要 `--reply-all`。仅当用户明确希望包含所有人时才使用 `--reply-all`。
5. `delete`, `mark-read`, and `mark-unread` may proceed from a clear user request without an additional confirmation. `delete` moves the message to the Deleted Items folder, where the user can still recover it; tell the user that, and do not describe it as permanent or unrecoverable.
   只要用户请求清晰，`delete`、`mark-read` 和 `mark-unread` 即可执行，无需额外确认。`delete` 会把消息移入"已删除项目"文件夹，用户仍可在那里找回；要把这一点告诉用户，不要把它描述为永久删除或不可恢复。
6. For mailbox-summary requests, exclude likely spam, phishing, or irrelevant bulk promotions by default unless the user explicitly asks for junk or spam, and briefly note that filtering if you used it.
   对于邮箱摘要类请求，默认排除疑似垃圾邮件、钓鱼邮件或无关的批量促销，除非用户明确要求查看垃圾邮件；如果使用了过滤，要简要说明。
7. If the CLI reports an auth failure or disconnected state, stop and route the user through the `--status` connect flow before retrying.
   如果 CLI 报告认证失败或已断开状态，先停止，引导用户走 `--status` 连接流程，然后再重试。
8. Never surface raw Graph identifiers (message ids, conversation ids) or other internal response fields (change keys, `@odata` fields, page/skip tokens, raw JSON) in text shown to the user — including in summaries, lists, or per-item annotations. Reuse the ids only internally to chain follow-up commands (rule 2). The sole exceptions are when the user explicitly asks for a raw id or you must show one to troubleshoot a failure.
   绝不要在展示给用户的文本中露出原始 Graph 标识符（消息 id、会话 id）或其他内部响应字段（change key、`@odata` 字段、分页/跳过 token、原始 JSON）——包括在摘要、列表或逐条注释中。这些 id 只能在内部复用，用于串联后续命令（规则 2）。唯一的例外是用户明确索要原始 id，或你必须展示某个 id 来排查故障。
9. The compatibility `date` field is Outlook's message-received time. Prefer `message_received_at.user_local` when presenting it. It is not the time of an event described inside the email; never infer a delivery, payment, trip, meeting, or other event time from it.
   兼容字段 `date` 是 Outlook 的消息接收时间。呈现时优先使用 `message_received_at.user_local`。它不是邮件中所描述事件的发生时间；绝不要从它推断交付、付款、行程、会议或其他事件的时间。
- Permission-withheld fields: a `list`/`search` result carrying a `withheld`
  entry had those fields removed by the user's "Access messages" permission —
  they are NOT empty. Answer from the remaining fields and mention that the
  permission hides the rest. Only when the user actually needs a withheld
  field for one specific message and `withheld.reason` is `requires_approval`,
  read that message with `get`, which shows the user the approval prompt.
  Never `get` during routine browsing just to fill previews.
  权限隐藏字段：`list`/`search` 结果中若带有 `withheld` 条目，表示那些字段已被用户的"访问消息"（Access messages）权限移除——它们并不是空的。请基于其余字段作答，并说明该权限隐藏了其余内容。只有当用户确实需要某条具体消息的被隐藏字段、且 `withheld.reason` 为 `requires_approval` 时，才用 `get` 读取该消息，这会向用户展示审批提示。绝不要在日常浏览时为了填充预览而调用 `get`。

【评论】规则 5 要求明确告知删除可恢复、不得描述为永久删除，属于对删除语义的诚实性约束；规则 8/9 与 outlook-calendar 技能一致，禁止外露内部标识符并严格限定时间字段的语义。
