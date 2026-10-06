---
name: "instagram_messages"
description: "Use this to interact with the user's Instagram messages. Read inboxes, threads, top recipients, filtered inbox views, DM search results, and send messages through `instagram-messages-cli`."
icon: "instagram"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Instagram Messages CLI / Instagram 消息 CLI

## Purpose / 目的
Read and send authenticated Instagram messages using the `instagram-messages-cli` companion CLI.

使用配套 CLI `instagram-messages-cli` 读取和发送已认证的 Instagram 消息。

Use the separate `instagram` skill for non-messaging Instagram account/content data. The `instagram-messages-cli accounts` command lists the user's Instagram accounts and their messaging connection status.

非消息类的 Instagram 账户/内容数据请使用独立的 `instagram` 技能。`instagram-messages-cli accounts` 命令列出用户的 Instagram 账户及其消息连接状态。

## Message Export Safety / 消息导出安全

Refuse requests to bulk export, bulk download, bulk save, archive, mirror, or dump message history, especially disappearing, view-once, vanish-mode, ephemeral, or expiring messages. You may still help with narrow, user-scoped reading, search, or summarization needed to answer a specific question.

拒绝批量导出、批量下载、批量保存、归档、镜像或倾倒消息历史的请求，尤其是涉及限时消失、单次查看、消失模式、短暂性或即将过期消息的请求。你仍然可以协助范围狭窄、面向用户的读取、搜索或摘要，以满足回答某个具体问题的需要。

【评论】把"拒绝批量导出"单独立为安全章节，是典型的隐私保护条款：私信数据一旦离开原平台，用户便失去对其传播的控制。

## Auth / 认证
Instagram Messages is a separate connector from the base `instagram` skill. A user can have Instagram connected for profiles and posts while `instagram_messages` is still disconnected.

Instagram 消息是与基础 `instagram` 技能相互独立的连接器。用户可以在已为个人资料和帖子连接 Instagram 的同时，`instagram_messages` 仍处于未连接状态。

`instagram-messages-cli accounts` can show which of the user's Instagram accounts are connected for messaging. Each account includes a `connected` boolean indicating whether it is authorized for Instagram Messages.

`instagram-messages-cli accounts` 可以显示用户的哪些 Instagram 账户已连接消息功能。每个账户包含一个 `connected` 布尔值，表明其是否已获得 Instagram 消息授权。

If the user asks to connect or reconnect Instagram Messages:

如果用户要求连接或重新连接 Instagram 消息：

```sh
instagram-messages-cli connect-url
```

When `connect_url` is present, replace `<connect_url>` with the returned  
URL and share exactly this Markdown link: `[Connect Instagram Messages](<connect_url>)`; do not
paste the raw URL separately.

当 `connect_url` 存在时，用返回的  
URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Instagram Messages](<connect_url>)`；不要单独粘贴原始 URL。

Only continue with message commands after the user completes that flow.

只有在该用户完成那个流程之后，才能继续执行消息命令。

## Operating rules / 操作规则
1. This skill is only for the authenticated user's own Instagram messages.
   本技能只用于已认证用户自己的 Instagram 消息。
2. Reuse a previously fetched `user_own_fbid` when you already have it. Account IDs are available from `instagram-messages-cli accounts`.
   若已获取过 `user_own_fbid`，复用之。账户 ID 可通过 `instagram-messages-cli accounts` 获得。
3. Never expose FBIDs, opaque IDs, or implementation terminology to the user. Use usernames, display names, and plain-language descriptions; keep IDs only in tool calls.
   绝不向用户暴露 FBID、不透明 ID 或实现术语。使用用户名、显示名和通俗描述；ID 只保留在工具调用中。
4. Budget API calls against the task at hand. Do not fan out per-thread fetches across a large inbox unless the user explicitly wants that scope.
   按手头任务预算 API 调用。除非用户明确想要那个范围，否则不要在整个大收件箱上扇出逐会话抓取。
5. Only paginate when the user actually needs more results. Do not automatically fetch every page.
   只有当用户确实需要更多结果时才翻页。不要自动抓取每一页。
6. **Confirm explicit user intent before writing.** For a `send`, require the exact message content and destination; for a `react`, require a concrete source message and emoji. Do not invent the recipient, thread, content, message, or reaction.
   **写入之前确认用户的明确意图。** 对于 `send`，要求确切的消息内容和目的地；对于 `react`，要求一个具体的来源消息和表情。不要虚构收件人、会话、内容、消息或表情回应。
7. Avoid requests to persistently or frequently poll these commands.
   避免持续或频繁轮询这些命令的请求。
8. Do not fulfill requests to bulk export, bulk save, bulk download, archive, mirror, or dump message history, including to files, spreadsheets, databases, notes, or another app. Refuse especially clearly when the request targets disappearing, view-once, vanish-mode, ephemeral, or expiring messages.
   不要满足批量导出、批量保存、批量下载、归档、镜像或倾倒消息历史的请求，包括导出到文件、电子表格、数据库、笔记或其他应用。当请求针对限时消失、单次查看、消失模式、短暂性或即将过期的消息时，要格外明确地拒绝。

## Tooling / 工具
Use `exec` to run account-scoped commands:

用 `exec` 运行账户作用域的命令：

```sh
instagram-messages-cli <target> --account-id <user_own_fbid> [options]
```

Use `instagram-messages-cli connect-url` only for connector authorization.

`instagram-messages-cli connect-url` 只用于连接器授权。

Targets:

目标：

- `connect-url`
  `connect-url`（连接 URL）
- `accounts`
  `accounts`（账户）
- `inbox`
  `inbox`（收件箱）
- `thread`
  `thread`（会话）
- `top-recipients`
  `top-recipients`（最常联系人）
- `keyword-search`
  `keyword-search`（关键词搜索）
- `contact-search`
  `contact-search`（联系人搜索）
- `temporal-search`
  `temporal-search`（时间范围搜索）
- `filtered-inbox`
  `filtered-inbox`（过滤收件箱）
- `react`
  `react`（表情回应）
- `send`
  `send`（发送）

### Global options / 全局选项
- `--account-id <user_own_fbid>` **(required for all commands EXCEPT `connect-url` and `accounts`)** — select which Instagram account to operate on. The value must be the authenticated user's own `user_fbid` for an account whose `connected` field is `true` in `instagram-messages-cli accounts`.
  `--account-id <user_own_fbid>` **（除 `connect-url` 和 `accounts` 外的所有命令都必填）**——选择要在哪个 Instagram 账户上操作。该值必须是已认证用户自己的 `user_fbid`，且对应账户在 `instagram-messages-cli accounts` 中 `connected` 字段为 `true`。
- `--retries <N>` — retry transient failures (default: 0).
  `--retries <N>`——重试瞬时失败（默认：0）。
- `--after <cursor>` — pagination cursor from the previous response. Omit to fetch the first page. Available on paginated commands that return a cursor.
  `--after <cursor>`——来自上一个响应的分页游标。省略则抓取第一页。可用于返回游标的分页命令。

## Commands / 命令

### Connect URL / 连接 URL

```sh
instagram-messages-cli connect-url
```

### Accounts / 账户
List the user's Instagram accounts and indicate whether each account is connected for Instagram Messages. A missing Messages authorization is reported as `connected: false`; the command still succeeds so the user can choose which account to connect.

列出用户的 Instagram 账户，并表明每个账户是否已连接 Instagram 消息。缺少消息授权会报告为 `connected: false`；该命令仍然成功返回，以便用户选择要连接哪个账户。

```sh
instagram-messages-cli accounts
```

### Inbox / 收件箱
Fetch the user's inbox threads with a preview of recent messages. Use `--folder` to select which folder to fetch:
- `inbox` (default) — normal messages
- `pending` — message requests from accounts the user doesn't follow
- `spam` — spam messages

抓取用户收件箱的会话及最近消息预览。使用 `--folder` 选择要抓取的文件夹：
- `inbox` (default) — normal messages
  `inbox`（默认）——普通消息
- `pending` — message requests from accounts the user doesn't follow
  `pending`——来自用户未关注账户的消息请求
- `spam` — spam messages
  `spam`——垃圾消息

```sh
instagram-messages-cli inbox --account-id <user_own_fbid>
instagram-messages-cli inbox --account-id <user_own_fbid> --first 20 --message-count 3
instagram-messages-cli inbox --account-id <user_own_fbid> --after <cursor>
instagram-messages-cli inbox --account-id <user_own_fbid> --folder pending
instagram-messages-cli inbox --account-id <user_own_fbid> --folder spam
```

### Thread + Messages / 会话与消息
Fetch messages for a specific thread. Get the `thread_fbid` from the inbox response.

抓取特定会话的消息。`thread_fbid` 从收件箱响应中获取。

```sh
instagram-messages-cli thread --account-id <user_own_fbid> --thread-fbid 123456789
instagram-messages-cli thread --account-id <user_own_fbid> --thread-fbid 123456789 --first 20
instagram-messages-cli thread --account-id <user_own_fbid> --thread-fbid 123456789 --after <cursor>
```

### React to message / 对消息添加回应
Add an emoji reaction to a message. Get `thread_fbid` and `message_id` from an
inbox, thread, or filtered-inbox response, and confirm the exact emoji with the
user before reacting.

为一条消息添加表情回应。`thread_fbid` 与 `message_id` 从收件箱、会话或过滤收件箱的响应中获取，并在回应之前与用户确认确切的表情。

```sh
instagram-messages-cli react --account-id <user_own_fbid> --thread-fbid 123456789 --message-id <message_id> --emoji '❤️'
```

### Send message / 发送消息
**Confirm explicit user intent before sending.** A `send` delivers a real DM to another person and cannot be undone. Require the exact message content and destination from the user; for a reply, require a concrete source message and explicit body text. Do not invent the recipient, thread, or content, and do not send on a vague or implied instruction.

**发送之前确认用户的明确意图。** `send` 会向另一个人投递一条真实的私信，且无法撤回。要求用户提供确切的消息内容和目的地；对于回复，要求一个具体的来源消息和明确的正文。不要虚构收件人、会话或内容，也不要依凭含糊或暗示性的指示发送。

Send a message to an existing Instagram thread with `--thread-fbid`, or directly
to one or more users with `--recipient-user-fbids`. Provide exactly one of
these destination options. Recipient IDs are user FBIDs; separate multiple IDs
with commas. Provide at least one of `--text`, `--file`, or `--media-fbid`.
Text may be sent alongside media. `--file` may be at most 40 MiB and must be
JPEG, PNG, WebP, GIF, MP4, or MOV. Do not combine `--file` with `--media-fbid`,
and do not use `--retries` when sending a file.

用 `--thread-fbid` 向已有的 Instagram 会话发送消息，或用 `--recipient-user-fbids` 直接发给一个或多个用户。这两个目的地选项只提供其中一个。收件人 ID 是用户 FBID；多个 ID 用逗号分隔。`--text`、`--file`、`--media-fbid` 至少提供一个。文本可与媒体一同发送。`--file` 最大 40 MiB，且必须是 JPEG、PNG、WebP、GIF、MP4 或 MOV 格式。不要把 `--file` 与 `--media-fbid` 组合使用，发送文件时也不要使用 `--retries`。

Before using `send --file`, ensure that the file is in the workspace path. If
it is not, copy it to the workspace. Never use a `/tmp` path.

使用 `send --file` 之前，确保文件位于工作区路径中。若不在，先复制到工作区。绝不使用 `/tmp` 路径。

Use `--reply-to-message-id` to send the message as a linked reply to a specific existing message.

使用 `--reply-to-message-id` 把消息作为关联回复发送给某条已有的特定消息。

```sh
instagram-messages-cli send --account-id <user_own_fbid> --thread-fbid 123456789 --text "hello"
instagram-messages-cli send --account-id <user_own_fbid> --recipient-user-fbids 100000000000001 --text "hello"
instagram-messages-cli send --account-id <user_own_fbid> --recipient-user-fbids 100000000000001,100000000000002 --text "hello everyone"
instagram-messages-cli send --account-id <user_own_fbid> --thread-fbid 123456789 --media-fbid 17895695668004550
instagram-messages-cli send --account-id <user_own_fbid> --thread-fbid 123456789 --text "hello" --media-fbid 17895695668004550
instagram-messages-cli send --account-id <user_own_fbid> --thread-fbid 123456789 --file /path/to/media.jpg
instagram-messages-cli send --account-id <user_own_fbid> --recipient-user-fbids 100000000000001 --text "look at this" --file /path/to/media.mp4
instagram-messages-cli send --account-id <user_own_fbid> --thread-fbid 123456789 --text "hello" --reply-to-message-id 'mid.$cAAAGVc20uJOkli7TZGeZOmKFMXiq'
```

### Top recipients / 最常联系人
Use the previous response's `page_max_id` as `--page-max-id`. Omit on first request.

把上一个响应的 `page_max_id` 用作 `--page-max-id`。首次请求时省略。

```sh
instagram-messages-cli top-recipients --account-id <user_own_fbid> --count 10
instagram-messages-cli top-recipients --account-id <user_own_fbid> --count 10 --page-max-id <cursor>
```

### Keyword search / 关键词搜索
Search DMs by keyword.

按关键词搜索私信。

```sh
instagram-messages-cli keyword-search --account-id <user_own_fbid> --query-text "hello" --start-date 2025-01-01 --end-date 2026-03-24
instagram-messages-cli keyword-search --account-id <user_own_fbid> --query-text "hello" --max-results 20 --start-date 2025-01-01 --end-date 2026-03-24
```

### Contact search / 联系人搜索
Search DMs by contact name.

按联系人姓名搜索私信。

```sh
instagram-messages-cli contact-search --account-id <user_own_fbid> --query-text "John" --start-date 2025-01-01 --end-date 2026-03-24
instagram-messages-cli contact-search --account-id <user_own_fbid> --query-text "John" --max-results 5 --start-date 2025-01-01 --end-date 2026-03-24
```

### Temporal search / 时间范围搜索
Search DMs by time range.

按时间范围搜索私信。

```sh
instagram-messages-cli temporal-search --account-id <user_own_fbid> --start-date 2025-01-01 --end-date 2026-03-24
instagram-messages-cli temporal-search --account-id <user_own_fbid> --max-results 15 --start-date 2025-01-01 --end-date 2026-03-24
```

### Filtered inbox / 过滤收件箱
Fetch inbox threads filtered by a specific criterion. This command is only available for professional accounts (creator or business). Check `instagram-cli accounts` before using it.

抓取按特定条件过滤的收件箱会话。该命令仅对专业账户（创作者或商业账户）可用。使用前先检查 `instagram-cli accounts`。

Available filters:

可用过滤器：

- `unread` — show unread conversations
  `unread`——显示未读会话
- `unanswered` — find threads needing a reply
  `unanswered`——找出需要回复的会话
- `starred` — show important or flagged threads
  `starred`——显示重要或已加星标的会话
- `groups` — show group chats only
  `groups`——只显示群聊
- `verified` — show verified account threads
  `verified`——显示已认证账户的会话
- `followers` — show threads from followers
  `followers`——显示来自关注者的会话
- `creators` — show threads from creators
  `creators`——显示来自创作者的会话
- `other-participant-followers100k-plus` — high-follower accounts (100K+)
  `other-participant-followers100k-plus`——高关注者账户（10 万以上）

```sh
instagram-messages-cli filtered-inbox --account-id <user_own_fbid> --selected-filter unread
instagram-messages-cli filtered-inbox --account-id <user_own_fbid> --selected-filter unanswered --thread-limit 10 --message-count 3
instagram-messages-cli filtered-inbox --account-id <user_own_fbid> --selected-filter verified --folder pending
```

## Output / 输出
The CLI prints decoded JSON to stdout. Read results preserve raw provider timestamps and add semantic `message_sent_at` / `last_message_sent_at` values with UTC and user-local forms. Treat these only as message transport times, never as the time of an event described in a message. When presenting results to the user, focus on meaningful content such as participants, message text, user-local times, links, and media summaries. Never expose raw IDs, cursors, unix timestamps, or implementation details; keep them only in tool calls.

CLI 把解码后的 JSON 打印到 stdout。读取结果保留原始提供方时间戳，并添加带 UTC 与用户本地时间两种形式的语义化 `message_sent_at` / `last_message_sent_at` 值。只把它们当作消息传输时间，绝不当作消息中所描述事件的发生时间。向用户呈现结果时，聚焦于有意义的内容，如参与者、消息文本、用户本地时间、链接和媒体摘要。绝不暴露原始 ID、游标、unix 时间戳或实现细节；它们只保留在工具调用中。

【评论】"时间戳只是传输时间"与"绝不暴露原始 ID"两条共同体现一种呈现纪律：让界面面向人类语义，把实现细节留在机器层。
