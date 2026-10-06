<!-- BILINGUAL-EN-ZH -->

# Activity Feed / 活动动态

The activity feed is the sidebar-visible log of completed and in-progress work.
Every entry corresponds to a card the user can see on their activity sidebar:
emails sent, files created, web searches, reminders, goals, and other
noteworthy actions. It is not the Feed tab (the personal newspaper of
published feed units); those units are served only by the `feed` and
`feed.units` tools, never by this log.

活动动态（activity feed）是侧边栏可见的、记录已完成与进行中工作的日志。每条目对应用户可在活动侧边栏看到的一张卡片：已发送的邮件、已创建的文件、网页搜索、提醒、目标以及其他值得注意的操作。它不是 Feed 标签页（已发布 feed 单元的“个人报纸”）；那些单元只由 `feed` 和 `feed.units` 工具提供，绝不来自本日志。

## When To Use / 何时使用

Use this ledger when you need to know what the user already sees on their
sidebar, when the user references recent activity, or when you want to avoid
redundantly narrating work that is already visible on the sidebar.

当你需要知道用户在侧边栏上已经能看到什么、当用户提及最近的活动，或当你想避免重复讲述侧边栏上已可见的工作时，使用这份台账。

## How To Read / 如何读取

Read the activity feed read-only from the daemon sandbox API (no database
access required):

通过 daemon 沙箱 API 以只读方式读取活动动态（无需数据库访问）：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/activity/recent?limit=10"
```

Query parameters:

查询参数：

- `is_goal` — `true` for goal/activity-thread entries, `false` for simple activity entries; omit for all.
  `is_goal` — 目标/活动线程条目用 `true`，简单活动条目用 `false`；省略则返回全部。
- `activity_type` — filter by kind: `email_sent`, `message_sent`, `file_created`, `file_updated`, `reminder_set`, `web_search`, `task_running`, `goal`.
  `activity_type` — 按类型过滤：`email_sent`、`message_sent`、`file_created`、`file_updated`、`reminder_set`、`web_search`、`task_running`、`goal`。
- `limit` — max entries, newest first by finish-or-created time (default 10, max 100).
  `limit` — 最大条目数，按完成或创建时间由新到旧排列（默认 10，最大 100）。

The response is `{"ok":true,"result":{"entries":[...]}}`. Each entry has:

响应格式为 `{"ok":true,"result":{"entries":[...]}}`。每个条目包含：

- `timestamp` — RFC3339 finish-or-created time.
  `timestamp` — RFC3339 格式的完成或创建时间。
- `activity_type` — kind of activity (see values above).
  `activity_type` — 活动类型（取值见上文）。
- `status` — `success`, `error`, `pending`, `blocked`, or `stopped`.
  `status` — `success`、`error`、`pending`、`blocked` 或 `stopped`。
- `title` — short user-facing title shown on the sidebar card.
  `title` — 侧边栏卡片上展示的面向用户的简短标题。
- `status_title` — compact status label for the card.
  `status_title` — 卡片的紧凑状态标签。
- `subtitle` — longer context line shown below the title.
  `subtitle` — 标题下方展示的较长上下文行。
- `task_label` — optional task label for grouped activity.
  `task_label` — 分组活动的可选任务标签。
- `message_id` — associated message id when the activity originated from a chat turn.
  `message_id` — 当活动源自一轮聊天时关联的消息 ID。

## Query Examples / 查询示例

Recent sidebar activity (non-goal):

最近的侧边栏活动（非目标）：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/activity/recent?is_goal=false&limit=10"
```

Recent goals:

最近的目标：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/activity/recent?is_goal=true&limit=10"
```

Activity by type:

按类型查询活动：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/activity/recent?activity_type=email_sent&limit=10"
```
