---
description: Live external data in a web artifact (quotes, scores, weather, news, anything current-at-open). Which kind owns the request, the ctx channels for fullstack pages, the build-time snapshot rule for static pages (data fixed at build, never fetched at open), refresh and caching patterns, and the ban on simulated live data.
builders: web
---
<!-- BILINGUAL-EN-ZH -->
# Live external data / 实时外部数据

## First, Which Kind Owns the Request / 首先：哪一类接手该请求

A `web_fullstack` page fetches server-side with the full toolset below. A
`web_static` page has no server and no SDK: `ctx.*` does not exist there, and its
data is fixed when it is built.

`web_fullstack` 页面在服务端用下文的完整工具集获取数据。`web_static` 页面没有服务器也没有 SDK：`ctx.*` 在其中不存在，其数据在构建时即被固定。

- Anything whose content must be current whenever the user opens it (live
  prices, scores, schedules, news): `web_fullstack`.
  凡内容必须在用户每次打开时都保持最新的事物（实时价格、比分、日程、新闻）：`web_fullstack`。
- Any searchable data as a dated snapshot: a static page can bake real data at
  build time through the bundled web-search CLI and present it labeled as of its
  fetch time (pattern below).
  任何可作为带日期快照呈现的可搜索数据：静态页面可以在构建时通过内置 web-search CLI 烘焙真实数据，并标注抓取时间后呈现（模式见下文）。
- Anything needing an API key, an account, server-side caching, or a scheduled  
  refresh: `web_fullstack`.
  凡需要 API 密钥、账户、服务端缓存或定时刷新的事物：`web_fullstack`。

If you are building the static kind and the data genuinely needs a server, do not
approximate it in the browser: finish with `web_artifacts.exit_build`
(`status: "failure"`) explaining that it needs to be built server-backed, so the
main agent can rebuild it that way.

如果你在构建静态页面而数据确实需要服务器，不要在浏览器里近似模拟：以 `web_artifacts.exit_build`（`status: "failure"`）结束，说明它需要以服务端支撑的方式构建，以便主代理据此重建。

## In a web_fullstack Artifact: the ctx Channels / 在 web_fullstack 工件中：ctx 通道

Work down this list inside a server action and take the first channel that fits.

在 server action 中按此列表依次排查，取第一个适用的通道。

1. **`ctx.tool.finance_ticker(symbol, options?)`**: quotes and price history. One
   ticker symbol per call ("META GOOG" is rejected); fetch a watchlist with
   `Promise.all`. The result's `instrument` carries price, currency, change,
   change_percent, intraday high/low, 52-week range, `market_status`, and `as_of`;
   request `interval: "1m" | "30m" | "1d" | "1w" | "1mo"` to also get OHLCV
   `history` points (oldest first) for charts. Select the interval from the
   user's requested time range: use `1m` for a current-day/current-session
   chart and `30m` for a coarser multi-day intraday chart. These are independent
   upstream series: on a trading day, `1m` may already carry current-day bars
   while the `30m` series still ends at the prior trading day and carries no
   current-day bars yet. Do not treat `30m` as an aggregation of `1m`, and do
   not switch intervals automatically.
   **`ctx.tool.finance_ticker(symbol, options?)`**：行情报价与价格历史。每次调用一个 ticker 代码（"META GOOG" 会被拒绝）；用 `Promise.all` 抓取自选列表。结果的 `instrument` 含价格、币种、涨跌额、change_percent、日内最高/最低、52 周区间、`market_status` 和 `as_of`；请求 `interval: "1m" | "30m" | "1d" | "1w" | "1mo"` 还能获得用于图表的 OHLCV `history` 数据点（由旧到新）。按用户请求的时间范围选择 interval：当日/当前交易时段图表用 `1m`，更粗粒度的多日日内图表用 `30m`。这些是相互独立的上游序列：交易日当天，`1m` 可能已包含当日 K 线，而 `30m` 序列可能仍止于前一交易日、尚无当日 K 线。不要把 `30m` 当作 `1m` 的聚合，也不要自动切换 interval。
2. **`ctx.tool.sports_data(query)`**: scores and fixtures. Each item carries teams,
   `score`, `status` (`closed` / `inprogress` / `scheduled`), and `starts_at` in
   UTC; player and team statistics are present only when the upstream has them.
   **`ctx.tool.sports_data(query)`**：比分与赛程。每条包含球队、`score`、`status`（`closed` / `inprogress` / `scheduled`）以及 UTC 的 `starts_at`；球员和球队统计仅在上游有时才存在。
3. **`ctx.tool.weather(query, options?)`**: resolved location, current
   conditions, daily forecast, optional hourly, and alerts. When the user asks
   for a specific hourly horizon, set `hourly_hours` to that requested count
   (maximum 48); both web-artifact runtimes apply it to the upstream hourly
   series.
   **`ctx.tool.weather(query, options?)`**：解析后的位置、当前天气、每日预报、可选的逐小时预报以及预警。当用户要求特定的小时数范围时，将 `hourly_hours` 设为所请求数量（最大 48）；两个 web 工件运行时都会将其应用于上游的逐小时序列。
4. **`ctx.tool.web_search(query)`**: everything else. Snippets are usually enough;
   pipe them through `ctx.inference.complete(prompt, { schema })` to distill typed
   rows instead of opening pages. Do not spawn an agent task just to search.
   **`ctx.tool.web_search(query)`**：其余一切。摘要通常已经够用；将其通过 `ctx.inference.complete(prompt, { schema })` 蒸馏为带类型的行，而不是打开页面。不要只为搜索而派生代理任务。
5. **Plain `fetch()` to a public JSON API**: works from any server action; the
   runtime routes it through the platform egress proxy, and simple GET reads are
   covered by the artifact's trusted read authority, scheduled refreshes
   included. Declare every host you will fetch at runtime in the plan's
   `data_plan.declared_hosts` as an audit inventory (omit it when sourcing runs
   only through `ctx.tool.web_search`). Prove the endpoint returns useful data
   for this query, region, and date in the build's test phase; types and docs do
   not.
   **对公共 JSON API 使用普通 `fetch()`**：在任何 server action 中都可用；运行时通过平台出口代理路由，简单的 GET 读取属于工件的受信读取权限范围，定时刷新也包括在内。把运行时将要抓取的每个主机在计划的 `data_plan.declared_hosts` 中声明为审计清单（仅当数据获取完全经由 `ctx.tool.web_search` 时可省略）。在构建的测试阶段证明该端点对此查询、地区和日期确实返回有用数据；类型和文档证明不了。
6. **`ctx.agent.spawnTask(message, { expectsAction })`**: a detached research agent
   with browsing and full tool use, for gathering that genuinely needs multiple
   steps. It returns a task handle immediately, never the answer; the spawned agent
   delivers results by calling the action you name. Never block a page load on it.
   **`ctx.agent.spawnTask(message, { expectsAction })`**：一个具备浏览和完整工具使用的独立研究代理，用于确实需要多个步骤的采集。它立即返回任务句柄，绝不直接返回答案；被派生的代理通过调用你指定的 action 交付结果。绝不让页面加载阻塞在它上面。

Every observation field on the managed channels is nullable by design: the
upstream may omit price, score, or forecast pieces. Degrade the display; do not
throw, and do not substitute invented values.

受管通道上的每个观测字段按设计都可空：上游可能省略价格、比分或预报的某些部分。让显示降级；不要抛异常，也不要用编造的值替代。

**Fetch-on-open is the default freshness pattern.** The page calls a read action
on load; the action serves the latest cached row from `ctx.db` and refreshes from
the source when the cache is stale. Pick a staleness budget that matches the
domain (quotes: minutes while the market is open; weather: an hour; schedules: a
day) and always render the data's `as_of` time. Keep the load path fast: serve
cache first, refresh behind it or on a user-visible refresh control; the managed
channels default to a 90 second timeout inside the action's own deadline.

**打开即抓取是默认的新鲜度模式。**页面在加载时调用读取 action；该 action 先返回 `ctx.db` 中最新的一行缓存，缓存过陈旧时再从源头刷新。选择与领域匹配的陈旧度预算（行情：开盘期间按分钟计；天气：一小时；日程：一天），并始终渲染数据的 `as_of` 时间。保持加载路径快速：先返回缓存，在其后刷新或由用户可见的刷新控件触发；受管通道默认 90 秒超时，包含在 action 自身的时限之内。

**Wiring a scheduled prefetch.** At create time the schedules come from the
build request's `Refresh triggers`. Register each job from the builder seat with
`cron.add_artifact_action` with the requested schedule and verified action
invocation. Record the installed job in `space.json` `managedCronJobs` and
confirm it with `cron.status`. This invokes the action directly and is silent on
success. If the user asked for a chat response after every refresh, use an agent
schedule with an explicit delivery target instead. Scheduled runs execute
unattended under the same web-reading approval. Each occurrence dispatches at
most once. After dispatch, timeout, failure, or interruption can leave committed
effects and will not trigger an automatic retry. Check the saved state before
retrying manually; a crash before the write can leave an occurrence incomplete.

**接好定时预取。**创建时，调度来自构建请求的 `Refresh triggers`。从构建者席位用 `cron.add_artifact_action` 注册每个任务，附带所请求的调度和经过验证的 action 调用。把已安装的任务记录在 `space.json` 的 `managedCronJobs` 中，并用 `cron.status` 确认。它会直接调用该 action，成功时不产生输出。若用户要求每次刷新后都有聊天回复，则改用带明确送达目标的代理调度。定时运行在同样的网页读取审批下无人值守执行。每次触发最多分发一次。分发、超时、失败或中断之后可能留下已提交的副作用，且不会触发自动重试。手动重试前先检查已保存的状态；写入前的崩溃可能让一次触发不完整。

**Cache shape.** Store snapshots keyed by subject (ticker, venue, query) with
`as_of` and any status the source reports (`market_status`, `inprogress`). Readers
take the latest row per key; keep a small retention window and prune the rest so
the table stays bounded.

**缓存结构。**按主体（ticker、场馆、查询）为键存储快照，附带 `as_of` 和源头报告的任何状态（`market_status`、`inprogress`）。读取方取每个键的最新一行；保留较小的保留窗口并清理其余，使表保持有界。

## In a web_static Page: a Dated Snapshot, Fixed at Build / 在 web_static 页面中：构建时固定的带日期快照

Do not wire a static page to fetch data at open: no external APIs from the page, no polling, and never a public CORS
proxy to reach a source that blocks browsers; every open-time dependency is a
third party the user silently takes on, and a page that needs one is the
server-backed kind mis-built as static. Assets are different: CDN libraries,
web fonts, and the page's own images follow the seat's asset rules; the line
here is about data. What a static page carries honestly is a real snapshot:

不要把静态页面接成打开时抓取数据：不从页面调用外部 API、不轮询，也绝不用公共 CORS 代理去访问阻止浏览器的数据源；每一个打开时的依赖都是用户默默承担的第三方，需要这种依赖的页面属于被误建为静态的服务端支撑型。资产另当别论：CDN 库、网络字体和页面自身图片遵循该席位的资产规则；此处这条线针对的是数据。静态页面能诚实承载的是一份真实快照：

- Source the data during the build with the bundled CLI: run
  `/opt/hatch/bin/web-search "<query>" --out <file>` and read the file back;
  stdout carries only a trimmed summary, and large payloads truncate in the
  shell. It is the same search engine the fullstack `ctx.tool.web_search`
  rides. Include the needed timeframe in the query and check source dates;
  use `--verticals finance|sports|weather` for domain hints, one ticker per
  finance query. Each call searches one query; use separate calls for other
  questions.
  构建期间用内置 CLI 采集数据：运行 `/opt/hatch/bin/web-search "<query>" --out <file>` 并读回该文件；stdout 只承载精简摘要，大负载在 shell 中会被截断。它与 fullstack 的 `ctx.tool.web_search` 使用的搜索引擎相同。把所需时间范围写进查询并核对来源日期；用 `--verticals finance|sports|weather` 提供领域提示，金融查询每个只带一个 ticker。每次调用只搜索一个查询；其他问题用单独的调用。
- Distill the JSON into the typed constant the page renders, and keep the
  source names.
  把 JSON 蒸馏为页面渲染所用的带类型常量，并保留来源名称。
- Bake the fetch time in as `as_of` and render it, with the sources named: the
  page presents a dated snapshot, and that is its contract.
  把抓取时间烘焙为 `as_of` 并渲染出来，同时列明来源：页面呈现的是带日期的快照，这就是它的契约。
- When the user later wants newer numbers, an ordinary `edit` re-runs this
  builder and re-bakes.
  用户之后想要更新的数字时，一次普通的 `edit` 会重跑该构建器并重新烘焙。

The CLI is the builder's tool, never the page's: it runs during the build, and
the finished page must not invoke it, reference its path, or wrap it in an
invented endpoint for the page to call. The page renders the baked constant
and nothing else.

CLI 是构建者的工具，绝不是页面的：它在构建期间运行，完成的页面不得调用它、引用其路径，也不得把它包装成虚构的端点供页面调用。页面只渲染烘焙好的常量，别无其他。

A shared copy of a static page carries the same dated snapshot; that is what
sharing preserves.

静态页面的分享副本携带同一份带日期快照；分享保留的正是这一点。

## Non-negotiables (both kinds) / 不可妥协项（两类皆适用）

- **Never simulate live data.** No random-walk price animation, no seeded numbers
  presented as current, no fabricated movement timed to market hours. When the
  source fails, show the honest error or the stale cache labeled with its `as_of`;
  an honest empty state beats a convincing fake.
  **绝不模拟实时数据。**不做随机游走的价格动画，不把种子生成的数字冒充当前值，不按交易时段编造走势。当数据源失败时，展示诚实的错误或标注 `as_of` 的陈旧缓存；诚实的空状态胜过以假乱真的伪造。
- **Session math belongs to the venue, not the viewer.** Trading hours and
  day-bucketing for a fixed market are computed in that market's timezone (US
  markets: `America/New_York` via `Intl.DateTimeFormat`), never from the local
  clock alone.
  **交易时段计算归属市场所在地，而非观看者。**固定市场的交易时段与按日分桶在该市场的时区中计算（美股：通过 `Intl.DateTimeFormat` 使用 `America/New_York`），绝不只按本地时钟。
