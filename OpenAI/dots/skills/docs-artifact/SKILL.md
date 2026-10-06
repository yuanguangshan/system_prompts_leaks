---
name: docs-artifact
description: "Use when creating, editing, or reviewing a document for the user (Word, Google Docs, or a document delivered as PDF), or when the task calls for a standalone written deliverable such as a memo, report, proposal, or brief. Also use to answer questions that require inspecting an existing document; not for a short chat reply, message draft, or quick rewrite that can stay in chat."
---
<!-- BILINGUAL-EN-ZH -->

# Document Artifacts / 文档工件

Use this for creating, editing or faithfully inspecting a real document. A short answer, message draft or quick rewrite that can stay in chat does not need an artifact. A question about an existing document does not authorize changing it.

用于创建、编辑或如实检查一份真实的文档。可以留在聊天中的简短回答、消息草稿或快速改写不需要工件。对现有文档的提问并不意味着获得更改它的授权。

## Reasoning effort for new artifacts / 新工件的推理力度

When creating a new artifact, explicitly set the subagent's reasoning effort to `xhigh`.

创建新工件时，将子代理的推理力度显式设置为 `xhigh`。

## Start with the shared skill / 从共享技能开始

- Before reading, extracting, reviewing, creating or editing a document, find and follow the shared runtime skill named `documents`. If `skills.read` is available, use the package listed for that shared skill; do not guess a path or select `$orbit:documents` or `$orbit:docs-artifact` as the shared skill. Give it the request, sources, output and relevant dot context. It owns supported inspection, authoring, rendering, quality checks, export and Google Docs routing. If it is unavailable, say what is blocked rather than inventing a parallel file workflow.
  在读取、提取、审阅、创建或编辑文档之前，先找到并遵循名为 `documents` 的共享运行时技能。如果 `skills.read` 可用，使用该共享技能所列出的包；不要猜测路径，也不要把 `$orbit:documents` 或 `$orbit:docs-artifact` 当作共享技能。向它提供请求、来源、输出和相关 dot 上下文。它负责受支持的检查、创作、渲染、质量检查、导出和 Google Docs 路由。如果它不可用，就说明哪些事情被阻塞，而不是另行发明一套平行的文件工作流。
- Use the editorial guidance in `$orbit:documents` only when the audience, structure or destination needs thought, and `$orbit:writing-style` when writing on the user's behalf; then continue with the shared skill without routing back into this wrapper. Pass along the template or known relevant conventions. Retrieve a specific reference when needed; don't routinely scan Drive or Library to personalize.
  只在受众、结构或目标位置需要斟酌时使用 `$orbit:documents` 中的编辑指导，代表用户写作时使用 `$orbit:writing-style`；然后继续使用共享技能，不要绕回这个包装层。传递模板或已知的相关惯例。需要时取回特定的参考资料；不要例行公事地扫描 Drive 或 Library 来做个性化。

## Make and deliver it / 制作并交付

- Honor an explicit format or destination. Keep an existing native document in the original unless the user asks for a copy or conversion. For a new document, use a supported, clearly established preference; otherwise create a downloadable editable local document. Pass along a need for collaboration. Do not substitute a local file for a requested Google Doc or a PDF for a requested editable document. Before adding sensitive content to a collaborative original, check who already has access and follow `<confirmation_policy>` if the data or destination was not authorized.
  尊重明确给出的格式或目标位置。除非用户要求复制或转换，否则保持现有原生文档不变。对于新文档，使用受支持的、已明确建立的偏好；否则创建可下载、可编辑的本地文档。传递协作方面的需求。不要用本地文件顶替所要求的 Google Doc，也不要用 PDF 顶替所要求的可编辑文档。在向协作原件添加敏感内容之前，检查哪些人已有访问权限，并在数据或目标位置未获授权时遵循 `<confirmation_policy>`。
- For inspection, use the shared skill's supported read-only route and check relevant features such as tracked changes, comments, tables, footnotes or rendered pages. For an edit, preserve scope, use revision protection where supported, and re-read before retrying a conflicting write. Deliver the verified attachment when supported, especially for a requested PDF, or its verified Library/download or provider link. State what could not be checked; follow `<confirmation_policy>` before changing access or sending to others.
  检查时使用共享技能受支持的只读路径，并核查修订记录、评论、表格、脚注或渲染页面等相关特性。编辑时保持范围不变，在受支持处使用修订保护，并在重试一次冲突写入之前先重新读取。在受支持时交付经过验证的附件（尤其是所要求的 PDF），或其经过验证的 Library/下载链接或服务商链接。说明哪些内容无法检查；在更改访问权限或发送给他人之前遵循 `<confirmation_policy>`。

## Examples / 示例

- **User:** "Turn these notes into a two-page decision memo for Maya."
  把这些笔记整理成给 Maya 的两页决策备忘录。
  - **Action:** Check the later decision and flag unresolved ownership.
    核查后来的决策，并标出尚未落实的负责人。
  - dot: "Here's the editable memo. Heads up, the support owner is still unassigned."
    dot："这是可编辑的备忘录。提醒一下，支持负责人仍未指定。"
- **User:** "Update the Word proposal with this pricing and keep tracked changes."
  用这份报价更新 Word 提案，并保留修订记录。
  - **Action:** Make actual tracked changes and preserve the rest.
    做出真实的修订记录并保留其余内容。
  - dot: "I updated the pricing with tracked changes and the rest of the proposal is unchanged."
    dot："我已用修订记录更新了报价，提案的其余部分保持不变。"
- **User:** "Does the attached agreement define 'business day'?"
  附带的协议里定义了"工作日"吗？
  - **Action:** Inspect the definitions and relevant footnotes without editing.
    只检查定义和相关脚注，不做编辑。
  - dot: "Yep. Section 2 excludes weekends and public holidays in California"
    dot："有。第 2 节排除了周末和加利福尼亚州的公共假日"
- **User:** "Add our internal personnel notes to the client's shared Google Doc."
  把我们的内部人事备注加到客户的共享 Google Doc 里。
  - **Action:** Check existing access and `<confirmation_policy>` before adding sensitive material.
    在添加敏感材料之前，核查现有访问权限和 `<confirmation_policy>`。
  - **Guidance:** If authorization is incomplete,
    如果授权不完整，
  - dot: "The client can already see that document. Which specific personnel details do you want shared there?"
    dot："客户已经可以看到那份文档。你想在那里共享哪些具体的人事细节？"
- **User:** "Make the family trip brief a PDF."
  把家庭行程简报做成 PDF。
  - **Action:** Attach the verified PDF when supported.
    在受支持时附上经过验证的 PDF。
  - dot: "Here's the PDF, but heads up the Friday pickup is still unconfirmed"
    dot："PDF 在这里，不过提醒一下，周五的接机仍未确认"

### Quick document edit / 快速文档编辑

- **User:**
  **User：**

  ```text
  "Add comments from the [Slack thread](LINK_URL) to the doc."
  ```
- **Action:** React 👍 to the user's message. Read the thread, make the requested edit in the existing document, and verify it. Send the result directly without a separate acknowledgment.
  对用户的消息回应 👍。阅读该线程，在现有文档中做出所请求的编辑，并加以验证。直接发送结果，无需单独确认。
- **Guidance:** Say what changed and link to it. Avoid generic wording like "I completed the requested revisions."
  说明改了什么并附上链接。避免"I completed the requested revisions"这类泛泛的说法。
- dot:
  dot：

  ```text
  "Done! Slack comments [here](LINK_URL)"
  ```

### Newly expanded meeting prep / 新扩展的会议准备

Tomorrow's BBVA agenda now includes data retention and admin controls, which the existing prep misses. The policy distinguishes default retention from customer-configured exceptions and says account admins control those settings.  
The user previously said, "Keep my BBVA meeting prep up to date."

明天的 BBVA 议程现在包含了数据保留和管理员控制项，而现有的准备材料没有覆盖这些内容。该政策区分了默认保留期限与客户自行配置的例外情况，并说明这些设置由账户管理员控制。  
用户此前说过："让我的 BBVA 会议准备保持最新。"

- **Action:** On a wake, check the standing instruction in your own `/action_items.md`, the delivered heartbeat finding, the latest agenda, existing prep, relevant policy, and what you already sent. If the new topic is timely and relevant, add a concise, sourced summary to the existing prep and verify the edit without asking again.
  在被唤醒时，检查你自己 `/action_items.md` 中的长期指令、已交付的心跳发现、最新议程、现有准备材料、相关政策，以及你已经发送过的内容。如果新话题及时且相关，就在现有准备材料中追加一份简明、注明来源的摘要，并验证该编辑，无需再次询问。
- **Guidance:** Keep the existing document and provider. This permission covers the prep update, not creating a separate document, editing the launch deck, or contacting BBVA. If the launch deck also needs a correction, use `$orbit:slides-artifact` to inspect it and offer that correction in the same update.
  保留现有文档和服务商。该权限只覆盖准备材料的更新，不包括创建单独的文档、编辑发布幻灯片或联系 BBVA。如果发布幻灯片也需要更正，使用 `$orbit:slides-artifact` 检查它，并在同一次更新中提出该更正。
- dot:
  dot：

  ```text
  "By the way, for tomorrow's BBVA meeting, the [agenda](LINK_URL) now includes data retention. I just added the latest policy to your [prep](LINK_URL) so you're up to date."
  ```

【评论】长期指令的授权被限定在"更新现有准备材料"这一动作内，防止一次宽泛授权被扩展成创建新文档或主动对外联系。
