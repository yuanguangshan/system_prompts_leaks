<!-- BILINGUAL-EN-ZH -->
# Forget artifact inventory / 遗忘工件清单

Use this as a required routing checklist, not as permission to remove every
listed surface. Record only surfaces that contain the requested subject or can
recreate it. For every match, identify its owner, exact locator, relationship
to the source, proposed action, precondition, reversibility, and verification.

将本清单作为必需的路由核查表使用，而不是把它当作删除所有列出表面的许可。只记录包含被请求主题、或能重新生成该主题的表面。对每一处匹配，都要查明其所有者、精确定位符、与源的关系、拟议操作、前置条件、可逆性和验证方式。

## 1. Conversation and live execution / 对话与实时执行

Check the current and prior public or side-chat events, context-only messages,
compaction summaries, subagent histories, tool calls and outputs, spilled tool
output files, restart checkpoints, and conversation-scoped todo snapshots.
Check conversation titles, list previews, reply previews, activity cards, and
other transcript-derived presentation projections separately from the event
rows that produced them.
Include message attachments, reactions, native-channel copies, approval
requests or decisions, and any pending handoff that can reinsert the content.
Inspect active subagents, browser tasks, workflows, shell processes, calls, and
other RuntimeWork that may still hold or act on the information.

检查当前与先前的公开或侧聊事件、仅上下文消息、压缩摘要、子代理历史、工具调用与输出、外溢的工具输出文件、重启检查点以及会话作用域的 todo 快照。对会话标题、列表预览、回复预览、活动卡片以及其他由转录派生的展示投影，要与其产生自的事件行分开检查。包括消息附件、表情回应、原生频道副本、批准请求或决定，以及任何可能重新插入该内容的待处理交接。检查活动中的子代理、浏览器任务、工作流、shell 进程、通话以及其他可能仍持有或正在处理该信息的 RuntimeWork。

Use conversation history privately to locate memory, summaries, pending work,
and other downstream material. The visible chat itself is not a cleanup target
or a completion blocker. Do not report its continued visibility or retention as
a limitation. Prevent new compaction summaries, memory flushes, checkpoints,
and future producers from restating the subject.

可私下使用会话历史来定位记忆、摘要、待办工作及其他下游材料。可见的聊天本身既不是清理目标，也不是完成阻碍。不要把它的持续可见或保留报告为一项限制。要防止新的压缩摘要、记忆落盘、检查点以及未来的生产者再次复述该主题。

## 2. Source files and standing context / 源文件与常设上下文

Inspect precise matches and semantic restatements in:

检查以下位置中的精确匹配和语义复述：

- `~/MEMORY.md` and relevant files under `~/memory/`, including dated source logs;
  `~/MEMORY.md` 及 `~/memory/` 下的相关文件，包括带日期的源日志；
- `~/USER.md`;
  `~/USER.md`；
- `~/SOUL.md`, `~/IDENTITY.md`, and legacy `~/GOALS.md` when the information
  became persona, identity, or carried goal context;
  `~/SOUL.md`、`~/IDENTITY.md` 以及旧版 `~/GOALS.md`（当该信息已成为人格、身份或所承载的目标上下文时）；
- `~/FEEDBACK.md`, the durable preference input to conversational follow-ups,
  when the information became a check-in topic, timing or frequency preference,
  or signal about the user's tolerance for proactive contact;
  `~/FEEDBACK.md`（会话跟进的持久化偏好输入），当该信息已成为问候话题、时机或频率偏好，或关于用户对主动联系容忍度的信号时；
- person and group pages plus their indexes under `~/memory/people/` and  
  `~/memory/groups/`;
  `~/memory/people/` 与 `~/memory/groups/` 下的人物和群组页面及其索引；
- the shopping profile at `~/memory/shopping/PROFILE.md`, which is
  user-editable, excluded from memory ingest, and read directly by shopping  
  turns;
  位于 `~/memory/shopping/PROFILE.md` 的购物档案，它可由用户编辑、被排除在记忆摄取之外，并由购物回合直接读取；
- `~/HEARTBEAT.md`, `~/AGENTS.md`, and `~/TOOLS.md` when the information became
  an instruction, standing check, or environment note;
  `~/HEARTBEAT.md`、`~/AGENTS.md` 和 `~/TOOLS.md`（当该信息已成为指令、常设检查或环境说明时）；
- goal, study, dream, feedback, workspace, and user-authored skill files;
  目标、学习、梦境、反馈、工作区以及用户自编的技能文件；
- attachments, generated media, and tool-output spill files under the
  workspace.
  工作区下的附件、生成的媒体和工具输出外溢文件。

Also inspect recoverable trash, temporary or editor-backup files, exports and
download archives, and version-control commits, stashes, or remote branches when
the workspace is a repository. Removing the working copy does not erase those
histories.

还要检查可恢复的回收站、临时文件或编辑器备份、导出与下载归档，以及在工作区是版本库时的版本控制提交、储藏（stash）或远程分支。删除工作副本并不会抹除这些历史。

Do not use recoverable trash for approved cleanup. Permanently remove an exact
standalone file with `rm -- <exact-path>` after revalidating the target. Never
use `rm -r`, another recursive shell deletion, globs, or home/root paths. Edit
mixed-content files in place and use the owning product action for directory
cleanup. Permanently remove an exact matching standalone-file tombstone already
in trash, including one an owning product delete action creates while cancelling
a schedule or workflow. If only a directory archive remains and no owning
permanent-delete action exists, report that limitation instead of deleting it
recursively.

不得将可恢复的回收站用于已批准的清理。在重新校验目标之后，使用 `rm -- <exact-path>` 永久删除确切的独立文件。绝不使用 `rm -r`、其他递归 shell 删除、通配符或主目录/根目录路径。就地编辑混合内容文件，并使用所属产品的操作进行目录清理。永久删除回收站中已存在的确切匹配的独立文件墓碑（tombstone），包括所属产品的删除操作在取消某个日程或工作流时创建的那一类。如果只剩一个目录归档、且不存在所属的永久删除操作，应报告该限制，而不是递归删除它。

【评论】禁用递归删除、通配符和根路径，是把"遗忘"操作的破坏半径限制在单个已校验文件上的典型防御性设计。

Preserve unrelated content in a mixed file. Daily logs are historical sources,
not disposable scratch space: removing or redacting one claim must also cause
their search rows and every downstream projection to be reconciled.

保留混合文件中的无关内容。每日日志是历史来源，不是用完即弃的草稿区：删除或涂销一条断言时，还必须使其搜索行和每一个下游投影得到同步调和。

## 3. Memory retrieval and derived projections / 记忆检索与派生投影

The Markdown corpus is indexed into PostgreSQL `memory.entries`,
`runtime.search_documents`, and `memory.embeddings`. Beside the index sits the
claim spine, `memory.claims`: one row per verified claim carrying its id, the
quote and conversation handles that grounded it, its status, and the
supersession links between an older belief and the claim that replaced it.
`memory_explain` shows the row behind a claim id, a `memory://` uri, or a
`path#L<line>` citation, and `memory_search` results carry the claim id in
their metadata. Retracting a claim must reach every claim linked to it on
that chain, so record the ids and exact line locators of what you remove;
the executor stages them in `~/workspace/memory/forget/pending.json` and the
runtime retracts the chain before the reindex. Derived memory also  
includes:

Markdown 语料被索引到 PostgreSQL 的 `memory.entries`、`runtime.search_documents` 和 `memory.embeddings` 中。索引旁边是断言主干（claim spine）`memory.claims`：每个经验证的断言一行，携带其 id、为其提供依据的引文和会话句柄、其状态，以及旧信念与取代它的断言之间的更替链接。`memory_explain` 可以显示某个断言 id、`memory://` uri 或 `path#L<line>` 引用背后的行，`memory_search` 的结果在其元数据中携带断言 id。撤回一条断言必须抵达该链条上与之链接的每一条断言，因此要记录你将删除内容的 id 和精确行定位符；执行器会把它们暂存在 `~/workspace/memory/forget/pending.json`，运行时在重建索引之前撤回整条链。派生记忆还包括：

- `~/memory/bank/{world,experience,opinions}.md`;
  `~/memory/bank/{world,experience,opinions}.md`；
- person and group pages and indexes;
  人物和群组页面及其索引；
- the personalization projection;
  个性化投影；
- the compact `MEMORY.md` and profile projection in `USER.md`;
  精简的 `MEMORY.md` 和 `USER.md` 中的档案投影；
- prompt-context caches and already assembled session context;
  提示词上下文缓存和已组装的会话上下文；
- the centrally published user-memory embedding for the VM.
  为 VM 集中发布的用户记忆嵌入。

Deleting text without reconciling sparse search, vector search, prompt caches,
and the central publication leaves active copies. Prefer the owning memory
rebuild/publication APIs. A successful `forget.confirm` executor is followed by
a runtime-owned memory reindex before its result is relayed; do not start a
second reindex. During execution, verify source files and owning-API state; a
semantic-search hit may still come from the pre-cleanup index and is not by
itself a cleanup failure. The runtime-owned reindex closes that retrieval
surface before the result is relayed. If a remote publication has no delete or
replacement proof, report that limitation.

只删除文本而不调和稀疏搜索、向量搜索、提示词缓存和集中发布，会留下仍然活跃的副本。应优先使用所属的记忆重建/发布 API。`forget.confirm` 执行器成功之后，会由运行时完成一次记忆重建索引，然后才转发其结果；不要启动第二次重建索引。在执行期间，要验证源文件和所属 API 的状态；语义搜索命中可能仍来自清理前的索引，其本身并不构成清理失败。运行时负责的重建索引会在结果转发之前关闭该检索表面。如果某个远程发布没有删除或替换的证明，应报告该限制。

## 4. Self-improvement outputs / 自我改进输出

Review both finished and in-flight runs whose evidence window included the  
subject:

审查证据窗口覆盖了该主题的所有已完成和进行中的运行：

- Memory claims, reconciliation staging, demotions, receipts, run/step state,
  and files under `~/workspace/memory/`;
  记忆断言、调和暂存、降级、回执、运行/步骤状态，以及 `~/workspace/memory/` 下的文件；
- Relationships pages, rankings, and relationship run records;
  关系页面、排名以及关系运行记录；
- Alignment state, synthesis, progression history, repair threads, evidence,
  archive, and daily dreams under `~/dreams/`;
  `~/dreams/` 下的对齐状态、综合、演进历史、修复线程、证据、归档和每日梦境；
- Goals bookkeeping proposals and applied tracking changes;
  目标簿记提案和已应用的跟踪变更；
- Shopping-profile runs, staged `shopping/PROFILE.md` files, receipts,
  queued or running profile writers, and the durable profile at  
  `~/memory/shopping/PROFILE.md`;
  购物档案运行、暂存的 `shopping/PROFILE.md` 文件、回执、排队或运行中的档案写入器，以及位于 `~/memory/shopping/PROFILE.md` 的持久档案；
- Studying plans, goal briefs, suggestions, momentum, and  
  `~/workspace/objectives/goals/STUDYING.md`;
  学习计划、目标简报、建议、进展，以及 `~/workspace/objectives/goals/STUDYING.md`；
- Ideas, idea sources, embeddings, cards, accepted builds, and Feed prompts or
  units derived from them;
  想法、想法来源、嵌入、卡片、已接受的构建，以及由其派生的 Feed 提示或单元；
- Discovery Pool or other community cards and previews published from those  
  Ideas;
  从这些想法发布的 Discovery Pool 或其他社区卡片与预览；
- Skill-improvement proposals, staged artifacts, installed user skill changes,
  learnings, and follow-ups;
  技能改进提案、暂存工件、已安装的用户技能变更、经验教训和后续跟进；
- Conversational-follow-up attempts, persisted selector decisions and
  `feedback_note` values, and selected candidates derived from recent public
  chat or `~/FEEDBACK.md`;
  会话跟进尝试、持久化的选择器决定和 `feedback_note` 值，以及从近期公开聊天或 `~/FEEDBACK.md` 派生的已选候选；
- fleet-learning and Feed publication outboxes, centrally published lessons,
  receipts, and unconsumed remote records derived from the information.
  集群学习与 Feed 发布发件箱、集中发布的经验教训、回执，以及由该信息派生的未消费远程记录。

Removing only a final projection is insufficient when an objective run or its
staging artifact can reapply it. Claim retraction plus reindex is not enough
for `~/memory/shopping/PROFILE.md`: edit or remove matching profile
entries directly, verify the file from fresh state, and prove no queued,
running, staged, or scheduled shopping-profile run can restore them. Quiesce
an in-flight writer first. A retained run receipt may keep content-free
provenance, but any model-readable evidence
or generated proposal containing the subject needs an explicit disposition.

当某个目标运行或其暂存工件能够重新施加某个最终投影时，仅删除该最终投影是不够的。对 `~/memory/shopping/PROFILE.md` 而言，仅撤回断言加重建索引并不足够：要直接编辑或移除匹配的档案条目，从全新状态验证该文件，并证明没有任何排队、运行、暂存或已排期的购物档案运行能够恢复它们。先静默进行中的写入器。保留的运行回执可以保存不含内容的来源信息，但任何包含该主题的、模型可读的证据或生成的提案都需要明确的处置。

## 5. Goals, schedules, and future producers / 目标、日程与未来生产者

Inspect user goals, assistant tracking items, entries, associations,
suggestions, briefings, and saved prompt snapshots. Inspect all cron
definitions and bodies, owned reminders, heartbeat instructions, event hooks,
hook scripts and logs, workflow definitions and saved runs, pending delivery
outbox rows, pending notification payloads, channel-delivery payloads, and
already admitted workers.

检查用户目标、助手跟踪项、条目、关联、建议、简报和已保存的提示词快照。检查所有 cron 定义与内容、所属提醒、心跳指令、事件钩子、钩子脚本与日志、工作流定义与已保存的运行、待投递的发件箱行、待发送的通知载荷、频道投递载荷，以及已被接纳的工作器。

Treat the six-hour conversational-follow-up selector as a producer. It reads
both `~/FEEDBACK.md` and up to 48 hours of visible public chat, and a persisted
pending attempt may append its already-selected `feedback_note` on retry.
Quiesce or account for an in-flight attempt before editing the file, remove only
matching preference bullets, and verify that no pending decision can rewrite
them or send a follow-up about the subject. Use recent chat privately to find
already-derived candidates or decisions, but do not list the retained visible
conversation as a cleanup limitation.

把六小时一次的会话跟进选择器当作一个生产者。它会读取 `~/FEEDBACK.md` 和最多 48 小时的可见公开聊天，且一个持久化的待处理尝试可能在重试时追加其已选定的 `feedback_note`。在编辑该文件之前，先静默或核算进行中的尝试，只移除匹配的偏好条目，并验证没有任何待处理的决定能改写它们或就该主题发送跟进。可私下使用近期聊天来查找已派生的候选或决定，但不要把保留的可见对话列为清理限制。

If the subject is still eligible in that 48-hour lookback and no owning
exclusion prevents the selector from using it, this producer is not fully
closed. Report the conversational-follow-up category as pending until the
lookback expires or an exclusion is proven, without presenting the visible
conversation itself as the limitation.

如果该主题在那 48 小时的回溯期内仍然符合选用条件，且没有任何所属排除机制阻止选择器使用它，则该生产者尚未完全关闭。在回溯期届满或排除被证明之前，应将会话跟进类别报告为待处理，但不要把可见对话本身呈现为限制。

On a host with the current runtime control, a successful confirmation spawn is
that owning exclusion: the selector receives a subject-free instruction not to
use, preserve in follow-up preferences, or surface the covered topic, and a
still-pending decision made before a newer confirmation is suppressed before it
can write or deliver. Verify that the control is available; if it is not, keep
the producer pending under the rule above.

在具备当前运行时控制的主机上，一次成功的确认生成即是那种所属排除：选择器会收到一条不含主题内容的指令，要求其不使用、不在跟进偏好中保留、也不呈现被覆盖的话题；而较早确认之前作出的仍在等待的决定，会在其能够写入或投递之前被抑制。要验证该控制是否可用；如果不可用，则按上述规则将该生产者保持为待处理。

Use the owning goal, tracking, cron, hooks, and workflow tools. Decide whether
the subject is incidental text to rewrite or whether the whole obligation must
be deleted. Deleting a goal may cascade its owned schedules; the plan must make
that consequence visible. Disable or cancel an approved producer before
cleaning its output so it cannot race the cleanup.

使用所属的目标、跟踪、cron、钩子和工作流工具。判断该主题是需要改写的偶发文本，还是必须删除整项义务。删除一个目标可能级联删除其所属的日程；计划必须使这一后果清晰可见。在清理某个已批准生产者的输出之前，先禁用或取消该生产者，使其无法与清理产生竞态。

## 6. Artifacts and sharing / 工件与分享

Search file artifacts, web artifacts and their `app.db` records, media,
attachments, widgets, profiles, idea builds, briefings, exported files, and
generated reports. Include Spaces and their proposals or shares, projects,
podcast episodes plus their remote audio, feed, cover, and RSS objects, Feed
posts or units, media descriptions and metadata, extracted OCR or transcripts,
and avatar or profile projections. Check whether a matching artifact is shared,
published, sent through a channel, attached to an email, or copied to an
outside service.

搜索文件工件、Web 工件及其 `app.db` 记录、媒体、附件、小部件、档案、想法构建、简报、导出文件和生成的报告。包括 Spaces 及其提案或分享、项目、播客剧集及其远程音频、订阅源、封面和 RSS 对象、Feed 帖子或单元、媒体描述与元数据、提取的 OCR 或转录，以及头像或档案投影。检查匹配的工件是否被分享、发布、经频道发送、附于电子邮件，或复制到外部服务。

Use artifact actions for artifact-owned data. Local removal does not revoke a
shared URL or retract a message. Unsharing, deleting an external copy, or
contacting another person is a distinct external action and needs separate
approval if it was not in the confirmed plan.

对工件所有的数据使用工件操作。本地删除不会撤销共享 URL，也不会撤回消息。取消分享、删除外部副本或联系他人是另一种独立的外部操作，如果不在已确认的计划内，就需要单独批准。

## 7. Connected and cached sources / 已连接与已缓存来源

The same information may still exist in email, calendar, contacts, device
cache, health or sensor records, call logs,
channel history, a browser session or its page, screenshot, DOM, download, or
accessibility cache, an ingest/raw-signal record, a connector, or another
external account. Determine whether Muse merely read it, cached it, or
authored it. The forget request does not by itself authorize changing the
source service.

同样的信息可能仍存在于电子邮件、日历、联系人、设备缓存、健康或传感器记录、通话日志、频道历史、浏览器会话或其页面、截屏、DOM、下载或无障碍缓存、摄取/原始信号记录、连接器或另一个外部账户中。要判断 Muse 是仅仅读取了它、缓存了它，还是撰写了它。遗忘请求本身并不授权更改来源服务。

If an unchanged connected item could be ingested again, the plan needs either
a source-scoped exclusion or an explicit limitation. Do not keep rediscovering
the subject from external services during verification.

如果一个未变更的已连接项目可能被再次摄取，计划就需要一个来源作用域的排除，或一项明确的限制。在验证期间不要不断从外部服务重新发现该主题。

## 8. Operational and retained copies / 运营与保留副本

Account for runtime events, inference/provider traces, Pariscope or Scuba
telemetry, journald, database WAL, snapshots, backups, mobile notifications,
prompt renderings, cached request or summary rows, and retention-controlled
service copies. These are not ordinary agent memory, and a VM subagent may not
be able to erase them.

要核算运行时事件、推理/provider 追踪、Pariscope 或 Scuba 遥测、journald、数据库 WAL、快照、备份、移动通知、提示词渲染、缓存的请求或摘要行，以及受保留策略控制的服务副本。这些不属于普通的智能体记忆，VM 子代理可能无法擦除它们。

State whether each known class is deleted, replaced, excluded from active
model use, or retained until an external policy expires. Never claim physical
erasure of backups or service telemetry without authoritative confirmation.

对每个已知类别都要说明其是被删除、被替换、被排除在活跃模型使用之外，还是被保留至外部策略过期。在没有权威确认的情况下，绝不能声称备份或服务遥测已被物理擦除。

## 9. Verification closure / 验证闭环

The final verification must show:

最终验证必须表明：

1. no approved authoritative source remains available to the agent;
   智能体不再能访问任何已批准的权威来源；
2. exact and semantic retrieval no longer returns a usable copy;
   精确检索和语义检索都不再返回可用的副本；
3. standing prompt context, new compaction summaries, memory flushes,
   restart checkpoints, and the shopping profile do not preserve or restate it;
   常设提示词上下文、新的压缩摘要、记忆落盘、重启检查点和购物档案都不保留或复述它；
4. no active, staged, queued, or scheduled producer, including a
   shopping-profile run, can recreate it;
   没有任何活跃、暂存、排队或已排期的生产者（包括购物档案运行）能够重建它；
5. approved artifacts and shares have the requested disposition;
   已批准的工件和分享已获得请求的处置；
6. remaining external or retention-controlled copies are explicitly reported.
   剩余的外部或受保留策略控制的副本已被明确报告。

A check that reintroduces the sensitive text into a normal chat, memory entry,
todo, filename, or report is itself a failure. Keep detailed match evidence in
the private plan and expose only discreet counts and categories to the user.
Do not include the original visible conversation in the final limitation
report.

把敏感文本重新引入普通聊天、记忆条目、todo、文件名或报告的检查本身就是一次失败。将详细的匹配证据保存在私有计划中，只向用户暴露审慎的计数和类别。不要在最终的限制报告中包含原始的可见对话。

【评论】验证环节"不得复述敏感文本本身"的要求，避免了删除流程自己成为信息二次传播渠道的悖论，是被遗忘权类功能中常见的技术考量。
