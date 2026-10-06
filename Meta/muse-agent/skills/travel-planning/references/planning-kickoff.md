<!-- BILINGUAL-EN-ZH -->
# Complex planning kickoff / 复杂规划的开场

Use this guidance only after the request has been classified as substantial
travel planning. A bounded booking should already have left Travel Planning for
Booking and its provider.

仅在请求已被归类为实质性旅行规划之后才使用本指引。有边界的订票应当早已离开 Travel Planning，转给 Booking 及其提供商。

## Use the right conversation / 使用正确的会话

Read the runtime's current `chat` field. When `chat=side_chat`, keep the work
in that conversation. Do not create a side chat from another side chat.

读取运行时当前的 `chat` 字段。当 `chat=side_chat` 时，把工作留在该会话内。不要从另一个侧聊再创建侧聊。

When substantial planning starts in the main chat and `chat.create` is
available, explain in one or two sentences that a dedicated trip chat keeps the
itinerary, research, and revisions together. Then ask whether the user wants to
move the work there. Do not create the side chat before the user accepts. Do
not ask again after the user chooses to stay in the main chat. When
`chat.create` is unavailable, continue in the main chat and do not offer a
handoff you cannot perform.

当实质性规划从主聊开始且 `chat.create` 可用时，用一两句话解释：专属的行程聊天能把行程、研究和修订放在一起。然后询问用户是否愿意把工作挪过去。用户接受之前不要创建侧聊。用户选择留在主聊之后不要再问。当 `chat.create` 不可用时，在主聊中继续，不要提出你无法执行的移交。

After the user accepts in the main chat:

用户在主聊中接受之后：

1. Call `chat.create` with `context_mode: "fork"` and a short trip-specific
   name. The fork inherits relevant main-chat history.
   用 `context_mode: "fork"` 和一个简短的行程专名调用 `chat.create`。分叉会继承主聊中相关的历史。
2. Call `chat.send_message` with the returned `chat_id` to start the planning
   turn there. The `message` field on `chat.create` is only context for naming;
   it does not deliver a turn.
   用返回的 `chat_id` 调用 `chat.send_message`，在那里启动规划轮。`chat.create` 上的 `message` 字段只是命名的上下文；它不会交付一轮对话。
3. Tell the side-chat agent to continue the `travel_planning` workflow from the
   inherited request. Tell it to inspect relevant connected context before it
   asks questions. Tell it to summarize what is already known. Tell it to ask
   only for material gaps. Do not paste unnecessary private source content into
   the handoff.
   告诉侧聊智能体从继承的请求继续 `travel_planning` 工作流。告诉它在提问之前先检查相关的已连接上下文，概述已知内容，只就实质性缺口提问。不要在移交中粘贴不必要的私有源内容。
4. Do not poll for its answer. When the active client supports `ui.list` and
   `ui.navigate`, discover the `chat.session` target with `ui.list`. Then use
   `ui.navigate` in `perform` mode to open the new chat. Do not claim the new
   chat opened unless `ui.navigate` succeeds.
   不要轮询它的回答。当活动客户端支持 `ui.list` 和 `ui.navigate` 时，先用 `ui.list` 发现 `chat.session` 目标，然后以 `perform` 模式使用 `ui.navigate` 打开新聊天。除非 `ui.navigate` 成功，否则不要声称新聊天已打开。

If chat creation or message handoff fails, continue in the current chat. If the
handoff succeeds but navigation is unsupported or fails, do not duplicate the
planning work. Identify the new side chat so the user can open it. A
provider-backed or Shared Agent chat cannot receive this cross-chat handoff.
Continue where the request originated.

若聊天创建或消息移交失败，在当前聊天中继续。若移交成功但导航不受支持或失败，不要重复规划工作。指明新的侧聊，让用户可以自行打开。提供商托管或 Shared Agent 聊天无法接收这种跨聊天移交。在请求发起之处继续。

## Learn before asking / 先了解，再提问

Do not mine private mail, calendar, memory, or profile data in a shared or
multi-user conversation. Do not surface private-derived details there. Continue
from what the participants stated, or ask the user to continue in a private
Hatch chat before using personal connectors.

在共享或多用户会话中，不要挖掘私人邮件、日历、记忆或档案数据。不要在那里呈现由私有数据派生的细节。从参与者已陈述的内容继续，或请用户先转到私有的 Hatch 聊天，再使用个人连接器。

Build the initial brief from evidence already available for this trip:

用此次旅行已有的证据构建初始简报：

1. Read the user's request, relevant conversation, saved preferences, and
   current profile or location context.
   阅读用户的请求、相关会话、已保存的偏好，以及当前档案或位置上下文。
2. Check relevant visible calendars for hard dates, travel, and out-of-office
   context under the entrypoint's calendar rules.
   按入口点的日历规则，检查相关的可见日历中的确定日期、旅行和外出（out-of-office）背景。
3. When `gmail` or `outlook_mail` appears in the current Skills catalog and is
   not loaded, read it before use. If mail is connected, run narrow read-only
   searches for recent confirmations or receipts in only the categories that
   matter now. An opportunistic personalization read is not a user request to
   connect mail. If connection or read scope is unavailable, skip the read. Do
   not show a connection link. Do not pause the trip. Offer connection only
   when the user asks to connect or use mail. For example:
   当 `gmail` 或 `outlook_mail` 出现在当前技能目录中且尚未加载时，使用前先阅读它。若邮件已连接，只对当下重要的类别运行窄范围的只读搜索，查找近期的确认或收据。一次机会主义的个性化读取并不构成用户连接邮件的请求。若连接或读取范围不可用，跳过读取。不要展示连接链接。不要搁置行程。仅当用户要求连接或使用邮件时才提供连接。例如：
   - flights: repeated airlines, origin airports, cabin, loyalty program,
     baggage, and explicitly chosen seats;
     航班：重复的航空公司、出发机场、舱位、常旅客计划、行李，以及明确选择的座位；
   - stays: repeated brands or property styles, room type, location pattern,
     and cancellation behavior;
     住宿：重复的品牌或物业风格、房型、位置模式和取消行为；
   - rental cars: repeated company, vehicle class, transmission, pickup, and
     insurance choices; and
     租车：重复的公司、车型等级、变速箱、取车方式和保险选择；以及
   - rail, transfers, activities, or dining only when they affect this trip.
     火车、接送、活动或餐饮，仅当它们影响本次旅行时。
4. Summarize the usable brief as stated facts, well-supported observed
   patterns, and unresolved choices. Ask the user to correct material inferred
   defaults. Offer a plausible direction for the user to react to.
   把可用的简报归纳为：已陈述的事实、有充分依据的观察模式，以及未决的选择。请用户纠正重要的推断默认值。给出一个合理方向供用户回应。

Base an inferred preference on repeated, recent, and comparable choices rather
than on a single old booking. First establish that the user, not a spouse,
employee, guest, or other traveler named in the user's inbox, made or used the
choice. A seat assignment, hotel stay, or car class may still reflect
availability, price, another traveler, or a one-off trip. Do not promote one
occurrence into a standing preference. Keep a destination-specific or
party-specific pattern scoped to that destination or party. Do not expose
unrelated mail or any confirmation, ticket, loyalty, or reservation identifier
encountered solely during preference mining. Do not search the whole inbox
without a task-relevant bound. Do not treat account access as authority to book
or spend.

推断的偏好必须基于重复、近期且可比较的选择，而不是一次陈旧的预订。先确认该选择是用户本人做出或使用的，而不是收件箱中提到的配偶、雇员、客人或其他旅行者。一个座位分配、一次酒店入住或一个车型等级，仍可能只反映当时的可用性、价格、同行者或一次性出行。不要把单次出现拔高成长期偏好。目的地特定或同行群体特定的模式要限定在该目的地或该群体之内。不要暴露无关邮件，也不要暴露仅在偏好挖掘过程中遇到的任何确认号、票号、会员号或预订标识。没有任务相关边界时不要搜索整个收件箱。不要把账户访问权当作预订或消费的授权。

【评论】该节对"从邮件推断用户偏好"施加了多重防错：先确认行为主体是用户本人、排除单次事件、限定模式范围、禁止暴露无关标识，体现了对个性化与隐私边界的谨慎处理。

If mail is disconnected or the relevant skill is absent, continue planning. Do
not pressure the user to connect mail solely for personalization. Ask for only
the smallest coherent set of missing facts that materially changes the plan.
Do not walk through a fixed questionnaire.

若邮件未连接或相关技能缺失，继续规划。不要为了个性化而催促用户连接邮件。只索要会对计划产生实质影响的最小连贯事实集。不要按固定问卷逐项过一遍。

## Make place recommendations visual / 让地点推荐可视化

When recommending a real destination, neighborhood, hotel, restaurant,
attraction, or venue, read `places_search` and `image_search` when they appear
in the current Skills catalog and are not already loaded. Use `places_search`
as a grounding and map companion while Travel Planning owns the composed trip
answer. Provider images, such as current event art or seat views, may be more
decision-useful than a generic photo.

推荐真实的目的地、街区、酒店、餐厅、景点或场所时，若 `places_search` 和 `image_search` 出现在当前技能目录中且尚未加载，先阅读它们。Travel Planning 负责组合后的行程回答，`places_search` 用作事实依据和地图伙伴。提供商图片（例如当前活动的宣传图或座位视角）可能比通用照片更有助于决策。

- Resolve the exact recommended place first, then choose one representative
  photo whose subject actually matches it. Keep the photo beside its option.
  先解析确切的推荐地点，再选择一张主体确实与之匹配的代表性照片。把照片放在其选项旁边。
- Prefer current provider or place-detail photos. Otherwise use an
  `image_search` result and retain its source page for provenance.
  优先使用当前的提供商照片或地点详情照片。否则使用 `image_search` 结果，并保留其来源页以备溯源。
- For an `image_search` result, use its `media_url` when one is present.
  Otherwise use its `thumbnail_cdn_url`. Preserve the chosen locator
  byte-for-byte. Retain `page_url` as the source. Do not try to render a
  non-HTTP `thumbnail_url`, an internal handle, or a candidate reference.
  对 `image_search` 结果，有 `media_url` 时使用 `media_url`，否则使用 `thumbnail_cdn_url`。选定的定位符逐字节保留。`page_url` 作为来源保留。不要试图渲染非 HTTP 的 `thumbnail_url`、内部句柄或候选引用。
- On a chat surface that renders remote images, keep the returned image beside
  its exact option. Follow a provider's documented image-rendering contract
  when using provider media.
  在能渲染远程图片的聊天界面上，把返回的图片放在其确切选项旁边。使用提供商媒体时，遵循该提供商已文档化的图片渲染契约。
- Use real photos for real places. Do not generate a realistic-looking
  substitute, scrape an arbitrary page, guess a CDN URL, or reuse one photo for
  several candidates.
  真实地点用真实照片。不要生成逼真的替代图、不要抓取任意页面、不要猜测 CDN URL，也不要把一张照片复用给多个候选。
- Show one photo per recommended option as the default so the set stays
  scannable. A photo is qualitative context. Do not treat a photo as evidence
  of current hours, condition, inventory, or price.
  默认每个推荐选项展示一张照片，保持整组可扫读。照片是定性背景。不要把照片当作当前营业时间、状况、库存或价格的证据。
- Use a native map or provider visual when it improves the decision. A side
  chat cannot rely on option widgets. In a side chat, pair supported inline
  photos with concise text and verified links.
  能改进决策时使用原生地图或提供商视觉元素。侧聊不能依赖选项组件。在侧聊中，把受支持的内联图片与简洁文本和已验证的链接配对。
- When a verified photo cannot be rendered, omit the photo. Do not fabricate a
  photo. Give the user the grounded recommendation with its source or map.
  当无法渲染经过验证的照片时，省略照片。不要伪造照片。把有依据的推荐连同其来源或地图交给用户。
