<!-- BILINGUAL-EN-ZH -->

# Flight itinerary discovery / 航班行程探索

Use itinerary discovery to find a good route shape before live-offer work when
dates, airports, stops, or the ordering of several cities is open. For a
straightforward route with bounded dates and travelers, skip this step. Let
Booking own the live-offer search.

当日期、机场、经停或多城市顺序尚未确定时，先使用行程探索找到一个好的路线形态，再进入实时报价环节。对于日期和旅客都已确定的简单路线，跳过此步骤。把实时报价搜索交给 Booking。

## When ITA Matrix helps / ITA Matrix 适用的场景

ITA Matrix helps you explore:

ITA Matrix 帮助你探索：

- open-jaw and multi-city orders;
  缺口程（open-jaw）和多城市顺序；
- flexible dates or nearby origin and destination airports;
  弹性日期或出发/目的地附近机场；
- nonstop versus connection and stopover tradeoffs;
  直飞与中转、经停之间的权衡；
- airline, alliance, connection-airport, or time-of-day constraints; and
  航空公司、联盟、中转机场或时段限制；以及
- schedule patterns that affect lodging, transfers, or a fixed event.
  影响住宿、中转或固定活动的时刻安排。

ITA Matrix is a discovery interface. Its result is not a hold, a live booking
offer, or proof that Duffel or an airline website can sell the same itinerary
and fare now. Do not make it a mandatory detour for every flight request.

ITA Matrix 是一个探索界面。其结果不是占位，不是实时预订报价，也不能证明 Duffel 或航空公司网站现在能以同样的行程和票价出售。不要把它变成每个航班请求的强制性绕路。

【评论】该文件明确区分"探索界面的展示价"与"可售的实时报价"，防止把规划工具的指示性价格当作可成交价格，是预订类技能中常见的期望管理条款。

## Use a read-only browser task / 使用只读浏览器任务

Use `browser.spawn_task` for the official ITA Matrix search at
`https://matrix.itasoftware.com/search` because it is an interactive website
workflow. Give the browser task a self-contained brief. The browser task does
not inherit the conversation. Include:

使用 `browser.spawn_task` 在 `https://matrix.itasoftware.com/search` 上执行官方 ITA Matrix 搜索，因为这是一个交互式网站流程。给浏览器任务一份自包含的简报。浏览器任务不继承对话。简报要包含：

- exact origins and destinations, plus allowed nearby airports;
  确切的出发地和目的地，以及允许的附近机场；
- dates or date ranges and the trip order;
- dates or date ranges and the trip order;
  日期或日期范围与行程顺序；
- traveler counts and requested cabin;
  旅客人数和请求的舱位；
- maximum connections and any airline, airport, time, overnight, self-transfer,
  or separate-ticket restrictions;
  最大中转次数，以及任何航空公司、机场、时间、过夜、自转机或分开出票限制；
- the comparison objective, such as shortest practical route, best arrival, or
  a useful low-price pattern; and
  比较目标，例如最短可行路线、最佳到达时间或有用的低价模式；以及
- an instruction to research only, make no purchase, and return normalized
  candidate details with the page's retrieval time.
  只做调研、不做购买、返回规范化候选详情并附页面检索时间的指示。

Do not include traveler names, documents, payment details, or other personal
data that schedule discovery does not need. An ITA Matrix research session is
not an airline checkout session.

不要包含时刻探索用不到的旅客姓名、证件、支付详情或其他个人数据。ITA Matrix 调研会话不是航空公司的结账会话。

## Capture a stable itinerary signature / 记录稳定的行程签名

For each candidate, record every segment in order:

对每个候选，按顺序记录每个航段：

- local departure date and time;
  当地出发日期和时间；
- origin and destination airport codes;
  出发地和目的地机场代码；
- marketing carrier and flight number;
  市场承运人和航班号；
- operating carrier and flight number when different;
  实际承运人和航班号（与市场承运人不同时）；
- local arrival date and time, including next-day arrival;
  当地到达日期和时间，包括次日到达；
- connection airport and layover duration;
  中转机场和中转时长；
- cabin, and whether cabin varies by segment; and
  舱位，以及舱位是否因航段而异；以及
- whether the result appears to be one fare/ticket or that fact is unknown.
  该结果是否看起来为同一票价/同一张票，或该事实未知。

Also retain the search inputs, source, retrieval time, and any displayed fare
or fare construction. A displayed amount is an indicative planning signal only.
Keep it separate from a later live total. Do not describe a fare basis, booking
class, or ticketing carrier as preserved unless the live provider returns the
same commercial terms.

同时保留搜索输入、来源、检索时间以及任何显示的票价或票价构成。显示金额只是指示性的规划信号。要与之后的实时总价分开保存。除非实时提供方返回了相同的商业条款，否则不要声称票价基础、预订舱位或出票承运人得以保留。

Flight number alone is not an itinerary identity. Codeshares, operating
carriers, repeated flight numbers, local-date boundaries, airport swaps, and
schedule changes can all make a superficially similar result different.

仅凭航班号不构成行程标识。代码共享、实际承运人、重复的航班号、当地日期分界、机场变更和时刻变更都可能使表面上相似的结果实际不同。

## Produce a short candidate set / 产出简短的候选集

Return two to four candidates that expose real tradeoffs. Do not return a raw
result dump. For each, summarize:

返回两到四个能展现真实权衡的候选。不要倾倒原始结果。对每个候选，总结：

- the complete route and important local times;
  完整路线和重要的当地时间；
- total journey time, connections, overnight or airport-change risk;
  总行程时间、中转、过夜或换机场风险；
- why it fits the trip better or worse;
  它为何更适合或不适合这次旅行；
- the indicative displayed amount, currency, and retrieval time when present;  
  and
  显示的指示性金额、币种和检索时间（存在时）；以及
- what remains unknown until live booking verification.
  在实时预订验证之前仍未知的信息。

Do not rank an option as cheaper when fees, ticket structure, or the relevant
passenger total is unclear. Do not call a route refundable or changeable from a
generic cabin or fare-brand label.

当费用、票务结构或相关旅客总价不明确时，不要把某个选项排为更便宜。不要凭通用的舱位或票价品牌标签断言某条路线可退款或可改签。

## Failure and fallback / 失败与回退

A CAPTCHA, bot wall, broken date picker, incomplete result, or lost browser
session is a source failure. It is not evidence that no flight exists. Do not
loop on the same broken interaction. Continue with a live provider search when
the trip is bounded enough, use another reliable schedule source when planning
still needs it, or explain the narrow evidence gap.

验证码、机器人屏障、损坏的日期选择器、不完整的结果或丢失的浏览器会话都属于来源故障。它不能证明不存在航班。不要在同一个损坏的交互上循环。在行程要素足够明确时改用实时提供方搜索；在规划仍需要时使用另一个可靠的时刻来源；或向用户说明这一狭小的证据缺口。

When the user selects a candidate, or when price or availability is needed to
choose, continue with
`/opt/hatch/skills/travel-planning/references/booking-handoff.md`. Do not send
the user away to reproduce the ITA Matrix search.

当用户选定候选，或需要价格和可订性来做选择时，继续执行
`/opt/hatch/skills/travel-planning/references/booking-handoff.md`。不要让用户自己去复现 ITA Matrix 搜索。
