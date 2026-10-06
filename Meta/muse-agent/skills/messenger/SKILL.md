<!-- BILINGUAL-EN-ZH -->
---
name: "messenger_read"
title: "Messenger"
icon: "messenger"
description: "Work with the user's Messenger account: read call history; read and search contacts; read, search, and summarize conversations; send, react to, unsend, or edit messages; and message Marketplace listing threads."
category: "communication"
metadata: { "includeInPrompt": true }
---

# Messenger / Messenger

Use Messenger Companion to work with the user's personal Messenger account.

使用 Messenger Companion 处理用户的个人 Messenger 账户。

## Routing and Safety / 路由与安全

Messenger on Muse has two separate features:

Muse 上的 Messenger 有两个相互独立的功能：

| User wants to... | Feature | What to do |
|---|---|---|
| Message Muse on Messenger | **Messenger side chat** | Use `chat.connection_status` and the connection flow in `~/docs/chat-connections/messenger.md` |
| Read Messenger call history or work with personal-account messages | **Messenger Companion** | Follow this skill |
| Read native phone call history or place a phone call | **Paired phone / phone calling** | Use the native phone or call-placement feature, not Messenger Companion |
| Connect or disconnect "Messenger" without specifying which | **Ambiguous** | Briefly explain both and clarify |

| 用户想要… | 功能 | 该怎么做 |
|---|---|---|
| 在 Messenger 上给 Muse 发消息 | **Messenger 侧聊** | 使用 `chat.connection_status` 与 `~/docs/chat-connections/messenger.md` 中的连接流程 |
| 读取 Messenger 通话历史或处理个人账户消息 | **Messenger Companion** | 遵循本技能 |
| 读取原生手机通话历史或拨打电话 | **配对手机 / 电话拨打** | 使用原生手机或拨号功能，而不是 Messenger Companion |
| 连接或断开"Messenger"而未指明是哪个 | **有歧义** | 简要说明两者并请用户澄清 |

- Use Companion only when the user explicitly asks to work with their personal
  account. It sends as the user, not as Muse.

  仅当用户明确要求处理其个人账户时才使用 Companion。它以用户的身份发送，而不是以 Muse 的身份。

- Never target the Messenger side chat with `hatch_messenger_cli send`,
  whether by `--cid`, `--to`, or an auto-reply rule. Exclude that thread during
  recipient resolution; messages there must use the Messenger side-chat connection flow in  
  `~/docs/chat-connections/messenger.md`.

  绝不要用 `hatch_messenger_cli send` 把消息发到 Messenger 侧聊——无论通过 `--cid`、`--to` 还是自动回复规则。在收件人解析时排除该会话；发到那里的消息必须使用 `~/docs/chat-connections/messenger.md` 中的 Messenger 侧聊连接流程。

- Never use Companion to deliver assistant notifications, cron or monitor
  results, reminders, or status updates. A user-configured auto-reply rule is
  the only background-send exception: it may check and reply in ordinary or
  Marketplace conversations, but it must not carry Muse's own notifications or
  results. Other background workers return results with `muse.notify_main_agent`;
  the runtime delivers through the originating chat.

  绝不要用 Companion 递送助手通知、cron 或监控结果、提醒或状态更新。用户配置的自动回复规则是唯一的后台发送例外：它可以在普通会话或 Marketplace 会话中检查并回复，但不得携带 Muse 自己的通知或结果。其他后台工作者用 `muse.notify_main_agent` 返回结果，由运行时通过来源会话递送。

- Treat every Marketplace message that could create or change a real-world
  commitment as requiring fresh user confirmation. This includes proposing,
  accepting, confirming, rescheduling, or cancelling a meetup, pickup,
  delivery, hold, price, or payment arrangement, and claiming that the user is
  leaving, en route, nearby, or has arrived. Show the exact outgoing message
  and wait for the user's explicit confirmation immediately before each send.
  A permission approval, an earlier confirmation, or approval of a recurring,
  cron, monitoring, or auto-reply task never authorizes a later commitment.
  Background work must instead return the proposed reply and its context with
  `muse.notify_main_agent` so the main conversation can ask the user; if that
  confirmation cannot be obtained, leave the message unsent.

  把每一条可能创设或变更现实世界承诺的 Marketplace 消息，都当作需要用户当场重新确认来对待。这包括提议、接受、确认、改期或取消碰面、取货、交货、保留、价格或付款安排，以及声称用户正在出发、在路上、在附近或已到达。展示确切的待发消息，并在每次发送前立即等待用户的明确确认。一次权限批准、一次更早的确认，或对某个循环任务、cron、监控或自动回复任务的批准，都绝不授权此后的一次承诺。后台工作必须改为用 `muse.notify_main_agent` 返回拟议回复及其上下文，让主会话去询问用户；如果无法获得该确认，就让消息保持未发送。

  【评论】把"现实世界承诺"单独划为需要即时确认的高风险类别，并显式否定历史授权对后续承诺的延续效力，是针对代理身份被滥用的防护设计。

- In user-facing replies, never say `sync`, `synced`, `syncing`, `cache`,
  `cached`, `widened`, `local data`, `authentication state`, `keys`,
  `credentials`, `tokens`, or connected hardware. Do not state how many messages were
  searched, read, synced, or available. Describe the result instead: "I checked
  the latest messages" or "I checked further back." If complete coverage could
  not be established, say so without naming the underlying mechanism.

  在面向用户的回复中，绝不要说 `sync`、`synced`、`syncing`、`cache`、`cached`、`widened`、`local data`、`authentication state`、`keys`、`credentials`、`tokens` 或已连接的硬件。不要陈述搜索、读取、同步或可用的消息数量。改为描述结果："我查看了最新消息"或"我往前查了更多"。如果无法确认完整覆盖，就如实说明，但不要点明底层机制。

  【评论】对同步/缓存/凭据等术语的全面禁用，是把实现细节从用户视野中剥离的体验设计，同时也避免暗示本地存有消息数据库。

- Refuse requests to bulk export, download, save, archive, mirror, or dump
  message history, especially disappearing, view-once, vanish-mode, ephemeral,
  or expiring messages. Narrow reading, search, and summarization for a specific
  question are allowed.

  拒绝批量导出、下载、保存、归档、镜像或倾倒消息历史的请求，尤其是限时可见、单次可见、消失模式、临时性或即将过期的消息。针对具体问题的窄范围读取、搜索和摘要是允许的。

## Tooling / 工具

Run the JSON CLI through `exec`:

通过 `exec` 运行这个 JSON CLI：

```sh
hatch_messenger_cli <subcommand> [options]
```

Use only this supported CLI; never access local Messenger storage directly.

只使用这个受支持的 CLI；绝不要直接访问本地 Messenger 存储。

| Task | Command |
|---|---|
| Check connection | `check` |
| Get connect/disconnect link | `connect-url` / `disconnect-url` |
| Refresh normal data | `sync both [--since-days N]` |
| Refresh contacts | `sync contacts [--search "query"] [--limit N]` |
| Refresh threads/messages | `sync threads [--since-days N] [--older N]` / `sync messages --cid '<ID>' [--since-days N] [--older N]` |
| Read Messenger call history | `call-logs [--limit N] [--since-timestamp <MS>] [--to-timestamp <MS>]` |
| Read contacts | `contacts [--search "query"] [--limit N]` |
| Read threads | `threads [--limit N] [--search "query"] [--unread] [--folder mplace]` |
| Read a thread | `messages --cid '<ID>' [--limit N] [--before <TS>] [--after <TS>]` |
| Search messages | `search [query] [--sender "name_or_id"] [--cid '<ID>'] [--after <TS>] [--before <TS>] [--limit N]` |
| Repair unreadable rows | `repair` |
| Send/initiate/react/edit/unsend | See the mutation workflows below |

| 任务 | 命令 |
|---|---|
| 检查连接 | `check` |
| 获取连接/断开链接 | `connect-url` / `disconnect-url` |
| 刷新常规数据 | `sync both [--since-days N]` |
| 刷新联系人 | `sync contacts [--search "query"] [--limit N]` |
| 刷新会话/消息 | `sync threads [--since-days N] [--older N]` / `sync messages --cid '<ID>' [--since-days N] [--older N]` |
| 读取 Messenger 通话历史 | `call-logs [--limit N] [--since-timestamp <MS>] [--to-timestamp <MS>]` |
| 读取联系人 | `contacts [--search "query"] [--limit N]` |
| 读取会话 | `threads [--limit N] [--search "query"] [--unread] [--folder mplace]` |
| 读取单个会话 | `messages --cid '<ID>' [--limit N] [--before <TS>] [--after <TS>]` |
| 搜索消息 | `search [query] [--sender "name_or_id"] [--cid '<ID>'] [--after <TS>] [--before <TS>] [--limit N]` |
| 修复不可读的行 | `repair` |
| 发送/发起/回应/编辑/撤回 | 见下文的变更操作工作流 |

### User-facing output / 面向用户的输出

- Never expose raw tool metadata: IDs (`msg_id`, `message_id`, `mid.$...`,
  `conversation_id`, `cid.$...`, caller/callee FBIDs, `otid`), device counts,
  encryption details, delivery or receipt state, raw JSON, or cache statistics.

  绝不暴露原始工具元数据：各类 ID（`msg_id`、`message_id`、`mid.$...`、`conversation_id`、`cid.$...`、主叫/被叫 FBID、`otid`）、设备计数、加密细节、送达或回执状态、原始 JSON 或缓存统计。

- The only ID-derived output exceptions are an FBID embedded in an allowed
  Facebook profile link and the target portion of a user-requested Messenger
  thread link. Never display either ID separately or put a full namespaced ID
  in a URL.

  仅有的两类由 ID 派生的输出例外是：内嵌在允许的 Facebook 主页链接中的 FBID，以及用户请求的 Messenger 会话链接中的目标部分。绝不要单独显示这两种 ID，也不要把带命名空间的完整 ID 放进 URL。

- A URL found in message text or message-level URL metadata may be returned
  when requested, but expose only the URL, not its surrounding metadata.

  消息文本或消息级 URL 元数据中发现的 URL，在被请求时可以返回，但只暴露 URL 本身，不暴露其周围的元数据。

- After a successful mutation, confirm briefly and naturally. Do not narrate
  the service response or claim delivery/read status.

  变更操作成功后，简短而自然地确认。不要复述服务端响应，也不要声称送达/已读状态。

## Connect or Disconnect Companion / 连接或断开 Companion

Check first:

先做检查：

```sh
hatch_messenger_cli check
```

If Companion is not connected, run `hatch_messenger_cli connect-url`. When the
response contains `connect_url`, share exactly
`[Connect Messenger](<connect_url>)`, without also pasting the raw URL. Do not
collect secrets or guide the user through manual authentication in chat. After
the user links in Settings, continue with the requested operation.

如果 Companion 未连接，运行 `hatch_messenger_cli connect-url`。当响应包含 `connect_url` 时，分享 `[Connect Messenger](<connect_url>)` 这一确切形式，不要同时粘贴原始 URL。不要收集机密信息，也不要在聊天中引导用户做手动鉴权。用户在设置中完成关联后，继续执行所请求的操作。

To disconnect Companion, run `hatch_messenger_cli disconnect-url`. When the
response contains `disconnect_url`, share exactly
`[Disconnect Messenger](<disconnect_url>)`, without also pasting the raw URL.
Disconnecting Companion removes its access to the personal account but does not
affect the Messenger side-chat connection. Reconnect later with `connect-url` and the same link
rule.

要断开 Companion，运行 `hatch_messenger_cli disconnect-url`。当响应包含 `disconnect_url` 时，分享 `[Disconnect Messenger](<disconnect_url>)` 这一确切形式，不要同时粘贴原始 URL。断开 Companion 会移除它对个人账户的访问权，但不影响 Messenger 侧聊连接。之后用 `connect-url` 和同样的链接规则重新连接。

## Sync Threads and Messages / 同步会话与消息

Read commands use locally retained Messenger data. Sync before claiming
freshness or completeness.

读取命令使用本地保留的 Messenger 数据。在声称新鲜或完整之前，先同步。

Use `hatch_messenger_cli sync both` for normal refreshes. Add `--since-days N`
when the question reaches farther back. For older history in one thread, use:

常规刷新用 `hatch_messenger_cli sync both`。当问题涉及更早的时间时加 `--since-days N`。要取单个会话中更早的历史，用：

```sh
hatch_messenger_cli sync messages --cid '<CONVERSATION_ID>' --older 100
```

`sync both` is incremental and fills message gaps. If `_cache.gaps` overlaps
the question, repeat it until the relevant gaps clear. If the requested period
predates `_cache.oldest_cached_ts`, extend it with `sync both --since-days N`.
For a single-thread gap, prefer targeted `sync messages`. Qualify the answer if
complete coverage still cannot be established, using the user-facing language
from Routing and Safety.

`sync both` 是增量的，会填补消息缺口。如果 `_cache.gaps` 与问题重叠，就重复执行直到相关缺口清零。如果请求的时间段早于 `_cache.oldest_cached_ts`，用 `sync both --since-days N` 扩展。对单会话缺口，优先用有针对性的 `sync messages`。如果仍然无法确认完整覆盖，就用"路由与安全"一节中面向用户的措辞对答案加以限定。

# Resolve Contacts / 解析联系人

```sh
hatch_messenger_cli sync contacts
hatch_messenger_cli contacts --search "Alice"
```

The default refresh fetches up to 100 contacts and merges them into the contact
snapshot. If the user asks for more, rerun with a limit larger than 100 or the
previous bounded limit, up to 10,000. `contacts` reads the complete retained
snapshot; `--search` performs case-insensitive substring matching on names.

默认刷新最多取回 100 个联系人，并把它们合并进联系人快照。如果用户要求更多，就用大于 100 的上限或之前的受限上限重跑，至多 10,000。`contacts` 读取完整的保留快照；`--search` 对名字做不区分大小写的子串匹配。

If local search returns no result, retry once with
`sync contacts --search "<name>"`, then repeat the local search. Also use that
targeted refresh when matches are ambiguous or incomplete, the user asks to
look beyond known contacts, or current server results are required.

如果本地搜索没有结果，用 `sync contacts --search "<name>"` 重试一次，然后重复本地搜索。当匹配有歧义或不完整、用户要求查找已知联系人之外的人、或需要当前的服务器结果时，也使用这种定向刷新。

### Contact profile links / 联系人主页链接

In contact-list/search results and the **Recipients:** field of a send preview,
link each full display name when a profile URL or FBID is available. Prefer an
explicit `profile_url` or `vanity_url`; otherwise use
`https://www.facebook.com/profile.php?id=<fbid>`. Keep names plain when neither
is known. Do not show a raw FBID, a bare profile URL, or a shortened name.

在联系人列表/搜索结果以及发送预览的 **Recipients:** 字段中，只要有主页 URL 或 FBID 可用，就给每个完整显示名加链接。优先用明确的 `profile_url` 或 `vanity_url`；否则用 `https://www.facebook.com/profile.php?id=<fbid>`。两者都未知时，名字保持纯文本。不要显示裸 FBID、裸主页 URL 或截短的名字。

Use plain contact names in thread labels, thread-search results, unrelated
assistant prose, and success confirmations. Never add a profile link to the
outgoing message unless the user explicitly asked to send that exact linked
text.

在会话标签、会话搜索结果、无关的助手正文以及成功确认中使用纯文本联系人名。绝不要在待发消息中加主页链接，除非用户明确要求发送那段带链接的文字。

## Read Messenger Call History / 读取 Messenger 通话历史

Use `hatch_messenger_cli call-logs` for the latest Messenger audio and video call
history. `--limit` accepts 0 through 50 and defaults server-side to 20. Use
`--since-timestamp <MS>` for an inclusive lower bound and `--to-timestamp <MS>`
for an exclusive upper bound; when both are present, the lower bound must be
less than the upper bound.

用 `hatch_messenger_cli call-logs` 获取最新的 Messenger 音频与视频通话历史。`--limit` 接受 0 到 50，服务端默认 20。`--since-timestamp <MS>` 是包含下界，`--to-timestamp <MS>` 是排除上界；两者同时给出时，下界必须小于上界。

Report the annotated time, event, audio/video type, duration, direction, and
missed status without exposing caller, callee, conversation, or message IDs or
raw JSON. An empty result means no Messenger calls were returned for that
bounded query; it does not establish that the user has no native phone call
history or no calls outside the retained Messenger history window.

报告带注记的时间、事件、音频/视频类型、时长、方向和未接状态，不暴露主叫、被叫、会话或消息 ID，也不暴露原始 JSON。空结果表示该有界查询没有返回 Messenger 通话；它并不能证明用户没有原生手机通话历史，也不能证明在保留的 Messenger 历史窗口之外没有通话。

## Read Threads and Messages / 读取会话与消息

List or resolve threads before reading one:

读取某个会话之前，先列出或解析会话：

```sh
hatch_messenger_cli threads --limit 20
hatch_messenger_cli threads --search "Bao" --limit 10
hatch_messenger_cli messages --cid '<CONVERSATION_ID>' --limit 20
```

`threads --search` matches names and nicknames. If exactly one thread matches,
read it; otherwise ask the user to disambiguate. Folder filtering applies only
to `threads`, not `sync`:

`threads --search` 匹配名字和昵称。如果恰好一个会话匹配，就读取它；否则请用户消歧。文件夹过滤只适用于 `threads`，不适用于 `sync`：

```sh
hatch_messenger_cli threads --folder mplace --limit 20
hatch_messenger_cli threads --folder mplace --unread
```

For older retained rows, use `messages --before <TIMESTAMP_MS>`. For unread
catch-up, read `threads --unread`, then the chosen thread with  
`messages --after <read_timestamp>`.

要取更早的保留行，用 `messages --before <TIMESTAMP_MS>`。补读未读时，先读 `threads --unread`，再用 `messages --after <read_timestamp>` 读所选会话。

If a row says `(decrypt failed)` or `(no thread key)`, treat only that row as
unreadable. Run `repair` and read the thread again before drawing a conclusion;
see Repair Unreadable Messages.

如果某行显示 `(decrypt failed)` 或 `(no thread key)`，只把该行当作不可读。先运行 `repair` 并重读该会话，再下结论；参见"修复不可读的消息"。

### Return URLs from messages / 返回消息中的 URL

When asked for a URL from a message, inspect both its text and message-level URL
metadata. Return the relevant value verbatim, preserving scheme, hostname,
port, path, query, fragment, casing, and percent-encoding. Do not normalize,
shorten, decode/re-encode, or rebuild it from a preview. Do not open it unless
the user asks.

当被要求给出消息中的 URL 时，同时检查其文本和消息级 URL 元数据。按原样返回相关值，保留协议、主机名、端口、路径、查询、片段、大小写和百分号编码。不要规范化、缩短、解码/重编码，也不要从预览重建它。除非用户要求，否则不要打开它。

### Count total messages / 统计消息总数

1. Run `threads` and compare the returned list with
   `_cache.cached_thread_count`; if needed, rerun with that count as `--limit`.
2. Deduplicate threads by `conversation_id`.
3. For each thread, run `messages --cid '<CONVERSATION_ID>' --limit 1` and read  
   `_cache.cached_message_count`.
4. Sum those counts, not the one-row `messages` arrays.

   1. 运行 `threads`，把返回列表与 `_cache.cached_thread_count` 比较；必要时以该计数作为 `--limit` 重跑。
   2. 按 `conversation_id` 给会话去重。
   3. 对每个会话，运行 `messages --cid '<CONVERSATION_ID>' --limit 1` 并读取 `_cache.cached_message_count`。
   4. 对这些计数求和，而不是对单行的 `messages` 数组求和。

Sync first for a current or complete count. Qualify an incomplete total using
the user-facing wording above, and never expose the IDs used in the calculation.

要得到当前或完整的计数，先同步。对不完整的总数，用上文面向用户的措辞加以限定，并绝不暴露计算中用到的 ID。

### Link to a specific conversation / 链接到具体会话

Only create a deep link when the user asks. Resolve the exact thread and inspect
both `conversation_id` and `is_e2ee`:

只在用户要求时创建深链接。解析确切的会话，并检查 `conversation_id` 与 `is_e2ee`：

- Open one-to-one: `is_e2ee: false` and
  `cid.c.<VIEWER_FBID>:<CONTACT_FBID>`. Compare both values with
  `_cache.self_user_id` and use the other value:  
  `https://www.messenger.com/t/<CONTACT_FBID>`.

  开放式一对一：`is_e2ee: false` 且 `cid.c.<VIEWER_FBID>:<CONTACT_FBID>`。把两个值都与 `_cache.self_user_id` 比较，取另一个值：`https://www.messenger.com/t/<CONTACT_FBID>`。
- Open group: `is_e2ee: false` and `cid.g.<GROUP_ID>`:  
  `https://www.messenger.com/t/<GROUP_ID>`.

  开放式群聊：`is_e2ee: false` 且 `cid.g.<GROUP_ID>`：`https://www.messenger.com/t/<GROUP_ID>`。
- E2EE chat: `is_e2ee: true` and `cid.g.<THREAD_ID>`:  
  `https://www.messenger.com/e2ee/t/<THREAD_ID>`.

  E2EE 聊天：`is_e2ee: true` 且 `cid.g.<THREAD_ID>`：`https://www.messenger.com/e2ee/t/<THREAD_ID>`。

Groups and E2EE chats share the `cid.g.` prefix, so never infer encryption from
the ID prefix, thread type, or name. If `is_e2ee` is absent, sync and resolve
again. For an open one-to-one, do not build the link unless
`_cache.self_user_id` identifies the other participant unambiguously. Use only
the documented numeric portion; never substitute another field, guess, or put
the `cid.c.`/`cid.g.` namespace in the URL. Return a descriptive Markdown link
without separately showing the ID.

群聊与 E2EE 聊天共用 `cid.g.` 前缀，因此绝不要从 ID 前缀、会话类型或名称推断加密状态。如果 `is_e2ee` 缺失，先同步再重新解析。对开放式一对一，除非 `_cache.self_user_id` 能无歧义地识别对方，否则不要构建链接。只使用文档说明的数字部分；绝不替换为其他字段、绝不猜测，也绝不把 `cid.c.`/`cid.g.` 命名空间放进 URL。返回一个描述性的 Markdown 链接，不单独显示 ID。

## Search Messages / 搜索消息

```sh
hatch_messenger_cli search "dinner"
hatch_messenger_cli search --sender "Alice"
hatch_messenger_cli search "dinner" --cid '<CONVERSATION_ID>'
```

- Search is literal substring matching over retained data, not semantic search.
  Quote multi-word queries and try alternate terms when useful.

  搜索是对保留数据的字面子串匹配，不是语义搜索。多词查询要加引号，必要时尝试其他说法。
- Resolve an ambiguous conversation with `threads --search` before scoping the
  query. For exhaustive searches, paginate with `--before <oldest_timestamp>`.

  在限定查询范围之前，先用 `threads --search` 消除会话歧义。要穷尽搜索，用 `--before <oldest_timestamp>` 翻页。
- Sync first when freshness or completeness matters.

  当新鲜度或完整性重要时，先同步。
- Bound date questions with `search --after <ms>` and optionally
  `--before <ms>`. `sync --since-days N` widens available history but does not
  filter search results.

  用 `search --after <ms>` 以及可选的 `--before <ms>` 给日期类问题设界。`sync --since-days N` 扩宽可用历史，但不过滤搜索结果。
- `message_sent_at` and `last_message_sent_at` are transport times. Never infer
  when an event mentioned inside a message occurred from those timestamps.

  `message_sent_at` 与 `last_message_sent_at` 是传输时间。绝不要从这些时间戳推断消息内提到的事件是何时发生的。

## Message Text Input / 消息文本输入

For every send, Marketplace initiation, or edit, pass the body through
`--text-stdin` as the final option and a single-quoted heredoc:

每次发送、Marketplace 发起或编辑，都把正文通过 `--text-stdin` 作为最后一个选项、以单引号 heredoc 传入：

```sh
--text-stdin << 'MESSENGER_INPUT'
exact message text
MESSENGER_INPUT
```

There is no `--text` flag. The quoted delimiter prevents shell expansion and
preserves `$`, quotes, and newlines. Choose a delimiter absent from the body;
never use an unquoted heredoc. Feed the heredoc directly to the CLI, without
replacing it with an `echo`/`printf` pipeline or command substitution.
`--text-stdin` cannot undo shell expansion that changed the body beforehand.

没有 `--text` 旗标。带引号的分隔符阻止 shell 展开，并保留 `$`、引号和换行。选择一个正文中不存在的分隔符；绝不使用不带引号的 heredoc。把 heredoc 直接喂给 CLI，不要用 `echo`/`printf` 管道或命令替换替代。`--text-stdin` 无法撤销事先已改变正文的 shell 展开。

## Send a Message / 发送消息

Send only on an explicit request and never to a guessed recipient.

只在明确请求时发送，且绝不发给猜测出来的收件人。

### Resolve the recipient / 解析收件人

1. Use `threads --search` for an existing conversation.
2. A request for a one-to-one, direct, or individual chat is a hard constraint.
   Before using `--cid`, verify `thread_type` is not a group and participant
   metadata contains no one beyond the user and intended recipient. Never fall
   back to a matching group.
3. If there is no verified one-to-one thread, follow the contact workflow. Use
   `--to <FACEBOOK_USER_ID>` only when that exact FBID appears on the intended
   person's current name-based `contacts --search` result. When only an FBID was
   initially available, it must exactly match an unambiguous record from
   unfiltered `contacts`.
4. An ID from thread participants, message metadata, memory, earlier output, or
   user text does not authorize `--to`. If the exact contact cannot be resolved,
   ask for the person's name or explain that the contact cannot be resolved.

   1. 用 `threads --search` 找已有会话。
   2. 要求一对一、直接或单人聊天是硬约束。使用 `--cid` 之前，验证 `thread_type` 不是群聊，且参与者元数据中除用户与预期收件人外没有别人。绝不回退到匹配上的群聊。
   3. 如果没有经验证的一对一会话，走联系人工作流。只有当那个确切的 FBID 出现在目标人物当前的按名字 `contacts --search` 结果中时，才使用 `--to <FACEBOOK_USER_ID>`。如果最初只有 FBID，它必须与未过滤的 `contacts` 中一条无歧义的记录精确匹配。
   4. 来自会话参与者、消息元数据、记忆、更早输出或用户文本的 ID 都不授权使用 `--to`。如果无法解析确切联系人，请用户提供人名，或说明无法解析该联系人。

### Preview and execute / 预览与执行

Immediately before sending, show:

发送前立即展示：

- **Thread:** only for `GROUP` or `SECURE_MESSAGE_OVER_WA_GROUP`;
- **Recipients:** resolved full names, linked according to Contact profile links;
- **Message:** the exact outgoing text as a Markdown blockquote; and
- **Attachment:** when present, followed on its own line by
  `![filename](sandbox://workspace/<workspace-relative-path>)` outside the
  blockquote.

- **Thread:** 仅对 `GROUP` 或 `SECURE_MESSAGE_OVER_WA_GROUP` 展示；
- **Recipients:** 解析出的全名，按"联系人主页链接"的规则加链接；
- **Message:** 以 Markdown 引用块展示确切的待发文本；以及
- **Attachment:** 如有附件，在引用块之外单独一行跟 `![filename](sandbox://workspace/<workspace-relative-path>)`。

Use names rather than IDs. Wait for explicit confirmation; if the user changes
anything, show the revised preview and wait again. Then call exactly one form:

使用名字而不是 ID。等待明确确认；如果用户改了任何内容，展示修订后的预览并再次等待。然后只调用以下两种形式之一：

```sh
hatch_messenger_cli send --cid '<CONVERSATION_ID>' [--attach <FILE>] --text-stdin << 'MESSENGER_INPUT'
message text
MESSENGER_INPUT
```

```sh
hatch_messenger_cli send --to '<FACEBOOK_USER_ID>' [--attach <FILE>] --text-stdin << 'MESSENGER_INPUT'
message text
MESSENGER_INPUT
```

For attachments, use a readable local path. If the source is outside the
workspace, copy it into `~/workspace/` first and use that same copy in both the
preview and `--attach`. Leading `~` resolves to the Muse home directory.

附件使用可读的本地路径。如果来源在工作区之外，先把它复制进 `~/workspace/`，并在预览与 `--attach` 中使用同一份副本。前导 `~` 解析为 Muse 主目录。

### Automatic replies / 自动回复

The user may create a recurring rule that checks their ordinary or Marketplace
conversations and replies for them. Build it once they have named the
conversations, how often to check, and the exact reply text; ask only for
settings that are genuinely missing.

用户可以创建一个循环规则，检查其普通会话或 Marketplace 会话并代为回复。等用户说清会话、检查频率和确切的回复文本后再构建；只询问确实缺失的设置。

When setting up an automatic reply, reject reply text that creates or changes
a real-world Marketplace commitment and suggest a non-committal alternative.
Write the exception into the saved rule itself: if an incoming message or a
proposed reply involves scheduling or other commitment details covered by
Routing and Safety, send no reply and report the conversation and exact
proposed text to the main conversation for fresh confirmation. Include this
exception in the setup preview so the user knows which messages need follow-up.

设置自动回复时，拒绝会创设或变更现实世界 Marketplace 承诺的回复文本，并建议一个不留承诺的替代写法。把这一例外写进保存的规则本身：如果来消息或拟议回复涉及"路由与安全"所覆盖的排期或其他承诺细节，就不发送回复，并把该会话与确切的拟议文本报告给主会话，以获得当场确认。在设置预览中包含这一例外，让用户知道哪些消息需要后续处理。

Before creating the rule, preview the conversations it covers, how often it
checks, and the exact reply text as a Markdown blockquote. Wait for explicit
confirmation, then create it. The rule's individual replies are not previewed
again in chat. When Messenger approvals are on, a reply sends without a separate
approval only if the user previously allowed that conversation from this
Messenger account; replies to other conversations wait for approval. The user
can revoke that allowance under Settings > Connectors > Messenger > Manage
recipient permissions.

创建规则之前，预览它覆盖的会话、检查频率，以及作为 Markdown 引用块的确切回复文本。等待明确确认后再创建。规则的单次回复不再在聊天中逐一预览。当 Messenger 审批开启时，只有当用户此前已从该 Messenger 账户允许过该会话时，回复才无需单独审批即发送；发给其他会话的回复则等待审批。用户可以在 Settings > Connectors > Messenger > Manage recipient permissions 下撤销该许可。

### Initiate a Marketplace conversation / 发起 Marketplace 会话

Use `marketplace initiate` only when the user explicitly asks to contact the
seller of a specific listing. Resolve the listing and its positive numeric ID,
then preview:

只有当用户明确要求联系某个具体商品的卖家时才使用 `marketplace initiate`。解析该商品及其正数 ID，然后预览：

- **Listing:** its title or URL, not only its ID; and
- **Message:** the exact text as a Markdown blockquote.

- **Listing:** 其标题或 URL，而不仅是 ID；以及
- **Message:** 以 Markdown 引用块展示的确切文本。

Wait for explicit confirmation, then call once using the Message Text Input  
rule:

等待明确确认，然后按"消息文本输入"规则调用一次：

```sh
hatch_messenger_cli marketplace initiate \
  --listing-id <LISTING_ID> \
  --text-stdin << 'MESSENGER_INPUT'
message text
MESSENGER_INPUT
```

`created: true` means a new thread was created and the message was sent.
`created: false` means the existing buyer/seller thread was returned and no
message was sent. In the latter case, use the returned `conversation_id`
internally to sync and read the thread, then use the normal send preview and
confirmation workflow for a follow-up. Never call `marketplace initiate` again
for that follow-up.

`created: true` 表示创建了新会话且消息已发送。`created: false` 表示返回了已有的买家/卖家会话且没有发送消息。在后一种情况下，内部使用返回的 `conversation_id` 同步并读取该会话，然后按正常的发送预览与确认流程发送后续消息。绝不要为该后续消息再次调用 `marketplace initiate`。

## React to a Message / 对消息添加表情回应

Reactions work in open and E2EE conversations with the same command. The
vendored Messenger CLI chooses the correct transport from the resolved thread.
You may react to either the user's own message or another participant's
message. `--cid` is optional at the wrapper; when omitted, the vendored CLI
owns conversation inference and any target-specific requirement. The vendor
also validates `--emoji`, which must be exactly one emoji. The command adds a
reaction when none exists and changes the user's current reaction when one does.

表情回应在开放会话和 E2EE 会话中使用同一命令。内置的 Messenger CLI 从解析出的会话中选择正确的传输方式。你可以对用户自己的消息或其他参与者的消息添加回应。在包装器层面 `--cid` 是可选的；省略时，会话推断与任何目标特定要求由内置 CLI 负责。厂商还校验 `--emoji`，它必须恰好是一个 emoji。该命令在不存在回应时添加回应，在已存在时更改用户当前的回应。

1. Resolve the exact target from `messages` or `search`; never guess its ID. If
   an older target is missing, deep-fetch that thread with
   `sync messages --cid '<CONVERSATION_ID>' --older 200` or `--since-days N`,
   then resolve it again.
2. Immediately before previewing, run
   `sync messages --cid '<CONVERSATION_ID>'` and read the target again. If it
   changed or is ambiguous, refresh the preview.
3. Show **Thread:** only for a group, **Message:** with the target text when it
   has text, and **Reaction:** with either the exact emoji or "Remove your
   reaction." Use names, never raw IDs.
4. Wait for explicit confirmation. If the target or reaction changes, preview
   again. Single-quote `--mid`, the emoji, and `--cid` when provided so the
   shell cannot expand `$` sequences or alter the emoji.
5. Call `react` once per requested state. Do not retry a failure or trigger
   approval again; report the returned reason once and stop.

   1. 从 `messages` 或 `search` 解析确切目标；绝不猜测其 ID。如果较旧的目标缺失，用 `sync messages --cid '<CONVERSATION_ID>' --older 200` 或 `--since-days N` 深取该会话，然后重新解析。
   2. 预览前立即运行 `sync messages --cid '<CONVERSATION_ID>'` 并重读目标。如果它变了或有歧义，刷新预览。
   3. 展示 **Thread:** 仅对群聊；**Message:** 在目标有文本时带出目标文本；**Reaction:** 展示确切的 emoji 或"移除你的回应"。使用名字，绝不用裸 ID。
   4. 等待明确确认。如果目标或回应变了，重新预览。提供 `--mid`、emoji 和 `--cid` 时都用单引号包裹，使 shell 无法展开 `$` 序列或改动 emoji。
   5. 每个所请求的状态只调用一次 `react`。失败不重试、不再触发审批；把返回的原因报告一次然后停止。

Add or change the user's reaction (exactly one emoji):

添加或更改用户的回应（恰好一个 emoji）：

```sh
hatch_messenger_cli react --cid '<CONVERSATION_ID>' --mid '<MESSAGE_ID>' --emoji '❤️'
```

Remove the user's current reaction:

移除用户当前的回应：

```sh
hatch_messenger_cli react --cid '<CONVERSATION_ID>' --mid '<MESSAGE_ID>' --remove
```

`--cid` may be omitted; the wrapper forwards it only when provided and otherwise
leaves inference to the vendored CLI. `--emoji` and `--remove` are mutually
exclusive; use `--emoji` to add or change a reaction and `--remove` to remove
it. Emoji validity is enforced by the vendor. After success, confirm briefly
and naturally without exposing message IDs, transport details, or raw tool output.

`--cid` 可以省略；包装器只在提供时转发它，否则把推断留给内置 CLI。`--emoji` 与 `--remove` 互斥；用 `--emoji` 添加或更改回应，用 `--remove` 移除回应。emoji 有效性由厂商强制校验。成功后，简短自然地确认，不暴露消息 ID、传输细节或原始工具输出。

## Edit or Unsend an Existing Message / 编辑或撤回已发送的消息

Messenger does not support unsending messages from Marketplace conversations.
If an unsend target belongs to Marketplace, stop before preview or approval,
do not call `hatch_messenger_cli unsend`, and tell the user plainly that the
message cannot be unsent from a Marketplace conversation.

Messenger 不支持撤回 Marketplace 会话中的消息。如果撤回目标属于 Marketplace，就在预览或审批之前停止，不要调用 `hatch_messenger_cli unsend`，并直截了当地告诉用户无法从 Marketplace 会话撤回该消息。

For supported non-Marketplace conversations, both operations work in open and
E2EE threads. `--cid` is optional for both edit and unsend: the wrapper forwards
it when provided and otherwise leaves conversation inference and validation to
the vendored CLI. Apply this shared workflow:

在受支持的非 Marketplace 会话中，两种操作都适用于开放与 E2EE 会话。edit 与 unsend 的 `--cid` 都是可选的：包装器在提供时转发，否则把会话推断与校验留给内置 CLI。套用这一共享工作流：

1. Resolve the exact target from `messages` or `search`; never guess its ID.
   Only operate on a message whose `sender_id` matches `_cache.self_user_id`.
2. If an older target is missing, deep-fetch that thread with
   `sync messages --cid '<CONVERSATION_ID>' --older 200` or
   `--since-days N` (up to 30), then resolve it again.
3. Immediately before the preview, run `sync messages --cid '<CONVERSATION_ID>'`
   and read the target again. If it changed or is ambiguous, refresh the preview.
4. Show **Thread:** only when `thread_type` is `GROUP` or
   `SECURE_MESSAGE_OVER_WA_GROUP`, plus the original **Timestamp:** in the
   user's local timezone. Never show the sender, a raw ID, or a millisecond
   timestamp.
5. Wait for explicit confirmation immediately before executing. Single-quote
   `--mid` and `--cid` when provided so the shell cannot expand `$` sequences.
6. Call the mutation at most once per target per user request. Do not retry a
   failure or trigger approval again; report the returned reason once and stop.

   1. 从 `messages` 或 `search` 解析确切目标；绝不猜测其 ID。只操作 `sender_id` 与 `_cache.self_user_id` 匹配的消息。
   2. 如果较旧的目标缺失，用 `sync messages --cid '<CONVERSATION_ID>' --older 200` 或 `--since-days N`（至多 30）深取该会话，然后重新解析。
   3. 预览前立即运行 `sync messages --cid '<CONVERSATION_ID>'` 并重读目标。如果它变了或有歧义，刷新预览。
   4. 仅当 `thread_type` 为 `GROUP` 或 `SECURE_MESSAGE_OVER_WA_GROUP` 时展示 **Thread:**，并附上用户本地时区的原始 **Timestamp:**。绝不显示发送者、裸 ID 或毫秒时间戳。
   5. 执行前立即等待明确确认。提供 `--mid` 和 `--cid` 时用单引号包裹，使 shell 无法展开 `$` 序列。
   6. 每个目标每次用户请求至多调用一次该变更操作。失败不重试、不再触发审批；把返回的原因报告一次然后停止。

### Unsend / 撤回

Unsend removes the user's message for everyone. The preview also includes  
**Message:** with the exact current text as a Markdown blockquote.

撤回会把用户的消息对所有人移除。预览还包括以 Markdown 引用块展示的 **Message:**，即当前的确切文本。

```sh
hatch_messenger_cli unsend [--cid '<CONVERSATION_ID>'] --mid '<MESSAGE_ID>'
```

Only unsend when the user explicitly asks to unsend, delete, take back, or
remove their own message. Do not treat an initial `not found` as final until the
deep-fetch step above has been attempted.

只有当用户明确要求撤回、删除、收回或移除其自己的消息时才撤回。在尝试过上述深取步骤之前，不要把最初的 `not found` 当作最终结果。

### Edit / 编辑

Edit only when the user explicitly asks to edit, fix, or reword one of their
own text messages. The preview also includes **Before:** with the current text
and **After:** with the exact replacement, each as a Markdown blockquote. If
the requested replacement changes, preview again.

只有当用户明确要求编辑、修改或改写其自己的文本消息时才编辑。预览还包括 **Before:**（当前文本）与 **After:**（确切替换文本），各为一个 Markdown 引用块。如果请求的替换内容变了，重新预览。

```sh
hatch_messenger_cli edit [--cid '<CONVERSATION_ID>'] --mid '<MESSAGE_ID>' --text-stdin << 'MESSENGER_INPUT'
replacement text
MESSENGER_INPUT
```

The replacement must exactly match the confirmed Message Text Input body.
Recipients see an "edited" marker immediately; editing does not generate a
notification or unread bolding.
Messenger typically limits edits to about 15 minutes after sending and about
five edits per message; stale, over-limit, and non-text edits fail.

替换文本必须与确认过的"消息文本输入"正文精确一致。接收方会立即看到"已编辑"标记；编辑不会生成通知或未读加粗。Messenger 通常把编辑限制在发送后约 15 分钟内、每条消息约 5 次；过期、超限和非文本的编辑会失败。

## Repair Unreadable Messages / 修复不可读的消息

When rows show `(decrypt failed)` or `(no thread key)`, run:

当行显示 `(decrypt failed)` 或 `(no thread key)` 时，运行：

```sh
hatch_messenger_cli repair
```

Repair is local and non-destructive: it refreshes epoch keys once, retries
already retained ciphertext, leaves rows it still cannot decrypt untouched,
and fetches no messages. Re-read the thread afterward. A later rerun is safe.

修复是本地且非破坏性的：它刷新一次 epoch 密钥，重试已保留的密文，对仍无法解密的行原样保留，并且不取回任何消息。之后重读该会话。稍后重跑是安全的。

These placeholders describe individual rows, not the thread or Companion's
E2EE support. Never claim that encrypted conversations are inaccessible. If it
matters, say only that those particular messages could not be read.

这些占位描述的是个别行，而不是该会话或 Companion 的 E2EE 支持能力。绝不要声称加密会话不可访问。如果有必要，只说那几条具体消息无法读取。

Do not use repair for freshness, missing history, gaps, send failures, or
write-registration cleanup. If it reports unavailable epoch refresh or stale
device state, ask the user to disconnect and reconnect Companion. Do not loop
on a failed repair.

不要把 repair 用于新鲜度、缺失历史、缺口、发送失败或写入注册清理。如果它报告 epoch 刷新不可用或设备状态过期，请用户断开并重新连接 Companion。不要在失败的修复上循环。
