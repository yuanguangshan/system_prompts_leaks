---
name: "withings"
description: "Use when linking Withings or reading Withings body measurements, activity, sleep, workout, heart, and intraday data."
icon: "withings"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Withings / Withings

Query Withings body measurements, activity, sleep, and workout data via the bundled `withings` CLI.

通过内置的 `withings` 命令行工具查询 Withings 的身体测量、活动、睡眠和锻炼数据。

## When to Use / 何时使用

Activate when the user asks about their Withings data:
- Body metrics (weight, BMI, body fat, blood pressure, heart rate)
  身体指标（体重、BMI、体脂、血压、心率）
- Daily activity (steps, calories, distance, active duration)
  日常活动（步数、卡路里、距离、活动时长）
- Sleep sessions (score, stages, duration, breathing rate)
  睡眠时段（评分、分期、时长、呼吸频率）
- Workout sessions (run/walk/cycle/etc.)
  锻炼时段（跑步/步行/骑行等）

## Tooling / 工具

Three subcommands cover most needs:

三个子命令覆盖大多数需求：

| Subcommand | Purpose |
|---|---|
| `withings status` | Connection state (`{ok, status, connect_url?, disconnect_url?, reason?}`). |
| `withings list-fields --category <CAT>` | Field list for a category (snake_case, units in the name). |
| `withings query --category <CAT> --start-date <YYYY-MM-DD> [--end-date --interval --fields]` | Typed snake_case records. Returns a JSON array. |

| 子命令 | 用途 |
|---|---|
| `withings status` | 连接状态（`{ok, status, connect_url?, disconnect_url?, reason?}`）。 |
| `withings list-fields --category <CAT>` | 某类别的字段列表（snake_case，单位含在名称中）。 |
| `withings query --category <CAT> --start-date <YYYY-MM-DD> [--end-date --interval --fields]` | 类型化的 snake_case 记录。返回 JSON 数组。 |

### Categories / 类别

- `daily-metrics` — Daily rollup of Activity (steps/calories/distance/HR) + Measures (weight/BP/body composition). Supports `--interval hourly|daily|weekly` (defaults to `daily`). Aggregation rules: **sum** for counters (`step_count`, `active_energy_burned_kcal`, `distance_walking_running_meters`), **avg** for `hr_average_bpm`, **max** for `hr_max_bpm`, **last-of-bucket** for body measurements (weight/BP/body fat etc.).
  `daily-metrics` — 活动（步数/卡路里/距离/心率）+ 测量（体重/血压/身体成分）的每日汇总。支持 `--interval hourly|daily|weekly`（默认 `daily`）。聚合规则：计数器取**和**（`step_count`、`active_energy_burned_kcal`、`distance_walking_running_meters`），`hr_average_bpm` 取**平均**，`hr_max_bpm` 取**最大**，身体测量（体重/血压/体脂等）取**桶内最后一个值**。
- `sleep` — One row per sleep session (score, stages, duration, HR, breathing).
  `sleep` — 每个睡眠时段一行（评分、分期、时长、心率、呼吸）。
- `workout` — One row per workout session (translated `workout_type` name, duration, distance, calories, HR).
  `workout` — 每个锻炼时段一行（翻译后的 `workout_type` 名称、时长、距离、卡路里、心率）。

Use `list-fields` to discover available fields per category.

用 `list-fields` 发现各类别可用的字段。

### Output / 输出

JSON to stdout. Datetimes are local `YYYY-MM-DD HH:MM:SS` per record's timezone. Session rows include `id` (prefixed `withings_<id>`), `start_datetime`, `end_datetime`, `timezone`. `daily-metrics` buckets include `date` / `hour` / `week_start` + `record_count` instead of `id`.

JSON 输出到 stdout。日期时间按每条记录所在时区以本地 `YYYY-MM-DD HH:MM:SS` 表示。时段行包含 `id`（前缀 `withings_<id>`）、`start_datetime`、`end_datetime`、`timezone`。`daily-metrics` 桶包含 `date` / `hour` / `week_start` + `record_count`，而没有 `id`。

### Examples / 示例

```bash
# Recent workouts
withings query --category workout --start-date 2026-05-01

# Weight trend (last reading per day)
withings query --category daily-metrics --start-date 2026-04-01 --fields body_mass_kg

# Weekly step totals
withings query --category daily-metrics --start-date 2026-04-01 --interval weekly --fields step_count
```

## Auth / 认证

Before any Withings API call:
1. Run `withings status`.
   运行 `withings status`。
2. If `status` is `not_connected`, share `[Connect Withings](<connect_url>)` exactly — never paste the raw URL.
   如果 `status` 为 `not_connected`，原样分享 `[Connect Withings](<connect_url>)`——绝不要粘贴原始 URL。
3. After the user completes the callback, re-run `withings status` and proceed only when `connected`.
   用户完成回调后，重新运行 `withings status`，仅在 `connected` 时继续。

For disconnect: run `withings disconnect` and share `[Disconnect Withings](<disconnect_url>)` if present.

断开连接：运行 `withings disconnect`，如存在则分享 `[Disconnect Withings](<disconnect_url>)`。

Never print tokens or credentials.

绝不要打印令牌或凭据。

## Operating Rules / 操作规则

1. Treat linking as one-time onboarding — don't re-prompt for auth unless calls keep failing.
   把关联当作一次性引导——除非调用持续失败，否则不要反复提示认证。
2. Prefer `query --category <CAT>` over the legacy commands. Field names are snake_case with units in the name (`body_mass_kg`, `step_count`, `hr_average_bpm`, `sleep_total_duration_sec`) — never expose Withings's numeric meas-type IDs to the user.
   优先用 `query --category <CAT>` 而不是旧版命令。字段名是 snake_case 且单位含在名称中（`body_mass_kg`、`step_count`、`hr_average_bpm`、`sleep_total_duration_sec`）——绝不要向用户暴露 Withings 的数字 meas-type ID。
3. **If unsure of a field name, run `withings list-fields --category <CAT>` BEFORE composing `query --fields`.** Withings-native names (`calories`, `distance`, `hr_average`, `weight`) are NOT valid — they're translated to snake_case with units (`energy_burned_kcal`, `distance_meters`, `hr_average_bpm`, `body_mass_kg`). An empty filter result usually means the field name was wrong, not that the data is missing.
   **如果不确定字段名，先运行 `withings list-fields --category <CAT>`，再组装 `query --fields`。** Withings 原生名称（`calories`、`distance`、`hr_average`、`weight`）是无效的——它们会被翻译为带单位的 snake_case（`energy_burned_kcal`、`distance_meters`、`hr_average_bpm`、`body_mass_kg`）。过滤结果为空通常意味着字段名写错了，而不是数据缺失。
4. Default to the last 7 days when the user asks for recent trends without specifying dates.
   当用户询问近期趋势而未指定日期时，默认取最近 7 天。
5. For broad trends, use `--category daily-metrics --interval weekly` instead of pulling every reading. The `--interval` flag already aggregates per the rules above — do NOT apply additional aggregation (sum/avg) to the bucketed output client-side.
   对于大范围趋势，用 `--category daily-metrics --interval weekly`，而不是拉取每一条读数。`--interval` 标志已按上述规则聚合——不要在客户端对分桶输出再做额外聚合（求和/平均）。
6. If `query` returns `[]`, say so clearly — never invent values. If the user is asking for a metric not surfaced by `list-fields`, drop to the legacy escape hatch (`measures --meas-types <ID>` or a more specific subcommand like `heart-list`; see [references/commands.md](references/commands.md) for the meas-type ID table).
   如果 `query` 返回 `[]`，明确说出来——绝不要编造数值。如果用户要的指标未出现在 `list-fields` 中，退回到旧版逃生通道（`measures --meas-types <ID>` 或更具体的子命令如 `heart-list`；meas-type ID 对照表见 [references/commands.md](references/commands.md)）。

## Legacy / Advanced Commands / 旧版/高级命令

Pre-`query` per-endpoint passthroughs that return raw `{ok, status, body: <Withings native>}` envelopes. Use only as an **escape hatch** for data not exposed by `query` — e.g. a meas-type missing from the translation table (`withings measures --meas-types 130` for AFib ECG result), high-frequency sleep sensor data (`withings sleep`), or ECG signals (`withings heart-list` / `heart-get`).

早于 `query` 的逐端点透传命令，返回原始的 `{ok, status, body: <Withings native>}` 信封。只作为 `query` 未暴露数据的**逃生通道**使用——例如翻译表中缺失的 meas-type（`withings measures --meas-types 130` 获取房颤 ECG 结果）、高频睡眠传感器数据（`withings sleep`），或 ECG 信号（`withings heart-list` / `heart-get`）。

Quick reference for the other passthroughs:
- `withings activity` — raw daily Activity rows.
  `withings activity` — 原始的每日活动行。
- `withings sleep-summary` — raw nightly sleep summaries with Withings-native field names.
  `withings sleep-summary` — 使用 Withings 原生字段名的原始夜间睡眠摘要。
- `withings intraday` — minute-resolution activity sensor data.
  `withings intraday` — 分钟级分辨率的活动传感器数据。
- `withings devices` — list paired Withings devices.
  `withings devices` — 列出已配对的 Withings 设备。

Full command matrix, meas-type IDs, and Withings → Muse field mapping: [references/commands.md](references/commands.md).

完整命令矩阵、meas-type ID 以及 Withings → Muse 字段映射：[references/commands.md](references/commands.md)。
- Permission-withheld data: while the user's "Read measurements and heart
  data" permission is deny or ask, `query --category daily-metrics` wraps its
  rows in `{"withheld": {...}, "records": [...]}` with body metrics (weight,
  blood pressure, SpO2, heart measurements) removed, and a request for only
  those fields fails with `data_class_excluded`. Key off the `withheld`
  marker: when present, the permission is restricting those fields — say so
  instead of "no readings". When the user actually needs a withheld metric
  and the permission requires approval (`withheld.reason` is
  `requires_approval`, or a body-metrics-only query failed with
  `data_class_excluded` saying it requires approval), run
  `withings measures --category 1 --start-date <d> --end-date <d>`, which is
  governed by that permission and shows the user the approval prompt. Sleep
  and workout results are never wrapped; a response without the marker is an
  ordinary result.
  权限被扣留的数据：当用户的"读取测量和心脏数据"权限为 deny 或 ask 时，`query --category daily-metrics` 会把其行包装在 `{"withheld": {...}, "records": [...]}` 中，并移除身体指标（体重、血压、SpO2、心脏测量），而只请求这些字段的查询会以 `data_class_excluded` 失败。以 `withheld` 标记为准：当它出现时，说明权限正在限制这些字段——要如实说明，而不是说"没有读数"。当用户确实需要某个被扣留的指标且该权限需要审批（`withheld.reason` 为 `requires_approval`，或仅身体指标的查询以 `data_class_excluded` 失败并说明需要审批）时，运行 `withings measures --category 1 --start-date <d> --end-date <d>`，它受该权限管辖并会向用户展示审批提示。睡眠和锻炼结果从不被包装；没有该标记的响应就是普通结果。
  【评论】数据按权限类别在服务端过滤，代理通过 `withheld` 标记区分"无数据"与"被权限扣留"，避免把隐私限制误报为设备没有数据。
