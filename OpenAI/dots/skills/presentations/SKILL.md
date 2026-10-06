---
name: presentations
description: "Help choose a presentation's audience, story, outline, or use of evidence and visuals. For creating, editing, or inspecting actual slides, use $orbit:slides-artifact; use this skill for editorial decisions alongside it."
---
<!-- BILINGUAL-EN-ZH -->

# Presentations / 演示文稿

Help shape what the audience should understand or decide. For actual slide creation, edits or inspection, use `$orbit:slides-artifact`; this skill supplies the story and does not replace it.

帮助确定受众应当理解或决定的内容。实际的幻灯片创建、编辑或检查请使用 `$orbit:slides-artifact`；本技能负责提供故事线，并不能替代它。

## Find the story / 找到故事线

- Read the brief and relevant sources. Work out who will see the deck, what they know, what they need to decide, and whether it is a live talk or something to read alone. Check for recent decisions or figures before reusing old claims; look up comparable decks only when the user asks or the task needs them.
  阅读需求说明与相关素材。弄清谁会看到这份演示、他们已经了解什么、需要决定什么，以及它是现场演讲还是供独自阅读的材料。在复用旧结论之前，先核查近期的决策或数据；仅在用户要求或任务需要时才查找同类演示。
- Choose the point the audience should leave with and how each section supports it. Draft the outline; ask only if an unknown decision or claim changes the story. Use `$orbit:writing-style` for the user's voice and pass along relevant conventions for slide titles, density, charts, branding and notes.
  确定受众离场时应带走的核心观点，以及每个部分如何支撑它。起草大纲；只有当未知的决策或说法会改变故事线时才提问。使用 `$orbit:writing-style` 把握用户的行文口吻，并传递幻灯片标题、信息密度、图表、品牌和备注方面的相关惯例。
- Match the requested scope. Updating three charts does not call for a new design or a reordered deck. Use a supplied template or established conventions where relevant.
  匹配所要求的范围。更新三张图表并不意味着需要全新设计或重排整份演示。在相关之处使用提供的模板或既定惯例。

## Shape the slides / 打磨幻灯片

- Choose titles and visuals that carry the point. A chart should answer a question; use a table or words when clearer. Put extra detail in notes or an appendix where useful. A deck meant to stand alone needs enough explanation on the slides.
  选择能承载观点的标题与视觉元素。图表应当回答一个问题；当表格或文字更清晰时则改用它们。在有用之处把补充细节放入备注或附录。需要独立阅读的演示，其幻灯片本身要有足够的解释。
- Ground claims, charts and quotes in actual sources. Check units, dates, comparison periods, scales and whether old headlines still fit. Include sources for important claims. Label estimates, mockups and unresolved decisions; never invent results or product screenshots.
  让论断、图表和引述都以真实来源为依据。核查单位、日期、对比区间、刻度，以及旧结论是否仍然成立。为重要论断注明来源。对估算、样机和未决决策加以标注；绝不编造结果或产品截图。
- Give `$orbit:slides-artifact` the audience, story, sources, scope, template and destination. It owns provider routing, design execution, rendering, exports and verification; do not start a separate provider workflow here. Follow an explicit destination; otherwise keep an existing native deck, use a supported established preference for a new one, or default to an editable local deck.
  向 `$orbit:slides-artifact` 提供受众、故事线、来源、范围、模板和目标位置。它负责服务商路由、设计执行、渲染、导出与验证；不要在这里另起独立的服务商工作流。遵循明确给出的目标位置；否则保留现有的原生演示文稿，新建时使用受支持的既定偏好，或默认创建可编辑的本地演示文稿。
- Before adding sensitive information to a collaborative deck, have the artifact workflow check who already has access and follow `<confirmation_policy>` if the data or destination was not authorized. Creating a deck does not authorize sending it. A chat-only outline or presentation advice can stay in chat.
  在向协作演示文稿添加敏感信息之前，让 artifact 工作流检查当前哪些人已有访问权限，并在数据或目标位置未获授权时遵循 `<confirmation_policy>`。创建演示文稿并不意味着获得发送它的授权。仅停留在聊天中的大纲或演示建议可以留在聊天里。

【评论】"创建演示文稿不等于授权发送"是一条最小权限式的边界条款，用于防止内容被越权外发到协作渠道之外。

## Examples / 示例

- **User:** "Refresh the board deck for Friday."
  为周五的董事会演示换上新内容。
  - **Action:** Check whether the new figures still support the old headlines and make the board's decision clear.
    检查新数据是否仍支持旧的结论点，并让董事会需要做的决策清晰明确。
  - dot: "I moved the decision up front and the new numbers no longer support last month's growth headline"
    dot："我把决策内容提到了最前面，而且新数据已无法支撑上个月的增长结论"
- **User:** "Make a demo for the customer who asked about onboarding."
  为咨询过上手流程（onboarding）的客户做一个演示。
  - **Action:** Focus on those questions and use real screenshots or label mockups.
    聚焦这些问题，并使用真实截图或将样机明确标注。
  - dot: "The outline centers on setup and permissions, plus I labeled the proposed screen as a mockup"
    dot："大纲围绕设置与权限展开，另外我把提议的界面标注为了样机"
- **User:** "Just help me outline a five-minute update."
  只帮我列一个五分钟更新的提纲。
  - **Action:** Keep it in chat.
    留在聊天中即可。
  - dot: "I'd use three beats: what shipped, what's blocked, and the decision you need"
    dot："我会用三个要点：已交付的内容、受阻的事项，以及需要你拍板的决策"
- **User:** "Update only slides 3–5."
  只更新第 3–5 页幻灯片。
  - **Action:** Preserve the rest and route the file through the artifact skill.
    保持其余部分不变，并将文件交由 artifact 技能处理。
  - dot: "Slides 3–5 are updated and the forecast is still labeled as an estimate"
    dot："第 3–5 页已更新，预测数字仍标注为估算"
