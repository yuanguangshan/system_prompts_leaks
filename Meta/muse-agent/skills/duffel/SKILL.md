<!-- BILINGUAL-EN-ZH -->
---
name: "duffel"
description: "Use Duffel to search, book, pay for, or manage flights. Use Duffel to monitor an already booked flight's fare when the user directly asks for ongoing price monitoring."
allowed-tools:
  - "exec"
  - "widget.create"
  - "read"
  - "write"
  - "edit"
  - "tracking.list"
  - "tracking.get"
  - "tracking.search"
  - "tracking.create"
  - "tracking.update"
  - "tracking.create_entry"
  - "tracking.set_status"
  - "user_goal.list"
  - "user_goal.get"
  - "user_goal.search"
  - "user_goal.create_entry"
  - "cron.list"
  - "cron.view"
  - "cron.status"
  - "cron.add"
  - "cron.update"
  - "cron.remove"
metadata: { "includeInPrompt": false }
---

# Duffel Flight Booking / Duffel 航班预订

For a user-facing flight search or booking, first read  
`/opt/hatch/skills/booking/SKILL.md`,  
`/opt/hatch/skills/booking/references/flights.md`, and
`/opt/hatch/skills/booking/references/presentation.md`. Those files define the
discovery, privacy, presentation, and commitment rules. Use this file for the
Duffel CLI contract. Do not emit HTML, call `create_options`, or duplicate
flight choices in another widget; the native flight widget owns flight
selection and uses the action chosen for the current flow.

面向用户的航班搜索或预订，请先阅读  
`/opt/hatch/skills/booking/SKILL.md`、  
`/opt/hatch/skills/booking/references/flights.md` 与
`/opt/hatch/skills/booking/references/presentation.md`。这些文件定义了探索、隐私、呈现与承诺规则。本文件仅用于 Duffel CLI 约定。不要输出 HTML、调用 `create_options`，也不要在其他组件中重复航班选项；原生航班组件拥有航班选择权，并使用当前流程所选的操作。

Use only this public CLI surface:

只使用以下公开 CLI 接口：

```text
search
seat-options
validate-booking
book
booking-status
cancellation-quote
cancel-booking
```

Do not invent identifiers, call undocumented endpoints, or expose `off_`, `ord_`,
or `ore_` ids to users. Do not show a Duffel command. Keep sensitive values out
of chat unless the booking JSON, `validate-booking`, or `book` accepts them for
the active checkout. Never request payment-card data or account credentials.
Run each search command directly into a distinct temporary JSON file;
do not pipe the search through `head` or a parser, which can hide errors or
required offer data. After the direct search succeeds, a read-only command may
inspect the saved JSON to group and rank offers, but it must not overwrite the
file. Reuse the unchanged file for presentation. If an earlier search was
piped, truncated, reduced, or overwritten, do not repeat it in the same turn;
explain that a complete result was not retained and ask the user to continue in
a new turn. A hand-built summary is not sufficient input for the required
structured flight list.

不要编造标识符、调用未在文档中列出的端点，或向用户暴露 `off_`、`ord_`、`ore_` 开头的 id。不要展示 Duffel 命令。敏感值不要出现在聊天中，除非当前结账流程的 booking JSON、`validate-booking` 或 `book` 接受它们。绝不索要支付卡数据或账户凭据。每条搜索命令都要直接重定向到一个独立的临时 JSON 文件；不要把搜索通过 `head` 或解析器管道传输，那样可能掩盖错误或必需的报价数据。直接搜索成功后，只读命令可以检查已保存的 JSON 来分组和排序报价，但不得覆盖该文件。呈现时复用这份未改动的文件。如果之前的搜索被管道传输、截断、缩减或覆盖，不要在同一轮中重复它；应说明完整结果未被保留，并请用户在新的一轮中继续。手工拼出的摘要不足以作为所需结构化航班列表的输入。

【评论】强制搜索结果先完整落盘为 JSON 文件再复用，是一种防止模型凭记忆复述或篡改报价数据的防幻觉设计。

## Booking flow / 预订流程

1. Start by searching the complete itinerary and exact passenger mix together.
   首先将完整行程与确切的乘客组合放在同一次搜索中。
2. Present the shortlist through  
   `/opt/hatch/skills/booking/references/flights.md`.
   通过 `/opt/hatch/skills/booking/references/flights.md` 呈现入围列表。
3. If Travel Planning delegated the search in `planning only` posture, return
   the selected itinerary to Travel Planning and stop. Do not continue into
   seats, traveler details, validation, or checkout until the posture changes
   to `ready to book`.
   如果 Travel Planning 以 `planning only` 姿态委派了搜索，把选定的行程交回 Travel Planning 并停止。在姿态变为 `ready to book` 之前，不要进入座位、旅客信息、校验或结账环节。
4. Optionally show `seat-options` and collect bookable seats.
   可选地展示 `seat-options` 并收集可预订的座位。
5. Collect required passenger details and any loyalty account the user wants
   attached.
   收集所需的旅客信息，以及用户希望关联的任何常旅客账户。
6. Let `/opt/hatch/skills/booking/references/flights.md` choose between the
   native path and the airline website. When it chooses the airline website,  
   follow  
   `/opt/hatch/skills/booking/references/browser-booking.md`. Recheck the
   itinerary and terms. Do not require the user to complete the checkout.
   由 `/opt/hatch/skills/booking/references/flights.md` 在原生路径与航空公司网站之间做选择。若它选择航空公司网站，则遵循 `/opt/hatch/skills/booking/references/browser-booking.md`。重新核对行程与条款。不要要求用户必须完成结账。
7. Otherwise continue through the supported native path without asking the
   user to choose an implementation provider. Run `validate-booking` until
   `ready_to_book: true`, select an exact returned Stripe Link payment method,
   and call `book` once with the same offer, booking document, and seats.
   否则继续走受支持的原生路径，无需让用户选择实现方提供商。反复运行 `validate-booking` 直至 `ready_to_book: true`，选择一个确切返回的 Stripe Link 支付方式，并用同一报价、预订文档与座位调用一次 `book`。
8. On success, report every carrier confirmation reference/PNR and ticket
   status. Do not add the Duffel name to the message. Any required provider
   attribution belongs to the provider-controlled approval or receipt surface.
   成功后，报告每家承运方的确认编号/PNR 与出票状态。不要在消息中加入 Duffel 名称。任何必需的提供商署名都应放在由提供商控制的批准或收据界面。

Do not skip validation or silently change the selected offer, route, passenger
mix, fare, or seats before booking.

预订前不要跳过校验，也不要悄悄更改已选报价、航线、乘客组合、票价或座位。

## Search and compare / 搜索与比较

```sh
duffel search --origin SFO --destination LAX --departure-date 2026-09-30 \
  --limit 300
duffel search --origin SFO --destination LAX --departure-date 2026-09-30 \
  --return-date 2026-10-04 --adults 2 --child-age 8 --lap-infants 1 --limit 300
duffel search --leg SFO:LAX:2026-09-30 --leg LAX:JFK:2026-10-03 --adults 2 \
  --limit 300
duffel search --origin SFO --destination LAX --departure-date 2026-09-30 \
  --max-connections 0 --sort duration --limit 300
```

Use `--limit 300` for comparison searches so repeated fare variants from one
carrier do not crowd plausible alternatives out of the inspected result set.
If the CLI or provider returns a lower supported cap, use the complete result
it returned and do not retry merely to reach 300. Present only the 3–6 most
useful distinct offers. Start with every flight requested as one itinerary in
the same search: use  
`--return-date` for a round trip and repeated `--leg` arguments for multi-city
travel.

比较类搜索使用 `--limit 300`，以免同一承运方重复的票价变体把合理的其他选项挤出被检查的结果集。如果 CLI 或提供商返回更低的支持上限，就使用它返回的完整结果，不要仅为凑够 300 而重试。只呈现最有用的 3–6 个不同报价。第一步就把用户要求的全部航班作为一条行程放进同一次搜索：往返行程使用 `--return-date`，多城市行程使用多个 `--leg` 参数。

Treat each search as a live snapshot, not a price-monitoring loop. Do not
refresh an identical search in a shell loop or launch duplicate searches in the
background. The CLI suppresses completed and concurrent duplicate searches and
bounds all provider attempts within one user turn. On `duplicate_search`, reuse
the completed result. On `search_in_progress`, wait for the original invocation.
On `search_terminal`, do not retry. On `search_budget_exhausted`, use any results
already available and ask the user before expanding in a new turn.

把每次搜索视为实时快照，而不是价格监控循环。不要在 shell 循环里刷新同一搜索，也不要在后台发起重复搜索。CLI 会抑制已完成与并发的重复搜索，并把所有提供商尝试限定在一轮用户对话之内。遇到 `duplicate_search` 时复用已完成的结果；遇到 `search_in_progress` 时等待原始调用；遇到 `search_terminal` 时不要重试；遇到 `search_budget_exhausted` 时使用已有结果，并先询问用户再在新一轮中扩展搜索。

For flexible dates or airports, choose the smallest useful set of distinct
searches and run them sequentially. Show useful results as soon as they are
available. If the requested comparison exceeds the CLI's turn budget, tell the
user which combinations remain and ask them to continue in a new turn. Do not
silently search the Cartesian product of every possible date and airport.

对于弹性日期或机场，选择数量最少但有用的不同搜索组合并顺序执行。有用的结果一出来就展示。如果所要求的比较超出了 CLI 的单轮预算，告知用户还剩哪些组合，并请其在新一轮中继续。不要悄悄地对所有可能的日期与机场做笛卡尔积式搜索。

Use one search ordered by the user's stated priority; default to
`--sort duration` when they give none. Do not automatically run separate
duration, price, nonstop, and one-stop passes. Add or widen
`--max-connections` only when the user requested that constraint or the first
search produced no useful match. Run a targeted `--airlines` search when a
requested, preferred, or credibly expected carrier is absent from the broad
results, even when another carrier produced useful offers. A second sort is a
separate live search, so run one only when the user's requested comparison
genuinely requires it.

按用户明确给出的优先级做一次排序搜索；未给出时默认 `--sort duration`。不要自动分别运行时长、价格、直飞与一次中转等多轮搜索。仅当用户要求该约束、或第一次搜索没有有用结果时，才增加或放宽 `--max-connections`。当用户指定、偏好或可合理预期存在的承运方没有出现在宽泛结果中时，即使其他承运方已产出有用报价，也要用 `--airlines` 做一次定向搜索。第二次排序就是一次新的实时搜索，因此仅当用户要求的比较确实需要时才执行。

Before presenting, group the saved offers by complete itinerary signature:
all segment airports, local timestamps, marketing and operating flight numbers,
and segment order. Do not let repeated fare brands for one schedule consume the
shortlist. Retain a distinct fare variant only when its cabin, baggage,
refundability, changeability, or price creates a material user tradeoff.

呈现之前，按完整行程签名对已保存的报价分组：全部航段机场、本地时间戳、市场与实际承运航班号，以及航段顺序。不要让同一时刻表的重复票价品牌占满入围名单。只有当某个票价变体在舱位、行李、可退性、可改性或价格上造成实质性的用户取舍时才保留它。

Check carrier coverage after grouping. Use the user's preferences, the active
travel-plan handoff, the response's carrier metadata when present, and verified
route evidence to identify a materially relevant missing carrier. Search that
carrier directly with `--airlines` before moving to Matrix or its website. If
the targeted search still has no usable offer, follow the fallback in
`/opt/hatch/skills/booking/references/flights.md`. Never conclude that Duffel
does not support an airline, or that the airline does not fly the route, from
its absence in one broad result set.

分组之后检查承运方覆盖情况。结合用户偏好、当前 travel-plan 交接、响应中的承运方元数据（若有）以及经验证的航线证据，识别实质相关但缺失的承运方。先用 `--airlines` 直接搜索该承运方，再转向 Matrix 或其网站。如果定向搜索仍无可用报价，遵循 `/opt/hatch/skills/booking/references/flights.md` 中的回退方案。绝不要仅凭某次宽泛结果集中缺席，就断定 Duffel 不支持某家航空公司，或该航空公司不飞这条航线。

Use exact three-letter airport IATA codes, not metro codes such as `NYC`.
Resolve unambiguous places, but confirm ambiguous dates or materially different
airport choices. Do not guess an obscure or ambiguous code. Results are
hard-filtered and nearby-airport inventory is excluded. When multiple airport
pairs are acceptable, start with the preferred pair and expand to another pair
only when requested or when the first search has no useful match; do not search
every pair preemptively. An active travel-plan handoff that asks for the best
option across named metro-area airports counts as an explicit request: search
the smallest relevant set sequentially even if the first airport has a useful
match. Zero results means only that the scanned Duffel inventory contained no
match. For an exact-flight request, match the number in returned Duffel offers
and do not substitute a different flight or use schedule data as proof it is
bookable.

使用精确的三字机场 IATA 代码，而不是 `NYC` 之类的都会区代码。无歧义的地点可直接解析，但有歧义的日期或实质不同的机场选择需要向用户确认。不要猜测生僻或有歧义的代码。结果会被硬过滤，邻近机场的库存被排除。当可以接受多个机场组合时，从首选组合开始，只在用户要求或首次搜索没有有用结果时扩展到另一组合；不要预先搜遍所有组合。要求在指名的都会区多个机场之间选最优的 travel-plan 交接视为显式请求：即使首选机场已有有用结果，也要顺序搜索最小的相关集合。零结果只说明扫描到的 Duffel 库存中没有匹配。对于指定航班请求，须匹配返回的 Duffel 报价中的该航班，不得替换成其他航班，也不得把时刻表数据当作可预订的证明。

Passenger pricing is fixed at search:

乘客定价在搜索时即已固定：

- `--adults` is the adult count.
  `--adults` 是成人人数。
- Repeat `--child-age` for each seated traveler aged 2–17, using age on the
  itinerary's final travel date.
  对每位 2–17 岁有座旅客重复使用 `--child-age`，以其在行程最后旅行日期时的年龄为准。
- `--lap-infants` counts travelers under two without a seat; each must have a
  different accompanying adult.
  `--lap-infants` 统计两岁以下无座旅客；每名婴儿必须对应不同的陪同成人。

Ask for missing ages once. Ask whether an infant is seated or on a lap. Do not
infer it. This integration cannot reliably book an infant under two in their
own seat, so do not validate or pay for that case; direct it to the airline.
Do not search a known child as an adult or claim a child will be repriced at
booking. Search again whenever the passenger mix changes.

缺失的年龄只询问一次。询问婴儿是有座还是抱坐，不要自行推断。本集成无法可靠地为两岁以下婴儿预订单独座位，因此不要为该情况做校验或付款，而应引导用户去找航空公司。不要把已知儿童当成人搜索，也不要声称儿童会在预订时重新计价。乘客组合一旦变化就重新搜索。

Prefer one combined Duffel offer and booking for the complete itinerary. If no
combined offer contains every requested leg, or the user explicitly asks
for separate tickets, split only with their agreement before the first `book`.
Explain that each
booking has its own confirmation, fare conditions, approval, and payment; later
offers are not held while an earlier one is purchased, so a later failure can
leave earlier tickets booked. Compare the combined total. Compare useful
alternatives such as cheapest, fastest, nonstop, and explicitly refundable,
and identify all legs and total party price. Treat native-provider inventory as
one live source, not exhaustive coverage across every airline channel.

优先为完整行程选择一个合并的 Duffel 报价并一次预订。如果没有合并报价包含全部所需航段，或用户明确要求分开出票，只有在首次 `book` 之前征得其同意才能拆分。要说明每笔预订都有各自的确认编号、票价条件、审批与支付；购买前一笔时后继报价不会被保留，因此后一笔失败可能导致前几张票已被订出。比较合并总价。比较实用的备选，如最便宜、最快、直飞和明确可退款，并标明全部航段与全队总价。把原生提供商库存视为一个实时来源，而不是覆盖所有航空公司渠道的完整集合。

Present the shortlist with native flight rows through `widget.create`, using
`kind: "list"`. Read the Flights section in
`/opt/hatch/skills/booking/references/flights.md` for
complete-trip rows and presentation.
For list creation, redirect each search to a distinct temporary JSON file.
Check the exit status and inspect the saved response for errors and available
offers before creating the list. Keep the saved output unchanged:

通过 `widget.create` 以 `kind: "list"` 用原生航班行呈现入围列表。完整行程行与呈现方式请阅读 `/opt/hatch/skills/booking/references/flights.md` 的 Flights 部分。创建列表时，把每次搜索重定向到一个独立的临时 JSON 文件。创建列表之前检查退出状态，并检查已保存的响应中有无错误及可用报价。保持已保存输出不变：

```sh
duffel search --origin SFO --destination JFK --departure-date 2026-10-14 \
  --return-date 2026-10-18 --adults 1 --limit 300 > /tmp/sfo-jfk-flights.json
```

Create one row per selected complete offer. Set its `type` to "flight" and
its `data` to the file path and that offer's JSON pointer, for example:

每个被选中的完整报价创建一行。把该行的 `type` 设为 "flight"，把 `data` 设为文件路径与该报价的 JSON 指针，例如：

```json
{
  "flight_details_file": "/tmp/sfo-jfk-flights.json",
  "json_pointer": "/offers/0"
}
```

Use the original offer's zero-based index in `/offers/<index>`. Keep the file
until `widget.create` succeeds. Do not copy the offer JSON into the tool call,
rewrite timestamps, remove fields, or generate a separate widget payload file.
The runtime reads the selected object, maps timestamps, and
stores the complete flight data. It does not retain the path or pointer.
Keep all outbound, return and connecting segments in that one row. Show the
whole offer price once. The server derives an identifying booking reply from
those facts. For direct booking or a trip that is ready to book, omit the
list-level `flight_action` and use the default booking CTA. For a Travel
Planning handoff in `planning only` posture, set `flight_action` to  
`{"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`  
so selection returns the itinerary to the plan instead of implying checkout.
Continue with the same complete offer id through validation and booking only
after the flow advances to booking.
Do not reinterpret a row tap as a request to buy only its first leg. Treat
`refundable` and `changeable` as three-valued: `yes`, `no`, or `not stated`.

在 `/offers/<index>` 中使用原报价从零开始的索引。在 `widget.create` 成功之前保留该文件。不要把报价 JSON 复制进工具调用、改写时间戳、删除字段，也不要另行生成组件负载文件。运行时会读取所选对象、映射时间戳并存储完整航班数据，不会保留路径或指针。把去程、返程与衔接航段都放在同一行中。整单价格只展示一次。服务器会依据这些事实生成用于识别的预订回复。对于直接预订或已可预订的行程，省略列表级 `flight_action` 并使用默认预订 CTA。对于处于 `planning only` 姿态的 Travel Planning 交接，把 `flight_action` 设为 `{"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`，使选择动作把行程交回计划，而不是暗示结账。只有流程推进到预订之后，才在校验与预订中沿用同一完整报价 id。不要把用户点选某一行重新解释为“只买其第一段航段”的请求。把 `refundable` 与 `changeable` 视为三值：`yes`、`no` 或 `not stated`。

Fare brand, cabin, baggage, refundability, and changeability are optional
provider facts. Preserve passenger/segment differences; render missing facts as
"not stated." Do not render missing facts as "none," "not allowed," or
"non-refundable." Airport-local
times must not be timezone-converted without an explicit airport timezone.
`--refundable` means refundable before departure with no stated positive
penalty, not "cancel anytime." Do not infer baggage or airline-standard fees.

票价品牌、舱位、行李、可退性与可改性是可选的提供商事实。保留乘客/航段差异；缺失的事实呈现为 "not stated"。不要把缺失事实呈现为 "none"、"not allowed" 或 "non-refundable"。在没有明确机场时区的情况下，不得对机场本地时间做时区转换。`--refundable` 指起飞前可退且未声明正手续费，不等于“随时可取消”。不要推断行李或航司标准费用。

After selection, use `validate-booking`'s `selected_offer` as the canonical
itinerary, price, baggage, and fare-condition snapshot.

选择之后，把 `validate-booking` 返回的 `selected_offer` 作为行程、价格、行李与票价条件的权威快照。

## Tracked fare monitoring / 已跟踪票价监控

Use this flow when the user directly asks for ongoing monitoring of an already
booked flight. Use this flow when the runtime hands off a validated automatic
booking source for an upcoming flight. Treat either trigger as authorization to
create the watch after the required booking facts are present. Do not ask
whether to start monitoring after the runtime handoff. Do not require a
connected email account. Ask the user only for booking facts that the requested
monitoring needs and that the current context does not supply.

当用户直接要求持续监控已订航班时使用此流程；当运行时为即将出行的航班移交一个经过验证的自动预订来源时也使用此流程。任一触发都视为授权：在所需预订事实齐备后创建监控。运行时移交之后不要再询问是否开始监控。不要求关联电子邮件账户。仅就所需监控需要且当前上下文未提供的预订事实询问用户。

Collect the exact dated itinerary and passenger mix. Collect the booked cabin
or fare family, included bags, and stop pattern. Collect the paid all-in total,
booking currency, and known cancellation, change, or rebooking costs.

收集确切带日期的行程与乘客组合。收集所订舱位或票价族、包含的行李以及经停模式。收集已支付的全包总价、预订货币，以及已知的取消、改签或重新预订费用。

Search user goals and Tracking items for the trip. Use a matching user goal as
the canonical owner. Otherwise, reuse a matching active Tracking item. If
neither exists, create one Tracking item. Resolve the canonical owner before
scheduling a job. Do not create a shadow trip, a price-tracker notebook entry,
or a parallel tracking record. Keep confirmation references, ticket numbers,
and connector identifiers out of user-visible goal text.

在用户目标与 Tracking 条目中查找该行程。有匹配的用户目标时以其为权威所有者；否则复用匹配的活跃 Tracking 条目；两者都不存在时创建一个 Tracking 条目。在调度任务之前先确定权威所有者。不要创建影子行程、价格记录本条目或并行跟踪记录。确认编号、票号与连接器标识不得出现在用户可见的目标文本中。

Store the minimum machine-readable fare state in
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`. Store the
booking fingerprint, source fingerprints, traveler relationship and count,
immutable booked itinerary, paid total, fare terms, current facts, last
comparison, last-notified fingerprint, and cron identifiers. Use
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` only as
goal-owned job state. Do not store full messages, raw connector output, names,
ticket numbers, identity documents, contact data, payment data, loyalty
numbers, Known Traveler Numbers, or CLEAR identifiers.

把最小化的机器可读票价状态存入 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`。存储预订指纹、来源指纹、旅客关系与人数、不可变的已订行程、已付总额、票价条款、当前事实、上次比较、上次通知指纹以及 cron 标识。`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` 仅用作目标所有的任务状态。不要存储完整消息、连接器原始输出、姓名、票号、身份证件、联系方式、支付数据、常旅客号、Known Traveler Number 或 CLEAR 标识。

【评论】状态文件只存指纹与行程事实、排除姓名与证件等个人数据，体现了最小化存储个人信息的设计取向。

List the canonical owner's cron jobs before creating a fare job. Reuse an
equivalent goal-owned fare job when one exists. If no equivalent job exists,
add one fare job for the trip. Use `flight-price-<goal-slug>` as its identifier.
Set its owner to `goal:<goal-slug>`. Schedule the ordinary check daily. Write a
self-contained cron body because the cron worker receives neither this skill
nor the surrounding conversation. Include the goal identifier,
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`, and the exact
Duffel query. Include the comparison rules, alert thresholds, deduplication
rule, failure behavior, and stop date. Record the cron identifier in  
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`.

创建票价任务之前，先列出权威所有者的 cron 任务。存在等价的目标所有票价任务时予以复用；没有等价任务时，为该行程添加一个票价任务，标识符用 `flight-price-<goal-slug>`，所有者设为 `goal:<goal-slug>`。常规检查按每日调度。cron 正文必须自包含，因为 cron 工作进程既收不到本技能文件，也收不到周边对话。正文中要包含目标标识符、`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` 与确切的 Duffel 查询，并包含比较规则、告警阈值、去重规则、失败行为和停止日期。把 cron 标识记录到 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`。

Run exactly one Duffel `search` during each fare cron run. Search the complete
itinerary and passenger mix together with `--limit 30`. Do not use a browser or
web search for repricing. Do not retry `duplicate_search` in the same turn. On
`duplicate_search`, reuse a completed result only when the CLI returns that
completed result in the same command response. On `search_in_progress`, wait
for the original invocation. Retain only results that match the booked dated
flight numbers, airports, passenger count, cabin or fare family, included bags,
stop pattern, and material refund or change restrictions. Compare a candidate
only when its priced scope exactly matches the baseline's complete priced
scope, including every outbound, return, or multi-city leg, dated segment
identity, passenger mix, cabin or fare family, included bags, stop pattern,
currency, and taxes or fees basis. Do not compare a one-way or single-leg offer
with a round-trip or multi-leg paid total. Do not prorate a booking total across
legs unless provider-authored line-item prices establish the exact segment
amount. When priced scope is missing or mismatched, record the check as
non-comparable in  
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`. When priced
scope is missing or mismatched, clear any pending candidate tied to that
mismatch. When priced scope is missing or mismatched, keep the run silent
unless degraded-coverage policy separately triggers. Compute actionable net
savings after known cancellation, change, and rebooking costs.

每次票价 cron 运行中只运行一次 Duffel `search`。用 `--limit 30` 把完整行程与乘客组合放在一起搜索。不要用浏览器或网页搜索重新计价。同一轮内不要重试 `duplicate_search`。遇到 `duplicate_search` 时，只有 CLI 在同一条命令响应中返回已完成结果时才复用它；遇到 `search_in_progress` 时等待原始调用。只保留与已订航班匹配的结果：带日期的航班号、机场、乘客人数、舱位或票价族、包含行李、经停模式，以及实质性的退改限制。仅当候选的计价范围与基线的完整计价范围完全一致时才进行比较，范围包括每一段去程、返程或多城市航段、带日期的航段身份、乘客组合、舱位或票价族、包含行李、经停模式、货币以及税费/手续费口径。不要把单程或单段报价与往返或多段的已付总价进行比较。除非提供商出具的逐项价格能确定确切的航段金额，否则不要把预订总价分摊到各航段。计价范围缺失或不匹配时，把该次检查记为不可比较，写入 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`；计价范围缺失或不匹配时，清除与该不匹配相关的待定候选；计价范围缺失或不匹配时，保持本次运行静默，除非降级覆盖策略另行触发。在扣除已知取消、改签与重新预订费用后计算可行动的净节省。

Treat these baseline gaps as missing priced scope. When the booked fare family
is not stated, a basic, light, saver, or otherwise more restrictive fare brand
is never comparable. When the paid total includes seats, bags, or other extras,
record them separately and compare fares only after removing them or adding the
candidate's current price for the same extras. When the booking source does not
establish whether the paid total covers every traveler, do not compare it.

以下基线缺口视为计价范围缺失。已订票价族未注明时，basic、light、saver 或其他更严格的票价品牌永远不可比较。已付总价包含座位、行李或其他附加项时，单独记录它们，只有在剔除这些项或加上候选当前对同一附加项的价格之后才比较票价。预订来源无法确定已付总价是否覆盖每位旅客时，不要进行比较。

When no pending candidate exists, persist the best threshold-crossing
comparable candidate in  
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` with comparable
itinerary signature, total, currency, and check time. Do not notify the user
for the first threshold-crossing candidate. Temporarily update the same fare
cron to run once about one hour after the current run. When a pending candidate
exists, run one Duffel `search`. Compare equivalent inventory against the
pending candidate and booked facts. Confirm the pending candidate only when
equivalent inventory still qualifies above either threshold. When the pending
candidate is confirmed, continue to the persistence and notification rules
below. Clear the pending candidate when equivalent inventory disappears, no
longer matches the exact comparable criteria, or falls below both thresholds.
Retain the pending candidate with a source-failure record when the confirmation
search cannot complete because a source failed. Restore the same fare cron to
daily cadence after a confirmed candidate, disconfirmed candidate, or source
failure. Keep the run silent after a disconfirmed candidate or source failure
unless degraded-coverage policy triggers.

没有待定候选时，把最佳越阈可比候选持久化到 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`，连同可比行程签名、总价、货币与检查时间。对第一个越阈候选不要通知用户。把同一票价 cron 临时改为在本次运行约一小时后再运行一次。存在待定候选时，运行一次 Duffel `search`，把等价库存与待定候选及已订事实进行比较。只有等价库存仍能超过任一阈值时才确认待定候选。待定候选确认后，继续执行下文的持久化与通知规则。等价库存消失、不再精确满足可比标准、或低于两个阈值时，清除待定候选。确认搜索因来源失败而无法完成时，保留待定候选并记录来源失败。确认候选、否决候选或来源失败之后，把同一票价 cron 恢复为每日节奏。否决候选或来源失败之后保持运行静默，除非降级覆盖策略触发。

【评论】“首次越阈不通知、约一小时后复查确认”的两段式机制，用于抑制瞬时价格波动造成的误报。

Alert only when confirmed net savings exceed either 10 percent of the original
paid total or the booking-currency equivalent of USD 100. Convert USD 100 with
a fresh FX source. Record the FX source time. When reliable FX is unavailable,
apply only the percentage threshold.

只有当确认的净节省超过原支付总额的 10%，或超过按预订货币折算的 100 美元等值时才告警。用实时汇率来源换算 100 美元，并记录汇率来源时间。没有可靠汇率时，只适用百分比阈值。

Before notifying, read the goal's recent Tracking activities for a later change
to the booked itinerary, such as a rebooked leg or an added traveler. When the
booking changed, update the baseline, clear the pending candidate, and keep the
run silent.

通知之前，先读取该目标的近期 Tracking 活动，检查已订行程此后是否发生变更，例如某段被改签或新增旅客。预订已变更时，更新基线、清除待定候选，并保持本次运行静默。

Persist the comparison and stable alert fingerprint before notifying the user.
Add one concise Tracking activity before notifying the user. State when the
fare was confirmed and that fares can change before the user acts. Ask whether
the user wants the main agent to investigate a credit, refund, or rebooking path.
When the user accepts, run one fresh Duffel `search` before quoting savings.
Treat the user's answer as authorization for investigation and preparation
only. Do not cancel, rebook, contact a provider, accept terms, or spend money
without the approval required for that action. Do not promise that a credit or
refund will be granted.

通知用户之前先持久化比较结果与稳定告警指纹，并先添加一条简洁的 Tracking 活动。说明票价是何时确认的，以及在用户行动之前票价仍可能变化。询问用户是否希望主代理调查抵扣、退款或改签路径。用户接受后，先运行一次全新的 Duffel `search` 再引用节省金额。把用户的答复仅视为对调查与准备的授权。未经该操作所需的批准，不要取消、改签、联系提供商、接受条款或支出资金。不要承诺一定能获得抵扣或退款。

Keep checks silent when no comparable result exists, the price is unchanged, or
the movement stays below both thresholds. After three consecutive expected
source failures, add a degraded-coverage activity to Tracking. Stop the fare job
when the useful change or cancellation window closes, the trip is cancelled
without a replacement, or the Tracking item is completed. Remove the fare job
when the Tracking lifecycle ends.

没有可比结果、价格未变、或波动低于两个阈值时，检查保持静默。连续三次预期内的来源失败后，向 Tracking 添加一条覆盖降级活动。实用改签或取消窗口关闭、行程被取消且无替代、或 Tracking 条目完成时，停止票价任务。Tracking 生命周期结束时移除票价任务。

## Seats / 座位

Use `seat-options` only after shortlisting. It returns passenger-specific,
bookable Duffel services; an omitted seat is unavailable, while `$0.00` is a
bookable seat with no added fee. Use `row` and `section` for adjacency. Seats on
opposite sides of an aisle are not adjacent. Seat maps are carrier-dependent
and inventory remains live.

只在确定入围之后使用 `seat-options`。它返回针对特定乘客、可预订的 Duffel 服务；未列出的座位不可用，而 `$0.00` 是无附加费的可预订座位。用 `row` 和 `section` 判断相邻。过道两侧的座位不算相邻。座位图因承运方而异，库存始终保持实时。

Choose at most one seat per passenger per segment. Do not duplicate a seat.
Pass every choice to validation and booking using the returned one-based
passenger and segment numbers:

每段航程每名乘客最多选一个座位，不得重复选座。用返回的从一开始计的乘客与航段编号，把每个选择传入校验与预订：

```sh
duffel validate-booking --offer-id off_… --booking-json '{...}' \
  --seat 1:1:28B --seat 2:1:28C
```

## Passenger details / 旅客信息

After offer selection, ask in chat for only the values required by the booking
JSON or validation:

选定报价后，在聊天中只询问 booking JSON 或校验所要求的值：

- each traveler's legal given/family name, birth date, title, and gender;
  每位旅客的法定名/姓、出生日期、称谓与性别；
- one reachable trip email and phone, reusable for passengers when designated
  as their shared contact;
  一个可联系的行程邮箱与电话，被指定为乘客共享联系方式时可复用；
- the accompanying adult for each lap infant; and
  每名抱坐婴儿的陪同成人；以及
- passport details only when validation requires them.
  仅在校验要求时才收集护照信息。

Use chat-supplied values only for this checkout; do not persist them elsewhere
or repeat them to the user. Never request payment-card data or account
credentials in chat.

聊天中提供的值只用于本次结账；不要持久化到别处，也不要向用户复述。绝不在聊天中索要支付卡数据或账户凭据。

Allowed titles are `mr`, `ms`, `mrs`, `miss`, and `dr`; gender is `m` or `f`.
A required passport uses `type: "passport"`, its number as
`unique_identifier`, a two-letter uppercase `issuing_country_code`, and
`expires_on` in `YYYY-MM-DD`. Do not send an empty document, invent identity
data, or ticket an unborn traveler; explain that booking must wait until birth.
Do not promise the same fare, availability, or adjacent seats later.

允许的称谓是 `mr`、`ms`、`mrs`、`miss` 和 `dr`；性别为 `m` 或 `f`。需要护照时，用 `type: "passport"`、号码作为 `unique_identifier`、两位大写字母的 `issuing_country_code`，以及 `YYYY-MM-DD` 格式的 `expires_on`。不要提交空证件、编造身份信息，也不要为未出生的旅客出票；应说明预订必须等到出生之后。不要承诺以后仍有相同票价、余位或相邻座位。

Mention loyalty once. If the user wants an account attached, collect its airline
IATA code and account number in chat and add it to the matching passenger's
`loyalty_programme_accounts`. Validation must confirm support and attachment;
omit a failed account only with the user's agreement. Mention optional seat
availability once without blocking.

常旅客只提及一次。用户希望关联账户时，在聊天中收集其航司 IATA 代码与账户号，并加入对应乘客的 `loyalty_programme_accounts`。校验必须确认支持与关联成功；关联失败时仅在征得用户同意后省略。可选座位情况只提及一次，不构成阻塞。

Keep passengers in search order. Do not include Duffel passenger ids or fields
owned by the CLI (`id`, `infant_passenger_id`, `user_id`, `type`,
`selected_offers`, `services`, `payment`, or `payments`). The CLI assigns offer
slots, validates age/type compatibility, and rejects other undocumented fields
before checkout.

乘客保持搜索顺序。不要包含 Duffel 乘客 id 或由 CLI 拥有的字段（`id`、`infant_passenger_id`、`user_id`、`type`、`selected_offers`、`services`、`payment`、`payments`）。CLI 会分配报价槽位、校验年龄/类型兼容性，并在结账前拒绝其他未在文档中列出的字段。

```json
{
  "data": {
    "passengers": [{
      "given_name": "Legal given name",
      "family_name": "Legal family name",
      "born_on": "1990-01-01",
      "title": "ms",
      "gender": "f",
      "email": "trip-contact@example.com",
      "phone_number": "+14155550123",
      "loyalty_programme_accounts": [
        {"airline_iata_code": "UA", "account_number": "…"}
      ]
    }]
  }
}
```

For a lap infant, add `"accompanying_adult": 1`, using the adult's one-based
position in the passenger array.

对于抱坐婴儿，添加 `"accompanying_adult": 1`，取该成人在乘客数组中从一开始计的位置。

## Validate before payment / 支付前校验

```sh
duffel validate-booking --offer-id off_… --booking-json '{...}'
```

Validation refreshes the live offer and checks expiry/current total, passenger
count/order/ages/types, required contacts and documents, lap-infant linkage,
seats, and loyalty. It creates no Stripe request and has no HITL. Ask for every
`required_missing` or `invalid` item together and rerun. Optional omissions do
not block, but requested loyalty must not be silently discarded.

校验会刷新实时报价，并检查有效期/当前总价、乘客数量/顺序/年龄/类型、必需的联系信息与证件、抱坐婴儿关联、座位以及常旅客。它不会创建 Stripe 请求，也没有 HITL。对所有 `required_missing` 或 `invalid` 项一次性询问并重跑。可选项缺省不阻塞，但用户要求关联的常旅客账户不得被悄悄丢弃。

Before `book`, compare every `selected_offer.slices[].from` and `.to` with the
chosen airports. Stop on any mismatch. Do not infer a route from search args,
flight number, or city name.

`book` 之前，把每个 `selected_offer.slices[].from` 与 `.to` 同所选机场逐一比对。任何不匹配即停止。不要从搜索参数、航班号或城市名推断航线。

A first booking may require the user's account email and legal name for
a Duffel payment profile. This is not a passenger and is created internally
only after purchase approval. When validation requests it, pass the same values
to validation and booking:

首次预订可能需要用户的账户邮箱与法定姓名来创建 Duffel 支付档案。这不是乘客，且仅在购买批准之后于内部创建。校验要求提供时，把相同的值传给校验与预订：

```sh
duffel validate-booking --offer-id off_… --booking-json '{...}' \
  --customer-email owner@example.com \
  --customer-given-name Account --customer-family-name Owner
```

## Book once / 仅预订一次

```sh
stripe-link payment-methods list --format json

duffel book --offer-id off_… --payment-method-id pm_… \
  --booking-json '{...}' --seat 1:1:28B --seat 2:1:28C \
  --customer-email owner@example.com \
  --customer-given-name Account --customer-family-name Owner
```

If Stripe Link is disconnected, explain the one-time virtual-card flow and
provide its connection link. If the user cannot use that connection, an
airline-site fallback is allowed only when its secure browser handoff or
provider-owned payment surface actually renders for the user. Do not offer
to receive a card number, expiry, security code, billing address, or other
payment credential in chat. Do not transfer chat-supplied card data into the
browser. If neither secure path is available, stop at the prepared itinerary,
say payment is blocked, and provide the exact safe handoff; do not invent a
chat-card workaround. Omit `--customer-*` when validation did not request them.

若 Stripe Link 未连接，说明一次性虚拟卡流程并提供其连接链接。用户无法使用该连接时，仅当航空公司网站的安全浏览器交接或提供商自有支付界面确实能为用户渲染时，才允许回退到航空公司网站。不要提出在聊天中接收卡号、有效期、安全码、账单地址或其他支付凭据。不要把聊天中提供的卡片数据转移到浏览器。两条安全路径都不可用时，停在已备好的行程上，说明支付被阻断，并给出确切的安全交接方式；不要发明聊天收卡之类的变通办法。校验未要求 `--customer-*` 时省略它们。

【评论】明确禁止在聊天中接收支付凭据、也禁止把聊天中的卡片数据转入浏览器，是典型的支付数据防泄露条款。

`book` refreshes and revalidates before any Stripe request. Its single approval
covers the itinerary, passengers, seats/fees, fare conditions, total, optional
first-time booking profile, one-time virtual card, 3DS, and immediate paid order.
Do not use Duffel balance, held-order, inline-card, or separate order/payment
flows.

`book` 在发起任何 Stripe 请求之前都会刷新并重新校验。其单次批准覆盖行程、乘客、座位/费用、票价条件、总价、可选的首次预订档案、一次性虚拟卡、3DS 以及立即支付的订单。不要使用 Duffel 余额、挂单、内联卡或订单与支付分离等流程。

The trusted approval must show compact passenger names, plus a separate lap-infant
row with name, birth date, accompanying adult, and `no seat`; city and airport
codes, flights, operating carriers, schedules, fare/cabin, selected seats,
masked loyalty, baggage, refund/change conditions, price breakdown, and legal
links. Label absent provider conditions as not stated.

可信批准界面必须显示简明的乘客姓名，外加单独的抱坐婴儿行（姓名、出生日期、陪同成人、`no seat`）；城市与机场代码、航班、实际承运方、时刻、票价/舱位、所选座位、脱敏的常旅客信息、行李、退改条件、价格明细以及法律链接。提供商未给出条件处标注为 not stated。

After success, treat a requested loyalty account marked
`not_confirmed_on_order` as unverified, not rejected: explain that the booking
response did not confirm it, it may still be attached, and the user should
check with the airline using the PNR and add it only if absent. Do not claim
attachment failed solely because the response omitted it. Do not retry or
rebook a successful order to repair loyalty.

成功之后，把标记为 `not_confirmed_on_order` 的已请求常旅客账户视为未验证而非被拒绝：说明预订响应未确认它、它可能仍会被关联，用户应凭 PNR 向航空公司核实，仅在缺失时再添加。不要仅因响应未包含它就声称关联失败。不要为修复常旅客关联而重试或重新预订已成功的订单。

## Recovery / 故障恢复

| Result | Action |
|---|---|
| Validation failure or expired offer before approval | Correct all fields together. If the offer expired, report that and ask the user to continue in a new turn before searching again. No spend exists. |
| `stripe_link_action_required` with `auto_resume` | Send the complete Markdown link once, wait for the user to connect, then rerun the identical `book`. |
| `replacement_required` | Stop; do not change card/offer or create another spend without explicit recovery guidance. |
| Partially completed split booking | Stop. Report the successful bookings and PNRs plus every unpurchased leg. Do not buy a replacement or cancel anything without the user's explicit instruction. |
| Definitive native-provider rejection | Say no booking was created and the card was not captured; any temporary authorization will fall off on the issuer's schedule. Do not retry until the user explicitly requests a new attempt in a later message. |
| Timeout, 5xx, or ambiguous mutation | Do not claim success/failure or retry. Inspect `booking-status`; otherwise escalate. |
| Ambiguous customer-profile creation | No payment started; do not retry profile creation. Escalate for profile recovery. |
| Success with `resource_authorization.persisted: false` | Report the PNR/order and warn later management may be unavailable. Do not repeat the booking. |

| 结果 | 操作 |
|---|---|
| 批准前校验失败或报价过期 | 一次性修正全部字段。若报价已过期，如实报告并请用户在新一轮中再重新搜索。此时尚无任何支出。 |
| `stripe_link_action_required` 且有 `auto_resume` | 完整的 Markdown 链接只发送一次，等待用户连接后，原样重跑同一 `book`。 |
| `replacement_required` | 停止；在没有明确恢复指引的情况下，不要更换卡片/报价或产生新的支出。 |
| 部分完成的拆分预订 | 停止。报告已成功的预订与 PNR，以及所有未购买的航段。未经用户明确指示，不要购买替代票或取消任何内容。 |
| 原生提供商明确拒绝 | 说明未创建预订、卡片未被扣款；任何临时预授权将按发卡行的时间安排自行失效。在用户于后续消息中明确要求重试之前，不要重试。 |
| 超时、5xx 或结果不明确的变更操作 | 不要宣称成功/失败，也不要重试。检查 `booking-status`；否则上报。 |
| 客户档案创建结果不明确 | 支付未开始；不要重试档案创建。上报以进行档案恢复。 |
| 成功但 `resource_authorization.persisted: false` | 报告 PNR/订单，并警告后续管理可能不可用。不要重复该预订。 |

Identical offer plus normalized booking data form a resumable checkout identity.
A newly searched offer is a new checkout and can create another authorization
hold, so do not use it as an automatic retry.

相同报价加规范化预订数据构成一个可恢复的结账身份。新搜出的报价属于新的结账，可能产生另一笔预授权冻结，因此不要把它当作自动重试来使用。

## Review, cancellation, and limits / 查看、取消与限制

```sh
duffel booking-status
duffel booking-status ord_…
duffel cancellation-quote ord_…
duffel cancel-booking ore_…
```

Without an id, `booking-status` lists locally authorized orders and can recover
a lost order id. `cancellation-quote` does not cancel; report its exact known
refund, destination, airline credits, and expiry. A null refund amount is
unknown, not zero: say that the carrier did not provide a quote. `cancel-booking`
re-checks the exact quote and asks once before the irreversible cancellation.
Do not invent an unknown refund or fee.

不带 id 时，`booking-status` 列出本地已授权的订单，并可找回丢失的订单 id。`cancellation-quote` 不会取消预订；要如实报告其给出的已知退款金额、去向、航司积分与有效期。退款金额为 null 表示未知而非零：应说明承运方未提供报价。`cancel-booking` 会重新核对确切报价，并在不可逆取消之前询问一次。不要编造未知的退款或费用。

Unsupported: paid bags or other services, partial-passenger cancellation,
date/time or name changes, pets, unaccompanied-minor service, check-in, and
boarding passes. Do not improvise with raw APIs; direct the user to the
airline or support.

不支持：付费行李或其他附加服务、部分乘客取消、日期/时间或姓名变更、宠物、无成人陪伴儿童服务、值机与登机牌。不要用原始 API 自行变通；引导用户联系航空公司或客服。

## Approval rules / 审批规则

- No HITL: search, seat options, validation, booking status, cancellation quote.
  无 HITL：搜索、座位选项、校验、预订状态、取消报价。
- Exactly one HITL: native `book`, airline browser checkout, or cancellation. The
  airline browser checkout uses the same single purchase-approval boundary as  
  `book`.
  恰好一次 HITL：原生 `book`、航空公司浏览器结账或取消。航空公司浏览器结账与 `book` 使用同一个单次购买批准边界。
- Any change to flight, date, passengers, fare, services, or total requires a
  fresh purchase approval.
  航班、日期、乘客、票价、服务或总价任何一项变更，都需要新的购买批准。
- Retry `search` only when its response explicitly says `retriable: true`;
  successful, duplicate, in-progress, terminal, and budget-exhausted searches
  must not be repeated in the same turn. Other reads may be retried. Do not
  automatically retry an ambiguous mutation.
  仅当 `search` 的响应明确给出 `retriable: true` 时才重试；成功、重复、进行中、终态与预算耗尽的搜索不得在同一轮内重复。其他读取操作可以重试。不要自动重试结果不明确的变更操作。
