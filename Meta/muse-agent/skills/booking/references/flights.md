<!-- BILINGUAL-EN-ZH -->

# Flights / 航班

Use this reference for direct flight-booking requests. It does not turn the
request into a trip plan.

在用户直接提出机票预订请求时使用本参考。它不会把该请求转化为行程规划。

## Build the flight profile before asking / 在询问前构建乘机画像

Use authorized memory, email, airline accounts, wallet/card profiles, linked
financial data, and prior bookings to determine, when available:

利用已授权的记忆、电子邮件、航空公司账户、钱包/银行卡资料、关联的财务数据以及历史订单，在可用时确定以下信息：

- preferred and avoided airlines or alliances
  偏好及回避的航空公司或航空联盟
- aisle/window preference, preferred rows or zones, and tolerance for middle  
  seats
  靠过道/靠窗的偏好、偏好的排数或区域，以及对中间座的容忍度
- cabin and fare-class preferences, including whether basic economy is  
  acceptable
  舱位与票价等级偏好，包括是否接受基础经济舱
- preferred origin, destination, and alternate airports
  偏好的出发地、目的地及备选机场
- loyalty programs, status, and usable membership linkage
  常旅客计划、会员等级及可用的会员关联
- card products whose known airline benefits or transferable points matter;
  use Plaid only to identify linked products or relevant transactions, not as
  proof of benefits, reward rules, or point balances
  其已知航空权益或可转积分具有实际意义的银行卡产品；Plaid 仅用于识别已关联的产品或相关交易，不得作为权益、奖励规则或积分余额的证明
- airline credits, vouchers, companion certificates, upgrade instruments, and
  their expiry or restrictions
  航空公司报销额度（credit）、代金券、同行人凭证、升舱工具，以及它们的有效期或限制条件

When a loyalty program or status is relevant, mention the airline or alliance
and safe tier name before presenting the shortlist. Do not expose the membership
number.

当常旅客计划或会员等级与任务相关时，在展示候选清单前先提及航空公司或联盟以及可安全披露的等级名称。不得暴露会员号。

【评论】"可安全披露的等级名称"与"不得暴露会员号"体现了分层隐私设计：可以透露身份语境，但不透露可定位到个人的标识符。

## Minimum search facts / 最低搜索要素

Origin, destination, departure date, directionality, and traveler count are
required to price. Search for one adult and a one-way trip when "me" and a
single travel date make those assumptions natural, then label the assumptions.
Ask before search only when the missing fact creates a materially different
route or could select the wrong passenger.

机票询价需要出发地、目的地、出发日期、行程方向和出行人数。当用户以"我"为出行者且仅给出单一出行日期、使这些假设显得自然时，按一名成人和单程进行搜索，然后明确标注这些假设。仅当缺失的事实会导致路线实质不同、或可能选错乘客时，才在搜索前询问。

Resolve relative dates in the origin timezone. Check the calendar for hard
conflicts and infer realistic departure windows from the request and history,
but do not infer a different date.

相对日期按出发地时区解析。检查日历中是否存在硬性冲突，并根据请求与历史记录推断现实的出发时间窗，但不得推断出不同的日期。

## Search and route / 搜索与航线

Search live flight inventory with an available native provider such as
`duffel`. Also check the airline's authenticated site when loyalty status,
credits, companion benefits, upgrades, or cardholder pricing could materially
change the result. Use browser checkout when the native provider cannot apply
the relevant benefit or cannot complete the chosen itinerary.

使用可用的原生服务商（如 `duffel`）搜索实时航班库存。当常旅客等级、报销额度、同行人权益、升舱或持卡人价格可能实质改变结果时，还应查看航空公司已登录的网站。当原生服务商无法套用相关权益或无法完成所选行程时，使用浏览器结账。

Keep the native provider name internal. Do not mention Duffel in the message or
ask the user to choose between Duffel and the airline. Present airlines,
itineraries, fares, and user benefits. Introduce the airline website explicitly
only when signing in could unlock or verify points, status benefits, credits,
certificates, member pricing, or another relevant advantage, or when the
user asks to book direct.

对原生服务商的名称保密。不要在消息中提及 Duffel，也不要让用户在 Duffel 与航空公司之间做选择。呈现的是航空公司、行程、票价和用户权益。仅当登录可能解锁或验证积分、会员等级权益、报销额度、凭证、会员价格或其他相关优势，或用户明确要求直接向航空公司预订时，才明确引入航空公司网站。

【评论】这是典型的"白牌"渠道设计：预订实际上经过第三方服务商，但提示词要求对用户隐藏中介身份，只呈现航空公司界面。

### Search shortest useful journeys first / 优先搜索最短的有效行程

Unless the user explicitly prioritizes something else, optimize first for
elapsed journey time, including connections. When Duffel is available, the
`Search and compare` section in `/opt/hatch/skills/duffel/SKILL.md` is the
single authority for provider search ordering, pass count, filters, and airport
expansion. Use the returned result set to surface a meaningfully cheaper longer
option when one is available; do not require a second provider pass solely to
produce that comparison.

除非用户明确优先考虑其他因素，否则首先以总旅行时间（含中转）为优化目标。当 Duffel 可用时，`/opt/hatch/skills/duffel/SKILL.md` 中的 `Search and compare` 一节是服务商搜索顺序、搜索轮数、过滤条件与机场扩展的唯一权威。利用返回的结果集展示明显更便宜但更耗时的备选项（如存在）；不要仅为生成该对比而要求服务商进行第二轮搜索。

A targeted Duffel result of zero means only that Duffel did not return a
matching bookable offer; it is not evidence that the airline does not fly the
route.

Duffel 定向搜索结果为零仅说明 Duffel 未返回匹配的可订报价；这不能证明该航空公司不执飞此航线。

Use [ITA Matrix](https://matrix.itasoftware.com/search) as the route-coverage
backstop after the first Duffel search when:

在首次 Duffel 搜索之后，出现以下情况时，使用 [ITA Matrix](https://matrix.itasoftware.com/search) 作为航线覆盖的兜底：

- a requested, preferred, or otherwise expected carrier is missing
  缺失了用户要求、偏好或其他理应预期的承运商
- Duffel returns fewer than three credible itinerary shapes
  Duffel 返回的可信行程形态少于三种
- alternate airports, connection points, nearby dates, or a complex routing
  could materially improve the result
  备选机场、中转点、邻近日期或复杂航线可能实质改善结果

Matrix remains optional only when Duffel already provides credible coverage
and no relevant carrier or route appears missing. Matrix is not a booking
provider and its displayed fare is not proof that an itinerary remains
available for sale.

仅当 Duffel 已提供可信覆盖且未见相关承运商或航线缺失时，Matrix 才是可选项。Matrix 不是预订服务商，其显示的票价也不能证明该行程仍可售。

【评论】明确指出第三方比价工具显示的票价不代表实时可售，这是针对数据时效性可能造成误导的防御性条款。

For a promising Matrix result, retain enough detail to reproduce it: travel
dates, airport pair, marketing and operating carriers, flight numbers, local
times, cabin, stops, and fare or booking-code details when shown. Then locate
the same itinerary in live bookable inventory:

对于 Matrix 上有希望的结果，保留足以复现它的细节：出行日期、机场对、市场承运商与实际承运商、航班号、当地时间、舱位、经停，以及显示出的票价或订座代码细节。然后在实时可订库存中定位同一行程：

1. Check `duffel` when it is available.
   若 `duffel` 可用则先查询它。
2. If Duffel cannot reproduce the itinerary and price, check the operating or
   ticketing airline's website.
   若 Duffel 无法复现该行程及价格，则查看实际承运或出票航空公司的网站。
3. Compare the same passenger mix, cabin and fare conditions, currency, and
   full total, including unavoidable fees. A similar schedule or headline
   price is not an exact match.
   对比相同的乘客构成、舱位与票价条件、币种以及包含不可避免费用的全价总额。相近的时刻表或标价并不等于精确匹配。

Present the itinerary as bookable only at the current verified Duffel or
airline price. If neither source can reproduce it, use Matrix only as routing
evidence, explain that its fare could not be verified, and do not offer that
fare for selection or checkout. If the itinerary matches but the price does
not, show the current bookable price and clearly note the change.

仅在当前已验证的 Duffel 或航空公司价格下，才将该行程作为可预订项呈现。若两个来源都无法复现，则仅将 Matrix 用作航线证据，说明其票价无法验证，且不提供该票价供选择或结账。若行程一致但价格不一致，展示当前可订价格并清楚注明变化。

If Matrix exposes a useful carrier or nonstop itinerary that Duffel omitted,
check that airline's own website even when Duffel returned other acceptable
offers. Do not stop after presenting only the carriers that happened to occupy
Duffel's first result page.

若 Matrix 显示了 Duffel 遗漏的有用承运商或直飞行程，即使 Duffel 已返回其他可接受的报价，也应查看该航空公司的自有网站。不要在仅呈现恰好占据 Duffel 首页结果的承运商后就停止。

Provider rollout and skill rollout are separate. If `duffel` is absent or
returns its availability-gate response, do not retry it during the turn and do
not imply that native flight booking is available. Continue directly with the
airline through the browser. The rest of this flight flow still applies.

服务商上线与技能上线彼此独立。若 `duffel` 不存在或返回其可用性门控响应，不要在本轮对话中重试，也不要暗示原生机票预订可用。直接改用浏览器经由航空公司继续。本航班流程的其余部分仍然适用。

Compare cash and points only with live redemption data. Show taxes and fees on
an award, the transfer ratio and delay for transferable points, and the cash
value being displaced. Do not transfer points merely to discover availability.

仅在拥有实时兑换数据时才比较现金与积分。展示奖励票的税费、可转积分的转换比例与到账延迟，以及被放弃的现金价值。不要为了探测是否有票而转移积分。

Prefer a card based on concrete value for this itinerary. Relevant value can
include credit eligibility, free bags, lounge access, insurance, or bonus
earning. Do not prefer a card based on a generic points claim. Confirm that a
credit is valid for the operating or marketing carrier and has not expired.

基于对该行程的具体价值来推荐银行卡。相关价值可包括报销额度适用性、免费行李、贵宾厅使用权、保险或加速积分。不要基于泛泛的积分宣传推荐银行卡。确认报销额度对实际承运或市场承运商有效且未过期。

## Rank on the whole journey / 按全程排序

Lead with the shortest reasonable itinerary, normally preferring nonstop over
a connection. Then include a meaningfully cheaper longer option and other
distinct tradeoffs such as the best preferred-airline or flexible fare. Do not
fill the shortlist with near-duplicate cheap results while faster or nonstop
choices exist.

以最短且合理的行程领衔，通常直飞优先于中转。随后纳入明显更便宜但更耗时的选项，以及其他有区分度的权衡项，例如最佳的偏好航空公司选项或灵活票价。当存在更快或直飞的选项时，不要用近乎重复的低价结果填满候选清单。

When the user has status, favor eligible flights on that airline or alliance
when the schedule, fare, and operating carrier make the status benefits useful.
Keep a materially faster or cheaper non-status option visible and state the
tradeoff; status is a ranking advantage, not an instruction to overpay or accept
a poor itinerary.

当用户拥有会员等级时，若时刻、票价与实际承运商能使等级权益发挥作用，则优先该航空公司或联盟的合资格航班。同时保留实质更快或更便宜的无等级选项并说明权衡；等级是排序上的加分项，而不是要求多付钱或接受糟糕行程的指令。

Compare:

对比以下维度：

- full cash total or points plus cash, including bags and seat fees
  全额现金总价或积分加现金，含行李费与选座费
- operating airline and flight number
  实际承运航空公司与航班号
- departure and arrival airports, terminals when relevant, and local times
  出发与到达机场（必要时含航站楼）及当地时间
- next-day arrival markers
  次日到达标记
- stops, connection airports, connection length, and total duration
  经停、中转机场、中转时长与总时长
- cabin, fare brand, seat availability, and upgrade state
  舱位、票价品牌、座位可得性与升舱状态
- baggage allowance
  行李额度
- refund, change, same-day-change, and credit-expiry rules
  退票、改签、当日改签与报销额度到期规则
- loyalty earnings and material card/status benefits
  常旅客累积以及有实质意义的银行卡/会员等级权益

Reject unsafe or unrealistic connections and airport changes. Do not call a
fare "cheapest" when required bags or seats make it more expensive.

排除不安全或不现实的中转以及更换机场的方案。当必需的行李或座位使其总价更高时，不得将该票价称为"最便宜"。

If preferences conflict, explain the decisive tradeoff: for example, the
preferred airline costs more but uses the user's credit and avoids a middle
seat.

若偏好之间相互冲突，解释起决定作用的权衡：例如偏好航空公司更贵，但可使用用户的报销额度并避免中间座。

## Flight comparison response / 航班对比的响应方式

Every user-visible comparison of live flight options must use `widget.create`
with `kind: "list"` and flight rows whenever `widget.create` is present in the
current tool set. Tool availability is the capability signal: do not choose a
Markdown table because client support is uncertain, the schema is lengthy, the
provider output is large, or Markdown is easier. Attempt the structured list
before writing the response. Use the Markdown fallback only when
`widget.create` is absent or one valid `widget.create` call returns an explicit
unsupported or rendering error.

只要当前工具集中存在 `widget.create`，所有面向用户的实时航班选项对比都必须使用带 `kind: "list"` 和航班行的 `widget.create`。工具可用性即能力信号：不得因为客户端支持不确定、schema 冗长、服务商输出体量大或 Markdown 更省事而改用 Markdown 表格。必须在撰写响应之前先尝试结构化列表。仅当 `widget.create` 不存在，或一次有效的 `widget.create` 调用返回明确的不支持或渲染错误时，才使用 Markdown 兜底。

Do not delegate a native-provider flight comparison to a generic research or
browser worker. Keep its search and widget creation in the user-facing agent.
If a worker already searched, require its unchanged response path and candidate
pointers; never relay its table. Confirm a successful `widget.create` before
responding. Invalid arguments require a corrected retry, not Markdown.

不要把原生服务商的航班对比委托给通用研究或浏览器工作进程。其搜索与小组件创建必须保留在面向用户的智能体中。若工作进程已完成搜索，要求其返回未更改的响应路径与候选指针；绝不转发表格。响应之前必须确认 `widget.create` 已成功。参数无效时应纠正后重试，而不是改用 Markdown。

Do not use a shopping widget or a duplicate option picker. Search the complete
requested itinerary together and put complete offers for the same itinerary
and dates in one comparison list. Show the best four distinct matching trips,
or every match when fewer than four exist.

不要使用购物小组件或重复的选项选择器。将完整的请求行程作为整体一起搜索，并把相同行程与日期的完整报价放入同一个对比列表。展示最佳的四个互不相同的匹配行程；若匹配项不足四个，则全部展示。

The widget requires the complete normalized provider offer. Do not pipe or
truncate the search response, and do not replace it with a hand-built summary.
Redirect every search directly into a distinct temporary JSON file on its first
attempt, verify its exit status and response, and keep the saved JSON unchanged
until widget creation succeeds. If an earlier search was piped, truncated, or
reduced by a custom parser, do not repeat it in the same turn; ask the user to
continue in a new turn so the complete response can be retained.

小组件需要完整且规范化的服务商报价。不得通过管道传输或截断搜索响应，也不得用手工摘要替代。每次搜索在首次尝试时就直接重定向到独立的临时 JSON 文件，校验其退出状态与响应，并在小组件创建成功之前保持已保存的 JSON 不变。若先前的搜索被管道传输、截断或被自定义解析器削减，不要在同一轮中重复执行；应请用户在新的一轮中继续，以便保留完整响应。

For native-provider JSON, create one row per selected complete offer with
`type: "flight"` and point directly into the unchanged search file:

对于原生服务商的 JSON，为每个选定的完整报价创建一行 `type: "flight"`，并直接指向未更改的搜索文件：

```json
{
  "flight_details_file": "/tmp/flight-search.json",
  "json_pointer": "/offers/0"
}
```

Use the original offer's zero-based array index. Do not copy the offer JSON into
the tool call, rewrite timestamps, remove fields, or generate a separate widget
payload file. The runtime reads the selected object, enriches its presentation,
and stores the complete flight data without retaining the path or pointer.

使用原始报价的从零开始的数组索引。不要把报价 JSON 复制进工具调用，不要改写时间戳、删除字段，也不要另行生成小组件载荷文件。运行时读取所选对象、丰富其呈现并保存完整航班数据，且不保留该路径或指针。

For browser results, use inline `data: {"flight_data": FlightData}` or save that
same verified flight object and use `flight_details_file`. Preserve the complete
offer: private mapping `id`, passengers, full-party `price` and `currency`, every
slice and segment, airport-local timestamps, source durations, carrier and
operating-carrier details, flight numbers, cabin, fare brand, bags, stops, and
three-valued fare conditions. Do not calculate elapsed durations by subtracting
timestamps from different airport timezones. Do not expose the offer id.

对于浏览器结果，使用内联的 `data: {"flight_data": FlightData}`，或保存同一个已验证的航班对象并使用 `flight_details_file`。保留完整报价：私有映射 `id`、乘客、全体乘客的 `price` 与 `currency`、每一段航程（slice）与每一航节（segment）、机场当地时间戳、原始时长、承运商与实际承运商细节、航班号、舱位、票价品牌、行李、经停以及三值票价条件。不要用不同机场时区的时间戳相减来计算耗时。不得暴露报价 id。

Present the widget once with `present_now: true`, or include its returned embed
token unchanged when it was created without immediate presentation. Do not
repeat the rows in prose or a Markdown table.

以 `present_now: true` 一次性呈现小组件，或在创建时未立即呈现的情况下原样附上其返回的嵌入令牌。不要在正文或 Markdown 表格中重复这些行。

After presenting a flight widget, keep the accompanying message to a short
acknowledgement. Add details only to answer something the user explicitly
asked or explain a material mismatch with their request. Do not repeat
information available in the widget, add a "Details" section, or ask the user
to type their flight selection. Let the user select flights through the widget.

呈现航班小组件后，随附消息只做简短确认。补充细节仅用于回答用户明确询问的内容，或解释与用户请求的重大不符。不要复述小组件中已有的信息，不要添加"Details"部分，也不要让用户以打字方式输入航班选择。让用户通过小组件选择航班。

Choose the list action from the current flow:

根据当前流程选择列表动作：

- For direct booking or a trip that is `ready to book`, omit `flight_action`.
  The runtime supplies its default booking CTA and derives an identifying
  booking reply from the complete offer.
  对于直接预订或处于 `ready to book` 状态的行程，省略 `flight_action`。运行时会提供默认的预订 CTA，并从完整报价推导出可识别的预订回复。
- When Travel Planning hands off a `planning only` trip, set the list-level
  `flight_action` to `{"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`.
  Return that selected itinerary to the plan so Travel can offer the separate
  choice to book now or keep planning and book later.
  当旅行规划移交一个 `planning only` 的行程时，将列表级 `flight_action` 设为 `{"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`。把所选行程返回给计划，使 Travel 能另行提供"现在预订"或"继续规划、稍后预订"的选择。

Do not author a row-level `response_message`. The runtime appends the exact
complete itinerary and total to either response. Treat the interaction as
selection of the complete offer, not as approval to purchase it. Add
`flight_link` only for a verified details URL.

不要自行撰写行级 `response_message`。运行时会把确切的完整行程与总价附加到任一回复之后。将该交互视为对完整报价的选择，而不是购买的批准。仅在已验证的详情 URL 时才添加 `flight_link`。

【评论】"选择不等于购买授权"把点击行为与支付授权明确区分开，属于防止误触下单的安全条款。

Include the entire offer in one row during comparison and after selection: one
slice for a one-way trip, two for a round trip, and every requested slice for a
multi-city trip. Do not split or duplicate a round trip into outbound and return
rows, pair legs from different offers, or invent per-leg prices. Keep separately
ticketed one-way offers separate and explicit.

在对比阶段与选择之后，都把完整报价放入同一行：单程包含一个航程，往返包含两个，多城市则包含所有请求的航程。不要把往返拆分或复制为去程行与回程行，不要拼接来自不同报价的航节，也不要编造按航节计的价格。对分别出票的单程报价保持其独立与明确。

If `widget.create` is absent, or its valid invocation explicitly fails because
the client cannot render the native flight list, follow
`/opt/hatch/skills/booking/references/presentation.md` and use its compact
flight Markdown table. Do not infer lack of support without attempting the tool
when it is available. In either format,
every itinerary must show all of:

若 `widget.create` 不存在，或其有效调用因客户端无法渲染原生航班列表而明确失败，则遵循 `/opt/hatch/skills/booking/references/presentation.md` 并使用其紧凑的航班 Markdown 表格。当工具可用时，不得未经尝试就推断其不受支持。无论哪种格式，每个行程都必须完整展示以下信息：

- full party price and currency, labeled as one-way or round-trip
  全体乘客价格与币种，并标注单程或往返
- each leg's local departure and arrival times and airports
  每一航节的当地出发与到达时间及机场
- each leg's total elapsed duration
  每一航节的总耗时
- nonstop or the exact stop count
  直飞或确切的经停次数
- every connection airport and layover duration
  每个中转机场及中转时长
- marketing carrier, operating carrier when different, and flight numbers
  市场承运商、实际承运商（如不同）与航班号
- cabin and fare brand when stated
  舱位与票价品牌（如有标注）

Do not defer price, duration, stops, or layovers to a follow-up. Calculate
each layover from the adjacent provider segment timestamps at the same airport;
if the timestamps are insufficient or ambiguous, write `layover length not
stated` rather than omitting it or guessing. Keep baggage and change/refund
terms in the native widget. For the Markdown fallback, keep shared terms in
one short `Details:` line after the table and option-specific differences in
their rows. Ask the user to reply with the option label only when using the
Markdown fallback.

不得把价格、时长、经停或中转留待后续补充。每个中转时长都用同一机场相邻服务商航节的时间戳计算；若时间戳不足或有歧义，写 `layover length not stated`，而不是省略或猜测。行李与退改条款保留在原生小组件中。使用 Markdown 兜底时，把共通条款放在表格后的一行简短 `Details:` 中，把各选项特有的差异写在其所在行。仅在使用 Markdown 兜底时，才请用户以选项标签回复。

Do not use HTML or present the same inventory in a second format.

不要使用 HTML，也不要以第二种格式重复呈现同一批库存。

## Prepare and book / 准备并预订

Price the selected offer again immediately before final review. Surface any
fare, seat, baggage, airport, schedule, or operating-carrier change.

在最终确认前立即对所选报价重新询价。呈现票价、座位、行李、机场、时刻或实际承运商的任何变化。

When known airline status, points, credits, or certificates could apply, ask
one combined decision before choosing the purchase channel: whether to sign in
to the airline website and compare cash and points, check cash only, or check
points only. Recommend comparing both when time permits. Use the signed-in
airline path to verify member benefits and redemption availability; do not
infer a points balance from status or transfer points without separate explicit
approval.

当已知的航空公司会员等级、积分、报销额度或凭证可能适用时，在选择购买渠道之前先询问一个合并的决策：是登录航空公司网站对比现金与积分、仅查现金，还是仅查积分。时间允许时建议两者都比较。使用已登录的航空公司路径验证会员权益与兑换可用性；不得从会员等级推断积分余额，未经另行明确批准不得转移积分。

At final review include traveler count, exact itinerary, cabin and fare brand,
assigned or selected seats, bags, total, points/credits applied, amount charged,
and fare rules. Ask before a points transfer, certificate use, or paid upgrade.

最终确认时须包含出行人数、确切行程、舱位与票价品牌、已分配或已选座位、行李、总价、已应用的积分/报销额度、扣款金额与票价规则。积分转移、凭证使用或付费升舱之前须先询问。

Follow the native provider skill for checkout fields. Payment
credentials and account passwords remain secure-surface-only. Do not expose
native-provider commands, offer ids, or validation fields.

结账字段遵循原生服务商技能的规定。支付凭据与账户密码仍然仅限安全界面使用。不得暴露原生服务商命令、报价 id 或校验字段。

【评论】"仅限安全界面使用"（secure-surface-only）意味着敏感凭据只允许出现在受保护输入通道，避免进入可被展示、记录的对话文本。

Verify the booking with the airline/provider confirmation and passenger name
record. Return the route, dates, flight numbers, seats, total, and safe
confirmation reference. Do not display full loyalty, passport, Known Traveler,
redress, or payment numbers.

用航空公司/服务商的确认信息与乘客订座记录（PNR）核验预订。返回航线、日期、航班号、座位、总价以及可安全展示的确认编号。不得展示完整的常旅客号、护照号、Known Traveler 号、申诉号（redress）或支付号码。
