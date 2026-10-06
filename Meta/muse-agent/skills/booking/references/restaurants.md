<!-- BILINGUAL-EN-ZH -->
# Restaurants / 餐厅

Use this reference when the user wants a table booked, whether they name the
restaurant or ask for a recommendation.

当用户想订位时使用本参考，无论他们是指名餐厅还是请求推荐。

## Build the dining profile before asking / 在提问之前先构建用餐画像

Use authorized memory, reservation history, email, calendar, and connected
restaurant platforms to determine, when available:

利用经授权的记忆、预订历史、邮件、日历和已连接的餐厅平台来确定以下信息（在可用时）：

- restaurants previously booked and whether the user returned
  之前预订过的餐厅，以及用户是否再次光顾
- the platform successfully used for a particular restaurant: Resy,
  OpenTable, Tock, SevenRooms, the restaurant site, or another service
  某家餐厅曾成功使用的平台：Resy、OpenTable、Tock、SevenRooms、餐厅官网或其他服务
- cuisine and taste preferences, price comfort, usual dining times, and
  preferred neighborhoods
  菜系与口味偏好、可接受的价格区间、惯常用餐时间和偏好的街区
- indoor/outdoor, bar/counter/table, quiet/lively, and occasion preferences
  室内/室外、吧台/料理台/餐桌、安静/热闹以及场合偏好
- saved guest profile and any explicit dietary or accessibility needs
  已保存的客人档案，以及任何明确的饮食或无障碍需求

Do not infer allergies, dietary restrictions, or accessibility needs from a
menu choice or one old reservation. Do not expose the user's reservation
history; use it to improve the recommendation.

不要从某次点餐选择或一条旧预订推断过敏、饮食限制或无障碍需求。不要向外泄露用户的预订历史；用它来改进推荐即可。

【评论】禁止从单次行为推断过敏等敏感属性，属于在个性化与过度画像之间划定边界的隐私条款。

## Minimum search facts / 最低搜索要素

Date, party size, and a usable time window are required. A named restaurant
also needs the correct location when there are multiple branches. When the
user says "tomorrow" with no time, use a well-supported usual dining time or
search a reasonable dinner window and label it; ask only if no reliable default
exists or the time would change the result materially.

日期、人数和可用的时间窗口是必需的。指名餐厅时，若有多家分店还需要正确的位置。当用户只说"明天"而未给出时间时，使用有充分依据的惯常用餐时间，或搜索一个合理的晚餐窗口并加以标注；只有在不存在可靠默认值、或时间会实质性改变结果时才询问。

For an unnamed restaurant, infer area from location/context, cuisine and price
from established preferences, and calendar timing where useful. Present the
assumption with the recommendation rather than asking a long intake form.

对于未指名的餐厅，从位置/上下文推断区域，从既有偏好推断菜系和价位，并在有用处参考日历时间。把假设与推荐一并呈现，而不是让用户填一份冗长的信息采集表。

## Search and route / 搜索与路由

For a named restaurant, first use the venue's official reservation link or the
platform that successfully handled prior reservations there. Otherwise use a
connected reservation provider such as `opentable` or Resy, then another
platform, the venue's own site through the browser, and `phone.place_call` when
available. Absence from one platform is not evidence that the restaurant is
unbookable.

对于指名餐厅，首先使用该场所的官方预订链接，或之前在那里成功完成过预订的平台。否则使用已连接的预订服务提供商（如 `opentable` 或 Resy），然后是其他平台、通过浏览器访问的场所官网，以及可用时的 `phone.place_call`。在某个平台上搜不到并不代表这家餐厅订不到位。

Before using OpenTable, read `/opt/hatch/skills/opentable/SKILL.md` for its
connection and command contract. OpenTable is a provider within this booking
flow, not a replacement for it.

在使用 OpenTable 之前，先阅读 `/opt/hatch/skills/opentable/SKILL.md` 了解其连接与命令契约。OpenTable 是本预订流程中的一个服务提供商，不是对流程本身的替代。

If OpenTable is available but disconnected, recommend its low-friction
connection and show the exact supported connection action once. Briefly explain
that connecting improves live availability, saved-profile use, and booking,
then continue checking the restaurant's official reservation link or another
reputable platform without waiting for the connection. If the user connects,
check the current booking state before resuming the OpenTable path. Do not create
another reservation when the same restaurant, date, time, and party size is
already confirmed through another path. Tell the user that the reservation is
already booked. Offer OpenTable only for a later change or cancellation when it
can manage that reservation. When another path has an active hold or an
ambiguous submission, resolve that state before using OpenTable. Resume the
original OpenTable request without making the user repeat it only when no
confirmed reservation, active hold, or ambiguous submission can cause a
duplicate.

如果 OpenTable 可用但未连接，推荐其低摩擦的连接方式，且只展示一次确切的受支持连接操作。简要说明连接可以改善实时空位查询、已保存档案的使用和预订，然后继续检查餐厅的官方预订链接或其他可靠平台，而不等待连接完成。如果用户完成了连接，在恢复 OpenTable 路径之前先检查当前预订状态。当同一餐厅、日期、时间和人数已经通过其他路径确认时，不要再创建一条预订。告知用户该预订已经订好。仅当 OpenTable 今后能够管理该预订时，才为后续更改或取消提供 OpenTable。当另一条路径存在有效的暂存（hold）或状态不明的提交时，先解决该状态再使用 OpenTable。只有在确认预订、有效暂存或状态不明提交都不会造成重复时，才在不让用户重复请求的情况下恢复原有的 OpenTable 请求。

A provider miss, error, or unavailable connector ends only that provider
attempt, not the booking task. Continue to the restaurant's official website
and reservation link immediately, then another reputable platform, and then
`phone.place_call` when available. Do not ask the user whether to try the
website or another channel. Find actual availability and advance as far as
possible. Ask only for a choice or required detail that cannot be recovered from an
authorized profile or account. For accounts other than the low-friction
OpenTable connection above, offer sign-in when it materially improves inventory
or checkout.

某个提供商查询失败、报错或连接器不可用，只结束那一次提供商尝试，而不是整个预订任务。立即转向餐厅官网及其预订链接，然后是其他可靠平台，再然后是可用时的 `phone.place_call`。不要询问用户是否要试试网站或其他渠道。查明实际空位并尽可能向前推进。只询问无法从授权档案或账户中恢复的选择或必需细节。对于除上述低摩擦 OpenTable 连接之外的账户，在登录能实质性改善库存或结账时才建议登录。

For an unnamed restaurant, search live inventory in addition to reviews. Rank
only tables that fit the date, party, location, and time window.

对于未指名的餐厅，除评论之外还要搜索实时空位。只对符合日期、人数、位置和时间窗口的餐桌进行排序。

Use an authenticated account when it unlocks saved preferences, exclusive
inventory, or a complete booking. Do not create an account or join a paid
membership without approval.

当已认证的账户能解锁已保存的偏好、专属库存或完整预订时，使用它。未经批准，不要创建账户或加入付费会员。

## Rank on the actual table / 按实际餐桌排序

Compare:

比较维度：

- exact available time and table/seating type
  确切的可订时间与餐桌/座位类型
- cuisine and fit with known tastes
  菜系以及与已知口味的契合度
- total deposit, prepaid minimum, service charge, or cancellation/no-show fee
  押金总额、预付最低消费、服务费或取消/未到场费用
- price level or relevant menu format
  价位或相关的菜单形式
- location and travel time when context makes it important
  当上下文使其重要时的位置与路程时间
- special terms such as set menu, outdoor exposure, age limit, or dining-time  
  limit
  特殊条款，如套餐、户外座位、年龄限制或用餐时长限制

If the exact time is unavailable, show the nearest times. Do not silently book
an adjacent slot, a different branch, bar seating, outdoors, or a waitlist.

如果确切时间不可订，展示最接近的时间。不要悄悄预订相邻时段、另一家分店、吧台座位、户外座位或候补名单。

【评论】"不得静默预订相邻时段或替代座位"体现了代理行动的知情同意原则：对用户未确认的实质变更必须显式呈现而非默认执行。

## Prepare and book / 准备与预订

At final review include restaurant and location, local date/time, party size,
seating type, deposit or prepayment, cancellation/no-show terms, and any special
request being submitted. A "special occasion" note is not a promise that the
restaurant will provide anything.

最终确认时应包含餐厅及其位置、当地日期/时间、人数、座位类型、押金或预付款、取消/未到场条款，以及正在提交的任何特殊请求。"特殊场合"备注并不是餐厅会提供任何东西的承诺。

Use stored contact details through the provider or an authorized profile. Do
not ask for name, email, or phone merely to begin searching another booking
channel. Ask only for contact fields still missing after an exact table is
found and the chosen checkout requires them. Ask for missing dietary or
accessibility information only when the user raised it or the booking
requires it.

通过提供商或授权档案使用已存储的联系信息。不要仅为开始搜索另一个预订渠道就索要姓名、邮箱或电话。只有在找到确切餐桌、且所选结账流程需要、而联系字段仍然缺失时才询问。仅当用户主动提起或预订本身要求时，才询问缺失的饮食或无障碍信息。

Verify with the reservation confirmation or a reservation visible in the
provider account. Return restaurant, address, date/time, party size, seating,
safe confirmation reference, and cancellation deadline.

用预订确认信息或提供商账户中可见的预订进行核实。返回餐厅、地址、日期/时间、人数、座位、安全确认编号（safe confirmation reference）和取消期限。
