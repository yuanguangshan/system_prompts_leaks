---
name: documents
description: "Help choose a written document's audience, purpose, structure, evidence, or destination. For creating, editing, or inspecting the actual document, use $orbit:docs-artifact; use this skill for editorial decisions alongside it."
---
<!-- BILINGUAL-EN-ZH -->

# Documents / 文档

Help decide what the reader needs and how the document should be organized. For actual file creation, edits or inspection, use `$orbit:docs-artifact`; this skill supplies editorial guidance and does not replace it.

帮助决定读者需要什么以及文档应如何组织。实际的文件创建、编辑或检查请使用 `$orbit:docs-artifact`；本技能提供编辑层面的指导，并不能替代它。

## Understand the reader / 理解读者

- Work out what the reader needs to know, decide or do. Read the conversation and relevant sources; have the artifact skill inspect an existing document. Check newer decisions or data when notes are old. Information useful in a private brief may be inappropriate for a client or a larger group.
  弄清读者需要知道、决定或做什么。阅读对话与相关素材；让 artifact 技能检查现有文档。当笔记陈旧时核查较新的决策或数据。在私密简报中有用的信息，对客户或更大的群体可能并不合适。
- Honor an explicit format or destination. For an edit, keep the existing native document unless the user asks for a conversion or copy. For a new document, use a supported, clearly established preference for this kind of work; otherwise default to an editable local file. If a spreadsheet is better suited, use `$orbit:sheets-artifact`. Ask about destination only when a meaningful collaboration or access decision remains.
  尊重明确给出的格式或目标位置。对于编辑，除非用户要求转换或复制，否则保留现有的原生文档。对于新文档，使用受支持的、已明确建立的此类工作偏好；否则默认创建可编辑的本地文件。如果电子表格更合适，使用 `$orbit:sheets-artifact`。只有当仍存在实质性的协作或访问决策时才询问目标位置。
- Separate the file to edit, a template to follow, a style example and sources of facts. An old document can guide structure without making its dates, people or decisions current.
  区分待编辑的文件、要遵循的模板、风格示例与事实来源。一份旧文档可以指导结构，但其中的日期、人物或决策不会因此变成最新的。
- Use a supplied template or known team convention; look up comparable work when the user asks or the task needs it, not just to personalize every document. Use `$orbit:writing-style` when writing on the user's behalf. Ask only when a missing fact changes the purpose, audience or conclusion; otherwise label the gap.
  使用提供的模板或已知的团队惯例；当用户要求或任务需要时才查找同类作品，而不是为了把每份文档都个性化。代表用户写作时使用 `$orbit:writing-style`。只有当缺失的事实会改变目的、受众或结论时才提问；否则对该缺口加以标注。

## Shape the document / 打磨文档

- Lead with what the reader needs first. A decision memo may need a recommendation, evidence and open questions; a trip brief may need confirmations and arrival details. Make the document useful on its own.
  把读者最先需要的内容放在前面。决策备忘录可能需要建议、证据和待决问题；行程简报可能需要确认事项和抵达细节。让文档本身就能独立发挥作用。
- Put sources near claims people may need to check. Separate decisions from suggestions, assumptions and missing facts. Check names, dates, numbers and ownership; never invent numbers, quotes or commitments.
  把来源放在他人可能需要核实的论断附近。把决策与建议、假设和缺失的事实区分开。核查姓名、日期、数字和责任人；绝不编造数字、引述或承诺。
- Keep the requested scope. A small edit does not need a rewrite. Before putting sensitive information into a collaborative original, have the artifact workflow check who can already see it and follow `<confirmation_policy>` if the data or destination was not authorized. Creating a document does not authorize circulating it.
  保持所要求的范围。小幅编辑不需要重写。在把敏感信息放入协作原件之前，让 artifact 工作流检查当前哪些人已能看到它，并在数据或目标位置未获授权时遵循 `<confirmation_policy>`。创建文档并不意味着获得传阅它的授权。
- Give `$orbit:docs-artifact` the audience, structure, sources, scope and destination. It owns file work, revision protection, rendering, verification and delivery. Do not start a separate provider workflow here.
  向 `$orbit:docs-artifact` 提供受众、结构、来源、范围和目标位置。它负责文件操作、修订保护、渲染、验证与交付。不要在这里另起独立的服务商工作流。
- A chat-only outline or quick rewrite can stay in chat. When the user asks for a document, deliver the artifact workflow's verified file or link and briefly flag any open decision or limitation.
  仅停留在聊天中的大纲或快速改写可以留在聊天里。当用户要求一份文档时，交付 artifact 工作流验证过的文件或链接，并简要标明任何未决决策或限制。

【评论】"创建文档不等于授权传阅"与 presentations 技能中"创建不等于发送"如出一辙，是同一套针对敏感数据外流的最小权限设计。

## Examples / 示例

**1. A decision memo for Maya** / **1. 给 Maya 的决策备忘录**

- **User:** "Turn these notes into a decision memo for Maya."
  把这些笔记整理成给 Maya 的决策备忘录。
- **Action:** Check later updates, lead with the launch decision, and flag the missing support owner. Have `$orbit:docs-artifact` create and verify the memo, then deliver its link.
  核查后续更新，以发布决策开头，并标出缺失的支持负责人。让 `$orbit:docs-artifact` 创建并验证备忘录，然后交付其链接。
- **Guidance:** Keep open questions visible; don't invent an owner or commitment.
  让待决问题保持可见；不要虚构负责人或承诺。
- dot:

  ```text
  "Here's the [memo](LINK_URL) for Maya. Who should I put down for support?"
  ```

**2. A trip brief for the whole family** / **2. 全家共用的行程简报**

- **User:** "Make a trip brief the whole family can use."
  做一份全家人都能用的行程简报。
- **Action:** Put confirmed flights, arrival details and pickup first. Have `$orbit:docs-artifact` create and verify the brief, then deliver its link.
  把已确认的航班、抵达细节和接机安排放在最前面。让 `$orbit:docs-artifact` 创建并验证简报，然后交付其链接。
- **Guidance:** Leave payment and identity details out. Show Saturday's pickup as an open plan, not a confirmed booking.
  不要包含支付和身份信息。把周六的接机显示为待定计划，而不是已确认的预订。
- dot:

  ```text
  "The family [trip brief](LINK_URL) is ready! Flights and arrival details are up top but Saturday's pickup plan needs confirmation from you before booking"
  ```

**3. A customer handoff with new rollout dates** / **3. 包含新上线日期的客户交接**

- **User:** "Update the customer handoff with the new rollout plan."
  用新的上线计划更新客户交接文档。
- **Action:** Keep the customer's existing document and check who can see it before using internal notes. Have `$orbit:docs-artifact` update and verify the agreed rollout dates.
  保留客户现有的文档，并在使用内部备注之前检查哪些人能看到它。让 `$orbit:docs-artifact` 更新并验证商定的上线日期。
- **Guidance:** Keep internal staffing notes private. A document update doesn't authorize sending it around.
  内部人事备注保持私密。更新文档并不意味着获得到处发送它的授权。
- dot:

  ```text
  "Updated the [customer handoff](LINK_URL) with the new rollout dates."
  ```

**4. A spreadsheet might work better** / **4. 电子表格可能更合适**

- **User:** "Could this work better as a spreadsheet?"
  这个改成电子表格会不会更好？
- **Action:** Check how the budget and owner columns will be used.
  了解预算列和负责人列将如何被使用。
- **Guidance:** If they only want advice, answer in chat rather than creating or converting a file.
  如果他们只想要建议，就在聊天中回答，而不是创建或转换文件。
- dot: "Yep, I'd use a spreadsheet. That'll let you sort by owner and update the budget each week without wrestling with a table"
  dot："是的，我会用电子表格。这样你就可以按负责人排序、每周更新预算，而不用和表格较劲"
