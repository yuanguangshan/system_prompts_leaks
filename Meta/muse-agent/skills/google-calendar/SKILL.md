---
name: "google_calendar"
description: "Work with the user's Google Calendar: agenda views, event details, and scheduling changes."
icon: "google_calendar"
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Google Calendar / Google 日历

Everything runs through `hatch_gws_cli calendar ...`. Raw API calls are space-separated (`<resource> <method>`) and take a `--params` JSON object, plus a `--json` body when creating or editing. A timed event's start/end is a `dateTime` in RFC3339 with a timezone offset, like `2026-04-16T14:00:00-07:00`; an all-day event uses a date-only `date` like `2026-04-16` instead (never mix the two on one event). Run `hatch_gws_cli calendar <command> --help` for a command's flags, and `hatch_gws_cli schema calendar.events.insert` (and so on) for a raw method's `--params` and `--json` shape before you use one you are unsure of.

一切都通过 `hatch_gws_cli calendar ...` 运行。原始 API 调用以空格分隔（`<resource> <method>`），接受 `--params` JSON 对象，创建或编辑时再加 `--json` 请求体。定时事件的开始/结束是带时区偏移的 RFC3339 `dateTime`，如 `2026-04-16T14:00:00-07:00`；全天事件则使用仅含日期的 `date`，如 `2026-04-16`（同一事件绝不混用两者）。运行 `hatch_gws_cli calendar <command> --help` 查看命令标志；对没把握的原始方法，先用 `hatch_gws_cli schema calendar.events.insert` 等命令查其 `--params` 与 `--json` 结构。

## Connecting / 连接

Calendar needs a one-time connect before commands return data. Run `hatch_gws_cli calendar status`. If it comes back not connected and returns a `connect_url`, post it as `[Connect Google Calendar](<connect_url>)`, then stop and wait for the user to tap it. If no URL is returned, report that connection is unavailable and stop; never invent one. Connecting is the user's step: never open the sign-in, drive a browser to it, send the user to Settings, or ask for credentials. If status is unavailable, say Google Calendar is unavailable on this device and stop. Disconnect with `hatch_gws_cli calendar disconnect`. Post `[Disconnect Google Calendar](<disconnect_url>)` only when the command returns that URL; otherwise rerun `status` and report its state without inventing a link. If a later command fails with an auth error, missing connector, or not-connected status, rerun `status`: if not connected with a URL, post the connect link and wait; if connected, retry once; if unavailable or missing a URL, report that and stop. Auth flows only through `status` and `disconnect`. Never hand-author credential files or run raw `gws auth`.

日历需要先完成一次性连接，命令才会返回数据。运行 `hatch_gws_cli calendar status`。若返回未连接且带 `connect_url`，以 `[Connect Google Calendar](<connect_url>)` 形式发布，然后停止并等待用户点击。若未返回 URL，报告连接不可用并停止；绝不编造 URL。连接是用户自己的步骤：绝不打开登录页、用浏览器导航过去、把用户送到设置页，或索要凭据。若 status 不可用，说明 Google 日历在此设备上不可用并停止。用 `hatch_gws_cli calendar disconnect` 断开连接。仅当命令返回该 URL 时才发布 `[Disconnect Google Calendar](<disconnect_url>)`；否则重跑 `status` 并如实报告其状态，不编造链接。若后续命令因认证错误、连接器缺失或未连接状态而失败，重跑 `status`：未连接且有 URL 时，发布连接链接并等待；已连接时，重试一次；不可用或缺 URL 时，如实报告并停止。认证只经由 `status` 与 `disconnect` 进行。绝不手写凭据文件，也不运行原始的 `gws auth`。

If the user asks to add or link another Google Calendar account, run `hatch_gws_cli calendar status` and post the exact `add_account_url` it returns as `[Add Google Calendar account](<add_account_url>)`. Never reuse `connect_url` for an additional account. If `add_account_url` is absent, say that adding another account is unavailable. To remove one specific Google Calendar account while keeping the others, direct the user to that account under Connectors in Settings; do not disconnect Google Calendar entirely.

若用户要求添加或关联另一个 Google 日历账户，运行 `hatch_gws_cli calendar status` 并将其返回的 `add_account_url` 原样以 `[Add Google Calendar account](<add_account_url>)` 形式发布。绝不为额外账户复用 `connect_url`。若 `add_account_url` 不存在，说明添加其他账户不可用。要移除某个特定的 Google 日历账户而保留其他账户，引导用户前往设置中连接器下的该账户条目；不要整体断开 Google 日历。

## More than one account / 多账户

A user can link several Google accounts. Calendar service calls use the default account when `--account` is omitted, so most tasks need nothing extra and there is no need to list accounts first. When the user clearly means a specific one of several, list them with `hatch_gws_cli calendar accounts` (identity only, no tokens) and match their words to a `display_name`, then add `--account <account_id>` to the command. `--account` selects which linked account runs a calendar service call and which account a capability-aware `status --for-command` checks; plain `status`, `disconnect`, and `accounts` remain connector-wide lifecycle commands. If you cannot match the user's words to exactly one `display_name`, ask which account rather than guessing; a wrong or unlinked `--account` fails closed ("account may not be linked") — surface that and confirm the account, never silently retry on the default.

用户可以关联多个 Google 账户。省略 `--account` 时，日历服务调用使用默认账户，因此多数任务无需额外操作，也不必先列出账户。当用户明确指多个账户中的某一个时，用 `hatch_gws_cli calendar accounts` 列出它们（仅身份信息，不含令牌），把用户的话与某个 `display_name` 匹配，然后在命令中加 `--account <account_id>`。`--account` 决定日历服务调用用哪个已关联账户执行，以及具备能力感知的 `status --for-command` 检查哪个账户；普通的 `status`、`disconnect` 和 `accounts` 仍是覆盖整个连接器的生命周期命令。若无法把用户的话与唯一的 `display_name` 匹配，询问是哪个账户，而不是猜测；错误或未关联的 `--account` 会保守失败（"account may not be linked"）——如实呈现并确认账户，绝不悄悄改用默认账户重试。

If Google returns `403`, `insufficientPermissions`, or "insufficient authentication scopes," run `hatch_gws_cli calendar status --for-command <command-or-method>` with the same `--account <account_id>` selection as the failed operation. When `scope_status` is `not_granted`, post the returned `scope_add_url` exactly as `[Additional Google Calendar access](<scope_add_url>)` and tell the user to choose that same account on Google's consent screen. Do not substitute `add_account_url`: that starts a new-account flow rather than adding access to the selected account. Wait for consent to finish before retrying the command.

若 Google 返回 `403`、`insufficientPermissions` 或 "insufficient authentication scopes"，以与失败操作相同的 `--account <account_id>` 选择运行 `hatch_gws_cli calendar status --for-command <command-or-method>`。当 `scope_status` 为 `not_granted` 时，将返回的 `scope_add_url` 原样以 `[Additional Google Calendar access](<scope_add_url>)` 形式发布，并告知用户在 Google 授权页选择同一个账户。不要用 `add_account_url` 替代：那会启动新账户流程，而不是为所选账户添加权限。等待授权完成后再重试该命令。

## Common flows / 常用流程

### Read the agenda / 读取日程

`+agenda` is the default for any read-only "what's on my calendar", "what's coming up", or multi-day summary. It searches every visible calendar.
- Match the window to the question: `+agenda --today` for today, `+agenda --week` or `+agenda --days 7` for a range.
- One calendar: `+agenda --calendar <name-or-id>`.
- Add `--format json` when you need to parse the result.

对任何只读的"我日历上有什么""接下来有什么"或多日摘要，默认用 `+agenda`。它会搜索所有可见日历。
- 窗口与问题匹配：今天用 `+agenda --today`，一段范围用 `+agenda --week` 或 `+agenda --days 7`。
- 单个日历：`+agenda --calendar <name-or-id>`。
- 需要解析结果时加 `--format json`。

### Look up calendars, events, and free time / 查询日历、事件与空闲时间

- Which calendars exist: `calendarList list`. Use it when the user asks what calendars are connected or wants to target one by name.
  存在哪些日历：`calendarList list`。当用户询问连接了哪些日历，或想按名称指定某个日历时使用。
- Events in a known calendar, or to get an event's ID before editing it: `events list --params '{"calendarId":"primary","timeMin":"<start>","timeMax":"<end>"}'`. Use this over `+agenda` only when you need a specific calendar, fields `+agenda` omits, or event IDs.
  已知日历中的事件，或编辑前获取事件 ID：`events list --params '{"calendarId":"primary","timeMin":"<start>","timeMax":"<end>"}'`。仅当需要特定日历、`+agenda` 省略的字段或事件 ID 时才用它替代 `+agenda`。
- One event's full detail: `events get --params '{"calendarId":"primary","eventId":"<id>"}'`.
  单个事件的完整详情：`events get --params '{"calendarId":"primary","eventId":"<id>"}'`。
- Whether a time is free or busy: `freebusy query --json '{"timeMin":"<start>","timeMax":"<end>","items":[{"id":"primary"}]}'`.
  某时段空闲还是忙碌：`freebusy query --json '{"timeMin":"<start>","timeMax":"<end>","items":[{"id":"primary"}]}'`。

### Create, update, or delete events / 创建、更新或删除事件

- Create: `events insert --params '{"calendarId":"primary"}' --json '{"summary":"Focus block","start":{"dateTime":"<start>"},"end":{"dateTime":"<end>"}}'`. Add `attendees`, `location`, or `description` as needed. A block, hold, or focus event must show as busy: leave `transparency` unset or set it to `"opaque"`, never `"transparent"` (that shows the time as free) unless the user wants it to read as free. For an all-day event use date-only values instead, `"start":{"date":"2026-04-16"},"end":{"date":"2026-04-17"}`, where the end date is exclusive.
  创建：`events insert --params '{"calendarId":"primary"}' --json '{"summary":"Focus block","start":{"dateTime":"<start>"},"end":{"dateTime":"<end>"}}'`。按需添加 `attendees`、`location` 或 `description`。封锁、占位或专注类事件必须显示为忙碌：`transparency` 保持未设或设为 `"opaque"`，绝不设 `"transparent"`（那会把该时段显示为空闲），除非用户希望它显示为空闲。全天事件改用仅日期的值：`"start":{"date":"2026-04-16"},"end":{"date":"2026-04-17"}`，其中结束日期不含当日。
- Update: `events patch --params '{"calendarId":"primary","eventId":"<id>"}' --json '{"location":"Room A"}'`. Patch only the fields that change. A patch REPLACES a whole array, so to add or remove a guest, first read the event and send back the COMPLETE attendee list with your change applied — never send only the new guest, or you silently drop the others.
  更新：`events patch --params '{"calendarId":"primary","eventId":"<id>"}' --json '{"location":"Room A"}'`。只 patch 有变化的字段。patch 会整体替换数组，因此要添加或移除一位参会者时，先读取事件，再把应用了变更的完整与会者列表发回——绝不只发新参会者，否则会悄悄丢掉其他人。
- Delete: `events delete --params '{"calendarId":"primary","eventId":"<id>"}'`.
  删除：`events delete --params '{"calendarId":"primary","eventId":"<id>"}'`。

Carry the account, calendar ID, and event ID from the read that found the event straight through the write, and use them exactly. `primary` above is only the default when the user has not pointed at another calendar; once a read locates an event on some calendar, target that same calendar and account for the update or delete — never fall back to `primary`. Never invent or rewrite an event ID.

把找到该事件的那次读取所用的账户、日历 ID 和事件 ID 一路带到写入操作，并严格原样使用。上文的 `primary` 只是用户未指定其他日历时的默认值；一旦某次读取在某个日历上定位到事件，更新或删除就针对同一日历和账户——绝不回退到 `primary`。绝不编造或改写事件 ID。

For any create, deletion, or guest-visible change, add `"sendUpdates":"all"` to `--params` whenever attendees exist before or after the write. This includes adding the first guest, recurrence or conference changes, and time, location, title, description, or attendee changes. A deletion sent with `"sendUpdates":"all"` gives each guest a cancellation notice. If the user asks not to notify anyone, explain that `"sendUpdates":"none"` can stop the change from reaching guests' calendars, then confirm before using it.

任何创建、删除或对参会者可见的更改，只要写入前后存在参会者，就在 `--params` 中加 `"sendUpdates":"all"`。包括添加第一位参会者、重复规则或会议链接变更，以及时间、地点、标题、描述或参会者的变更。带 `"sendUpdates":"all"` 发出的删除会给每位参会者发送取消通知。若用户要求不通知任何人，解释 `"sendUpdates":"none"` 可以阻止更改到达参会者的日历，然后确认后再使用。

## Rules / 规则

- Private event creation, private updates, and event deletion may proceed from a clear user request without an additional confirmation. Creating an event with guests, changing an event in a way that notifies guests, and calendar or ACL management require confirmation because event content or access changes reach other people. Before those outward-facing writes, restate the specific event, time, recipients, and notification effect.
  私有事件的创建、私有更新和事件删除，在用户清晰的要求下即可执行，无需额外确认。创建带参会者的事件、以会通知参会者的方式更改事件，以及日历或 ACL 管理，则需要确认，因为事件内容或访问权的变更会影响其他人。在这些对外写入之前，复述具体事件、时间、接收者和通知效果。
- Commands that return the saved event in their response (such as timed-event reads, creates and updates) add `event_starts_at` / `event_ends_at`; recurring instances also add `event_original_starts_at`. Each carries UTC and user-local forms; prefer `user_local` in replies. All-day bounds remain calendar dates and must not be timezone-shifted. Trust the events `+agenda` returns for the window you asked for, and get "today" and "tomorrow" right against the current date.
  响应中返回已保存事件的命令（如定时事件读取、创建与更新）会附加 `event_starts_at` / `event_ends_at`；重复事件的实例还附加 `event_original_starts_at`。两者都带 UTC 和用户本地时间形式；回复中优先用 `user_local`。全天事件的边界保持为日历日期，不得做时区偏移。信任 `+agenda` 对所查询窗口返回的事件，并依据当前日期正确理解"今天"和"明天"。
- Talk to the user in plain language only. Never show raw commands, status words like not_connected or unavailable, opaque provider/event/calendar IDs, etags, page tokens or cursors, raw JSON, or other internal response fields, unless the user asks for them. You may name an account by its display name or recognizable email when you need to disambiguate which account you mean. Keep IDs and pagination cursors internally to chain follow-up commands.
  对用户只使用平实语言。除非用户要求，绝不显示原始命令、not_connected 或 unavailable 之类的状态词、不透明的供应商/事件/日历 ID、etag、页面令牌或游标、原始 JSON 或其他内部响应字段。需要区分所指账户时，可以用显示名或可识别的邮箱指称账户。ID 和分页游标只保留在内部，用于串起后续命令。
- Never print tokens, secrets, or credential material. Redact them if they appear in tool output.
  绝不打印令牌、机密或凭据材料。若它们出现在工具输出中，予以遮蔽。
