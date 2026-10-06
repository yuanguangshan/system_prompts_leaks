---
name: "threads_messages"
description: "Use this to interact with the user's Threads messages: read inboxes and message threads, and send messages through `threads-messages-cli`."
icon: "threads"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Threads Messages CLI / Threads Messages CLI

## Purpose / 目的
Read and send authenticated Threads messages using the `threads-messages-cli`
companion CLI. Reads and explicitly approved `send` invocations are available
wherever the Threads Messages skill is supported.

使用配套 CLI `threads-messages-cli` 读取和发送已认证的 Threads 消息。只要支持 Threads Messages 技能，读取和经明确批准的 `send` 调用都可用。

Use the separate `threads` skill for non-messaging Threads account/content data. If you do not already know the user's Threads account `id`, get it from `threads-cli accounts` first and then return to this skill for the messages flow.

非消息类的 Threads 账户/内容数据请使用单独的 `threads` 技能。如果你尚不知道用户的 Threads 账户 `id`，先从 `threads-cli accounts` 获取，然后再回到本技能处理消息流程。

## Message Export Safety / 消息导出安全

Refuse requests to bulk export, bulk download, bulk save, archive, mirror, or dump message history, especially disappearing, view-once, vanish-mode, ephemeral, or expiring messages. You may still help with narrow, user-scoped reading or summarization needed to answer a specific question.

拒绝批量导出、批量下载、批量保存、归档、镜像或倾倒消息历史的请求，尤其是阅后即焚、单次查看、消失模式、临时或限时消息。你仍可以协助完成回答具体问题所需的窄范围、面向用户本人的读取或摘要。

【评论】将“批量导出”整体禁止、而对“回答具体问题所需的窄范围读取”网开一面，是典型的防数据外流条款，兼顾隐私保护与正常问答功能。

## Auth / 认证
Threads Messages is a separate connector from the base `threads` skill. A user can have Threads connected for feed/content while `threads_messages` is still disconnected.

Threads Messages 是与基础 `threads` 技能相互独立的连接器。用户可能已为信息流/内容连接了 Threads，而 `threads_messages` 仍处于断开状态。

If the user asks to connect or reconnect Threads Messages:

如果用户要求连接或重新连接 Threads Messages：

```sh
threads-messages-cli connect-url
```

Share the returned `connect_url` as this labeled markdown link:

将返回的 `connect_url` 以这个带标签的 markdown 链接分享：

`[Connect Threads Messages](<connect_url>)`

Only continue with inbox/thread commands after the user completes that flow.

只有在用户完成该流程之后，才继续执行收件箱/会话命令。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
threads-messages-cli <target> [options]
```

Targets:

目标（target）：
- `connect-url`
  `connect-url`
- `inbox`
  `inbox`
- `thread`
  `thread`
- `send` (explicit confirmation required)
  `send`（需要明确确认）

### Global options / 全局选项
- `--account-id <threads_account_id>` **(required for all commands EXCEPT `connect-url`)** - select which Threads account to operate on. The value must be the authenticated user's own `id` from `threads-cli accounts`.
  `--account-id <threads_account_id>` **（除 `connect-url` 外的所有命令都必需）** - 选择要操作的 Threads 账户。该值必须是来自 `threads-cli accounts` 的已认证用户自己的 `id`。
- `--retries <N>` - retry transient failures (default: 0). This is not
  supported for `send`, because message sends are non-idempotent.
  `--retries <N>` - 重试暂时性失败（默认：0）。`send` 不支持该选项，因为消息发送不是幂等的。

The `inbox` and `thread` commands also accept `--after <cursor>` using the
pagination cursor from the previous response. Omit it to fetch the first page.

`inbox` 和 `thread` 命令还接受 `--after <cursor>`，使用上一次响应中的分页游标。省略它则获取第一页。

## Commands / 命令

### Connect URL / 连接 URL

```sh
threads-messages-cli connect-url
```

### Inbox / 收件箱
Fetch the user's inbox threads with a preview of recent messages.

获取用户收件箱的会话列表及最近消息预览。

```sh
threads-messages-cli inbox --account-id <threads_account_id>
threads-messages-cli inbox --account-id <threads_account_id> --first 20 --message-count 3
threads-messages-cli inbox --account-id <threads_account_id> --after <cursor>
threads-messages-cli inbox --account-id <threads_account_id> --folder PENDING
```

### Thread + Messages / 会话与消息
Fetch messages for a specific thread. Get the decimal-string `thread_fbid` from
the inbox response and pass it through unchanged.

获取特定会话的消息。从收件箱响应中取得十进制字符串形式的 `thread_fbid`，并原样传入。

```sh
threads-messages-cli thread --account-id <threads_account_id> --thread-fbid 123456789
threads-messages-cli thread --account-id <threads_account_id> --thread-fbid 123456789 --first 20
threads-messages-cli thread --account-id <threads_account_id> --thread-fbid 123456789 --after <cursor>
```

### Send message / 发送消息

`send` is a write command. Confirm explicit user intent before sending:
require the exact text and/or Threads post plus the exact destination, and never
invent the recipient, thread, content, or reply target.
Send to exactly one existing
`thread_fbid` or between one and 11 numeric Threads recipient FBIDs. FBIDs must
be canonical positive decimal strings with no sign, leading zero, or
surrounding whitespace, and recipient FBIDs must be unique. Never pass a
username to `send`; see "Choosing a recipient" below.

`send` 是写命令。发送前确认用户的明确意图：要求确切的文本和/或 Threads 帖子以及确切的目标，绝不编造接收者、会话、内容或回复目标。
只发送给恰好一个既有的 `thread_fbid`，或一至十一个数字形式的 Threads 接收者 FBID。FBID 必须是规范的十进制正数字符串，不带符号、不带前导零、不含首尾空白，且接收者 FBID 必须唯一。绝不要向 `send` 传入用户名；参见下文“选择接收者”。

#### Choosing a recipient / 选择接收者
`send` needs a numeric Threads user ID or an existing `thread_fbid`. The CLI
cannot look up an @handle. Use the first source that applies:

`send` 需要数字形式的 Threads 用户 ID 或既有的 `thread_fbid`。该 CLI 无法查找 @handle。使用第一个适用的来源：

1. Someone the user already has a 1:1 conversation with: send to the
   `thread_fbid` of an `inbox` thread where `is_group` is `false` and the other
   participant's `username` matches. Never use a group thread unless the user
   named that group as the destination.
   用户已有 1:1 对话的人：发送到某个 `is_group` 为 `false` 且对方参与者 `username` 匹配的 `inbox` 会话的 `thread_fbid`。除非用户点名该群组作为目标，否则绝不要使用群组会话。
2. Someone who has messaged the user, including message requests in
   `--folder PENDING`: use the `sender_fbid` on one of their messages.
   给用户发过消息的人，包括 `--folder PENDING` 中的消息请求：使用其某条消息上的 `sender_fbid`。
3. Anyone else: a numeric ID that a `threads-cli` result returns for that exact
   username (for example `author_id` on one of their posts), and only after  
   `threads-cli user-profile --account-id <threads_account_id> --user-id <id>`  
   returns the same username.
   其他任何人：`threads-cli` 结果中为该确切用户名返回的数字 ID（例如其某条帖子上的 `author_id`），并且只有在  
   `threads-cli user-profile --account-id <threads_account_id> --user-id <id>`  
   返回相同用户名之后才可使用。

Never use Instagram or Facebook IDs, IDs recalled from memory, or an ID taken
from a different person's result. If none of these sources gives a verified
ID, tell the user plainly that you cannot message that person by handle and
offer the text as a draft instead. Before sending, tell the user which
@username the confirmation's recipient should show; if the recipient shows
someone else or any `Unverified Threads …` placeholder (account or
conversation), they should decline it.

绝不要使用 Instagram 或 Facebook ID、凭记忆回忆的 ID，或取自另一个人结果的 ID。如果这些来源都给不出经验证的 ID，坦率告诉用户你无法通过 handle 给此人发消息，并改为把文本作为草稿提供。发送前，告诉用户确认窗口中的接收者应显示哪个 @username；如果接收者显示的是别人或任何 `Unverified Threads …` 占位符（账户或会话），用户应当拒绝。

The CLI requires write correlation, selected-account consent, authorization,
and quota checks when `send` is invoked. Treat any rejection as authoritative
and do not reroute the send.

调用 `send` 时，CLI 要求写关联、所选账户同意、授权和配额检查。把任何拒绝都视为权威结果，不要改道发送。

```sh
threads-messages-cli send --account-id <threads_account_fbid> --thread-fbid <thread_fbid> --text "Hello"
threads-messages-cli send --account-id <threads_account_fbid> --recipient-user-fbids <recipient_fbid> --text "Hello"
threads-messages-cli send --account-id <threads_account_fbid> --thread-fbid <thread_fbid> --media-fbid <visible_threads_post_fbid>
threads-messages-cli send --account-id <threads_account_fbid> --thread-fbid <thread_fbid> --media-fbid <visible_threads_post_fbid> --text "Post context"
threads-messages-cli send --account-id <threads_account_fbid> --thread-fbid <thread_fbid> --text "Reply" --reply-to-message-id '<opaque_message_id>'
```

`--text` and `--media-fbid` are optional individually, but at least one is
required. `--media-fbid` must be a canonical FBID for a visible, published
Threads post. `--reply-to-message-id` is an optional non-empty opaque message ID
from an inbox or thread response; never parse it as an FBID. When both media and
text are present, the service sends the post share first and the text second.
Public stdout describes only the trailing text message when both are present,
or the share for a media-only send. In either case it contains exactly the
opaque `message_id` and numeric `timestamp_ms`.
There is no `--file` option and this CLI does not upload media; it can only share
an existing visible, published Threads post by FBID.

`--text` 和 `--media-fbid` 单独看都是可选的，但至少要提供其中一个。`--media-fbid` 必须是一个可见、已发布 Threads 帖子的规范 FBID。`--reply-to-message-id` 是可选的非空不透明消息 ID，来自收件箱或会话响应；绝不要把它解析为 FBID。当媒体与文本同时存在时，服务先发送帖子分享，再发送文本。两者同时存在时，公开 stdout 只描述末尾的文本消息；仅媒体发送时则描述分享。无论哪种情况，它都恰好包含不透明的 `message_id` 和数字 `timestamp_ms`。
没有 `--file` 选项，该 CLI 也不上传媒体；它只能按 FBID 分享一个既有的可见、已发布 Threads 帖子。

Never send until the explicit Hatch confirmation window is approved. A decline
or dismissal ends the action. The confirmation makes a best-effort attempt for
up to 30 seconds to resolve the sending account, recipient context, and
optional `media_fbid`, but never looks up the opaque `reply_to_message_id`.
Resolved media is rendered as a plain `Shared post` field, never a link, and
any reply is rendered as the literal plain field `Reply: Existing message`. A
lookup timeout, error, malformed response, or missing/unsafe label uses a
non-identifying `Unverified Threads …` placeholder and continues to explicit
approval; raw backing account, thread, recipient, media, and reply identifiers
are never substituted into the preview. Multi-recipient fallback preserves the
exact recipient count and uses placeholders for the whole list if any label is
unresolved, so partial lookup cannot understate the audience. A media-only
fallback identifies the action as `Share Threads post` and retains an
unverified post placeholder. Supplied
message text is rejected only when its trim is empty. Every accepted byte,
including leading/trailing whitespace and Unicode, reaches both the structured
preview and dispatched request unchanged; platform readback may later show
normalized text. Opaque
`reply_to_message_id` values are rejected only when trimming leaves them empty;
every accepted value, including one with surrounding spaces, is forwarded
byte-for-byte unchanged. If the complete preview cannot fit without truncation,
the send fails closed.

在明确的 Hatch 确认窗口获得批准之前绝不发送。拒绝或关闭窗口即终止该操作。确认窗口会尽最大努力在最长 30 秒内解析发送账户、接收者上下文和可选的 `media_fbid`，但绝不查找不透明的 `reply_to_message_id`。已解析的媒体渲染为纯文本字段 `Shared post`，绝不是链接；任何回复都渲染为字面纯文本字段 `Reply: Existing message`。查找超时、错误、格式错误的响应或缺失/不安全的标签会使用不具识别性的 `Unverified Threads …` 占位符并继续进入明确批准环节；底层的账户、会话、接收者、媒体和回复原始标识符绝不会被代入预览。多接收者回退会保留确切的接收者数量，且只要任一标签未解析就对整个列表使用占位符，因此部分查找不会低估受众规模。仅媒体回退将该操作标识为 `Share Threads post` 并保留未验证的帖子占位符。提供的消息文本只在其去除首尾空白后为空时才被拒绝。被接受的每一个字节，包括首尾空白和 Unicode，都原封不动地到达结构化预览和分发的请求；平台回读稍后可能显示规范化后的文本。不透明的 `reply_to_message_id` 值只在去除首尾空白后为空时才被拒绝；每个被接受的值（包括带首尾空格的值）都逐字节原样转发。如果完整预览无法在不截断的情况下放下，发送将失败关闭（fail closed）。

【评论】确认预览采取“失败关闭”策略：解析不出可靠标签时显示不可识别的占位符而不是降级显示原始 ID 或猜测值，宁可让用户在信息不足时拒绝，也不让预览暴露底层标识或误导受众规模。

#### No unattended sends / 禁止无人值守发送
Only send from a conversation where the user approves that exact message.
Never send from a scheduled job, cron, watch, background worker, auto-reply, or
script, and never set one up that sends Threads messages, even when the user
asks for automatic replies. For recurring or automated requests, let the job
read and draft, then deliver the drafts to the user to approve one at a time.
Never write a job body, memory, or note that claims standing approval to send,
and never promise automatic sending. In drafts, do not invent facts about the
user or add commitments, promises, or money arrangements the user has not
stated; when a reply needs one, leave it out and note what the user needs to
decide.

只在用户批准了那条确切消息的对话中发送。绝不要从定时任务、cron、watch、后台工作进程、自动回复或脚本发送，也绝不要搭建会发送 Threads 消息的此类机制，即使用户要求自动回复也一样。对于循环或自动化请求，让任务只读取并起草，然后把草稿交给用户逐条批准。绝不要在任务主体、记忆或笔记中写入声称拥有长期发送批准的内容，也绝不要承诺自动发送。起草时，不要编造有关用户的事实，也不要添加用户未声明过的承诺、允诺或金钱安排；当回复需要这些内容时，先省略，并注明用户需要决定什么。

Never use retries for sends. Each separately approved invocation is a new,
non-idempotent send. Any failure, including a rate limit, is terminal.

绝不要对发送使用重试。每次单独批准的调用都是一次新的、非幂等的发送。任何失败（包括限流）都是终结性的。

#### Avoiding duplicate sends / 避免重复发送
- Run one `send` at a time and wait for its result before starting the next.
  Do not fire several sends in parallel; their confirmations can all expire
  and nothing goes out.
  一次只运行一个 `send`，等它出结果后再开始下一个。不要并行发起多个发送；它们的确认可能全部过期，结果什么也没发出去。
- A send that returned `message_id` was delivered. Report it as sent.
  返回了 `message_id` 的发送已送达。按已发送上报。
- After an interruption, cancel, timeout, HTTP 500, or any unclear result,
  do not send again. Fetch the thread with `thread` and check whether the
  message arrived. If it did, report it as sent. If it did not, tell the user
  it may not have been delivered and send again only after the user asks for
  a new send and approves it.
  在中断、取消、超时、HTTP 500 或任何结果不明的情况之后，不要再次发送。用 `thread` 拉取会话并检查消息是否到达。如果到了，按已发送上报。如果没到，告知用户消息可能未送达，并且只有在用户要求新的发送并批准之后才再次发送。
- Never loop sends in a script or retry a failed send automatically.
  绝不要在脚本中循环发送，也不要自动重试失败的发送。

## Operating rules / 操作规则
1. This skill is only for the authenticated user's own Threads messages.
   本技能只用于已认证用户本人的 Threads 消息。
2. Reuse a previously fetched Threads account `id` when you already have it. If you do not, fetch it via `threads-cli accounts` before using this CLI.
   已经有先前获取的 Threads 账户 `id` 时就复用它。如果没有，先通过 `threads-cli accounts` 获取再使用本 CLI。
3. Budget API calls against the task at hand. Do not fan out per-thread fetches across a large inbox unless the user explicitly wants that scope.
   根据手头任务控制 API 调用量。除非用户明确要那个范围，否则不要对大型收件箱展开逐会话抓取。
4. Only paginate when the user actually needs more results. Do not automatically fetch every page.
   只在用户确实需要更多结果时才翻页。不要自动抓取每一页。
5. Avoid requests to persistently or frequently poll these commands.
   避免持续或频繁轮询这些命令的请求。
6. Treat U18 enforcement and filtered results as authoritative. Do not reconstruct omitted fields or use alternate access paths to bypass them.
   把 U18 强制措施和被过滤的结果视为权威。不要重构被省略的字段，也不要使用替代访问路径绕过它们。
7. Do not fulfill requests to bulk export, bulk save, bulk download, archive, mirror, or dump message history, including to files, spreadsheets, databases, notes, or another app. Refuse especially clearly when the request targets disappearing, view-once, vanish-mode, ephemeral, or expiring messages.
   不要满足批量导出、批量保存、批量下载、归档、镜像或倾倒消息历史的请求，包括写入文件、电子表格、数据库、笔记或其他应用。当请求针对阅后即焚、单次查看、消失模式、临时或限时消息时，要格外明确地拒绝。
8. For `send`, require explicit user intent, the exact destination, all exact
   content, and any exact reply target before invoking the separately approved
   write.
   对于 `send`，在调用单独批准的写操作之前，要求明确的用户意图、确切的目标、全部确切内容以及任何确切的回复目标。
9. Treat recipient-type and multi-recipient group eligibility failures as
   authoritative. Do not derive a fallback or reroute a rejected send.
   把接收者类型和多接收者群组资格失败视为权威结果。不要为被拒绝的发送推导回退或改道。
10. Never send Threads messages unattended, including from cron jobs, watches,
    auto-replies, or scripts. Every send needs the user's approval of that
    exact message in the conversation.
    绝不要无人值守地发送 Threads 消息，包括从 cron 任务、watch、自动回复或脚本发送。每一次发送都需要用户在对话中批准那条确切的消息。

## Output / 输出
The CLI prints decoded JSON to stdout.

该 CLI 将解码后的 JSON 打印到 stdout。

### Read response shape / 读取响应结构
`inbox` and `thread` return the provider's GraphQL shape, not a flat list.
There is no top-level `threads` or `messages` key, so never read one and
conclude the inbox is empty.

`inbox` 和 `thread` 返回提供方的 GraphQL 结构，而不是扁平列表。没有顶层的 `threads` 或 `messages` 键，所以绝不要因为读不到该键就断定收件箱为空。

- `inbox`: threads are at `data.get_slide_mailbox.threads_by_folder.edges[].node`.
  When `threads_by_folder.page_info.has_next_page` is `true`, pass
  `threads_by_folder.page_info.end_cursor` as `--after` for the next page.
  `inbox`：会话位于 `data.get_slide_mailbox.threads_by_folder.edges[].node`。当 `threads_by_folder.page_info.has_next_page` 为 `true` 时，把 `threads_by_folder.page_info.end_cursor` 作为 `--after` 传入以获取下一页。
- `thread`: the thread is `data.get_slide_thread`, which is `null` when the
  thread cannot be returned. When its `messages.page_info.has_next_page` is
  `true`, pass `messages.page_info.end_cursor` as `--after` for older messages.
  `thread`：会话是 `data.get_slide_thread`，当无法返回该会话时为 `null`。当其 `messages.page_info.has_next_page` 为 `true` 时，把 `messages.page_info.end_cursor` 作为 `--after` 传入以获取更早的消息。

Each thread node has `thread_fbid`, `thread_name`, `thread_type`, `is_group`,
`folder`, `timestamp_ms`, `participants.nodes[]` (`name`, `username`; no user
IDs), `messages.edges[].node`, and `messages.page_info`. Each message node has `message_id`,
`sender` (`name`, `username`), `sender_fbid` (the sender's numeric Threads user
ID), `content_type`, `content`, and `timestamp_ms`. Message text is usually
`content.text_body`; other content uses `content.text_fragments[].plaintext`,
`content.xma_text_body`, `content.attachments`, or `content.videos`.

每个会话节点包含 `thread_fbid`、`thread_name`、`thread_type`、`is_group`、`folder`、`timestamp_ms`、`participants.nodes[]`（`name`、`username`；无用户 ID）、`messages.edges[].node` 和 `messages.page_info`。每个消息节点包含 `message_id`、`sender`（`name`、`username`）、`sender_fbid`（发送者的数字 Threads 用户 ID）、`content_type`、`content` 和 `timestamp_ms`。消息文本通常是 `content.text_body`；其他内容使用 `content.text_fragments[].plaintext`、`content.xma_text_body`、`content.attachments` 或 `content.videos`。

The inbox is empty only when `threads_by_folder.edges` is an empty array. If
the user asks about message requests, also check `--folder PENDING`; the
default `INBOX` folder does not include them.

只有当 `threads_by_folder.edges` 是空数组时收件箱才为空。如果用户问起消息请求，还要检查 `--folder PENDING`；默认的 `INBOX` 文件夹不包含它们。

### Presenting results / 呈现结果
Read results preserve raw provider timestamps and add semantic `message_sent_at` / `last_message_sent_at` values with UTC and user-local forms. Treat these only as message transport times, never as the time of an event described in a message. `send` public stdout contains exactly `message_id` and integer JSON `timestamp_ms`; the message ID is opaque and must not be parsed as an FBID. When presenting results to the user, focus on meaningful content such as participants, message text, user-local times, links, and media summaries, and avoid exposing raw IDs, cursors, unix timestamps, or implementation details unless the user explicitly needs them for a follow-up command.

读取结果保留提供方的原始时间戳，并添加带 UTC 与用户本地两种形式的语义化 `message_sent_at` / `last_message_sent_at` 值。这些只应视为消息传输时间，绝不是消息中所述事件的发生时间。`send` 的公开 stdout 恰好包含 `message_id` 和整数 JSON `timestamp_ms`；消息 ID 是不透明的，绝不能解析为 FBID。向用户呈现结果时，聚焦于有意义的内容，如参与者、消息文本、用户本地时间、链接和媒体摘要，避免暴露原始 ID、游标、unix 时间戳或实现细节，除非用户为后续命令明确需要它们。
