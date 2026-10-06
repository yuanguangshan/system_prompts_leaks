<!-- BILINGUAL-EN-ZH -->
# Campaign budget / 广告系列预算

Read this after the non-budget inputs required by the estimator and advertiser
constraints are stable. Unresolved requested geography blocks pricing.
Existing-campaign budget diagnosis belongs to `tool-routing.md`.

在估算器所需的非预算输入与广告主约束都已稳定之后再阅读本文。未解决的请求地域会阻塞定价。现有广告系列的预算诊断属于 `tool-routing.md`。

For campaign planning, this is a pricing substep that also requires the
executable destination defined in `campaign-planning.md`, even when the request
emphasizes budget, results, or learning. Return its decision to that controller.
Only a request solely for a price or budget explanation is pricing-only; it does
not require an execution-bound URL or object identity.

对于广告系列规划而言，本文件是一个定价子步骤，它还要求具备 `campaign-planning.md` 中定义的可执行目标，即使请求强调的是预算、结果或学习期也是如此。将其决策返回给该控制流程。只有纯粹要求解释价格或预算的请求才属于仅定价请求；它不需要绑定执行的 URL 或对象标识。

## Price one stable proposal / 为一个稳定方案定价

After compact discovery confirms `ads_budget_estimate`, read its live schema
with `describe-tool --input-only`. Use its current objective, optimization, and
result enums, then run the proposal once through the typed command:

在紧凑发现确认存在 `ads_budget_estimate` 之后，用 `describe-tool --input-only` 读取其实时 schema。使用其当前的目标（objective）、优化（optimization）和结果（result）枚举值，然后通过类型化命令运行一次该方案：

```sh
/opt/hatch/bin/meta-ads-cli estimate-budget \
  --account-id <ACCOUNT_ID> \
  --advertiser-request '<COMPLETE_REQUEST>' \
  --campaign campaign_1 '<CAMPAIGN_NAME>' <OBJECTIVE> <OPTIMIZATION> CBO \
  --ad-set campaign_1 ad_set_1 '<AD_SET_NAME>' \
  --country <COUNTRY_CODE> \
  --target-country ad_set_1 <COUNTRY_CODE> \
  --advantage-audience ad_set_1 on
```

The typed command owns JSON serialization, request correlation, and final
live-schema validation. Do not use `call-tool`, CLI help, source inspection, or
trial calls to construct the request.
Set `yield_ms` to 120000. If execution still backgrounds, wait for its automatic
result; a running command is not a failure and must not be invoked again.

类型化命令负责 JSON 序列化、请求关联以及最终的实时 schema 校验。不要使用 `call-tool`、CLI 帮助、源码检查或试探性调用来构造请求。
将 `yield_ms` 设为 120000。如果执行仍转入后台，等待其自动返回结果；正在运行的命令不是失败，不得再次调用。

- Add every campaign with  
  `--campaign KEY NAME OBJECTIVE OPTIMIZATION CBO|ABO` and every ad set with  
  `--ad-set CAMPAIGN_KEY KEY NAME`.
  使用  
  `--campaign KEY NAME OBJECTIVE OPTIMIZATION CBO|ABO` 添加每个广告系列，使用  
  `--ad-set CAMPAIGN_KEY KEY NAME` 添加每个广告组。
- Prospecting is the default. Add `--campaign-stage KEY retargeting` only when
  current evidence establishes retargeting.
  默认是拉新（prospecting）。只有当前证据能确立再营销（retargeting）时才添加 `--campaign-stage KEY retargeting`。
- Compare every ad set's final targeting before the call. When all are
  identical, use `--country` for plan context, give each ad set one geography
  shape with `--target-country`, `--city-key`, or `--region-key`, and add
  returned `--interest-id`, `--custom-audience-id`, `--locale-id`, `--age-range  
  KEY MIN MAX`, `--gender KEY all|men|women`, and `--advantage-audience KEY  
  on|off` values.
  在调用之前比较每个广告组的最终定向。当它们完全相同时，使用 `--country` 作为方案上下文，用 `--target-country`、`--city-key` 或 `--region-key` 为每个广告组给定一种地域形态，并添加返回的 `--interest-id`、`--custom-audience-id`、`--locale-id`、`--age-range  
  KEY MIN MAX`、`--gender KEY all|men|women` 和 `--advantage-audience KEY  
  on|off` 值。
- Add bounded schedules with `--ad-set-start KEY ISO_8601` and  
  `--ad-set-end KEY ISO_8601`.
  使用 `--ad-set-start KEY ISO_8601` 和  
  `--ad-set-end KEY ISO_8601` 添加有界限的排期。

For resolved city targeting, retain `--country` as context, replace the keyed
country with the returned city key, and append only returned interest IDs.

对于已解析的城市定向，保留 `--country` 作为上下文，用返回的城市键替换带键的国家，并且只附加返回的兴趣 ID。

The estimator publishes one targeting-based cost per campaign. When any ad-set
targeting differs, the only targeting arguments allowed are the campaign's
top-level `--country` values: omit every keyed targeting flag on the first call.
Keep executable audiences unchanged and label the estimate country-level, not
audience-specific. Never test the rejected divergent shape, merge audiences, or
broaden them to make pricing succeed.

估算器为每个广告系列发布一个基于定向的成本。当任何广告组的定向不一致时，唯一允许的定向参数是该广告系列顶层的 `--country` 值：第一次调用时省略所有带键的定向标志。保持可执行受众不变，并将该估算标注为国家层级，而非针对特定受众。绝不要测试被拒绝的分歧形态，也不要为了定价成功而合并或放宽受众。

Budget and constraint flags are plan-level numeric values, never campaign or
ad-set keys:

预算与约束标志是方案层级的数值，绝不是广告系列或广告组的键：

- `--daily-budget AMOUNT` or `--lifetime-budget AMOUNT`
  `--daily-budget AMOUNT` 或 `--lifetime-budget AMOUNT`
- `--max-daily AMOUNT` or `--max-lifetime AMOUNT`
  `--max-daily AMOUNT` 或 `--max-lifetime AMOUNT`
- `--max-cost-per-result AMOUNT`
  `--max-cost-per-result AMOUNT`
- `--min-roas NUMBER`
  `--min-roas NUMBER`

When budget is open, omit daily and lifetime budget flags; never invent a seed.
When the advertiser committed an amount, pass it with `--currency`. A lifetime
budget requires duration or end dates. Use maximum, cost, and ROAS constraints
only when advertiser-supplied.

当预算未定时，省略每日和总预算标志；绝不编造一个种子金额。当广告主已承诺金额时，用 `--currency` 传入。总预算需要时长或结束日期。只有广告主提供了上限、成本和 ROAS 约束时才使用它们。

Pass an advertiser-stated result target with `--goal-value`,  
`--goal-result-type`, and optional `--goal-period day|week|month|total` only  
when it matches the priced optimization event. Never transfer a Purchase, lead,
booking, or other outcome count to views, clicks, or conversations without a
verified conversion rate; retain it only as business context.

只有在与所定价的优化事件匹配时，才用 `--goal-value`、  
`--goal-result-type` 以及可选的 `--goal-period day|week|month|total` 传入广告主声明的结果目标。在没有经过验证的转化率的情况下，绝不要将购买、线索、预订或其他成果数量换算到展示、点击或会话上；只能将其保留为业务上下文。

Use the complete current multi-turn `advertiser_request` required by
`SKILL.md`. Ask before pricing when cadence or currency is ambiguous. Never
supply a cost or CPA; the estimator derives it.

使用 `SKILL.md` 要求的完整当前多轮 `advertiser_request`。当投放节奏或币种不明确时，先询问再定价。绝不要自行提供成本或 CPA；由估算器推导。

Before pricing placements that include Instagram, resolve a compatible
Instagram identity. If none exists, exclude Instagram and state the limitation.

在为包含 Instagram 的版位定价之前，先解析一个兼容的 Instagram 身份。如果不存在，则排除 Instagram 并说明该限制。

Use `total_budget` for the whole proposal and `per_campaign[]` for allocation.
Prefer formatted or major-unit values and preserve both currencies when
conversion is returned. Validate budget mode, assumptions, and warnings against
the proposal. Make one schema-grounded correction for a mismatch; otherwise
leave pricing unresolved.

用 `total_budget` 表示整个方案，用 `per_campaign[]` 表示分配。优先使用已格式化或主单位数值，并在返回换算时保留两种币种。对照方案校验预算模式、假设和警告。若不匹配，做一次有 schema 依据的更正；否则保持定价未解决状态。

## Interpret returned evidence / 解读返回的证据

Amount basis explains why the budget was selected. Cost source, confidence,
sample size, fallback reason, and caveat explain how its projection was
grounded. Neither alone proves what the advertiser should spend.

金额依据解释预算为何被选定。成本来源、置信度、样本量、回退原因和注意事项解释其预测的依据。两者单独都不能证明广告主应当花费多少。

| Basis | Advertiser-facing meaning |
| --- | --- |
| `budget_honored` | Their chosen spend and returned projection |
| `target_derived` | Spend associated with their goal and timeframe; goal-matched, not mandatory or necessarily affordable |
| `account_history` | Compatible observed account behavior; name the evidence and sample |
| `learning_floor` | Estimated spend for roughly 50 optimization events per ad set in rolling seven days; useful for exiting learning quickly, not a minimum or guarantee |
| `account_minimum` | Lowest accepted spend; not proof of useful volume |
| `delivery_floor` | Minimum supported delivery plan; not proof of viability or profitability |
| `mixed` | Explain each material component separately |

| 依据 | 面向广告主的含义 |
| --- | --- |
| `budget_honored` | 他们选择的支出及返回的预测 |
| `target_derived` | 与其目标和时间范围相关联的支出；与目标匹配，但非强制，也不一定负担得起 |
| `account_history` | 账户中观察到的兼容行为；点名证据与样本 |
| `learning_floor` | 为每个广告组在滚动七天内约 50 次优化事件估算的支出；有助于快速退出学习期，但不是最低要求也不是保证 |
| `account_minimum` | 最低可接受的支出；不能证明有用量 |
| `delivery_floor` | 受支持的最低投放计划；不能证明可行性或盈利性 |
| `mixed` | 分别解释每个重要组成部分 |

Treat `account_history` as measured only for the compatible returned sample.
Label `peer_benchmark` and `forecast` as estimates. Treat `hardcode` as a
low-confidence static fallback; its cost and derived amounts are directional.
All expected results remain projections.

仅在返回的兼容样本范围内将 `account_history` 视为实测。将 `peer_benchmark` 和 `forecast` 标注为估算。将 `hardcode` 视为低置信度的静态回退；其成本和推导金额仅具方向性。所有预期结果都仍是预测。

`fallback_reason` explains why a stronger source declined. `cost_caveat`
qualifies the selected source. For example, `cost_not_split_by_stage` means the
account evidence combines prospecting and retargeting; it does not make that
evidence synthetic.

`fallback_reason` 解释更强的来源为何被弃用。`cost_caveat` 对所选来源加以限定。例如，`cost_not_split_by_stage` 表示账户证据混合了拉新和再营销；这并不会使该证据变成合成数据。

Preserve returned shortfalls, warnings, feasibility, and supported actions.
Never silently raise spend, reallocate budget, or drop a campaign. The estimator
takes no placement input, so never call its estimate placement-specific.

保留返回的缺口、警告、可行性和受支持的操作。绝不静默提高支出、重新分配预算或删掉某个广告系列。估算器不接受版位输入，因此绝不要声称其估算针对特定版位。

Use returned formatted amounts. Otherwise display money at the currency's
normal minor-unit precision without feeding display rounding into another
calculation. Translate machine metadata into advertiser language; do not expose
enum tokens, snake case, nulls, or internal statuses. Limit volume interpretation
to the returned projection, source/confidence, and applicable benchmark; leave
stability, effectiveness, and usefulness uncharacterized unless returned.

使用返回的已格式化金额。否则，以该币种正常的最小单位精度显示金额，且不要把显示用的舍入结果带入后续计算。将机器元数据转译为广告主语言；不要暴露枚举记号、snake case、null 或内部状态。对量的解读仅限于返回的预测、来源/置信度以及适用的基准；除非有返回，否则不要对稳定性、效果和实用性下结论。

## Classify the budget decision / 对预算决策分类

Return `settled` or `open` with the basis, amount, projection, learning
relationship, and material constraint:

返回 `settled` 或 `open`，并附上依据、金额、预测、与学习期的关系以及实质性约束：

| Inputs and result | State and handoff |
| --- | --- |
| Supplied amount honored | `settled`; preserve it and its projection |
| Supplied amount below learning floor | Amount stays `settled`; open a feasibility choice and lead with the supported recommendation |
| Supplied amount meets floor, or learning is not applicable | `settled`; no feasibility fork unless another returned constraint creates one |
| No budget; compatible account-history recommendation | `settled`; include evidence and sample |
| No budget; `target_derived` amount | `open`; goal cost is known, affordability is not |
| No budget or goal; only learning floor | `open`; explain it without adopting it |
| Only account minimum or delivery floor | `open`; ask for sustainable spend or an aligned result target |
| Feasibility conflicts with a stated limit or target | Preserve the supplied decision and return the conflict plus supported paths |

| 输入与结果 | 状态与移交 |
| --- | --- |
| 提供的金额被采纳 | `settled`；保留该金额及其预测 |
| 提供的金额低于学习下限 | 金额保持 `settled`；开启一个可行性选择，并以受支持的建议开头 |
| 提供的金额达到下限，或学习期不适用 | `settled`；除非另一个返回的约束造成分叉，否则没有可行性分叉 |
| 无预算；有兼容的账户历史建议 | `settled`；附上证据与样本 |
| 无预算；`target_derived` 金额 | `open`；目标成本已知，可负担性未知 |
| 既无预算也无目标；只有学习下限 | `open`；解释它但不采纳它 |
| 仅有账户最低额或投放下限 | `open`；询问可持续支出或对齐的结果目标 |
| 可行性与声明的限额或目标冲突 | 保留所提供的决策，并返回冲突及受支持的路径 |

A supplied amount does not become open because it equals a minimum or another
amount could buy more results. A target-derived amount remains open whether it
is below, equal to, or above the learning floor because the advertiser has not
accepted its affordability. When neither budget nor goal was supplied, prefer
compatible account history; otherwise never invent a generic small-business
range or default a large learning benchmark.
Never present a minimum-only result as viable, effective, or a starting budget.

提供的金额不会因为它等于某个最低额、或换个金额能买到更多结果，就变成未定（open）。目标推导的金额无论低于、等于还是高于学习下限都保持未定，因为广告主尚未接受其可负担性。当预算和目标都未提供时，优先使用兼容的账户历史；否则绝不编造一个泛泛的小企业区间，也不默认采用一个宏大的学习基准。绝不要把仅有最低额的结果呈现为可行、有效或起步预算。

【评论】“仅给出账户最低额时不得描述为可行或有效”是一条防止低估投放效果的表述约束，与禁止臆测学习表现共同构成对模型输出边界的限制。

For budget constraints, return these option families to the controller:

对于预算约束，向控制流程返回以下选项族：

- Supplied amount below floor: lead with a verified upstream event when it is
  the supported recommendation; otherwise lead with `Keep <supplied amount>`.
  Other applicable paths are `Keep <supplied amount>`, `Use <learning-floor
  amount>`, and `Change the budget`.
  金额低于下限：当验证过的上游事件是受支持的建议时以其开头；否则以 `Keep <supplied amount>` 开头。其他适用路径为 `Keep <supplied amount>`、`Use <learning-floor
  amount>` 和 `Change the budget`。
- Target-derived amount: lead with a verified upstream event when it is the
  supported recommendation; otherwise lead with `Use <goal-matched amount>`.
  Other applicable paths are `Use <goal-matched amount>`, `Use <learning-floor
  amount>` when materially distinct at display precision, and `Choose another
  budget`.
  目标推导金额：当验证过的上游事件是受支持的建议时以其开头；否则以 `Use <goal-matched amount>` 开头。其他适用路径为 `Use <goal-matched amount>`、在显示精度下有实质差异时的 `Use <learning-floor
  amount>`，以及 `Choose another
  budget`。
- No budget, goal, or history: `Use <learning-floor amount>` when returned,
  `Set a budget`, `Set a result goal`.
  无预算、无目标也无历史：若返回了则提供 `Use <learning-floor amount>`，以及 `Set a budget`、`Set a result goal`。

List each path once. Add a verified upstream event only when executable. A spend
option selects the same formatted amount and cadence shown in the explanation.
An account minimum or delivery floor alone is never a `Use <amount>` option;
offer only `Set a budget` and `Set a result goal`.

每条路径只列一次。只有可执行时才添加验证过的上游事件。支出选项选择与解释中展示的相同的格式化金额和节奏。仅凭账户最低额或投放下限绝不能构成 `Use <amount>` 选项；只提供 `Set a budget` 和 `Set a result goal`。

In full-research planning, execute the controller terminal in that response: an
open budget or material feasibility fork calls `muse.create_options` with the
applicable family first, then writes the explanation and question in the final
response and stops after its embedded token; a settled budget with no
other constraint proceeds to the complete plan and its
`Approve this strategy` / `No, make changes` widget.
In guided mode, return every successful result to `campaign-guided.md`; its
budget or feasibility steer is required even when the amount is settled. Never
substitute a prose question or placeholder.

在完整研究规划中，执行该响应的控制流程终点：未定的预算或实质性可行性分叉先用适用选项族调用 `muse.create_options`，然后在最终响应中写出解释和问题，并在其内嵌 token 之后停止；已定且无其他约束的预算则继续生成完整计划及其 `Approve this strategy` / `No, make changes` 组件。
在引导模式中，把每个成功结果返回给 `campaign-guided.md`；即使金额已定，其预算或可行性引导也是必需的。绝不要用一段散文式提问或占位符替代。

## Learning and allocation safeguards / 学习期与分配保障

Use a returned learning floor directly. If no monetary floor is returned but
one ad set has a returned weekly event projection, compare that projection with
roughly 50 events per rolling seven days without inventing a dollar floor.

直接使用返回的学习下限。如果没有返回金额下限，但某个广告组有返回的每周事件预测，则将该预测与滚动七天内约 50 次事件的基准比较，而不要凭空编造一个美元下限。

- At or above the floor, say the projection is expected to support the volume
  recommended for exiting learning quickly; never guarantee exit.
  达到或高于下限时，说明该预测预期可支持快速退出学习期所建议的量；绝不保证退出。
- Below it, say the plan can run and optimize for the event, but projected
  volume is below the benchmark. Stop there: `likely slower learning` and
  `less stable delivery` are unsupported even when phrased as possibilities.
  低于下限时，说明该计划可以运行并为该事件优化，但预测量低于基准。到此为止：`likely slower learning` 和 `less stable delivery` 这类说法即使以可能性的口吻表述也不受支持。
- Do not predict `Learning limited`, an exit date, slower learning,
  instability, or higher cost unless returned.
  除非有返回，否则不要预测 `Learning limited`、退出日期、更慢的学习、不稳定或更高的成本。
- Absence of a floor does not prove learning is absent. If an authoritative
  result marks learning not applicable, say the benchmark does not apply.
  没有下限并不能证明学习期不存在。如果权威结果标记学习期不适用，则说明该基准不适用。
- Learning is per ad set. Use only returned per-ad-set volume; if absent,
  sufficiency is unresolved. Consolidate ad sets without distinct delivery
  mechanics before recommending more spend.
  学习期按广告组计。只使用返回的按广告组的量；如果缺失，充分性即为未解决。在建议增加支出之前，先合并没有独立投放机制的广告组。
- A campaign-level budget is pooled and may allocate unevenly. Never divide it
  or its results by ad-set count. Omit `expected_daily_share` from the
  advertiser-facing response; it is estimator modeling, not an allocation. An
  ad-set floor does not guarantee that delivery amount.
  广告系列层级的预算是共享池，分配可能不均匀。绝不要将它或其结果除以广告组数量。在面向广告主的响应中省略 `expected_daily_share`；它是估算器建模，不是分配结果。广告组下限并不保证该投放金额。

【评论】“滚动七天内约 50 次优化事件”是 Meta 广告学习阶段的公认经验基准；文档要求在该基准未被工具返回时禁止推断学习表现，属于防止模型超出数据依据发挥的约束设计。

## Event feasibility / 事件可行性

Ground event availability in tracking or native-destination evidence. An event
accepted by the estimator proves only that it can be priced.

事件可用性必须基于追踪或原生目标证据。被估算器接受的事件只能证明它可以被定价。

If the requested event cannot be measured, do not price a replacement. Return
the tracking constraint and verified executable paths to
`campaign-planning.md`. If the event is measurable but volume is below its
learning benchmark, recommend the closest verified higher-frequency event first
when evidence supports it as the more feasible path. Retain the requested event
at the current or goal-matched spend as an explicit alternative; never replace
it silently. Without a supported upstream event, lead with the requested event.

如果请求的事件无法测量，不要为替代事件定价。将追踪约束和已验证的可执行路径返回给 `campaign-planning.md`。如果事件可测量但量低于其学习基准，当证据支持某个最接近的已验证更高频事件是更可行路径时，先推荐该事件。同时把请求的事件按当前或与目标匹配的支出保留为显式备选；绝不静默替换它。在没有受支持的上游事件时，以请求的事件开头。

An upstream event optimizes for that action, not the original outcome. Do not
carry the original numeric target into its estimate or equate projected
upstream events with outcomes. Do not promise later retargeting, timing, CPM,
cost, or sales lift without returned evidence.

上游事件为该动作本身优化，而不是原始成果。不要把原始数字目标带入其估算，也不要把预测的上游事件等同于成果。在没有返回证据的情况下，不要承诺之后的再营销、时间安排、CPM、成本或销售提升。

Changing optimization invalidates the estimate. Treat the first result as
feasibility evidence and price only the selected final structure once. Keep a
superseded amount only when it materially explains the change.

更改优化目标会使估算失效。把第一个结果当作可行性证据，只为选定的最终结构定价一次。仅在被取代的金额能实质性解释这一变化时才保留它。

Return the decision to `campaign-planning.md`, which owns the next question,
options widget, or complete review. For a campaign-planning request, do not
finish inside this pricing substep. Guided mode returns to `campaign-guided.md`.

将决策返回给 `campaign-planning.md`，由它负责下一个问题、选项组件或完整评审。对于广告系列规划请求，不要在这个定价子步骤内收尾。引导模式返回 `campaign-guided.md`。

## Failures / 失败处理

A deterministic argument or schema failure leaves budget unresolved until the
input or interface changes. Retry the unchanged request only once when the
failure is explicitly transient. A successful response missing required pricing
fields is incomplete, not a reason to inspect CLI help or repeat the call.

确定性的参数或 schema 失败会使预算保持未解决状态，直到输入或接口改变。只有当失败被明确标记为暂时性时，才对未改变的请求重试一次。缺少必需定价字段的成功响应属于不完整，而不是去查看 CLI 帮助或重复调用的理由。
