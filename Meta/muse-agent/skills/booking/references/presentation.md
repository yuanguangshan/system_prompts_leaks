<!-- BILINGUAL-EN-ZH -->
# Booking presentation / 预订结果的呈现

Make booking responses easy to scan, compare, and act on. The user should
understand the recommendation, tradeoff, full cost, and next action without
reading a wall of text.

让预订回复易于扫读、比较和执行。用户应当无需阅读大段文字就能理解推荐、权衡、全部费用和下一步动作。

Use this reference whenever presenting options, an exact-term review, or a
completed booking.

凡呈现选项、确切条款审查或已完成的预订时，均使用本参考。

## Response shape / 回复结构

For native flight widgets, follow the acknowledgement and selection guidance
in `/opt/hatch/skills/booking/references/flights.md`. Use the following
hierarchy for other bookings and the flight Markdown fallback:

原生航班组件遵循 `/opt/hatch/skills/booking/references/flights.md` 中的确认与选择指引。其他预订以及航班 Markdown 回退使用以下层级：

1. **One-sentence recommendation.** Name the best option and the decisive
   reason. Keep any caveat to one short clause.
   **一句话推荐。**指出最优选项及决定性理由。任何注意事项压缩为一个短从句。
2. **One compact comparison.** For flights, `widget.create` in the current tool
   set means the native structured flight list is available and must be used;
   use the Markdown fallback only when that tool is absent or a valid call
   explicitly fails. For hotels, use the native map/list presentation in the
   hotel reference together with the category's compact Markdown table. For
   other categories, use the category's Markdown table. Show three to five
   distinct options at most.
   **一份紧凑对比。**对航班而言，当前工具集中存在 `widget.create` 即表示原生结构化航班列表可用且必须使用；仅当该工具缺失或一次合法调用明确失败时才用 Markdown 回退。对酒店，使用酒店参考中的原生地图/列表呈现，配合该类别的紧凑 Markdown 表格。其他类别使用该类别的 Markdown 表格。最多展示三到五个不同选项。
3. **One decision.** For flights, let the user select the complete itinerary
   through the native flight widget; do not add a duplicate picker or ask for a
   typed option label. Other booking categories continue to use their
   category-specific presentation contract.
   **一次决策。**对航班，让用户通过原生航班组件选择完整行程；不要添加重复的选择器，也不要要求输入选项标签。其他预订类别继续使用其类别专属的呈现契约。

Do not emit HTML or use `create_options`. Other than the hotel map/list
presentation, do not use a native picker/list widget for non-flight bookings.
Do not duplicate prices, schedules, terms, or the comparison as prose. Put
important decision fields in the primary comparison and give secondary facts
in a short labeled block only after the user selects an option or asks for more
detail.

不要输出 HTML，也不要使用 `create_options`。除酒店地图/列表呈现外，非航班预订不得使用原生选择器/列表组件。不要把价格、时刻、条款或对比复述成散文。重要的决策字段放入主对比；次要事实只在用户选定某个选项或要求更多细节后，以简短的带标签块给出。

When there is only one exact match, skip the comparison table and use short
labeled Markdown fields.

当只有一个精确匹配时，跳过对比表格，改用简短的带标签 Markdown 字段。

## Markdown rules / Markdown 规则

- Keep the recommendation above the table to one sentence.
  表格上方的推荐保持为一句话。
- Use stable option labels such as `A`, `B`, and `C` in every follow-up.
  在每次后续对话中使用稳定的选项标签，如 `A`、`B`、`C`。
- Keep each table to six columns or fewer. Combine closely related facts in one
  cell with semicolons or short phrases.
  每张表不超过六列。紧密相关的事实用分号或短语合并进一个单元格。
- Put the full party or stay total in the table. Do not make the user calculate
  it from a per-person or nightly teaser price.
  把全团或整个入住期的总价放进表格。不要让用户从每人价或每晚噱头价自行计算。
- Keep fields in the same order for every option.
  所有选项的字段顺序保持一致。
- Bold only the recommended option label and the most important changed fact.
  仅对推荐选项标签和最重要的变化事实使用粗体。
- Render absent provider facts as `Not stated`. Do not silently omit a field
  whose absence could change the decision.
  供应商缺失的信息渲染为 `Not stated`。绝不悄悄省略其缺失可能改变决策的字段。
- Use airport-local, property-local, venue-local, or restaurant-local dates and
  times. Include the year when ambiguous.
  使用机场当地时间、酒店当地时间、场馆当地时间或餐厅当地时间。有歧义时附上年份。
- Do not include raw provider identifiers, tool names, command output, or
  implementation details.
  不包含原始供应商标识符、工具名、命令输出或实现细节。

Verified imagery may still be shown with ordinary Markdown image syntax after
the comparison table. Use only stable public HTTPS images from the current
provider, venue, or verified listing. Label each image accurately. Do not use a
generic property photo as the exact room, a restaurant interior as a named dish,
or a venue map as the view from a specific seat. Omit imagery when no reliable
source exists.

经验证的图片仍可在对比表格之后用普通 Markdown 图片语法展示。只使用来自当前供应商、场馆或经验证房源的稳定公共 HTTPS 图片。每张图片都要准确标注。不得把通用酒店照片当作具体房间、把餐厅内景当作某道指定菜品、把场馆地图当作某个特定座位的视野。没有可靠来源时省略图片。

## Flights / 航班

Use the native structured flight list described in
`/opt/hatch/skills/booking/references/flights.md` for every live flight
comparison when `widget.create` is available. Do not substitute Markdown based
on uncertainty, convenience, output size, or schema complexity. Use this table
only when `widget.create` is absent or a valid widget call explicitly returns an
unsupported or rendering error:

`widget.create` 可用时，每次实时航班对比都使用 `/opt/hatch/skills/booking/references/flights.md` 所述的原生结构化航班列表。不要以不确定、图方便、输出大小或 schema 复杂度为理由改用 Markdown。仅当 `widget.create` 缺失，或一次合法的组件调用明确返回不支持或渲染错误时，才使用下表：

| Option | Route and local times | Duration | Stops and layovers | Flight and cabin | Full total |
|---|---|---:|---|---|---:|
| **A (Recommended)** | SFO 8:10 AM → JFK 4:45 PM | 5h 35m | Nonstop | UA 123 · Economy Flex | $642 round-trip |
| B (Cheapest) | SFO 7:00 AM → DEN 10:25 AM → JFK 4:10 PM | 6h 10m | 1 stop · DEN 1h 05m | UA 456/789 · Economy | $511 round-trip |

| 选项 | 航线与当地时间 | 时长 | 经停与中转 | 航班与舱位 | 全额总价 |
|---|---|---:|---|---|---:|
| **A（推荐）** | SFO 8:10 AM → JFK 4:45 PM | 5h 35m | 直飞 | UA 123 · Economy Flex | $642 往返 |
| B（最便宜） | SFO 7:00 AM → DEN 10:25 AM → JFK 4:10 PM | 6h 10m | 1 次经停 · DEN 1h 05m | UA 456/789 · Economy | $511 往返 |

Use one row per complete itinerary; keep outbound and return together in the
same row using concise labels when it is a round trip. Every row must show:

每个完整行程一行；往返时用简洁标签把去程和回程保持在同一行。每行必须显示：

- full party price and currency, labeled one-way or round-trip
  全团价格与币种，标注单程或往返
- each leg's local departure and arrival airports and times
  每一航段的当地出发/到达机场与时间
- each leg's total elapsed duration
  每一航段的总耗时长
- nonstop or exact stop count
  直飞或准确经停次数
- every connection airport and layover duration
  每个中转机场及中转时长
- marketing carrier and flight number, plus operating carrier when different
  市场承运航司与航班号，实际承运不同时一并注明
- cabin and fare brand when stated
  有说明时给出舱位与票价品牌

Do not hide price, duration, stops, or layovers in a follow-up. Calculate each
layover from adjacent provider timestamps at the same airport. If that cannot be
done reliably, write `Layover length not stated`.

不要在后续对话中才披露价格、时长、经停或中转。每段中转时长由同一机场相邻的供应商时间戳计算。若无法可靠计算，写 `Layover length not stated`。

After the Markdown fallback comparison, add at most one short `Details:` line
for shared baggage or fare-rule caveats. Do not show the same inventory in a second format.

Markdown 回退对比之后，最多添加一行简短的 `Details:`，用于共用的行李或票价规则注意事项。不要以第二种格式重复展示同一批库存。

## Hotels / 酒店

Use one row per exact property-room-rate combination:

每个"酒店-房型-价格"精确组合一行：

| Option | Hotel and exact room | Location | Full stay total | Terms and benefits | Why it fits |
|---|---|---|---:|---|---|

| 选项 | 酒店与具体房型 | 位置 | 整段入住总价 | 条款与权益 | 推荐理由 |
|---|---|---|---:|---|---|

Include exact room and bed, dates, total including mandatory fees, amount due at
the property, cancellation deadline, and material loyalty benefits. When
location matters, show verified travel time to the named place rather than a
vague neighborhood claim.

包含具体房型和床型、日期、含强制性费用的总价、到店应付金额、取消截止时间以及重要的会员权益。位置重要时，给出到指定地点的经验证交通时间，而不是模糊的"邻近某区"说法。

With the first multi-property shortlist, automatically provide the native map
or list described in the hotel reference. Keep its option labels and order
identical to the Markdown comparison. If proximity to a particular place
matters, show verified travel times in the comparison rather than adding a
second location table.

首次给出多家酒店候选清单时，自动提供酒店参考中描述的原生地图或列表。其选项标签与顺序须与 Markdown 对比保持一致。若与某特定地点的距离重要，在对比中展示经验证的交通时间，而不是再加一张位置表。

Attach each useful verified property image to that property's map marker or
list row. Never show a separate unlabeled image gallery. After selection, offer
exact-room images separately and label unmatched images `Property photo`.

把每张有用的经验证酒店图片挂到该酒店的地图标记或列表行上。绝不展示单独的无标注图片墙。选定之后，单独提供具体房型的图片，未匹配到房型的图片标注 `Property photo`。

## Restaurants / 餐厅

Use one row per exact live table:

每个确切的实时餐位一行：

| Option | Restaurant | Time and seating | Cuisine and price | Location | Deposit or cancellation terms |
|---|---|---|---|---|---|

| 选项 | 餐厅 | 时间与座位类型 | 菜系与价位 | 位置 | 押金或取消条款 |
|---|---|---|---|---|---|

Show exact available time, seating type, cuisine, price level, address or travel
time, and any deposit, prepaid minimum, service charge, cancellation fee, or
no-show exposure. The recommendation sentence should name the one taste,
location, timing, or value reason that makes the first option best.

展示确切的可订时间、座位类型、菜系、价位、地址或交通时间，以及任何押金、预付最低消费、服务费、取消费或爽约风险。推荐句应指出让首选选项最优的那一个口味、位置、时段或性价比理由。

When reliable imagery exists, show at most one verified food, dining-room, or
exterior image for the recommended option below the table. Use a dish name only
when the source identifies it; otherwise label it `Food at <restaurant>` or  
`Dining room`.

当有可靠图片时，在表格下方为推荐选项最多展示一张经验证的菜品、餐厅内景或门面图。仅当来源指明菜品名时才使用菜名；否则标注 `Food at <restaurant>` 或  
`Dining room`。

## Event tickets / 活动门票

Use one row per exact ticket group:

每个确切的票档一行：

| Option | Event and local time | Exact seats | Primary or resale | Delivered total | Important terms |
|---|---|---|---|---|---:|

| 选项 | 活动与当地时间 | 确切座位 | 官方或转售 | 含费到手总价 | 重要条款 |
|---|---|---|---|---|---:|

Preserve ticket count, section, row, exact seats when assigned, view
restrictions, primary/resale status, fees, delivery method, and hold expiry.
When a verified seat-view image exists, show it below the table with its exact
section label; otherwise provide a verified venue-map link without implying a
specific view. Include a verified actionable listing or purchase link for every
option offered as the user's next choice. Do not present an extra option without
the same action path as the rest of the shortlist.

保留票数、看台区、排、分配时的确切座位、视野限制、官方/转售状态、费用、交付方式和锁票到期时间。当存在经验证的座位视野图时，在表格下方连同其确切看台标签一并展示；否则提供经验证的场馆地图链接，且不暗示特定视野。为提供的每个选项附上经验证可操作的 listings 或购买链接，作为用户的下一步选择。多出来的选项若不具备与其他候选相同的操作路径，就不要展示。

## Final review and confirmation / 最终审查与确认

Do not show another large comparison after selection. Use a compact two-column
Markdown table for the commitment boundary:

选定之后不要再展示大型对比。用一张紧凑的两列 Markdown 表格呈现承诺边界：

| Review | Selected terms |
|---|---|
| Item | Exact itinerary, property and room, table, or seats |
| Date and time | Local date and time |
| Total | Full amount, currency, amount due now, and amount due later |
| Applied value | Credits, points, certificates, or benefits |
| Payment | Safe payment label only |
| Terms | Material cancellation, refund, change, or no-show terms |

| 审查项 | 已选条款 |
|---|---|
| 项目 | 确切行程、酒店与房型、餐位或座位 |
| 日期与时间 | 当地日期与时间 |
| 总价 | 全额、币种、现在应付与后续应付金额 |
| 抵扣价值 | 抵用金、积分、券或权益 |
| 支付 | 仅安全支付标签 |
| 条款 | 重要的取消、退款、改期或爽约条款 |

The trusted connector or browser approval remains authoritative. Do not create
a second confirmation question when the tool already asks the user to approve
the same terms.

受信连接器或浏览器审批仍是权威。当工具已就相同条款请求用户批准时，不要再制造第二个确认问题。

After success, return a compact Markdown receipt with:

成功之后，返回一份紧凑的 Markdown 回执，包含：

- `Status:` Confirmed, Held, Waitlisted, or Blocked
  `Status:` Confirmed、Held、Waitlisted 或 Blocked
- `Confirmation:` the safe human-readable reference
  `Confirmation:` 安全的人类可读参考号
- `When and where:` local date, time, and location
  `When and where:` 当地日期、时间与地点
- `Total:` amount paid or committed
  `Total:` 已支付或已承诺的金额
- `Next:` the next deadline or action, only when one exists
  `Next:` 下一个截止时间或动作，仅在实际存在时给出

Do not display secrets or full payment, passport, traveler-security, or loyalty
numbers.

不得显示机密，以及完整的支付、护照、旅客安全或会员号码。
