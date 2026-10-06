---
name: "outlook_calendar"
description: "View, create, update, and delete events in the user's Outlook Calendar."
icon: "outlook_calendar"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Outlook Calendar / Outlook 日历

## Purpose / 用途
Manage Outlook Calendar events with the `outlook-calendar` companion CLI. The
connector supports both personal Microsoft accounts and Microsoft 365 work or
school accounts.

使用 `outlook-calendar` 配套 CLI 管理 Outlook 日历事件。该连接器同时支持个人 Microsoft 账户和 Microsoft 365 工作或学校账户。

## Tooling / 工具
Use `exec` to run the installed CLI directly from `PATH`.

使用 `exec` 直接从 `PATH` 运行已安装的 CLI。

```sh
outlook-calendar --status
outlook-calendar disconnect
outlook-calendar list --page-size 10 [--time-min <RFC3339>] [--time-max <RFC3339>] [--query <text>] [--page-token <token>]
outlook-calendar get "<event_id>"
outlook-calendar create --summary "<title>" --start "<RFC3339-or-date>" --end "<RFC3339-or-date>" --timezone "<iana_tz>" [--attendee <email>] [--location <text>] [--description <text>] [--all-day]
outlook-calendar update "<event_id>" [--summary <title>] [--start <RFC3339>] [--end <RFC3339>] [--timezone <iana_tz>] [--location <text>] [--description <text>]
outlook-calendar delete "<event_id>"
```

For `list`, use `--page-size` for result count. `--top`, `--limit`, and
`--max-results` are compatibility aliases only; do not use them in new
commands. `list` uses `--time-min` / `--time-max` for time bounds. `--start`
and `--end` are canonical only for `create` / `update`; `list` accepts them
only as compatibility aliases.

对于 `list`，使用 `--page-size` 指定结果数量。`--top`、`--limit` 和 `--max-results` 只是兼容性别名，不要在新命令中使用它们。`list` 使用 `--time-min` / `--time-max` 作为时间边界。`--start` 和 `--end` 只对 `create` / `update` 是规范参数；`list` 仅将它们作为兼容性别名接受。

JSON output contract:

JSON 输出契约：

- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`.
  `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`。
- `list`: parse `ok`, `count`, `next_page_token` (when present, pass back as `--page-token`), `retrieved_at`, and `events[]` with fields such as `id`, `summary`, `description`, `start`, `end`, `event_starts_at`, `event_ends_at`, `location`, `is_all_day`, and `attendees`.
  `list`：解析 `ok`、`count`、`next_page_token`（若存在，作为 `--page-token` 传回）、`retrieved_at`，以及包含 `id`、`summary`、`description`、`start`、`end`、`event_starts_at`、`event_ends_at`、`location`、`is_all_day`、`attendees` 等字段的 `events[]`。
- `get`: parse `ok`, `retrieved_at`, and `event`.
  `get`：解析 `ok`、`retrieved_at` 和 `event`。
- `create` / `update`: parse `ok`, `action`, `event_id`, and `web_link`.
  `create` / `update`：解析 `ok`、`action`、`event_id` 和 `web_link`。
- `delete`: parse `ok`, `action`, and `event_id`.
  `delete`：解析 `ok`、`action` 和 `event_id`。

## Auth / 认证
This skill uses the CLI-managed Outlook connector. Do not hand-write auth files.

本技能使用由 CLI 管理的 Outlook 连接器。不要手写认证文件。

First-use or reconnect flow:

首次使用或重新连接流程：

1. Run `outlook-calendar --status`.
   运行 `outlook-calendar --status`。
2. If the response includes `connect_url`, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Outlook Calendar](<connect_url>)`; do not paste the raw URL separately. Wait for the user to finish linking.
   如果响应包含 `connect_url`，将 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Outlook Calendar](<connect_url>)`；不要单独粘贴原始 URL。等待用户完成关联。
3. Re-run the same status command before continuing.
   在继续之前重新运行同一状态命令。
4. If the response includes `disconnect_url`, the account is already connected.
   如果响应包含 `disconnect_url`，说明账户已经连接。

For disconnect requests, run `outlook-calendar disconnect`. When `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Outlook Calendar](<disconnect_url>)`; do not paste the raw URL separately. If it is absent, say the account is already disconnected.

对于断开连接的请求，运行 `outlook-calendar disconnect`。若存在 `disconnect_url`，将 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Outlook Calendar](<disconnect_url>)`；不要单独粘贴原始 URL。若不存在，则告知用户账户已经断开连接。

## Operating Rules / 操作规则
1. Use `list` first when you need an event ID or when the user asks about a date range.
   当你需要事件 ID，或用户询问某个日期范围时，先使用 `list`。
2. All timed values must be RFC3339 / ISO 8601. Use date-only values only with `--all-day`.
   所有带时间的值必须采用 RFC3339 / ISO 8601 格式。仅日期的值只能与 `--all-day` 搭配使用。
3. Always supply a valid IANA timezone on `create`, and on `update` whenever start or end times change.
   `create` 时始终提供有效的 IANA 时区；`update` 中只要开始或结束时间发生变化也要提供。
4. Confirm with the user before creating an event that sends invitations.
   在创建会发送邀请的事件之前，先与用户确认。
5. Private event updates may proceed from a clear user request. The helper reads
   the current event before updating it; organizer-owned meetings with attendees
   require confirmation because Outlook sends meeting-update email.
   私有事件的更新可以在明确的用户请求下进行。辅助工具会在更新前读取当前事件；由组织者持有且有与会人的会议需要确认，因为 Outlook 会发送会议更新邮件。
6. The current helper does not support attendee changes on `update`. If the user asks to add or remove invitees on an existing event, explain that limitation instead of emitting an unsupported `--attendee` flag.
   当前辅助工具不支持在 `update` 中更改与会人。如果用户要求在既有事件上添加或移除受邀者，应解释这一限制，而不要发出不受支持的 `--attendee` 标志。
7. `delete` may proceed from a clear, unambiguous user request without an additional confirmation.
   只要用户请求清晰明确，`delete` 即可执行，无需额外确认。
8. Event IDs are opaque Graph values. Reuse the exact `id` returned by `list` or `get`.
   事件 ID 是不透明的 Graph 值。复用 `list` 或 `get` 返回的确切 `id`。
9. Never use the shared connector helper CLI for Outlook Calendar; status and linking must go through `outlook-calendar --status`.
   绝不要将共享的连接器辅助 CLI 用于 Outlook 日历；状态查询和关联必须通过 `outlook-calendar --status` 进行。
10. Never surface raw Graph identifiers (event ids, calendar ids) or other internal response fields (change keys, page/skip tokens, raw JSON) in text shown to the user — including in summaries, lists, or per-item annotations. Reuse the ids only internally to chain follow-up commands (rule 8). The sole exceptions are when the user explicitly asks for a raw id or you must show one to troubleshoot a failure.
    绝不要在展示给用户的文本中露出原始 Graph 标识符（事件 id、日历 id）或其他内部响应字段（change key、分页/跳过 token、原始 JSON）——包括在摘要、列表或逐条注释中。这些 id 只能在内部复用，用于串联后续命令（规则 8）。唯一的例外是用户明确索要原始 id，或你必须展示某个 id 来排查故障。
11. For timed events, prefer `event_starts_at.user_local` and `event_ends_at.user_local`; the raw Graph fields remain for compatibility. All-day values are calendar dates, not instants, and must not be timezone-shifted.
    对于定时事件，优先使用 `event_starts_at.user_local` 和 `event_ends_at.user_local`；原始 Graph 字段仅为兼容性保留。全天事件的值是日历日期而非时间点，绝不能做时区偏移。
- Permission-withheld fields: a `list` result carrying a `withheld` entry had
  organizer and attendees removed by the user's "Access events" permission —
  the meeting is NOT guest-less. Answer from the remaining fields and mention
  that the permission hides the guest list. Only when the user actually needs
  a withheld field for one specific event and `withheld.reason` is
  `requires_approval`, read that event with `get`, which shows the user the
  approval prompt. Never `get` during routine browsing just to fill guest
  lists.
  权限隐藏字段：`list` 结果中若带有 `withheld` 条目，表示组织者和与会人已被用户的"访问事件"（Access events）权限移除——该会议并非没有参会人。请基于其余字段作答，并说明该权限隐藏了参会人列表。只有当用户确实需要某个具体事件的被隐藏字段、且 `withheld.reason` 为 `requires_approval` 时，才用 `get` 读取该事件，这会向用户展示审批提示。绝不要在日常浏览时为了填充参会人列表而调用 `get`。

【评论】规则 10 禁止向用户展示原始 Graph 标识符等内部字段，属于面向用户的输出净化约束；末尾的 `withheld` 条款则规定了权限隐藏数据的读取须以用户触发审批为前提。
