---
name: "flightaware"
title: "FlightAware AeroAPI"
description: "Use for questions about a specific flight's departure or arrival time, including "when's my flight?" and confirmation of remembered times, plus flight status, delays, and cancellations. Verify the exact dated flight before answering; memory identifies the itinerary but does not verify its current schedule. Use FlightAware to monitor operational changes for an upcoming booked flight when the user directly asks for ongoing monitoring."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# FlightAware / FlightAware（航班信息）

## Connecting / 连接
There is nothing for the user to connect and no key to enter. Run `flightaware status` to check it is reachable, or `flightaware status --verify` to also confirm a live read works. If status comes back unavailable, tell the user FlightAware is not reachable from this device right now and try again later. Never ask the user for a FlightAware API key.

用户无需连接任何东西，也无需输入密钥。运行 `flightaware status` 检查其可达性，或运行 `flightaware status --verify` 进一步确认实时读取可用。若状态返回不可用，告诉用户当前此设备无法访问 FlightAware，请稍后再试。绝不向用户索要 FlightAware API 密钥。

## Command syntax / 命令语法
Identifiers are positional arguments, not flags. Output is always JSON; add `--compact` for single-line JSON.

标识符是位置参数，不是标志。输出始终是 JSON；加 `--compact` 得到单行 JSON。

- `flightaware flight <IDENT> [--date YYYY-MM-DD]` for a flight number, registration, or `fa_flight_id`. `--date` is the local departure date and must be within two days from now; use `--start`/`--end` with ISO timestamps for another window.
  `flightaware flight <IDENT> [--date YYYY-MM-DD]` 用于航班号、注册号或 `fa_flight_id`。`--date` 是当地出发日期，必须在从现在起两天之内；其他时间窗口用 `--start`/`--end` 加 ISO 时间戳。
- `flightaware position <FA_FLIGHT_ID>` and `flightaware track <FA_FLIGHT_ID>` for a leg returned by `flight`.
  `flightaware position <FA_FLIGHT_ID>` 与 `flightaware track <FA_FLIGHT_ID>` 用于 `flight` 返回的某段航段。
- `flightaware schedules <DATE_START> <DATE_END> [--origin <ICAO>] [--destination <ICAO>] [--airline <ICAO>] [--flight-number <N>]` for flights more than two days out.
  `flightaware schedules <DATE_START> <DATE_END> [--origin <ICAO>] [--destination <ICAO>] [--airline <ICAO>] [--flight-number <N>]` 用于两天以后的航班。

## Common flows / 常见流程

### Track a flight / 追踪航班
Run `flight` with the airline flight number to get its current status, then use the returned `fa_flight_id` with `position` for where it is now or `track` for its recent path. For a picture, run `map` with `--save <path>` to write a static image of the flight.

用航空公司的航班号运行 `flight` 获取其当前状态，然后用返回的 `fa_flight_id` 配合 `position` 查看当前位置，或用 `track` 查看近期路径。需要图片时，运行 `map` 并加 `--save <path>` 写出航班的静态图像。

### Monitor a tracked trip / 监控被追踪的行程

Use this flow when the user directly asks for ongoing monitoring of an upcoming
booked flight. Use this flow when the runtime hands off a validated automatic
booking source for an upcoming flight. Treat either trigger as authorization to
create the watch after the required flight facts are present. Do not ask whether
to start monitoring after the runtime handoff. Do not require a connected email
account. Ask the user only for flight facts that the requested monitoring needs
and that the current context does not supply. Resolve each exact dated leg
before scheduling a job.

当用户直接要求持续监控一个即将到来的已订航班时，使用本流程。当运行时为即将到来的航班交接来一个经过验证的自动订票来源时，也使用本流程。在所需的航班事实齐备之后，把任一触发都视为创建监控的授权。在运行时交接之后，不要再问是否开始监控。不要求已连接的电子邮件账户。只询问所请求的监控需要、且当前上下文未提供的航班事实。在调度任务之前，先确定每一段带确切日期的航段。

Search user goals and Tracking items for the trip. Use a matching user goal as
the canonical owner. Otherwise, reuse a matching active Tracking item. If
neither exists, create one Tracking item. Resolve the canonical owner before
scheduling a job. Do not create a FlightAware-specific database or a duplicate
Tracking item.

在用户目标与 Tracking 条目中搜索该行程。匹配的用户目标作为规范所有者。否则，复用匹配的活跃 Tracking 条目。若两者都不存在，创建一个 Tracking 条目。在调度任务之前先确定规范所有者。不要创建 FlightAware 专用数据库，也不要创建重复的 Tracking 条目。

Store the minimum operational state in
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`. Use
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` only as
goal-owned job state. Do not create a separate travel database.

把最小运行状态存储在 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`。`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` 只用作目标所有的任务状态。不要另建旅行数据库。

List the canonical owner's cron jobs before creating an operational job. Reuse
an equivalent goal-owned operational job when one exists. If no equivalent job
exists, add one operational job for the trip. Use
`flight-status-<goal-slug>` as its identifier. Set its owner to
`goal:<goal-slug>`. Schedule the operational job at creation with the actual
cron schedule for the next active leg's current cadence band. Use a daily cron
schedule while the next active leg departs more than 48 hours from now. Use an
hourly cron schedule from 48 hours before departure until 6 hours before
departure. Use a 10 to 15 minute cron schedule from 6 hours before departure
through landing. Do not create or retain a high-frequency cron schedule that
relies on `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` to
skip source calls. Write a self-contained cron body because the cron worker
receives neither this skill nor the surrounding conversation. Include the
Tracking goal, `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`,
exact leg identity, and booked baseline. Include the deduplication rule, failure
policy, cadence transitions, and stop conditions. Include the exact FlightAware
command lines each run needs, written with the syntax above.

在创建运行任务之前，先列出规范所有者的 cron 任务。已存在等价的目标所有运行任务时，复用之。若不存在等价任务，为该行程添加一个运行任务。用 `flight-status-<goal-slug>` 作为其标识符。把其所有者设为 `goal:<goal-slug>`。创建时即按下一个活跃航段当前节奏带的真实 cron 计划来调度该运行任务。当下一个活跃航段距现在出发超过 48 小时，用每日 cron 计划。从出发前 48 小时到出发前 6 小时，用每小时 cron 计划。从出发前 6 小时直到落地，用 10 到 15 分钟的 cron 计划。不要创建或保留那种依赖 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` 来跳过数据源调用的高频 cron 计划。编写自包含的 cron 正文，因为 cron 工作进程既收不到本技能、也收不到周围的对话。要包含 Tracking 目标、`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`、确切的航段标识和已订基线。要包含去重规则、失败策略、节奏切换和停止条件。要包含每次运行所需的确切 FlightAware 命令行，按上述语法书写。

At the start of each operational run, compare the existing cron schedule with
the next active leg's current cadence band. Update the same cron job before any
FlightAware source call when the existing cron schedule differs from the
current cadence band. Repair an existing high-frequency cron schedule before
any FlightAware source call when the next active leg is outside the
high-frequency band. Keep the current cadence until the active leg lands or a
connection risk remains unresolved. Advance the same job to the next active
leg, then stop it after the final landing.

每次运行开始时，把既有 cron 计划与下一个活跃航段当前的节奏带比较。当既有 cron 计划与当前节奏带不一致时，在任何 FlightAware 数据源调用之前，先更新同一个 cron 任务。当下一个活跃航段不在高频带内时，在任何 FlightAware 数据源调用之前，先修复既有的高频 cron 计划。保持当前节奏，直到活跃航段落地、或联程风险仍未解除。把同一个任务推进到下一个活跃航段，最后一段落地之后停止它。

Compare strategic schedule changes with the booked baseline. Compare
operational changes with the last notified facts. Apply the cancellation
corroboration and status interpretation rules in the Rules section. For a
material movement, flight or carrier change, confirmed cancellation, diversion,
terminal change, qualifying gate change, qualifying equipment change,
qualifying delay, or credible connection risk, create one stable event
fingerprint. Treat the first delay estimate at 30 minutes or more as a
qualifying delay. Treat a later delay-only change as a qualifying delay only
when it crosses from 30 minutes or more to less than 30 minutes, enters a worse
band among 30 to 59 minutes, 60 to 119 minutes, and 120 minutes or more, or
differs by 60 minutes or more from the last notified delay estimate. Do not
include the exact delayed minute value in a delay-only fingerprint. Treat
terminal changes, airport changes, confirmed cancellations, diversions, and
credible connection risks as immediate alert candidates. Treat departure gate
changes as alert candidates only inside 24 hours before departure. Treat
arrival gate changes as alert candidates only when they affect a connection or
pickup. Treat equipment changes as alert candidates only when they affect
cabin, seat, or itinerary facts. Keep final landing silent unless the user
asked for landing notification or a pickup or connection impact remains.
Keep a first gate or terminal assignment, an on-time status, boarding, taxiing,
departure, and in-flight progress silent; none of them is a change. Name only
the current gate in a departure gate alert. From 90 minutes before departure,
send each departure gate change on the run that finds it. Earlier, send at most
one departure gate alert per leg per hour: record a newer gate that arrives
within the hour in  
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json` and send the
then-current gate on the first run after the hour. Stay silent when the gate
returns to the last notified gate, and fold a gate change into a concurrent
delay or cancellation message instead of sending it separately. In a scheduled
run, keep an unconfirmed cancellation silent: record it in
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`, recheck the
exact dated leg, and seek confirmation from an airline or airport source. A
cancellation is confirmed only when FlightAware's `cancelled` flag and status
text agree, or an airline or airport source confirms it. Notify as soon as it
is confirmed, and keep checking on later runs while it stays unconfirmed.
Write these silence rules, including the gate timing and the cancellation
confirmation test, into the cron body with the other alert rules; the cron
worker does not receive this skill.

战略性时刻表变更与已订基线比较。运行性变更与上次已通知的事实比较。应用 Rules 一节中的取消佐证与状态解读规则。对于实质性移动、航班或承运人变更、已确认取消、备降、航站楼变更、够格的登机口变更、够格的机型变更、够格的延误或可信的联程风险，创建一个稳定的事件指纹。首个 30 分钟及以上的延误估计视为够格延误。此后的仅延误变化，只有在以下情况才视为够格延误：从 30 分钟及以上降到 30 分钟以下、进入更差的档位（30 至 59 分钟、60 至 119 分钟、120 分钟及以上三档之中）、或与上次已通知的延误估计相差 60 分钟及以上。仅延误的指纹中不要包含确切的延误分钟数。航站楼变更、机场变更、已确认取消、备降和可信联程风险视为立即告警候选。出发登机口变更只在出发前 24 小时之内才视为告警候选。到达登机口变更只在影响联程或接机时才视为告警候选。机型变更只在影响客舱、座位或行程事实时才视为告警候选。最终落地保持静默，除非用户要求落地通知，或仍有接机或联程影响。首次分配登机口或航站楼、准点状态、登机、滑行、起飞和飞行中进展保持静默；它们都不是变更。出发登机口告警中只报当前登机口。从出发前 90 分钟起，每条出发登机口变更在发现它的那次运行就发送。更早时，每段航段每小时最多发送一条出发登机口告警：把一小时内到达的更新登机口记录到  
`~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`，并在该小时结束后的第一次运行发送当时的当前登机口。当登机口回到上次已通知的登机口时保持静默；登机口变更要并入并发的延误或取消消息，而不是单独发送。在计划运行中，未确认的取消保持静默：把它记录到 `~/workspace/goals/<goal-slug>/hidden_files/travel/flight.json`，复核带确切日期的航段，并从航空公司或机场来源寻求确认。只有当 FlightAware 的 `cancelled` 标志与状态文本一致、或航空公司或机场来源确认时，取消才算已确认。一旦确认立即通知；在其保持未确认期间，后续运行持续检查。把这些静默规则（包括登机口时机与取消确认判据）和其他告警规则一起写入 cron 正文；cron 工作进程收不到本技能。

【评论】用"事件指纹 + 静默规则 + 分档阈值"来压制重复通知，是告警系统里典型的降噪设计：宁可少报，也不让监控变成骚扰。

Persist the fingerprint before sending a message. Add one Tracking activity
before sending a message. Do not add another activity or message for the same
fingerprint. Keep unchanged checks silent. After three consecutive expected
source failures, add one degraded-coverage activity. Do not treat a failed read
as evidence that nothing changed. Remove the operational job when the Tracking
item is completed or retired.

发送消息之前先持久化指纹。发送消息之前添加一条 Tracking 活动。同一指纹不要再添加活动或消息。无变化的检查保持静默。连续三次预期内的数据源失败后，添加一条降级覆盖（degraded-coverage）活动。不要把一次读取失败当作"什么都没变"的证据。Tracking 条目完成或退役时，移除该运行任务。

### Find flights in the air / 查找空中航班
Use `search` to find airborne flights by origin, destination, or area. Default to one page of results and ask before pulling more.

用 `search` 按出发地、目的地或区域查找空中的航班。默认只取一页结果，拉取更多之前先询问。

### Airport activity / 机场动态
Use `airport` for an airport's details, `airport-delays` for current delays, `airport-flights` for arrivals and departures, and `airport-weather` for conditions. `nearby-airports` finds airports around a location.

`airport` 查机场详情，`airport-delays` 查当前延误，`airport-flights` 查到达与出发，`airport-weather` 查天气状况。`nearby-airports` 查找某位置周边的机场。

### Airline activity / 航空公司动态
Use `operator` for an airline's details and `operator-flights` for its recent and scheduled flights.

`operator` 查航空公司详情，`operator-flights` 查其近期与计划航班。

### History / 历史
For past flights, use the `history` commands with an explicit date range. They cover flights, tracks, routes, airport activity, and an aircraft's last flight.

查询过去的航班用 `history` 命令并给出明确的日期范围。它们覆盖航班、轨迹、航线、机场动态和一架飞机的最近一次飞行。

### Predictions and schedules / 预测与时刻表
Use `foresight` for FlightAware's predicted status and positions, `schedules` for scheduled flights between two dates, and `disruptions` for cancellation and delay counts.

`foresight` 查 FlightAware 的预测状态与位置，`schedules` 查两个日期之间的计划航班，`disruptions` 查取消与延误计数。

## Other commands / 其他命令
The flows above cover the common cases. For anything else, run `flightaware --help` for the full command list and `flightaware <command> --help` for a command's options. This includes aircraft owner and type lookups, route and count queries, and advanced search syntax.

上面的流程覆盖了常见情况。其他情况，运行 `flightaware --help` 查看完整命令列表，运行 `flightaware <command> --help` 查看某命令的选项。这包括机主与机型查询、航线与计数查询以及高级搜索语法。

## Rules / 规则
- Use the airline's ICAO flight number when you can (`UAL123`, not `UA123`). If the user's flight number is ambiguous, resolve it with `canonical-flight` first.
  能用航空公司的 ICAO 航班号就用（`UAL123`，而不是 `UA123`）。若用户的航班号有歧义，先用 `canonical-flight` 解析。
- For "where is my flight", get the flight first, pick the right date and leg, then look up its position or track. Do not guess an `fa_flight_id`.
  "我的航班在哪"：先取航班，选对日期与航段，再查其位置或轨迹。绝不猜测 `fa_flight_id`。
- A cancellation is a high-impact terminal claim. Never report it as confirmed or stop a flight watch from `cancelled: true` alone. If the flight carries `muse_cancellation_evidence.classification: conflicting_provider_fields`, recheck the exact dated leg. If the fields still conflict, seek confirmation from an airline or airport source. Without it, say the cancellation is unconfirmed when answering the user directly, and keep a scheduled monitoring run silent until it is confirmed; keep monitoring either way. Only treat the cancellation as confirmed when FlightAware's boolean and status text agree, or an independent source confirms it.
  取消是高影响的终局性断言。绝不仅凭 `cancelled: true` 就把它报告为已确认或停止航班监控。若航班带有 `muse_cancellation_evidence.classification: conflicting_provider_fields`，复核带确切日期的航段。若字段仍然冲突，从航空公司或机场来源寻求确认。没有确认时，直接回答用户时要说取消尚未确认，且在确认之前计划监控运行保持静默；无论哪种情况都继续监控。只有当 FlightAware 的布尔值与状态文本一致、或独立来源确认时，才把取消视为已确认。
  【评论】"双源确认才宣布取消"体现了对高影响断言的证据分级：同一提供方内部字段冲突时，不允许单一数据源直接触发终局性结论。
- If FlightAware says a flight or aircraft is blocked or has no data, tell the user plainly and do not try to work around it.
  若 FlightAware 表示某航班或飞机被屏蔽或没有数据，如实告诉用户，不要试图绕过。
- Flight reads preserve AeroAPI's raw times and add semantic UTC/user-local fields such as `scheduled_gate_departure_at`, `estimated_takeoff_at`, `actual_landing_at`, and `scheduled_gate_arrival_at`. Prefer their `user_local` values in replies. Position samples similarly add `position_observed_at`.
  航班读取保留 AeroAPI 的原始时间，并添加语义化的 UTC/用户本地字段，如 `scheduled_gate_departure_at`、`estimated_takeoff_at`、`actual_landing_at` 与 `scheduled_gate_arrival_at`。回复中优先使用其 `user_local` 值。位置样本同样添加 `position_observed_at`。
- FlightAware does not cover airline policies, terminal maps, baggage, booking, or customer service. For those, use web search and make clear which details came from the web rather than FlightAware.
  FlightAware 不涵盖航空公司政策、航站楼地图、行李、订票或客服。这些用网页搜索，并明确哪些细节来自网页而非 FlightAware。
- Reply in plain language about the flight, not about how you looked it up. Do not show the user commands, ids, tokens, or raw status codes.
  用平实的语言回答航班本身，而不是你怎么查到的。不要向用户展示命令、ID、令牌或原始状态码。

## Limits / 限制
- You can't book travel, buy tickets, change a reservation, or contact an airline or airport.
  你不能预订旅行、购票、更改预订，或联系航空公司或机场。
- Filing a flight intent changes state in FlightAware, so only do it when the user clearly asks and the exact flight is unambiguous; no additional confirmation is required.
  提交航班意图会改变 FlightAware 中的状态，所以只在用户明确要求且确切航班无歧义时执行；无需额外确认。
