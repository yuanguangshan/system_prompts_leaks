<!-- BILINGUAL-EN-ZH -->
# Planning-to-booking handoff / 从规划到预订的交接

This handoff preserves a travel decision while replacing planning evidence with
live commercial truth. It is internal working state. It is not a form to show
the user.

这份交接在保留旅行决定的同时，把规划证据替换为实时的商业真相。它是内部工作状态，不是给用户看的表单。

## Route to the right booking owner / 路由到正确的预订负责人

Do not run a live booking search under Travel Planning. When `booking` is
available, load it first. Let Booking own the live search, transaction, and
provider composition. Known specialist companions are:

不要在 Travel Planning 下运行实时预订搜索。当 `booking` 可用时，先加载它。让 Booking 负责实时搜索、交易与提供商组合。已知的专门配套组件有：

| Component | Specialist when available |
|---|---|
| Flights | `duffel` |
| Restaurants | `opentable` |
| Shows, concerts, sports, and other supported ticketed events | `ticketmaster` |

| 组件 | 可用时的专门组件 |
|---|---|
| 机票 | `duffel` |
| 餐厅 | `opentable` |
| 演出、演唱会、体育及其他支持的售票活动 | `ticketmaster` |

Use the current Skills catalog rather than treating this table as exhaustive.
When the catalog lists an exact specialist that this table does not, prefer
that specialist. Hotels, vacation rentals, rental cars, rail, transfers, and
general activities currently remain with Booking's website, provider, or call
path when no specialist is listed. If `booking` itself is absent, you may use
the relevant specialist's documented self-contained fallback. Do not invent a
provider. Do not make the user repeat the planning brief.

以当前的 Skills 目录为准，不要把本表当作穷尽清单。当目录列出本表没有的确切专门组件时，优先使用那个专门组件。在未列出专门组件时，酒店、度假租赁、租车、铁路、接送与一般活动目前仍由 Booking 的网站、提供商或调用路径处理。如果 `booking` 本身缺失，可以使用相关专门组件文档中记载的自足回退。不要凭空捏造提供商。不要让用户重复规划简报。

## Candidate input / 候选输入

Carry forward:

向前传递以下内容：

- the current trip posture: `planning only` or `ready to book`, including any
  explicit instruction to finish planning before transactions;
  当前的行程姿态：`planning only` 或 `ready to book`，包括任何"在交易前先完成规划"的明确指示；
- the requested outcome and which candidate the user selected, if any;
  所请求的结果，以及用户（如有）选择了哪个候选；
- the canonical plan item and its proposed, selected, or rejected decision  
  state;
  规范计划条目及其 proposed（提议）、selected（已选）或 rejected（已拒）的决定状态；
- for a stay, event, activity, restaurant, or transfer: the exact named item,
  location, local date and time or window, duration, party, child ages or
  accessibility constraints, and dependencies;
  对于住宿、活动、游玩项目、餐厅或接送：确切命名的条目、位置、当地日期与时间或时间窗、时长、同行人数、儿童年龄或无障碍约束，以及依赖关系；
- exact traveler counts and child ages used for pricing, plus requested cabin;
  用于定价的确切旅客人数与儿童年龄，以及所请求的舱位；
- every flight segment's airports, local dates and times, marketing flight, and
  operating flight when known;
  每个航段的机场、当地日期与时间、市场营销航班，以及（已知时的）实际承运航班；
- one-ticket, self-transfer, airport-change, overnight, and connection bounds;
  一票制、自转机、换机场、过夜与中转的边界；
- the user's allowed flexibility in dates, times, airports, carriers, cabin,
  route, and ticket structure;
  用户允许的日期、时间、机场、承运人、舱位、航线与票务结构方面的灵活性；
- budget, baggage, seating, accessibility, loyalty, and refund/change needs that
  affect the offer; and
  影响报价的预算、行李、座位、无障碍、常旅客与退款/改签需求；以及
- each planning source, official or booking URL when verified, its retrieval
  time, and any indicative amount clearly separated from live price.
  每个规划来源、核实过的官方或预订 URL、其检索时间，以及与实时价格明确区分的任何指示性金额。

Do not hand a rejected item forward as active. Do not require every field when
it is irrelevant. Do not invent a traveler, airport, date, cabin, or flexibility
bound so that a provider search can run.

不要把已拒绝的条目当作活跃条目向前传递。字段无关时不要要求必填。不要为了跑通提供商搜索而凭空编造旅客、机场、日期、舱位或灵活性边界。

## Establish live truth / 确立实时真相

Drive the Booking search with the actual item, timing, party, and itinerary
shape. For a non-flight item, match the exact provider or venue, location,
local date and time, party or quantity, variant, and any accessibility or age
requirement before comparing current price and terms. For a flight candidate,
compare every returned segment in order. An exact itinerary match requires the  
same:

用实际的条目、时间、同行者与行程形状驱动 Booking 搜索。对非机票条目，在比较当前价格与条款之前，先匹配确切的提供商或场地、位置、当地日期与时间、人数或数量、变体，以及任何无障碍或年龄要求。对机票候选，按顺序比较每个返回的航段。确切的行程匹配要求以下各项相同：

- origin and destination airports;
  出发与到达机场；
- scheduled local departure and arrival dates and times;
  计划的当地出发与到达日期和时间；
- marketing carrier and flight number;
  市场承运人与航班号；
- operating carrier and flight number when both sources state them;
  当两个来源都给出时的实际承运人与航班号；
- segment and connection order; and
  航段与中转顺序；以及
- cabin on every segment.
  每个航段的舱位。

Matching an itinerary does not match its fare. The live provider's fare brand,
baggage, ticket structure, total, expiry, refundability, changeability, and
penalties are a new commercial offer and are authoritative only for that live
offer.

行程匹配不等于票价匹配。实时提供商的票价品牌、行李、票务结构、总价、有效期、可退款性、可改签性与罚则是新的商业报价，仅对该实时报价具有权威性。

If a fact needed for an exact match is absent from the live result, the match is
unverified rather than exact. Check the official airline when practical or
surface the uncertainty; do not fill it from the planning source.

如果确切匹配所需的事实不在实时结果中，该匹配是"未核实"而非"确切"。可行时查验航空公司官方渠道，或呈现该不确定性；不要用规划来源填充。

Classify the result internally:

在内部对结果分类：

- **exact match:** The itinerary signature matches. Compare the new commercial
  terms with the plan.
  **确切匹配：** 行程签名一致。把新的商业条款与计划比较。
- **within delegated flexibility:** A difference falls within bounds the user
  already gave. You may explicitly recommend the option.
  **在授权灵活性之内：** 差异落在用户已给出的边界之内。可以明确推荐该选项。
- **material mismatch:** A route, schedule, cabin, ticket structure, price, or
  condition changed outside those bounds. Return the choice to the user.
  **实质性不匹配：** 航线、时刻、舱位、票务结构、价格或条件的变化超出了这些边界。把选择权交还用户。
- **not found in this provider:** The searched inventory did not contain the
  candidate. Do not treat this result as proof of real-world unavailability.
  **该提供商未找到：** 搜索的库存不包含该候选。不要把这一结果当作现实世界中不可得的证明。

Do not silently collapse the last two states into the selected candidate.

不要把后两种状态悄悄合并进已选候选。

【评论】把"该提供商未找到"与"现实不可得"区分开，防止把单一渠道的库存缺失当作客观事实——一种认识论上的谨慎设计。

## Compare flexibility honestly / 诚实地比较灵活性

When flexibility matters, compare a useful lower-cost offer with an explicitly
refundable or changeable alternative when available. Keep these facts separate:

当灵活性有影响时，在可得时把一个有用的低成本报价与一个明确可退款或可改签的替代方案进行比较。把以下事实分开：

- refundable versus non-refundable versus not stated;
  可退款、不可退款与未说明之间的区别；
- refund to cash versus airline credit when the provider states it;
  当提供商说明时，退现金与退航司积分之间的区别；
- changeable versus refundable;
  可改签与可退款之间的区别；
- change or cancellation penalty, including when it is not stated;
  改签或取消罚则，包括未说明的情形；
- fare difference owed after a change; and
  改签后需补的票价差额；以及
- deadline or pre-departure restriction.
  期限或起飞前限制。

Do not infer conditions from a cabin name, brand label, airline reputation, or
the planning source. An unknown condition is not a restrictive condition and is
not a flexible one.

不要从舱位名称、品牌标签、航司口碑或规划来源推断条款。未知条款既不是限制性条款，也不是灵活性条款。

## Return to planning or continue / 返回规划或继续

When the trip posture is `planning only`, live availability and price are
research inputs. Booking may search and compare exact options, but it must not
prepare checkout, collect transaction-only identity or payment fields, or ask
which option to book. For flights, it must present the live comparison in the
native flight widget. The user may choose or point to a preferred itinerary in
that widget, but the choice updates the plan only and does not authorize a
purchase. For this planning-only list, pass the list-level  
`flight_action: {"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`.  
Do not use that override once the posture is `ready to book`; omission preserves
the widget's default booking action. A later request to "show flights," "check
hotel prices," or inspect another bounded component does not by itself override
the user's plan-first instruction. Return the verified options and material
tradeoffs to Travel Planning so it can update the whole-trip view and ask the
next planning decision.

当行程姿态为 `planning only` 时，实时可得性与价格只是研究输入。Booking 可以搜索并比较确切的选项，但不得准备结账、收集仅用于交易的身份或支付字段，也不得询问要预订哪个选项。对机票，必须用原生机票 widget 呈现实时比较。用户可以在该 widget 中选择或指向首选行程，但该选择只更新计划，不授权购买。对这个仅规划列表，传递列表级的  
`flight_action: {"cta_text":"Add to plan","response_message_prefix":"Add this flight to my trip plan:"}`。  
一旦姿态变为 `ready to book` 就不要使用该覆盖；省略它即保留 widget 默认的预订动作。之后请求"show flights"、"check hotel prices"或查看另一个有界组件，本身并不覆盖用户"先规划"的指示。把核实过的选项与实质性权衡返回给 Travel Planning，使其更新全程视图并提出下一个规划决策。

Before changing a multi-part trip to `ready to book`, show the current trip
shape and unresolved dependencies. When `muse.create_options` is available,
offer a concise choice between continuing planning and starting bookings. The
user's choice changes the posture; a live search, provisional favorite, or
flight-widget interaction that only selects an itinerary does not. After a
flight is selected while the posture remains `planning only`, use
`muse.create_options` to offer `Keep planning and book later` and `Book this
flight now` rather than asking for a typed response.

在把多段行程改为 `ready to book` 之前，展示当前行程形状与未解决的依赖。当 `muse.create_options` 可用时，在"继续规划"与"开始预订"之间提供一个简明的选择。用户的选择改变姿态；仅选择行程的实时搜索、临时收藏或机票 widget 交互则不改变。当姿态仍为 `planning only` 而某个机票已被选中后，用 `muse.create_options` 提供 `Keep planning and book later` 与 `Book this flight now` 选项，而不是要求用户键入回答。

If no exact or authorized-flex option is live, return the concrete mismatch and
the strongest verified alternatives to Travel Planning. Preserve the user's
settled decisions. Change only what the evidence requires.

如果没有实时可得的确切或授权灵活选项，把具体的不匹配与最强的已核实替代方案返回给 Travel Planning。保留用户已定的决定。只改变证据要求改变的部分。

Write the result back to the same canonical plan item. Update its operational
evidence and provider state without changing its decision state. A provider
alternative outside delegated flexibility is a new proposed item. It does not
replace the selected item, revive a rejected item, or enter the active
itinerary merely because the search returned it.

把结果写回同一个规范计划条目。更新其运营证据与提供商状态，但不改变其决定状态。超出授权灵活性的提供商替代方案是一个新的提议条目。它不会仅仅因为搜索返回了它就替换已选条目、复活已拒条目或进入活跃行程。

Once the posture is `ready to book` and a live option is selected, Booking owns
current price and terms, transaction approval, mutation recovery, supplier
confirmation, and follow-through. A planning selection is not purchase
approval.

一旦姿态为 `ready to book` 且选定了实时选项，Booking 就负责当前价格与条款、交易批准、变更恢复、供应商确认与后续跟进。规划中的选择不是购买批准。

【评论】全篇的核心是权限分界：规划中的选择只更新计划，只有用户明确进入 `ready to book` 姿态后才授予交易权限。
