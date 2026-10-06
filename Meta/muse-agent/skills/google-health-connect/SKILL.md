---
name: "google_health_connect"
title: "Health Connect"
description: "The user's synced Google Health Connect data from their Android device: daily metrics (steps, distance, calories, heart rate, HRV, VO2max), sleep sessions (stages, quality, efficiency), and workouts."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Health Connect / Health Connect

Read the user's synced Google Health Connect data with the `health-cli`
binary. Every command below takes `--provider healthconnect`.

使用 `health-cli` 二进制程序读取用户已同步的 Google Health Connect 数据。下方每条命令都需要带 `--provider healthconnect`。

This skill serves Android devices only. If the user's paired device is not an
Android device, it has no data for them — check the `platform` field returned
by `device.list` when you are unsure.

本技能仅适用于 Android 设备。如果用户配对的设备不是 Android 设备，其中不会有该用户的数据——不确定时请检查 `device.list` 返回的 `platform` 字段。

## When to Use / 使用时机

When the user asks about their synced Health Connect data, sync status, or  
sources:

当用户询问其已同步的 Health Connect 数据、同步状态或数据来源时：

- All-day metrics/vitals: steps, distance, calories, heart rate, HRV, VO2max.
  全天指标/体征：步数、距离、卡路里、心率、HRV、VO2max。
- Sleep sessions: stages, quality, efficiency, awakenings.
  睡眠会话：睡眠阶段、质量、效率、觉醒。
- Workouts: type, duration, calories, distance, HR.
  运动记录：类型、时长、卡路里、距离、心率。

## Tooling / 工具

Binary: `health-cli`. **`--provider healthconnect` is required on every command
below** — it is not optional and has no default; omitting it is a usage error.
Commands group by data *shape* under `query`:

二进制程序：`health-cli`。**下方每条命令都必须带 `--provider healthconnect`**——它不是可选项，也没有默认值；省略即属用法错误。命令在 `query` 下按数据*形态*分组：

【评论】要求每条命令显式携带提供方参数且不设默认值，可防止命令被误用于其他健康数据提供方，是一种防误操作设计。

- `query metrics --provider healthconnect` — all-day metric rollups
  `query metrics --provider healthconnect` — 全天指标汇总
- `query samples --provider healthconnect` — raw, unbucketed data points (intraday HR/steps, GPS, sleep stages)
  `query samples --provider healthconnect` — 原始、未分桶的数据点（日内心率/步数、GPS、睡眠阶段）
- `query sessions --provider healthconnect` — discrete sessions: sleep, workout
  `query sessions --provider healthconnect` — 离散会话：睡眠、运动
- `status --provider healthconnect` — data-sync status
  `status --provider healthconnect` — 数据同步状态
- `auth connect --provider healthconnect`, `auth disconnect --provider healthconnect`
  `auth connect --provider healthconnect`、`auth disconnect --provider healthconnect`
- `delete` — **erase stored records** (destructive; see below)
  `delete` — **删除已存储的记录**（具破坏性；见下文）

**Multi-device (rare):** when a window spans >1 device the envelope adds
`"multi_node": true` — `query metrics` returns per-device `node_groups`;
`query samples`/`sessions` add a `node_id` to each record (samples CSV gains a
leading `node_id` column). Single-device output is unchanged; never sum or
double-count across devices.

**多设备（少见）：**当查询窗口跨越多台设备时，信封会附加 `"multi_node": true`——`query metrics` 返回按设备划分的 `node_groups`；`query samples`/`sessions` 会为每条记录附加 `node_id`（samples CSV 会增加一个前置 `node_id` 列）。单设备输出保持不变；绝不要跨设备求和或重复计数。

### query metrics — all-day metric rollups / query metrics — 全天指标汇总

```bash
health-cli query metrics --provider healthconnect \
  --start-date <YYYY-MM-DD> [--end-date <YYYY-MM-DD>] \
  [--interval hourly|daily|weekly] [--fields a,b,c] [--timeout-secs N]
```

`daily-metrics` is the only domain so there is **no `--category`**. One row per
`--interval` bucket (default `daily`). `--start-date` required; `--end-date`
defaults to today. Output is an envelope `{ "coverage": {...}, "records": [...] }`
(see Coverage below).

`daily-metrics` 是唯一的领域，因此**没有 `--category`**。每个 `--interval` 桶一行（默认 `daily`）。`--start-date` 为必填；`--end-date` 默认为今天。输出是一个信封 `{ "coverage": {...}, "records": [...] }`（见下文 Coverage）。

`--list-fields` reads the metric field names from the synced data →  
`{ ok, provider, category, observed_count, fields: [{ name, observed, count? }] }`.  
`observed: true` (with a row `count`) means the name is present in this user's
data — those are exactly the names `--fields` matches. `observed: false` means
the build knows the field but nothing has synced yet. If the field set can't be
read the response carries `degraded: true` and every `observed` is `null`.

`--list-fields` 从已同步的数据中读取指标字段名 →  
`{ ok, provider, category, observed_count, fields: [{ name, observed, count? }] }`。  
`observed: true`（附带行数 `count`）表示该名称存在于该用户的数据中——这些正是 `--fields` 所匹配的名称。`observed: false` 表示当前构建认识该字段但尚无数据同步。若无法读取字段集，响应会带有 `degraded: true` 且所有 `observed` 均为 `null`。

### query sessions — discrete sessions / query sessions — 离散会话

```bash
health-cli query sessions --provider healthconnect --category sleep|workout \
  --start-date <YYYY-MM-DD> [--end-date <YYYY-MM-DD>] [--fields a,b,c]
```

- `sleep` — one row per sleep session.
  `sleep` — 每个睡眠会话一行。
- `workout` — one row per workout/activity.
  `workout` — 每次运动/活动一行。

Sessions are never bucketed (no `--interval`). Discovery: `--list-categories` →
`{ ok, provider, categories }`; `--list-fields` (needs `--category`) →
`{ ok, provider, category, fields }` (e.g. workout has `is_indoor`).

会话数据永不分桶（没有 `--interval`）。发现机制：`--list-categories` → `{ ok, provider, categories }`；`--list-fields`（需要 `--category`）→ `{ ok, provider, category, fields }`（例如 workout 含有 `is_indoor`）。

**Output:** normalized snake_case records; the same concept uses the same field
name (e.g. workout `average_speed_mps`, `elevation_gain_meters`,
`hr_average_bpm`). Session records carry a provider-prefixed `id`
(`healthconnect_…`) + `start_datetime`/`end_datetime`/`timezone`; metric buckets
carry `date`/`hour`/`week_start` + `record_count`.

**输出：**规范化的 snake_case 记录；同一概念使用同一字段名（例如 workout 的 `average_speed_mps`、`elevation_gain_meters`、`hr_average_bpm`）。会话记录带有以提供方为前缀的 `id`（`healthconnect_…`）以及 `start_datetime`/`end_datetime`/`timezone`；指标桶带有 `date`/`hour`/`week_start` 以及 `record_count`。

**Coverage:** queries return an envelope `{ "coverage": {...}, "records": [...] }`.
`coverage.complete` is `false` when days in the window aren't synced from the
device; `coverage.warning` then holds the exact `backfill_data_source` command
to run. **Surface that warning to the user / act on it** — results are partial
until the backfill completes.

**覆盖率（Coverage）：**查询返回信封 `{ "coverage": {...}, "records": [...] }`。当窗口内的某些天未从设备同步时，`coverage.complete` 为 `false`；此时 `coverage.warning` 中给出需要执行的确切 `backfill_data_source` 命令。**必须将该警告呈现给用户 / 并据此行动**——在回填完成之前结果都是不完整的。

### query samples — raw, unbucketed data points / query samples — 原始、未分桶的数据点

```bash
health-cli query samples --provider healthconnect --start-date <YYYY-MM-DD> \
  [--end-date <YYYY-MM-DD>] [--start-time <HH:MM[:SS]>] [--end-time <HH:MM[:SS]>] \
  [--fields <type1,...>] [--limit <n>] [--format stdout|csv] [--output <path>]
health-cli query samples --provider healthconnect --list-fields   # discover sample types
```

Individual samples — finer than the `query metrics` rollups (intraday HR/steps,
GPS, sleep-stage timelines; `query sessions` gives stage totals only). `--fields`
selects sample **types** (not output columns) — run `--list-fields` to see the
types this user has synced. GPS is `location` (raw lat/lon — don't infer place
names from it). For dense reads use `--output <path>`: it writes CSV into the
agent filesystem for a later script step and prints only a summary, keeping
thousands of rows out of context.

单条样本——比 `query metrics` 的汇总更细粒度（日内心率/步数、GPS、睡眠阶段时间线；`query sessions` 只给出阶段汇总）。`--fields` 选择的是样本**类型**（而非输出列）——运行 `--list-fields` 可查看该用户已同步的类型。GPS 即 `location`（原始经纬度——不要据此推断地名）。对于高密度读取，使用 `--output <path>`：它会把 CSV 写入代理文件系统供后续脚本步骤使用，并且只打印摘要，避免数千行数据进入上下文。

【评论】用 `--output` 把大体量数据落盘、只让摘要进入上下文，是控制上下文窗口占用的典型做法。

### status — data-sync status / status — 数据同步状态

```bash
health-cli status --provider healthconnect [--timeout-secs N] \
  [--start-date <YYYY-MM-DD>] [--end-date <YYYY-MM-DD>] [--check-missing-entries]
```

Returns `{ categories: [{ name, record_count, earliest_datetime,
latest_datetime }] }`. With `--check-missing-entries` (requires `--start-date`):
a deep per-30-min-interval gap report (`unsynced_dates`, `missing_intervals`, …).

返回 `{ categories: [{ name, record_count, earliest_datetime, latest_datetime }] }`。配合 `--check-missing-entries`（需要 `--start-date`）：可生成按 30 分钟间隔深挖的缺口报告（`unsynced_dates`、`missing_intervals` 等）。

### auth connect / auth connect

```bash
health-cli auth connect --provider healthconnect [--timeout-secs N]
```

Run this and follow the instruction to connect to Health Connect data.

运行此命令并按照指引连接 Health Connect 数据。

### auth disconnect / auth disconnect

```bash
health-cli auth disconnect --provider healthconnect [--timeout-secs N]
```

Health Connect cannot be disconnected through this CLI or an in-chat widget.
Tell the user to open **Settings → Connectors → Health Connect → Manage**, then
revoke the app's Health Connect permissions in Android settings.

无法通过此 CLI 或聊天内组件断开 Health Connect。请告诉用户打开**设置 → 连接器（Connectors）→ Health Connect → 管理（Manage）**，然后在 Android 系统设置中撤销该应用的 Health Connect 权限。

### delete — erase stored Health Connect records / delete — 删除已存储的 Health Connect 记录

```bash
health-cli delete --provider healthconnect --all             # every Health Connect record
health-cli delete --provider healthconnect --device NODE_ID  # one paired device
health-cli delete --provider healthconnect --start-date 2026-03-01 --end-date 2026-03-31
```

Use for any request to delete, remove, erase, wipe, or clear Health Connect
data. Never improvise it by moving files, disconnecting the connector, or
writing to the database — those neither remove records nor leave an audit
trail. A delete never spans providers; to clear both, run it once per provider.

凡是涉及删除、移除、擦除、清空或清除 Health Connect 数据的请求都应使用此命令。绝不要通过移动文件、断开连接器或写数据库等方式自行变通——这些做法既不会删除记录，也不会留下审计痕迹。删除操作从不跨提供方；若要清空两个提供方，需对每个提供方各执行一次。

【评论】将破坏性删除收敛到专用子命令，并禁止用间接手段变通，目的是保证删除操作留有审计痕迹、行为可预期。

`--all` takes no other filter; `--device`/`--start-date`/`--end-date` are used
instead of it. All record kinds go at once — no per-category or per-record-ID
delete. Dates are `YYYY-MM-DD` in `JARVIS_USER_TIMEZONE` or epoch seconds; a
record is in range when it starts before the end and ends after the start, so a
session straddling a bound is included. `--start-date` alone deletes from that
day onward, `--end-date` alone up to and including it.

`--all` 不接受任何其他过滤条件；`--device`/`--start-date`/`--end-date` 用于替代它。所有记录类型一次性处理——不支持按类别或按记录 ID 删除。日期为 `JARVIS_USER_TIMEZONE` 时区的 `YYYY-MM-DD` 或 epoch 秒；只要记录的开始时间早于结束边界、结束时间晚于开始边界即视为在范围内，因此跨越边界点的会话也会被包含。仅给 `--start-date` 表示从当天起删除；仅给 `--end-date` 表示删除至当天（含当天）。

**Destructive and irreversible**, behind a fresh one-time approval showing the
normalized filters. Run it **once per request** — settle provider, device and
dates in conversation first. Denial exits **code 3** with nothing deleted; that
is final, so report it rather than retrying other filters or the other provider.

**具破坏性且不可逆**，执行前需经过一次全新的一次性审批，审批界面会展示规范化后的过滤条件。每个请求只运行**一次**——先在对话中确定提供方、设备和日期。审批被拒绝时以**退出码 3** 结束且不删除任何内容；该结果是最终态，应如实报告，而不是改用其他过滤条件或另一提供方重试。

**Zero is a success**, not a miss — nothing matched, so say so and stop rather
than widening dates or re-running with `--all`. Report `deleted.records`, not
`total_rows`, which also counts child value rows. If Health Connect on the
device still holds the records, a later sync can bring them back.

**返回零也是一种成功**，并非遗漏——只是没有匹配到任何内容，因此应如实说明并停止，而不是扩大日期范围或改用 `--all` 重跑。应报告 `deleted.records`，而不是 `total_rows`，后者还会计入子值行。如果设备上的 Health Connect 仍持有这些记录，后续同步可能将它们带回。

【评论】"零也是成功"与"拒绝即最终"条款旨在防止模型把空结果或审批失败误判为异常而自动扩大删除范围，属于针对代理行为的安全约束。

## Auth / 授权

Device-synced; no login. Data availability depends on the Health Connect
permission being approved on the user's Android device + sync. Use `status` to
see synced ranges and the `backfill_data_source` device action (surfaced in
query `coverage.warning`) to pull a range.

数据通过设备同步获取；无需登录。数据可用性取决于用户 Android 设备上是否批准了 Health Connect 权限以及同步情况。使用 `status` 查看已同步的范围，并使用 `backfill_data_source` 设备操作（通过查询的 `coverage.warning` 呈现）来拉取某个范围的数据。

## Operating Rules / 操作规则

1. Pick the command by data shape: `query metrics` (all-day rollups),
   `query samples` (raw points — only when individual samples matter; prefer
   `metrics` for trends), or `query sessions --category sleep|workout` (discrete
   events).
   按数据形态选择命令：`query metrics`（全天汇总）、`query samples`（原始数据点——仅在确实需要单条样本时使用；趋势分析优先用 `metrics`），或 `query sessions --category sleep|workout`（离散事件）。
2. Always pass `--provider healthconnect` and `--start-date`; `--end-date`
   defaults to today. Use current-date context for relative asks ("today",
   "last week"); use sample time bounds for narrow intraday windows.
   始终传入 `--provider healthconnect` 与 `--start-date`；`--end-date` 默认为今天。对相对表述（"今天"、"上周"）使用当前日期上下文；对狭窄的日内窗口使用样本时间边界。
3. **Coverage:** if a `query metrics`/`sessions` returns `coverage.complete:
   false`, tell the user the data is partial and relay/act on `coverage.warning`
   (run the `backfill_data_source` action, then re-query).
   **覆盖率：**若 `query metrics`/`sessions` 返回 `coverage.complete: false`，应告知用户数据不完整，并转达/执行 `coverage.warning`（先运行 `backfill_data_source` 操作，再重新查询）。
4. `query metrics` defaults to `--interval daily`; sleep spans midnight (include
   both evening start and morning end dates).
   `query metrics` 默认 `--interval daily`；睡眠会跨越午夜（需同时包含晚间的开始日期与次日的结束日期）。
5. On empty results, the range likely has no data; widen it, or check `status` /
   coverage and backfill.
   若结果为空，该范围很可能没有数据；可扩大范围，或检查 `status` / 覆盖率并执行回填。
