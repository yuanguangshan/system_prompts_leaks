<!-- BILINGUAL-EN-ZH -->
# Hotels / 酒店

Use this reference for direct hotel-booking requests. Do not expand the ask
into flights, activities, transportation, or a full itinerary.

本参考用于直接的酒店预订请求。不要把请求扩展为机票、活动、交通或完整行程。

Ask date, occupancy, account, and option questions in plain Markdown. Do not
call `create_options`. Use the native map/list presentation below for results.

用纯 Markdown 询问日期、入住人数、账户与选项问题。不要调用 `create_options`。结果使用下方的原生地图/列表呈现。

Keep implementation details out of traveler-facing responses: do not mention
APIs, tools, response fields, integrations, or their limitations. Never replace
a missing provider fact with reputation or general knowledge. If a requested
material fact cannot be verified, say only that it could not be verified for
the displayed option; otherwise omit it.

面向旅行者的回复中不要包含实现细节：不要提及 API、工具、响应字段、集成或其局限。绝不用口碑或一般知识替代缺失的提供商事实。如果某个被要求的重要事实无法核实，只说明"针对所展示的选项无法核实"；否则干脆省略。

【评论】"绝不用口碑或一般知识替代缺失的提供商事实"是一条防幻觉约束：宁可声明无法核实，也不允许用模型常识补全关键预订事实。

## Build the stay profile before asking / 提问之前先构建住宿画像

Use authorized memory, email, hotel and booking-platform accounts, and prior
stays to determine, when available:

利用已授权的记忆、电子邮件、酒店与预订平台账户以及既往入住记录，在可得时确定：

- properties and neighborhoods previously used in the destination
  目的地中以前使用过的物业与街区
- explicit likes/dislikes from prior stays
  既往入住中明确表达的喜好/厌恶
- boutique versus large hotel, independent versus chain, and desired service  
  level
  精品酒店还是大型酒店、独立酒店还是连锁酒店，以及期望的服务水平
- usual room type, bed type, floor, view, quiet-room, accessibility, and
  smoking preferences
  常用的房型、床型、楼层、景观、安静房、无障碍与吸烟偏好
- price comfort and willingness to prepay
  价格承受度与预付意愿
- loyalty programs, status, points, free-night certificates, upgrade benefits,
  breakfast, parking, resort-credit, or late-checkout eligibility
  常旅客计划、会员等级、积分、免费晚证书、升级权益、早餐、停车、度假村额度或延迟退房资格
- preferred booking channel, including direct hotel accounts and platforms
  such as Booking.com
  首选预订渠道，包括酒店直销账户与 Booking.com 等平台

Do not treat one historical stay as a permanent style preference. Do not infer
accessibility requirements or willingness to accept a non-refundable rate.

不要把某一次历史入住当作永久性的风格偏好。不要推断无障碍需求，也不要推断愿意接受不可退款房价。

## Minimum search facts / 最低搜索要素

Destination or property, check-in, check-out, guest count, and room count are
required. If the user names only a night, infer a one-night stay and label it.
Use the destination timezone for dates. Ask early only if destination ambiguity
or occupancy would invalidate the search.

目的地或物业、入住日期、退房日期、客人数与房间数为必需项。如果用户只说了"一晚"，推断为一晚住宿并加以标注。日期使用目的地时区。只有当目的地歧义或入住人数会导致搜索失效时才提前询问。

Every child must have an exact age before searching because age changes lodging
eligibility and price. Ask once for any missing ages; do not infer them or
search children as adults. Search again whenever the room occupancy or a child
age changes.

每个儿童在搜索前都必须有确切年龄，因为年龄会改变住宿资格与价格。对缺失的年龄只问一次；不要推断，也不要把儿童当成人搜索。房型入住人数或儿童年龄变化时重新搜索。

【评论】要求儿童提供精确年龄源于酒店业的入住资格与计价惯例（年龄常决定能否入住或是否收费），因此这里明确禁止推断。

If the location is broad, use the stated purpose, calendar event, prior
neighborhood history, or current context to center the search. Say what the
search is centered on.

如果地点范围太宽泛，使用所述目的、日历事件、既往街区历史或当前上下文来收拢搜索中心。说明搜索以什么为中心。

Do not block the first useful search on location preferences. With the initial
shortlist, ask whether they care about proximity to a particular place,
neighborhood, event, office, or transit stop. If they name a target, calculate
or verify travel time and rerank the options around it.

不要让首次有效搜索阻塞在位置偏好上。拿到初始入围名单后，再询问用户是否在意与某个地点、街区、活动、办公室或公交站的距离。如果用户说出目标，计算或核实通行时间，并据此重排选项。

When `widget.create` is available, automatically show a `local_map` for a
multi-property shortlist. Use only provider coordinates or coordinates
verified with an available maps/places tool; never guess them. Give every
marker the same stable option label and property name used in the comparison,
put the full-stay price and decisive term in its subtitle, and attach that
property's verified image as `background_image_url` when available.
Prefer coordinates returned with the lodging result. Do not geocode properties
that already have both coordinates, do not delay the initial map to fetch
missing photos or other per-property enrichment, and create the shortlist map
in one widget operation rather than iteratively rebuilding it.

当 `widget.create` 可用时，对多物业入围名单自动展示 `local_map`。只使用提供商坐标或经可用的地图/地点工具核实过的坐标；绝不猜测。给每个标记使用与对比表中一致的稳定选项标签和物业名称，把全住宿期总价与决定性条款放入其副标题，并在可用时把该物业已核实的图片附加为 `background_image_url`。优先使用随住宿结果一起返回的坐标。不要为已有双坐标的物业重新地理编码，不要为补取照片或其他逐物业增强信息而推迟初始地图，并以一次 widget 操作创建入围地图，而不是迭代重建。

If exact coordinates cannot be verified or the map call fails, create a
generic native `list` whose rows use the same labels and order, with one
verified property image attached to its corresponding hotel. The list is a
visual summary, not a selection control; keep the compact Markdown comparison
as the place where complete terms and the user's `A`/`B`/`C` choice are clear.
Never emit an unlabeled image grid. If neither widget is available, place each
labeled image immediately beside or below its hotel's Markdown entry.

如果确切坐标无法核实或地图调用失败，创建一个通用的原生 `list`，其行使用相同的标签与顺序，并为每家酒店附上对应的已核实物业图片。该列表是视觉摘要，不是选择控件；保持紧凑的 Markdown 对比表，作为完整条款与用户 `A`/`B`/`C` 选择清晰呈现之处。绝不输出无标签的图片网格。如果两种 widget 都不可用，把每张带标签的图片紧挨或紧放在其酒店 Markdown 条目旁边或之下。

## Search and route / 搜索与渠道

Use an installed lodging provider for live lodging discovery and rate refreshes
when available; otherwise use another connected accommodation tool or the
browser. Do not treat a flight-only integration as supporting hotels. Confirm
shortlisted rates rather than trusting teaser prices. Check the authenticated
hotel-chain site when status, member pricing, points, or certificates matter.
Check a connected booking platform when it contains useful history, member
pricing, or better inventory.

在可用时使用已安装的住宿提供商进行实时住宿发现与价格刷新；否则使用另一款已连接的住宿工具或浏览器。不要把仅支持机票的集成当作支持酒店。核实入围房价，而不要轻信宣传价。当会员等级、会员价、积分或证书有影响时，查验经过认证的酒店集团官网。当某个已连接的预订平台包含有用的历史、会员价或更好的库存时，查验该平台。

For the selected rate, use native checkout only when the freshly refreshed
provider data explicitly marks that exact rate eligible. When a selected rate
requires provider-hosted checkout, always surface its validated URL. If the
user has already asked to complete the booking, continue through that exact URL
in the browser; otherwise leave it as an actionable link without asking an
additional question. A provider-hosted checkout URL is the provider's supported
completion path, not a failed provider path. Do not switch rooms, rates, or
terms to force native checkout. Booking retrieval, cancellation terms, and
cancellation apply only to bookings represented by an owned capability in the
selected provider skill.

对选定的房价，只有当刚刷新过的提供商数据明确标记该确切房价符合条件时才使用原生结账。当选定房价需要提供商托管的结账页时，始终呈现其经过验证的 URL。如果用户已要求完成预订，就在浏览器中沿该确切 URL 继续；否则将其保留为可操作的链接，不额外追问。提供商托管结账 URL 是提供商支持的完成路径，不是失败的提供商路径。不要为了强行使用原生结账而更换房间、房价或条款。预订检索、取消条款与取消操作仅适用于在所选提供商技能中由自有能力代表的预订。

【评论】"不要为了强行使用原生结账而更换房间、房价或条款"强调渠道中立：完成路径取决于提供商支持，而不是代理自身的偏好。

Compare direct and third-party rates on equivalent rooms and terms. A cheaper
third-party rate may lose status credit, upgrades, breakfast, flexibility, or
direct support; a direct rate is not automatically better.

在同等房间与同等条款上比较直销价与第三方价。更便宜的第三方价可能失去等级累计、升级、早餐、灵活性或直销支持；直销价也不自动更优。

When moving to a hotel or booking-platform website, ask whether the user has
an account there and wants to sign in so member rates, loyalty benefits, saved
preferences, and booking history can be used. Continue as a guest if they do
not. Do not require a new account.

转到酒店或预订平台网站时，询问用户在那里是否有账户并愿意登录，以便使用会员价、常旅客权益、保存的偏好与预订历史。如果不愿意，则以访客身份继续。不要要求创建新账户。

When browser continuation applies under the rule above, follow the shared
browser-booking workflow. Do not switch properties, room types, dates, or rate
terms because one channel fails.

当根据上述规则需要浏览器继续时，遵循共享的浏览器预订工作流。不要因为某个渠道失败而更换物业、房型、日期或房价条款。

## Rank on the full stay / 按整个住宿周期排序

Compare:

比较以下方面：

- full stay total, taxes, mandatory resort/destination fees, and anything due
  at the property
  整个住宿期的总价、税费、强制的度假村/目的地费用，以及需要在物业现场支付的一切
- room and bed type, occupancy, and whether the room is guaranteed
  房型与床型、入住人数，以及房间是否获得保证
- exact location and travel time to the stated purpose
  确切位置以及到达所述目的地的通行时间
- cancellation deadline, refundability, prepayment, deposit, and card hold
  取消期限、可退款性、预付款、押金与银行卡预授权
- included breakfast, parking, Wi-Fi, credits, and meaningful status benefits
  包含的早餐、停车、Wi-Fi、额度以及有实际意义的会员等级权益
- points earned or redeemed and certificate value
  赚取或兑换的积分以及证书价值
- check-in/check-out times and late-arrival requirements
  入住/退房时间以及晚到要求

Do not compare only nightly rates. Disclose the charged-now and due-at-property
amounts separately. A property with only an estimated total, missing mandatory
fees, or unknown current cancellation terms is not a bookable contender. Keep
it outside the ranked shortlist until rechecked. Do not label a rate
`refundable` when its cancellation deadline has passed; reverify it.

不要只比较每晚房价。分开披露现在需付与到店需付的金额。只有估算总价、缺少强制费用或当前取消条款不明的物业不是可预订的候选。在重新核查之前把它排除在排序入围名单之外。当某个房价的取消期限已过时，不要标注为 `refundable`；要重新核实。

## Prepare and book / 准备并预订

Revalidate the chosen room and rate immediately before final review. Surface
any room, bed, view, refundability, fee, or benefit change.

在最终复查之前立即重新验证所选房间与房价。呈现任何房间、床型、景观、可退款性、费用或权益的变化。

At final review include property and address, dates, guests/rooms, exact room
and bed, rate name, full total, due now, due at property, deposit/hold,
inclusions, loyalty/points/certificate use, and cancellation deadline.

最终复查应包含物业与地址、日期、客人/房间数、确切房型与床型、房价名称、总价、现在需付、到店需付、押金/预授权、包含项、常旅客/积分/证书使用情况以及取消期限。

Use secure provider or profile fields for identity, membership, and payment data.
Do not expose full values in chat.

身份、会员与支付数据使用安全的提供商或档案字段。不要在聊天中暴露完整值。

Verify with a provider confirmation number and a reservation visible in the
hotel or booking-platform account when possible. Return the safe confirmation
reference, dates, room, total, due-at-property amount, and cancellation
deadline.

尽可能用提供商确认号以及酒店或预订平台账户中可见的预订来验证。返回安全的确认引用、日期、房间、总价、到店需付金额与取消期限。
