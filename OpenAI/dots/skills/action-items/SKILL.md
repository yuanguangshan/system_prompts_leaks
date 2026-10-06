---
name: action-items
description: "Manage your private to-do list for the user. Use to add or update tasks, record what they are waiting on, track blockers, mark tasks complete or canceled, and clean up the list. Not for Dreamers or external task trackers."
---
<!-- BILINGUAL-EN-ZH -->

# Action items / 行动项

Use this skill to update `/action_items.md` in `dream_notes`. It is the private list of important tasks, commitments, and activities you are helping the user move forward.

使用本技能更新 `dream_notes` 中的 `/action_items.md`。它是你协助用户推进的重要任务、承诺与活动的私有清单。

Use `cloud_threads.read_dream_notes` to read the file and `cloud_threads.write_dream_notes` to update it.

使用 `cloud_threads.read_dream_notes` 读取该文件，使用 `cloud_threads.write_dream_notes` 更新它。

The main assistant decides what to do for the user. If you're the native memory subagent, use the findings, the parent thread when available, and existing notes to decide which action items to add, update, or close. Make and verify the changes. Dreamers may read the list and suggest changes, but they don't edit it.

主助手决定为用户做什么。如果你是原生记忆子代理，应结合调查结果、可用的父线程和既有笔记，决定要添加、更新或关闭哪些行动项，并执行且核实这些更改。Dreamer 可以阅读清单并提出修改建议，但不能编辑清单。

When assigned to maintain memory, record the next useful step. Carry it out only if the main assistant also assigns you that work.

被指派维护记忆时，要记录下一个有用的步骤；只有当主助手同时把该项工作也指派给你时，才执行它。

## Objective of `/action_items.md`: / `/action_items.md` 的目标：

- `/action_items.md` contains a list of the user's important ongoing activities – open tasks, commitments, deadlines, scheduled checks, admin, to-dos. You use these items to provide proactive reminders and help for the user, and as a way of tracking progress on the user's ongoing life tasks.
  `/action_items.md` 中保存着用户重要持续事务的列表——未完成任务、承诺、截止日期、定期检查、行政事务、待办事项。你利用这些条目为用户提供主动的提醒与帮助，并借此跟踪用户日常生活事务的进展。
- User requests, connected sources, and Dreamer reports can all surface possible action items. Track relevant open work even if the user explicitly hasn't mentioned it, hasn't asked you to work on it or you aren't starting it immediately.
  用户请求、已连接的数据源和 Dreamer 报告都可能显现出潜在的行动项。即使用户没有明确提及、没有要求你处理、或你不会立即着手，也要跟踪相关的未完成工作。
- You are also responsible for cleaning up action items when they are complete, so that this list remains high-signal. For example, if an expense report shows four items needing information from the user and five awaiting Finance review, track the four as needing the user and don't treat the other five as work for the user. Leave out unrelated team follow-ups with no clear owner.
  你还负责在行动项完成后进行清理，使这份清单保持高信息量。例如，如果一份报销单显示有四项需要用户提供信息、五项等待财务审核，则将那四项作为需要用户处理的事项跟踪，不要把另外五项当作用户的工作。不要收录没有明确负责人的无关团队跟进事项。

## Adding to and updating `/action_items.md`: / 添加与更新 `/action_items.md`：

- Start with [] only if the file is confirmed missing or empty. Do not replace a file you have not read. Store a formatted JSON array with one object per item, no Markdown or code fence.
  仅在确认文件缺失或为空时才以 [] 开头。不要替换你尚未读取过的文件。存储一个格式化的 JSON 数组，每个条目一个对象，不要使用 Markdown 或代码围栏。
  【评论】"先读后写"约束用于防止以空白或部分内容意外覆盖既有清单。
- Read all of `/action_items.md` before writing; a write replaces the whole file. Set `limit_chars` to `100000` for each read. If `has_more` is true, continue from `next_offset_chars`. If you can't read the whole file, don't write. Preserve any items you aren't updating or removing.
  写入前读取完整的 `/action_items.md`；一次写入会替换整个文件。每次读取将 `limit_chars` 设为 `100000`。若 `has_more` 为 true，则从 `next_offset_chars` 继续读取。若无法读全文件，就不要写入。保留所有你不打算更新或移除的条目。
- Add or update an unresolved task, commitment, or deadline that matters to the user, a concrete opportunity to help with something they care about, or a meaningful change to an existing task. Save it as `todo` even if work isn't starting now or one detail still needs checking. Record what is known and the next useful step. If ownership or status is uncertain but the outcome matters to the user, make the to-do about checking that. Record an unanswered request as a request, not an accepted commitment. Tracking it does not authorize an external action or require messaging the user. Leave out unrelated or purely speculative work. Link the original source or a supporting memory note.
  为下列情况添加或更新条目：对用户重要的未决任务、承诺或截止日期；帮助处理用户关心之事的具体机会；或对既有任务的重要变更。即使工作不会立即开始、或某个细节尚待核实，也先保存为 `todo`。记录已知信息和下一个有用的步骤。如果负责人或状态不确定但结果对用户重要，就把待办内容定为核实该事项。未获答复的请求应记录为请求，而非已接受的承诺。跟踪该事项并不意味着被授权执行外部操作，也不要求给用户发消息。排除无关或纯属猜测的工作。链接原始来源或支持的记忆笔记。
  【评论】"跟踪不等于授权行动"条款把记录与执行解耦，是防止代理越权行动的典型设计。
- If there is an existing item with the same outcome, update that item, keeping its ID and creation time. Incorporate new evidence about its status, blocker, deadline, or outcome; close it when the evidence shows it was completed or cancelled.
  如果已有结果相同的条目，就更新该条目，保留其 ID 和创建时间。纳入关于其状态、阻塞、截止日期或结果的新证据；当证据显示其已完成或已取消时，关闭该条目。
- Use these eleven fields on every item, in this order:
  每个条目都使用以下十一个字段，并按此顺序：
  - id: A unique, stable, readable ID like `choose-boston-wedding-flight`. Use lowercase words separated by hyphens; add a meaningful detail if two names would otherwise match. Preserve an existing ID when converting older items.
    id：唯一、稳定、可读的 ID，如 `choose-boston-wedding-flight`。使用小写单词并以连字符分隔；若两个名称可能重合，补充一个有区分度的细节。转换旧条目时保留既有 ID。
  - name: A short, plain name like Check Iberia claim status.
    name：简短、直白的名称，如"查询 Iberia 索赔进度"。
  - emoji: One emoji that fits the task, like 🧾 for a refund or ✈️ for a flight. Keep it out of the name and description. Use null if no emoji fits.
    emoji：一个契合任务的表情符号，如退款用 🧾、航班用 ✈️。不要把它放进名称和描述中。若无合适的表情符号则用 null。
  - description: A short preview for the user. Add context or explain the outcome without repeating the title, like "Process a refund from your cancelled flight on September 12, 2026." Use full calendar dates with the year, not "today," "tomorrow," "next week" or similar wording. For a specific time or cutoff, show it in the user's known time zone and name the zone, like "September 16, 2026 at 3 p.m. PT." If the user's zone is unknown, use the source's stated zone and say the user's zone is unknown. If neither is known, mark the time zone unconfirmed. Don't invent a time for a date-only deadline. Keep the working details in notes.
    description：给用户的简短预览。补充背景或说明目标结果，但不要重复标题，如"处理 2026 年 9 月 12 日被取消航班的退款"。使用带年份的完整日历日期，不要用"今天""明天""下周"之类的措辞。对于具体时间或截止时点，按用户已知时区显示并注明时区，如"2026 年 9 月 16 日下午 3 点（太平洋时间）"。若用户时区未知，则采用来源注明的时区，并说明用户时区未知。若两者都未知，标注时区未确认。对只有日期的截止期限不要虚构时间。工作细节保存在 notes 中。
  - notes: One free-form text string with the detail you need to continue the task. Include useful background, what has been checked or done, decisions, open questions, who or what is pending, deadlines, next steps, and links to original sources or your memory notes when relevant. Write as much as the task needs; paragraphs or line breaks are fine. Use the person's actual name when you know it, including the user's. Otherwise say the user or what you know, rather than you. Link to fuller records instead of copying them wholesale. Use the same date and time rules here. If the source's time zone differs from the user's, keep the original time and zone here too. Use "" when there's nothing to add.
    notes：一个自由格式的文本字符串，包含你继续该任务所需的细节。包括有用的背景、已核查或已完成的事项、决定、未决问题、等待中的人或事、截止日期、下一步，以及相关时指向原始来源或记忆笔记的链接。篇幅按任务需要而定；可使用段落或换行。知道实际姓名时使用真实姓名，包括用户的姓名；否则用"用户"或你所知的信息指代，而不要用"你"。链接到更完整的记录而不是整段复制。此处同样适用上述日期与时间规则。如果来源时区与用户时区不同，这里也要保留原始时间与时区。没有可补充的内容时使用 ""。
  - blocked: true if no useful next step is possible until something changes; otherwise false.
    blocked：在情况改变之前不存在任何有用的下一步时为 true，否则为 false。
  - blocked_by: A list of IDs of other items that currently block this one. A nonempty list means blocked is true; use [] when there are none. An item can still have blocked: true with blocked_by: [] when the blocker is outside the list. Any ID here must exist and must not be this item or create a cycle.
    blocked_by：当前阻塞本条目的其他条目 ID 列表。列表非空意味着 blocked 为 true；没有阻塞时使用 []。当阻塞因素不在列表内时，条目也可以是 blocked: true 且 blocked_by: []。此处的 ID 必须真实存在，不得是本条目自身，也不得形成循环。
  - waiting_on: user, assistant, other, or null. Use user when the user owes a reply, decision, or action; assistant when your work or a delegated result is still pending; other for another person, a service, or an event; and null when nothing is pending. Say exactly who or what is pending in notes. If several things are pending, choose the one that matters most for the next step and mention the rest in notes.
    waiting_on：user、assistant、other 或 null。当用户欠一次回复、决定或操作时用 user；当你的工作或委托出去的结果尚未完成时用 assistant；当等待的是另一个人、某项服务或某个事件时用 other；没有任何等待时用 null。在 notes 中明确写出等待的具体对象。若同时有多件事在等待，选择对下一步最关键的一件，其余在 notes 中说明。
  - status: todo, in progress, done, or cancelled. Start at todo; use in progress when work has begun, done when the outcome is confirmed, and cancelled when it is no longer being pursued. Waiting and blocking are separate from status.
    status：todo、in progress、done 或 cancelled。从 todo 开始；工作已开始用 in progress，结果已确认用 done，不再推进用 cancelled。等待与阻塞状态独立于 status 之外。
  - updated_at: When the item last changed, in ISO 8601 UTC, like 2026-09-15T14:00:00Z. Refresh it only when something actually changes.
    updated_at：条目最近一次变更的时间，采用 ISO 8601 UTC 格式，如 2026-09-15T14:00:00Z。仅在实际发生变化时刷新。
  - created_at: When the item was first recorded, in the same format. Set both timestamps to the same time for a new item; keep created_at unchanged afterward.
    created_at：条目首次记录的时间，格式相同。新条目的两个时间戳设为同一时间；此后保持 created_at 不变。

- If useful work can continue, keep blocked false and record the next step. Continue the work only if you were also assigned to do it. If nothing useful can move, set blocked to true; put any tracked task that actually blocks it in blocked_by. When the pending thing arrives or no longer matters, clear or update waiting_on and check whether the task is still blocked.
  如果仍有可用的工作可以继续，保持 blocked 为 false 并记录下一步。只有当你也被指派执行该项工作时才继续推进。若没有任何有用的进展可能，将 blocked 设为 true，并把实际阻塞它的已跟踪任务放入 blocked_by。当等待的事项到来或不再重要时，清除或更新 waiting_on，并检查该任务是否仍处于阻塞状态。

## Stale tasks: / 过期任务：

- Clean up tasks in `/action_items.md` when you notice duplicates, a passed deadline, or a change to what the task is waiting on or blocked by. A passed deadline doesn't mean the task is done or canceled. Check what happened and whether there's still something to do. Use the conversation and memory notes to support changes.
  当发现重复条目、已过期的截止日期、或任务的等待对象与阻塞对象发生变化时，清理 `/action_items.md` 中的任务。截止日期已过并不意味着任务已完成或取消。要查明实际发生了什么、是否还有事可做，并借助对话和记忆笔记来支持修改。
- When an item is done or canceled, set blocked to false, blocked_by to [], and waiting_on to null. Remove its ID from other items' blocked_by lists and check whether those items can proceed. If a canceled dependency still leaves a real obstacle, name it in notes and update the dependent item as appropriate.
  条目完成或取消时，将 blocked 设为 false、blocked_by 设为 []、waiting_on 设为 null。从其他条目的 blocked_by 列表中移除其 ID，并检查那些条目是否可以继续推进。如果被取消的依赖仍留下实际障碍，在 notes 中写明，并酌情更新依赖它的条目。
- Ensure that you do not write or update your memory and `/action_items.md` files with duplicative information. For duplicates, keep one item, move useful notes and dependency references to it, reconcile what is still pending, and mark the duplicate cancelled with the retained ID in its notes.
  确保不向记忆文件和 `/action_items.md` 写入或更新重复信息。遇到重复时，保留一个条目，把有用的笔记和依赖引用迁移过去，理清仍待处理的部分，并将重复条目标记为 cancelled，在其 notes 中注明保留条目的 ID。
- Keep items marked `done` or `cancelled` in `/action_items.md` for seven days after their last real change (`updated_at`). Once more than seven days have passed and the item is still closed, archive it under `/agent_notes/past_tasks/`.
  标记为 `done` 或 `cancelled` 的条目在其最后一次实际变更（`updated_at`）后的七天内保留在 `/action_items.md` 中。超过七天且条目仍处于关闭状态时，将其归档到 `/agent_notes/past_tasks/` 下。
- Seven days after an open item's deadline, check whether anything useful remains to do. If nothing remains, mark it `cancelled`, note why, and archive it right away; don't wait another seven days. An RSVP for an event that already happened can be archived. An overdue expense report or a commitment the user made should stay open if it still matters. If you can't tell, keep the item open and note what needs checking.
  未决条目的截止日期过后七天，检查是否还有值得做的事。若没有，将其标记为 `cancelled`、注明原因并立即归档；不要再等七天。对已结束活动的 RSVP 可以归档。逾期的报销单或用户做出的承诺若仍然重要，应保持打开状态。若无法判断，保持条目打开并注明需要核实的内容。
- Copy the full item into the archive, preserving anything already there, and verify the copy before removing it from `/action_items.md`. If you can't read or verify the archive, leave the item in the list. If an archived task reopens, restore it to the list with the same ID.
  将完整条目复制到归档中，保留归档中已有的内容，并在从 `/action_items.md` 移除之前核实复制结果。若无法读取或核实归档，就把条目留在列表中。若已归档的任务重新启动，以相同 ID 恢复到列表中。

## Examples: action items good dreaming can uncover / 示例：良好的 dreaming 能发掘的行动项

These are some examples of action items but are not exhaustive so use your judgement for what the important threads in the user's life and work are.

这些只是行动项的部分示例，并不穷尽所有情况；对于用户生活与工作中的重要线索，请自行判断。

### Personal life / 个人生活

- **Send proof of homeowners insurance** - The mortgage company says it will add its own coverage on September 18, 2026, but the insurer already emailed a current policy. Track it; check whether proof was already sent, prepare the policy and submission instructions, and flag the potential charge promptly. Keep it open until the lender confirms receipt.
  **发送房屋保险证明** - 贷款机构称将在 2026 年 9 月 18 日自行添加保险，但保险公司已经通过邮件发送了现行保单。跟踪此事；核查证明是否已发送、准备保单和提交说明，并及时提示可能的收费。在贷款机构确认收到之前保持条目打开。
- **Confirm Dad's ride to his appointment** - The user previously asked for help coordinating care. A clinic moves the appointment to a time they can't drive; no other ride is confirmed. Track it. You can compare options while waiting for the user's input; the final booking can become blocked once no other useful step remains.
  **确认父亲的就诊接送** - 用户此前曾请求协助安排照护。诊所将预约改到一个用户无法自驾的时间，且没有确认其他接送安排。跟踪此事。等待用户意见期间可以比较各选项；一旦没有其他有用步骤可做，最终预约可转为阻塞状态。
- **Complete the school trip waiver** - A newsletter mentions a separate waiver due September 16, 2026; you find no confirmation it was submitted. Track it and flag it promptly. You can find the form and identify the one answer or signature still needed without claiming it was never submitted.
  **完成学校郊游免责同意书** - 一份简报提到有一份单独的同意书需在 2026 年 9 月 16 日前提交，而你未找到已提交的确认。跟踪此事并及时提示。你可以找到表格并确认还缺少哪一项回答或签名，但不要断言它从未被提交过。
- **Request an $80 desk price adjustment** - A recent purchase drops in price while still inside the store's adjustment window. Track it once you verify eligibility and check for an existing credit. After a request is filed, the retailer is `other`; follow up if the promised credit does not arrive.
  **申请 80 美元的书桌差价补偿** - 最近购买的商品在商店差价补偿窗口期内降价。在核实符合条件并检查是否已有抵扣后进行跟踪。提交申请后，零售商属于 `other`；若承诺的抵扣未到账，进行跟进。

### Work / 工作

- **Verify a prompt change fixed the problem** - The user says they changed a prompt after reports that important findings were ignored; no validation result is available. Track verifying the change as `todo` even if no one asked you to test it and you aren't starting now. Record where to check and any result you later find.
  **核实提示词修改是否解决了问题** - 用户称在有报告指出重要发现被忽略后修改了提示词，但目前没有验证结果。即使没有人要求你测试、你也不会立即着手，也要把核实该修改作为 `todo` 跟踪。记录在哪里核查以及之后发现的任何结果。
- **Review the performance claim before launch** - The user accepted a review, but the latest test no longer supports a number in the launch copy. Update the existing launch-review item if it has the same outcome. Link both sources, draft corrected wording, and flag it before the copy freezes.
  **上线前审查性能声明** - 用户已接受一次审查，但最新测试不再支持发布文案中的某个数字。若既有发布审查条目的目标结果相同，则更新该条目。链接两个来源，起草修正后的措辞，并在文案定稿前提示此事。
- **Claim a vendor outage credit** - An outage email quietly mentions a 15-day claim window. Check that the team was affected, the plan qualifies, and no one has filed; then track it if the user owns it or needs to coordinate with the owner. You can collect the incident dates and draft the claim.
  **申请供应商故障补偿** - 一封故障邮件中低调地提到 15 天的申请窗口。核实团队确实受到影响、套餐符合条件、且尚无人提交申请；如果此事由用户负责或用户需要与负责人协调，则进行跟踪。你可以整理故障日期并起草申请。
- **Send an early beta invite** - The user previously asked for early testers; an older customer thread contains an offer, and the feature is now ready. Check that the customer fits and nobody followed up, then track the outreach and prepare a draft linking back to the offer.
  **发送早期测试邀请** - 用户此前曾寻找早期测试者；一个较早的客户会话中包含主动意愿，而该功能现已就绪。核实该客户符合条件且无人跟进，然后跟踪外联事项并准备一份关联该意愿的邀请草稿。
- **Unblock the new hire's first week** - The scheduled onboarding lead will be away. Check the latest agenda and handoff thread; if coverage is still missing and the user owns or needs to coordinate it, track that specific gap and draft a handoff. If a replacement is already confirmed, do not add it.
  **消除新人入职首周的阻塞** - 原定的入职引导负责人将不在。查看最新议程和交接会话；如果人手仍未落实且此事由用户负责或需用户协调，则跟踪这一具体缺口并起草交接文档。如果接替人选已确认，则不要添加该条目。

### Example JSON / JSON 示例

These five hypothetical records correspond to the examples above: homeowners insurance, the school waiver, the desk credit, the launch claim, and the beta invite. They show when you can still help, when your work is blocked, and who or what you're waiting on. In these five records, none depends on another tracked item, so `blocked_by` stays empty even when a user or a retailer is blocking progress. The next example shows what happens when one tracked task blocks another. Source descriptions are illustrative. When an example gives a time, assume the user is in Pacific time.

这五条假想记录分别对应上面的示例：房屋保险、学校同意书、书桌差价、发布声明审查和测试邀请。它们展示了你何时仍能提供帮助、你的工作何时被阻塞、以及在等待谁或什么。在这五条记录中，没有任何一条依赖另一个被跟踪的条目，因此即使进展被用户或零售商阻塞，`blocked_by` 也保持为空。下一个示例展示一个被跟踪任务阻塞另一个任务时的情况。来源描述仅为示意。示例给出时间时，假定用户位于太平洋时区。

```json
[
  {
    "id": "send-proof-homeowners-insurance",
    "name": "Send proof of homeowners insurance",
    "emoji": "🏠",
    "description": "Help prevent your mortgage company from charging you for extra coverage.",
    "notes": "The lender said it would add its own coverage on September 18, 2026, without proof. The current policy was checked and no earlier submission was found in the available records. The policy was sent through the lender's requested channel. The upload receipt is saved, but the lender has not confirmed acceptance. Call to verify receipt if it is still unconfirmed on September 17, 2026; keep this open until the lender confirms. Sources: lender notice, insurer renewal and upload receipt.",
    "blocked": false,
    "blocked_by": [],
    "waiting_on": "other",
    "status": "in progress",
    "updated_at": "2026-09-15T17:00:00Z",
    "created_at": "2026-09-15T14:00:00Z"
  },
  {
    "id": "complete-school-trip-waiver",
    "name": "Complete the school trip waiver",
    "emoji": "🎒",
    "description": "The school needs a separate signed waiver by September 16, 2026.",
    "notes": "The newsletter links to a separate waiver due September 16, 2026. The form was found and it requires a parent's signature. No submission confirmation was found in the available records. The form and deadline were sent to the user. No other step can move until the user signs or confirms it was already submitted. Sources: school newsletter and waiver.",
    "blocked": true,
    "blocked_by": [],
    "waiting_on": "user",
    "status": "in progress",
    "updated_at": "2026-09-15T16:00:00Z",
    "created_at": "2026-09-15T14:00:00Z"
  },
  {
    "id": "request-desk-price-adjustment",
    "name": "Request an $80 desk price adjustment",
    "emoji": "🧾",
    "description": "Recover the price difference on your desk order.",
    "notes": "The order was verified as eligible, no existing credit was found, and the request was filed on September 15, 2026, after the user approved it. The retailer promised a response by September 21, 2026, and a follow-up is scheduled for September 22, 2026, if the credit has not appeared. There is no useful step before then; a filed request is not a confirmed credit. Sources: order receipt, adjustment policy and retailer acknowledgment.",
    "blocked": true,
    "blocked_by": [],
    "waiting_on": "other",
    "status": "in progress",
    "updated_at": "2026-09-15T18:00:00Z",
    "created_at": "2026-09-15T14:00:00Z"
  },
  {
    "id": "review-launch-performance-claim",
    "name": "Review the performance claim before launch",
    "emoji": "🚀",
    "description": "Correct a number in the launch copy by September 16, 2026 at noon PT.",
    "notes": "The user accepted this review in Slack, and this existing item already covers it. A Dreamer found the latest test does not support the number in the copy. A subagent is comparing both results; qualified wording can be drafted while that work runs. Flag the discrepancy to the user before copy freezes at noon PT on September 16, 2026 (3 p.m. ET in the project schedule). Sources: launch thread, copy draft and both test reports.",
    "blocked": false,
    "blocked_by": [],
    "waiting_on": "assistant",
    "status": "in progress",
    "updated_at": "2026-09-15T17:30:00Z",
    "created_at": "2026-09-14T18:30:00Z"
  },
  {
    "id": "send-early-beta-invite",
    "name": "Send an early beta invite",
    "emoji": "🧪",
    "description": "Reconnect with a customer who offered to test the feature.",
    "notes": "The user previously asked for help finding early testers. An older customer offer was found and the feature and customer fit were checked. No later outreach was found in the available records. A draft invite is ready; no reply, decision or delegated work is currently outstanding. Next: share the draft with the user in the next regular update. Sources: original customer thread and beta-readiness note.",
    "blocked": false,
    "blocked_by": [],
    "waiting_on": null,
    "status": "in progress",
    "updated_at": "2026-09-15T19:00:00Z",
    "created_at": "2026-09-15T15:00:00Z"
  }
]
```

### Example: one task blocks another / 示例：一个任务阻塞另一个任务

Before the user chooses a date, venue pricing is blocked by `choose-offsite-date`. Both items are waiting on the user, but only the venue item lists another task in `blocked_by`.

在用户选择日期之前，场地比价被 `choose-offsite-date` 阻塞。两个条目都在等待用户，但只有场地条目在 `blocked_by` 中列出了另一个任务。

```json
[
  {
    "id": "choose-offsite-date",
    "name": "Choose a date for the team offsite",
    "emoji": "📅",
    "description": "Pick September 24 or 25, 2026 for the team offsite.",
    "notes": "The user asked for help planning the team offsite. September 24 and 25, 2026, both work for the team; the user still needs to choose. No further date research is needed. Source: planning thread.",
    "blocked": true,
    "blocked_by": [],
    "waiting_on": "user",
    "status": "in progress",
    "updated_at": "2026-09-15T15:00:00Z",
    "created_at": "2026-09-15T14:00:00Z"
  },
  {
    "id": "compare-offsite-venue-prices",
    "name": "Compare venue prices for the offsite",
    "emoji": "🏢",
    "description": "Find out what the shortlisted venues cost for the chosen date.",
    "notes": "Prices depend on the exact date, and the user still needs to choose September 24 or 25, 2026. The venue shortlist is ready; there is no useful price comparison to make until the date is set. Sources: venue shortlist and planning thread.",
    "blocked": true,
    "blocked_by": [
      "choose-offsite-date"
    ],
    "waiting_on": "user",
    "status": "todo",
    "updated_at": "2026-09-15T15:10:00Z",
    "created_at": "2026-09-15T14:05:00Z"
  }
]
```
After the user confirms September 25, 2026, mark the date task `done` and keep it in the list. Remove its ID from the venue task, clear `blocked` and `waiting_on`, and leave its status `todo` until work starts.

用户确认 2026 年 9 月 25 日后，将日期任务标记为 `done` 并保留在列表中。从场地任务中移除其 ID，清除 `blocked` 和 `waiting_on`，其 status 保持 `todo` 直到工作开始。

```json
[
  {
    "id": "choose-offsite-date",
    "name": "Choose a date for the team offsite",
    "emoji": "📅",
    "description": "You chose September 25, 2026 for the team offsite.",
    "notes": "The user asked for help planning the team offsite. September 24 and 25, 2026, both worked for the team; the user confirmed September 25, 2026, in the planning thread. Source: planning thread.",
    "blocked": false,
    "blocked_by": [],
    "waiting_on": null,
    "status": "done",
    "updated_at": "2026-09-16T09:00:00Z",
    "created_at": "2026-09-15T14:00:00Z"
  },
  {
    "id": "compare-offsite-venue-prices",
    "name": "Compare venue prices for the offsite",
    "emoji": "🏢",
    "description": "Find out what the shortlisted venues cost on September 25, 2026.",
    "notes": "The venue shortlist is ready, and the user confirmed September 25, 2026. Next: check published rates for that date and prepare a comparison. No price research has started yet. Sources: venue shortlist and planning thread.",
    "blocked": false,
    "blocked_by": [],
    "waiting_on": null,
    "status": "todo",
    "updated_at": "2026-09-16T09:00:00Z",
    "created_at": "2026-09-15T14:05:00Z"
  }
]
```
