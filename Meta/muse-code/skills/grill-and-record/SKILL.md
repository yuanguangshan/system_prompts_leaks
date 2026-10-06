---
name: grill-and-record
description: Run an explicitly requested decision interview and record each settled decision in durable project documentation.
---
<!-- BILINGUAL-EN-ZH -->

# Grill and Record / 追问并记录

Use this skill only when the user explicitly asks for grilling plus durable documentation or directly invokes this skill. Complexity, ambiguity, or a possible need for docs alone never activates it. This skill carries its own interview and documentation contract. Do not load or invoke `grill` or `domain-modeling` at runtime.

只有当用户明确要求"追问式访谈 + 持久化文档"，或直接调用本技能时才使用它。仅凭复杂性、歧义或可能需要文档，都不能激活本技能。本技能自带访谈与文档契约。运行时不要加载或调用 `grill` 或 `domain-modeling`。

## Interview Contract / 访谈契约

1. Research discoverable facts in the repository, issue, docs, and code before asking the user. Ask only for judgments or facts that cannot be discovered.
   向用户提问之前，先在仓库、议题、文档和代码中调研可查明的事实。只询问无法自行发现的判断或事实。
2. Ask one decision-forcing question at a time. State the recommended answer and the reason briefly, then wait for the answer.
   每次只问一个迫近决策的问题。简要给出推荐答案和理由，然后等待回答。
3. Prefer the host's structured input surface for bounded decisions:
   对有界决策，优先使用宿主的结构化输入界面：
   - Use `request_user_input` when it is exposed and the decision fits 2-3 short, mutually exclusive choices. Put the recommended answer first.
     Send one question per call with `id`, `header`, `question`, and `options`; omit `Other` and `None` because the client provides a free-form escape. Never set the top-level `auto_resolution_ms` in a grilling interview: a grilling decision always waits for the human answer, because auto-resolving to the recommended default is never safe when the whole point of the interview is the user's judgment.
     当 `request_user_input` 可用且决策适于 2-3 个简短、互斥的选项时使用它。把推荐答案放在首位。
     每次调用只发送一个问题，包含 `id`、`header`、`question` 和 `options`；省略 `Other` 和 `None`，因为客户端自带自由输入出口。追问式访谈中绝不要设置顶层 `auto_resolution_ms`：追问式决策必须等待人工回答，因为当访谈的全部意义就在于用户的判断时，自动采用推荐默认值绝不安全。
     【评论】禁用自动裁决是这类访谈技能的关键安全条款：超时自动选默认值会把"征询用户判断"变成走过场。
   - On Claude Code, use `AskUserQuestion` for the same bounded choice.
     在 Claude Code 上，同样的有界选择使用 `AskUserQuestion`。
   - Use plain chat only when the structured surface is unavailable or the answer needs free-form discussion.
     只有当结构化界面不可用，或答案需要自由讨论时，才使用普通聊天。
   Never pose a bounded checkpoint as plain assistant text or as a free-form question before offering structured choices.
   在提供结构化选项之前，绝不要把有界检查点作为普通助手文本或自由式问题提出。
4. Use comparison tables only when the user explicitly requested one or the question concerns agent-product behavior, such as Claude Code versus Codex.
   只有当用户明确要求，或问题涉及智能体产品行为对比（例如 Claude Code 与 Codex 之争）时，才使用对比表。
5. Follow dependent decisions until the skill decides the decision tree is exhausted. Never ask a final "are we done?" meta-question.
   沿着有依赖关系的决策一路追问，直到技能判断决策树已经穷尽。绝不要问"我们问完了吗"这类收尾元问题。
6. Summarize the settled contract: goals, non-goals, decisions, constraints, risks, validation, and unresolved items.
   汇总已敲定的契约：目标、非目标、决策、约束、风险、验证方式以及未决事项。

Ending the interview never authorizes implementation. Implement only after a separate explicit user request.

访谈结束绝不等于授权实现。只有在用户另行明确提出请求后才进行实现。

## Documentation Contract / 文档契约

1. Before the first question, resolve the target document from the user's named target and the repository's existing documentation conventions. Inspect local instructions, indexes, specs, ADRs, glossaries, and nearby docs. If the target is undeterminable, ask one question. Never invent a universal `docs/grilling/<date>.md` location.
   在第一个问题之前，根据用户指定的目标和仓库既有的文档约定确定目标文档。检查本地说明、索引、规格、ADR、术语表及邻近文档。无法确定目标时，问一个问题。绝不要凭空发明通用的 `docs/grilling/<date>.md` 位置。
2. Create or update the target as **Draft**. Write each settled decision immediately as Draft instead of waiting for the interview to end. Keep unresolved questions visibly marked.
   以 **Draft**（草稿）状态创建或更新目标文档。每个敲定的决策立即以草稿形式写入，不要等访谈结束。未决问题要保持醒目标记。
3. If interrupted or cancelled, preserve the Draft and all file edits, mark unresolved questions, and never auto-revert documentation changes.
   若被中断或取消，保留草稿和所有文件编辑，标出未决问题，绝不自动回滚文档改动。
4. Read detailed `CONTEXT.md`, ADR, glossary, or other format references only when that document type is actually being written.
   只有当真的要写该类型文档时，才去阅读详细的 `CONTEXT.md`、ADR、术语表或其他格式参考。
5. Normal Write/Edit tool events are the live proof of documentation work. Do not emit duplicate `Updated <path>` status lines.
   常规 Write/Edit 工具事件就是文档工作的实时证明。不要重复输出 `Updated <path>` 状态行。
6. An explicit docs request is a hard completion condition: the session cannot finish successfully without a useful documentation creation or update. "No docs needed" with zero file changes never satisfies it.
   明确的文档要求是硬性完成条件：没有一次有用的文档创建或更新，会话就不能算成功完成。"不需要文档"而零文件改动永远无法满足该条件。
7. Only explicit user acceptance may change a document from Draft to **Final**. Exhausting the decision tree does not imply acceptance.
   只有用户明确接受才能把文档从 Draft 变为 **Final**。穷尽决策树并不代表用户已接受。
8. In the final response, list every changed documentation path and whether it stayed Draft or became Final.
   在最终答复中，列出每个被更改的文档路径，以及它保持在 Draft 还是变成了 Final。
