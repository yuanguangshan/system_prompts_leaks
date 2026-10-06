<!-- BILINGUAL-EN-ZH -->

# Event tickets / 活动门票

Use this reference for sports, concerts, theater, comedy, and other ticketed
events.

本参考适用于体育、演唱会、戏剧、喜剧以及其他需购票的活动。

## Resolve the event before asking / 提问前先确定活动

Use authorized context, calendar, ticketing accounts, email, and live schedules
to resolve:

利用已授权的上下文、日历、票务账户、电子邮件和实时赛程/日程来确定：

- exact team, artist, show, or event
  确切的球队、艺人、演出或活动
- the next relevant occurrence and its local date/time
  下一个相关场次及其当地日期/时间
- home versus away, venue, city, and competition/season when ambiguous
  在有歧义时确定主场还是客场、场馆、城市以及赛事/演出季
- prior ticketing platforms, favorite sections, price comfort, and seat-view  
  preferences
  以往使用的票务平台、偏好的看台区域、价格承受范围和座位视野偏好
- memberships, season-ticket access, presales, fan-club benefits, or cardholder  
  offers
  会员资格、季票权限、预售、粉丝俱乐部权益或持卡人优惠
- usual party size only as a hint; do not assume companions are attending
  常规人数仅作为提示；不要假定同伴一定参加

For "the Patriots' next game," verify the official schedule and whether "next"
means the next game overall or the next home game in the current context. Ask
only if both remain plausible and lead to different purchases.

对于"爱国者队的下一场比赛"，要核实官方赛程，并确认"下一场"在当前语境中指总体的下一场还是下一个主场。只有当两种解释都合理且会导致不同购买时才追问。

## Minimum search facts / 最少搜索事实

The exact event occurrence and ticket count are required. Seating and budget
can be inferred as search preferences from reliable history, but they are not
permission to exceed a stated or established ceiling. Accessibility seating
must be explicit. Do not infer it. An unanswered ticket-count question remains
unresolved even if the user answers another part of the question. Never call
seat inventory such as `top-picks` until the exact count is known; ask again.

确切的活动场次和门票数量是必需的。座位和预算可以从可靠历史推断为搜索偏好，但这不是超过已说明或已确立上限的许可。无障碍座位必须由用户明确提出，不得推断。即使用户回答了问题中的其他部分，未获回答的门票数量问题仍视为未解决。在确切数量已知之前，绝不要调用 `top-picks` 之类的座位库存；要再次询问。

## Search and route / 搜索与路径

Check official schedules first. Search primary inventory through the venue or
an available provider such as `ticketmaster`, including authenticated presales
or account offers. Then compare reputable verified resale inventory when it is
legal and useful. Clearly label primary versus resale.

先查官方赛程/日程。通过场馆或可用的提供方（如 `ticketmaster`）搜索一级市场库存，包括经过身份验证的预售或账户优惠。然后在合法且有用时比较信誉良好的已验证转售库存。明确标注一级市场与转售。

Before using Ticketmaster, read `/opt/hatch/skills/ticketmaster/SKILL.md` for
its command contract. Ticketmaster is a provider within this booking flow, not
a replacement for it.

在使用 Ticketmaster 之前，先阅读 `/opt/hatch/skills/ticketmaster/SKILL.md` 了解其命令契约。Ticketmaster 是本预订流程中的一个提供方，不能取代该流程。

Search output or a buy link is not a completed booking. Use the authenticated
browser to choose exact seats and reach final review. If an onsale has not
opened, report the verified onsale time; do not pretend inventory is sold out.

搜索结果或购买链接不等于已完成预订。使用经过身份验证的浏览器选择确切座位并进入最终审查。如果开售尚未开始，报告已核实的开售时间；不要假装库存已售罄。

Do not use speculative or unverified ticket listings. Do not create a paid fan
club membership, season plan, or credit-card application to unlock inventory.

不要使用投机性或未经验证的票源。不要为了解锁库存而创建付费粉丝俱乐部会员、季票计划或信用卡申请。

【评论】"不得为解锁票源而新建付费会员或信用卡申请"限制了代理的付费开户行为，属于防止未经授权金融承诺的条款。

## Rank on the exact seats and delivered price / 按确切座位与到手总价排序

Compare:

比较：

- section, row, exact seats, and how many are together
  区域、排、确切座位，以及多少座位连在一起
- sightline, distance, side/angle, and obstructed or limited-view notation
  视线、距离、侧面/角度，以及视线受阻或视野受限标注
- primary versus resale and the seller/marketplace guarantee
  一级市场与转售，以及卖家/平台保障
- itemized ticket price, service/facility/order fees, taxes, and delivered total
  逐项的票价、服务/设施/订单费、税费，以及到手总价
- ticket format, delivery timing, transfer restrictions, and resale policy
  票券形式、交付时间、转让限制和转售政策
- refund/postponement/cancellation terms
  退款/延期/取消条款
- material membership, presale, or cardholder benefit
  实质性的会员、预售或持卡人权益

Do not recommend seats from a venue map alone when actual inventory is
available. Do not silently move to a different date, venue, performance, ticket
count, seating area, or resale listing.

当实际库存可用时，不要仅凭场馆平面图推荐座位。不要在未经说明的情况下改到不同的日期、场馆、场次、票数、看台区域或转售票源。

Treat shade, roof coverage, weather exposure, and obstructed view as
listing-specific claims. Do not infer them from a side or section name. Without
a current source that explicitly supports the attribute for that section or
ticket group, say it is unverified; general venue facts may support only a
labeled best-likelihood recommendation, never a guarantee.

将遮阳、屋顶覆盖、露天暴露和视线受阻视为针对具体票源的声明。不要从看台或区域名称推断。如果没有当前来源明确支持该区域或票组的这一属性，就说它未经验证；场馆的常识性事实最多只能支持一个标注了"可能性最高"的建议，绝不能当作保证。

## Prepare and book / 准备与预订

Seat holds expire quickly. Keep the same checkout alive, show the hold expiry,
and refresh the exact seats and total if approval arrives late.

座位占位很快过期。保持同一结账会话存活，展示占位过期时间，如果批准来得太晚则刷新确切座位与总额。

At final review include event, venue, local date/time, ticket count, exact
section/row/seats, primary or resale status, itemized fees, delivered total,
delivery method, transfer restrictions, and refund/postponement terms.

在最终审查时包含活动、场馆、当地日期/时间、票数、确切的区域/排/座位、一级或转售状态、逐项费用、到手总价、交付方式、转让限制以及退款/延期条款。

Use the saved ticketing profile and secure wallet at checkout. Do not expose
full membership or payment numbers.

在结账时使用已保存的票务资料和安全钱包。不要暴露完整的会员号或支付号码。

Verify with an order number and tickets or a delivery record visible in the
account. Return the event details, seats, total, safe order reference, delivery
state, and any deadline. If tickets will arrive later, say so plainly; an order
confirmation is not the same as tickets already delivered.

用订单号以及账户中可见的票券或交付记录进行验证。返回活动详情、座位、总额、安全的订单引用、交付状态和任何截止时间。如果票券稍后才会到达，要明确说明；订单确认不等于票券已交付。
