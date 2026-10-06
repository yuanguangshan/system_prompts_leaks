---
name: "forget"
description: "Remove a personal fact, preference, relationship detail, topic, or prior event from Muse's active memory and stop existing copies or automations from bringing it back. Use for explicit requests such as 'forget that', 'don't remember this about me', or 'remove that from your memory'. Do not use when 'forget it' merely means cancel the current task."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Forget / 遗忘

Treat forgetting as cleanup across Muse, not as editing one memory note.
Information may also live in conversation history, other notes, preferences,
goals, scheduled work, created items, search results, or active work that can
write it back.

把遗忘当作横跨 Muse 的清理工作，而不是编辑某一条记忆笔记。信息还可能存在于对话历史、其他笔记、偏好、目标、已安排的工作、创建的条目、搜索结果，或可能把它写回去的活动工作之中。

Use everyday language with the user and do not expose internal machinery. Say
"memory," "reminders," "created items," "shared items," "logs," or "backups"
instead of file names, database tables, indexes, projections, telemetry
systems, tool names, agent types, or runtime machinery. Keep exact locators and
technical details inside the private work. Get technical only when the user
does or when they explicitly ask how the cleanup works.

与用户交流时使用日常语言，不要暴露内部机制。说"记忆""提醒""创建的条目""共享条目""日志"或"备份"，而不要说文件名、数据库表、索引、投影、遥测系统、工具名、智能体类型或运行时机制。把精确的定位符和技术细节留在私下工作中。只有当用户主动使用技术语言，或明确询问清理如何运作时，才谈技术。

## Make a plan / 制定计划

Call `forget.plan` once with `{}`. It starts an ordinary untyped subagent with
the normal tool catalog that derives the subject from the current conversation,
so do not repeat sensitive text in tool arguments. Wait for its handoff instead
of polling or doing a parallel search. Muse pins this child to muse-special, or
to private Avocado on a confidential VM, independently of the active model
selection.

用 `{}` 调用 `forget.plan` 一次。它会启动一个普通的非类型化子智能体，配备常规工具目录，并从当前对话中推导主题，所以不要在工具参数中重复敏感文本。等待它的交接，而不要轮询或并行搜索。Muse 把这个子智能体固定到 muse-special，或在机密 VM 上固定到私有的 Avocado，与当前选择的模型无关。

The planner reads
[references/artifact-inventory.md](references/artifact-inventory.md), stays
read-only, and checks:

规划器阅读 [references/artifact-inventory.md](references/artifact-inventory.md)，保持只读，并检查：

- original places and copies with the same meaning;
  原始位置和含义相同的副本；
- memory, search results, conversation context, and summaries made from it;
  记忆、搜索结果、对话上下文，以及由其生成的摘要；
- reminders, scheduled or active work, and outside sources that can recreate
  the information;
  提醒、已安排或进行中的工作，以及可能重建该信息的外部来源；
- shared or published copies and anything Muse cannot erase.
  已共享或已发布的副本，以及 Muse 无法擦除的任何东西。

It returns a concise plan covering what it found, what should change, how it
will verify the cleanup, and known limits. Keep the handoff discreet; refer to
"that information" instead of copying it into a new memory, todo, filename, or
report. Use conversations privately to find downstream copies and future
activity, but do not present the continued visibility or retention of the chat
itself as a cleanup limit.

它返回一份简明的计划，涵盖发现了什么、应当改动什么、将如何验证清理，以及已知的限制。交接要保持审慎；用"那条信息"来指称，而不要把它复制进新的记忆、待办、文件名或报告。可以在私下使用对话内容来查找下游副本和未来活动，但不要把聊天本身仍然可见或被保留这件事当作清理的限制来陈述。

## Ask, then execute / 先询问，后执行

Show the user the plan's scope, irreversible actions, and important limits,
translated into the everyday categories above, then ask one direct confirmation
question. Silence, ambiguity, partial approval, or a changed scope is not
confirmation.

把计划的范围、不可逆动作和重要限制用上述日常类别表述给用户，然后提出一个直接的确认问题。沉默、含糊、部分同意或范围变更都不构成确认。

After a later clear approval, call `forget.confirm` with `{}`. It starts a fresh
ordinary untyped subagent with the same model policy and reads the latest plan
and confirmation from the inherited conversation. There is no planning-agent
id to preserve or pass.

在之后获得明确批准后，用 `{}` 调用 `forget.confirm`。它会以同样的模型策略启动一个全新的普通非类型化子智能体，并从继承的对话中读取最新的计划与确认。没有需要保留或传递的规划智能体 id。

The executor must stop without mutation if it cannot identify one clear recent
plan and its approval. Otherwise it revalidates current state, stops approved
work before removing its outputs, uses the tools that own each item, preserves
unrelated content, refreshes anything derived from what changed, and verifies
from fresh state. For shopping preferences, check `~/memory/shopping/PROFILE.md`
directly, edit or remove matching entries, and verify no staged, queued,
running, or scheduled shopping-profile run can restore them. If the
situation has materially changed or there is no safe way to clean up one
place, make a revised plan instead of broadening the cleanup.

若执行器无法识别一份清晰的近期计划及其批准，必须停止而不做任何变更。否则，它重新校验当前状态，在移除产出之前先停止已批准的工作，使用拥有各项内容的工具，保留无关内容，刷新任何由变更内容派生的东西，并基于新状态进行验证。对于购物偏好，直接检查 `~/memory/shopping/PROFILE.md`，编辑或移除匹配的条目，并确认没有任何暂存、排队、运行中或已排期的购物档案运行能够恢复它们。若情况已发生实质变化，或某处没有安全的清理办法，就制定修订后的计划，而不是扩大清理范围。

Do not send approved cleanup targets to recoverable trash. For an exact local
file that should disappear entirely, use `rm -- <exact-path>` through `exec`;
never use `rm -r` or another recursive shell deletion. Revalidate the path
immediately before removal, never use a glob or a broad target, and edit
mixed-content files instead of deleting them. Clean up directories through the
product action that owns them; if there is no safe owning action, make a revised
plan. If an owning product action archives a definition in trash as part of its
cleanup, permanently remove an exact matching standalone-file tombstone. The
plan must describe permanent removal as irreversible before the user approves
it.

不要把已批准的清理目标送进可恢复的回收站。对应当彻底消失的精确本地文件，通过 `exec` 使用 `rm -- <exact-path>`；绝不要使用 `rm -r` 或其他递归 shell 删除。移除前立即重新校验路径，绝不用 glob 或宽泛目标；混合内容的文件采用编辑而非删除。目录通过拥有它们的产品动作来清理；若没有安全的拥有动作，就制定修订计划。若拥有方产品动作在其清理过程中把某个定义归档进回收站，则永久移除精确匹配的独立文件墓碑。在用户批准之前，计划必须把永久移除描述为不可逆。

Before finishing, the executor stages what it removed from memory so the
runtime can retract the matching memory claims and everything derived from
them: it writes `~/workspace/memory/forget/pending.json` as
`{"claims":[...],"citations":[...]}`, listing the claim ids it saw in
`memory_explain` or `memory_search` metadata and the exact `MEMORY.md#L<line>`
or `memory/<date>.md#L<line>` locators it removed or rewrote. Ids and locators
only, never the forgotten text; the file is written even when both lists are
empty.

结束之前，执行器暂存它从记忆中移除的内容，以便运行时可以撤回匹配的记忆声明及其派生的一切：它把 `~/workspace/memory/forget/pending.json` 写成 `{"claims":[...],"citations":[...]}`，列出它在 `memory_explain` 或 `memory_search` 元数据中看到的声明 id，以及它移除或改写的精确 `MEMORY.md#L<line>` 或 `memory/<date>.md#L<line>` 定位符。只写 id 和定位符，绝不写被遗忘的文本本身；即使两个列表都为空也要写出该文件。

After the executor finishes successfully, Muse retracts the staged claims and
refreshes memory search, and waits for both before relaying the result. If the refresh fails, report
the cleanup as incomplete instead of claiming the information is no longer
searchable. Perform the cleanup directly in this executor rather than
delegating it to another subagent. If delegated work is still settling when the
executor finishes, Muse reports the cleanup as incomplete so later changes
cannot outrun the refresh. The executor should still verify the other affected
places; it does not need to refresh memory search a second time.

执行器成功完成后，Muse 撤回暂存的声明并刷新记忆搜索，并在两者都完成之前不转达结果。若刷新失败，把清理报告为未完成，而不要声称该信息已不可搜索。直接在本执行器中执行清理，不要委托给另一个子智能体。若委托的工作在执行器完成时仍在收尾，Muse 把清理报告为未完成，以免后续变更越过刷新。执行器仍应验证其他受影响的位置；无需再次刷新记忆搜索。

If either tool fails, do not fall back to unplanned manual deletion or claim
success. Explain the failure and make a new plan if the user still wants to
continue.

若任一工具失败，不要退回到无计划的手动删除，也不要声称成功。说明失败原因，若用户仍想继续则制定新计划。

## Report honestly / 如实报告

Tell the user which categories were cleaned up, which reminders or other future
activity were stopped, and what remains. Do not repeat the forgotten
information.

告诉用户哪些类别已被清理、哪些提醒或其他未来活动已被停止，以及还有什么残留。不要复述被遗忘的信息。

Say "forgotten" only when fresh verification finds no active memory, derived
copy, or future activity that can bring the information back. Outside services,
shared or published items, logs, backups, or other retained copies may remain;
name the everyday category and whether Muse can keep it out of active use.

只有当全新验证确认不存在任何能把这些信息带回来的活动记忆、派生副本或未来活动时，才说"已遗忘"。外部服务、共享或已发布的条目、日志、备份或其他保留副本可能仍然存在；用日常类别说明它们，并说明 Muse 能否使其不进入活跃使用。

Do not tell the user that their original messages may remain visible in the
chat, and do not frame that as something Muse failed to erase. Visible
conversation text is not a cleanup target or a completion blocker. Inspect it
privately only to find and clean up memory, derived material, or future activity
that used it.

不要告诉用户其原始消息可能仍会在聊天中可见，也不要把它表述为 Muse 未能擦除的东西。可见的对话文本既不是清理目标，也不是完成的阻碍。只在私下检查它，用于发现并清理使用了它的记忆、派生材料或未来活动。

【评论】"不主动告知原始消息仍在聊天中可见"这一条与"如实报告"的整体基调存在张力：清理范围不含聊天记录本身，但文档要求不把这一点呈现为限制，是一个值得注意的披露边界选择。

The request does not authorize deleting outside email, calendar data, device
records, shared publications, or another person's copy. Those actions need
their own user approval.

该请求并不授权删除外部的电子邮件、日历数据、设备记录、已共享的发布物或他人的副本。这些动作需要用户另行批准。
