---
name: "google_tasks"
description: "Manage the user's Google Tasks: lists, task details, creation, updates, and completion."
icon: "google_tasks"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Google Tasks / Google 任务

Everything runs through `hatch_gws_cli tasks <resource> <method>`. The two resources are `tasklists` and `tasks`, so a task command names `tasks` twice — `hatch_gws_cli tasks tasks list` (service `tasks`, resource `tasks`, method `list`). Parameters go in a `--params` JSON object, and creates and edits add a `--json` request body. The flows below give the exact command for each job, so use them verbatim; run `hatch_gws_cli tasks <resource> --help` for a command's flags, and `hatch_gws_cli schema tasks.tasks.insert` (and so on) for a method's `--params`/`--json` shape.

一切操作都通过 `hatch_gws_cli tasks <resource> <method>` 完成。两个资源是 `tasklists` 和 `tasks`，因此任务命令会把 `tasks` 写两次——`hatch_gws_cli tasks tasks list`（服务 `tasks`，资源 `tasks`，方法 `list`）。参数放在 `--params` JSON 对象中，创建和编辑还要加上 `--json` 请求体。下面的流程给出了每项工作的确切命令，请逐字使用；命令的标志用 `hatch_gws_cli tasks <resource> --help` 查看，方法的 `--params`/`--json` 结构用 `hatch_gws_cli schema tasks.tasks.insert`（依此类推）查看。

A task's `due` is a date given as an RFC3339 timestamp like `2026-04-16T00:00:00Z`, but Google Tasks uses only the date — there is no time of day and no reminders. The default task list is `@default`.

任务的 `due` 是以 RFC3339 时间戳形式给出的日期，如 `2026-04-16T00:00:00Z`，但 Google Tasks 只使用日期——没有具体时刻，也没有提醒。默认任务列表是 `@default`。

## Connecting / 连接
Tasks needs a one-time connect before commands return data. Run `hatch_gws_cli tasks status`. If it is not connected, post the exact `connect_url` it returns as `[Connect Google Tasks](<connect_url>)` and wait for the user to tap it. Don't invent a URL, send the user to Settings, or ask for credentials.

在命令能返回数据之前，Tasks 需要一次性的连接。运行 `hatch_gws_cli tasks status`。如果尚未连接，把它返回的 `connect_url` 原样以 `[Connect Google Tasks](<connect_url>)` 形式发布，等待用户点击。不要杜撰 URL，不要让用户去 Settings，也不要索要凭据。

To disconnect, run `hatch_gws_cli tasks disconnect` and post its `disconnect_url` as `[Disconnect Google Tasks](<disconnect_url>)`.

要断开连接，运行 `hatch_gws_cli tasks disconnect`，并把其 `disconnect_url` 以 `[Disconnect Google Tasks](<disconnect_url>)` 形式发布。

If a command reports an auth failure or not-connected, rerun `status` and follow the link it returns. If status is unavailable, say Google Tasks isn't available on this device and stop. Auth flows only through `status` and `disconnect`, with no hand-authored credential files or raw `gws auth`.

如果命令报告认证失败或未连接，重新运行 `status` 并使用它返回的链接。如果 status 不可用，说明 Google Tasks 在此设备上不可用并停止。认证只通过 `status` 和 `disconnect` 进行，不要手写凭据文件，也不要直接使用 `gws auth`。

## Common flows / 常见流程

### See your lists and tasks / 查看你的列表和任务
- Your to-do lists: `hatch_gws_cli tasks tasklists list`. Use when the user asks what lists they have or wants to target one by name.
  你的待办列表：`hatch_gws_cli tasks tasklists list`。当用户询问自己有哪些列表，或想按名称指定某个列表时使用。
- Open tasks on a list: `hatch_gws_cli tasks tasks list --params '{"tasklist":"@default","showCompleted":false}'`. Use `@default` unless the user named a list (resolve its ID with `hatch_gws_cli tasks tasklists list` first). To include finished tasks, set `"showCompleted":true,"showHidden":true` (completed tasks drop off the default view over time).
  列表上的未完成任务：`hatch_gws_cli tasks tasks list --params '{"tasklist":"@default","showCompleted":false}'`。除非用户指名了某个列表（先用 `hatch_gws_cli tasks tasklists list` 解析其 ID），否则使用 `@default`。要包含已完成的任务，设置 `"showCompleted":true,"showHidden":true`（已完成任务会随时间从默认视图中消失）。
- Tasks due in a window: `hatch_gws_cli tasks tasks list --params '{"tasklist":"@default","dueMin":"<start>","dueMax":"<end>"}'` (RFC3339) — use for "what's due this week".
  某时间窗口内到期的任务：`hatch_gws_cli tasks tasks list --params '{"tasklist":"@default","dueMin":"<start>","dueMax":"<end>"}'`（RFC3339）——用于"这周有什么到期"。
- One task's full detail or its ID before editing: `hatch_gws_cli tasks tasks get --params '{"tasklist":"@default","task":"<id>"}'`.
  编辑前获取单个任务的完整详情或其 ID：`hatch_gws_cli tasks tasks get --params '{"tasklist":"@default","task":"<id>"}'`。

### Add a task / 添加任务
`hatch_gws_cli tasks tasks insert --params '{"tasklist":"@default"}' --json '{"title":"Buy groceries","notes":"Milk, eggs","due":"<date>"}'`. Only `title` is required; add `notes` and a `due` date when the user gives them.

`hatch_gws_cli tasks tasks insert --params '{"tasklist":"@default"}' --json '{"title":"Buy groceries","notes":"Milk, eggs","due":"<date>"}'`。只有 `title` 是必需的；当用户给出 `notes` 和 `due` 日期时再加上。

### Complete or reopen a task / 完成或重新打开任务
- Mark done: `hatch_gws_cli tasks tasks patch --params '{"tasklist":"@default","task":"<id>"}' --json '{"status":"completed"}'`.
  标记完成：`hatch_gws_cli tasks tasks patch --params '{"tasklist":"@default","task":"<id>"}' --json '{"status":"completed"}'`。
- Reopen: the same with `'{"status":"needsAction"}'`.
  重新打开：同上，改用 `'{"status":"needsAction"}'`。

### Edit or reschedule a task / 编辑或改期任务
`hatch_gws_cli tasks tasks patch --params '{"tasklist":"@default","task":"<id>"}' --json '{"title":"...","due":"<date>"}'`. Patch only the fields that change.

`hatch_gws_cli tasks tasks patch --params '{"tasklist":"@default","task":"<id>"}' --json '{"title":"...","due":"<date>"}'`。只修补发生变化的字段。

### Organize / 整理
- Move, nest as a subtask, or reorder: `hatch_gws_cli tasks tasks move --params '{"tasklist":"@default","task":"<id>","parent":"<parent-id>","previous":"<sibling-id>"}'` — omit `parent` for top level, omit `previous` to move to the top.
  移动、嵌套为子任务或重新排序：`hatch_gws_cli tasks tasks move --params '{"tasklist":"@default","task":"<id>","parent":"<parent-id>","previous":"<sibling-id>"}'`——顶层省略 `parent`，移到顶部省略 `previous`。
- Manage lists: `hatch_gws_cli tasks tasklists insert --json '{"title":"Work"}'`, `hatch_gws_cli tasks tasklists patch --params '{"tasklist":"<id>"}' --json '{"title":"..."}'`, `hatch_gws_cli tasks tasklists delete --params '{"tasklist":"<id>"}'`.
  管理列表：`hatch_gws_cli tasks tasklists insert --json '{"title":"Work"}'`、`hatch_gws_cli tasks tasklists patch --params '{"tasklist":"<id>"}' --json '{"title":"..."}'`、`hatch_gws_cli tasks tasklists delete --params '{"tasklist":"<id>"}'`。

### Delete or clear / 删除或清理
- Delete one task: `hatch_gws_cli tasks tasks delete --params '{"tasklist":"@default","task":"<id>"}'`.
  删除单个任务：`hatch_gws_cli tasks tasks delete --params '{"tasklist":"@default","task":"<id>"}'`。
- Hide completed tasks from a list's normal view (they stay retrievable with `showHidden`, not deleted): `hatch_gws_cli tasks tasks clear --params '{"tasklist":"@default"}'`.
  从列表的常规视图中隐藏已完成任务（它们仍可用 `showHidden` 取回，并未删除）：`hatch_gws_cli tasks tasks clear --params '{"tasklist":"@default"}'`。

Carry the tasklist ID and task ID from the read that found the item straight through the write, and use them exactly. `@default` is only the default when the user has not pointed at a specific list; once a read locates a task on some list, target that same list for the update, complete, move, or delete — never fall back to `@default`. Never invent or rewrite a task or list ID.

把读取到该项时得到的任务列表 ID 和任务 ID 原样带到写入操作中，并精确使用。只有当用户未指定具体列表时，`@default` 才是默认值；一旦读取操作在某个列表上定位到任务，后续的更新、完成、移动或删除都要针对同一个列表——绝不要退回 `@default`。绝不要杜撰或改写任务或列表 ID。

## Rules / 规则
- Task and task-list writes may proceed from a clear, unambiguous user request without an additional confirmation. Resolve the exact task and list before editing, moving, completing, or deleting.
  任务和任务列表的写入操作，凭用户清晰明确的要求即可进行，无需额外确认。在编辑、移动、完成或删除之前，先确定具体的任务和列表。
- Talk to the user in plain language only. The commands and their JSON output are for you, not the user: keep them out of your replies — no command or flag (`hatch_gws_cli`, `--params`), no status word (`not_connected`, `unavailable`), no task or list id, no API field (`etag`, `updated`, page tokens), and no raw JSON. Say "your to-do list" or name a task by its title. A task's own content is not an id: its title, notes, and due date are what the user asked for, so keep those in your reply.
  只用平实的语言与用户交谈。命令及其 JSON 输出是给你用的，不是给用户的：不要出现在回复中——不出现命令或标志（`hatch_gws_cli`、`--params`）、状态词（`not_connected`、`unavailable`）、任务或列表 ID、API 字段（`etag`、`updated`、分页令牌），也不出现原始 JSON。说"你的待办列表"，或用标题指称任务。任务自身的内容不是 ID：标题、备注和截止日期正是用户要的，应保留在回复中。
- Get due dates right against the current date and the user's timezone; report them as plain dates.
  结合当前日期和用户所在时区把截止日期算对；以纯日期形式报告。
- `due` remains a date-only value. For actual instants, prefer the added
  `task_record_updated_at` and `task_completed_at` UTC/user-local fields.
  `due` 仍然只是日期值。对于确切时刻，优先使用新增的 `task_record_updated_at` 和 `task_completed_at` UTC/用户本地时间字段。
- Never print tokens, secrets, or credential material. Redact them if they appear in tool output.
  绝不要打印令牌、密钥或凭据材料。如果它们出现在工具输出中，要予以遮蔽。

## Limits / 限制
- Tasks are date-based only: a `due` date carries no time of day, and Google Tasks sends no reminders or notifications — don't promise a specific time or an alert. If the user wants a timed reminder, suggest Google Calendar instead.
  任务只基于日期：`due` 日期不带具体时刻，Google Tasks 也不发送提醒或通知——不要承诺具体时间或警报。如果用户想要定时提醒，建议改用 Google 日历。
- Tasks are personal to the connected account: no sharing a list, no assigning a task to someone else, no attachments.
  任务属于已连接账号的个人事务：不能共享列表，不能把任务指派给他人，也没有附件。
