<!-- BILINGUAL-EN-ZH -->
# Withings CLI Commands / Withings CLI 命令

Use the installed `withings` CLI from `PATH`. All commands return JSON.

使用 `PATH` 上已安装的 `withings` CLI。所有命令均返回 JSON。

---

## Recommended path — healthkit-shape surface / 推荐路径——healthkit 形态的接口面

These three subcommands cover most agent needs with normalized snake_case fields, units in the name, and category-aware aggregation. Prefer these for new code.

这三个子命令以规范化的 snake_case 字段、名称内含单位以及感知类别的聚合，覆盖了代理的大多数需求。新代码优先使用它们。

### `withings status` / `withings status`

Returns connection state for the active auth backend.

返回当前认证后端的连接状态。

```json
{
  "ok": true,
  "status": "connected",
  "connect_url": null,
  "disconnect_url": "https://..."
}
```

When not connected:

未连接时：

```json
{"ok": true, "status": "not_connected", "connect_url": "https://...", "reason": "withings is not connected"}
```

### `withings list-fields --category CATEGORY` / `withings list-fields --category CATEGORY`

Returns the field list for a category. `CATEGORY` ∈ `daily-metrics`, `sleep`, `workout`.

返回某个类别的字段列表。`CATEGORY` ∈ `daily-metrics`、`sleep`、`workout`。

### `withings query --category CATEGORY --start-date YYYY-MM-DD [...]` / `withings query --category CATEGORY --start-date YYYY-MM-DD [...]`

Flags:

标志：

| Flag | Required | Description |
|---|---|---|
| `--category` | yes | `daily-metrics`, `sleep`, or `workout` |
| `--start-date` | yes | `YYYY-MM-DD`, inclusive |
| `--end-date` | no | `YYYY-MM-DD`, inclusive, defaults to today |
| `--interval` | no | `hourly`, `daily`, `weekly` — daily-metrics only |
| `--fields` | no | Comma-separated subset of fields |

| 标志 | 是否必填 | 说明 |
|---|---|---|
| `--category` | 是 | `daily-metrics`、`sleep` 或 `workout` |
| `--start-date` | 是 | `YYYY-MM-DD`，含端点 |
| `--end-date` | 否 | `YYYY-MM-DD`，含端点，默认为今天 |
| `--interval` | 否 | `hourly`、`daily`、`weekly`——仅 daily-metrics 支持 |
| `--fields` | 否 | 逗号分隔的字段子集 |

`query` returns a JSON **array** of records (not wrapped in `{ok, status, body}`),
with one exception: while the user's "Read measurements and heart data"
permission is deny or ask, `query --category daily-metrics` returns
`{"withheld": {...}, "records": [...]}` (rows without body metrics). Sleep and
workout always return the plain array.

`query` 返回记录的 JSON **数组**（不包裹在 `{ok, status, body}` 中），只有一个例外：当用户的"Read measurements and heart data"权限为 deny 或 ask 时，`query --category daily-metrics` 返回 `{"withheld": {...}, "records": [...]}`（不含身体指标的行）。sleep 和 workout 始终返回普通数组。

### Output contract (new surface) / 输出契约（新接口面）

- Field names are snake_case with units in the name (no `measuregrps`, no integer meas-type IDs).
  字段名为 snake_case 且单位含在名称中（没有 `measuregrps`，也没有整数 meas-type ID）。
- Datetimes are local `YYYY-MM-DD HH:MM:SS` strings — NOT Unix epochs.
  日期时间为本地 `YYYY-MM-DD HH:MM:SS` 字符串——不是 Unix 时间戳。
- Sessions (`sleep`, `workout`) include `id` prefixed `withings_<id>`.
  会话（`sleep`、`workout`）包含前缀为 `withings_<id>` 的 `id`。
- Aggregated `daily-metrics` buckets include `date` / `hour` / `week_start` + `record_count` (no `id`).
  聚合的 `daily-metrics` 桶包含 `date` / `hour` / `week_start` + `record_count`（没有 `id`）。
- `--fields foo` returns records with only `foo` + always-kept fields (`start_datetime`, `end_datetime`, `timezone`, `date`, `hour`, `week_start`, `record_count`, `id`).
  `--fields foo` 返回的记录只含 `foo` 加上始终保留的字段（`start_datetime`、`end_datetime`、`timezone`、`date`、`hour`、`week_start`、`record_count`、`id`）。

### Daily-metrics aggregation rules / daily-metrics 聚合规则

| Aggregation | Fields |
|---|---|
| **Sum** | `step_count`, `active_energy_burned_kcal`, `total_calories_kcal`, `distance_walking_running_meters`, `elevation_climbed_meters`, `soft_activity_duration_sec`, `moderate_activity_duration_sec`, `intense_activity_duration_sec` |
| **Avg** | `hr_average_bpm` |
| **Max** | `hr_max_bpm` |
| **Last** (most recent reading in bucket wins) | `body_mass_kg`, `body_fat_percentage`, `fat_free_mass_kg`, `fat_mass_kg`, `muscle_mass_kg`, `bone_mass_kg`, `hydration_kg`, `blood_pressure_systolic_mmhg`, `blood_pressure_diastolic_mmhg`, `vo2_max`, `spo2_percentage`, `body_temperature_celsius`, `skin_temperature_celsius`, `pulse_wave_velocity_meters_per_sec`, `basal_metabolic_rate_kcal`, `metabolic_age_years`, `visceral_fat`, `height_meters` |

| 聚合方式 | 字段 |
|---|---|
| **Sum（求和）** | `step_count`, `active_energy_burned_kcal`, `total_calories_kcal`, `distance_walking_running_meters`, `elevation_climbed_meters`, `soft_activity_duration_sec`, `moderate_activity_duration_sec`, `intense_activity_duration_sec` |
| **Avg（平均）** | `hr_average_bpm` |
| **Max（最大）** | `hr_max_bpm` |
| **Last（取桶内最近一次读数）** | `body_mass_kg`, `body_fat_percentage`, `fat_free_mass_kg`, `fat_mass_kg`, `muscle_mass_kg`, `bone_mass_kg`, `hydration_kg`, `blood_pressure_systolic_mmhg`, `blood_pressure_diastolic_mmhg`, `vo2_max`, `spo2_percentage`, `body_temperature_celsius`, `skin_temperature_celsius`, `pulse_wave_velocity_meters_per_sec`, `basal_metabolic_rate_kcal`, `metabolic_age_years`, `visceral_fat`, `height_meters` |

### Withings → Muse field name mapping (new surface) / Withings → Muse 字段名映射（新接口面）

Translation table used internally by `query`. Agents only need to know the right-hand column (snake_case names returned in `query` output). Unmapped Withings meas-type IDs are dropped from `query` output — use the legacy `measures` command for those (see escape hatch below).

`query` 内部使用的翻译表。代理只需知道右列（`query` 输出返回的 snake_case 名称）。未映射的 Withings meas-type ID 会从 `query` 输出中丢弃——此类数据使用旧版 `measures` 命令（见下方逃生通道）。

| Withings meas-type ID | Muse field |
|---|---|
| 1 | `body_mass_kg` |
| 4 | `height_meters` |
| 5 | `fat_free_mass_kg` |
| 6 | `body_fat_percentage` |
| 8 | `fat_mass_kg` |
| 9 | `blood_pressure_diastolic_mmhg` |
| 10 | `blood_pressure_systolic_mmhg` |
| 11 | `hr_average_bpm` |
| 12 | `temperature_celsius` |
| 35 | `co2_ppm` |
| 54 | `spo2_percentage` |
| 71 | `body_temperature_celsius` |
| 73 | `skin_temperature_celsius` |
| 76 | `muscle_mass_kg` |
| 77 | `hydration_kg` |
| 88 | `bone_mass_kg` |
| 91 | `pulse_wave_velocity_meters_per_sec` |
| 123 | `vo2_max` |
| 135 | `qrs_interval_ms` |
| 136 | `pr_interval_ms` |
| 137 | `qt_interval_ms` |
| 138 | `corrected_qt_interval_ms` |
| 139 | `atrial_fibrillation_ppg` |
| 155 | `vascular_age_years` |
| 167 | `nerve_health_score_conductance_feet` |
| 168 | `extracellular_water_kg` |
| 169 | `intracellular_water_kg` |
| 170 | `visceral_fat` |
| 174 | `fat_free_mass_segmental_kg` |
| 175 | `muscle_mass_segmental_kg` |
| 196 | `electrodermal_activity_feet` |
| 226 | `basal_metabolic_rate_kcal` |
| 227 | `metabolic_age_years` |

| Withings meas-type ID | Muse 字段 |
|---|---|
| 1 | `body_mass_kg` |
| 4 | `height_meters` |
| 5 | `fat_free_mass_kg` |
| 6 | `body_fat_percentage` |
| 8 | `fat_mass_kg` |
| 9 | `blood_pressure_diastolic_mmhg` |
| 10 | `blood_pressure_systolic_mmhg` |
| 11 | `hr_average_bpm` |
| 12 | `temperature_celsius` |
| 35 | `co2_ppm` |
| 54 | `spo2_percentage` |
| 71 | `body_temperature_celsius` |
| 73 | `skin_temperature_celsius` |
| 76 | `muscle_mass_kg` |
| 77 | `hydration_kg` |
| 88 | `bone_mass_kg` |
| 91 | `pulse_wave_velocity_meters_per_sec` |
| 123 | `vo2_max` |
| 135 | `qrs_interval_ms` |
| 136 | `pr_interval_ms` |
| 137 | `qt_interval_ms` |
| 138 | `corrected_qt_interval_ms` |
| 139 | `atrial_fibrillation_ppg` |
| 155 | `vascular_age_years` |
| 167 | `nerve_health_score_conductance_feet` |
| 168 | `extracellular_water_kg` |
| 169 | `intracellular_water_kg` |
| 170 | `visceral_fat` |
| 174 | `fat_free_mass_segmental_kg` |
| 175 | `muscle_mass_segmental_kg` |
| 196 | `electrodermal_activity_feet` |
| 226 | `basal_metabolic_rate_kcal` |
| 227 | `metabolic_age_years` |

Workout-type translation (`workout_type` field in `query --category workout` output):  
`1=Walk, 2=Run, 3=Hiking, 4=Skating, 5=BMX, 6=Cycling, 7=Swimming, 8=Surfing, 9=Kitesurfing, 10=Windsurfing, 12=Tennis, 13=Table tennis, 14=Squash, 15=Badminton, 16=Weightlifting, 17=Calisthenics, 18=Elliptical, 19=Pilates, 20=Basketball, 21=Soccer, 22=Football, 27=Golf, 28=Yoga, 30=Boxing, 34=Skiing, 35=Snowboarding, 36=Rowing, 42=Climbing, 45=Indoor walk, 46=Indoor running, 47=Indoor cycling, 187=Stretching, 188=Cross training, 191=Fitness, 195=Other`, etc. (full list in source: `withings/src/schema.rs::WITHINGS_WORKOUT_TYPE_TO_NAME`). Unknown integers are dropped from `workout_type`.

锻炼类型翻译（`query --category workout` 输出中的 `workout_type` 字段）：  
`1=Walk, 2=Run, 3=Hiking, 4=Skating, 5=BMX, 6=Cycling, 7=Swimming, 8=Surfing, 9=Kitesurfing, 10=Windsurfing, 12=Tennis, 13=Table tennis, 14=Squash, 15=Badminton, 16=Weightlifting, 17=Calisthenics, 18=Elliptical, 19=Pilates, 20=Basketball, 21=Soccer, 22=Football, 27=Golf, 28=Yoga, 30=Boxing, 34=Skiing, 35=Snowboarding, 36=Rowing, 42=Climbing, 45=Indoor walk, 46=Indoor running, 47=Indoor cycling, 187=Stretching, 188=Cross training, 191=Fitness, 195=Other` 等（完整列表见源码：`withings/src/schema.rs::WITHINGS_WORKOUT_TYPE_TO_NAME`）。未知整数会从 `workout_type` 中丢弃。

---

## Legacy / Advanced commands / 旧版 / 高级命令

These passthroughs return raw Withings response shapes (`{ok, status, body: <Withings native>}`). Use them when:
- You need a meas-type that isn't in the translation table above (escape hatch).
- You need every individual reading within a day instead of aggregated last-of-day.
- You need high-frequency sleep / heart-recording sensor data.

这些透传命令返回原始的 Withings 响应形态（`{ok, status, body: <Withings native>}`）。在以下情况使用：
- 需要上表未包含的 meas-type（逃生通道）。
- 需要一天内的每一次单独读数，而不是聚合后的当日最后一次。
- 需要高频睡眠 / 心脏记录传感器数据。

### Auth / 认证

- `withings status` — Returns connector status with `connect_url` / `disconnect_url` when available.
  `withings status`——返回连接器状态，可用时附 `connect_url` / `disconnect_url`。
- `withings authorize-url` — Returns the authorization URL for the active backend. Parse `authorize_url`. Prefer `withings status` (returns the same URL as `connect_url`).
  `withings authorize-url`——返回当前后端的授权 URL。解析 `authorize_url`。优先使用 `withings status`（以 `connect_url` 返回同一 URL）。
- `withings disconnect` — Disconnects from the active backend.
  `withings disconnect`——从当前后端断开连接。

### Data reads (raw passthrough) / 数据读取（原始透传）

- `withings measures [--category 1] [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--meas-types 1,4,11] [--offset <n>] [--last-update <ts>]`
- `withings activity [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--offset <n>] [--last-update <ts>]`
- `withings sleep-summary [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--offset <n>] [--last-update <ts>]`
- `withings sleep [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--meas-types <csv>]`
- `withings workouts [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--offset <n>] [--last-update <ts>]`
- `withings intraday [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD]`
- `withings heart-list [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--offset <n>]`
- `withings heart-get --signal-id <id>`
- `withings devices`

### Measurement Type IDs (`--meas-types` for legacy `measures` command) / 测量类型 ID（旧版 `measures` 命令的 `--meas-types`）

Full reference, including IDs not exposed by the new `query` surface (use this when the agent needs raw access):

完整参考，包括新 `query` 接口面未暴露的 ID（代理需要原始访问时使用此表）：

| ID | Metric |
|----|--------|
| 1 | Weight (kg) |
| 4 | Height (m) |
| 5 | Fat Free Mass (kg) |
| 6 | Fat Ratio (%) |
| 8 | Fat Mass Weight (kg) |
| 9 | Diastolic BP (mmHg) |
| 10 | Systolic BP (mmHg) |
| 11 | Heart Pulse (bpm) |
| 12 | Temperature (C) |
| 54 | SpO2 (%) |
| 71 | Body Temperature (C) |
| 73 | Skin Temperature (C) |
| 76 | Muscle Mass (kg) |
| 77 | Hydration (kg) |
| 88 | Bone Mass (kg) |
| 91 | Pulse Wave Velocity (m/s) |
| 123 | VO2 Max |
| 130 | AFib result (ECG) |
| 135 | QRS interval (ms) |
| 136 | PR interval (ms) |
| 137 | QT interval (ms) |
| 138 | Corrected QT interval (ms) |
| 139 | AFib result (PPG) |
| 155 | Vascular Age |
| 167 | Nerve Health Score |
| 168 | Extracellular Water (kg) |
| 169 | Intracellular Water (kg) |
| 170 | Visceral Fat |
| 173 | Fat Free Mass (segments) |
| 174 | Fat Mass (segments) |
| 175 | Muscle Mass (segments) |
| 196 | Electrodermal Activity |
| 226 | Basal Metabolic Rate |
| 227 | Metabolic Age |
| 229 | Electrochemical Skin Conductance |

| ID | 指标 |
|----|--------|
| 1 | 体重 (kg) |
| 4 | 身高 (m) |
| 5 | 去脂体重 (kg) |
| 6 | 体脂率 (%) |
| 8 | 脂肪重量 (kg) |
| 9 | 舒张压 (mmHg) |
| 10 | 收缩压 (mmHg) |
| 11 | 心率 (bpm) |
| 12 | 温度 (C) |
| 54 | 血氧 SpO2 (%) |
| 71 | 体温 (C) |
| 73 | 皮肤温度 (C) |
| 76 | 肌肉量 (kg) |
| 77 | 体水分 (kg) |
| 88 | 骨量 (kg) |
| 91 | 脉搏波传导速度 (m/s) |
| 123 | 最大摄氧量 VO2 Max |
| 130 | 房颤结果 (ECG) |
| 135 | QRS 间期 (ms) |
| 136 | PR 间期 (ms) |
| 137 | QT 间期 (ms) |
| 138 | 校正 QT 间期 (ms) |
| 139 | 房颤结果 (PPG) |
| 155 | 血管年龄 |
| 167 | 神经健康评分 |
| 168 | 细胞外水分 (kg) |
| 169 | 细胞内水分 (kg) |
| 170 | 内脏脂肪 |
| 173 | 去脂体重（分段） |
| 174 | 脂肪重量（分段） |
| 175 | 肌肉量（分段） |
| 196 | 皮肤电活动 |
| 226 | 基础代谢率 |
| 227 | 代谢年龄 |
| 229 | 皮肤电导 |

### Legacy output contract / 旧版输出契约

- Read commands return top-level `ok`, `status`, and `body`.
  读取命令返回顶层的 `ok`、`status` 和 `body`。
- `measures` uses `--category 1` for real measures and `--category 2` for user objectives.
  `measures` 用 `--category 1` 表示真实测量值，`--category 2` 表示用户目标。
- Date parameters accept `YYYY-MM-DD` format. The CLI converts dates to Unix timestamps where the API requires it.
  日期参数接受 `YYYY-MM-DD` 格式。API 要求时，CLI 会把日期转换为 Unix 时间戳。
- Value scaling: each measure in `body.measuregrps[].measures[]` is encoded as `{value, unit, type}` where the real value = `value * 10^unit`.
  数值缩放：`body.measuregrps[].measures[]` 中的每个测量值编码为 `{value, unit, type}`，实际值 = `value * 10^unit`。

### Pagination / 分页

- Commands that support `--offset` return `body.more` (boolean) and `body.offset` when more pages are available. Pass `--offset <value>` to fetch the next page.
  支持 `--offset` 的命令在有更多页时返回 `body.more`（布尔值）和 `body.offset`。传入 `--offset <value>` 获取下一页。
- The new `query` subcommand handles pagination internally (up to 20 pages per category).
  新的 `query` 子命令在内部处理分页（每个类别最多 20 页）。
