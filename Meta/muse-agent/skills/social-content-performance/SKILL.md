---
name: "social_content_performance"
title: "Social Content Performance"
description: "Analyze the user's own Instagram account and post performance using linked-account analytics."
icon: "instagram"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Social Content Performance / 社交内容表现

## Purpose / 用途

Analyze the user's own Instagram account or post performance, changes over
time, comparisons among the user's own posts or accounts, and content ideas
grounded in the user's own results.

分析用户自己的 Instagram 账户或帖子表现、随时间的变化、用户自己的帖子或账户之间的比较，以及以用户自身结果为依据的内容点子。

Do not use it for competitor, peer, industry, or benchmark comparisons. Those
belong to `social_competitor_analysis`, even when the question is phrased as
"how do I compare" or "is this normal for my category?"

不要把它用于竞品、同行、行业或基准比较。那些属于 `social_competitor_analysis`，即使问题被表述为"我比起来怎么样"或"这在我的类目里正常吗？"

【评论】这一段是技能间的路由边界：把"自身数据"与"对标/基准"两类问题分派给不同技能，避免单一技能越界取数。

## Commands / 命令

Run these Instagram analytics commands through `exec`:

通过 `exec` 运行这些 Instagram 分析命令：

```sh
instagram-cli accounts
instagram-cli analytics-metric-metadata \
  --account-id <user_own_fbid> --metric-name <metric>
instagram-cli analytics-account-insights \
  --account-id <user_own_fbid> --time-range last-28d
```

`--metric-name` may be repeated. Instagram account insights read the one
account selected by `--account-id`. For several accounts, invoke the command
once per returned account ID and combine the results. Use
`--include-time-series` only when the question needs a trend.

`--metric-name` 可以重复。Instagram 账户洞察读取 `--account-id` 选定的那一个账户。对多个账户，按每个返回的账户 ID 各调用一次命令并合并结果。只有问题需要趋势时才用 `--include-time-series`。

For post content, use the installed Instagram CLI and follow its
account-resolution and access rules:

帖子内容使用已安装的 Instagram CLI，并遵循其账户解析与访问规则：

```sh
instagram-cli accounts
instagram-cli posts --account-id <user-own-fbid> [filters]
instagram-cli post --account-id <user-own-fbid> --id <post-id>
```

Never pass an identifier to the Instagram CLI unless its documented flow
returned it. If the CLI cannot resolve the requested account or post,
say that the post-level data is unavailable; do not substitute another account
or infer post metrics from account totals.

绝不向 Instagram CLI 传入非其文档化流程返回的标识符。如果 CLI 无法解析所请求的账户或帖子，就说明帖子级数据不可用；不要用别的账户替代，也不要从账户总量推断帖子指标。

## Workflow / 工作流

### 1. Resolve the account / 1. 解析账户

Begin with `instagram-cli accounts`. Treat its result as the ownership and
permission boundary.

从 `instagram-cli accounts` 开始。把其结果当作所有权与权限边界。

- If exactly one returned account matches the request, use its handle and ID.
  若恰好有一个返回账户匹配请求，使用其 handle 与 ID。
- If several accounts could match, ask one concise question listing their
  handles, then stop until the user chooses.
  若多个账户都可能匹配，提出一个列出它们 handle 的简明问题，然后停下等用户选择。
- If a named account is not returned, explain that this analysis can use only
  an account the user manages, then stop.
  若点名的账户不在返回结果中，解释本分析只能使用用户管理的账户，然后停止。

Keep opaque account and post IDs inside tool calls. Do not show them to the
user.

不透明的账户与帖子 ID 只留在工具调用内部。不要展示给用户。

### 2. Read the requested evidence / 2. 读取所需证据

For account metrics, run `analytics-account-insights` through `instagram-cli`.
Match the requested time range where the command supports it.

账户指标通过 `instagram-cli` 运行 `analytics-account-insights`。在命令支持的范围内匹配所请求的时间范围。

For a content-performance question, also read the relevant posts through the
Instagram path. Account insights are account-level evidence and cannot
establish which individual post performed best. Likewise, captions, media, and
dates do not establish reach, views, saves, or shares unless a post-level
response contains those values.

对内容表现问题，还要通过 Instagram 路径读取相关帖子。账户洞察是账户级证据，无法确立哪条帖子表现最好。同样，文案、媒体与日期不能确立触达、浏览、收藏或分享数，除非帖子级响应包含这些值。

Resolve every metric used in the answer with `instagram-cli
analytics-metric-metadata`. Use the returned display name and definition
verbatim. If no display name is returned, omit that metric; if no definition is
returned, do not invent one.

回答中用到的每个指标都用 `instagram-cli analytics-metric-metadata` 解析。按原样使用返回的显示名称与定义。若没有返回显示名称，省略该指标；若没有返回定义，不要编造。

Do not use web search, social search, outside benchmarks, or remembered figures
as evidence about the user's account performance.

不要把网络搜索、社交搜索、外部基准或记忆中的数字当作关于用户账户表现的证据。

【评论】这是一条证据来源白名单约束：账户表现只认当前工具输出，禁止用检索结果或模型记忆补数，属于防幻觉设计。

### 3. Analyze at the returned grain / 3. 在返回的粒度上分析

Identify the metric, grain, entity, and window that answer the question. Compare
only like-for-like values, accounting for material differences in post age,
format, and paid versus organic delivery.

确定能回答问题的指标、粒度、实体与窗口。只比较同类的值，并考虑帖子年龄、格式以及付费与自然分发的实质性差异。

- Preserve paid/organic and follower/non-follower breakdowns when present.
  存在付费/自然与粉丝/非粉丝细分时保留它们。
- Never attach an account-wide value to one post or format.
  绝不把账户级数值安到单条帖子或单个格式上。
- Do not sum reach or accounts-reached across days unless the backend reports
  an aggregate.
  除非后端报告了聚合值，不要跨天累加触达或触达账户数。
- Treat missing and `null` as unavailable, never zero.
  把缺失与 `null` 当作不可用，绝不当零。
- Treat one post as an example, not a pattern.
  把单条帖子当作例子，而不是规律。
- Do not infer causation, audience intent, demographics, sensitive traits, or
  algorithm behavior from correlations.
  不要从相关性推断因果、受众意图、人口统计特征、敏感特质或算法行为。

Analyze the complete eligible set before making a set-level claim such as
`top`, `best`, `average`, `typical`, or a ranking. Keep that completeness in the
analysis; do not display every item unless the user asks for the full set.

在做出 `top`、`best`、`average`、`typical` 之类的集合级论断或排名之前，先分析完整的合格集合。在分析中保持这种完整性；除非用户要求全集，不要展示每一项。

Prioritize the few findings that most directly answer the request. Support each
finding with a clear comparison and both values. Omit secondary patterns unless
they materially qualify the conclusion. Recommend an action only when it follows
from a supported comparison, and promise no outcome.

优先呈现最直接回答请求的少数发现。每个发现都要有清晰的对比与两个数值支撑。省略次要模式，除非它们实质性地限定结论。只有当行动建议来自有支撑的对比时才提出，并且不许诺结果。

### 4. Present the result / 4. 呈现结果

Lead with the answer. State Instagram, the actual returned window, and the
analyzed scope. Present only the strongest findings and the examples needed to
support them. If a ranking uses many posts, state the analyzed population but
show only the relevant leaders unless the user requests the full ranking.

先给答案。说明来源是 Instagram、实际返回的窗口与分析的范围。只呈现最强的发现与支撑它们所需的例子。若排名涉及很多帖子，说明分析的总体，但只展示相关的领先者，除非用户请求完整排名。

Use concise prose for the answer, interpretation, and caveats. Choose supporting
presentations according to what the evidence needs:

答案、解读与注意事项使用简练的行文。根据证据需要选择辅助呈现方式：

- Use a metric-card widget only for a non-comparative snapshot of distinct
  headline KPIs for one account and one window; a card may include a subdued
  prior-period delta as context.
  指标卡 widget 只用于单个账户、单个窗口的不同头条 KPI 的非对比快照；卡片可以包含弱化的上期差值作为背景。
- When presenting a comparison between subjects, use a Markdown table for exact
  values or a chart when the relative shape matters.
  呈现主体之间的比较时，精确数值用 Markdown 表格，相对形态重要时用图表。
- Use no supporting presentation when prose communicates the result clearly.
  行文已能清楚传达结果时不使用辅助呈现。

Combine formats when each communicates a distinct part of the answer. Avoid
repeating the same data across prose, widgets, and tables unless the repetition
is necessary to support a conclusion. Create a widget only when it adds clear
value.

当每种格式各自传达答案的不同部分时组合使用。避免在行文、widget 与表格之间重复同一数据，除非重复对支撑结论必要。只在 widget 有明确增益时创建。

A metric-card grid is a visual dashboard of independent cards, never a
Markdown table or an HTML table. Its cards show different KPIs for the same
subject, not the same KPI repeated across comparison subjects. Each card
contains a short label, one prominent value, and optional subdued prior-period
context such as `+12% vs prior 28d`.

指标卡网格是独立卡片的可视化仪表盘，绝不是 Markdown 表格或 HTML 表格。它的卡片展示同一主体的不同 KPI，而不是同一 KPI 在比较主体间重复。每张卡片包含一个短标签、一个突出数值，以及可选的弱化上期背景，如 `+12% vs prior 28d`。

Build metric cards with a responsive CSS grid using
`repeat(auto-fill, minmax(260px, 1fr))`. Give each card its own rounded border
and consistent padding. Every card remains one normal grid cell; never stretch
a leftover card across the row or use `grid-column` spanning. Avoid table
headers, rows, cells, gridlines, gradients, gauges, decorative icons, heavy
shadows, and oversized type.

指标卡用响应式 CSS 网格构建，使用 `repeat(auto-fill, minmax(260px, 1fr))`。每张卡片有自己的圆角边框与一致的内边距。每张卡片始终保持一个普通网格单元；绝不让剩余卡片横跨整行，也不使用 `grid-column` 跨列。避免表头、行、单元格、网格线、渐变、仪表盘、装饰图标、重阴影与超大字号。

Keep any widget responsive and restrained, using only `--hatch-widget-*`
colors. Build it with `widget.create`, `kind: "html"`, and `present_now: false`.
If creation fails, answer without the widget and do not fetch again.

任何 widget 都保持响应式与克制，只使用 `--hatch-widget-*` 颜色。用 `widget.create`、`kind: "html"` 与 `present_now: false` 构建。若创建失败，不用 widget 作答，也不要再次获取。

Label a displayed post with the first five words of its returned caption,
adding an ellipsis only when more words follow. If no caption is available, use
its returned date and format. Reuse the label consistently and link every
singled-out post with its exact returned permalink. Never construct a permalink
or expose a raw ID. If no verified link exists, describe only the supported
aggregate pattern.

展示的帖子用其返回文案的前五个词作标签，仅当后面还有词时加省略号。若没有文案，用其返回的日期与格式。标签保持一致复用，每条被单独点出的帖子都链接其确切返回的 permalink。绝不构造 permalink 或暴露原始 ID。若没有已核实的链接，只描述有支撑的聚合模式。

If the requested metric is unavailable at the requested grain, say exactly
that and offer the nearest available measure without presenting it as a
substitute fact. Report consent, eligibility, account-linking, and tool errors
faithfully; do not retry a policy denial or work around it with another tool.

若所请求的指标在所请求的粒度上不可用，就确切说明，并提供最接近的可用度量，但不把它当作替代事实呈现。如实报告同意、资格、账户连接与工具错误；不要重试被政策拒绝的操作，也不要用别的工具绕过。

## Final checks / 最终检查

Before replying, verify:

回复之前，核对：

- every account came from the `instagram-cli accounts` result;
  每个账户都来自 `instagram-cli accounts` 结果；
- every number came from the current tool output or an explicit calculation;
  每个数字都来自当前工具输出或明确的计算；
- every post claim and link came from that same post's row;
  每条帖子论断与链接都来自同一条帖子的行；
- metric names and definitions match metric metadata;
  指标名称与定义同指标元数据一致；
- missing values were not converted to zero;
  缺失值没有被转换为零；
- no peer benchmark, causal claim, or sensitive inference slipped in;
  没有混入同行基准、因果论断或敏感推断；
- the answer contains only the most useful findings and supporting examples;
  答案只包含最有用的发现与支撑例子；
- any widget adds a useful chart rather than duplicating the response;
  任何 widget 都增加了有用的图表而不是重复回复内容；
- no raw account ID, post ID, tool name, or backend field name appears in the
  user-facing answer.
  面向用户的答案中没有出现原始账户 ID、帖子 ID、工具名或后端字段名。
