<!-- BILINGUAL-EN-ZH -->

Summarize the preceding session so the same agent can continue without rereading the context being replaced. Do not use tools, solve the task, or continue the work. Return only the handoff.

总结前一段会话，使同一个代理无需重读将被替换的上下文即可继续工作。不要使用工具、不要解决任务、不要继续推进工作。只返回交接摘要。

Use each heading exactly once, in this order. Keep every section concise; use prose or bullets as the material warrants, and state each fact once.

每个标题只用一次，顺序如下。各节保持简洁；根据材料情况选用行文或列表，每条事实只陈述一次。

## Primary Request And Intent / 主要请求与意图

## User Constraints And Preferences / 用户约束与偏好

## Current State / 当前状态

## Files, APIs, Commands, And Tests / 文件、API、命令与测试

## Decisions And Rationale / 决策与理由

## Errors, Failed Attempts, And Fixes / 错误、失败尝试与修复

## Open Questions And Risks / 未决问题与风险

## Pending Tasks And Next Step / 待办任务与下一步

## User Message Timeline / 用户消息时间线

Preserve the current objective and deliverables; exact active instructions, constraints, prohibitions, corrections, completion conditions, ordering requirements, and required literals; completed, in-progress, and remaining work; concrete state and evidence; files, APIs, commands, tests, identifiers, and counts; decisions and rationale; failures and fixes; questions, blockers, and risks. For an exact required response, preserve whether surrounding text, labels, or code fences are forbidden.

保留当前目标与交付物；保留仍然生效的指令、约束、禁令、更正、完成条件、顺序要求与必须逐字使用的内容；保留已完成、进行中与剩余的工作；保留具体状态与证据；保留文件、API、命令、测试、标识符与计数；保留决策及其理由；保留失败与修复；保留问题、阻碍与风险。对于要求精确回复的内容，须保留“是否禁止周边文本、标签或代码围栏”的信息。

Merge prior summaries with later events and use the protected trailing messages identified below to determine the latest combined state. Current State distinguishes completed work from work in progress. Pending Tasks And Next Step states unfinished outcomes and one next state-changing action, not a turn-by-turn execution script. Do not list completed work as pending or add response boundaries the user did not state.

把先前摘要与后续事件合并，并利用下文指出的受保护尾部消息确定最新的合并状态。“当前状态”要区分已完成的工作与进行中的工作。“待办任务与下一步”陈述未完成的结果和下一个改变状态的动作，而不是逐轮执行脚本。不要把已完成的工作列为待办，也不要添加用户未声明的回复边界。

A fulfilled one-time prerequisite belongs only in completed state: retain its ordering and non-repeat constraints, identifying literals, exact command, and concrete result as past-tense evidence. User Message Timeline quotes only genuine user requests in order when later requests modify earlier intent. Handoff requests are runtime control, not user history; never list this or any prior handoff request. Use prior summaries only as source material. Carry active constraints forward until the task ends or the user supersedes them. Do not invent facts or claim completion without evidence.

已完成的一次性前置条件只归入已完成状态：以过去时证据的形式保留其顺序性与不可重复约束、识别用的字面量、确切命令和具体结果。“用户消息时间线”只按顺序引用真实的用户请求，同时体现后一个请求对先前意图的修改。交接（handoff）请求属于运行时控制，不是用户历史；绝不列出本次或任何先前的交接请求。先前摘要只作素材使用。仍然生效的约束要一直沿用，直到任务结束或用户将其取代。不得编造事实，也不得在没有证据的情况下声称已完成。

【评论】“绝不列出交接请求”这条约束用于防止压缩摘要把运行时控制消息误记为用户意图，属于上下文压缩的保真性设计。
