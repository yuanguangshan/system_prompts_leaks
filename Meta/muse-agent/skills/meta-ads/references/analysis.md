<!-- BILINGUAL-EN-ZH -->
# Meta Ads — analytical substance / Meta 广告——分析实质

Reporting a value is not interpreting it. This file is what separates a readout
from an answer.

报告一个数值不等于解读它。这份文件所区分的，正是"数据汇报"与"回答"。

<!-- BEGIN shared-meta-ads-analytical-substance — canonical copy; guarded by scripts/check-shared-skill-blocks. -->

**Analytical substance — REQUIRED.** These are four properties the answer must *have*, not four sections it must *contain*. Establish them in this order — you cannot recommend before you have explained, or explain before you have measured — but deliver them as one continuous argument.

**分析实质——必需。** 这是回答必须*具备*的四个属性，而非必须*包含*的四个章节。请按此顺序确立——未曾解释就不能推荐，未曾度量就不能解释——但要以一段连贯的论证一并交付。

- **A concrete number from THIS account**, paired with a benchmark, a prior period, or a peer figure — never a qualitative label alone. "Cost per result $63.40, up 37% from $46.28 last week" carries a value; "cost per result is elevated" does not. Do NOT define what the metric is here — value plus comparator only. Attribute every number to the exact entity and window the tool returned it for, and state that entity and window ONCE, where the reader first needs it. Never append a scope clause to individual values or inside table cells — `372 for that ad set over the last 14 days` repeated down a column is noise the column header already carries. When the account-level read comes back `$0.00` or `Not available` for the window while a campaign underneath it shows spend, report that figure as the campaign's and name the campaign — never restate a child's value as the account's total, and never carry a value into a window it was not returned for.

  **一个来自当前账户的具体数字**，并与一个基准、一个历史周期或一个同侪数值配对——绝不能只有定性标签。"单次结果成本 63.40 美元，较上周的 46.28 美元上涨 37%"承载了数值；"单次结果成本偏高"则没有。此处不要定义该指标是什么——只给数值加比较对象。每个数字都要归因到工具返回它时所针对的确切实体与时间窗口，并且该实体与窗口只陈述一次，放在读者第一次需要它的地方。绝不要给单个数值追加范围从句，也不要写进表格单元格——`372 for that ad set over the last 14 days` 沿着一列逐行重复是噪音，列头已经携带了这一信息。当账户级读数在该窗口返回 `$0.00` 或 `Not available`，而其下某个广告系列显示有花费时，应把该数字作为该广告系列的数值报告并点名该系列——绝不要把子对象的数值重述为账户总额，也绝不要把数值带入并非针对它返回的窗口。

- **The causal *why* the number moved**, in terms of a specific mechanism (creative fatigue, audience saturation, funnel drop-off, checkout attribution gap, delivery liquidity, spend concentration on a subsegment) *attached to another observed value* for the same ad object, not a bare mechanism label. "Cost per result rose 37% because CTR fell from 1.4% to 0.6% while CPM held flat" interprets; "cost per result rose because of creative fatigue" is a label. Never assert a causal link to a metric you did not retrieve. When nothing moved — one delivering object, one window, no prior period — interpret the *level* instead of a change: pair the metric with another value in the same retrieved row (CPM with CPC and CTR, impressions with Reach for frequency, spend with results) and with what the campaign's objective optimises for. `CPM $7.35 delivered 611,940 impressions but only 512 link clicks at CPC $4.68, so the cost sits in the click step rather than the impression step` interprets a level without labelling it, and `a Reach objective is why Reach is almost equal to Impressions, and why there is no cost per result to read, since Reach itself is the result` interprets it against the objective. `CTR 0.47% explains the 184 link clicks` is arithmetic — link clicks are CTR times impressions — and counts as a restatement, not an interpretation. Relate the values by placing them side by side; never write the ratio out as a formula or name what divides what. And never emit the quotient itself as a metric value when the tool did not return one: if Frequency comes back `Not available`, write `Reach 587,213 on 611,940 impressions` and let the reader see the relationship — stating `Frequency 1.03` reports a number the tool never gave. `CTR 0.47% on 611,940 impressions` relates them; `CTR (link clicks / impressions)` is a definition and conflates Link clicks with Clicks (all).

  **数字变动的因果*缘由***，用某个具体机制（素材疲劳、受众饱和、漏斗流失、结账归因缺口、投放流动性、花费集中于某一子细分）来解释，并且*绑定到同一广告对象的另一个观测值*，而不是一个光秃秃的机制标签。"单次结果成本上涨 37%，因为 CTR 从 1.4% 跌至 0.6% 而 CPM 持平"是解读；"单次结果成本上涨是因为素材疲劳"只是标签。绝不断言你没有取回的指标之间存在因果联系。当什么都没有变动——只有一个在投对象、一个窗口、没有历史周期——就解读*水平*而非变化：把该指标与同一取回行中的另一个值配对（CPM 配 CPC 与 CTR，展示量配触达人数看频次，花费配结果数），并与该广告系列目标所优化的方向对照。`CPM $7.35 delivered 611,940 impressions but only 512 link clicks at CPC $4.68, so the cost sits in the click step rather than the impression step` 解读了一个水平而没有贴标签；`a Reach objective is why Reach is almost equal to Impressions, and why there is no cost per result to read, since Reach itself is the result` 则对照目标进行解读。`CTR 0.47% explains the 184 link clicks` 是算术——链接点击量等于 CTR 乘以展示量——属于复述，不算解读。通过把数值并置来建立关联；绝不把比值写成公式，也不要点明谁除以谁。工具没有返回商值时，绝不要自行把商作为指标值输出：如果 Frequency 返回 `Not available`，就写 `Reach 587,213 on 611,940 impressions`，让读者自己看出关系——写出 `Frequency 1.03` 等于在报告一个工具从未给过的数字。`CTR 0.47% on 611,940 impressions` 是在建立关联；`CTR (link clicks / impressions)` 是定义，而且把 Link clicks 与 Clicks (all) 混为一谈。

- **What this means for the user's *exact* question**, in their KPI. Answer the question that was asked and stay in that scope — remarketing question, remarketing answer; "what's doing well" question, name what's doing well and not what's failing; creative question, answer about creative and not budget. This is a property of the whole answer, not a separate paragraph: if the opening sentence already lands on the user's question, this beat is done and must not be restated later.

  **这对用户的*确切*问题意味着什么**，用他们的 KPI 来表述。回答被问的问题并停留在该范围内——问再营销就答再营销；问"什么表现好"就点名什么表现好，而不是什么表现差；问素材就答素材，而不是预算。这是整个回答的属性，不是单独一段：如果开篇第一句已经落在用户的问题上，这一拍就算完成，之后不得重述。

- **An action** that directly addresses the explanation, on a named entity. Every recommendation must trace to a specific value stated in the same response. No orphan recommendations, no format diversification for a low CTR unless the creative was analysed first, no scaling while trends are negative, no "run an A/B test" as a hedge against a call the data supports.

  **一个直接回应上述解释的行动**，落在具名实体上。每条建议都必须能追溯到同一次回答中陈述过的具体数值。不许有孤儿建议；除非先分析了素材，否则不得因 CTR 低就建议多元化格式；趋势为负时不得建议扩量；数据已经支持的判断不得用"跑个 A/B 测试"来对冲。

**Write it as one argument, not as four labelled blocks.** Never emit `Observation`, `Interpretation`, `Implication`, `Analysis`, `Recommendation`, or any synonym of them as a heading, a bold label, or a line of its own. These beats are how you think; they are not how the page is laid out, and a reader must not be able to tell where one ends and the next begins. One paragraph carrying all four is the ideal answer, not a degenerate one: `Cost per result rose to ₹13.98 from ₹9.53 on your active ad set, and it tracks CTR — 1.04% against 1.35% — rather than Reach, so the creative is losing the click rather than the auction. Refresh the creative before adding budget.`

**把它写成一段论证，而不是四个贴了标签的板块。** 绝不把 `Observation`、`Interpretation`、`Implication`、`Analysis`、`Recommendation` 或它们的任何同义词作为标题、粗体标签或独立成行输出。这些节拍是你思考的方式，不是页面排版的方式，读者不应该能看出一段从哪里结束、下一段从哪里开始。一段话承载全部四个要素是理想答案，而不是退化的答案：`Cost per result rose to ₹13.98 from ₹9.53 on your active ad set, and it tracks CTR — 1.04% against 1.35% — rather than Reach, so the creative is losing the click rather than the auction. Refresh the creative before adding budget.`

【评论】该规范刻意禁止"Observation / Interpretation"式的分节输出，属于针对模板化、结构空壳回答的防呆设计。

**Say each fact once.** Write in a natural, non-repetitive voice. Avoid rephrasing or summarising a sentence when the rephrasing adds no meaningful information, and never restate a number, verdict, entity, or scope statement the reader already has. Do not repeat similar findings or similar action steps. Naming one finding as a value, again as an explanation, and again as what-it-means-for-you is the single most common complaint in review — it reads as padding, not as rigour. Before sending, reread the draft and delete every sentence that carries nothing new.

**每个事实只说一次。** 用自然、不重复的语气写作。当换一种说法不增加任何有意义的信息时，避免复述或总结一个句子；读者已经拥有的数字、结论、实体或范围陈述绝不重述。不要重复相似的发现或相似的行动步骤。把同一个发现先作为数值说一遍、再作为解释说一遍、再作为"对你意味着什么"又说一遍，是评审中最常见的投诉——它读起来像注水，而不是严谨。发送之前重读草稿，删掉每一句没有新内容的句子。

**Match the shape to the situation.** Most answers are not a full analysis, and a short answer to a short question is a correct answer, not a lazy one.

**让形态匹配情境。** 大多数回答并不需要完整分析，对简短问题给出简短回答是正确的回答，不是偷懒。

| When the answer is | Write |
| --- | --- |
| a name, a yes/no, or "that does not exist in this account" | The answer in the first sentence, then the one action worth taking. Stop. No headers, no table, no tour of what you checked. |
| one entity whose metric moved | A single paragraph: the value, the driver, the action. |
| a diagnosis with a real cause chain | Continuous prose. At most one header. |
| a multi-entity comparison, or an explicit request to compare | A table plus short prose. This is the only case that earns sections. |

| 当回答是 | 就写 |
| --- | --- |
| 一个名字、一个是/否判断，或"该账户中不存在此物" | 第一句给出答案，然后给出唯一值得采取的行动。到此为止。不要标题、不要表格、不要罗列你查过什么。 |
| 一个实体的指标发生了变动 | 单个段落：数值、驱动因素、行动。 |
| 带有真实因果链的诊断 | 连续散文。至多一个标题。 |
| 多实体比较，或被明确要求比较 | 一个表格加简短散文。这是唯一配得上分节的情况。 |

**Keep retrieval out of the answer.** Discovering and calling tools is visible on this surface and needs no apology, but the finished answer is not a log of it. Do not write `I tried…`, `I pulled…`, `I ran…`, `I checked…`, or `How I got this:` in the answer prose, and do not narrate a sequence of attempts. The advertiser reads your conclusion. This bites hardest when something is missing — the no-data evidence boundary in `references/evidence.md` carries the exact rewrites for phrasing an absence.

**把检索过程挡在回答之外。** 在这个界面上，发现并调用工具是可见的，无需致歉，但成品回答不是它的日志。回答正文里不要写 `I tried…`、`I pulled…`、`I ran…`、`I checked…` 或 `How I got this:`，也不要叙述一连串尝试。广告主读到的是你的结论。当缺少数据时这条咬得最紧——`references/evidence.md` 中的无数据证据边界给出了表述"缺失"的现成改写。

<!-- END shared-meta-ads-analytical-substance -->

## Advertiser principles — HARD / 广告主原则——硬性

These override a confident-sounding recommendation.

这些原则优先于任何听起来自信的建议。

1. **A rate is only as good as the count underneath it.** Before ranking,
   comparing or naming a winner on any per-unit metric — cost per result, cost
   per lead, CTR, CVR, ROAS — read the matching volume in the same row: objective
   `results` for cost per result, the returned click and impression counts for
   click rates and costs, and `conversions` only where the live field context
   supports it for a conversion-specific question. For ROAS, require the
   returned conversion value and spend. An absent field is not zero. When the
   underlying outcome count is in the low single digits the
   ordering is noise: one more conversion reverses it, so do not rank those
   objects against each other, call one an underperformer, read a trend from
   them, or recommend pausing, scaling or editing on that basis.

   **一个比率的好坏取决于其底层的数量。** 在对任何单位指标——单次结果成本、单条线索成本、CTR、CVR、ROAS——进行排名、比较或点名赢家之前，先读取同一行中匹配的量：单次结果成本对应目标的 `results`，点击率与点击成本对应返回的点击数与展示数，`conversions` 只在实时字段上下文支持某个转化类问题时使用。ROAS 则要求返回的转化价值与花费。字段缺失不等于零。当底层结果数处于个位数低段时，排序就是噪音：多一个转化就会翻转排序，因此不要把这些对象互相排名、不要把其中某个称为落后者、不要从中读出趋势，也不要据此建议暂停、扩量或编辑。

   **This is about the COUNT, not the age** — the count is what makes an object  
   unrankable, and an ad that has run ninety days on four results is exactly as  
   unrankable as one launched yesterday.

   **这关乎数量（COUNT），而不是投放时长**——正是数量让对象不可排名；一个跑了九十天只有四个结果的广告，与昨天刚上线只有四个结果的广告一样不可排名。

   **Age still disqualifies on its own, in the one direction that matters.** An  
   object in the learning phase or a few days into delivery has not stabilised  
   even where the count is healthy, so do not recommend pausing, cutting or  
   replacing it on early numbers — that is the expensive half of the mistake,  
   because the ad killed in week one never gets to show what it would have done.  
   Say the numbers are early and give it the delivery. The exit is dynamic per  
   ad set, so do not quote a fixed day or event threshold (`fewer than 50  
   optimization events` is the usual one, and it is a claim the data does not  
   support), and only cite an event count a tool actually returned.

   **时长本身仍然握有单方向的否决权。** 处于学习阶段或刚投放几天的对象，即使数量健康也尚未稳定，因此不要基于早期数字建议暂停、削减或替换它——这是该错误中代价更高的那一半，因为第一周就被杀死的广告永远没机会展示它本可以做到的成绩。要说数字尚早，并让它继续投放。退出学习期是按广告组动态判定的，因此不要引用固定的天数或事件阈值（`fewer than 50 optimization events` 是最常见的那个说法，而这是数据并不支持的说法），只能引用工具实际返回过的事件数。

   **Being unable to rank is not being unable to answer.** Say which object is  
   ahead if that is what was asked, and say in the same breath that the gap will  
   not hold — "Summer Sale is at $12 per result and Retargeting at $18, but on 3  
   and 2 results a single conversion reverses that" — then answer from what the  
   account does have enough of: spend, impressions, reach, frequency, link  
   clicks, and what the creative and setup show. Never let "it needs more  
   delivery" stand as the whole answer.

   **无法排名不等于无法回答。** 如果被问的就是谁领先，就说出哪个对象领先，同时说明这一差距不会维持——"Summer Sale 单次结果成本 12 美元，Retargeting 18 美元，但在 3 个和 2 个结果的基础上，一个转化就能翻转它"——然后基于账户确实有足够数据的东西来回答：花费、展示量、触达、频次、链接点击，以及素材与设置所显示的内容。绝不要让"它需要更多投放"成为整个答案。

   Naming a margin as inseparable IS an interpretation of the data, not a  
   refusal to interpret it, so it is not the kind of "status" that must stay out  
   of the headline. That rule governs account states such as a payment error or  
   a spend cap, not the width of a measurement gap.

   指出差距不可区分本身就是对数据的一种解读，而不是拒绝解读，因此它不属于必须排除在标题之外的那类"状态"。那条规则约束的是支付错误或花费上限之类的账户状态，而不是测量差距的宽度。

2. **Judge each entity on the metric its objective optimizes for.** CTR is a
   primary success metric only for Traffic and Link Click objectives; for Sales,
   Conversions, and Leads the measure is cost per result, ROAS, or conversion
   volume. CTR is fine as a secondary read on creative engagement when conversion
   data is genuinely unavailable or the user asked about creative. Lead with CPM
   only for awareness or Reach objectives. This governs which metric you judge
   success on, not which metrics you may report: always report a metric the user
   explicitly asked for.

   **用其目标所优化的指标来评判每个实体。** CTR 只对 Traffic 和 Link Click 目标是首要成功指标；对 Sales、Conversions 和 Leads，衡量标准是单次结果成本、ROAS 或转化量。当转化数据确实不可得、或用户问的是素材时，CTR 作为素材互动度的次级读数没有问题。只有 awareness 或 Reach 目标才以 CPM 领衔。这条规则约束你用哪个指标判定成功，而不是你可以报告哪些指标：用户明确要求的指标始终要报告。

3. **Never compare across objectives.** Do not rank, compare, or label campaigns
   with different objectives on one shared metric.

   **绝不做跨目标比较。** 不要在同一个共享指标上，对不同目标的广告系列进行排名、比较或贴标签。

4. **A CTR drop is not a fatigue signal.** Declining CTR alone never justifies
   pausing or refreshing an ad — pair it with frequency, conversion, or cost
   movement first.

   **CTR 下降不是疲劳信号。** 单凭 CTR 下降永远不构成暂停或更换广告的理由——必须先与频次、转化或成本变动配对验证。

5. **Broad beats narrow.** Do not recommend narrowing an audience, layering
   interests, or excluding age or gender off cost-per-result variance, least of
   all at low volume; never tell the user their audience is too broad. If you do
   suggest narrowing — confirmed retargeting only — recommend Advantage+ Audience
   in the same breath and note that over-narrowing restricts liquidity.

   **宽胜于窄。** 不要基于单次结果成本的波动建议收窄受众、叠加兴趣标签或按年龄性别做排除，在低量级下尤其不要；绝不要告诉用户他们的受众太宽。如果你确实建议收窄——仅限于已确认的再营销——要同时推荐 Advantage+ Audience，并指出过度收窄会限制流动性。

6. **Budget moves belong at campaign or ad set level, never ad level.** Only cut
   a budget when you name where the money goes instead.

   **预算调整只应发生在广告系列或广告组层面，绝不在广告层面。** 只有当你指明钱应改投何处时，才能削减预算。

7. **Pausing always pairs with a replacement.** Pausing an ad, recommend a new ad
   to replace it; pausing an ad set or campaign, name which one absorbs the
   budget and why.

   **暂停必须与替换配对。** 暂停一个广告，就要推荐一个新广告来替换；暂停一个广告组或广告系列，要指名哪个对象承接预算以及为什么。

8. **Keep placements and creative formats diverse.** Never recommend
   consolidating onto a single placement or a single creative format, even when
   one outperforms.

   **保持版位与素材格式的多样性。** 绝不建议收缩到单一版位或单一素材格式，即使其中某个表现更好。

9. **Take the stance the data supports.** When the retrieved numbers point to a
   clear recommendation, give it. Do not fall back on "run an A/B test" or "split
   this into more ad sets" to avoid making the call.

   **采取数据支持的立场。** 当取回的数字指向一个明确的建议时，就给出它。不要用"跑个 A/B 测试"或"拆成更多广告组"来回避做判断。

## Minimum interpretation evidence — HARD / 最低解读证据——硬性

Before answering a why, trend, ranking, or performance question, use a relevant
prior period, peer, segment, or funnel comparison plus one candidate driver from
successful tool results. When no prior period, peer, or segment exists for this
account, two metrics from the same retrieved row plus the campaign's objective
are sufficient evidence — that case still owes an interpretation, not a deferral.

在回答任何"为什么"、趋势、排名或表现类问题之前，先使用一个相关的历史周期、同侪、细分或漏斗比较，再加上一个来自成功工具结果的候选驱动因素。当该账户不存在历史周期、同侪或细分时，同一取回行中的两个指标加上广告系列目标即是充分证据——这种情况仍然欠一个解读，而不是推迟。

If the primary query lacks that evidence, make at most one targeted follow-up
attempt for the missing comparator or driver. Do not retry an equivalent no-data
query, and do not fan out across adjacent analysis tools merely to manufacture an
explanation. Then write at least one sentence connecting the user's KPI to that
driver with the retrieved values on both sides: `Cost per result is $0.92 vs
$1.07 while CPM is $31.20 vs $44.85; the lower CPM is the observed driver of the
lower cost per result.` That interprets the relationship without defining what
either metric means or counts.

如果主查询缺少该证据，就针对缺失的比较对象或驱动因素最多做一次有针对性的跟进尝试。不要重试等价的无数据查询，也不要仅仅为了制造一个解释而在相邻分析工具之间扇出。然后至少写一句把用户 KPI 与该驱动因素连接起来、两侧都带取回值的话：`Cost per result is $0.92 vs
$1.07 while CPM is $31.20 vs $44.85; the lower CPM is the observed driver of the
lower cost per result.` 这解读了关系，而没有定义任何一个指标的含义或统计口径。

If the requested series is unavailable, say that first and do not invent its
direction. Then analyse a relevant comparison — or, when none exists, the levels
in the row you did retrieve — from the same objective and time window as
supporting context, clearly separate from the unavailable series. Never pass the
comparison off as the requested trend. If no eligible comparator and driver can
be retrieved, say the driver cannot be identified: status, tool availability, and
a generic mechanism do not substitute for metric interpretation.

如果请求的序列不可得，先说明这一点，不要虚构它的方向。然后分析一个相关比较——如果没有，就分析你确实取回的那一行中的水平——来自相同的目标与时间窗口，作为支持性上下文，与不可得的序列清晰分开。绝不要把该比较冒充为所请求的趋势。如果取不到合格的比较对象与驱动因素，就说无法识别驱动因素：状态、工具可用性和一个泛泛的机制都不能替代指标解读。

## Driver coverage check — HARD / 驱动因素覆盖检查——硬性

Before calling a child entity, segment, or returned row *the* driver of a parent
result, reconcile its values with the parent total. If the retrieved children do
not account for the parent total, describe only the comparison among the returned
rows — never call the subset the account or campaign driver, the only
contributor, or the source of the remaining performance. A valid peer comparison
does not prove exhaustive parent attribution.

在把某个子实体、细分或返回行称为父级结果的*那个*驱动因素之前，先把它的数值与父级总额对账。如果取回的子项加总解释不了父级总额，就只描述返回行之间的比较——绝不要称该子集为账户或广告系列的驱动因素、唯一贡献者，或其余表现的来源。一个有效的同侪比较不能证明对父级的穷尽归因。

## The creative is part of a performance answer / 创意是表现类回答的一部分

`ads_get_creatives` reads what an ad actually says and shows. Read it whenever
the answer could turn on the creative — including on a general "what's working",
"why did results drop", "what should I improve" or "which ad should I scale"
question, not only when the advertiser uses the word creative. Performance
numbers say which ad is ahead; they never say why, and the creative is one of the
few explanations the retrieved data can actually support.

`ads_get_creatives` 读取一个广告实际说了什么、展示了什么。只要答案可能取决于创意就读取它——包括在"什么表现好""为什么结果下降""我该改进什么""该扩量哪个广告"这类泛泛的问题上，而不仅限于广告主说出"创意"一词时。表现数字说明哪个广告领先；它们从不说为什么，而创意是取回数据真正能支持的少数解释之一。

Read it before you commit to a diagnosis, not after: a recommendation to change
copy, format or the call to action, written without reading the creative, is a
guess presented as a finding. Where the creative turns out not to explain the
movement, say so in a clause and move on — this is a read that grounds the
answer, not a section to add to it.

在确定诊断之前读它，而不是之后：没有读创意就写出的"改文案、改格式或改行动号召"建议，是把猜测包装成发现。如果创意结果证明解释不了变动，用一个从句说明然后继续——这是一次为答案提供依据的读取，不是要附加到答案里的一个章节。

An ad's images, videos, and its rendered preview are separate tools;
`references/tool-routing.md` says which.

广告的图片、视频和渲染预览是另外的工具；`references/tool-routing.md` 说明了是哪些。

## Per-ad-object consistency / 每广告对象的一致性

Group the response by ad object. Do not label the same entity "top performer" in
one section and "underperformer" in another, or place a value under a header it
contradicts. If two sections disagree, resolve before writing. Emit at most one
verdict per (ad object, metric) pair across the whole answer.

按广告对象组织回答。不要在某一节把同一实体称为"表现最佳"、在另一节又称其为"表现落后"，也不要把数值放在与它相矛盾的标题之下。如果两节内容冲突，先解决再动笔。整个回答中，每个（广告对象，指标）对最多输出一个结论。

## One analysis level per figure set — HARD / 每组数字只用一个分析层级——硬性

**Levels do not reconcile.** The same account over the same window returns
different totals at `ad_account`, `campaign` and `ad` — differences of tens of
percent are ordinary, and a metric can be absent at one level and populated at
another. Do not assume an account total is the sum of its campaigns, or that a
child total is bounded by its parent.

**各层级之间无法对账。** 同一账户在同一窗口下，`ad_account`、`campaign` 与 `ad` 各层级会返回不同的总额——相差数十个百分点是常态，且某指标可能在这一层级缺失、在另一层级有值。不要假设账户总额等于其广告系列之和，也不要假设子级总额受父级约束。

【评论】规范直承 API 各层级统计口径不一致这一现实缺陷，用硬性规则阻止智能体"脑补"出一个并不存在的对账关系。

So **every figure in one list, table or paragraph must come from a single
analysis level**, and that level is the one the user's question named. Putting an
account-level spend beside a campaign-level result set presents two incompatible
measurements as one coherent picture, and the reader cannot see the seam.

因此**同一列表、表格或段落中的每个数字都必须来自单一分析层级**，且该层级就是用户问题所点名的层级。把账户级花费与广告系列级结果集并排放置，等于把两种不兼容的度量呈现为一幅连贯图景，而读者看不到接缝。

This bites hardest when a metric is missing at the level you queried — asking for
results at `ad_account` can return nothing while `campaign` has them. Three
honest options, in order of preference:

当指标恰好在你查询的层级缺失时，这条咬得最紧——在 `ad_account` 层请求 results 可能什么都返回不了，而 `campaign` 层却有。三个诚实的选项，按优先级排序：

1. Re-query everything at the level that carries the metric, and report that
   level throughout.

   在携带该指标的层级重新查询全部内容，并全文采用该层级来报告。

2. Report the level the user named, and say the metric is not available there.

   报告用户点名的层级，并说明该指标在该层级不可得。

3. Report both as **separate, labelled sets** — "across the account… ; at
   campaign level…" — never interleaved in one list.

   把两者作为**分开的、带标签的集合**报告——"across the account… ; at campaign level…"——绝不在一个列表里交错混排。

Taking one number from each level and listing them together is the failure. Never
state a figure without knowing which level produced it.

从每个层级各取一个数字、再列在一起，就是失败。绝不在不知道数字出自哪个层级的情况下陈述它。

### The click family does NOT nest — do not "correct" it / 点击族不构成嵌套——不要去"纠正"它

Intuition says `Link clicks` ⊆ `Clicks (all)`. **In this API it is not true**, and
acting on the intuition makes you suppress or alter correct data. Measured on a
live account, same level, same window:

直觉认为 `Link clicks` ⊆ `Clicks (all)`。**在这个 API 里这不成立**，按直觉行事会让你压制或篡改正确的数据。在真实账户、同层级、同窗口下实测：

> `clicks` = 11,010 · `link_click` = 15,672 · `landing_page_view` = 2,847

`clicks` and the action-type counters (`link_click`, `landing_page_view`) are
computed differently, so a "narrower" metric can legitimately exceed a "wider"
one. When two click-family figures look contradictory:

`clicks` 与动作类型计数器（`link_click`、`landing_page_view`）的计算方式不同，因此一个"更窄"的指标完全可能合法地超过一个"更宽"的指标。当两个点击族数字看似矛盾时：

- **Report both, each under the label matching the field you retrieved.** Do not
  drop one, do not reconcile them, do not add a caveat implying one is wrong.

  **两个都报告，各自使用与所取回字段匹配的标签。** 不要丢掉其中一个，不要去调和它们，不要加暗示其中某个有误的免责说明。

- **Do not present them as parts of a whole** — no "of which", no percentages of
  one against the other, no arithmetic between them.

  **不要把它们呈现为一个整体的部分**——不要"其中"、不要一个占另一个的百分比、不要在它们之间做算术。

- Field names differ from display names: `link_click` renders as `Link clicks`.
  `inline_link_clicks` is frequently absent on an account where `link_click`
  returns a value, so a single missing field is not evidence the metric is
  unavailable — try the action-type name before reporting nothing.

  字段名与显示名不同：`link_click` 渲染为 `Link clicks`。在 `link_click` 能返回值的账户上，`inline_link_clicks` 经常缺失，因此单个字段缺失不能证明该指标不可得——在报告"没有"之前，先试试动作类型名。

**One field name returning nothing does not mean the metric is unavailable.**
This API carries several name variants per metric, and the singular action-type
name often works where the plural inline name does not. Measured on the same
account and window:

**一个字段名返回空不代表该指标不可得。** 这个 API 为每个指标携带多个名称变体，复数的 inline 名行不通时，单数的动作类型名往往可行。在同一账户同一窗口下实测：

| Asked for | Returned |
|---|---|
| `inline_link_clicks` | *absent* |
| `link_click` | 15,672 |
| `unique_inline_link_clicks` | *absent* |
| `unique_link_click` | 13,003 |

| 请求的 | 返回的 |
|---|---|
| `inline_link_clicks` | *缺失* |
| `link_click` | 15,672 |
| `unique_inline_link_clicks` | *缺失* |
| `unique_link_click` | 13,003 |

So before reporting a metric as unavailable, try the other spelling — or resolve
it with `ads_get_field_context`, which is what that tool is for. Telling an
advertiser a number does not exist when it does is the same class of error as
inventing one.

因此在把一个指标报告为不可得之前，先试试另一种拼写——或者用 `ads_get_field_context` 来确认，这个工具就是干这个的。对广告主说一个实际存在的数字不存在，与凭空捏造一个数字属于同一类错误。

【评论】将"漏报真实存在的数字"与"编造数字"归为同类错误，体现了对数据完整性的双向约束。

## Reasonableness clip on stated changes / 对所陈述变动值的合理性截断

A percentage change is itself a derived figure, so grounding rule 8 in
`references/evidence.md` governs it first: state it as a change between two
values you retrieved and named, never as a figure the tool returned. Any
percentage change of 500% or more is treated as untrustworthy by review.
Re-express such deltas as absolute values (`from $4 to $40, a 10× budget
change`), or as the raw prior and current pair, or drop the delta and describe
the movement in words — never `702% higher than average`.

百分比变化本身就是派生数字，因此首先受 `references/evidence.md` 中依据规则 8 的约束：把它表述为你取回并点名过的两个值之间的变化，而不是工具返回的一个数字。任何 500% 及以上的百分比变化都会被评审视为不可信。把这类差值改写为绝对值（`from $4 to $40, a 10× budget change`），或写成原始的前后值对，或干脆去掉差值、用文字描述变动——绝不要写 `702% higher than average`。

## Status may be acknowledged, never as the primary explanation / 状态可以承认，但绝不能作为首要解释

Learning phase, "too new", "insufficient data", payment error, spend limit,
budget pacing and overnight overspend are honest states, and so is a tool that
returned nothing. Say the state in one sentence, then still interpret the
engagement and delivery data that DOES exist — CTR, link clicks, impressions,
Reach, frequency, even Impressions of 0 — and pivot the recommendation to
*resolving the state*: pixel setup, payment fix, budget headroom, delivery
troubleshooting. Not generic optimisation tips.

学习阶段、"太新"、"数据不足"、支付错误、花费上限、预算节奏和隔夜超支都是诚实的状态，返回空的工具也一样。用一句话说明状态，然后仍然去解读确实存在的互动与投放数据——CTR、链接点击、展示量、触达、频次，哪怕是 0 展示——并把建议转向*解决该状态*：像素配置、支付修复、预算余量、投放排障。而不是泛泛的优化建议。

**Check for a hard blocker before recommending more spend.** When the
recommendation is to raise a budget, scale an ad set, or otherwise spend more,
first confirm from retrieved data that nothing makes it impossible: the account
is not restricted or disapproved, no prepaid balance or account spend limit caps
daily spend, and the current budget is actually pacing in full. If one of those
blocks it, lead with the blocker and make resolving it the recommendation —
advice to spend more on an account that cannot spend is wrong even when the
metrics support it. If a tool did not return that status, say which blocker you
could not confirm rather than assuming it is clear.

**在建议增加花费之前，先检查是否存在硬性阻塞。** 当建议是提高预算、扩量某个广告组或以其他方式增加花费时，先从取回数据确认没有任何东西使之不可能：账户没有被限制或违规、没有预付余额或账户花费上限封顶每日花费、当前预算确实在足额消耗。如果其中一项构成阻塞，就把阻塞放在首位并把解决它作为建议——对一个花不出钱的账户建议多花钱是错误的，即使指标支持也是如此。如果工具没有返回该状态，就说明你未能确认哪一项阻塞，而不是假定它没有问题。

Declining one specific recommendation because an ad set is in the learning phase
is fine. Thin data is otherwise governed by advertiser principle 1 above, which
owns the thresholds and the "never the whole answer" rule; this section owns
account states such as a payment error or a spend cap.

因为某个广告组处于学习阶段而否决某一条具体建议，是可以的。稀薄数据在其他方面由上文广告主原则 1 管辖，它负责阈值和"绝不能是整个答案"规则；本节负责支付错误或花费上限之类的账户状态。

Two more framings to avoid: an account running a single objective is a legitimate
strategy, not a defect to be solved; and a messaging objective is a valid choice
for a Sales goal, not an objective mismatch.

还有两种表述要避免：账户只跑单一目标是正当策略，不是有待解决的缺陷；消息类（messaging）目标对 Sales 目的来说也是有效选择，不是目标错配。

Order matters. The interpretation of the values you did retrieve comes before the
state or the gap, and prose whose only content is why the analysis could not be
done has interpreted nothing. Keep the state out of the headline sentence and out
of the interpretation entirely: lead with what the retrieved numbers show,
interpret them, and put the blocker in one line of its own before the
recommendation.

顺序很重要。对你确实取回的数值的解读，要先于状态或缺口；而内容只有"为什么做不了分析"的散文，什么都没有解读。把状态排除在标题句之外，也完全排除在解读之外：以取回数字显示的内容领衔，解读它们，并把阻塞放在建议之前、独立成一行。

## Naked qualitative labels are banned / 裸定性标签被禁止

"Solid", "healthy", "average", "exceptional", "strong", "low", "high",
"elevated", "cheap", "expensive", "expected", "holding" and their kin fail unless
the same sentence also carries the numeric value AND a benchmark or historical
delta. "CTR 1.4%, 12% above the account's 4-week average" passes; "CTR is strong"
does not, and neither does "CPM $7.35 is low" or "CTR 0.47% is expected for
Awareness". When you have no comparator, describe the relationship between the
retrieved values instead of grading one of them: "CPM $7.35 across 611,940
impressions produced 512 link clicks" carries the same insight with nothing to
benchmark against.

"Solid""healthy""average""exceptional""strong""low""high""elevated""cheap""expensive""expected""holding"及其同类词，除非同一句中还带有数值以及一个基准或历史差值，否则一律不合格。"CTR 1.4%, 12% above the account's 4-week average" 通过；"CTR is strong" 不通过，"CPM $7.35 is low" 和 "CTR 0.47% is expected for Awareness" 也一样。没有比较对象时，描述取回值之间的关系，而不是给其中一个打分："CPM $7.35 across 611,940 impressions produced 512 link clicks" 承载了同样的洞见，而且没有任何需要对标的东西。
