---
name: "apple_healthkit"
title: "Apple Health"
description: "The user's synced Apple Health (HealthKit) data: daily metrics (steps, distance, calories, heart rate, HRV, VO2max), sleep sessions (stages, quality, efficiency), and workouts."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Apple Health / Apple Health

Read the user's synced Apple HealthKit data with the `health-cli` binary. Every
command below takes `--provider healthkit`.

使用 `health-cli` 二进制读取用户已同步的 Apple HealthKit 数据。下文每条命令都带 `--provider healthkit`。

This skill serves iOS devices only. If the user's paired device is not an
iPhone, it has no data for them — check the `platform` field
returned by `device.list` when you are unsure.

本技能仅服务于 iOS 设备。若用户配对的设备不是 iPhone，则其中没有其数据——不确定时检查 `device.list` 返回的 `platform` 字段。

# Data Source / 数据来源

Despite syned from apple healthkit, do not assume the data is originated from ios devices only. It could come from other sources/devices which share data with Apple Healthkit.

尽管数据是从 Apple HealthKit 同步的，也不要假定它只来自 iOS 设备。它可能来自与 Apple HealthKit 共享数据的其他来源/设备。

## When to Use / 何时使用

When the user asks about their synced HealthKit data, sync status, or sources:
- All-day metrics/vitals: steps, distance, calories, heart rate, HRV, VO2max.
- Sleep sessions: stages, quality, efficiency, awakenings.
- Workouts: type, duration, calories, distance, HR.

当用户询问其已同步的 HealthKit 数据、同步状态或来源时：
- 全天指标/生命体征：步数、距离、卡路里、心率、HRV、VO2max。
- 睡眠会话：阶段、质量、效率、醒来次数。
- 锻炼：类型、时长、卡路里、距离、心率。

## Tooling / 工具

Binary: `health-cli`. **`--provider healthkit` is required on every command
below** — it is not optional and has no default; omitting it is a usage error.
Commands group by data *shape* under `query`:
- `query metrics --provider healthkit` — all-day metric rollups
- `query samples --provider healthkit` — raw, unbucketed data points (intraday HR/steps, GPS, sleep stages)
- `query sessions --provider healthkit` — discrete sessions: sleep, workout
- `status --provider healthkit` — data-sync status
- `auth connect --provider healthkit`, `auth disconnect --provider healthkit`
- `delete` — **erase stored records** (destructive; see below)

二进制：`health-cli`。**下文每条命令都必须带 `--provider healthkit`**——它不是可选项，也没有默认值；省略即属用法错误。命令在 `query` 下按数据*形态*分组：
- `query metrics --provider healthkit` —— 全天指标汇总
- `query samples --provider healthkit` —— 原始、未分桶的数据点（日内心率/步数、GPS、睡眠阶段）
- `query sessions --provider healthkit` —— 离散会话：睡眠、锻炼
- `status --provider healthkit` —— 数据同步状态
- `auth connect --provider healthkit`、`auth disconnect --provider healthkit`
- `delete` —— **擦除已存储的记录**（破坏性；见下文）

**Multi-device (rare):** when a window spans >1 device the envelope adds
`"multi_node": true` — `query metrics` returns per-device `node_groups`;
`query samples`/`sessions` add a `node_id` to each record (samples CSV gains a
leading `node_id` column). Single-device output is unchanged; never sum or
double-count across devices.

**多设备（少见）：**当查询窗口跨越多于 1 台设备时，信封会加上 `"multi_node": true`——`query metrics` 返回按设备划分的 `node_groups`；`query samples`/`sessions` 给每条记录加上 `node_id`（samples 的 CSV 会多出一个前置 `node_id` 列）。单设备输出不变；绝不在设备间求和或重复计数。

### query metrics — all-day metric rollups / query metrics —— 全天指标汇总

```bash
health-cli query metrics --provider healthkit \
  --start-date <YYYY-MM-DD> [--end-date <YYYY-MM-DD>] \
  [--interval hourly|daily|weekly] [--fields a,b,c] [--timeout-secs N]
```

`daily-metrics` is the only domain so there is **no `--category`**. One row per
`--interval` bucket (default `daily`). `--start-date` required; `--end-date`
defaults to today. Output is an envelope `{ "coverage": {...}, "records": [...] }`
(see Coverage below).

`daily-metrics` 是唯一的领域，因此**没有 `--category`**。每个 `--interval` 桶一行（默认 `daily`）。`--start-date` 必填；`--end-date` 默认为今天。输出是信封 `{ "coverage": {...}, "records": [...] }`（见下文 Coverage）。

`--list-fields` reads the metric field names from the synced data →  
`{ ok, provider, category, observed_count, fields: [{ name, observed, count? }] }`.  
`observed: true` (with a row `count`) means the name is present in this user's
data — those are exactly the names `--fields` matches. `observed: false` means
the build knows the field but nothing has synced yet. If the field set can't be
read the response carries `degraded: true` and every `observed` is `null`.

`--list-fields` 从已同步数据中读取指标字段名 →  
`{ ok, provider, category, observed_count, fields: [{ name, observed, count? }] }`。  
`observed: true`（带行 `count`）表示该名称存在于该用户的数据中——这些正是 `--fields` 能匹配的名称。`observed: false` 表示构建版本认识该字段但尚无数据同步。若字段集无法读取，响应带 `degraded: true` 且所有 `observed` 为 `null`。

### query sessions — discrete sessions / query sessions —— 离散会话

```bash
health-cli query sessions --provider healthkit --category sleep|workout \
  --start-date <YYYY-MM-DD> [--end-date <YYYY-MM-DD>] [--fields a,b,c]
```

- `sleep` — one row per sleep session.
  `sleep` —— 每个睡眠会话一行。
- `workout` — one row per workout/activity.
  `workout` —— 每次锻炼/活动一行。

Sessions are never bucketed (no `--interval`). Discovery: `--list-categories` →
`{ ok, provider, categories }`; `--list-fields` (needs `--category`) →
`{ ok, provider, category, fields }` (e.g. workout has `is_indoor`).

会话永不分桶（没有 `--interval`）。发现：`--list-categories` → `{ ok, provider, categories }`；`--list-fields`（需要 `--category`）→ `{ ok, provider, category, fields }`（例如 workout 有 `is_indoor`）。

**Output:** normalized snake_case records; the same concept uses the same field
name (e.g. workout `average_speed_mps`, `elevation_gain_meters`,
`hr_average_bpm`). Session records carry a provider-prefixed `id`
(`healthkit_…`) + `start_datetime`/`end_datetime`/`timezone`; metric buckets
carry `date`/`hour`/`week_start` + `record_count`.

**输出：**规范化的 snake_case 记录；同一概念使用同一字段名（如 workout 的 `average_speed_mps`、`elevation_gain_meters`、`hr_average_bpm`）。会话记录带提供方前缀的 `id`（`healthkit_…`）+ `start_datetime`/`end_datetime`/`timezone`；指标桶带 `date`/`hour`/`week_start` + `record_count`。

**Coverage:** queries return an envelope `{ "coverage": {...}, "records": [...] }`.
`coverage.complete` is `false` when days in the window aren't synced from the
device; `coverage.warning` then holds the exact `backfill_data_source` command
to run. **Surface that warning to the user / act on it** — results are partial
until the backfill completes.

**覆盖率：**查询返回信封 `{ "coverage": {...}, "records": [...] }`。当窗口内有日期尚未从设备同步时，`coverage.complete` 为 `false`；此时 `coverage.warning` 保存应运行的确切 `backfill_data_source` 命令。**把该警告呈现给用户 / 据此行动**——回填完成前结果都是部分的。

### query samples — raw, unbucketed data points / query samples —— 原始、未分桶的数据点

```bash
health-cli query samples --provider healthkit --start-date <YYYY-MM-DD> \
  [--end-date <YYYY-MM-DD>] [--start-time <HH:MM[:SS]>] [--end-time <HH:MM[:SS]>] \
  [--fields <type1,...>] [--limit <n>] [--format stdout|csv] [--output <path>]
health-cli query samples --provider healthkit --list-fields   # discover sample types
```

Individual samples — finer than the `query metrics` rollups (intraday HR/steps,
GPS, sleep-stage timelines; `query sessions` gives stage totals only). `--fields`
selects sample **types** (not output columns) — run `--list-fields` to see the
types this user has synced. GPS is `location` (raw lat/lon — don't infer place
names from it). For dense reads use `--output <path>`: it writes CSV into the
agent filesystem for a later script step and prints only a summary, keeping
thousands of rows out of context.

单个样本——比 `query metrics` 的汇总更细（日内心率/步数、GPS、睡眠阶段时间线；`query sessions` 只给阶段总计）。`--fields` 选择的是样本**类型**（不是输出列）——运行 `--list-fields` 可查看该用户已同步的类型。GPS 是 `location`（原始经纬度——不要据此推断地名）。密集读取用 `--output <path>`：它把 CSV 写入代理文件系统供后续脚本步骤使用，只打印摘要，避免成千上万行进入上下文。

### status — data-sync status / status —— 数据同步状态

```bash
health-cli status --provider healthkit [--timeout-secs N] \
  [--start-date <YYYY-MM-DD>] [--end-date <YYYY-MM-DD>] [--check-missing-entries]
```

Returns `{ categories: [{ name, record_count, earliest_datetime,
latest_datetime }] }`. With `--check-missing-entries` (requires `--start-date`):
a deep per-30-min-interval gap report (`unsynced_dates`, `missing_intervals`, …).

返回 `{ categories: [{ name, record_count, earliest_datetime,
latest_datetime }] }`。带 `--check-missing-entries`（需要 `--start-date`）时：输出按 30 分钟间隔的深度缺口报告（`unsynced_dates`、`missing_intervals` 等）。

### auth connect / auth connect

```bash
health-cli auth connect --provider healthkit [--timeout-secs N]
```

Run this and follow the instruction to connect to apple healthkit data.

运行此命令，并按指引连接到 Apple HealthKit 数据。

### auth disconnect / auth disconnect

```bash
health-cli auth disconnect --provider healthkit [--timeout-secs N]
```

Apple Health cannot be disconnected through this CLI or an in-chat widget.
Tell the user to open **Settings → Connectors → Apple Health** and tap  
**Disconnect**.

Apple Health 无法通过本 CLI 或聊天内组件断开。让用户打开 **Settings → Connectors → Apple Health** 并点击  
**Disconnect**。

### delete — erase stored Apple Health (HealthKit) records / delete —— 擦除已存储的 Apple Health (HealthKit) 记录

```bash
health-cli delete --provider healthkit --all                # every Apple Health record
health-cli delete --provider healthkit --device NODE_ID     # one paired device
health-cli delete --provider healthkit --start-date 2026-03-01 --end-date 2026-03-31
```

Use for any request to delete, remove, erase, wipe, or clear HealthKit data.
Never improvise it by moving files, disconnecting the connector, or writing to
the database — those neither remove records nor leave an audit trail. A delete
never spans providers; to clear both, run it once per provider.

凡删除、移除、擦除、清空或清理 HealthKit 数据的请求都用它。绝不通过移动文件、断开连接器或写数据库来即兴替代——这些既不会移除记录，也不会留下审计痕迹。删除绝不跨提供方；要清空两者，对每个提供方各运行一次。

`--all` takes no other filter; `--device`/`--start-date`/`--end-date` are used
instead of it. All record kinds go at once — no per-category or per-record-ID
delete. Dates are `YYYY-MM-DD` in `JARVIS_USER_TIMEZONE` or epoch seconds; a
record is in range when it starts before the end and ends after the start, so a
session straddling a bound is included. `--start-date` alone deletes from that
day onward, `--end-date` alone up to and including it.

`--all` 不接受其他过滤器；`--device`/`--start-date`/`--end-date` 与其互斥使用。所有记录种类一次清除——没有按类别或按记录 ID 的删除。日期为 `JARVIS_USER_TIMEZONE` 下的 `YYYY-MM-DD` 或 epoch 秒；记录在其开始早于结束日、结束晚于起始日时属于范围内，因此跨越边界的会话也会包含。仅给 `--start-date` 时从当天起删，仅给 `--end-date` 时删到并含当天。

**Destructive and irreversible**, behind a fresh one-time approval showing the
normalized filters. Run it **once per request** — settle provider, device and
dates in conversation first. Denial exits **code 3** with nothing deleted; that
is final, so report it rather than retrying other filters or the other provider.

**破坏性且不可逆**，且须经过一次展示规范化过滤器的全新一次性审批。每个请求只运行**一次**——先在对话中敲定提供方、设备和日期。拒绝时以**退出码 3** 结束且未删除任何内容；这是终态，如实报告即可，不要用其他过滤器或另一个提供方重试。

**Zero is a success**, not a miss — nothing matched, so say so and stop rather
than widening dates or re-running with `--all`. Report `deleted.records`, not
`total_rows`, which also counts child value rows. Apple Health is device-synced,
so records still on the device can return on a later sync.

**零条也是成功**，不是遗漏——没有匹配到任何内容，如实说明并停止，而不是放宽日期或用 `--all` 重跑。报告 `deleted.records`，而不是 `total_rows`，后者还计入子值行。Apple Health 由设备同步，仍留在设备上的记录可能在后续同步中再次出现。

## Auth / 认证

Device-synced; no login. Data availability depends on the on-device Health
permission + sync. Use `status` to see synced ranges and the
`backfill_data_source` device action (surfaced in query `coverage.warning`) to
pull a range.

设备同步；无需登录。数据可用性取决于设备上的健康权限 + 同步。用 `status` 查看已同步范围，并用 `backfill_data_source` 设备动作（在查询的 `coverage.warning` 中呈现）拉取某个范围。

## Operating Rules / 操作规则

1. Pick the command by data shape: `query metrics` (all-day rollups),
   `query samples` (raw points — only when individual samples matter; prefer
   `metrics` for trends), or `query sessions --category sleep|workout` (discrete
   events).
   按数据形态选择命令：`query metrics`（全天汇总）、`query samples`（原始点——仅当单个样本确实重要时使用；趋势优先用 `metrics`），或 `query sessions --category sleep|workout`（离散事件）。
2. Always pass `--provider healthkit` and `--start-date`; `--end-date`
   defaults to today. Use current-date context for relative asks ("today",
   "last week"); use sample time bounds for narrow intraday windows.
   始终传 `--provider healthkit` 和 `--start-date`；`--end-date` 默认为今天。相对表述（"today"、"last week"）用当前日期上下文解释；狭窄的日内窗口用样本时间边界。
3. **Coverage:** if a `query metrics`/`sessions` returns `coverage.complete:
   false`, tell the user the data is partial and relay/act on `coverage.warning`
   (run the `backfill_data_source` action, then re-query).
   **覆盖率：**若 `query metrics`/`sessions` 返回 `coverage.complete: false`，告知用户数据不完整，并传达/执行 `coverage.warning`（运行 `backfill_data_source` 动作，然后重新查询）。
4. `query metrics` defaults to `--interval daily`; sleep spans midnight (include
   both evening start and morning end dates).
   `query metrics` 默认 `--interval daily`；睡眠跨越午夜（起止日期都要包含当晚开始日与次日早晨结束日）。
5. On empty results, the range likely has no data; widen it, or check `status` /
   coverage and backfill.
   结果为空时，该范围可能没有数据；扩大范围，或检查 `status` / 覆盖率并回填。
