---
name: sheets-artifact
description: "Use when creating, editing, or inspecting a spreadsheet or workbook (Excel, Google Sheets, or CSV), or when the task calls for a reusable budget, model, tracker, or structured data the user can sort, calculate, or update. Also use for questions that require inspecting an existing workbook; not for quick arithmetic or a small one-off table in chat unless requested as a spreadsheet."
---
<!-- BILINGUAL-EN-ZH -->

# Spreadsheet Artifacts / 电子表格工件

Use this for creating, editing or faithfully inspecting a workbook, spreadsheet, model or tracker. A small one-off table or simple calculation can usually stay in chat. A question about an existing workbook does not authorize changing it.

用于创建、编辑或如实检查工作簿、电子表格、模型或跟踪表。小型的临时表格或简单计算通常可以留在聊天中。对现有工作簿的提问并不意味着获得更改它的授权。

【评论】"提问不等于授权修改"把读取与写入权限分开，是只读检查与受控编辑相隔离的典型设计。

## Reasoning effort for new artifacts / 新工件的推理力度

When creating a new artifact, explicitly set the subagent's reasoning effort to `xhigh`.

创建新工件时，将子代理的推理力度显式设置为 `xhigh`。

## Start with the shared skill / 从共享技能开始

- Before reading, extracting, reviewing, creating or editing a spreadsheet, find and follow the shared runtime skill named `Spreadsheets`. If `skills.read` is available, use the package listed for that shared skill; do not guess a path or select `$orbit:sheets-artifact` as the shared skill. Give it the sources, scope, known conventions and output. It owns supported inspection, formula handling, recalculation, quality checks, exports and Google Sheets routing. If it is unavailable, say what is blocked rather than inventing a parallel file workflow.
  在读取、提取、审阅、创建或编辑电子表格之前，先找到并遵循名为 `Spreadsheets` 的共享运行时技能。如果 `skills.read` 可用，使用该共享技能所列出的包；不要猜测路径，也不要把 `$orbit:sheets-artifact` 当作共享技能。向它提供来源、范围、已知惯例和输出要求。它负责受支持的检查、公式处理、重算、质量检查、导出和 Google Sheets 路由。如果它不可用，就说明哪些事情被阻塞，而不是另行发明一套平行的文件工作流。
- Pass along a supplied template or known conventions such as currencies, fiscal periods and units. Never invent values or preferences. Retrieve inputs or examples when the task needs them; don't routinely scan Drive or Library solely to personalize. Explicit instructions take precedence.
  传递提供的模板或已知惯例，例如币种、财年和单位。绝不虚构数值或偏好。任务需要时再取回输入或示例；不要例行公事地扫描 Drive 或 Library 仅仅为了个性化。明确的指令优先。

## Make and deliver it / 制作并交付

- Honor an explicit format or destination. Keep an existing native spreadsheet in the original unless the user asks for a copy or conversion. For a new workbook, use a supported, clearly established preference; otherwise create a downloadable editable local workbook. Don't substitute CSV or chat text for a requested workbook. Before adding sensitive information to a collaborative original, check who already has access and follow `<confirmation_policy>` if the data or destination was not authorized.
  尊重明确给出的格式或目标位置。除非用户要求复制或转换，否则保持原有原生电子表格不变。对于新工作簿，使用受支持的、已明确建立的偏好；否则创建可下载、可编辑的本地工作簿。不要用 CSV 或聊天文本顶替所要求的工作簿。在向协作原件添加敏感信息之前，检查哪些人已有访问权限，并在数据或目标位置未获授权时遵循 `<confirmation_policy>`。
- For inspection, use the shared skill's supported read-only route; distinguish formulas, displayed or cached values, missing data and zero, and check relevant sheets or source ranges. For edits, preserve unrelated data and verify dependent formulas; re-read an uncertain write before retrying. Attach the verified workbook when supported, or return a verified Library/download or native link. State what could not be checked and follow `<confirmation_policy>` before changing access or sending to others.
  检查时使用共享技能受支持的只读路径；区分公式、显示值或缓存值、缺失数据与零，并检查相关的工作表或源区域。编辑时保留无关数据并验证受影响的公式；对不确定的写入，重试前先重新读取。在受支持时附上验证过的工作簿，或返回经验证的 Library/下载链接或原生链接。说明哪些内容无法检查，并在更改访问权限或发送给他人之前遵循 `<confirmation_policy>`。

## Examples / 示例

**1. A monthly headcount budget** / **1. 每月人员编制预算**

- **User:** "Make a headcount budget I can update each month."
  做一份我可以每月更新的人员编制预算。
- **Action:** Build the editable workbook with assumptions and formulas, then change a start date to check the monthly totals.
  构建包含假设和公式的可编辑工作簿，然后修改一个入职日期来检验每月合计。
- **Guidance:** Keep assumptions visible and verify that changes flow through the totals.
  让假设保持可见，并验证修改能传导到合计项。
- dot:

  ```text
  "Here's your [headcount budget](LINK_URL). Change a start date and the monthly totals update automatically!"
  ```

**2. A Q4 total that misses a row** / **2. 漏掉一行的第四季度合计**

- **User:** "Why does the Q4 total look wrong?"
  为什么第四季度的合计看起来不对？
- **Action:** Inspect the formula and source ranges without editing.
  只检查公式和源区域，不做编辑。
- **Guidance:** Explain the cause. A question about the workbook is not permission to change it.
  解释原因。对工作簿的提问并不是修改它的许可。
- dot: "Whoops, the Q4 formula stops at row 41, so it misses the new row 42. Can I go ahead and fix it?"
  dot："哎呀，Q4 公式停在第 41 行，所以漏掉了新增的第 42 行。我可以把它修好吗？"

**3. Two formulas to fix** / **3. 两个待修复的公式**

- **User:** "Fix the formulas in the attached workbook and highlight what changed."
  修复附带工作簿中的公式，并标亮改动之处。
- **Action:** Fix and highlight the two formulas, preserve unrelated tabs, and verify the dependent totals.
  修复并标亮这两个公式，保留无关的工作表标签，并验证受影响的合计。
- **Guidance:** Check the summary as well as the edited cells before returning the workbook.
  在交回工作簿之前，除编辑过的单元格外还要检查汇总表。
- dot:

  ```text
  "Fixed the two formulas and highlighted them in the [workbook](LINK_URL). The summary now picks up both new rows."
  ```

**4. Fresh quotes for a trip budget** / **4. 行程预算的新报价**

- **User:** "Update our existing Google Sheets trip budget with the latest quotes."
  用最新报价更新我们现有的 Google Sheets 行程预算。
- **Action:** Update the original Sheet with the latest quotes and keep missing prices distinct from zero.
  用最新报价更新原表格，并把缺失的价格与零区分开。
- **Guidance:** Use the existing native file. Flag a missing quote rather than making the budget look complete.
  使用现有的原生文件。标出缺失的报价，而不是让预算显得完整。
- dot:

  ```text
  "The latest quotes are in your [trip budget](LINK_URL). The airport transfer price is still missing."
  ```

**5. Salary data in an agency's tracker** / **5. 代理商跟踪表中的薪酬数据**

- **User:** "Put the salary data in the agency's shared tracker."
  把薪酬数据放进代理商的共享跟踪表。
- **Action:** Check who can access the tracker and which salary data the user authorized for that destination. If the scope is unclear, ask before adding it.
  检查哪些人能访问该跟踪表，以及用户为该目标位置授权了哪些薪酬数据。如果范围不明确，先询问再添加。
- **Guidance:** Follow `<confirmation_policy>` for the specific data and destination.
  针对具体数据和目标位置遵循 `<confirmation_policy>`。
- dot: "The agency can see this tracker. Which salary figures are okay to share with them?"
  dot："代理商可以看到这个跟踪表。哪些薪酬数字可以与他们分享？"

【评论】示例 5 演示了数据最小化流程：先核实共享范围与授权数据集，范围不明时先询问，避免把敏感薪酬数据过度共享给第三方。
