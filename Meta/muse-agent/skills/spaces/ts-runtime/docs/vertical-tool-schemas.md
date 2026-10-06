<!-- BILINGUAL-EN-ZH -->
# `ctx.tool` vertical schemas / `ctx.tool` 垂直领域 schema

Source of truth:

事实来源（source of truth）：

- Contract: `sdk/src/verticals.ts` (`TOOL_*_RESULT_SCHEMA`, option types,
  `SpaceToolClient`), re-exported from `sdk/src/index.ts`.
  契约：`sdk/src/verticals.ts`（`TOOL_*_RESULT_SCHEMA`、选项类型、`SpaceToolClient`），从 `sdk/src/index.ts` 再导出。
- Backend mapping: `worker/src/web_search.ts` (MASE `vertical_data` → typed result).
  后端映射：`worker/src/web_search.ts`（MASE `vertical_data` → 类型化结果）。

## Landing now vs. follow-up / 已落地与后续事项

This documents what is **landed and mapped today**:

本文记录的是**今日已落地并完成映射**的内容：

- **Weather** — enriched current conditions + daily/hourly forecast + alerts.
  **天气** —— 增强的当前天气 + 每日/每小时预报 + 预警。
- **Finance** — latest quote (with session change) + interval-keyed OHLCV history.
  `finance(query)` returns an **array** of matched instruments;
  `finance_ticker(symbol)` returns the **single** resolved instrument (same
  fields). `ctx.tool.finance` is **deprecated** and pulled from the guidance —
  use `finance_ticker` for a single ticker's quote/price/history (by symbol) and
  `web_search` for everything else, including resolving a company name to a
  ticker (search the name, read the symbol, then call `finance_ticker`),
  comparing several instruments, and analysis. The method stays available so
  existing artifacts keep working, but it will be deleted.
  **金融** —— 最新报价（含盘中变动）+ 按区间键控的 OHLCV 历史。`finance(query)` 返回匹配证券的**数组**；`finance_ticker(symbol)` 返回**单个**已解析的证券（字段相同）。`ctx.tool.finance` 已**弃用**并从指引中移除——单个证券代码的报价/价格/历史（按 symbol）请用 `finance_ticker`，其余一切（包括把公司名解析为代码：先搜索名称、读取 symbol，再调用 `finance_ticker`；比较多个证券；以及做分析）都用 `web_search`。该方法仍保持可用以让既有工件继续工作，但它将被删除。
- **Sports** — the flat per-event `items` list, enriched with normalized
  `sport`/`league`/`season` scope, explicit `home`/`away` teams, and parsed
  `player_statistics`/`team_statistics`. A sport-discriminated `games` union
  remains a **follow-up** (see §3).
  **体育** —— 扁平的逐赛事 `items` 列表，增强了规范化的 `sport`/`league`/`season` 范围、显式的 `home`/`away` 队伍，以及解析出的 `player_statistics`/`team_statistics`。按运动区分的 `games` 联合类型仍是**后续工作**（见 §3）。
- **Web search** — `web_search(query)`: a general web search with **no vertical
  filter** — the same plain search the agent's browser_search runs. **Web**
  results only — the exception to principle 1 below: **no `vertical_data`**;
  every field is mapped from plain web results into `results[]` (see §4).
  **Web 搜索** —— `web_search(query)`：不带**垂直领域过滤**的通用 web 搜索——与代理的 browser_search 运行的普通搜索相同。仅返回 **Web** 结果——是下文原则 1 的例外：**没有 `vertical_data`**；每个字段都从普通 web 结果映射到 `results[]`（见 §4）。

Design principles that still hold:

仍然成立的设计原则：

1. **Structure lives in `vertical_data`.** The typed contract below is what we map
   out of MASE's per-result `vertical_data` payload. Display text (`summary`,
   `excerpt`) is display-only — do not parse it; read the structured fields.
   **结构位于 `vertical_data`。** 下面的类型化契约就是我们从 MASE 每条结果的 `vertical_data` 载荷映射出的内容。展示文本（`summary`、`excerpt`）仅用于展示——不要解析它；读取结构化字段。
2. **Every observation is nullable/optional.** MASE may omit a field or surface it
   only in text, so the builders degrade missing data to `null` and emit a
   schema-valid stub on an empty payload rather than throwing.
   **每个观测值都可空/可选。** MASE 可能省略某字段或只在文本中出现，因此构建器把缺失数据降级为 `null`，在载荷为空时发出 schema 合法的存根而不是抛错。
3. **Caps live in the contract.** Heavy lists carry a `.max(...)` ceiling
   (`forecast_hourly` ≤48, finance `history.points` ≤400); the builders slice to
   the cap so a large upstream series never trips `.parse()`.
   **上限写在契约里。** 重量级列表带有 `.max(...)` 上限（`forecast_hourly` ≤48，金融 `history.points` ≤400）；构建器按上限切片，使大的上游序列永远不会让 `.parse()` 失败。

【评论】“全部字段可空 + 空载荷也发合法存根 + 大列表封顶”三者共同构成防御性 schema 设计：在上游字段不可信且随时缺失的前提下，保证下游解析既不抛错也不因超量数据而失败。

### Request knobs not yet plumbed to the backend / 尚未接通后端的请求参数

`ToolSearchOptions.until`, `ToolWeatherOptions.hourly_hours`, and the
`ToolFinanceOptions.interval` selector are not backend request fields today;
the backend returns its full forecast or candle sets. Both runtimes apply the
model-selected variants client-side: `hourly_hours` caps the hourly forecast,
`interval` selects one candle set, and `since`/`until` bound finance history.

`ToolSearchOptions.until`、`ToolWeatherOptions.hourly_hours` 和 `ToolFinanceOptions.interval` 选择器目前都不是后端请求字段；后端返回其完整的预报或 K 线集合。两个运行时都在客户端应用模型选定的变体：`hourly_hours` 限制每小时预报的点数，`interval` 选择一组 K 线，`since`/`until` 界定金融历史的范围。

---

## 1. Weather / 1. 天气

### Request / 请求

```ts
export interface ToolWeatherOptions extends ToolSearchOptions {
  /** Cap the upstream hourly forecast series at this many points, ≤48. */
  readonly hourly_hours?: number;
}
```

### Result (`TOOL_WEATHER_RESULT_SCHEMA`) / 结果（`TOOL_WEATHER_RESULT_SCHEMA`）

`location`, `summary`, `conditions`, `forecast_days`, `forecast_hourly?`,
`alerts?`, `sources`.

即 `location`（位置）、`summary`（摘要）、`conditions`（当前天气）、`forecast_days`（每日预报）、`forecast_hourly?`（每小时预报，可选）、`alerts?`（预警，可选）、`sources`（来源）。

**Supported (landed) fields:**

**已支持（已落地）字段：**

- `conditions`: `temperature`, `unit`, `description`, `feels_like`, `high`, `low`,
  `humidity_percent`, `precipitation_chance` (0–100), `precipitation_amount`
  (formatted string, e.g. `"0.37 in"`), `wind`, `uv_index`, `air_quality_index`,
  `air_quality_description`, `sunrise`, `sunset`.
  `conditions`：`temperature`、`unit`、`description`、`feels_like`、`high`、`low`、`humidity_percent`、`precipitation_chance`（0–100）、`precipitation_amount`（格式化字符串，如 `"0.37 in"`）、`wind`、`uv_index`、`air_quality_index`、`air_quality_description`、`sunrise`、`sunset`。
- `forecast_days[]`: `date`, `summary`, `high`, `low`, `precipitation_chance`,
  `precipitation_amount`, `wind`.
  `forecast_days[]`：`date`、`summary`、`high`、`low`、`precipitation_chance`、`precipitation_amount`、`wind`。
- `forecast_hourly[]` (≤48, present when the feed returns hourly): `time` (ISO),
  `temperature` (null — the feed omits it hourly today), `description`,
  `precipitation_chance`, `precipitation_amount`, `wind`. When supplied,
  `hourly_hours` selects the maximum returned points; omission preserves the
  existing maximum of 48.
  `forecast_hourly[]`（≤48，当数据源返回逐小时数据时存在）：`time`（ISO）、`temperature`（null——数据源目前不提供逐小时温度）、`description`、`precipitation_chance`、`precipitation_amount`、`wind`。提供 `hourly_hours` 时，它决定返回点的最大数量；省略则保持现有的 48 上限。
- `alerts[]`: active alert headlines (mapped from `{ event, severity }` objects).
  `alerts[]`：生效中的预警标题（从 `{ event, severity }` 对象映射而来）。

Upstream `vertical_data` arrives with measures as `{ value, unit }` objects
(`temperature`, `wind_speed`, `feels_like`, `precipitation_*`, daily high/low);
`humidity`, `uv_index`, `air_quality_index` are bare numbers;
`air_quality_description`, `sunrise`, `sunset` are strings.

上游 `vertical_data` 的度量值以 `{ value, unit }` 对象形式到达（`temperature`、`wind_speed`、`feels_like`、`precipitation_*`、每日高低温）；`humidity`、`uv_index`、`air_quality_index` 是裸数字；`air_quality_description`、`sunrise`、`sunset` 是字符串。

> Landed via the KES weather thrift + NLQ + MSL/WWW decoder work  
> (D107777759, D107957780, D107729843, D108000766); live-verified against  
> P2371491675. (The earlier "`feels_like` / precipitation are future KES work"  
> caveat no longer applies — they landed.)

> 通过 KES 天气 thrift + NLQ + MSL/WWW 解码器工作落地  
> （D107777759、D107957780、D107729843、D108000766）；已对照  
> P2371491675 完成线上验证。（先前“`feels_like` / 降水属于 KES 后续工作”的  
> 注意事项不再适用——它们已落地。）

---

## 2. Finance / 2. 金融

### Request / 请求

```ts
export interface ToolFinanceOptions extends ToolSearchOptions {
  /** Selects which candle set maps into `instruments[].history`. Omit for quote only. */
  readonly interval?: "1m" | "30m" | "1d" | "1w" | "1mo";
  // `since`/`until` (inherited) bound the history window — see below.
}
```

### Result (`TOOL_FINANCE_RESULT_SCHEMA`) / 结果（`TOOL_FINANCE_RESULT_SCHEMA`）

Per `instruments[]` entry: `name`, `symbol`, `summary`, `price`, `currency`,
`change`, `change_percent`, `market_status`, `as_of`, `url`, and an opt-in  
`history`.

每个 `instruments[]` 条目包含：`name`（名称）、`symbol`（代码）、`summary`（摘要）、`price`（价格）、`currency`（币种）、`change`（涨跌额）、`change_percent`（涨跌幅）、`market_status`（市场状态）、`as_of`（截至时间）、`url`，以及可选的（opt-in）`history`（历史）。

**Supported (landed) fields:**

**已支持（已落地）字段：**

- `change` ← `entity.attributes.change`; `change_percent` ←
  `entity.attributes.percentChange`; `as_of` ←
  `entity.attributes.lastUpdatedAt` (epoch seconds → ISO).
  `change` ← `entity.attributes.change`；`change_percent` ← `entity.attributes.percentChange`；`as_of` ← `entity.attributes.lastUpdatedAt`（epoch 秒 → ISO）。
- `history` (present only when an `interval` is requested):  
  `{ interval, since, until, points: [{ date, open?, high?, low?, close, volume? }] }`,  
  sorted oldest first, capped at 400. For intraday intervals (`1m` and `30m`)
  each point's `date` is a full ISO timestamp so same-day bars stay distinct;
  `1d`/`1w`/`1mo` use `YYYY-MM-DD`.
  `history`（仅在请求了 `interval` 时出现）：  
  `{ interval, since, until, points: [{ date, open?, high?, low?, close, volume? }] }`，  
  按最旧在前排序，上限 400。对于盘中区间（`1m` 和 `30m`），每个点的 `date` 是完整的 ISO 时间戳，使同日的 K 线保持可区分；`1d`/`1w`/`1mo` 使用 `YYYY-MM-DD`。
- **History window (`from`/`to`)** uses the inherited `since`/`until`, applied
  client-side to the candle set: `until` defaults to **today**; `since`
  defaults to an **interval-based look-back** ending at `until`
  (`1m` ≈ 1 day, `30m` ≈ 1 week, `1d` ≈ 3 months, `1w` ≈ 1 year,
  `1mo` ≈ 5 years). Bars outside `[from, to]` are dropped, and
  `history.since`/`history.until` echo the **effective** (resolved) window.
  Bounds are date-granular (`YYYY-MM-DD`).
  **历史窗口（`from`/`to`）** 使用继承的 `since`/`until`，在客户端应用于 K 线集合：`until` 默认为**今天**；`since` 默认为截止于 `until` 的**按区间回看**（`1m` ≈ 1 天，`30m` ≈ 1 周，`1d` ≈ 3 个月，`1w` ≈ 1 年，`1mo` ≈ 5 年）。`[from, to]` 之外的 K 线被丢弃，`history.since`/`history.until` 回显**生效的**（解析后的）窗口。边界为日期粒度（`YYYY-MM-DD`）。

`vd.candles` is a **top-level sibling of `entity`**, keyed
`{ daily, weekly, monthly, thirty_minute, one_minute }`; each bar is
`{ open, high, low, close, volume, timestamp }` (timestamp = epoch seconds). The
`interval` maps `1m→one_minute`, `30m→thirty_minute`, `1d→daily`,
`1w→weekly`, and `1mo→monthly`. The caller selects exactly one series;
`30m` is not derived from `1m`. For current-day/current-session charts choose
`1m`; choose `30m` for coarser multi-day intraday charts.

`vd.candles` 是 `entity` 的**顶层兄弟字段**，键为 `{ daily, weekly, monthly, thirty_minute, one_minute }`；每根 K 线是 `{ open, high, low, close, volume, timestamp }`（timestamp = epoch 秒）。`interval` 映射为 `1m→one_minute`、`30m→thirty_minute`、`1d→daily`、`1w→weekly`、`1mo→monthly`。调用者只选择一个序列；`30m` 不是从 `1m` 派生的。当日/当前交易时段图表选 `1m`；更粗的多日盘中图表选 `30m`。

> `market_status` is not surfaced by the integration yet, so it maps to `null`.  
> **Monthly candles are upstream-gated (empty today)**, so `history` for  
> `interval: "1mo"` returns empty `points` until that lands. Landed via the  
> finance NLQ + WWW decoder work (D107985840, D108000766); monthly tracked in  
> D107789956.

> 集成尚未提供 `market_status`，因此它映射为 `null`。  
> **月线被上游门禁（当前为空）**，因此 `interval: "1mo"` 的 `history` 在其落地前返回空的 `points`。通过  
> 金融 NLQ + WWW 解码器工作落地（D107985840、D108000766）；月线在  
> D107789956 中跟踪。

---

## 3. Sports / 3. 体育

Landed support is the **flat per-event list**, now carrying the normalized
scope (`sport`/`league`/`season`) and explicit `home`/`away` slots that the
upstream decoder surfaces at the top level of `vertical_data`:

已落地的支持是**扁平的逐赛事列表**，现在承载上游解码器在 `vertical_data` 顶层提供的规范化范围（`sport`/`league`/`season`）和显式的 `home`/`away` 槽位：

```ts
export const TOOL_SPORTS_DATA_RESULT_SCHEMA = z.object({
  summary: z.string(), // display-only
  items: z.array(
    z.object({
      title: z.string(),
      summary: z.string(),       // display-only (short excerpt body)
      url: nullableString.optional(),
      sport: nullableString.optional(),   // normalized token, e.g. "basketball"
      league: nullableString.optional(),  // "NBA" | "NFL" | "MLB" | … (omitted when ambiguous)
      season: z.object({                  // flat label OR structured object
        label: nullableString.optional(), // e.g. "2025-26" | "2026 REG"
        year: nullableNumber.optional(),  // e.g. 2025 (coerced from int or "2026")
        type: nullableString.optional(),  // "REG" | "PST"
        name: nullableString.optional(),  // "Regular Season" | "World Cup 2026"
        start_date: nullableString.optional(), // YYYY-MM-DD when provided
        end_date: nullableString.optional(),
      }).nullable().optional(),
      teams: z.array(z.string()).optional(), // order is not a home/away signal
      home: nullableString.optional(),    // home team name (when split upstream)
      away: nullableString.optional(),    // away team name (when split upstream)
      score: nullableString.optional(),      // "home-away", e.g. "90-94" (from home/away.score)
      status: nullableString.optional(),     // "closed" | "inprogress" | "scheduled" | …
      starts_at: nullableString.optional(),  // ISO 8601 UTC (legacy payloads only — see note)
      player_statistics: z.array(z.object({  // per-player, universal across sports
        player: z.string(),
        team: nullableString.optional(),
        position: nullableString.optional(),
        stats: z.record(z.string(), z.union([z.number(), z.string()])),
      })).optional(),                        // absent/empty when the feed omits it
      team_statistics: z.array(z.object({
        team: z.string(),
        qualifier: nullableString.optional(),
        stats: z.record(z.string(), z.union([z.number(), z.string()])),
      })).optional(),
    }),
  ),
  sources: z.array(toolSourceSchema),
});
```

Mapping notes (`worker/src/web_search.ts`):

映射说明（`worker/src/web_search.ts`）：

- `sport`/`league`/`status` read the normalized top-level `vertical_data` fields
  (D108265855 sources them from the KES `results` blob). `sport` is lowercased
  for a stable token contract; `status` falls back to the raw
  `event.attributes.status` for older payloads.
  `sport`/`league`/`status` 读取规范化的 `vertical_data` 顶层字段（D108265855 从 KES `results` blob 取得它们）。`sport` 被小写化以形成稳定的记号契约；对较旧的载荷，`status` 回退到原始的 `event.attributes.status`。
- `season` is normalized from a flat string (`→ { label }`) or a structured
  object into one shape; `year` is coerced from an int **or** a numeric string,
  and `start_date`/`end_date` are carried when present. Absent → null.
  `season` 从扁平字符串（`→ { label }`）或结构化对象规范化为同一种形状；`year` 从整数**或**数字字符串强转而来，`start_date`/`end_date` 在存在时保留。缺失 → null。
- `home`/`away` read the explicit `vertical_data.home`/`away` names; the upstream
  emits `{name, score}`. `score` ("home-away") is derived from those per-side
  scores, falling back to the legacy `event.attributes.results` blob for older
  payloads. `teams` still comes from `competitors`.
  `home`/`away` 读取显式的 `vertical_data.home`/`away` 名称；上游发出 `{name, score}`。`score`（“主队-客队”）由这些单边分数推导，对较旧的载荷回退到遗留的 `event.attributes.results` blob。`teams` 仍来自 `competitors`。
- `starts_at` came from `event.attributes.startDateUTC`. The trimmed decoder
  (D108695383) no longer emits `event.attributes`, so it resolves only for legacy
  payloads and is otherwise null.
  `starts_at` 来自 `event.attributes.startDateUTC`。精简后的解码器（D108695383）不再发出 `event.attributes`，因此它只对遗留载荷可解析，否则为 null。
- `player_statistics`/`team_statistics` arrive as **text** the WWW decoder
  passes through, parsed into the flexible `stats` records here. Both use the
  same block layout (confirmed against live prod):  
  ```
  Statistics - <label>:
    key: value
    key: value
  ```
  `player_statistics`/`team_statistics` 以**文本**形式由 WWW 解码器透传而来，在此解析为灵活的 `stats` 记录。两者使用相同的块布局（已对照线上生产环境确认）：  
  ```
  Statistics - <label>:
    key: value
    key: value
  ```
  - **player** — the label nests the team + qualifier, e.g.
    `"Ariel Hukporti (New York Knicks (Away))"` → `player` + `team`.
    **player** —— 标签嵌套了球队 + 限定词，例如 `"Ariel Hukporti (New York Knicks (Away))"` → `player` + `team`。
  - **team** — the label is `"<Team> (<Qualifier>)"`, e.g.
    `"New York Knicks (Away)"` → `team` + `qualifier`.  
    **team** —— 标签为 `"<Team> (<Qualifier>)"`，例如 `"New York Knicks (Away)"` → `team` + `qualifier`。  
  Values are numbers or quoted strings (e.g. `minutes: "1:52"`). The feed  
  (SportRadar) does not always populate them, so the fields are omitted when  
  empty. Confirmed against live NBA prod traffic; revisit if other sports diverge.
  值是数字或带引号的字符串（例如 `minutes: "1:52"`）。数据源（SportRadar）并不总是填充它们，因此为空时省略这些字段。已对照线上 NBA 生产流量确认；如果其他运动出现偏差再重新审视。

> **Payload trim (D108695383):** the decoder no longer emits the verbatim  
> per-event `summary` (a ~24 KB blob) or the raw `event.attributes` map; the  
> consumer reads only the small parsed fields. The item `summary` is retained but  
> now sources the short `excerpt` body (the giant blob is gone); the redundant  
> `player_statistics_text` raw fallback is dropped.

> **载荷精简（D108695383）：** 解码器不再发出逐字的逐赛事 `summary`（约 24 KB 的大块）或原始的 `event.attributes` 映射；消费方只读取较小的已解析字段。条目的 `summary` 保留，但现在来源于较短的 `excerpt` 正文（大块已移除）；冗余的 `player_statistics_text` 原始回退被删除。

**Deferred (not in this change):** a sport-discriminated `games` union (per-sport
period/score models) and canonical (cross-sport normalized) stat keys — today
stat keys are kept verbatim from the feed. These remain a future follow-up.

**延后（不在本次变更中）：** 按运动区分的 `games` 联合类型（每种运动的节次/比分模型）和规范化的（跨运动统一的）统计键——目前统计键按数据源原样保留。这些仍是未来的后续工作。

---

## 4. Web search (`web_search`; web-only, no `vertical_data`) / 4. Web 搜索（`web_search`；仅 web，无 `vertical_data`）

`web_search(query)` is a general web search run in the action — the **exception** to
the "structure lives in `vertical_data`" principle: it sends **no vertical**
(empty `verticals`), the same plain search the agent's browser_search runs by
default, and MASE returns **no `vertical_data`** for plain web results. So
`buildWebSearchResult` maps `summary.top[]` web entries directly into a typed
`results[]` (it does **not** use `selectByVertical`, which keys off
`vertical_data`).

`web_search(query)` 是在 action 中运行的通用 web 搜索——是“结构位于 `vertical_data`”原则的**例外**：它发送**无垂直领域**的请求（`verticals` 为空），与代理的 browser_search 默认运行的普通搜索相同，而 MASE 对普通 web 结果返回**没有 `vertical_data`**。因此 `buildWebSearchResult` 把 `summary.top[]` 的 web 条目直接映射进类型化的 `results[]`（它**不**使用以 `vertical_data` 为键的 `selectByVertical`）。

### Result (`TOOL_WEB_SEARCH_RESULT_SCHEMA`) / 结果（`TOOL_WEB_SEARCH_RESULT_SCHEMA`）

`results[]`, capped at 20 in upstream rank order, each:

`results[]`，按上游排名顺序上限 20 条，每条包含：

- `title`, `url`, `source` (publisher/hostname), `snippet` (display-only).
  `title`（标题）、`url`、`source`（发布者/主机名）、`snippet`（摘要，仅展示）。
- `published_at` — best-effort, derived primarily from the URL slug
  (`/YYYY/MM/DD/`); a hint, not authoritative.
  `published_at` —— 尽力而为，主要从 URL slug（`/YYYY/MM/DD/`）推导；是提示，不是权威值。
- `last_updated_raw` — the raw freshness phrase ("3 hours ago") exposed
  verbatim, **never parsed** (unreliable in both directions — the stale-source
  trap).
  `last_updated_raw` —— 原始新鲜度短语（“3 hours ago”）按原样暴露，**绝不解析**（双向都不可靠——即陈旧来源陷阱）。
- `favicon_url` — external source-domain icon URL the Space does **not** own;
  render only as a small source icon with an `onError` fallback, never as content
  imagery (use `media.generate_image` or self-host for real pictures). There is no
  `thumbnail` field — these tools return no real page images.
  `favicon_url` —— Space **不**拥有的外部来源域名图标 URL；只能作为带 `onError` 回退的小来源图标渲染，绝不能当作内容图像（真实图片请用 `media.generate_image` 或自行托管）。没有 `thumbnail` 字段——这些工具不返回真实的页面图像。
- `rank`, `is_index_page` (heuristic: section/tag/index page vs a real page).
  `rank`（排名）、`is_index_page`（启发式判断：栏目/标签/索引页还是真实页面）。

The result's `vertical` field is `""` (web_search pins no vertical).

结果的 `vertical` 字段为 `""`（web_search 不固定任何垂直领域）。

### Routing / 路由

`web_search` runs in the action — the same search the agent's browser search
runs — and is the path for any web query on any subject. Summarize results
with `ctx.inference.complete` when you need a narrative. For a stock
quote/price/history use `finance_ticker`; for scores/schedules/stats use  
`sports_data`.  
Don't spawn a Space task just to search — `spawnTask` runs this same search.
Reserve a task for what the agent loop adds *beyond* search: opening and reading
full pages (`browser.open`), browsing across several sites, multi-step research,
or other agent tools.

`web_search` 在 action 中运行——与代理的浏览器搜索运行的是同一个搜索——是任何主题的任何 web 查询的路径。需要叙述性内容时，用 `ctx.inference.complete` 汇总结果。股票报价/价格/历史用 `finance_ticker`；比分/赛程/统计用 `sports_data`。  
不要只为搜索而生成一个 Space 任务——`spawnTask` 运行的就是这个搜索。
任务要留给代理循环在搜索*之外*增加的能力：打开并阅读完整页面（`browser.open`）、跨多个站点浏览、多步骤研究，或其他代理工具。
