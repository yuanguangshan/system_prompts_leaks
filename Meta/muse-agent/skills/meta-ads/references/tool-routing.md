<!-- BILINGUAL-EN-ZH -->
# Meta Ads — choosing the right tool and arguments / Meta Ads——如何选对工具与参数

**This file covers reads.** Write tools — create, update, activate, delete,
connect, upload — are in `references/writes.md`, together with what each one
destroys; choosing a write and knowing its blast radius are the same decision.

**本文件只讲读取。**写入工具——创建、更新、激活、删除、关联、上传——见 `references/writes.md`，其中一并说明每个工具会破坏什么；选择某个写入操作与弄清其影响范围是同一个决定。

Confirm every name here against a successful `meta-ads-cli list-tools
--names-only` result from this conversation, and inspect each selected tool with
`meta-ads-cli describe-tool --name <tool>` before relying on it. Reuse successful
discovery and descriptors later in the same conversation. A failed list,
`status` output, or catalogue from an older CLI does not count. The server
catalogue is gated per tool. Never use bare `list-tools` or a `call-tool` probe
for discovery. A name below may not be exposed to this user; a name exposed to
this user may not be below. What
is durable is the *shape* of the mistakes — those repeat across every catalogue
version.

本文列出的每个名称，都必须与本次会话中一次成功的 `meta-ads-cli list-tools --names-only` 结果核对，并在依赖每个选定工具之前先用 `meta-ads-cli describe-tool --name <tool>` 检查它。同一会话中后续可复用成功的发现结果与描述符。失败的列表、`status` 输出或来自旧版 CLI 的目录都不算数。服务器目录按工具做门控。绝不要用裸 `list-tools` 或 `call-tool` 探测来做发现。下文列出的名称可能没有暴露给当前用户；暴露给当前用户的名称也可能不在下文中。持久不变的是这些错误的*形态*——它们在每一个目录版本中反复出现。

## First resolve which system owns the object / 先判断对象归哪个系统所有

This map starts only after the request has been established as Meta Ads. Do not
enter it merely because the user said `catalog`, `feed`, `product`, or
`audience`; those nouns are shared with other products. Ads ownership comes
from explicit advertising context, an Ads-specific object, prior conversation,
or a read-only lookup that resolves the supplied name/id inside the Meta Ads
account.

只有当请求已被确认为 Meta Ads 之后，才进入这张导航图。不要仅因为用户说了 `catalog`、`feed`、`product` 或 `audience` 就进入；这些名词也属于其他产品。Ads 归属来自明确的广告上下文、Ads 特有对象、此前的对话，或一次把所给名称/id 解析到 Meta Ads 账户内部的只读查询。

A specific catalog or product-feed name/id triggers this read-only lookup even
without prior Ads context. So does a specifically named feed paired with an
upload or refresh-schedule request, such as "change my Shopify nightly feed to
refresh hourly." Perform the Ads lookup before searching cron jobs, hooks,
tracking items, reminders, or local scripts; those are not evidence that an Ads
feed exists or does not exist. Resolve the necessary parent objects, then look
for one exact match with the appropriate Ads list/read tool. A unique match
establishes Ads ownership. No match or multiple plausible matches requires one
short disambiguating question and no write. Do not infer ownership from a word
such as `Shopify` in the object's name. A generic unnamed request such as
"update my feed" is not enough to run an Ads write.

即使没有先前的 Ads 上下文，一个具体的商品目录或商品 feed 名称/id 也会触发该只读查询。一个指名的 feed 搭配一次上传或刷新计划请求时同样触发，例如“把我 Shopify 的夜间 feed 改成每小时刷新”。在搜索 cron 任务、hooks、Tracking 条目、提醒或本地脚本之前，先执行 Ads 查询；这些并不是 Ads feed 存在或不存在的证据。先解析必要的父对象，再用合适的 Ads 列表/读取工具寻找唯一精确匹配。唯一匹配即可确立 Ads 归属。无匹配或有多个合理匹配时，只问一个简短的澄清问题，且不执行任何写入。不要从对象名称里的 `Shopify` 之类的词推断归属。“更新我的 feed”这类泛指且未命名的请求，不足以执行 Ads 写入。

Once a catalog, product feed, product set, pixel/dataset, Custom Audience,
campaign, ad set, ad, or creative is resolved inside Meta Ads, keep every read
and write about that object on this surface. A generic Feed or commerce tool is
not a fallback for a missing Ads tool. Check the live Ads catalogue first; if
the capability is absent, say so plainly.

一旦某个商品目录、商品 feed、商品集、pixel/dataset、Custom Audience、campaign、ad set、ad 或 creative 已在 Meta Ads 内解析确立，关于该对象的所有读写都留在本接口上。通用 Feed 或商务工具不是缺失 Ads 工具时的回退。先查看实时 Ads 目录；若能力不存在，就直说。

Three failure modes account for almost all of them:

三种失败模式几乎解释了全部此类问题：

- **Analysis levels are tool-specific.** An unsupported value may be rejected,
  remapped, or return no usable rows as rollout behavior changes. Use the live
  schema and never treat an empty analysis as proof that the account has no data.
  **分析层级是工具特定的。**随着放量行为变化，不受支持的取值可能被拒绝、被重映射，或返回无可用行。使用实时 schema，绝不要把空分析结果当作账户没有数据的证明。
- **A rejected argument fails the call**, so it is never evidence that the
  advertiser has no such thing.
  **被拒绝的参数会让调用失败**，因此它绝不能作为广告主没有此类东西的证据。
- **A guessed entity id does not error** — it returns *another object's data
  under the user's question*, which reads as a wrong answer rather than a failed
  one.
  **猜测的实体 id 不会报错**——它会在用户的提问下返回*另一个对象的数据*，读起来是一个错误的答案，而不是一次失败的调用。

【评论】“猜 id 不报错、只是答错问题”被列为最危险的失败模式，因为它不产生任何可供拦截的错误信号。

**Do not call `ads_agent`.** If the catalogue offers it, its description will
tell you it is the preferred tool and that it replaces the chain of
`ads_get_*` / `ads_insights_*` calls this file teaches. It is a prototype for
agent-to-agent delegation, and it is out of scope here for a reason that is not
about policy: it returns a *synthesized* answer rather than rows, so every
number you would report from it is one you did not retrieve and cannot check
against anything. Every grounding rule in `references/evidence.md` assumes you
saw the data. Build the chain yourself.

**不要调用 `ads_agent`。**如果目录里提供了它，其描述会告诉你它是首选工具，并说它取代了本文件所教的 `ads_get_*` / `ads_insights_*` 调用链。它是一个代理间委派的原型，在此超出范围的原因与政策无关：它返回的是*合成*的答案而非数据行，因此你据它报告的每个数字都不是你自己取回、也无法与任何东西核对的。`references/evidence.md` 中的每一条依据规则都假定你亲眼看到了数据。自己把调用链搭起来。

【评论】该条款预先处理了“工具描述自称首选”的提示词注入面：取舍依据是返回内容能否被核查，而不是工具的自我声明。

## Product concepts and how-to questions / 产品概念与操作方法类问题

For a Meta Ads concept, setup, specification, or general how-to question, call
`ads_get_help_article` first and ground the answer in its result. Do not answer
from model memory, browser search, or a general account/entity query. A pure
definition such as "What does lifetime budget mean?" needs no account discovery.
Policy questions use the retrieval order in `references/policy.md`.

对 Meta Ads 概念、设置、规格或一般操作方法问题，先调用 `ads_get_help_article` 并以其结果作为答案依据。不要凭模型记忆、浏览器搜索或一般账户/实体查询作答。“生命周期预算是什么意思？”这类纯定义不需要账户发现。政策类问题使用 `references/policy.md` 中的检索顺序。

## Performance and diagnosis / 效果与诊断

| The user is asking about | Call |
|---|---|
| standard delivery readout, or an account / campaign / ad set / ad comparison or ranking — however many metrics are requested | `ads_get_ad_entities`, at the level the question named |
| why a metric moved, including a drop or rise described as sudden; a trend or time series of CPC/CPM/cost per result/ROAS/CTR/CVR | `ads_insights_performance_trend` |
| whether the account has any detected anomaly or unusual signal, without asking for one named metric's movement over time | `ads_insights_anomaly_signal` |
| auction competitiveness — quality or bid ranking, why ads under-deliver, audience overlap | `ads_insights_auction_ranking_benchmarks` |
| how the account compares to similar advertisers or the industry | `ads_insights_industry_benchmark` |
| which optimization goal or objective fits their business and funnel | `ads_insights_advertiser_context` |
| what budget a campaign that does not exist yet would need, and what it would cost per result — "how much should I spend on this?" | `ads_budget_estimate` |
| whether budget or bid settings are capping delivery — "is anything limited by budget?", "where is there room to scale?" | `ads_insights_budget_scaling_analysis` |
| whether the budget *structure* costs efficiency — "should I consolidate?", "too many ad sets?" | `ads_insights_budget_liquidity_analysis` |
| what a *different* budget would produce — "what should I expect at each budget level?" | `ads_insights_budget_allocation_simulation` |
| account-level Opportunity Score and its recommendations | `ads_get_opportunity_score` |
| recent account changes, edit history, audit trail, who changed what and when | `ads_account_get_activity_logs` |
| account or delivery errors, why an ad is not running | `ads_get_errors` |
| which fields, breakdowns, or filter operators can be queried | `ads_get_field_context` |
| defining or explaining a metric or its formula | `ads_get_metric_definition` |

| 用户在问什么 | 调用 |
|---|---|
| 标准投放读数，或账户 / campaign / ad set / ad 的对比或排名——无论要求多少个指标 | `ads_get_ad_entities`，用问题所指名的层级 |
| 某指标为何变动，包括被称为“突然”的下降或上升；CPC/CPM/单次成果成本/ROAS/CTR/CVR 的趋势或时间序列 | `ads_insights_performance_trend` |
| 账户是否存在任何被检出的异常或非常规信号，而非询问某个指名指标随时间的变化 | `ads_insights_anomaly_signal` |
| 竞价竞争力——质量或出价排名、广告为何投放不足、受众重合 | `ads_insights_auction_ranking_benchmarks` |
| 账户与相似广告主或行业相比表现如何 | `ads_insights_industry_benchmark` |
| 哪种优化目标或营销目标适合其业务与漏斗 | `ads_insights_advertiser_context` |
| 尚未创建的 campaign 需要多少预算、单次成果成本是多少——“这个我该花多少钱？” | `ads_budget_estimate` |
| 预算或出价设置是否限制了投放——“有没有什么被预算限制住了？”“哪里还有放大空间？” | `ads_insights_budget_scaling_analysis` |
| 预算*结构*是否损失效率——“我该合并吗？”“ad set 是不是太多了？” | `ads_insights_budget_liquidity_analysis` |
| *另一个*预算会产生什么结果——“每个预算档位我该预期什么？” | `ads_insights_budget_allocation_simulation` |
| 账户级 Opportunity Score 及其建议 | `ads_get_opportunity_score` |
| 近期的账户变更、编辑历史、审计轨迹、谁在何时改了什么 | `ads_account_get_activity_logs` |
| 账户或投放错误、某条广告为何没有在跑 | `ads_get_errors` |
| 哪些字段、细分维度或过滤操作符可以查询 | `ads_get_field_context` |
| 定义或解释某个指标及其公式 | `ads_get_metric_definition` |

Match on intent, not exact words. You may call more than one tool when a question
genuinely spans intents, but lead with the single tool that most directly answers
it.

按意图匹配，而不是按字面词。当问题确实横跨多个意图时可以调用多个工具，但要以最直接回答该问题的那一个工具为主。

### Specialized reads are the primary call / 专用读取是首选调用

When one row above matches the question, call that tool before any general or
neighboring Ads tool. An `ad_account_id` or entity ID written in the user's
request is already available for a read: pass it directly when the selected
tool's live schema accepts it. Do not make `ads_get_ad_accounts`,
`ads_get_ad_entities`, `ads_get_field_context`, Opportunity Score, anomaly,
trend, scaling, or another analysis tool a prerequisite merely to gather context.

当上表某一行匹配问题时，先调用该工具，再考虑任何通用或相邻的 Ads 工具。用户请求中写明的 `ad_account_id` 或实体 ID 已经可用于读取：只要所选工具的实时 schema 接受，就直接传入。不要为了收集上下文，就把 `ads_get_ad_accounts`、`ads_get_ad_entities`、`ads_get_field_context`、Opportunity Score、异常、趋势、扩量或其他分析工具当成前置条件。

The specialized calls are not interchangeable:

这些专用调用彼此不可互换：

- "Why did this metric fall/rise over time?" is performance trend, even when the
  user says "suddenly." "Are there any anomalies?" is anomaly signal.
  “这个指标为什么随时间下降/上升？”属于效果趋势，即使用户说的是“突然”。“有没有异常？”属于异常信号。
- "What would another budget produce?" or "is it worth adding budget?" is
  allocation simulation. Current delivery caps are scaling analysis; fragmented
  budgets, consolidation, ABO/CBO structure are liquidity analysis.
  “换一个预算会怎样？”“加预算值不值？”属于预算分配模拟。当前投放上限属于扩量分析；预算碎片化、合并、ABO/CBO 结构属于流动性分析。
- "How does this compare with the industry or similar advertisers?" is industry
  benchmark, not an internal entity comparison.
  “这与行业或相似广告主相比如何？”属于行业基准，而不是内部实体对比。
- A Meta Ads definition, setup step, or product behavior is a help-article read,
  not a browser search or an answer from memory.
  Meta Ads 的定义、设置步骤或产品行为属于帮助文章读取，而不是浏览器搜索或凭记忆作答。

Add another Ads read only when the user asked for a second distinct result, a
required ID is genuinely missing, or the primary tool's successful output names
one specific evidence gap. Do not fan out preemptively. A failed adjacent mock or
tool is not evidence that the primary specialized capability is unavailable.
If the primary specialized call succeeds and answers the requested intent, stop
making Ads data calls and compose the response. Do not treat a concise or
synthetic-looking result as permission to call trend, entity, field-context, or
account-discovery tools for enrichment. In particular, a successful anomaly
result answers an anomaly-detection request; only add a trend call when the user
also asked why a named metric moved over time.

只有当用户要求第二个不同结果、必需 ID 确实缺失、或主工具的成功输出指出了某个具体证据缺口时，才追加另一次 Ads 读取。不要预先扇出。相邻的 mock 或工具失败并不能证明主专用能力不可用。如果主专用调用成功并回答了所请求的意图，就停止 Ads 数据调用并组织回复。不要把结果简洁或看起来合成，当作调用趋势、实体、字段上下文或账户发现工具来“加料”的许可。特别是，成功的异常结果已回答了异常检测请求；只有当用户还问了某个指名指标为何随时间变动时，才追加趋势调用。

### Analysis level / 分析层级

Insights tools take `analysis_level`; `ads_get_ad_entities` takes `level`. Never
carry a literal between those two families: entity levels are lower-case, while
the insights tools that expose a level declare upper-case, tool-specific values.
Some insights tools expose no level at all. Follow each selected tool's live
schema, omit `analysis_level` when it is absent, and make a separate call per
level rather than combining levels in one call.

Insights 工具接受 `analysis_level`；`ads_get_ad_entities` 接受 `level`。绝不要在这两个家族之间搬运字面值：实体层级是小写，而暴露层级的 insights 工具声明的是大写且随工具而定的取值。有些 insights 工具完全不暴露层级。遵循每个所选工具的实时 schema，没有 `analysis_level` 时就省略，并且每个层级单独调用一次，而不是把多个层级合并进一次调用。

**A plain account KPI or standard delivery readout uses
`ads_get_ad_entities` at `level=ad_account` and the requested window.** For an
analysis question such as "why did cost per result jump for my account?", use the
analysis tool only at a level its live schema supports. An empty analysis is not
an account-wide no-data result: read the exact requested window through
`ads_get_ad_entities` at the requested entity level before describing anything
as absent. If a metric is not defined at that level, read one level down and
label the narrower scope rather than re-labelling it as an account total.

**普通的账户 KPI 或标准投放读数，使用 `ads_get_ad_entities`、`level=ad_account` 加所请求的时间窗口。**对于“为什么我账户的单次成果成本跳涨了？”这类分析问题，只在分析工具实时 schema 支持的层级上使用它。空的分析结果不等于账户全域无数据：在把任何东西描述为“不存在”之前，先用 `ads_get_ad_entities` 在所请求的实体层级读取所请求的确切时间窗口。如果某指标在该层级没有定义，就向下读取一个层级并标注更窄的范围，而不是把它重新标注成账户总量。

**Budget tools are stage-specific.** Use `ads_budget_estimate` for an
uncreated proposal, allocation simulation for an existing campaign's what-if,
scaling analysis for current delivery caps, and liquidity analysis for
fragmentation. Do not substitute one for another. Label simulations as
forecasts at their assumed budget; when a delivery analysis returns nothing,
state what was absent rather than narrating the query.

**预算工具按阶段划分。**未创建的提案用 `ads_budget_estimate`，现有 campaign 的假设推演用分配模拟，当前投放上限用扩量分析，预算碎片化用流动性分析。不要彼此替代。把模拟标注为在其假设预算下的预测；投放分析没有返回内容时，说明缺的是什么，而不是复述查询过程。

### Reading `ads_get_ad_entities` / 读取 `ads_get_ad_entities`

- Choose the level that matches the question: `ad_account` for an overall view,
  `campaign` to compare campaigns, `adset` or `ad` to drill in. Answer at the
  exact level the user named.
  选择与问题匹配的层级：`ad_account` 看总体，`campaign` 比较 campaign，`adset` 或 `ad` 用于下钻。按用户指名的确切层级回答。
- Fetch only the rows your answer will show. When more entities qualify than a
  reply can present — about 20 — "which", "each", "every" or "all" still does
  not mean fetch them all: pass `sort` on the metric the question turns on (for
  example `amount_spent_descending`) with a `limit` near what you will show,
  answer from those rows, and say how many you showed and that more exist.
  Fetch the rest, with `limit` up to 1000, only after the user asks for them.
  A plural ask ("which ad sets…") still gets more than one entity, unless you
  say explicitly that only one qualifies.
  只取回复会展示的行。当符合条件的实体多于一条回复能呈现的数量——约 20 个——"which"、"each"、"every" 或 "all" 也不意味着全部取回：对问题所依托的指标传 `sort`（例如 `amount_spent_descending`），`limit` 取接近将要展示的数量，从这些行作答，并说明展示了多少条、还存在更多。其余的（`limit` 最多 1000）只在用户要求时再取。复数提问（“哪些 ad set……”）仍然应返回不止一个实体，除非你明确说明只有一个符合条件。
- Request only the fields the question needs, not everything.
  只请求问题需要的字段，而不是全部字段。
- Preserve the user's time-window shape. When the live schema exposes their
  named window as a `date_preset`, use that preset rather than calculating
  calendar dates; the server owns inclusivity and the ad-account timezone. Use
  `time_range` for explicit calendar dates or when no matching preset exists.
  For comparisons, use the schema's native comparison shape or query each
  period separately. For "all time" or "lifetime", use `maximum`, not
  `data_maximum`: the latter can pair results with spend that is no longer
  retained, reporting a spend and cost per result of 0.
  保持用户时间窗口的原有形态。当实时 schema 把用户指名的窗口暴露为 `date_preset` 时，使用该预设而不是自行计算日历日期；起止是否含端点与广告账户时区由服务器负责。明确的日历日期或无匹配预设时使用 `time_range`。对比场景使用 schema 原生的对比形态，或对每个时段分别查询。“全部时间”或“生命周期”用 `maximum`，不用 `data_maximum`：后者可能把结果与已不再保留的消耗配对，报告出 0 消耗和 0 单次成果成本。
- Treat status words as query scope, not as fields to display. If the user asks
  for objects that are `active`, `running`, `live`, `still on`, or excludes
  anything `paused`, `stopped`, or `turned off`, call `ads_get_field_context`
  for `effective_status`, then pass its supported filter on every applicable
  `ads_get_ad_entities` call. For the current contract that filter is  
  `"filtering":[{"field":"effective_status","operator":"IN","value":["ACTIVE"]}]`.  
  Merely requesting `effective_status` in `fields` does not filter the rows.
  把状态词当作查询范围，而不是要展示的字段。如果用户要的是 `active`、`running`、`live`、`still on` 的对象，或要排除 `paused`、`stopped`、`turned off` 的对象，先调用 `ads_get_field_context` 查 `effective_status`，然后在每次适用的 `ads_get_ad_entities` 调用中传入其支持的过滤条件。就当前契约而言，该过滤条件是 `"filtering":[{"field":"effective_status","operator":"IN","value":["ACTIVE"]}]`。仅在 `fields` 里请求 `effective_status` 并不会过滤行。
- Use breakdowns — placement, age, platform — only when the user asks why
  something happened or wants a segment view.
  只在用户询问某事为何发生或想要分群视图时才使用细分维度——版位、年龄、平台。
- Call `ads_get_field_context` (it takes only `field_names`) when you are not
  sure a field exists at the level you need, or after a response's
  `additional_info` reports a field unsupported. Never invent a name: drop it,
  or re-query with one the tool confirmed.
  当不确定某字段在所需层级是否存在，或响应的 `additional_info` 报告某字段不受支持时，调用 `ads_get_field_context`（它只接受 `field_names`）。绝不编造字段名：要么丢弃，要么用工具确认过的名称重新查询。
- In `filtering`, each entry's `value` is an array even when it contains one
  value. Confirm the field and operator with `ads_get_field_context`.
  在 `filtering` 中，每个条目的 `value` 都是数组，即使只含一个值。字段与操作符用 `ads_get_field_context` 确认。

**Conversion metrics differ by level, and one is not a stand-in for another.**
`conversions` is not an account-level field, and `cost_per_result` is
unavailable at `ad_account` once the account has more than one result type.
`cost_per_conversion` is the field that survives there, so keep it when the
advertiser asks what a conversion costs at account scope — do not quietly swap
in `cost_per_result`, or the reverse. For an account-wide *cost per result*
question, query at `campaign`, group the rows by the result type each one
returns, compare only within a group, and say plainly that the rows are not an
account-level rollup. `results` is the objective-defined outcome for an ad
object, not an alias for every conversion metric, so never substitute it
silently.

**转化类指标因层级而异，彼此不能互相替代。**`conversions` 不是账户级字段，一旦账户存在多种结果类型，`ad_account` 层级就拿不到 `cost_per_result`。`cost_per_conversion` 是在该层级仍然可用的字段，因此广告主在账户范围内问“一次转化多少钱”时保留它——不要悄悄换成 `cost_per_result`，反之亦然。对账户全域的*单次成果成本*问题，在 `campaign` 层级查询，按各行返回的结果类型分组，只在组内比较，并明确说明这些行不是账户级汇总。`results` 是广告对象由营销目标定义的成果，不是一切转化指标的别名，绝不要悄悄用它替代。

**`ads_get_errors` takes `entity_ids`, not `ad_account_id`.** To check an ad
account, pass the account id as a string inside `entity_ids`.

**`ads_get_errors` 接受 `entity_ids`，而不是 `ad_account_id`。**要检查广告账户，把账户 id 作为字符串放进 `entity_ids`。

### Reading the Opportunity Score / 读取 Opportunity Score

Treat the score as account-level only — never attribute it to a campaign, ad set,
or ad. State it in one clause and go straight to the fixes ("Opportunity score is
74 out of 100. Top fixes:"). Do not explain what the score is or what
higher and lower mean, and do not add reassurance. The score is the bare integer
as returned; "out of 100" is a separate scale annotation. Never write `74/100`,
`score of 74%`, or `74 points` — it is not a percentage, a fraction, or a point
count.

把该分数视为仅限账户级——绝不归因到某个 campaign、ad set 或 ad。用一个从句陈述分数，然后直接进入修复建议（“Opportunity score is 74 out of 100. Top fixes:”）。不要解释分数是什么、高低意味着什么，也不要添加安抚。分数就是返回的裸整数；“out of 100”是单独的量表标注。绝不要写 `74/100`、`score of 74%` 或 `74 points`——它不是百分比、分数或点数。

Order recommendations by `opportunity_score_lift`, highest first, and call that
value **points** (not "impact"). Use `lift_estimate` for the expected benefit and
`recommendation_content.body` for what to change. Where a recommendation has a
`url`, offer it as the place to apply the change; where it has a
`recommendation_signature`, say it can be applied programmatically. Quote
`lift_estimate` and `opportunity_score_lift` only as returned — if a
recommendation has no lift figure, describe the benefit qualitatively rather than
inventing one.

建议按 `opportunity_score_lift` 排序，从高到低，并把该数值称为**点**（而不是“影响”）。预期收益用 `lift_estimate`，要改什么用 `recommendation_content.body`。建议带 `url` 时，把它作为应用该修改的入口提供；带 `recommendation_signature` 时，说明可以程序化应用。`lift_estimate` 与 `opportunity_score_lift` 只按返回原样引用——如果某条建议没有提升数字，就定性描述收益，不要编造一个。

When a recommendation references specific objects, ground it with
`ads_get_ad_entities` so the advice names the actual campaign and its current
budget rather than a generic shape. When the score has no open recommendations,
do not stop there: interpret the retrieved performance metrics for the objects
that are delivering and answer the user's underlying question from those.

当建议引用具体对象时，用 `ads_get_ad_entities` 把建议落到实处，让它指名实际的 campaign 及其当前预算，而不是泛泛的形态。当分数没有待处理建议时，不要到此为止：为正在投放的对象解读已取回的效果指标，并据此回答用户背后的真实问题。

## Catalog and commerce / 商品目录与商务

### One tool per object type / 每种对象类型一个工具

There is no list tool and detail tool to choose between. Pick the tool by the
OBJECT TYPE being asked about, then pass whichever id you already have as
`entity_id`. Reading one entity returns it as a one-row page.

这里不存在“列表工具与详情工具二选一”的问题。按被询问的 OBJECT TYPE 选工具，然后把手头已有的那个 id 作为 `entity_id` 传入。读取单个实体时，它以单行页的形式返回。

| The user is asking about | Use | Pass as `entity_id` |
|---|---|---|
| which catalogs exist on an account or business | `ads_catalog_list_catalogs` | the business id, or omit it to list every catalog the viewer can reach — never an ad account id, which cannot scope this read |
| ONE catalog's metadata or settings | `ads_catalog_list_catalogs` | that catalog's id |
| products in a catalog | `ads_catalog_list_products` | the catalog id |
| ONE product's details or attributes | `ads_catalog_list_products` | that product's id |
| items INSIDE one product set | `ads_catalog_list_products` | that product set's id |
| product sets in a catalog | `ads_catalog_list_product_sets` | the catalog id |
| ONE product set (name, filter, count) | `ads_catalog_list_product_sets` | that product set's id |
| product sets containing ONE product (reverse lookup) | `ads_catalog_list_product_sets` | that product's id |
| feeds on a catalog | `ads_catalog_list_product_feeds` | the catalog id |
| ONE product feed's config or schedule | `ads_catalog_list_product_feeds` | that feed's id |
| upload sessions for ONE feed | `ads_catalog_get_product_feed_upload_sessions` | — takes `product_feed_id`, not `entity_id` |

| 用户在问什么 | 使用 | 作为 `entity_id` 传入 |
|---|---|---|
| 账户或企业下存在哪些商品目录 | `ads_catalog_list_catalogs` | 企业 id；省略则列出查看者可达的每个目录——绝不要传广告账户 id，它无法为该读取划定范围 |
| 某一个目录的元数据或设置 | `ads_catalog_list_catalogs` | 该目录的 id |
| 某个目录中的商品 | `ads_catalog_list_products` | 目录 id |
| 某一个商品的详情或属性 | `ads_catalog_list_products` | 该商品的 id |
| 某一个商品集内部的条目 | `ads_catalog_list_products` | 该商品集的 id |
| 某个目录中的商品集 | `ads_catalog_list_product_sets` | 目录 id |
| 某一个商品集（名称、过滤、数量） | `ads_catalog_list_product_sets` | 该商品集的 id |
| 包含某一个商品的商品集（反向查找） | `ads_catalog_list_product_sets` | 该商品的 id |
| 某个目录下的 feed | `ads_catalog_list_product_feeds` | 目录 id |
| 某一个商品 feed 的配置或计划 | `ads_catalog_list_product_feeds` | 该 feed 的 id |
| 某一个 feed 的上传会话 | `ads_catalog_get_product_feed_upload_sessions` | ——接受 `product_feed_id`，不是 `entity_id` |

**The `ads_catalog_get_*` readers these replaced are gone.**
`ads_catalog_get_catalogs`, `_get_details`, `_get_products`,
`_get_product_details`, `_get_product_sets`, `_get_product_set_details`,
`_get_product_set_products`, `_get_product_product_sets`,
`_get_product_feed_details` and `ads_catalog_search_product` no longer exist.
Calling one does not fail — it returns a SUCCESSFUL result whose only content is
a sentence saying the tool was removed. Do not read that sentence as a statement
about what this product can do, and never tell the advertiser a catalog
capability is unavailable because you saw it: retry with the tool named in the
table above.

**被这些工具取代的 `ads_catalog_get_*` 读取器已不存在。**`ads_catalog_get_catalogs`、`_get_details`、`_get_products`、`_get_product_details`、`_get_product_sets`、`_get_product_set_details`、`_get_product_set_products`、`_get_product_product_sets`、`_get_product_feed_details` 与 `ads_catalog_search_product` 都已不存在。调用它们并不会失败——它返回一个 SUCCESSFUL 结果，内容只有一句“该工具已被移除”。不要把那句话当作对这个产品能力的陈述，也绝不要因为见过它就告诉广告主某项目录能力不可用：改用上表中指名的工具重试。

### Health vs diagnostics vs event source / 健康度、诊断与事件源

These three families sound alike and are consistently confused. They are not
interchangeable.

这三个家族听起来相似，也一直被混用。它们不可互换。

| The user is asking about | Use | Scope |
|---|---|---|
| catalog-wide diagnostics, feed errors, item quality issues | `ads_catalog_get_diagnostics` | ONE catalog |
| dynamic ads delivery health, catalog-DA fit, DA coverage | `ads_catalog_get_dynamic_ads_health` | ONE catalog, scoped to DA |
| pixel or dataset EVENT SOURCE health for a catalog | `ads_catalog_event_source_get_health` | ONE event source under a catalog |
| which event sources are attached to a catalog — "what pixels are connected to this catalog?" | `ads_catalog_event_source_get`, taking the **catalog** id | catalog → its event sources |
| which catalogs an event source feeds — "which catalogs is this pixel linked to?" | `ads_catalog_event_source_get_catalogs`, taking the **event source** id | event source → its catalogs |

| 用户在问什么 | 使用 | 范围 |
|---|---|---|
| 目录级诊断、feed 错误、条目质量问题 | `ads_catalog_get_diagnostics` | 单个目录 |
| 动态广告投放健康度、目录与 DA 的契合度、DA 覆盖 | `ads_catalog_get_dynamic_ads_health` | 单个目录，限于 DA |
| 目录下某个 pixel 或 dataset 事件源的健康度 | `ads_catalog_event_source_get_health` | 目录下的单个事件源 |
| 目录挂了哪些事件源——“这个目录连了哪些 pixel？” | `ads_catalog_event_source_get`，传入**目录** id | 目录 → 其事件源 |
| 某个事件源供哪些目录使用——“这个 pixel 关联了哪些目录？” | `ads_catalog_event_source_get_catalogs`，传入**事件源** id | 事件源 → 其目录 |

The word "health" alone points to `ads_catalog_get_diagnostics` (catalog-wide).
Use `_event_source_get_health` only when the question is scoped to a pixel or
dataset event source, and `_get_dynamic_ads_health` only when it names dynamic
ads.

只出现“health（健康度）”一词时指向 `ads_catalog_get_diagnostics`（目录全域）。只有问题限定在某个 pixel 或 dataset 事件源时才用 `_event_source_get_health`，只有明确提到动态广告时才用 `_get_dynamic_ads_health`。

**The two event-source tools read in opposite directions, and their names do not
say which.** `ads_catalog_event_source_get` takes a catalog and returns its
sources; `ads_catalog_event_source_get_catalogs` takes a source and returns its
catalogs. Sending a catalog id as `event_source_id` is the guessed-id failure
this file opens with — it does not necessarily error, it answers a different
question. Treat "the pixels connected to my catalog" as
`ads_catalog_event_source_get`; it returns every connected type labelled with
`source_type` (`PIXEL`, `APP`, `OFFLINE_CONVERSION_DATA_SET`). CAPI is an
enhancement to a pixel or app source, not a source type of its own, so do not
report it as one.

**两个事件源工具的读取方向相反，而名称并不指明方向。**`ads_catalog_event_source_get` 接受目录、返回其事件源；`ads_catalog_event_source_get_catalogs` 接受事件源、返回其目录。把目录 id 当作 `event_source_id` 传入，正是本文件开头所说的猜 id 失败——它不一定会报错，而是在回答另一个问题。“连到我目录的 pixel 有哪些”应按 `ads_catalog_event_source_get` 处理；它返回每一种已连接的类型，并用 `source_type`（`PIXEL`、`APP`、`OFFLINE_CONVERSION_DATA_SET`）标注。CAPI 是对 pixel 或 app 源的增强，不是一种独立的源类型，不要把它报告为源类型。

### Chains / 调用链

Most former two-step chains are now a single call, because the reverse lookups
take the same `entity_id`: product → its product sets is
`ads_catalog_list_product_sets(entity_id=<product_id>)`, and catalog → its feeds
is `ads_catalog_list_product_feeds(entity_id=<catalog_id>)`. Do not insert a
listing call BETWEEN two steps the reverse lookup already joins -- reaching a
product's sets needs no product listing in front of it.

原来的大多数两步调用链现在都是一次调用，因为反向查找接受同一个 `entity_id`：商品 → 其商品集是 `ads_catalog_list_product_sets(entity_id=<product_id>)`，目录 → 其 feed 是 `ads_catalog_list_product_feeds(entity_id=<catalog_id>)`。不要在反向查找已经衔接的两步之间再插入一次列表调用——获取某商品的商品集，不需要先列商品。

**That is not licence to skip resolving the object the question is about, and
this is the dangerous direction.** `entity_id` has no default. If the advertiser
has not named a catalog and you do not already hold its id from this
conversation, call `ads_catalog_list_catalogs` first -- "what feeds do I have?"
names no catalog. Passing an id you did not resolve does not fail: it returns
that catalog's feeds, successfully, and the advertiser reads another catalog's
data as their own with no error anywhere to catch it. If more than one catalog
comes back, ask which before reading.

**这不是跳过解析问题所指对象的许可，而这正是危险的方向。**`entity_id` 没有默认值。如果广告主没有指名目录、而你会话中也没有其 id，先调用 `ads_catalog_list_catalogs`——“我有哪些 feed？”并没有指名任何目录。传入一个你没有解析过的 id 不会失败：它会成功地返回那个目录的 feed，广告主把另一个目录的数据当成自己的，而全程没有任何错误可供拦截。如果返回多个目录，读取前先问清是哪个。

A chain is still needed when you start from a SKU rather than an id: resolve it
with `ads_catalog_list_products(entity_id=<catalog_id>,
filter={"retailer_id":{"eq":"ABC-001"}})`, then use the `product_id` that comes
back. Do BOTH steps — do not stop at the lookup and hand the user the list.

当起点是 SKU 而不是 id 时仍需调用链：先用 `ads_catalog_list_products(entity_id=<catalog_id>, filter={"retailer_id":{"eq":"ABC-001"}})` 解析，再使用返回的 `product_id`。两步都要做——不要停在查询一步，把列表直接丢给用户。

### Arguments / 参数

Two habits cause almost every failure: reaching for the ad account when the tool
wants a catalog or set id, and pluralising an id.

几乎所有的失败都源于两个习惯：工具要的是目录或商品集 id 却递给广告账户 id，以及把 id 用成复数。

The `ads_catalog_list_*` readers all take `entity_id` — the id of the object
being read — plus `limit` and `cursor`. The remaining `get_*` tools each take
their own id:

`ads_catalog_list_*` 读取器都接受 `entity_id`——被读取对象的 id——外加 `limit` 与 `cursor`。其余 `get_*` 工具各自接受自己的 id：

| Tool | Required | Never pass |
|---|---|---|
| `ads_catalog_list_catalogs` | nothing — omit `entity_id` to list every reachable catalog | `ad_account_id`, `ad_account_ids`, `business_ids` (an ad account cannot scope this read at all; to scope, pass a single business or catalog id as `entity_id`) |
| `ads_catalog_list_products`, `_list_product_sets`, `_list_product_feeds` | `entity_id` | `catalog_id`, `product_set_id`, `after_cursor`, `cursor_after` |
| `ads_catalog_get_diagnostics`, `_get_data_sources`, `_get_dynamic_ads_health` | `catalog_id` | `after_cursor`, `cursor_after` |
| `ads_catalog_get_product_feed_upload_sessions` | `product_feed_id` | `catalog_id` |
| `ads_catalog_get_feed_rules` | `product_feed_id` | `feed_id`, `catalog_id` |
| `ads_catalog_event_source_get`, `_get_health` | `catalog_id` | — |
| `ads_catalog_event_source_get_catalogs` | `event_source_id` | `catalog_id` |

| 工具 | 必需 | 绝不要传 |
|---|---|---|
| `ads_catalog_list_catalogs` | 无——省略 `entity_id` 即列出所有可达目录 | `ad_account_id`、`ad_account_ids`、`business_ids`（广告账户根本无法为该读取划定范围；要限定范围，把单个企业或目录 id 作为 `entity_id` 传入） |
| `ads_catalog_list_products`、`_list_product_sets`、`_list_product_feeds` | `entity_id` | `catalog_id`、`product_set_id`、`after_cursor`、`cursor_after` |
| `ads_catalog_get_diagnostics`、`_get_data_sources`、`_get_dynamic_ads_health` | `catalog_id` | `after_cursor`、`cursor_after` |
| `ads_catalog_get_product_feed_upload_sessions` | `product_feed_id` | `catalog_id` |
| `ads_catalog_get_feed_rules` | `product_feed_id` | `feed_id`、`catalog_id` |
| `ads_catalog_event_source_get`、`_get_health` | `catalog_id` | —— |
| `ads_catalog_event_source_get_catalogs` | `event_source_id` | `catalog_id` |

**`filter` is optional on `ads_catalog_list_products`; omit it to list
everything.** Filter and field names are per-catalog — use only names the tool's
own error or the catalog's schema confirms, and never guess `product_type`,
`product_feed_id`, `custom_label_0`, `images_fetch_status` or `description`. For
paging, pass back the exact cursor the previous response returned under the
argument name `cursor`; a hand-built or renamed cursor is rejected.

**`filter` 在 `ads_catalog_list_products` 上是可选的；省略即列出全部。**过滤与字段名称因目录而异——只使用工具自身报错或目录 schema 确认过的名称，绝不要猜 `product_type`、`product_feed_id`、`custom_label_0`、`images_fetch_status` 或 `description`。翻页时，把上一次响应返回的 cursor 用参数名 `cursor` 原样传回；手工构造或改名的 cursor 会被拒绝。

**`product_id` is a numeric id, not a SKU or a product name.** Passing `MB-001`,
`ring` or `malla` as `entity_id` is rejected. Resolve the numeric id first with  
`ads_catalog_list_products(entity_id=<catalog_id>,  
filter={"retailer_id":{"eq":"MB-001"}})`.

**`product_id` 是数字 id，不是 SKU 或商品名。**把 `MB-001`、`ring` 或 `malla` 作为 `entity_id` 传入会被拒绝。先用 `ads_catalog_list_products(entity_id=<catalog_id>, filter={"retailer_id":{"eq":"MB-001"}})` 解析出数字 id。

## Datasets, pixels, and signals / 数据集、pixel 与信号

| The user is asking about | Use (detail) | Don't use (list) |
|---|---|---|
| ONE dataset's setup or configuration | `ads_get_dataset_details` | `ads_get_datasets` |
| ONE dataset's event quality or grade | `ads_get_dataset_quality` | `ads_get_datasets` |
| ONE dataset's event volume or stats | `ads_get_dataset_stats` | `ads_get_datasets` |
| ONE pixel's event configuration | `ads_pixel_event_read` | `ads_get_datasets` |
| ONE pixel's parameter setup | `ads_pixel_parameter_read` | `ads_get_datasets` |
| custom conversions on an account | `ads_get_customconversions` | — |

| 用户在问什么 | 使用（详情） | 不要用（列表） |
|---|---|---|
| 某一个数据集的设置或配置 | `ads_get_dataset_details` | `ads_get_datasets` |
| 某一个数据集的事件质量或评级 | `ads_get_dataset_quality` | `ads_get_datasets` |
| 某一个数据集的事件量或统计 | `ads_get_dataset_stats` | `ads_get_datasets` |
| 某一个 pixel 的事件配置 | `ads_pixel_event_read` | `ads_get_datasets` |
| 某一个 pixel 的参数设置 | `ads_pixel_parameter_read` | `ads_get_datasets` |
| 账户上的自定义转化 | `ads_get_customconversions` | —— |

Do not answer "how healthy is dataset X?" with `ads_get_datasets` alone — that
lists ids and carries no quality or stats. Always chain to the matching detail
tool.

不要只用 `ads_get_datasets` 回答“数据集 X 健康度如何？”——它只列出 id，不含质量或统计信息。一定要衔接到对应的详情工具。

| Tool | Required | Never pass |
|---|---|---|
| `ads_get_datasets` | `ad_account_id` **or** `business_id` — one of the two | `ad_account_ids` (plural is rejected) |
| `ads_get_dataset_details`, `_quality`, `_stats` | `dataset_id` | `ad_account_id` |
| `ads_get_customconversions` | `ad_account_id` | — |
| `ads_pixel_event_read`, `ads_pixel_parameter_read` | `items` | a bare `ad_account_id` or `pixel_id` at the top level |

| 工具 | 必需 | 绝不要传 |
|---|---|---|
| `ads_get_datasets` | `ad_account_id` **或** `business_id`——两者之一 | `ad_account_ids`（复数会被拒绝） |
| `ads_get_dataset_details`、`_quality`、`_stats` | `dataset_id` | `ad_account_id` |
| `ads_get_customconversions` | `ad_account_id` | —— |
| `ads_pixel_event_read`、`ads_pixel_parameter_read` | `items` | 顶层直接放裸的 `ad_account_id` 或 `pixel_id` |

**`ads_get_datasets` must be scoped.** Calling it with neither `ad_account_id`
nor `business_id` is the single most common failure here.

**`ads_get_datasets` 必须限定范围。**既不给 `ad_account_id` 也不给 `business_id` 就调用它，是这里最常见的单一失败原因。

**`ads_pixel_event_read` and `ads_pixel_parameter_read` take a LIST, not a flat
id.** `items` is a list of per-item read requests; each entry names either the
single object (`event_rule_id` or `parameter_id`) or the pixel to list from
(`pixel_id`). To read one pixel's event configuration, pass one item carrying
that `pixel_id` — not `pixel_id` on its own.

**`ads_pixel_event_read` 与 `ads_pixel_parameter_read` 接受的是一个 LIST，而不是扁平 id。**`items` 是逐项读取请求的列表；每个条目要么指名单个对象（`event_rule_id` 或 `parameter_id`），要么指明要从中列出的 pixel（`pixel_id`）。读取某个 pixel 的事件配置时，传入一个携带该 `pixel_id` 的条目——而不是单独传 `pixel_id`。

Conversions API setup, how-to, and documentation go through
`ads_get_help_article`, not here. Catalog event-source health belongs to the
catalog section above.

Conversions API 的设置、操作方法与文档走 `ads_get_help_article`，不在这里。目录事件源健康度属于上文的目录部分。

## Audiences and Pages / 受众与主页

| The user is asking about | Use (detail) | Don't use (list) |
|---|---|---|
| ONE custom audience's config, size, or source | `ads_get_custom_audience` | `ads_get_ad_account_custom_audiences` |
| which ad sets USE one custom audience | `ads_get_custom_audience_adsets` | `ads_get_ad_account_custom_audiences` |
| ONE account's audience inventory | `ads_get_ad_account_custom_audiences` | `ads_get_custom_audience` |
| creation-bound interests, locations, or languages that need canonical targeting objects | `ads_targeting_search` | free-form names in an ad-set write |

| 用户在问什么 | 使用（详情） | 不要用（列表） |
|---|---|---|
| 某一个自定义受众的配置、规模或来源 | `ads_get_custom_audience` | `ads_get_ad_account_custom_audiences` |
| 哪些 ad set 使用了某一个自定义受众 | `ads_get_custom_audience_adsets` | `ads_get_ad_account_custom_audiences` |
| 某一个账户的受众清单 | `ads_get_ad_account_custom_audiences` | `ads_get_custom_audience` |
| 创建时需要规范化定向对象的兴趣、地区或语言 | `ads_targeting_search` | 在 ad set 写入中直接使用自由格式的名称 |

If the user names an audience by name rather than id, list first to resolve the
id, then chain to the detail tool. Do not stop at the list.

如果用户用名称而不是 id 指名受众，先列表解析 id，再衔接到详情工具。不要停在列表一步。

Pick the Page-listing tool that matches the scope: `ads_get_ad_account_pages` for
Pages connected to an ad account, `ads_get_pages_for_business` for Pages under a
business, `ads_get_user_pages` for Pages available to the current user.

按范围选择主页列表工具：`ads_get_ad_account_pages` 用于广告账户关联的主页，`ads_get_pages_for_business` 用于企业下的主页，`ads_get_user_pages` 用于当前用户可用的主页。

**Most of these take the id of the object they read, and it is not the ad
account.** `ads_get_custom_audience` and `ads_get_custom_audience_adsets` take
`custom_audience_id` and do not accept `ad_account_id` at all.
`ads_get_pages_for_business` takes `business_id`. `ads_get_ig_media` needs
`ig_account_id` — resolve it with `ads_get_ig_accounts` first; the
`ad_account_id` it also accepts does not substitute for it.
`ads_get_ad_account_custom_audiences`, `ads_get_ad_account_pages` and
`ads_get_ig_accounts` are the ones keyed on `ad_account_id`, and
`ads_get_user_pages` takes no id at all.

**这些工具大多接受它们所读对象的 id，而这个对象不是广告账户。**`ads_get_custom_audience` 与 `ads_get_custom_audience_adsets` 接受 `custom_audience_id`，完全不接受 `ad_account_id`。`ads_get_pages_for_business` 接受 `business_id`。`ads_get_ig_media` 需要 `ig_account_id`——先用 `ads_get_ig_accounts` 解析它；它同时接受的 `ad_account_id` 不能替代。以 `ad_account_id` 为键的是 `ads_get_ad_account_custom_audiences`、`ads_get_ad_account_pages` 与 `ads_get_ig_accounts`，而 `ads_get_user_pages` 完全不接受 id。

Use `ads_targeting_search` before an ad-set create or update whenever a chosen
interest, location, or language still needs a platform ID. Batch all terms into
one call. Query the bare place name with the known country and the semantically
correct hint: `city`, `subcity`, `neighborhood`, `region`, `zip`, `address`,
`place`, or `geo_market`; a radius belongs only to an address/place requirement.
Treat queries as candidates, not results. Validate returned name, type, country,
and region together so a same-name place elsewhere is never selected. Pass
returned targeting objects through unchanged:
`targeting_results` to `targeting.interests`, `location_results` to
`targeting.geo_locations`, and `locale_results` to `targeting.locales`. Keep
raw IDs internal and reuse only results from this account and conversation.
When several returned places are plausible, select one only from distinguishing
advertiser or verified Page/business evidence; otherwise ask. Review
`unresolved_*` and warnings before writing. An unresolved exact location is not
permission to widen the geography, and an unresolved interest is not part of
the audience. A supplied ISO country code is already canonical and does not
need a lookup.

在创建或更新 ad set 之前，只要所选兴趣、地区或语言还需要平台 ID，就先使用 `ads_targeting_search`。把所有词合并进一次调用。用已知国家加上语义正确的提示查询裸地名：`city`、`subcity`、`neighborhood`、`region`、`zip`、`address`、`place` 或 `geo_market`；半径只属于地址/地点类需求。把查询当作候选，而不是结果。把返回的名称、类型、国家与区域放在一起校验，以免选中别处的同名地点。返回的定向对象原样传递：`targeting_results` 给 `targeting.interests`，`location_results` 给 `targeting.geo_locations`，`locale_results` 给 `targeting.locales`。原始 ID 留作内部使用，只复用本账户与会话中的结果。当多个返回地点都合理时，只有凭可区分的广告主或经验证的主页/企业证据才能选定其一；否则先询问。写入前检查 `unresolved_*` 与警告。一个未解析的精确地点不是放宽地理范围的许可，一个未解析的兴趣也不属于受众。用户提供的 ISO 国家代码已是规范化形式，无需查询。

This surface covers the paid and ad-linked Instagram lane —
`ads_get_ig_accounts` and `ads_get_ig_media`. Organic post and reel engagement
metrics, post content, and own-account Instagram management belong to the
`instagram` and `instagram-messages` skills, not here.

本接口覆盖付费与广告关联的 Instagram 通道——`ads_get_ig_accounts` 与 `ads_get_ig_media`。自然流量的帖子和 Reels 互动指标、帖子内容以及自有账户的 Instagram 管理，属于 `instagram` 与 `instagram-messages` 技能，不在这里。

## Experiments / 实验测试

| The user is asking about | Use (detail) | Don't use (list) |
|---|---|---|
| ONE A/B test's setup, status, or results | `ads_experiment_abtest_get_test` | `ads_experiment_list_tests` |
| ONE lift test's setup, status, or results | `ads_experiment_lift_get_test` | `ads_experiment_list_tests` |
| ONE account's experiment inventory | `ads_experiment_list_tests` | either detail tool |
| whether an ACCOUNT can create experiments | `ads_experiment_check_eligibility` | any list or detail tool |

| 用户在问什么 | 使用（详情） | 不要用（列表） |
|---|---|---|
| 某一个 A/B 测试的设置、状态或结果 | `ads_experiment_abtest_get_test` | `ads_experiment_list_tests` |
| 某一个提升（lift）测试的设置、状态或结果 | `ads_experiment_lift_get_test` | `ads_experiment_list_tests` |
| 某一个账户的实验清单 | `ads_experiment_list_tests` | 任一详情工具 |
| 某个账户能否创建实验 | `ads_experiment_check_eligibility` | 任何列表或详情工具 |

If the user names a test by name, status, date, or objective rather than by id,
list first to resolve the id, then chain to the matching detail tool. Do not
stop at the list.

如果用户用名称、状态、日期或目标而不是 id 指名测试，先列表解析 id，再衔接到对应的详情工具。不要停在列表一步。

## Creatives and media / 素材与媒体

| The user is asking about | Use | Don't use |
|---|---|---|
| ONE account's creative inventory | `ads_get_creatives` | `ads_get_creative_ads` |
| which ADS use ONE creative | `ads_get_creative_ads(creative_id)` | `ads_get_creatives` |
| ONE existing ad's preview or rendering | `ads_get_ad_preview` | `ads_get_creatives`, `ads_get_ad_images` |
| media images uploaded to the account | `ads_get_ad_images` | `ads_get_creatives` |
| media videos uploaded to the account | `ads_get_ad_videos` | `ads_get_creatives` |

| 用户在问什么 | 使用 | 不要用 |
|---|---|---|
| 某一个账户的素材清单 | `ads_get_creatives` | `ads_get_creative_ads` |
| 哪些广告使用了某一个素材 | `ads_get_creative_ads(creative_id)` | `ads_get_creatives` |
| 某一条已有广告的预览或渲染 | `ads_get_ad_preview` | `ads_get_creatives`、`ads_get_ad_images` |
| 上传到账户的媒体图片 | `ads_get_ad_images` | `ads_get_creatives` |
| 上传到账户的媒体视频 | `ads_get_ad_videos` | `ads_get_creatives` |

For "which ads use my creative X", always use `ads_get_creative_ads(creative_id)`
— `ads_get_creatives` returns creative objects, not the ads referencing them.

对“哪些广告在用我的素材 X”，始终使用 `ads_get_creative_ads(creative_id)`——`ads_get_creatives` 返回的是素材对象，而不是引用它们的广告。

For an existing object, `ads_get_ad_preview` takes `ad_id` or `creative_id`, plus
an optional `ad_format`; resolve it first with `ads_get_creatives`.
`ads_get_creative_ads` takes `creative_id`. An account-wide preview request is
not a single call — name the ad being previewed.

对已有对象，`ads_get_ad_preview` 接受 `ad_id` 或 `creative_id`，外加可选的 `ad_format`；先用 `ads_get_creatives` 解析它。`ads_get_creative_ads` 接受 `creative_id`。账户级的预览请求不是一次调用能完成的——要指名预览的是哪条广告。

**`ad_format` is the placement being previewed**, and one ad renders differently
in each: `MOBILE_FEED_STANDARD` (the default), `DESKTOP_FEED_STANDARD`,
`INSTAGRAM_STANDARD`, `INSTAGRAM_STORY`, `INSTAGRAM_REELS`,
`RIGHT_COLUMN_STANDARD`, `MESSENGER_MOBILE_INBOX_MEDIA`, `THREADS_STREAM`. When
the advertiser asks how an ad looks somewhere specific, pass the matching format
rather than previewing the default and describing the difference. Name the
placement in the advertiser's words — Instagram Stories, Facebook mobile feed —
per the coded-value rule in `references/response-style.md`.

**`ad_format` 是被预览的版位**，同一条广告在每个版位渲染各不相同：`MOBILE_FEED_STANDARD`（默认）、`DESKTOP_FEED_STANDARD`、`INSTAGRAM_STANDARD`、`INSTAGRAM_STORY`、`INSTAGRAM_REELS`、`RIGHT_COLUMN_STANDARD`、`MESSENGER_MOBILE_INBOX_MEDIA`、`THREADS_STREAM`。当广告主问广告在某个具体位置长什么样时，传入匹配的版位，而不是预览默认版位再描述差异。按 `references/response-style.md` 的编码值规则，用广告主的原话指名版位——Instagram 快拍、Facebook 移动版信息流。

Whether the creative mix is healthy, which creative performs best, or why
creative performance changed are performance questions, so the insights tools
above carry them. They do not answer them alone: the numbers rank the ads and the
creative explains them, so read `ads_get_creatives` as well — see "The creative is
part of a performance answer" in `references/analysis.md`.

素材组合是否健康、哪个素材表现最好、素材表现为何变化，都属于效果问题，由上述 insights 工具承载。但它们无法单独回答：数字给广告排名，素材解释原因，因此还要读取 `ads_get_creatives`——见 `references/analysis.md` 中“The creative is part of a performance answer”一节。

## Public Ad Library research / 公共广告库调研

`ads_library_search` queries Meta's public transparency dataset — every active
ad, and for regulated categories historical ads, across Facebook, Instagram,
Messenger and the Audience Network. This is public data about **any** advertiser,
not the user's own account, and the account-scope rules do not apply to it.

`ads_library_search` 查询 Meta 的公开透明度数据集——覆盖 Facebook、Instagram、Messenger 与 Audience Network 上的所有在投广告，受监管品类还包括历史广告。这是关于**任意**广告主的公开数据，不是用户自己的账户，账户范围规则对它不适用。

Use it for competitive research, brand discovery in a market, resolving a brand
name to a Facebook `page_id`, political and issue-ad monitoring, housing /
employment / credit transparency, and public creative examples. Do not use it for
the user's own creatives, their own performance, or policy questions.
Active ads prove only that a pattern is currently used. Never describe one as
working, winning, or effective without separate returned performance evidence.

用于竞品调研、市场中的品牌发现、把品牌名解析为 Facebook `page_id`、政治与议题广告监测、住房 / 就业 / 信贷透明度，以及公开素材示例。不要用于用户自己的素材、自身效果或政策问题。在投广告只能证明某种做法目前有人使用。没有单独返回的效果证据，绝不要把某条广告描述为有效、获胜或起作用。

It returns `estimated_total_count` and up to `limit` (max 50) records, each with
`id`, `page_id`, `page_name`, `ad_creative_link_title`, `ad_creation_time`,
`ad_delivery_start_time`, `ad_snapshot_url`, and `currency`. `ad_creative_body`
is intentionally not exposed — point the user to `ad_snapshot_url` for the full
creative.

它返回 `estimated_total_count` 与最多 `limit`（上限 50）条记录，每条含 `id`、`page_id`、`page_name`、`ad_creative_link_title`、`ad_creation_time`、`ad_delivery_start_time`、`ad_snapshot_url` 与 `currency`。`ad_creative_body` 被有意不暴露——完整素材请让用户查看 `ad_snapshot_url`。

Provide at least one of `search_terms`, `page_ids`, or `countries`. Country codes
are ISO-2 (US, GB, DE, IN). Do not pass `ad_account_id`, `ad_reached_countries`,
or `country` (singular) — none are accepted.

至少提供 `search_terms`、`page_ids` 或 `countries` 之一。国家代码为 ISO-2（US、GB、DE、IN）。不要传 `ad_account_id`、`ad_reached_countries` 或单数形式的 `country`——均不被接受。
