---
name: slides-artifact
description: "Use when the user asks to create, edit, or review slides or a presentation (PowerPoint or Google Slides), including when the task clearly calls for an audience-ready slide deck. Also use to answer questions that require inspecting an existing deck; not for general presentation advice or a chat-only outline."
---
<!-- BILINGUAL-EN-ZH -->

# Slide Artifacts / 幻灯片制品

Use this for creating, editing or faithfully inspecting a real slide deck. A chat-only outline or general presentation advice can stay in chat. A question about an existing deck does not authorize changing it.

本技能用于创建、编辑或如实检视真实的幻灯片文稿。仅存在于聊天中的大纲或一般性演示建议可以留在聊天里。针对现有文稿的提问并不意味着获准修改它。

## Reasoning effort for new artifacts / 新制品的推理力度

When creating a new artifact, explicitly set the subagent's reasoning effort to `xhigh`.

创建新制品时，将子代理的推理力度显式设置为 `xhigh`。

## Start with the shared skill / 从共享技能开始

- Before reading, extracting, reviewing, creating or editing slides, find and follow the shared runtime skill named `Presentations`. If `skills.read` is available, use the package listed for that shared skill; do not guess a path or select `$orbit:presentations` or `$orbit:slides-artifact` as the shared skill. Give it the brief, deck or template, audience, desired count and scope. It owns supported inspection, design execution, rendering, quality checks, exports and Google Slides routing. If it is unavailable, say what is blocked rather than inventing a parallel file workflow.
  在读取、提取、审阅、创建或编辑幻灯片之前，先找到并遵循名为 `Presentations` 的共享运行时技能。如果 `skills.read` 可用，使用该共享技能所列出的包；不要猜测路径，也不要把 `$orbit:presentations` 或 `$orbit:slides-artifact` 当作共享技能。向它提供需求简报、文稿或模板、受众、期望的页数和范围。受支持的检视、设计执行、渲染、质量检查、导出以及 Google Slides 路由均由它负责。如果它不可用，请说明哪些环节受阻，而不是另造一套平行的文件工作流。
- Use the editorial guidance in `$orbit:presentations` only when the audience or story needs thought, and `$orbit:writing-style` when writing on the user's behalf; then continue with the shared skill without routing back into this wrapper. Pass along known conventions or the supplied template. Find comparable decks when asked or when needed; don't routinely scan Drive or Library to personalize.
  仅当受众或叙事需要斟酌时才使用 `$orbit:presentations` 中的编辑指导，在代用户撰写时使用 `$orbit:writing-style`；然后继续使用共享技能，不要路由回这个包装层。传递已知的惯例或所提供的模板。在被要求或确有需要时查找可类比的文稿；不要例行公事地扫描 Drive 或资料库来做个性化。
【评论】该技能把实际执行权委托给共享技能，自身仅作为路由与约束层，这是多技能体系中常见的分层设计。

## Make and deliver it / 制作并交付

- Honor an explicit format or supported destination, including Google Slides, Figma or Spaces. Keep an existing native deck in the original unless the user asks for a copy or conversion. For a new deck, use a supported, clearly established preference; otherwise create a downloadable editable local presentation. A PDF does not replace a requested editable deck. Before adding sensitive information to a collaborative original, check who already has access and follow `<confirmation_policy>` if the data or destination was not authorized.
  遵从明确的格式或受支持的目标位置，包括 Google Slides、Figma 或 Spaces。除非用户要求复制或转换，否则保持现有原生文稿的原格式。对于新文稿，使用受支持且已明确确立的偏好；否则创建可下载、可编辑的本地演示文稿。PDF 不能替代用户要求的可编辑文稿。在向协作共享的原件添加敏感信息之前，先查清谁已有访问权限；如果数据或目标位置未获授权，则遵循 `<confirmation_policy>`。
- For inspection, use the shared skill's supported read-only route and check relevant visuals, chart labels or speaker notes; for authored work, inspect every slide and the exact export when supported. Preserve edit scope, and re-read an uncertain write before retrying. Attach the verified deck when supported or provide its verified Library/download or provider link. State any unverified export, conversion or link; follow `<confirmation_policy>` before changing access or sending to others.
  检视时，使用共享技能受支持的只读路径，并核对相关的视觉内容、图表标签或演讲者备注；对于自己创作的内容，在受支持的情况下检查每一页幻灯片和确切的导出结果。保持编辑范围不变，重试前先重新读取一次不确定的写入。在受支持的情况下附上已验证的文稿，或提供其经验证的资料库/下载或服务商链接。说明任何未经验证的导出、转换或链接；在更改访问权限或发送给他人之前遵循 `<confirmation_policy>`。

## Examples / 示例

**1. A leadership update with editable charts**

**1. 带可编辑图表的管理层汇报**

- **User:** "Make a six-slide leadership update with editable charts."
- **Action:** Create six slides, check the figures and headlines, and keep the charts editable.
- **Guidance:** Verify the deck and deliver the editable file, not just a PDF.
- dot:

  ```text
  "Here's your [six-slide deck](LINK_URL). The adoption chart uses September's numbers, and you can edit the charts."
  ```

- **User:** "帮我做一份六页、图表可编辑的管理层汇报。"
- **Action / 动作：** 创建六页幻灯片，核对数字和标题，并保持图表可编辑。
- **Guidance / 指导：** 验证文稿并交付可编辑文件，而不仅仅是 PDF。
- dot:
【评论】dot 是该体系中对代理对外话术的称呼，示例中用于固定输出风格。

  ```text
  "Here's your [six-slide deck](LINK_URL). The adoption chart uses September's numbers, and you can edit the charts."
  ```
【评论】代码块内的内容按规格要求原样保留，不添加中文。

**2. A focused customer-deck update**

**2. 针对性的客户文稿更新**

- **User:** "Update slides 3–5 of the customer deck and add speaker notes."
- **Action:** Update only those slides and their notes, then return the original provider link.
- **Guidance:** Preserve the rest of the deck and check the edited slides before delivering.
- dot:

  ```text
  "Updated slides 3–5 in your [customer deck](LINK_URL), with speaker notes for each."
  ```

- **User:** "更新客户文稿的第 3–5 页，并添加演讲者备注。"
- **Action / 动作：** 只更新这几页幻灯片及其备注，然后返回原始的服务商链接。
- **Guidance / 指导：** 保持文稿其余部分不变，交付前检查已编辑的页面。
- dot:

  ```text
  "Updated slides 3–5 in your [customer deck](LINK_URL), with speaker notes for each."
  ```

**3. Two different pilot dates**

**3. 两个不一致的试点日期**

- **User:** "Does this deck say when the pilot starts?"
- **Action:** Inspect both the slides and speaker notes without editing.
- **Guidance:** Report the conflicting dates rather than choosing one. A question about the deck is not permission to change it.
- dot: "There are two dates in there: October on the slides, September in slide 7's notes."

- **User:** "这份文稿有说试点什么时候开始吗？"
- **Action / 动作：** 只读取幻灯片和演讲者备注，不做编辑。
- **Guidance / 指导：** 报告相互冲突的日期，而不是自行选一个。针对文稿的提问并不是修改文稿的许可。
- dot: "There are two dates in there: October on the slides, September in slide 7's notes."

**4. Internal figures in a customer's deck**

**4. 出现在客户文稿中的内部数据**

- **User:** "Put the internal churn breakdown in the customer's shared deck."
- **Action:** Check who can access the deck and which data the user authorized for that destination. If the scope is unclear, ask before adding it.
- **Guidance:** Follow `<confirmation_policy>` for the specific data and destination.
- dot: "The customer can see that deck. Which churn figures are okay to share with them?"

- **User:** "把内部流失率明细放进客户那份共享文稿里。"
- **Action / 动作：** 查明谁可以访问该文稿，以及用户为该目标位置授权了哪些数据。如果范围不明确，先询问再添加。
- **Guidance / 指导：** 针对具体数据和目标位置遵循 `<confirmation_policy>`。
- dot: "The customer can see that deck. Which churn figures are okay to share with them?"

**5. A PDF copy that still needs a layout check**

**5. 仍需版式检查的 PDF 副本**

- **User:** "Can you send the deck as a PDF too?"
- **Action:** Export the PDF and provide it with the editable deck. In this example, the PDF exists but visual verification is unavailable.
- **Guidance:** Make the remaining layout check clear; don't imply that an uninspected export is ready to share.
- dot:

  ```text
  "Here's the [editable deck](LINK_URL), plus a [PDF copy](LINK_URL). The PDF still needs a quick layout check before you share it"
  ```

- **User:** "能不能把文稿也发一份 PDF？"
- **Action / 动作：** 导出 PDF，并将其与可编辑文稿一起提供。在本例中，PDF 已生成，但无法进行视觉验证。
- **Guidance / 指导：** 明确说明尚待完成的版式检查；不要暗示未经检查的导出结果已可分享。
- dot:

  ```text
  "Here's the [editable deck](LINK_URL), plus a [PDF copy](LINK_URL). The PDF still needs a quick layout check before you share it"
  ```

**6. A stale launch date**

**6. 过时的发布日期**

The active launch deck still says October 8, but a more recent decision confirms October 15. The user has not asked for a deck update, and no standing authorization covers it.

正在使用的发布文稿仍写着 10 月 8 日，但更近的一项决定确认为 10 月 15 日。用户并未要求更新文稿，也没有任何长期授权覆盖这一操作。

- **Action:** On a wake, review your own `/action_items.md`, the delivered heartbeat finding, the current deck, the latest decision, and what you already sent. If the finding is timely, offer to correct the shared deck. Do not change it before authorization.
- **Guidance:** Read-only inspection is already allowed; finding a stale date does not authorize an edit. If a separate meeting-prep finding is also timely, combine the related findings into one nudge. Standing permission to "Keep my BBVA meeting prep up to date" covers that prep, not this deck or contacting BBVA; use `$orbit:docs-artifact` for the prep document.
- dot:

  ```text
  "The [deck](LINK_URL) still says October 8, but the latest [decision](LINK_URL) has October 15. Should I fix the date?"
  ```

- **Action / 动作：** 在被唤醒时，检查自己的 `/action_items.md`、已送达的心跳发现、当前文稿、最新决定，以及你已发送的内容。如果该发现仍然有时效性，主动提出修正共享文稿。在获得授权之前不要更改它。
- **Guidance / 指导：** 只读检视本就允许；发现过时日期并不意味着获得编辑授权。如果另一项会议准备方面的发现同样有时效性，将相关发现合并为一次提醒。"让我的 BBVA 会议准备保持最新"这一长期许可只覆盖该准备工作，不覆盖这份文稿，也不覆盖联系 BBVA；准备文档应使用 `$orbit:docs-artifact`。
- dot:

  ```text
  "The [deck](LINK_URL) still says October 8, but the latest [decision](LINK_URL) has October 15. Should I fix the date?"
  ```
