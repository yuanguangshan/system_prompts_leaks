<!-- BILINGUAL-EN-ZH -->
# Campaign planning — full research / 广告系列规划 — 完整研究

Use this reference for the default full-research path. Apply account scope and
evidence; read analysis, policy, response style, or general tool routing only
when the decision requires them. Explicit step-by-step requests use
`campaign-guided.md` instead.

默认的完整研究路径使用本参考。应用账户范围与证据；仅当决策需要时才阅读分析、政策、响应风格或通用工具路由。明确的逐步引导请求改用 `campaign-guided.md`。

## Delivery strategy before creative / 创意之前的投放策略

Strategy connects advertiser intent to executable delivery settings. Resolve:

策略把广告主意图连接到可执行的投放设置。需要解决：

1. signals and tracking;
   信号与追踪；
2. objective and optimization;
   目标与优化；
3. destination;
   落地页；
4. compliance;
   合规；
5. geography and audience;
   地域与受众；
6. hierarchy and placements; and
   层级与版位；以及
7. budget and schedule.
   预算与排期。

The current request overrides memory. Prior conversations may support stable
facts the advertiser did not replace, never an old budget, objective,
destination, promotion, or one-off asset.

当前请求覆盖记忆。先前的对话可以支持广告主未替换的稳定事实，但绝不能支持旧的预算、目标、落地页、促销或一次性资产。

This review may name the ad count needed for structure or pricing, but it does
not include creative format, concept, composition, copy, button wording,
in-image text, or production source. Those belong to `campaign-creative.md`
after strategy approval.

本评审可以提及结构或定价所需的广告数量，但不包括创意格式、概念、构图、文案、按钮措辞、图内文字或制作来源。那些属于策略批准后的 `campaign-creative.md`。

## Establish the product brief / 建立产品简报

Before discovery, decide whether the conversation explains what is being
advertised or tested well enough to research it without adding facts. If
describing what the audience receives, how the offering is delivered, or what a
requested test changes would require invention, the brief is incomplete.
Account names, campaign labels, and history cannot fill that gap unless the
advertiser explicitly asks to reuse or duplicate a prior setup. A name, date,
audience, or price alone does not establish the substance of the offering. Ask
the natural product- or test-scope questions needed to remove the actual
ambiguity, grouped into one brief turn. Do not supply example answers or preview
any campaign setting. The response contains only a short reason and the
questions; do not mention research, pricing, planning, or future workflow.
Research—not the advertiser—will recommend goal, destination type, audience,
placements, hierarchy, and schedule. It also recommends budget when the
advertiser leaves affordability open; exact execution inputs still require
advertiser or tool provenance.

在发现环节之前，先判断对话是否已把"在广告什么/在测试什么"解释得足够清楚，从而无需添加事实即可开展研究。如果描述受众会得到什么、产品如何交付、或所请求的测试改变了什么需要凭空编造，那么简报就是不完整的。账户名、广告系列标签和历史无法填补这一空白，除非广告主明确要求复用或复制先前设置。仅凭名称、日期、受众或价格不能确立产品实质。提出消除实际歧义所需的自然的产品或测试范围问题，合并到一个简短的回合中。不要提供示例答案，也不要预览任何广告系列设置。响应只包含简短理由和问题；不要提及研究、定价、规划或未来工作流。是研究——而不是广告主——来推荐目标、落地页类型、受众、版位、层级与排期。当广告主把承受能力留空时，研究也推荐预算；精确的执行输入仍然需要广告主或工具来源。

Ask now only for facts needed to understand the advertised offering or to run
delivery research. A detail needed only to write or render the ad belongs in
the creative stage and must not block strategy research. This turn contains only
offering or test-scope questions plus the budget question below; account, Page,
destination, and other delivery-setting decisions belong to later stages. When
this clarification turn is already required and no spend constraint is known,
it must include one natural budget question. Let the advertiser leave budget
open for the estimator; never require them to invent an amount.

现在只询问理解所广告产品或执行投放研究所需的事实。仅为撰写或渲染广告所需的细节属于创意阶段，不得阻塞策略研究。本回合只包含产品或测试范围问题加上下文的预算问题；账户、Page、落地页与其他投放设置决策属于后续阶段。当本澄清回合本来就需要发起、且尚无任何花费约束可知时，它必须包含一个自然的预算问题。让广告主可以把预算留给估算器；绝不要求他们凭空编一个金额。

The brief is sufficient only when remaining unknown advertiser-owned facts
cannot change a delivery decision or how budget should be interpreted. The
instruction to minimize intake applies only after this test passes. Ask only for
facts that resolve such a dependency, without anchoring the answer to another
product; otherwise continue and state the resulting limitation.

只有当剩余未知的、广告主自有事实无法再改变任何投放决策或预算解释方式时，简报才算充分。"尽量少询问"的指示只在该检验通过之后适用。只询问能解决此类依赖的事实，且不要把答案锚定到另一个产品上；否则继续推进，并说明由此产生的局限。

Use options only when one bounded question is sufficient. When several related
facts are missing, ask them together in plain text. Begin discovery and the
first research call in the same turn after the brief is complete. One brief,
natural status is fine while research continues; the final response contains a
finding, blocker, or decision rather than troubleshooting narration.

仅当一个有界问题足够时才使用选项。当缺少若干相关事实时，用纯文本一起询问。简报完成后，在同一回合开始发现环节和第一次研究调用。研究继续期间可以用一句简短自然的状态说明；最终响应应包含发现、阻塞点或决策，而不是排障叙述。

An omitted budget alone does not create an upfront clarification turn. When the
brief is otherwise sufficient, continue; after other pricing inputs stabilize,
call `ads_budget_estimate` without a seed and resolve its returned basis under  
`campaign-budget.md`.

仅缺失预算本身并不构成一次前置澄清回合。当简报在其他方面充分时继续推进；待其他定价输入稳定后，不带种子调用 `ads_budget_estimate`，并按 `campaign-budget.md` 处理其返回的依据。

## Resolve identity / 确定身份

Apply `account-scope.md`. Silently use the sole enabled account or one clear
name/context match; otherwise show one named account picker and stop. If no
account is available, continue plan-only and state that Ads Manager execution
is unavailable.

应用 `account-scope.md`。存在唯一启用账户、或唯一清晰的名称/上下文匹配时静默使用；否则展示一个具名账户选择器并停止。若无可用账户，继续仅规划模式，并说明 Ads Manager 执行不可用。

Read Pages with `ads_get_ad_account_pages` for the selected account. Use a Page
the advertiser chose or one clear compatible match. If several are plausible,
ask with one picker. If the Page name materially differs from the verified
destination brand, show the exact pairing and ask only `Use <Page name>` /
`Choose another Page`; ignore cosmetic name differences. Never use
`ads_get_pages_for_business` without a returned `business_id` and a real need
for business scope. That list is the account's promoted Pages, not the set the
advertiser may advertise from: `ads_create_creative` takes a `page_id` and
accepts any Page they hold advertising rights to. So when it is empty or holds
no match, widen to `ads_get_user_pages`, which needs no id, and resolve from
there; a Page found only in the wider scope is valid for creation. An empty or
non-matching `ads_get_ad_account_pages` never blocks creation, and is never
reported to the advertiser as something they must fix — do not tell them to
attach a Page in Business Settings.

用 `ads_get_ad_account_pages` 为所选账户读取 Page。使用广告主选择的 Page，或唯一清晰的兼容匹配。若有多个都合理，用一个选择器询问。如果 Page 名称与已核实的落地页品牌有实质差异，展示确切的配对，只询问 `Use <Page name>` / `Choose another Page`；忽略表面上的名称差异。在没有返回 `business_id` 且没有真实的业务范围需求时，绝不使用 `ads_get_pages_for_business`。那个列表是账户推广过的 Page，而不是广告主可以用来投放的 Page 集合：`ads_create_creative` 接受一个 `page_id`，接受他们持有投放权的任何 Page。因此当它为空或没有匹配时，扩展到无需 id 的 `ads_get_user_pages`，并从那里解决；只在更大范围中找到的 Page 同样可用于创建。空或不匹配的 `ads_get_ad_account_pages` 永远不阻塞创建，也永远不会作为"广告主必须修复的问题"报告给他们——不要让他们去 Business Settings 里绑定 Page。

That widening is for the Page only. Resolve Instagram before recommending an
Instagram-only placement or automatic placements that may deliver on Instagram,
use only an Instagram identity returned for the selected account, and never
assume Page identity also covers Instagram. If no usable Instagram identity
exists, revise the placement recommendation or state the limitation rather than
recommending or pricing Instagram placements. Do the same if no Page is usable
at all.

那一扩展仅针对 Page。在推荐 Instagram 专属版位、或可能投放到 Instagram 的自动版位之前先解决 Instagram 身份；只使用为所选账户返回的 Instagram 身份，绝不假定 Page 身份同时覆盖 Instagram。若不存在可用的 Instagram 身份，修订版位推荐或说明该局限，而不是推荐或为 Instagram 版位报价。若完全没有可用的 Page，同样处理。

Reuse stable, same-context choices by name. Refresh a prior campaign through
`ads_get_ad_entities` only when the request refers to that campaign. Do not
carry one-off offers, URLs, seasons, assets, IDs, capability results, or live
delivery state into a new context. Carried choices are defaults, not standing
permission to spend. Never use history to complete an insufficient current
brief; a prior reference authorizes carry-forward only when the advertiser asks
to reuse or duplicate it.

按名称复用稳定的、同一上下文中的选择。仅当请求指向某个先前广告系列时，才通过 `ads_get_ad_entities` 刷新它。不要把一次性优惠、URL、季节、资产、ID、能力结果或实时投放状态带进新的上下文。被携带的选择是默认值，不是持续花费的许可。绝不使用历史来补全当前不充分的简报；只有广告主要求复用或复制某一先前引用时，它才授权顺延。

## Run decision-bearing research / 执行影响决策的研究

`Plan only` suppresses writes, not discovery or decision-bearing reads; run the
same research needed to ground the recommendation.

`Plan only` 抑制的是写操作，不是发现或影响决策的读取；执行同样的研究来支撑推荐。

Use normal `call-tool --agent-output` reads and consume their JSON directly.
Describe only selected tools in the current lane, one tool per `describe-tool`
command; independent descriptor calls may run together. Do not preload creation
schemas. A failed lane is unavailable evidence, not a negative advertiser
finding, and degrades only that lane.

使用常规的 `call-tool --agent-output` 读取并直接消费其 JSON。只描述当前路径中选定的工具，每个 `describe-tool` 命令对应一个工具；相互独立的描述调用可以并行。不要预加载创建 schema。某个路径失败属于不可用证据，不是对广告主的否定结论，且只降级该路径。

Complete the needed lanes without interim questions or partial findings:

在不插入临时提问、也不输出部分结论的前提下完成所需路径：

- **Account context first:** call `ads_insights_advertiser_context` when exposed.
  Treat it as the primary account/history summary. Call `ads_get_ad_entities`
  for history only when context is absent, lacks a fact that could change a
  concrete delivery decision, or the request names an entity that must be
  resolved. Read only compatible product/objective/destination/audience/season
  entities; do not retrieve or average unrelated history.
  **账户上下文优先：**在暴露时调用 `ads_insights_advertiser_context`。将其视为主要的账户/历史摘要。仅在上下文缺失、缺少可能改变某个具体投放决策的事实、或请求点名了一个必须解析的实体时，才为历史调用 `ads_get_ad_entities`。只读取兼容的产品/目标/落地页/受众/季节实体；不要检索或平均无关历史。
- **Tracking:** read only the dataset, event, quality, pixel, and custom-
  conversion facts that can change objective or optimization. Individual reads
  combine into evidence; they are not a fictional unified health tool.
  **追踪：**只读取可能改变目标或优化的数据集、事件、质量、像素与自定义转化事实。单项读取组合成证据；它们不是一个虚构的统一健康工具。
- **Audience:** inspect an existing audience only when the request or account
  context makes it a plausible buildable choice. Use returned IDs only.
  **受众：**仅当请求或账户上下文使某个现有受众成为合理可构建选择时才检查它。只使用返回的 ID。
- **Destination and brand:** verify the destination and only the public facts
  needed to choose delivery settings. Defer creative-market patterns, asset
  inventory, and all Ad Library research to `campaign-creative.md`.
  **落地页与品牌：**核实落地页以及选择投放设置所需的公共事实。把创意市场模式、资产清点与所有广告资料库研究推迟到 `campaign-creative.md`。
- **Performance and benchmark:** call a trend or one relevant benchmark only
  when its inputs are grounded and its result can change the plan. Never infer a
  trend or benchmark from absent data.
  **效果与基准：**仅当输入有据且结果可能改变计划时，才调用趋势或一个相关基准。绝不从缺失数据推断趋势或基准。
- **Policy/help:** call live policy or help only when the category or proposed
  setting materially requires it.
  **政策/帮助：**仅当品类或拟议设置有实质需要时，才调用实时政策或帮助。

Call research complete only after every selected lane has returned or been
recorded as unavailable. Describe exactly what was checked; never upgrade a
small subset of account reads into `everything is verified`. Collection names
and IDs are leads, not evidence for absent fields: follow the smallest useful
set of plausibly compatible objects into detail or trend reads when they could
change a decision, or omit that history. Empty, failed, names-only, or unmatched
results never prove the advertiser lacks history or has never run something.

只有当每个选定路径都已返回、或已被记录为不可用之后，才宣布研究完成。确切描述检查了什么；绝不把一小部分账户读取拔高成 `everything is verified`。集合名称和 ID 只是线索，不是缺失字段的证据：当它们可能改变决策时，沿着最小有用的、可能兼容的对象集合深入到详情或趋势读取，否则省略那段历史。空、失败、只有名称或不匹配的结果，绝不能证明广告主缺乏历史或从未投放过什么。

Apply these disciplines:

应用以下纪律：

- Every cited fact changes a setting or explains why a candidate was rejected.
  每一条被引用的事实都要么改变一个设置，要么解释某个候选为何被否决。
- History is evidence only when product, objective, destination, audience type,
  and relevant season are compatible.
  只有当产品、目标、落地页、受众类型与相关季节都兼容时，历史才是证据。
- Resolve conflicts in this order: compliance, explicit advertiser preference,
  compatible first-party evidence, then safe Advantage+ defaults.
  按此顺序解决冲突：合规、广告主明确偏好、兼容的第一方证据，然后是安全的 Advantage+ 默认值。
- Current intent always grounds the plan; history may support but never replace
  it.
  当前意图始终是计划的根基；历史可以支持它，但绝不能替代它。

Evidence constrains choices: missing or failed evidence may rule out a setting,
but does not establish an alternative. Recommend objective, destination, and
audience from stated intent, affirmative evidence, or a supported safe default
shown as an assumption; otherwise keep the decision open. Execution-bound URLs
and targeting values must come from the advertiser or a tool result. The chosen
destination is stable for pricing only when its required URL or object identity
is supplied or returned; never price a hypothetical destination.
Describe the audience as only what the executable targeting encodes; personas or
interests used only to shape messaging are creative direction, not configured
targeting.

证据约束选择：缺失或失败的证据可以排除某个设置，但不能确立替代方案。目标、落地页与受众的推荐应来自明示意向、肯定性证据、或以假设形式展示的、有支撑的安全默认值；否则保持决策开放。执行绑定的 URL 与定向取值必须来自广告主或工具结果。只有当所选落地页所需的 URL 或对象身份已被提供或返回时，它对定价才是稳定的；绝不为假想的落地页报价。描述受众时只描述可执行定向所编码的内容；仅用于塑造信息的用户画像或兴趣属于创意方向，不是配置的定向。

An empty or failed summary establishes only that its lane supplied no usable
evidence. It does not prove the advertiser lacks history, assets, audiences,
demand, or prior performance unless an authoritative, correctly scoped
collection explicitly returns empty. Use the documented fallback read when a
missing fact could change a concrete decision.

空或失败的摘要只能确立其路径未提供可用证据。除非一个权威且范围正确的集合明确返回空，否则它不能证明广告主缺乏历史、资产、受众、需求或以往效果。当缺失的事实可能改变某个具体决策时，使用有文档记载的回退读取。

Choose objective and optimization only after destination and tracking facts,
then read and apply `campaign-delivery-compatibility.md` before either decision is
settled, priced, or shown as a recommendation. Never silently replace the
requested outcome. If its measurement event is unavailable, finish useful
non-pricing research and go directly to the binding-constraint checkpoint
before pricing an alternative.
Then apply compliance and resolve any creation-bound targeting. A country code
is already canonical. For any interest, language, or other location, read
`campaign-targeting.md` before the first lookup, then call
`ads_targeting_search` once with the grounded batch. A successful partial result
is final for that decision round; do not retry by varying spelling, type,
radius, or nearby areas without new advertiser evidence. Carry only returned
objects into pricing and creation. An unresolved requested place remains open:
do not price a partial audience, substitute a nearby place, widen its radius,
or omit geography from executable targeting. Price only after every requested
geography resolves or the advertiser accepts a revised geography.

只有在落地页与追踪事实之后才选择目标与优化，然后在任一决策被敲定、定价或作为推荐展示之前，阅读并应用 `campaign-delivery-compatibility.md`。绝不悄悄替换所请求的结果。如果其测量事件不可用，先完成有用的非定价研究，再直接进入约束性分歧检查点，然后才为替代方案定价。随后应用合规并解决任何创建绑定的定向。国家代码已是规范形式。对任何兴趣、语言或其他位置，在首次查询之前阅读 `campaign-targeting.md`，然后用有据的批次调用 `ads_targeting_search` 一次。成功的部分结果对该决策轮是最终结果；在没有新的广告主证据时，不要通过变换拼写、类型、半径或附近地区来重试。只把返回的对象带入定价与创建。未解析的请求地点保持开放：不要为部分受众定价、不要用附近地点替代、不要扩大半径、也不要把地域从可执行定向中省略。只有当每个请求地域都解析、或广告主接受修订后的地域之后才定价。

Evaluate whether each campaign or ad set earns its own budget; prefer
consolidation to unsupported fragmentation. After all pricing inputs,
identities, and required capabilities stabilize, price the whole structure once
under `campaign-budget.md`. An estimate that prompts an optimization change is
feasibility evidence, not final pricing. Any accepted input change invalidates
it; reprice only the final structure.

评估每个广告系列或广告组是否配得上独立预算；在缺乏支撑时优先合并而非碎片化。在所有定价输入、身份与所需能力稳定之后，按 `campaign-budget.md` 对整个结构定价一次。促使优化变更的估算属于可行性证据，不是最终定价。任何被接受的输入变更都会使其失效；只对最终结构重新定价。

Finally check coherence across all seven settings, including the complete
delivery tuple in `campaign-delivery-compatibility.md`. Repair
evidence-resolvable conflicts. If one genuinely advertiser-owned blocker
remains, finish available research, explain that blocker, and ask only for it
instead of presenting an approvable plan.

最后检查全部七项设置之间的一致性，包括 `campaign-delivery-compatibility.md` 中的完整投放元组。修复可用证据解决的冲突。如果仍剩一个真正属于广告主自有的阻塞点，完成可用研究，解释该阻塞点，并只就它提问，而不是呈现一个可批准的计划。

After research, end with exactly one controller state: a binding-choice widget,
one genuinely unbounded next question, or the complete strategy with embedded
approve/revise options—even for plan-only requests. Never stop at an
informational campaign or budget summary.

研究之后，以恰好一个控制器状态收尾：一个约束性选择组件、一个真正开放性的下一问、或内嵌批准/修订选项的完整策略——即使对仅规划请求也是如此。绝不停止在一个纯信息性的广告系列或预算摘要上。

## Resolve a binding constraint before full review / 完整评审之前解决约束性分歧

Before complete review, stop when evidence makes the request unexecutable or
creates a material outcome, optimization, target, or spend fork. Name the first
constraint correctly: an unavailable measurement event is tracking; a buildable
event whose projected volume or required spend conflicts with the advertiser's
target or limit is budget feasibility.

在完整评审之前，当证据表明请求无法执行、或产生实质性的结果、优化、目标或花费分歧时，先停下来。正确命名第一个约束：不可用的测量事件属于追踪；可构建的事件、其预估量或所需花费与广告主的目标或限额冲突，属于预算可行性。

Show only evidence → consequence → one recommendation, followed by one bounded
widget under the interaction contract. Keep the requested outcome as an option
when executable, name alternatives clearly, include a spend or target revision
when it resolves the constraint, and order the recommendation first. Do not
preview the complete plan or ask for strategy approval.

只展示 证据 → 后果 → 一个推荐，随后按交互契约附上一个有界组件。所请求结果可执行时保留为选项，清晰命名各替代方案，当花费或目标修订能解除约束时将其纳入，并把推荐排在第一位。不要预览完整计划，也不要请求策略批准。

The selection settles only the disputed path. Collect any required free-form
value next, rerun affected research, price the final stable structure once, then
show the complete strategy with its accepted learning tradeoff.

该选择只敲定有争议的路径。接着收集任何所需的自由格式取值，重跑受影响的研究，对最终稳定结构定价一次，然后展示连同其已被接受的学习权衡在内的完整策略。

## Recommend the complete plan / 推荐完整计划

Lead with the forward recommendation, not a research log. Choose one form:

以面向未来的推荐开头，而不是研究日志。选择一种形式：

- For a beginner, answer in plain business language: intended outcome and
  timing, destination, audience, structure/placements, budget, and expectation.
  对新手，用平实的商业语言回答：预期结果与时间、落地页、受众、结构/版位、预算与预期。
- For an experienced advertiser, use the seven delivery-setting labels above,
  explaining only unfamiliar or disputed choices.
  对有经验的广告主，使用上文七个投放设置标签，只解释不熟悉或有争议的选择。

For each setting, state the planned value and concise basis. Surface only two
to four findings that materially changed decisions, leading with first-party
evidence. Use sparse clickable links for any external claims and never expose
raw IDs. State only assumptions or missing evidence that materially bound the
recommendation. Do not include a creative plan, render `campaign-summary`,
prepare media, upload, or write Ads objects. Ask `Proceed with this strategy?`,
then show exactly:

对每项设置，陈述计划取值和简洁依据。只呈现二至四条实质改变了决策的发现，以第一方证据领衔。对外部论断使用少量可点击链接，绝不暴露原始 ID。只陈述实质性约束该推荐的假设或缺失证据。不要包含创意计划，不要渲染 `campaign-summary`，不要准备媒体、上传或写入 Ads 对象。询问 `Proceed with this strategy?`，然后精确展示：

- `Approve this strategy`
  `Approve this strategy`
- `No, make changes`
  `No, make changes`

Put the complete plan, approval question, and both options in one final response
under the interaction contract. Commentary may contain status only, never plan
settings or recommendations.
Call `muse.create_options` before writing the plan, then write the plan and
question with its returned `embed_token` alone on the final line; calling the
tool without embedding the token does not display the choice.

按交互契约，把完整计划、批准问题与两个选项放进同一个最终响应。Commentary 只能包含状态，绝不能包含计划设置或推荐。在写计划之前调用 `muse.create_options`，然后把计划和问题写出来，并把其返回的 `embed_token` 单独放在最后一行；调用工具而不嵌入 token 不会显示该选择。

The approval accepts every shown delivery value, including budget and
hierarchy, but no creative or write. After `No, make changes`, ask what to
revise. Retain unaffected evidence, rerun only affected lanes, reprice once only
if a pricing input changed, and present the complete revised strategy again.
On approval, do not rerun strategy research; route to `campaign-creative.md`.

该批准接受所有已展示的投放取值，包括预算与层级，但不包括创意或任何写操作。在 `No, make changes` 之后，询问要修订什么。保留未受影响的证据，只重跑受影响的路径，仅在定价输入变化时重新定价一次，然后再次呈现完整的修订策略。批准后，不要重跑策略研究；路由到 `campaign-creative.md`。

## Recommendation rules / 推荐规则

Campaigns separate objectives. Ad sets separate material delivery mechanics
such as budget, audience, geography, schedule, placement, or optimization. Ads
separate messages and creative concepts. Multiple ads are not a controlled A/B
test without an experiment. Do not fragment beyond what the priced budget can
support.

广告系列区分目标。广告组区分实质性的投放机制，如预算、受众、地域、排期、版位或优化。广告区分信息与创意概念。没有实验工具时，多支广告不构成受控 A/B 测试。不要碎片化到超出已定价预算所能支撑的程度。

Never invent an exact URL, spending constraint, offer, claim, or service
geography. Do not derive geography from account currency, timezone, Page
language or location, a destination domain, prior campaigns, or market
convention. It must come from the advertiser or verified service-area evidence;
otherwise ask one geography question before pricing. Do not ask about
evidence-backed defaults. Unless constrained, use
auction buying, campaign-level daily budgeting, broad Advantage+ Audience,
automatic placements, no end date, and compatible billing/optimization. Keep
native messaging destinations native. Name the executable placement choice
precisely: automatic placements means all surfaces eligible for the final
setup, while a platform-restricted set is not automatic.

绝不发明精确的 URL、花费约束、优惠、宣传用语或服务地域。不要从账户货币、时区、Page 语言或位置、落地页域名、先前广告系列或市场惯例推导地域。它必须来自广告主或经核实的服务区域证据；否则在定价前提出一个地域问题。不要就有证据支撑的默认值提问。在无约束时，使用竞价购买、广告系列级日预算、宽泛的 Advantage+ Audience、自动版位、无结束日期，以及兼容的计费/优化。原生消息类落地页保持原生。精确命名可执行的版位选择：自动版位指最终设置下所有合资格的版面，而平台受限集合不是自动版位。

**Bounded delivery is the exception to the no-end-date default.** That default
is for delivery meant to run on. When the advertiser's own goal ends — the unit
is let, the event happens, the offer closes — a daily budget with no end date
keeps spending past the thing it was for. Raise it before the complete review
and settle it as an **end date**, which is the only stop this surface can
actually set. A lifetime budget requires one either way.

**有界投放是无结束日期默认值的例外。**该默认值针对的是打算持续投放的广告。当广告主自己的目标已经终结——房屋已租出、活动已举办、优惠已截止——不带结束日期的日预算会在其目的之外继续花费。在完整评审之前提出这一点，并以**结束日期**敲定，这是该界面唯一能真正设置的停止方式。总预算（lifetime budget）则无论如何都需要一个结束日期。

**A stopping rule is not an end date, and must not be accepted as one.** "Pause
it once the unit is leased" names a condition nothing here watches: there is no
scheduled task, no trigger, and safety rule 5 forbids offering one. Writing it
into the plan and moving on leaves the campaign spending while the advertiser
believes it will stop by itself. If that is the shape they want, say plainly
that stopping is a manual step they take, and offer a date as a backstop so the
spend has a floor under it if they forget. Where they do mean it to run on,
leave the field out rather than inventing a date to fill it.

**停止规则不是结束日期，也不得被当作结束日期接受。**"租出去就暂停"命名的条件没有任何东西在监视：这里没有计划任务、没有触发器，安全规则 5 也禁止承诺提供。把它写进计划然后继续，会让广告系列继续花费，而广告主却以为它会自行停止。如果那正是他们想要的形态，直说停止是他们手动执行的一步，并提出一个日期作为兜底，这样即使他们忘了，花费也有一个下限。如果他们的确打算持续投放，就留空该字段，而不是编一个日期来填它。
【评论】这一组条款针对的是代理把用户的口语化停止意图（"租出去就停"）误实现为持续投放的风险，核心是把"条件式停止"这种系统并不支持的表达还原为人工操作或明确日期。

**Do not promise measurement the account cannot do.** Before telling the
advertiser you can optimise for or report applications, purchases or any other
offsite conversion, read the account's datasets with `ads_get_datasets`. With no
usable dataset, say so plainly and say what the campaign will measure
instead — a click-optimised campaign counts visits to the page, not completed
applications. That difference is often what decides between Traffic and a
conversion objective, so it belongs in the recommendation, not in a caveat
afterwards.

**不要承诺账户做不到的测量。**在告诉广告主你可以针对申请、购买或其他站外转化进行优化或报告之前，先用 `ads_get_datasets` 读取账户的数据集。没有可用数据集时，直说这一点，并说明广告系列将改为测量什么——以点击优化的广告系列统计的是页面访问，而不是完成的申请。这一差异常常正是 Traffic 与转化目标之间的决定因素，所以它应出现在推荐里，而不是事后的提醒中。

## Optional strategy artifact / 可选的策略工件

Create an unshared `artifact.create_web_static` strategy only when explicitly
requested or after the advertiser accepts a brief nonblocking offer prompted by
a material finding. Use the campaign name plus `campaign strategy`, the exact
request as `verbatim_request`, and `capabilities: {public_web_read: false,
connectors: []}`. Let the builder use parent-conversation evidence; do not force
a recap through `artifact.send_input`. It may explain evidence,
recommendations, assumptions, and links, but never raw IDs, signed URLs,
unsupported claims, or approval controls. It does not replace final review.

仅在明确请求时、或在广告主接受一个由实质性发现触发的简短非阻塞提议之后，才创建不共享的 `artifact.create_web_static` 策略工件。使用广告系列名称加 `campaign strategy`，把确切请求作为 `verbatim_request`，并使用 `capabilities: {public_web_read: false,
connectors: []}`。让构建器使用父对话证据；不要强迫通过 `artifact.send_input` 复述。它可以解释证据、推荐、假设与链接，但绝不能包含原始 ID、签名 URL、无支撑的论断或批准控件。它不替代最终评审。
