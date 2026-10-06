<!-- BILINGUAL-EN-ZH -->
---
name: "booking"
description: "Primary entry point for direct flight, hotel, restaurant, or event-ticket transactions and for bounded live availability checks delegated by Travel Planning. Always use before provider-specific skills or browser work when the user asks to find live availability or prices, compare bookable options, book, or continue an active booking. Do not use for trip planning itself, broad inspiration, opening hours, schedules, flight status, or other factual questions without transaction intent."
metadata: { "includeInPrompt": true }
---

# Booking / 预订

Use plain Markdown for booking responses except for native flight comparisons
and the hotel map/list presentation described in the hotel reference. Do not
emit HTML or call `create_options`. Do not duplicate booking inventory in two
equivalent selection widgets.

预订回复使用纯 Markdown，原生航班对比与酒店参考中描述的酒店地图/列表呈现除外。不要输出 HTML，也不要调用 `create_options`。不要在两个等价的选控组件中重复同一批预订库存。

Turn a direct booking request into a verified reservation or purchase with as
little work from the user as possible. Examples include "book dinner tomorrow
for two," "get me a flight from SFO to JFK next Tuesday," "find me a hotel in
SoHo for Friday," and "get tickets to the Patriots' next game."

把直接预订请求转化为经过核验的预订或购买，并让用户的操作尽可能少。例如："订明天两人的晚餐"、"帮我订下周二从 SFO 到 JFK 的航班"、"帮我找周五在 SoHo 的酒店"、"买下爱国者队下一场比赛的门票"。

This skill covers four booking types:

本技能覆盖四种预订类型：

- flights: read `/opt/hatch/skills/booking/references/flights.md` and  
  `/opt/hatch/skills/booking/references/presentation.md`  
  before searching or booking; the required comparison fields affect which
  facts must be retained from search results
  航班：搜索或预订前阅读 `/opt/hatch/skills/booking/references/flights.md` 与 `/opt/hatch/skills/booking/references/presentation.md`；规定的对比字段会影响哪些事实必须从搜索结果中保留
- hotels: read `/opt/hatch/skills/booking/references/hotels.md` before searching
  or booking
  酒店：搜索或预订前阅读 `/opt/hatch/skills/booking/references/hotels.md`
- restaurants: read `/opt/hatch/skills/booking/references/restaurants.md`
  before searching or booking
  餐厅：搜索或预订前阅读 `/opt/hatch/skills/booking/references/restaurants.md`
- sports, concert, theater, and other event tickets: read
  `/opt/hatch/skills/booking/references/tickets.md` before searching or booking
  体育、演唱会、戏剧及其他活动门票：搜索或预订前阅读 `/opt/hatch/skills/booking/references/tickets.md`

Booking is the user-facing entry point. Provider-specific skills are secondary
operating manuals: load this skill and its category reference first, then load
the relevant provider skill before using that provider. Do not substitute a
provider skill or generic browser search for this orchestration layer.

Booking 是面向用户的入口。服务商专属技能是次级操作手册：先加载本技能及其类别参考，再在使用某服务商前加载对应的服务商技能。不要用服务商技能或通用浏览器搜索替代这一编排层。

For a browser checkout, read
`/opt/hatch/skills/booking/references/browser-booking.md` before starting it.
For a non-flight booking, read
`/opt/hatch/skills/booking/references/presentation.md` before presenting options,
a final review, or a confirmation. Read only the references required for the
current booking type.

浏览器结账前，先阅读 `/opt/hatch/skills/booking/references/browser-booking.md`。非航班类预订在呈现选项、最终确认或确认回执之前，先阅读 `/opt/hatch/skills/booking/references/presentation.md`。只阅读当前预订类型所需的参考。

Apply instructions that ask the user a question only in a live conversation.
In a detached worker, use authorized context to make reversible assumptions.
Put any unresolved blocker in the final message.

凡需要向用户提问的指令，只在实时对话中执行。在独立工作进程中，用已授权的上下文做可逆假设，并把任何未解决的阻塞项写进最终消息。

## Non-negotiable behavior / 不可协商的行为

- Load the category reference before searching or opening a provider, even when
  a provider-specific skill appears to cover the request.
  在搜索或打开服务商之前先加载类别参考，即使某个服务商专属技能看似已覆盖该请求。
- When a relevant, low-friction booking connector is disconnected, recommend
  connecting it and surface its supported connection action once when that
  would improve live availability, saved-profile use, or checkout. Do not make
  connection a prerequisite or wait idle for it: continue the same request
  through the official website, another reputable provider, and
  `phone.place_call` when that tool is available. A missing, gated,
  unavailable, or unsuccessful connector ends only that path.
  当相关且低摩擦的预订连接器未连接时，推荐连接它，并在能改善实时库存、已存档案使用或结账时，一次性呈现其支持的连接动作。不要把连接设为前置条件，也不要空等：经由官方网站、另一家信誉良好的服务商以及 `phone.place_call`（若该工具可用）继续同一请求。连接器缺失、受门控、不可用或连接失败，只终结那一条路径。
- Keep provider plumbing private. Do not expose internal provider names,
  commands, identifiers, offer ids, or tool choices.
  对服务商管道保密。不得暴露内部服务商名称、命令、标识符、报价 id 或工具选择。
- Present flight comparisons with the native structured flight list when it is
  available. For hotels, use the native map/list plus compact Markdown contract
  in the hotel reference. Present all other booking choices and decisions in
  plain Markdown. Do not use HTML or call `create_options`. When Travel Planning
  owns a plan-first workflow, return the selected candidate to it for the
  separate book-now-or-later decision.
  原生结构化航班列表可用时，用它呈现航班对比。酒店使用酒店参考中的原生地图/列表加紧凑 Markdown 契约。所有其他预订选择与决策用纯 Markdown 呈现。不要使用 HTML 或调用 `create_options`。当 Travel Planning 拥有先规划的工作流时，把选定的候选返回给它，由它另行做"现在订还是稍后订"的决定。
- For event tickets, do not search seat inventory until the exact occurrence
  and ticket count are known, including after a partial reply; an option count
  is not a ticket count. Never infer a seat attribute from its section name; if
  a current source does not state shade or coverage, call it unverified.
  活动门票在确切场次与票数确定之前不得搜索座位库存，包括在部分回复之后；选项数量不等于票数。绝不从看台区名称推断座位属性；若当前来源未说明遮阳或遮蔽情况，就标注为未核实。
- Do not ask the user to paste sensitive values into chat unless the active
  provider skill explicitly permits the field for checkout. Payment-card
  data and account credentials never qualify. Do not repeat sensitive values.
  除非当前服务商技能明确允许该字段用于结账，否则不要让用户把敏感值粘贴进聊天。支付卡数据与账户凭据永远不符合条件。不要复述敏感值。

## Boundary / 边界

Direct use requires booking intent. A specific venue or itinerary is not
required, but the desired outcome is a transaction rather than inspiration.
The exception is a bounded live availability or price check delegated by
Travel Planning, which retains trip-level ownership and receives the selected
candidate back without checkout preparation.

直接使用本技能需要预订意图。不要求具体场所或行程，但期望的结果是一笔交易而非灵感。例外是 Travel Planning 委托的有界实时库存或价格查询：行程的所有权仍在 Travel Planning，它只收回选定的候选，不做结账准备。

Do not use this skill for:

不要把本技能用于：

- planning a trip, building a multi-part itinerary, or filling in adjacent
  needs such as rides, childcare, activities, or meals the user did not ask
  to book; Travel Planning may still delegate one bounded live check
  规划旅行、构建多段行程，或补齐用户未要求预订的相邻需求（如用车、托育、活动、用餐）；Travel Planning 仍可委托一次有界的实时查询
- "where should I go?" exploration with no intent to reserve
  无预订意图的"我该去哪儿？"式探索
- factual questions such as opening hours, reviews, or flight status
  事实性问题，如营业时间、评价或航班状态
- changing or cancelling an existing booking unless the request explicitly
  asks for that and the relevant provider tool supports it
  修改或取消既有预订，除非请求明确要求且相关服务商工具支持

If a direct request contains several bookings, handle each requested item but
do not expand it into a broader trip or occasion plan.

若一个直接请求包含多项预订，逐项处理所请求的内容，但不要扩展成更大的旅行或活动方案。

## Operating principle: discover before asking / 运行原则：先发现、后询问

Do not begin with a questionnaire. Use available, authorized sources to fill
in the request and personalize the search before asking the user for
anything. Work from narrow, relevant queries; do not browse unrelated mail,
accounts, or transactions.

不要以问卷开场。在向用户提问之前，先用可用的已授权来源补全请求并个性化搜索。只做窄而相关的查询；不要翻阅无关的邮件、账户或交易。

Check, when available and relevant:

在可用且相关时检查：

1. Persistent preferences or prior booking context.
   持久偏好或既有预订上下文。
2. Connected email for confirmations, receipts, credits, memberships, and the
   channels previously used for comparable bookings.
   已连接的电子邮件：确认函、收据、报销额度、会员资格，以及过往同类预订所用的渠道。
3. Signed-in provider accounts and booking platforms for profile details,
   loyalty status, credits, points, booking history, and saved preferences.
   已登录的服务商账户与预订平台：档案详情、常旅客等级、报销额度、积分、预订历史与已存偏好。
4. Connected calendar for conflicts with the requested date or time.
   已连接的日历：与所请求日期或时间的冲突。
5. Connected wallet or card profile for saved payment methods and known
   benefits. Plaid, when available, may identify linked card products and
   relevant transactions, but it is not a source of points balances, reward
   rules, or unused travel credits. Do not expose full account or card numbers.
   已连接的钱包或银行卡档案：已存支付方式与已知权益。Plaid 在可用时可识别已关联的银行卡产品与相关交易，但它不是积分余额、奖励规则或未用旅行额度的来源。不得暴露完整账号或卡号。
6. Live inventory and the provider's current terms.
   实时库存与服务商当前条款。

Use what is already connected. For a relevant low-friction booking connector,
briefly recommend connecting it early when doing so improves discovery or
checkout, and provide only its supported connection action. Continue useful
unauthenticated discovery without waiting for the connection. For other
accounts or login walls, offer sign-in when authentication materially unlocks
the best path or is required to complete checkout. Do not turn it into an
up-front questionnaire.

利用已连接的资源。对相关的低摩擦预订连接器，若连接能改善发现或结账，就及早在推荐中简短提及，且只提供其支持的连接动作。不必等待连接，继续有用的无认证发现。对其他账户或登录墙，当认证能实质解锁最佳路径、或完成结账所必需时才提供登录。不要把它变成前置问卷。

## What to remember / 需要记住的内容

Build a small booking profile from explicit choices and reliable history.
Keep three kinds of data separate:

从显式选择与可靠历史构建一个小型预订档案。把三类数据分开：

- **Reusable preferences:** airports, airlines, seat location, cabin, hotel
  style, neighborhood, restaurant tastes, price comfort, usual dining time,
  seating preferences, and preferred booking channels. Save these through an
  available persistent memory or profile capability.
  **可复用偏好：** 机场、航空公司、座位位置、舱位、酒店风格、街区、餐厅口味、价格承受度、惯常用餐时间、座位偏好以及偏好的预订渠道。通过可用的持久记忆或档案能力保存。
- **Account facts:** loyalty program names, status tiers, point currencies,
  card products, travel-credit programs, and which booking accounts exist.
  Keep these only in an approved account/profile store or the connector that
  owns them.
  **账户事实：** 常旅客计划名称、等级、积分币种、银行卡产品、旅行报销额度计划，以及存在哪些预订账户。只保存在经批准的账户/档案存储或拥有它们的连接器中。
- **Sensitive booking identity:** legal name, date of birth, passport details,
  Known Traveler Number or TSA PreCheck number, loyalty membership numbers,
  and payment credentials. A provider skill may permit checkout fields
  in chat; use them only for that checkout. Never copy them into ordinary
  memory, notes, artifacts, or workspace files, or claim they were remembered.
  **敏感预订身份：** 法定姓名、出生日期、护照信息、Known Traveler Number 或 TSA PreCheck 号码、常旅客会员号以及支付凭据。服务商技能可以允许在聊天中输入结账字段；只将其用于该次结账。绝不把它们复制进普通记忆、笔记、产物或工作区文件，也不要声称已记住它们。

【评论】把"可复用偏好 / 账户事实 / 敏感身份"分级并禁止敏感字段进入持久记忆，是按数据敏感度分层存储的典型数据最小化设计。

Apply these evidence rules:

应用以下证据规则：

- An explicit statement is a preference.
  显式陈述即偏好。
- A repeated pattern across comparable bookings is a tentative preference.
  跨同类预订反复出现的模式是暂时性偏好。
- One old booking is a search hint, not a permanent preference.
  单次久远的预订只是搜索提示，不是永久偏好。
- Do not infer allergies, accessibility needs, identity fields, or willingness
  to spend from history.
  绝不从历史推断过敏、无障碍需求、身份字段或消费意愿。
- Do not overwrite an explicit preference with an inferred one. When recent
  evidence conflicts and changes the recommendation, surface the conflict at
  the decision point.
  不要用推断覆盖显式偏好。当近期证据与既有偏好冲突并改变推荐时，在决策点上说明该冲突。
- After a completed booking, update reusable preferences when the user made
  an explicit choice or the booking reinforces a repeated pattern. Store a
  concise fact, not the full itinerary or receipt.
  完成预订后，当用户做出显式选择、或该预订印证了重复模式时，更新可复用偏好。存一条简明事实，而不是完整行程或收据。

If persistent memory is unavailable, keep the profile session-local and say
nothing that implies it will survive this conversation.

若持久记忆不可用，档案只在会话内保留，且不要说任何暗示它会跨对话存续的话。

## Direct-booking flow / 直接预订流程

### 1. Normalize the request / 规范化请求

Resolve relative dates into exact local dates. Identify the booking type and
the minimum transaction fields for it. Carry reasonable assumptions into
search when they are reversible. For example, use one traveler when the user
says "me," or search near the user's home when context makes that clear. Label
those assumptions when presenting results.

把相对日期解析为确切的本地日期。识别预订类型及其最低交易字段。可逆的合理假设可带入搜索。例如用户说"我"时按一名出行者处理，上下文明确时在用户家附近搜索。呈现结果时标注这些假设。

Do not invent legal names, ages, identity numbers, accessibility needs,
dietary restrictions, or payment details.

绝不虚构法定姓名、年龄、身份证件号、无障碍需求、饮食限制或支付细节。

### 2. Gather context proactively / 主动收集上下文

Inspect the relevant sources above, preferably in parallel. Extract only facts
that can affect this booking. Record internally whether each fact was explicit,
observed repeatedly, or inferred from one prior booking.

检查上列相关来源，最好并行进行。只提取可能影响本次预订的事实。在内部记录每个事实是显式给出的、被反复观察到的，还是由单次历史预订推断的。

Do not ask for information already available from an authorized source. Delay
sensitive identity and payment fields until a chosen option actually requires
them.

不要询问已能从授权来源获得的信息。敏感身份与支付字段推迟到所选选项真正需要时再索取。

### 3. Ask only for a true blocker / 只在真正受阻时询问

Start searching as soon as a useful search can run. Ask only when a missing
fact would create materially different searches, risks booking the wrong
thing, or is required to transact and cannot be obtained securely elsewhere.

一旦能进行有用的搜索就开始搜索。只有当缺失的事实会导致搜索方向实质不同、有订错风险、或为交易所必需且无法从别处安全获得时才提问。

Keep at most one unanswered question in flight. Prefer a short choice with a
recommended default. Do not ask for optional preferences one by one; search
with the best available profile and let the results make tradeoffs concrete.

同时最多保留一个未回答的问题。优先提出带推荐默认值的简短选择。不要逐条询问可选偏好；用现有最佳档案搜索，让结果把权衡具体化。

### 4. Search through the best available path / 通过最佳可用路径搜索

Use the category reference's routing. In general:

遵循类别参考的路由。一般顺序：

1. Use a connected native provider or authenticated first-party account when
   it can complete the job and preserve relevant loyalty or credits.
   当已连接的原生服务商或已认证的一方账户能完成任务并保留相关常旅客权益或报销额度时，优先使用。
2. Use connected booking platforms and compare across them when channel,
   availability, or benefits differ.
   使用已连接的预订平台，并在渠道、库存或权益不同时跨平台比较。
3. Use the authenticated browser on the provider's own site.
   在服务商自有网站上使用已认证的浏览器。
4. Use a reputable marketplace or aggregator.
   使用信誉良好的市场或聚合平台。
5. Use `phone.place_call` when it is in the tool catalog and the business
   accepts bookings by phone. Follow the phone tool's confirmation rules.
   当 `phone.place_call` 在工具目录中且商家接受电话预订时使用它。遵循电话工具的确认规则。
6. Hand off only after the available paths are genuinely blocked. Give the
   exact link or number, the filled-in choices, what remains, and why you could
   not finish.
   只有在可用路径确实全部受阻后才移交。给出确切的链接或电话号码、已填好的选择、还剩什么、以及为什么无法完成。

Actually try an available path before declaring it unavailable. A help page,
search snippet, or missing result on one marketplace is not proof that the
booking cannot be made.

在宣布某条路径不可用之前要实际尝试过。帮助页、搜索摘要或某家聚合平台缺少结果，都不能证明该预订无法完成。

When one provider has no match, no inventory, or cannot complete the booking,
continue immediately through the next suitable provider, the venue or carrier's
official site, and then `phone.place_call` when that tool is available. Do not
ask whether to try the next path. Ask only when every available path is blocked
by a decision, identity field, authentication step, or commitment that requires
the user.

当某服务商无匹配、无库存或无法完成预订时，立即经由下一个合适的服务商、场所或承运商官网继续，再尝试 `phone.place_call`（若该工具可用）。不要询问是否尝试下一条路径。只有当所有可用路径都被需要用户决策、身份字段、认证步骤或承诺阻塞时才提问。

Do not silently substitute a different date, airport, property, restaurant,
event, seating class, fare class, or materially different price.

绝不悄悄替换成不同的日期、机场、物业、餐厅、活动、座位等级、票价等级或实质不同的价格。

### 5. Rank a small set of real options / 对少量真实选项排序

When a choice remains, show three to five bookable contenders, best first.
Include only details that separate them and the full amount the user will
pay, including taxes and mandatory fees. For comparisons without a flight
widget, say why the first option wins for this user, using confirmed
preferences and clearly labeled assumptions. For flight widgets, follow the
acknowledgement guidance in  
`/opt/hatch/skills/booking/references/flights.md`.

当仍需选择时，展示三到五个可预订的竞争选项，最优者在先。只纳入能区分它们的细节以及用户将支付的全额（含税费与强制费用）。没有航班小组件的对比，要用已确认的偏好与清楚标注的假设说明第一个选项为何对该用户胜出。航班小组件则遵循 `/opt/hatch/skills/booking/references/flights.md` 中的确认消息指引。

Render the choices according to
`/opt/hatch/skills/booking/references/presentation.md`. The native
structured list is the primary flight-comparison surface when available; a
compact Markdown table is the fallback for flights and the primary surface for
other booking types. Do not repeat either as a wall of prose.

按 `/opt/hatch/skills/booking/references/presentation.md` 渲染选项。原生结构化列表可用时是航班对比的首选承载面；紧凑 Markdown 表格是航班的兜底、也是其他预订类型的首选承载面。两者都不要再用大段文字复述。

Every option shown must be live enough to pursue now. Recheck stale inventory
before checkout. Keep estimated prices, missing terms, and unverified attributes
outside the bookable shortlist. Do not dump raw provider output or internal
identifiers.

展示的每个选项都必须真实可订、可立即跟进。结账前复查过期库存。估算价格、缺失条款与未核实属性不得进入可预订候选清单。不要倾倒原始服务商输出或内部标识符。

If one exact option was requested and is available on acceptable terms, skip
the comparison and prepare that option directly.

若用户点名了某个确切选项且其条件可接受，跳过对比，直接准备该选项。

### 6. Prepare to the commitment boundary / 准备到承诺边界为止

Fill in known details and advance the selected option to the final review
page. A free, clearly cancellable hold may be placed when it protects the
requested booking; immediately disclose the hold and its expiry.

填入已知细节，把选定选项推进到最终确认页。当免费且可明确取消的占位能保住所请求的预订时，可以创建；须立即披露该占位及其到期时间。

Before submitting a reservation or purchase, show a compact final review. This
rule applies to deposits, points transfers, non-refundable commitments, and
cancellation or no-show penalties. Include:

提交预订或购买之前，展示一份紧凑的最终确认。该规则适用于定金、积分转移、不可退款承诺以及取消或未到场罚则。内容包括：

- exact item, provider, date, time, party/traveler count, and selected variant
  确切项目、服务商、日期、时间、人数/出行人数以及所选变体
- itemized price, mandatory fees, credits or points applied, amount due now,
  and amount due later
  分项价格、强制费用、已应用的报销额度或积分、现在应付金额与之后应付金额
- cancellation, refund, change, and no-show terms that affect the decision
  影响决策的取消、退款、改签与未到场条款
- payment source only by safe label such as issuer, product, and last four when
  an authorized tool provides it
  支付来源仅以安全标签呈现（如发卡机构、产品名与末四位），且须由授权工具提供

Ask for confirmation on those exact terms through the relevant trusted
approval surface. When the provider tool will present the same approval, do
not add a duplicate chat confirmation. An earlier "book it" authorizes the
workflow, not a changed price or an undisclosed commitment.

通过相关的可信审批面就这些确切条款请求确认。当服务商工具会呈现同一审批时，不要在聊天中重复确认。早先的"订吧"授权的是该工作流，而不是变更后的价格或未披露的承诺。

【评论】"先前授权只覆盖当时的条款"把支付授权限定在用户看到的版本上，价格或承诺一旦变化即需重新确认，这是防止授权范围被静默扩张的设计。

Do not transfer points, buy points, apply a scarce certificate, or choose a
different payment method without making that choice visible at final review.

积分转移、购买积分、使用稀缺凭证或更换支付方式，若未在最终确认中让该选择可见，就不得执行。

### 7. Execute and verify / 执行并核验

Submit through the same prepared checkout or provider session. A loaded page,
pending spinner, or card authorization is not success. Report a booking only
after the provider returns a confirmation page, reference, ticket, or clearly
booked state.

通过同一个准备好的结账或服务商会话提交。页面加载完成、转圈中或卡授权通过都不等于成功。只有服务商返回确认页、确认号、票券或明确的已订状态后，才能报告预订成功。

Return a concise receipt with the human-readable booking details,
confirmation/reference number, total paid or committed, and the most important
deadline. Do not include full payment, passport, traveler-security, or loyalty
numbers.

返回一份简明回执，包含可读的预订详情、确认/参考号、已支付或已承诺的总金额以及最重要的截止期限。不要包含完整的支付、护照、旅客安检或常旅客号码。

If submission is ambiguous, say that it is ambiguous, check the provider
account and narrowly search for a confirmation email before retrying. Do not
risk a duplicate booking.

若提交结果不明确，就如实说明不明确，先检查服务商账户并窄范围搜索确认邮件，然后再考虑重试。不要冒重复预订的风险。

### 8. Close the loop / 闭环

Save the receipt or add the booking to the calendar only when the user has
requested it or has an established preference for that behavior. Record any
new reusable preference per "What to remember." State plainly whether the
booking is confirmed, held, waitlisted, or still blocked.

只有当用户提出要求、或已有既定偏好时，才保存回执或把预订加入日历。按"需要记住的内容"记录任何新的可复用偏好。明确说明预订处于已确认、已占位、候补还是仍被阻塞状态。

## Communication / 沟通

- Lead with progress or the recommendation, not process narration.
  以进展或推荐开篇，而不是叙述过程。
- Keep one decision in front of the user at a time.
  每次只让用户面对一个决策。
- Treat `/opt/hatch/skills/booking/references/presentation.md` as the
  response-format contract. Use the
  native structured list for flight options when available, a compact Markdown
  comparison for other options, and a short Markdown receipt for completion.
  把 `/opt/hatch/skills/booking/references/presentation.md` 视为响应格式契约。航班选项可用时用原生结构化列表，其他选项用紧凑 Markdown 对比，完成时用简短的 Markdown 回执。
- Keep the surrounding chat to a short recommendation and one next decision.
  Do not restate every field already visible in the presentation.
  周边的聊天内容保持为一段简短推荐加一个下一步决策。不要复述呈现中已可见的每个字段。
- Do not expose which emails, transactions, or old reservations were read.
  Summarize the useful preference instead.
  不要暴露读取过哪些邮件、交易或旧预订。以有用的偏好概括代之。
- Do not print secrets, full identity numbers, or full payment numbers.
  不打印机密、完整证件号或完整支付号码。
- Be honest about blocks and uncertainty. Trying is not booking.
  对阻塞与不确定保持诚实。尝试过不等于订成了。
