<!-- BILINGUAL-EN-ZH -->
# Operational itinerary / 可执行行程

Use this guidance when a trip spans several days, the user is revising an
existing itinerary, consequential logistics or pass economics need checking,
or the plan will become an artifact. Early inspiration can stay lightweight.
Apply this rigor before calling a plan feasible.

当行程跨越数天、用户正在修订既有行程、需要核查关键后勤或通票经济账，或该计划将成为工件（artifact）时，使用本指南。早期灵感阶段可以保持轻量。在判定一个计划可行之前，先应用这里的严格标准。

## Keep one canonical plan / 维持唯一权威计划

Maintain one active plan for the trip. Do not make the user reconcile several
chat answers. Keep the minimum trip-level context that changes the result:

为该行程维护唯一一份活跃计划。不要让用户去核对多个聊天答案。保留会改变结果的最小行程级上下文：

- home, current location, trip bases, local time zones, and travel dates;
  居住地、当前位置、行程落脚点、当地时区和出行日期；
- travelers, child ages, mobility or accessibility needs, and documents when  
  relevant;
  同行旅客、儿童年龄、行动或无障碍需求，以及相关时的证件；
- budget and currency, pace, interests, dietary needs, must-dos, exclusions,
  and transport preferences; and
  预算与币种、节奏、兴趣、饮食需求、必做事项、排除项和交通偏好；以及
- fixed anchors, unresolved choices, confirmed commitments, and dependencies.
  固定锚点、未决选择、已确认的承诺及其依赖。

For each itinerary item, keep three axes separate:

对每个行程条目，把三个维度分开维护：

- decision: proposed, selected, or rejected;
  决策：已提议、已选定或已否决；
- operational evidence: unchecked, estimated, or verified, including source
  and retrieval time; and
  运营证据：未核查、已估算或已验证，包括来源与获取时间；以及
- provider state: not applicable, not checked, found, available, requested,
  held, waitlisted, confirmed, unknown, failed, or cancelled.
  供应商状态：不适用、未检查、已找到、可预订、已请求、已持有、已候补、已确认、未知、失败或已取消。

An item can be selected but not yet verified or available. Continued
conversation, a follow-up question, or silence does not change a proposed item
to selected. Record a selection only from the user's explicit choice or an
already-established delegation. Keep a rejected item out of the active plan.
Keep the record that the item was rejected.

一个条目可以是已选定但尚未验证或尚不可预订。继续对话、追问或沉默都不会把已提议的条目变为已选定。只有来自用户的明确选择或既有的授权委托才记录为选定。已否决的条目不得留在活跃计划中，但要保留它曾被否决的记录。

A request to build or recommend an itinerary authorizes arranging grounded
recommendations into a proposed schedule. Keep those items visibly tentative.
The request does not itself change them to selected.

构建或推荐行程的请求，仅授权把有依据的推荐整理为已提议的日程。这些条目必须保持明显的暂定状态。该请求本身不会把它们变为已选定。

## Maintain the responsibility checklist / 维护责任清单

For a multi-part trip or planning work that will continue across turns,
maintain one compact responsibility checklist inside the canonical plan.
Record every open item with these fields:

对于多部分的行程或将持续多轮的规划工作，在权威计划内维护一份紧凑的责任清单。用以下字段记录每个未决事项：

- **Outcome:** State the result that must become true.
  **成果：** 说明必须成为现实的结果。
- **Owner:** Record one of `planning`, `booking`, an applicable direct provider,
  or `the user`.
  **责任方：** 记录 `planning`、`booking`、某个适用的直连供应商或 `the user` 之一。
- **Status:** Record `investigating`, `needs decision`, `ready for booking`,
  `booking in progress`, `user action`, `blocked`, `booked`, `confirmed`,
  `completed`, or `dropped`.
  **状态：** 记录 `investigating`、`needs decision`、`ready for booking`、`booking in progress`、`user action`、`blocked`、`booked`、`confirmed`、`completed` 或 `dropped`。
- **Next action or decision:** State the next step that advances the outcome.
  **下一步行动或决策：** 说明推进该结果的下一步。
- **Evidence state:** Record `unchecked`, `estimated`, or `verified`.
  **证据状态：** 记录 `unchecked`、`estimated` 或 `verified`。
- **Material deadline:** Record an exact deadline, `unknown`, or `none`.
  `unknown` means a material deadline may apply but is not known. `none` means
  no material deadline applies.
  **关键截止期：** 记录确切的截止期、`unknown` 或 `none`。`unknown` 表示可能存在关键截止期但尚不清楚；`none` 表示不存在关键截止期。

Assign `planning` to research, feasibility, logistics, or recommendation work
that the available tools support. Assign `booking` or an applicable direct
provider only through the rules in
`/opt/hatch/skills/travel-planning/references/booking-handoff.md`. Assign the
user only to a decision, private input, approval, or external action that the
available tools cannot perform.

把 `planning` 分配给可用工具能支持的研究、可行性、后勤或推荐工作。`booking` 或适用的直连供应商只能通过 `/opt/hatch/skills/travel-planning/references/booking-handoff.md` 中的规则分配。用户只承担可用工具无法完成的决策、私密输入、批准或外部行动。

Record `dropped` only after the user explicitly removes the outcome from the
trip. Do not declare the trip complete while a required item is `investigating`,
`needs decision`, `ready for booking`, `booking in progress`, `user action`, or  
`blocked`.

只有在用户明确把该成果从行程中移除后才记录 `dropped`。只要某个必需事项仍处于 `investigating`、`needs decision`、`ready for booking`、`booking in progress`、`user action` 或  
`blocked` 状态，就不要宣布行程已完成。

When a live text channel is available, present the checklist as a compact
Markdown trip-status table after the initial trip shape, after a material
change, or when the user asks what remains. Do not repeat it after minor
exchanges. Use at most these user-facing columns:

当有实时文本通道可用时，在初步行程成形后、发生实质变化后、或用户询问还剩什么事项时，把清单呈现为紧凑的 Markdown 行程状态表。轻微交流后不要重复它。面向用户的列至多使用：

| Component | Current plan | Status | Next step |
|---|---|---|---|

| 组成部分 | 当前计划 | 状态 | 下一步 |
|---|---|---|---|

Translate the richer internal owner, evidence, and deadline fields into those
columns without exposing internal workflow vocabulary. On a voice-only surface,
use a short spoken summary instead.

把更丰富的内部责任方、证据和截止期字段转译进这些列，同时不暴露内部工作流词汇。在纯语音界面上，改用简短的口播摘要。

This status view is ordinary chat content, not an artifact. Do not create or
update an artifact to keep the dashboard current during planning. Create the
single durable itinerary artifact only at the end of a coherent planning
session after the user asks for or accepts it, unless the user explicitly asks
for the artifact earlier.

这一状态视图是普通聊天内容，不是工件。规划期间不要为保持仪表盘最新而创建或更新工件。只有在一段完整规划会话结束且用户要求或接受时，才创建唯一一份持久行程工件，除非用户更早明确要求工件。

Treat a confirmed commitment as locked. Arrange the plan around a confirmed
commitment. Change or cancel a confirmed commitment only through Booking, or
through the applicable direct provider when Booking is unavailable, under the
same authorization and verification rules. When the user reports an existing
commitment but supplier evidence is not available, preserve it as a locked
dependency. Do not claim verified supplier confirmation for it. Do not
downgrade, duplicate, or silently rebook it.

把已确认的承诺视为锁定。围绕已确认的承诺安排计划。只有在相同的授权与验证规则下，通过 Booking、或在 Booking 不可用时通过适用的直连供应商，才能更改或取消已确认的承诺。当用户报告了既有承诺但拿不到供应商证据时，把它保留为锁定的依赖。不要声称已获得经验证的供应商确认。不要降级、复制或静默改订它。

## Revise without drift / 修订而不漂移

Apply an edit to the canonical plan rather than producing a competing full
version. Retain a concise change history with prior values so the user can
compare revisions or undo a planning edit without treating an older copy as
the active plan:

对权威计划应用编辑，而不是生成另一个相互竞争的完整版本。保留带先前值的简要变更历史，让用户可以比较各次修订或撤销一次规划编辑，而不把旧副本当成活跃计划：

- **Move:** retain the item's identity and decision state. Recheck hours,
  transit, tickets, and every downstream dependency affected by the new time.
  **移动：** 保留该条目的身份和决策状态。重新核查受新时间影响的开放时间、交通、票务及每个下游依赖。
- **Replace:** keep the old item rejected or removed as requested. The
  replacement starts proposed unless the user selected it explicitly.
  **替换：** 按请求把旧条目保持为已否决或已移除。除非用户明确选定了替换项，否则它以已提议状态起步。
- **Remove:** take only that item out of the active plan. Retain it in revision
  history for undo. Identify any resulting gap or broken dependency.
  **移除：** 只把该条目移出活跃计划。在修订历史中保留它以便撤销。识别由此产生的空档或断裂依赖。
- **Undo:** restore the preceding planning state. Do not use undo to reverse a
  real reservation, purchase, cancellation, or message.
  **撤销：** 恢复前一个规划状态。不要用撤销去反转真实的预订、购买、取消或消息。

Preserve unrelated days, constraints, rejections, and confirmed items. Do not
add a restaurant, activity, city, or detour merely to make the revised plan
look complete. After a material edit, summarize what moved, was added or
removed, and what now needs rechecking. Compare full versions only when the
user asks or the delta is otherwise hard to understand.

保留无关的日期、约束、否决项和已确认条目。不要仅仅为了让修订后的计划显得完整而添加餐厅、活动、城市或绕路。实质性编辑之后，概述移动了什么、新增或移除了什么、以及现在需要重新核查什么。只有在用户要求或差异难以理解时才比较完整版本。

When responsibility must survive the current conversation, reuse one relevant
durable owner. Do not create parallel trip records. A visible itinerary
artifact is a user deliverable. Do not treat that artifact as the only copy of
private working state. Read `/opt/hatch/skills/booking/SKILL.md` only when the
user needs live availability, exact commercial terms, or a transaction.

当责任必须跨越当前对话存续时，复用唯一一个相关的持久责任方。不要创建并行的行程记录。可见的行程工件是交付给用户的成果。不要把该工件当作私有工作状态的唯一副本。只有当用户需要实时可订状态、确切商业条款或一笔交易时，才读取 `/opt/hatch/skills/booking/SKILL.md`。

## Build a usable day / 构建可用的一天

Resolve the exact local dates, including year, and the relevant overnight base
or arrival and departure geography before calling a day or route feasible. If
one is missing, provide a useful conditional sketch from the facts that remain.
Ask for only the smallest blocking detail. Label the unresolved dependency. Do
not claim the schedule works.

在判定某一天或某条路线可行之前，先确定包括年份在内的确切当地日期，以及相关的过夜落脚点或到达/离开地理信息。如果缺失，就用现有事实给出有用的条件式草图。只询问最小的阻塞细节。标注未解决的依赖。不要声称该日程可行。

For each working day, establish a plausible local-time sequence. Arrange
selected items and proposed recommendations without conflating their states.
Include only details that make the experience usable:

对每个运作日，建立合理的当地时序。安排已选定条目和已提议推荐时不混淆两者的状态。只包含让体验可用的细节：

- time or honest window, place and locality, expected duration;
  时间或诚实的时间窗、地点与位置、预期时长；
- travel mode and current travel-time evidence from the preceding stop;
  交通方式以及来自上一站的当前行程时间证据；
- a realistic transfer, queue, check-in, or recovery buffer;
  现实的换乘、排队、入住或恢复缓冲；
- meal, rest, medication, or child-schedule windows when they constrain the  
  day;
  在其约束当天时的用餐、休息、服药或儿童作息时间窗；
- known cost and reservation state;
  已知成本与预订状态；
- a short reason the item fits; and
  该条目合适的简短理由；以及
- a backup when weather, scarcity, closure, or a tight connection makes the
  anchor fragile.
  当天气、稀缺、关闭或紧凑衔接使锚点脆弱时的备选方案。

A backup is a proposed contingency. Do not treat a backup as a second selected
anchor unless the user chooses it or delegates that choice.

备选是已提议的应急方案。除非用户选择它或委托做出该选择，否则不要把备选当作第二个已选定锚点。

Cluster nearby activities when it improves the day. Check opening days and
hours, last admission or service, ticket windows, arrival and departure times,
lodging check-in, and transport connections. Use a currently advertised route
or transit capability when one is available. Otherwise use a current operator
or official website and label remaining timing uncertainty. Do not present an
estimated travel time as checked.

当聚集邻近活动能改善当天体验时就聚集。核查开放日与开放时间、最后入场或服务时间、售票窗口、到达与出发时间、住宿入住以及交通衔接。有当前公布的路线或交通能力时使用之。否则使用当前运营商或官方网站，并标注剩余的时间不确定性。不要把估算的行程时间呈现为已核查。

A future travel duration returned by a route tool or operator is still an
estimate unless it is an exact scheduled service time. Label its source,
retrieval time, and material assumptions. Recheck volatile facts at the live
booking handoff and near the date of use, especially when the operator has not
yet published the relevant seasonal schedule.

路线工具或运营商返回的未来行程时长仍是估算，除非它是精确的时刻表服务时间。标注其来源、获取时间和关键假设。在实时预订交接时和临近使用日期时重新核查易变事实，尤其是当运营商尚未发布相关季节性时刻表时。

Carry party constraints through every day. Do not apply them only in the
introduction. Check age restrictions and child pricing, stroller or mobility
practicality, walking and transfer load, rest needs, meal timing, and the
user's stated pace. Flag an overloaded day. Offer the smallest useful
adjustment. Do not compress activities into impossible timings.

把同行人员约束贯穿到每一天，不要只在开头应用。核查年龄限制与儿童票价、婴儿车或行动便利性、步行与换乘负荷、休息需求、用餐时间以及用户声明的节奏。标记过载的一天。提供最小的有用调整。不要把活动压缩进不可能的时间安排。

## Ground consequential facts / 落实关键事实

Apply `/opt/hatch/skills/travel-planning/references/travel-fact-verification.md`
before the numbered source list in this section when
`/opt/hatch/skills/travel-planning/SKILL.md` directs it.

当 `/opt/hatch/skills/travel-planning/SKILL.md` 有此指示时，先应用 `/opt/hatch/skills/travel-planning/references/travel-fact-verification.md`，再使用本节的编号来源清单。

Use each source for what it can prove:

按各来源能证明的内容使用它们：

1. Supplier confirmations prove existing commitments.
   供应商确认能证明既有承诺。
2. A live booking provider proves current inventory and commercial terms.
   实时预订供应商能证明当前库存和商业条款。
3. An official venue, event, operator, or government source supports identity,
   operation, schedules, closures, rules, and published products or prices.
   官方场馆、活动、运营商或政府来源能支持身份、运营、时刻表、关闭信息、规则以及已发布的产品或价格。
4. Place search, reputable guides, reviews, and social posts support discovery
   and qualitative fit. Verify an operational claim from those sources against
   another source.
   地点搜索、可靠指南、评论和社交帖子能支持发现与定性契合。来自这些来源的运营性主张需用另一来源验证。

Resolve the exact place, event, beach, station, or experience before naming it
in the active plan. Record the source identity, its URL when one exists, and
the retrieval time for facts whose failure would change the itinerary. Keep a
checked fact distinct from an estimate and from live availability.

在活跃计划中点名某个地点、活动、海滩、车站或体验之前，先解析出确切对象。对一旦有误就会改变行程的事实，记录来源身份、其 URL（如存在）以及获取时间。把已核查的事实与估算、与实时可订状态区分开。

Open each consequential booking or information link. Verify that it reaches
the intended official item, date, or product. A dead or stale link is a source
failure. An unexpected redirect to mismatched content is a source failure. On
a source failure, find the current official path or mark the item unverified.
A normal canonical or locale redirect to the intended content is acceptable.
Link failure is not evidence that the item is closed, sold out, or nonexistent.
Likewise, an accessible page is not evidence that dated inventory is available.

打开每个关键的预订或信息链接。验证它确实到达预期的官方条目、日期或产品。失效或过期的链接是来源失败。意外重定向到不匹配内容是来源失败。发生来源失败时，找到当前的官方路径，或将该条目标记为未验证。到预期内容的正常规范化或地区重定向是可以接受的。链接失败不是该条目已关闭、售罄或不存在的证据。同样，页面可访问也不是特定日期库存可用的证据。

An official published admission, fare, or pass price is valid planning
evidence. Date-specific inventory, mandatory checkout fees, refundability, and
the final payable total are live commercial truth for Booking to establish.

官方公布的门票、票价或通票价格是有效的规划证据。特定日期的库存、强制结算附加费、可退款性以及最终应付总额，是由 Booking 确立的实时商业真相。

When sources disagree, prefer the source that directly owns the fact. State the
unresolved discrepancy when you cannot reconcile it. Do not fill a gap with a
plausible venue, event, schedule, price, or link.

当来源相互矛盾时，优先使用直接拥有该事实的来源。无法调和时，说明未解决的分歧。不要用一个看似合理的场馆、活动、时刻表、价格或链接来填补空缺。

## Calculate trip economics transparently / 透明计算行程经济账

For a pass, bundle, or material cost comparison, verify the exact current
product before calculating. Record:

对于通票、套餐或实质性成本比较，先验证确切的当前产品再计算。记录：

- product name, duration, validity pattern, class or zone, traveler eligibility,
  and current official price;
  产品名称、时长、有效期模式、舱位或区域、旅客资格及当前官方价格；
- each itinerary segment or admission, its ordinary price, and whether the
  product fully covers it, discounts it, requires a reservation or supplement,
  or does not cover it;
  每个行程段或入场项、其常规价格，以及该产品是全额覆盖、折扣覆盖、需要预订或补票、还是不覆盖；
- adult and child quantities and rules;
  成人与儿童数量及规则；
- source currency, taxes or fees when known, and unknowns;
  来源币种、已知的税费以及未知项；
- the exchange-rate source and date when conversion is necessary; and
  需要换算时的汇率来源与日期；以及
- the arithmetic for point-to-point total, product total, uncovered extras,
  and resulting difference.
  点对点总额、产品总额、未覆盖附加项及最终差额的算术过程。

Do not interpolate an unlisted duration, infer coverage from a product name,
or call a pass cheaper when required fares, supplements, eligibility, or
prices are missing. When the evidence cannot support a savings number, show
the known components. State what remains unresolved.

不要内插未列出的时长，不要从产品名称推断覆盖范围，也不要在缺少必需票价、附加费、资格或价格时声称通票更便宜。当证据无法支持节省金额时，展示已知组成部分，并说明尚未解决的部分。

## Produce one useful artifact after planning / 规划后产出一份有用的工件

After the plan reaches a coherent stopping point, when the user asks for or
accepts an itinerary artifact, create or update one canonical artifact rather
than creating a new copy after every revision. If the user explicitly asks for
an artifact earlier, honor that request. A useful artifact supports the whole
plan or a user-selected subset and includes:

当计划达到一个连贯的停止点，且用户要求或接受行程工件时，创建或更新唯一一份权威工件，而不是每次修订后都创建新副本。如果用户更早明确要求工件，则满足该请求。有用的工件支持整个计划或用户选定的子集，并包含：

- a chronological day-by-day view with local times, transit, buffers, costs,
  reservation state, rationale, and backups;
  按时间排列的逐日视图，含当地时间、交通、缓冲、成本、预订状态、理由和备选；
- one exact, trustworthy photo for each recommended physical place when
  available, following the "Make place recommendations visual" rules in  
  `/opt/hatch/skills/travel-planning/references/planning-kickoff.md`;
  在可用时为每个推荐的实体地点配一张精确、可信的照片，遵循 `/opt/hatch/skills/travel-planning/references/planning-kickoff.md` 中的“让地点推荐可视化”规则；
- official, source, or reservation links when available and freshness for
  consequential facts;
  可用时的官方、来源或预订链接，以及关键事实的时效信息；
- confirmed details kept visually distinct from selected and unresolved items;
  已确认细节在视觉上与已选定和未解决条目区分开；
- a concise change summary and a visible list of what is still open; and
  简明的变更摘要以及仍然未决事项的可见清单；以及
- a map or map links when geography materially helps and locations are
  verified, with a usable list fallback.
  当地理信息实质上有帮助且位置已验证时的地图或地图链接，并提供可用的清单回退。

Update the canonical artifact for later changes. Follow
`/opt/hatch/skills/booking/references/presentation.md` when presenting booking
options, a final review, or a confirmation. For artifact photos, follow
`image_search`'s artifact ingestion and preflight contract. Do not persist a
transient CDN thumbnail as the durable copy. Keep the artifact private by
default. Publish the artifact, update a published copy, or send it to another
person only after a separate explicit request and the applicable approval. When
recipient-scoped private sharing is not available, tell the user. Offer a
downloadable file the user can share privately. Do not describe a public link
as private.

后续变更要更新权威工件。在展示预订选项、最终评审或确认时，遵循 `/opt/hatch/skills/booking/references/presentation.md`。工件照片遵循 `image_search` 的工件摄取与预检契约。不要把瞬态 CDN 缩略图持久化为持久副本。工件默认保持私密。只有在单独的明确请求和适用的批准之后，才能发布工件、更新已发布副本或将其发送给他人。当无法进行按接收人限定的私密共享时，告知用户。提供用户可私下分享的可下载文件。不要把公开链接描述为私密的。

【评论】该文档把每个行程条目拆为决策、证据、供应商状态三个独立维度，本质上是在无状态的对话之上强制实现一套状态机，防止“聊过即视为确认”或“估算被当作已核实”。
