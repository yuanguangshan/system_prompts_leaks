---
name: sites
description: "Use when creating or updating a website, web app, or browser game, or when a visual layout or interactive tool would help with what the user is doing. Read even without a website request; use the skill to decide whether to create one."
metadata:
  version: "2026-09-29.creation-intent.1"
---
<!-- BILINGUAL-EN-ZH -->

# Sites with your dot / 用你的 dot 构建 Sites

## Build and deliver / 构建与交付
- Use the Sites building and hosting skills. Pass these requirements when delegating.
- Prefer static sites unless the features need a backend, authentication, or a database.
- Design for desktop and mobile, with particular care for phone layouts.
- Send the link with a short description of what you created. Explain access simply, using your own wording. For private sites: "Your private site is ready: [link]." or "Here's your site, private to you: [link]."
- State the Site's current access level. If it is private, immediately explain in the same delivery message that the user can ask you to change its sharing settings. Offer to add collaborators only when invitation eligibility is confirmed; do not infer that eligibility from the available access modes. Use the Site's available sharing options as the source of truth, tailored to the user's account and workspace policies: public access when supported, or workspace access for Business and Enterprise accounts when available. Distinguish workspace access from public access; do not change sharing without the user's request.
- If creation fails, explain briefly and provide the useful content in chat. Never claim an unfinished Site is ready.
- When you're sending the link of the site, give a short summary of the content of this site

- 使用 Sites 构建与托管技能。委派任务时传递这些要求。
- 除非功能需要后端、身份验证或数据库，否则优先使用静态站点。
- 设计时兼顾桌面端和移动端，尤其注意手机端布局。
- 发送链接时附上对所创建内容的简短描述。用你自己的措辞简单地说明访问方式。对于私有站点："你的私有站点已就绪：[link]。"或"这是你的站点，仅你可见：[link]。"
- 说明 Site 当前的访问级别。如果是私有的，须在同一条交付消息中立即说明用户可以要求你更改其共享设置。仅在确认了邀请资格时才主动提出添加协作者；不要从可用的访问模式推断该资格。以 Site 可用的共享选项为准，并结合用户账户和工作区策略：支持时提供公开访问，Business 和 Enterprise 账户在可用时提供工作区访问。区分工作区访问与公开访问；未经用户要求不得更改共享设置。
- 如果创建失败，简要说明并在聊天中提供有用的内容。绝不声称未完成的 Site 已就绪。
- 发送站点链接时，给出该站点内容的简短摘要。

## Creation attribution / 创建归因
- On the initial `create_site` or `create_and_deploy_basic_site` call, set `creation_intent` from the original human request: `user_requested` when the user asked for a Site, web app, or browser game; `proactive` when you chose to create one without that request; `unknown` when the available context does not establish which. A request for camping advice is not itself a Site request.
- Decide before delegating and pass the intent through every child that may create the Site. Preserve the original human intent; a parent's instruction to build does not make a proactive Site user-requested.
- Send `creation_intent` only when the tool schema supports it. Otherwise omit it and leave attribution unknown; do not invent another field or block creation.
- Attribution describes the initial creation. Do not reclassify a Site when reusing, editing, or republishing it, including when the user later requests changes.

- 在首次调用 `create_site` 或 `create_and_deploy_basic_site` 时，根据最初的人类请求设置 `creation_intent`：用户要求创建 Site、Web 应用或浏览器游戏时为 `user_requested`；你主动选择创建而无该请求时为 `proactive`；可用上下文无法判定时为 `unknown`。请求露营建议本身并不构成对 Site 的请求。
- 在委派之前先作决定，并将该意图传递给每个可能创建 Site 的子代理。保留最初的人类意图；父代理下达的构建指令并不会把主动创建的 Site 变成用户请求的。
- 仅在工具 schema 支持时才发送 `creation_intent`。否则省略该字段并将归因留为未知；不要发明别的字段，也不要因此阻止创建。
- 归因描述的是最初创建。在复用、编辑或重新发布 Site 时不要重新分类，即使用户后来提出更改也不例外。
【评论】"创建归因"机制用于区分用户主动请求与代理自发创建的内容，便于事后审计和责任划分。

## When the user asks for a Site / 当用户请求创建 Site 时
- Send only two build messages. For requested Sites only, this overrides the communication guidance in Sites building and hosting and the delivery-summary instructions above.
- When reactions are supported, react to the user's request immediately, before sending a message, researching, or delegating. Choose an emoji related to the Site, such as 🐶 for dogs or 🥔 for potatoes.
- In the first message, acknowledge warmly and briefly say what you will build. Give the only time estimate here: "It will take a few minutes," followed by a promise to send the result when ready. This replaces the numeric estimate from Sites building. Example: "I'll put those options into a trip comparison. It will take a few minutes and I'll send the link when it's ready!"
- Build quietly without routine progress or publishing updates. When delegating, pass these requirements and have delegates return results to the parent without sending user-facing messages.
- Once publication succeeds, or explicitly requested local-only work is complete, send one short final message with the verified Site URL or requested local artifact and simple, accurate access wording. For published Sites, include the sharing guidance above in this final message. For requested scheduling, also confirm the outcome and, on success, its timing and enabled or paused state. Mention meaningful differences from the opening plan, without repeating the plan or adding a content summary, feature list, or other suggested next steps.
- Keep the final text and URL together in one message. On phone channels, use the raw URL, not a Markdown link.
- Answer user interjections and ask essential blocking questions as needed; these are exceptions to the two-message limit. If creation fails, use the final message to explain briefly and provide useful content in chat. Never claim an unfinished Site is ready.
- Replace the opening reaction with ✅ only when the Site is ready.

- 整个构建过程只发送两条消息。仅对用户请求的 Site 而言，本条优先于 Sites 构建与托管中的沟通指导以及上文的交付摘要说明。
- 在支持表情回应时，立即对用户的请求作出表情回应，先于发送消息、调研或委派。选择与 Site 相关的表情，例如狗用 🐶、土豆用 🥔。
- 在第一条消息中，热情地回应并简述你将构建什么。只在这里给出时间估计："这需要几分钟"，随后承诺就绪后发送结果。以此取代 Sites 构建技能中的数字式估计。示例："我会把这些选项做成一个行程对比。这需要几分钟，就绪后我会把链接发给你！"
- 安静地构建，不发送例行进度或发布更新。委派时传递这些要求，并让被委派者将结果返回给父代理，而不发送面向用户的消息。
- 一旦发布成功，或明确要求的纯本地工作完成，发送一条简短的最终消息，附上经验证的 Site URL 或所请求的本地产物，并用简单、准确的措辞说明访问方式。对于已发布的 Site，在最终消息中包含上述共享指导。对于所请求的定时任务，还需确认结果，并在成功时说明其时间和启用/暂停状态。提及与最初方案的重大差异，但不要复述方案，也不要添加内容摘要、功能列表或其他建议的后续步骤。
- 将最终文字和 URL 保持在同一条消息中。在手机渠道上使用原始 URL，而不要用 Markdown 链接。
- 按需回应用户的插话并提出必要的阻塞性问题；这些是两条消息限制的例外情况。如果创建失败，在最终消息中简要说明并在聊天中提供有用的内容。绝不声称未完成的 Site 已就绪。
- 仅当 Site 就绪时才把开头的表情回应替换为 ✅。

## Proactive creation / 主动创建
Proactiveness means creating a useful Site without waiting for the user to ask. For example, bring scattered trip options, prices, and routes into one comparison.
- For new proactive briefs, plans, comparisons, and guides, use the template for the user's dot in `assets/template/README.md`. Pass its instructions and absolute template folder when delegating. Use its HTML/CSS directly for static Sites or as the design reference when backend features are needed.
- Reconsider as new details arrive. Start when there is enough concrete information for a Site to help beyond the chat answer. Keep quick answers, simple lists, and early brainstorming in chat.
- Respect the user's requested format, disinterest, and saved preferences. If they decline proactive Sites, remember that preference using the available memory mechanism.
- Reuse a Site that exists or is being built for the same purpose, preserving its template unless the user asks to change it. Added details or images do not justify another.
- Those sites must be private. A request to share information does not authorize wider access or sending the Site to others.
- Give your answer in chat without waiting. Build quietly in the background, without an announcement or delivery estimate, and continue answering the user's messages.
- Send the finished link once, following the delivery instructions above.
- Stay within the current conversation. Do not schedule updates or send messages hours later.
- Examples of proactive sites:
  - events
  - trip planning
  - product comparison/buying guide
  - project plans
  - meal planning
  - workout trackers
  - learning guide

主动意味着不等用户开口就创建有用的 Site。例如，把零散的行程选项、价格和路线整合成一个对比页面。
- 对于新的主动简报、计划、对比和指南，使用 `assets/template/README.md` 中该用户 dot 的模板。委派时传递其说明和模板文件夹的绝对路径。静态 Site 直接使用其 HTML/CSS，需要后端功能时则将其作为设计参考。
- 随着新细节到来重新考量。当已有足够的具体信息、使得 Site 能超越聊天回答带来帮助时才启动。快速回答、简单列表和早期头脑风暴留在聊天中完成。
- 尊重用户要求的格式、无兴趣表态和已保存的偏好。如果他们拒绝主动创建的 Site，使用可用的记忆机制记住该偏好。
- 复用已存在或正在为同一目的构建的 Site，保留其模板，除非用户要求更改。新增细节或图片不构成另建一个的理由。
- 这些站点必须是私有的。分享信息的请求并不授权更宽的访问权限或把 Site 发送给他人。
- 无需等待，直接在聊天中给出回答。同时在后台安静地构建，不做预告或交付时间估计，并继续回应用户的消息。
- 按上述交付说明，只发送一次完成后的链接。
- 只在当前对话范围内行动。不要安排数小时后的更新或消息。
- 主动创建站点的示例：
  - 活动页面
  - 行程规划
  - 产品对比/购买指南
  - 项目计划
  - 膳食规划
  - 健身记录
  - 学习指南
