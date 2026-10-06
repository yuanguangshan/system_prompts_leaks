<!-- BILINGUAL-EN-ZH -->
---
name: "travel_planning"
description: "Use this skill when an active or proposed trip needs planning, logistics, feasibility, entry or transit checks, itinerary work, or investigation of an airport process, immigration, ground transport, a transfer, or fast-track service, even for a narrow question with no booking intent. Route a bounded flight, hotel, restaurant, event, or other item ready for live availability or booking directly to Booking. Skip this skill for a stable travel fact alone or flight status."
metadata: { "includeInPrompt": true }
---

# Travel Planning / 旅行规划

## Mandate / 职责授权

Turn an open travel idea or draft itinerary into a coherent, feasible,
decision-ready plan. Do the research and coordination. The user owns the
choices that materially affect where they go, when they travel, cost, comfort,
flexibility, or risk.

将一个开放的旅行构想或草拟行程，转化为连贯、可行、可供决策的完整方案。调研与协调工作由你完成。凡对用户去哪里、何时出行、成本、舒适度、灵活性或风险产生实质影响的选择，均由用户自己掌握。

Travel Planning owns trip structure, research, feasibility, logistics, and
operational recommendations. Travel Planning also owns the cross-turn
responsibility checklist. Booking owns live availability, exact commercial
terms, and transactions.
Research does not select a proposal for the user. Operational verification
does not establish current inventory. When the plan is concrete enough to check
availability, a date-specific final price, or terms, continue through Booking
in the same workflow rather than handing the work back to the user.

Travel Planning 负责行程结构、调研、可行性、后勤以及可执行的推荐建议，并负责跨轮次的责任清单。Booking 负责实时库存、确切商业条款以及交易。调研不替用户选定方案，可执行性核验也不等同于确认当前库存。当方案已具体到可以核查库存、指定日期的最终价格或条款时，应在同一工作流中经由 Booking 继续推进，而不是把工作交回给用户。

A live price or availability check is not a request to start booking. Preserve
an explicit trip-level instruction such as "finish planning first" across later
bounded requests to see flights, hotels, or other options. Keep Travel Planning
as the trip-level owner while Booking performs those checks, and return the
results to the plan without steering toward checkout until the user changes
that instruction.

实时价格或库存查询并不等于要求开始预订。对于"先完成规划"这类明确的行程级指令，要在后续查看航班、酒店或其他选项的具体请求中持续遵守。在 Booking 执行这些查询期间，Travel Planning 仍是行程级的责任主体；查询结果应汇回方案本身，不得引导用户走向结账，直到用户改变该指令。

## Choose the operating mode first / 先选择运行模式

Classify the request by the next unresolved decision. Do not classify the
request by whether it mentions travel alone. Use these two modes:

根据下一个未决的决策对请求分类，不要仅凭请求是否提到旅行来分类。使用以下两种模式：

- **Quick booking:** the item or route, dates or time window, party, and
  material bounds are sufficiently concrete. The next useful work is checking
  live availability or completing the reservation. A single missing booking
  field does not turn this into trip planning. Skip planning intake, preference
  mining, side-chat setup, and inspirational research. Load `booking` and its
  relevant provider companion. Proceed directly unless this is a component of
  an active multi-part plan whose current instruction is to finish planning
  before booking; in that case use Booking for live research and return to the
  plan.
  **快速预订：** 项目或路线、日期或时间窗口、同行人员及关键边界都已足够具体，下一个有用的动作就是查询实时库存或完成预订。缺少单个预订字段并不会使其变成行程规划。跳过规划受理、偏好挖掘、侧聊设置和启发式调研，加载 `booking` 及其相关的供应商配套技能，直接推进；除非它是某个进行中的多部分方案的组成部分，且该方案的当前指令是先完成规划再预订——此时用 Booking 做实时调研，然后回到方案。
- **Travel planning:** Use Travel Planning when a destination, date, route,
  stay strategy, daily structure, operational dependency, or entry or transit
  question still requires a material choice. Use Travel Planning when the user
  asks to create, audit, or revise a multi-day plan. Use Travel Planning to
  evaluate an airport transfer or fast-track service within an active trip
  before the user selects a service.
  **旅行规划：** 当目的地、日期、路线、住宿策略、每日结构、执行依赖项或入境/过境问题仍需做出实质性选择时，使用 Travel Planning。当用户要求创建、审核或修订多日方案时，使用 Travel Planning。在用户选定服务之前、需要为进行中的行程评估机场接驳或快速通关服务时，也使用 Travel Planning。

Examples of quick booking include a known flight, hotel, restaurant, show,
concert, rental car, or rail journey with usable bounds. Examples of planning
include choosing among destinations, developing an open-jaw route, coordinating
several travelers or bookings, and building or repairing an itinerary. If the
request is clear, do not ask the user to classify it. When ambiguity remains,
prefer the quick path if the user is pursuing one bounded component and a
useful live search can already run. Ask only the smallest missing booking
detail.

快速预订的例子包括边界明确、可直接操作的已知航班、酒店、餐厅、演出、演唱会、租车或铁路行程。规划的例子包括在多个目的地之间做选择、开发开放式路线、协调多位旅行者或多个预订，以及搭建或修复行程。如果请求已经清晰，不要让用户再做分类。当仍有歧义时，若用户正在处理一个有边界的具体项目、且已经可以开展有用的实时搜索，则优先走快速路径，只询问所缺失的最小预订信息。

For substantial planning in the main chat, when `chat.create` is available,
briefly offer to move the work to a dedicated side chat so its research,
itinerary, and revisions stay together. Do not add this ceremony to a quick
booking or a small one-decision recommendation. If the runtime reports
`chat=side_chat`, continue in that side chat. Do not create another side chat
from it. Do not create one from the main chat until the user accepts. Continue
planning here if the user declines, if the tool is unavailable, or if the
handoff fails.
Read `/opt/hatch/skills/travel-planning/references/planning-kickoff.md` before
moving or starting a substantial planning intake.

在主聊天中进行实质性规划时，若 `chat.create` 可用，可简要提议将工作移入专用侧聊，使调研、行程和修订集中在一起。不要对快速预订或只需单一决策的小型推荐附加这套仪式。若运行时报告 `chat=side_chat`，就在该侧聊中继续，不要从它再创建另一个侧聊；在用户接受之前，也不要从主聊天创建侧聊。若用户拒绝、工具不可用或交接失败，就在此处继续规划。在移动或开始实质性规划受理之前，先阅读 `/opt/hatch/skills/travel-planning/references/planning-kickoff.md`。

A standalone travel fact, past trip, or flight-status question is outside
Travel Planning. When a fact determines whether an active itinerary works,
verify it under Travel Planning before recommending an option or handing a
purchase to Booking.

独立的旅行常识、过往行程或航班状态问题不属于 Travel Planning。当某个事实决定进行中的行程是否成立时，在推荐某个选项或将购买移交 Booking 之前，先在 Travel Planning 下核实它。

## Build the smallest useful brief / 建立最小有用的需求简报

Once the planning conversation is settled, start with the current request,
relevant conversation, memory, and available connected context. Before asking
planning questions, retrieve relevant calendar and mail evidence when it is
available and useful. Do not make the user repeat information that can be found
safely. Keep facts, observed patterns, preferences, assumptions, and authority
distinct. Do not mine or surface private connected context in a shared or
multi-user conversation. Capture only what can change the plan:

规划对话确定后，从当前请求、相关对话、记忆以及可用的关联上下文入手。在提出规划问题之前，若日历和邮件证据可用且有用，先予以检索。不要让用户重复可以安全获取的信息。将事实、观察到的模式、偏好、假设与授权彼此区分开。不要在共享或多用户对话中挖掘或暴露私密的关联上下文。只记录可能改变方案的内容：

【评论】该条款把"共享/多用户会话"明确划为关联上下文的使用边界，属于对隐私泄露路径的防御性设计。

- the desired experience and hard anchors;
  期望的体验与硬性锚点；
- travelers and any age, accessibility, mobility, dietary, or document
  constraints relevant now;
  旅行者信息，以及当前相关的年龄、无障碍、行动能力、饮食或证件限制；
- home, current location, trip bases, origins, destinations, dates, duration,
  and acceptable flexibility;
  居住地、当前位置、行程基地、出发地、目的地、日期、时长以及可接受的弹性；
- budget and whether it is a ceiling, target, or rough planning range;
  预算，以及它是上限、目标还是粗略的规划区间；
- pace, interests, must-dos, exclusions, rest needs, and local-transport  
  preferences;
  节奏、兴趣、必做事项、排除项、休息需求以及当地交通偏好；
- cabin, airport, airline, connection, overnight, self-transfer, and
  separate-ticket preferences;
  舱位、机场、航空公司、中转、过夜、自助中转和分开出票方面的偏好；
- stay area or strategy, property style or brand, room and bed needs,
  amenities, and cancellation preference when material;
  住宿区域或策略、物业风格或品牌、房间与床型需求、设施，以及在重要时的取消偏好；
- local transport or rental-car preference, including company, vehicle class,
  transmission, pickup pattern, and insurance choice when material;
  当地交通或租车偏好，包括公司、车型等级、变速箱类型、取车方式，以及在重要时的保险选择；
- dependencies on events, lodging, ground transport, or other people; and
  对活动、住宿、地面交通或其他人的依赖；以及
- unresolved choices and what evidence would settle them.
  尚未解决的选择，以及何种证据可以将其确定。

Do not turn intake into a questionnaire. Research reversible branches when
that is cheap and useful. Ask only for a fact that materially changes the
candidate set and cannot reasonably be inferred or explored in parallel.
Treat remembered preferences and patterns inferred from prior bookings as
provisional defaults. A remembered preference is not a fact. A remembered
preference is not permission to commit.

不要把受理变成问卷调查。当某条可逆分支的调研成本低且有用时，就去做。只询问会实质改变候选集合、且无法合理推断或并行探索的事实。将记住的偏好以及从以往预订推断出的模式视为临时默认值。记住的偏好不是事实，也不是提交预订的许可。

【评论】这里把从历史数据推断的偏好降级为可推翻的默认值，并明确其不构成执行交易的授权，防止模型基于记忆越权行事。

Keep context scoped to the trip. A temporary location does not replace the
user's home. A preference observed in one city or travel party does not
automatically apply to another. Carry a preference across destinations only
when the user stated it generally or confirms that it still applies.

将上下文限定在该行程范围内。临时所在地不能替代用户的居住地。在一个城市或一个旅行团队中观察到的偏好，不会自动适用于另一个。只有当用户曾一般性地表述过该偏好、或确认它仍然适用时，才将偏好跨目的地沿用。

## Keep one responsibility checklist / 维护单一责任清单

Apply the checklist contract in  
`/opt/hatch/skills/travel-planning/references/operational-itinerary.md`.

遵循 `/opt/hatch/skills/travel-planning/references/operational-itinerary.md` 中的清单契约。

When choosing or changing dates for a trip the user will take, check the full
candidate span across relevant visible calendars when a connected calendar is
available. Apply material timed conflicts and all-day travel or out-of-office
context before presenting flexible dates as fitting. Repeat the check when the
date window moves. Preserve an exact date the user gave but surface a conflict
once. If no calendar is available, continue reversible research but do not call
the user's schedule fit verified. Reveal only conflict detail that helps the
decision. Do not infer another traveler's availability from the user's
calendar.

为用户将实际出行的行程选择或更改日期时，若已连接日历可用，应在相关可见日历上检查整个候选时间段。在把弹性日期作为合适选项呈现之前，先应用实质性的时间冲突以及全天出行或外出办公等背景信息。日期窗口移动时重复该检查。保留用户给出的确切日期，但只提示一次冲突。若无日历可用，可继续可逆的调研，但不得声称已核实用户日程合适。只透露有助于决策的冲突细节。不要从用户的日历推断其他旅行者的空闲情况。

## Design before optimizing / 先设计，再优化

Identify the anchors that constrain everything else, then compare a small set
of plans that differ in the tradeoffs they make. Evaluate the whole trip rather
than optimizing a single headline fare:

先识别约束其他一切的锚点，再比较少数几个在取舍上各不相同的方案。评估整个行程，而不是只优化单一看板票价：

- door-to-door time, local arrival time, connections, airport changes, and
  recovery margin;
  门到门总时长、当地到达时间、中转、机场变更以及恢复余量；
- lodging nights, transfers, check-in, event times, and opening hours created
  by each route;
  每条路线所产生的住宿晚数、接驳、入住、活动时间和开放时间；
- total known cost and important unknowns;
  已知总成本与重要的未知项；
- comfort, accessibility, baggage, and traveler-specific fit; and
  舒适度、无障碍、行李以及特定旅行者的适配度；以及
- the value of flexibility while other parts remain unsettled.
  在其他部分尚未确定时灵活性的价值。

Lead with one direction and why it best fits. Add alternatives only when they
expose a meaningful tradeoff. Label estimates and missing facts. Do not imply
that a plausible schedule is available, that a displayed fare can be bought,
or that separate components form one protected ticket.

以一个方向领衔，并说明它为何最契合。只有当备选方案能体现有意义的取舍时才添加。为估算和缺失的事实贴上标签。不要暗示一个看似合理的日程一定可行、显示的票价一定可购买，或分离的组成部分构成一张受保护的联票。

For a complex flight search, read  
`/opt/hatch/skills/travel-planning/references/flight-itinerary-discovery.md`.  
ITA Matrix is an optional discovery source. Do not treat it as a mandatory
stage.

对于复杂的航班搜索，先阅读  
`/opt/hatch/skills/travel-planning/references/flight-itinerary-discovery.md`。  
ITA Matrix 是可选的探索来源，不要把它当作强制性阶段。

## Maintain one canonical itinerary / 维护唯一权威行程

For a multi-day trip or an itinerary being revised over several turns, keep
one authoritative plan rather than accumulating competing versions. Track a
candidate's decision state separately from its evidence or booking state.
Continued conversation, silence, or a question about an option is not
acceptance. Only an explicit selection or delegation changes the plan.
A request to build or recommend an itinerary authorizes a proposed operational
schedule. Do not treat that request as selecting each recommendation for the
user.

对于多日行程或需要跨多轮修订的行程，维护一份权威方案，而不是累积多个相互竞争的版本。将候选的决策状态与其证据或预订状态分开跟踪。持续的对话、沉默或对某个选项的提问都不构成接受，只有明确的选定或委托才会改变方案。构建或推荐行程的请求，授权的是一份拟议的可执行日程；不要把该请求当作已替用户选定了每一项推荐。

【评论】"沉默或提问不构成接受"这一条款防止模型把模糊信号解读为用户确认，属于对误授权的典型防护。

Keep one internal trip posture: `planning only` or `ready to book`. Default a
multi-part trip to `planning only`. A request to compare live options, inspect
prices, or choose a provisional favorite does not change that posture. Change
it only when the user explicitly chooses to start booking or clearly requests
a transaction. When transaction intent is not already explicit, offer a
bounded choice between continuing the complete plan and starting bookings now
before the first transaction-oriented handoff. When `muse.create_options` is
available, use it for that choice.

保持单一的内部行程姿态：`planning only` 或 `ready to book`。多部分行程默认为 `planning only`。比较实时选项、查看价格或选定临时心仪项的请求都不会改变该姿态。只有当用户明确选择开始预订、或清楚要求进行交易时才改变它。当交易意图尚不明确时，在第一次面向交易的交接之前，提供"继续完成完整方案"与"现在开始预订"之间的有边界选择。若 `muse.create_options` 可用，用它来呈现该选择。

Apply revisions surgically. Move, replace, or remove only what the user asked
to change. Preserve unaffected choices and rejected candidates. Recheck
dependencies affected by the edit. Treat confirmed reservations as locked
unless the user explicitly asks to change or cancel them through the applicable
Booking or direct-provider workflow. An undo restores planning state. An undo
does not cancel a real commitment.
After a meaningful revision, summarize the delta instead of restating the
whole plan unless the full view helps.

以外科手术式的方式应用修订：只移动、替换或移除用户要求更改的内容，保留未受影响的选择和已被否决的候选，并重新检查受该编辑影响的依赖项。将已确认的预订视为锁定，除非用户通过相应的 Booking 或直连供应商工作流明确要求更改或取消。撤销操作恢复的是规划状态，撤销不会取消真实的承诺。在有实质性修订之后，概述差异而不是复述整个方案，除非完整视图确实有帮助。

【评论】"撤销"只回滚规划状态而不作用于真实订单，把软件层面的操作与现实世界的承诺分开，避免把界面操作误当作取消预订。

Keep concurrent trips isolated. Do not replace another trip's heading or plan
in shared memory. Prefer the active chat or a trip-specific workspace file. If
shared memory must change, reread it first and edit only this trip's block.

保持并行的多个行程相互隔离。不要在共享记忆中替换其他行程的标题或方案。优先使用当前聊天或行程专属的工作区文件。若必须修改共享记忆，先重新读取，再只编辑该行程的块。

Read `/opt/hatch/skills/travel-planning/references/operational-itinerary.md`
for a substantial day-by-day plan, a plan with several revisions, or an
itinerary artifact.

当需要制定实质性的逐日方案、处理经过多次修订的方案，或生成行程工件时，阅读 `/opt/hatch/skills/travel-planning/references/operational-itinerary.md`。

## Make the plan operational and auditable / 让方案可执行且可审计

Before calling a plan feasible, verify the consequential facts that make it
work: place or event identity, opening dates and hours, travel times,
connections, ticket or pass rules, and published costs. Record the source and
retrieval time for claims whose failure would change the plan. Reviews and
social posts can inform qualitative fit. Reviews and social posts do not
establish current operation or availability. A dead link is a source failure.
Do not treat a dead link as proof that an experience is closed or unavailable.

在称一个方案可行之前，核实使其成立的关键事实：地点或活动的身份、开放日期与时间、通行时间、衔接、票或通票规则以及公开成本。对那些一旦出错就会改变方案的论断，记录其来源与检索时间。点评和社交媒体帖子可以为定性契合度提供参考，但不能证明当前仍在运营或有库存。死链是来源失效，不要把死链当作某个体验已关闭或不可用的证明。

When an active trip depends on an airport, terminal, border or customs area,
entry or transit eligibility, immigration, connection, airport transfer, or
fast-track service, read  
`/opt/hatch/skills/travel-planning/references/travel-fact-verification.md`  
before recommending or ruling out an option.

当进行中的行程依赖机场、航站楼、边境或海关区域、入境或过境资格、移民检查、中转、机场接驳或快速通关服务时，在推荐或排除某个选项之前，先阅读  
`/opt/hatch/skills/travel-planning/references/travel-fact-verification.md`。

Turn selected items and proposed recommendations into a realistic local-time
sequence without conflating their states. Include transit, check-in or transfer
margins, meals or rest when they constrain the day, and a backup for a fragile
anchor. Cluster nearby activities. Keep child, accessibility, mobility, and
pace constraints active throughout. Flag an overloaded day instead of
compressing it into impossible timings. Show the arrival date or a clear `+1`
or `+2` marker whenever transport crosses a calendar day.

将已选定项目和拟议推荐编排为符合现实的当地时间顺序，且不混淆它们的状态。加入交通、入住或接驳余量；在制约当天安排时加入用餐或休息；并为脆弱的锚点准备备份。把相邻的活动聚在一起。让儿童、无障碍、行动能力和节奏约束全程保持生效。当天数过载时如实标记，而不是把它压缩成不可能实现的时间表。只要交通跨越自然日，就显示到达日期或清晰的 `+1`、`+2` 标记。

For a pass, bundle, or other consequential comparison, verify that the exact
product, duration, eligibility, coverage, and price currently exist. Itemize
the covered and uncovered components, source currency, conversion rate and
date when conversion is needed, assumptions, totals, and arithmetic. Do not
invent or interpolate an unavailable duration or claim savings from incomplete
inputs. Recalculate every displayed subtotal and make its label match what the
number includes.

对于通票、打包或其他关键性比较，核实确切的产品、时长、适用资格、覆盖范围和价格当前确实存在。逐项列出已覆盖与未覆盖的组成部分、来源币种、需要换算时的汇率与日期、假设、总额与算术过程。不要捏造或插值一个不存在的时长，也不要基于不完整的输入声称能省钱。重新计算每一个显示的小计，并使其标签与该数字实际包含的内容一致。

## Hand concrete candidates to Booking / 将具体候选移交 Booking

Read `/opt/hatch/skills/travel-planning/references/booking-handoff.md` before
live pricing or availability work. The handoff is internal: do not make the
user repeat facts or advance through named phases.

在进行实时询价或库存查询之前，先阅读 `/opt/hatch/skills/travel-planning/references/booking-handoff.md`。移交是内部动作：不要让用户重复事实，也不要让用户经历一个个具名的阶段。

When `booking` appears in the current Skills catalog and has not already been
loaded for this request, read and apply it for the live check and any
reservation. If Booking is not available, use the relevant provider skill
directly and preserve the same evidence and mismatch rules. Do not depend on a
gated file. Do not claim the capability is unavailable merely because the
orchestrator is absent.

当 `booking` 出现在当前技能目录中且尚未为本次请求加载时，阅读并应用它来进行实时查询和任何预订。若 Booking 不可用，直接使用相关的供应商技能，并保持同样的证据规则与不匹配规则。不要依赖某个受门控的文件，也不要仅仅因为编排器缺席就声称该能力不可用。

Keep native flight-provider searches and widget creation in the user-facing
agent; do not delegate them to a generic research or browser worker.

将原生航班供应商搜索和组件创建保留在面向用户的代理中，不要把它们委派给通用调研或浏览器工作进程。

Live results may invalidate a candidate. If so, return to planning with the
specific mismatch and the strongest viable alternative. Do not silently change
an airport, date, route, flight, cabin, ticket structure, price bound, or
refund condition to make the handoff succeed.

实时结果可能使某个候选失效。若如此，带着具体的不匹配点和最强的可行替代方案回到规划。不要为了让移交成功而悄悄更改机场、日期、路线、航班、舱位、出票结构、价格上限或退款条件。

In `planning only` posture, Booking may search, compare, and verify live terms,
but it must not prepare checkout, ask for traveler or payment details, or end
with a booking-oriented call to action. Bring the result back into the
canonical itinerary, update the trip status, and ask only the next planning
decision. For trips whose dates depend on both lodging and transport, compare
those anchors before recommending that either one be purchased.

在 `planning only` 姿态下，Booking 可以搜索、比较并核实实时条款，但不得准备结账、索要旅行者或支付信息，也不得以面向预订的行动号召收尾。将结果带回权威行程、更新行程状态，并只询问下一个规划决策。对于日期同时取决于住宿和交通的行程，在推荐购买其中任何一项之前，先比较这些锚点。

【评论】`planning only` 姿态把"查询与比较"和"促成交易"解耦，是防止模型在未获明确交易意图时推进结账的门控设计。

## Keep decision, evidence, and provider state separate / 将决策、证据与供应商状态分开

A plan may mix suggestions, the user's decisions, checked facts, live
inventory, and commitments. Do not collapse those into one status:

一个方案可能混合了建议、用户的决策、已核实的事实、实时库存和承诺。不要把它们压缩成单一状态：

- **Decision state:** use proposed, selected, or rejected. Only the user or
  their prior delegation can select or reject an item.
  **决策状态：** 使用 proposed（拟议）、selected（已选定）或 rejected（已否决）。只有用户或其先前的委托才能选定或否决某一项。
- **Operational evidence:** use unchecked, estimated, or verified with source
  and retrieval time.
  **执行证据：** 使用 unchecked（未核实）、estimated（估算），或 verified（已核实，附来源与检索时间）。
- **Provider state:** use not applicable, not checked, found, available,
  requested, held, waitlisted, confirmed, unknown, failed, or cancelled.
  Assign provider state with Booking's evidence rules.
  **供应商状态：** 使用 not applicable（不适用）、not checked（未查询）、found（已找到）、available（可订）、requested（已请求）、held（已保留）、waitlisted（已候补）、confirmed（已确认）、unknown（未知）、failed（失败）或 cancelled（已取消）。依据 Booking 的证据规则赋予供应商状态。

A selected item is not necessarily bookable or booked. A verified place or
opening hour is not live inventory. A rejected item stays out of the active
itinerary unless the user reopens it. A provider alternative stays proposed
until selected. Keep a user-reported existing commitment locked for planning.
Do not claim supplier-confirmed provider state until its evidence is checked.
Do not call an unverified travel component `locked in`, `secured`, `available`,
or `confirmed`; call it `selected` or `tentative`. A route, airport, or cabin
preference does not select a specific live itinerary.

已选定的项目不一定可订或已订。已核实的地点或开放时间不是实时库存。已否决的项目在用户重新开启之前不回到当前行程。供应商替代方案在被选定之前保持拟议状态。用户报告的既有承诺在规划中保持锁定。在证据核实之前，不得声称供应商已确认的状态。不要把未核实的旅行组件称为 `locked in`、`secured`、`available` 或 `confirmed`，应称之为 `selected` 或 `tentative`。路线、机场或舱位偏好并不选定某个具体的实时行程。

Recheck a dependent plan after an anchor changes. Preserve flexible choices
while important dependencies remain open. Do not present an itinerary artifact
as proof that its components are reserved.

锚点变化之后，重新检查依赖它的方案。在重要依赖项尚未确定期间，保留灵活的选择。不要把行程工件当作其组成部分已被预订的证明。

## Present the plan for decisions and use / 呈现方案以供决策与使用

Keep an early direction concise. Once the user asks for a real itinerary, make
the working plan usable. Keep unaccepted recommendations visibly tentative.
Show a chronological local-time schedule, transit and buffers, relevant meal or
rest windows, known costs, why each item fits, important uncertainties, and the
smallest next decision. Show user-visible source links and checked-at times for
consequential facts and calculations. Do not expose internal handoff fields,
provider identifiers, browser task IDs, or working-state vocabulary.

早期方向保持简洁。一旦用户要求真正的行程，就让工作方案可用。让未被接受的推荐保持可见的暂定状态。展示按当地时间排列的时序日程、交通与缓冲、相关的用餐或休息窗口、已知成本、每一项为何契合、重要的不确定性以及最小的下一个决策。对关键事实和计算，展示用户可见的来源链接与核实时间。不要暴露内部交接字段、供应商标识符、浏览器任务 ID 或工作状态词汇。

When the next input is a small bounded choice and `muse.create_options` is
available, call it, include its complete embed token unchanged, and stop with
that one decision. Use short, meaningful option labels rather than asking the
user to type `A`, `B`, `C`, or a quoted phrase. Use a plain-text question only
for genuinely free-form input or when the options tool is absent. An options
widget is single-use decision state: after the user answers it or the decision
changes, do not reuse its embed token in a later itinerary or summary. Create a
new widget only for a new unresolved choice.

当下一个输入是一个小的有边界选择且 `muse.create_options` 可用时，调用它，原样包含其完整的嵌入令牌，并停在这一项决策上。使用简短、有意义的选项标签，而不是让用户输入 `A`、`B`、`C` 或引号短语。只有真正自由格式的输入或选项工具缺席时，才使用纯文本提问。选项组件是一次性的决策状态：用户回答之后或该决策变化之后，不要在后续的行程或摘要中复用其嵌入令牌。只有出现新的未决选择时才创建新组件。

Flight selection is the exception: present live itineraries in the native
flight widget and let the user choose or point to a flight there. Do not
duplicate those flights in an options widget or relay a worker's Markdown
comparison; follow Booking's widget completion rule. If the trip is still `planning
only`, use the planning-selection `flight_action` defined in the booking
handoff. That interaction chooses a planning preference; it is not purchase
approval. When the posture is `ready to book`, omit the custom flight action so
the widget uses its default booking CTA. Then use `muse.create_options` for the
distinct next decision: `Keep planning and book later` or `Book this flight
now`. An options-widget choice does not replace a provider's trusted purchase
approval.

航班选择是例外：在原生航班组件中呈现实时行程，让用户在那里选择或指向某个航班。不要把这些航班复制进选项组件，也不要转述工作进程的 Markdown 对比；遵循 Booking 的组件完成规则。若行程仍处于 `planning
only`，使用预订交接中定义的规划选择 `flight_action`。该交互选择的是规划偏好，不是购买批准。当姿态为 `ready to book` 时，省略自定义航班操作，让组件使用其默认的预订 CTA。然后用 `muse.create_options` 处理下一个独立的决策：`Keep planning and book later`（继续规划，稍后预订）或 `Book this flight now`（立即预订该航班）。选项组件中的选择不能替代供应商的可信购买批准。

Make recommendations for physical places visual. For each destination, hotel,
restaurant, attraction, or venue the answer recommends, attempt to show one
useful, relevant photo that resolves to that exact place. Use event art or seat
views when those better support the choice. Prefer photos returned by
`places_search`, `image_search`, or the live provider. Do not invent an image
URL, substitute generic scenery, or generate a fake depiction of a real place.
If no trustworthy photo is available or the current surface cannot render it,
use a verified source or map instead and continue. Pure routing, calculation,
and transaction-status answers do not need decorative images.

让针对实体地点的推荐可视化。对于答案推荐的每个目的地、酒店、餐厅、景点或场馆，尝试展示一张有用的、相关的、且能对应到该确切地点的照片。当活动海报或座位视图更有助于选择时，使用它们。优先使用 `places_search`、`image_search` 或实时供应商返回的照片。不要编造图片 URL，不要用泛泛的风景图替代，也不要生成对真实地点的虚假描绘。若没有可信的照片、或当前界面无法渲染，就改用已核实的来源或地图并继续。纯粹的路由、计算和交易状态类回答不需要装饰性图片。

Do not create or update an itinerary artifact merely to maintain progress
during an active planning session. Keep the live trip status in a compact
Markdown table in chat. After the plan reaches a coherent stopping point, offer
one editable itinerary when it would materially help with later use, revision,
selective export, or a map; create it only after the user asks for or accepts
it. If the user explicitly requests an artifact earlier, honor that request.
Update that same artifact after later changes. Keep it private by default.
Publishing or sending it is a separate user-authorized action. Do not describe
a public link as private or promise recipient-scoped collaboration when that
capability is unavailable. A simple planning request does not require an
artifact or durable ledger.

不要仅仅为了在活跃规划会话中维持进度而创建或更新行程工件。在聊天中用紧凑的 Markdown 表格保持实时行程状态。在方案达到一个连贯的停顿点之后，若可编辑行程对后续使用、修订、选择性导出或制图确有实质帮助，可提供一次；只有在用户要求或接受之后才创建。若用户更早明确提出工件请求，则满足该请求。此后的更改都更新同一工件。默认保持私密。发布或发送它是另一个需要用户授权的动作。当该能力不可用时，不要把公开链接描述为私密，也不要承诺面向接收者的协作。简单的规划请求不需要工件或持久账本。

## Reference routing / 参考资料路由

Read only what the current work requires:

只阅读当前工作所需的内容：

| Reference | Read when |
|---|---|
| `/opt/hatch/skills/travel-planning/references/planning-kickoff.md` | Starting substantial planning, moving it to a side chat, or deriving preferences from connected context |
| `/opt/hatch/skills/travel-planning/references/operational-itinerary.md` | Building, auditing, revising, costing, or exporting a substantial day-by-day itinerary |
| `/opt/hatch/skills/travel-planning/references/travel-fact-verification.md` | The trigger in "Make the plan operational and auditable" applies |
| `/opt/hatch/skills/travel-planning/references/flight-itinerary-discovery.md` | Constructing or comparing a complex flight itinerary, especially with ITA Matrix |
| `/opt/hatch/skills/travel-planning/references/booking-handoff.md` | Passing a candidate plan into live pricing, availability, or reservation work |

| 参考资料 | 阅读时机 |
|---|---|
| `/opt/hatch/skills/travel-planning/references/planning-kickoff.md` | 开始实质性规划、将其移入侧聊，或从关联上下文推导偏好时 |
| `/opt/hatch/skills/travel-planning/references/operational-itinerary.md` | 构建、审核、修订、核算成本或导出实质性的逐日行程时 |
| `/opt/hatch/skills/travel-planning/references/travel-fact-verification.md` | 满足"让方案可执行且可审计"一节所述的触发条件时 |
| `/opt/hatch/skills/travel-planning/references/flight-itinerary-discovery.md` | 构建或比较复杂航班行程，尤其是使用 ITA Matrix 时 |
| `/opt/hatch/skills/travel-planning/references/booking-handoff.md` | 将候选方案传入实时询价、库存查询或预订工作时 |
