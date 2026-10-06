<!-- BILINGUAL-EN-ZH -->

# Data Source Events / 数据源事件

A read-only log of events synced from connected devices: notifications, location, contacts, and more.

一份只读的事件日志，记录从已连接设备同步而来的事件：通知、位置、联系人等。

## Reading events / 读取事件

Read from the daemon sandbox API (no database access). List events filtered by `source`:

从守护进程沙箱 API 读取（不直接访问数据库）。按 `source` 过滤并列出事件：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/device-syncs?source=notifications&limit=20"
```

- `source`: any stream, whether a common device one (`contacts | location | notifications`) or a client-defined source. Omit to see all streams; more can appear as clients register them.
  `source`：任意流，可以是常见设备流（`contacts | location | notifications`），也可以是客户端自定义的来源。省略则查看所有流；随着客户端注册，还会出现更多流。
- `producer_id`: scope to one device.
  `producer_id`：将范围限定到单台设备。
- Newest first by `global_seq`.
  按 `global_seq` 倒序排列，最新在前。

The list returns metadata and a short `summary_preview`, not the raw payload. Each event has:

列表返回的是元数据和简短的 `summary_preview`，而不是原始载荷。每个事件包含：

- `global_seq`: canonical arrival order across all streams.
  `global_seq`：所有流中的规范到达顺序。
- `producer_id`: stable device id, e.g. `phone-1`.
  `producer_id`：稳定的设备 ID，例如 `phone-1`。
- `source`: payload family (above).
  `source`：载荷类别（见上文）。
- `received_at`: RFC3339 ingest time.
  `received_at`：RFC3339 格式的接收时间。
- `status`: processing/settlement state.
  `status`：处理/结算状态。
- `summary_preview`: worker message preview when notified.
  `summary_preview`：通知时的 worker 消息预览。

## Fetching a payload / 获取载荷

When `summary_preview` is not enough, fetch one event's stored payload by its `global_seq` (404 if unknown):

当 `summary_preview` 不够用时，按 `global_seq` 获取某个事件存储的载荷（未知则返回 404）：

```bash
curl -sS --unix-socket "$JARVIS_SANDBOX_API_SOCK" \
  "http://localhost/device-syncs/<global_seq>"
```

The response adds `payload` and `payload_representation` (`raw`, `summary`, or
`redacted`). Newly inserted events for `notification`, `notifications`, `sms`,
and `imessage` contain redacted metadata, never raw message text.

响应会额外包含 `payload` 和 `payload_representation`（取值为 `raw`、`summary` 或 `redacted`）。`notification`、`notifications`、`sms` 和 `imessage` 类别中新插入的事件只包含脱敏后的元数据，绝不包含原始消息文本。

【评论】通知、短信等新事件强制脱敏（只保留元数据、不含消息原文），是同步管道中内建的隐私最小化条款。

## When to use it / 何时使用

Use this ledger for recent synced device activity, like "did my phone sync any notifications recently?" For canonical contacts, calendar, or health reads, use the `contacts.search`/`calendar.search` device commands or the `device-data` skill (cached) and the typed health store instead.

该账本用于查询最近同步的设备活动，例如"我的手机最近同步过什么通知吗？"。至于规范的联系人、日历或健康数据读取，请改用 `contacts.search`/`calendar.search` 设备命令，或 `device-data` 技能（带缓存）和类型化的健康数据存储。
