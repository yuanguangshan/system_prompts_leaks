<!-- BILINGUAL-EN-ZH -->

# Web artifact actions / Web 工件操作

PostgreSQL ledger of every web artifact action invoked on this device, whether by a user
UI tap, an agent tool call, or a cron schedule. Every accepted invocation
creates one row, then updates that row when the worker succeeds, fails, or is
cancelled.

一份 PostgreSQL 账本，记录本设备上被调用的每一次 Web 工件操作，无论触发方式是用户在 UI 上点击、智能体工具调用，还是 cron 计划任务。每个被接受的调用都会创建一行，并在 worker 成功、失败或被取消时更新该行。

## When To Use / 何时使用

The web artifact's database holds current state and the action invocation timeline. Use this
ledger for historical questions, audit questions, and diagnostic questions where
the user is asking about recent web artifact activity or action failures.

Web 工件的数据库保存当前状态和操作调用时间线。当用户询问与最近 Web 工件活动或操作失败相关的历史、审计或诊断类问题时，使用本账本。

## How To Read / 如何读取

Read the ledger read-only from the daemon sandbox API (no database access
required). Filter by `space_slug`, by `invocation_id`, or by `source_kind`:

通过守护进程沙箱 API 以只读方式读取账本（无需数据库访问权限）。可按 `space_slug`、`invocation_id` 或 `source_kind` 过滤：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/spaces-actions?space_slug=<space_slug>&limit=20"
```

Results are newest first by invocation time. The response is
`{"ok":true,"result":{"invocations":[...]}}`. Each invocation has:

结果按调用时间倒序，最新在前。响应格式为 `{"ok":true,"result":{"invocations":[...]}}`。每个调用包含：

- `invocation_id` - stable id correlating HTTP request, runtime telemetry, worker logs, and debug UI rows.
  `invocation_id` - 稳定的 ID，用于关联 HTTP 请求、运行时遥测、worker 日志和调试 UI 中的行。
- `space_slug` - the web artifact the invocation belongs to (useful for cross-artifact reads by `invocation_id` or `source_kind`).
  `space_slug` - 该调用所属的 Web 工件（便于通过 `invocation_id` 或 `source_kind` 进行跨工件读取）。
- `action` - action name.
  `action` - 操作名称。
- `status` - `in_flight`, `succeeded`, `failed`, or `cancelled`.
  `status` - 取值为 `in_flight`、`succeeded`、`failed` 或 `cancelled`。
- `invoked_at_ms` - invocation time as Unix milliseconds.
  `invoked_at_ms` - 调用时间，Unix 毫秒时间戳。
- `settled_at_ms` - settlement time as Unix milliseconds (absent while in flight).
  `settled_at_ms` - 结算时间，Unix 毫秒时间戳（调用进行中时缺失）。
- `source_kind`, `source_ref`, `trigger_ref` - invocation attribution.
  `source_kind`、`source_ref`、`trigger_ref` - 调用来源归因。
- `args_preview` - bounded JSON snapshot of request args.
  `args_preview` - 请求参数的有界 JSON 快照。
- `result_preview` - bounded JSON snapshot of a unary success result.
  `result_preview` - 单次成功结果的有界 JSON 快照。
- `error` - truncated worker error message for failed actions.
  `error` - 失败操作的 worker 错误消息（已截断）。

【评论】请求参数与结果都只保留"有界快照"、错误消息被截断，账本以最小化数据量换取可审计性，避免完整载荷进入日志。

## Query Examples / 查询示例

A single invocation by id (across all web artifacts):

按 ID 查询单个调用（跨所有 Web 工件）：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/spaces-actions?invocation_id=<invocation_id>"
```
