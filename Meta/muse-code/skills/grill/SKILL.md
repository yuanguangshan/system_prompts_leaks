---
name: grill
description: Run an explicitly requested decision interview and record each settled decision in durable project documentation.
source: https://github.com/mattpocock/skills
license: MIT
---
<!-- BILINGUAL-EN-ZH -->
# Grill / Grill（决策访谈）

Use this skill only when the user explicitly asks for grilling plus durable documentation or directly invokes this skill; the `agents` coordinator handing it an unclear goal, or one decision needing the user's confirmation, before its plan is a direct invocation (its question is the user's judgement — target, measure, scope — that repository facts shape but never answer), and there the durable record is the project's own `PROJECT.md` or `library/` file, never a repository doc. Complexity, ambiguity, or a possible need for docs alone never activates it. This skill carries its own interview and documentation contract. Do not load or invoke `domain-modeling` at runtime.

仅在用户明确要求"拷问式访谈"加持久化文档，或直接调用本技能时使用；`agents` 协调者在制定计划前遇到不清晰的目标、或有一个需要用户确认的决策而交由本技能处理，也算直接调用（它的问题就是用户的判断 —— 目标、度量、范围 —— 仓库事实可以塑造但永远不能代替回答），且此时持久记录是项目自己的 `PROJECT.md` 或 `library/` 文件，绝不是仓库文档。复杂性、模糊性或可能需要文档，单凭这些绝不会激活本技能。本技能自带其访谈与文档契约。运行时不要加载或调用 `domain-modeling`。

## Interview Contract / 访谈契约

1. Research discoverable facts in the repository, issue, docs, and code before asking the user. Ask only for judgments or facts that cannot be discovered.
   在向用户提问之前，先研究仓库、issue、文档和代码中可发现的事实。只询问无法发现的判断或事实。
2. Ask one decision-forcing question at a time. State the recommended answer and the reason briefly, then wait for the answer.
   一次只问一个迫使决策的问题。简要给出推荐答案和理由，然后等待回答。
3. Ask every interview question in plain text as an ordinary assistant response. Do not use a question tool.
   每个访谈问题都以普通助手回复的纯文本形式提出。不要使用提问工具。
   - When a bounded decision benefits from 2-3 short, mutually exclusive choices, list them in plain text with the recommended answer first.
     当一个有边界的决策受益于 2-3 个简短且互斥的选项时，用纯文本列出它们，推荐答案放最前。
   - Invite the user to choose, modify, or discuss the choices instead of forcing a structured selection.
     邀请用户选择、修改或讨论这些选项，而不是强制结构化选择。
4. Use comparison tables only when the user explicitly requested one or the question concerns agent-product behavior, such as Claude Code versus Codex.
   仅当用户明确要求，或问题涉及智能体产品行为（例如 Claude Code 与 Codex 之比较）时，才使用对比表格。
5. Follow dependent decisions until the skill decides the decision tree is exhausted. Never ask a final "are we done?" meta-question.
   顺着依赖决策追问，直到本技能判定决策树已经穷尽。绝不要问"我们完成了吗"这类收尾元问题。
6. Summarize the settled contract: goals, non-goals, decisions, constraints, risks, validation, and unresolved items.
   概述已敲定的契约：目标、非目标、决策、约束、风险、验证方式和未决事项。

## Background formal evidence / 后台形式化证据

When the target project provides an applicable formal checker, follow its
local documentation to prepare the declared interview input and run the check
before choosing the outline and after relevant answers or source changes.
Read the completed invocation's results and source/input binding, then use
unresolved obligations and counterexamples to revise the next question.
Recheck changed inputs. Reuse answers only within their supported scope and
conditions, preserving independent choices. After each answer, write its source
and applicable scope into the declared input and the journal or Draft decision;
rerun and record the revised remaining obligations before the next outline.
A possible model assignment is not human acceptance; bounded coverage does
not cover unlisted questions.
Missing or stale evidence limits dependent conclusions while independent
permitted work continues. Ask one practical question in the user's language,
with ordinary choices and consequences; explain formal techniques when asked.

当目标项目提供了适用的形式化检查器时，按其本地文档准备所声明的访谈输入，并在选择提纲之前、以及收到相关答案或源码变更之后运行检查。读取完成调用的结果与源/输入绑定，然后用未解决的义务和反例修正下一个问题。输入变更后重新检查。答案只在其支持的范围内和条件下复用，并保留各独立选择。每次得到答案后，把其来源和适用范围写入所声明的输入以及日志或 Draft 决策；在下一次提纲之前重跑并记录修订后的剩余义务。一个可能的模型赋值不等于人类的接受；有边界的覆盖不涵盖未列出的问题。证据缺失或过时会限制依赖性结论，而独立的获许工作继续进行。用用户的语言提出一个实际的问题，配以通常的选项和后果；被问到时再解释形式化技术。

## Scope Contract / 范围契约

The interview is not finished until the settled contract fixes the scope in
writing and the user accepts that text explicitly:

访谈直到已敲定的契约把范围固定成文字、且用户明确接受该文字才算结束：

1. **Artifact-level boundary.** Name what the deliverable is (the documents,
   directories, PRs, or code paths in scope) and the artifact classes that are
   out of scope, such as follow-on specs, tests, runtime code, or task plans.
   **工件级边界。** 指明交付物是什么（在范围内的文档、目录、PR 或代码路径），以及范围外的工件类别，例如后续规格、测试、运行时代码或任务计划。
2. **Done means.** A short checklist of the observable conditions that finish
   the work: landed commits, closed issues, verified evidence. Nothing outside
   the checklist is a completion dependency.
   **完成的含义。** 一份简短清单，列明结束工作的可观察条件：已落地的提交、已关闭的 issue、已验证的证据。清单之外的任何东西都不是完成依赖。
3. **Staged designs are approved one stage at a time.** If a decision record
   describes later stages (an ADR that stages Constitution, spec, or runtime
   changes), record them as deferred proposals; accepting the record never
   approves the later stages. Each stage returns for its own interview.
   **分阶段设计一次只批准一个阶段。** 若决策记录描述了后续阶段（一份分阶段安排 Constitution、规格或运行时变更的 ADR），把它们记录为延期提案；接受该记录绝不等于批准后续阶段。每个阶段都要回来接受自己的访谈。
4. **Execution words never widen scope.** "Go", "do it all", or "land 1-5"
   authorize only the accepted boundary. When later work would add an artifact
   class, a new PR, or a task program outside the boundary, stop before
   producing it and take one of exactly two paths: obtain the owner's explicit
   approval of the wider boundary, recorded on the owning issue, or move the
   extra work into a follow-up issue that starts its own interview. Never build
   first and ask afterwards.
   **执行用语绝不扩大范围。** "Go"、"do it all" 或 "land 1-5" 只授权已被接受的边界。当后续工作会在边界之外增加一个工件类别、一个新 PR 或一套任务计划时，在产出之前停下，并恰好走两条路径之一：获得所有者对更宽边界的明确批准并记录在所属 issue 上，或把额外工作挪进一个开启自己访谈的后续 issue。绝不要先构建再询问。

【评论】"执行用语绝不扩大范围"一条把自然语言中的放行词与授权边界严格绑定，防止用户随口的"go"被解读为对更大范围的许可，属于对自主智能体权限蔓延的防护设计。

Post the accepted scope contract where the executing lane and its supervisors
can read it (for repository work, the owning issue), quoting the acceptance.
This comment is lane-coordination evidence, not a decision record: it quotes
the user's exact words with channel and time; the durable decision lives in the
record this skill writes, never in the comment.

把已接受的范围契约张贴在执行线及其监督者能看到的地方（仓库工作则张贴在所属 issue），引用接受原文。这条评论是线协调证据，不是决策记录：它引用用户原话并附渠道和时间；持久的决策存放在本技能写出的记录中，绝不在评论里。

Ending the interview never authorizes implementation. Implement only after a separate explicit user request.

访谈结束绝不等于授权实现。只有在用户单独明确提出请求之后才实现。

## Documentation Contract / 文档契约

1. Before the first question, resolve the target document from the user's named target and the repository's existing documentation conventions. Inspect local instructions, indexes, specs, ADRs, glossaries, and nearby docs. If the target is undeterminable, ask one question. Never invent a universal `docs/grilling/<date>.md` location.
   在第一个问题之前，根据用户指名的目标和仓库既有的文档约定确定目标文档。检查本地说明、索引、规格、ADR、术语表和邻近文档。若目标无法确定，问一个问题。绝不要凭空发明一个通用的 `docs/grilling/<date>.md` 位置。
2. Create or update the target as **Draft**. Write each settled decision immediately as Draft instead of waiting for the interview to end. Keep unresolved questions visibly marked.
   以 **Draft** 状态创建或更新目标文档。每个敲定的决策立即以 Draft 写入，而不是等访谈结束。未决问题要保持醒目标记。
3. If interrupted or cancelled, preserve the Draft and all file edits, mark unresolved questions, and never auto-revert documentation changes.
   若被中断或取消，保留 Draft 和所有文件编辑，标记未决问题，绝不自动回滚文档变更。
4. Read detailed `CONTEXT.md`, ADR, glossary, or other format references only when that document type is actually being written.
   仅当确实要写该类型的文档时，才阅读详细的 `CONTEXT.md`、ADR、术语表或其他格式参考。
5. Normal Write/Edit tool events are the live proof of documentation work. Do not emit duplicate `Updated <path>` status lines.
   常规的 Write/Edit 工具事件就是文档工作的实时证明。不要输出重复的 `Updated <path>` 状态行。
6. An explicit docs request is a hard completion condition: the session cannot finish successfully without a useful documentation creation or update. "No docs needed" with zero file changes never satisfies it.
   明确的文档请求是硬性完成条件：没有有用的文档创建或更新，会话就不能算成功完成。"不需要文档"加上零文件变更永远不满足该条件。
7. Only explicit user acceptance may change a document from Draft to **Final**. Exhausting the decision tree does not imply acceptance.
   只有用户明确接受才能把文档从 Draft 改为 **Final**。穷尽决策树并不蕴含接受。
8. In the final response, list every changed documentation path and whether it stayed Draft or became Final.
   在最终回复中，列出每个被更改的文档路径，以及它是保持 Draft 还是成为了 Final。

## Credit / 署名

This skill's name and interview approach — one recommended answer per
question, repository facts researched instead of asked, decisions written
down as they settle — come from the `grilling` skill (and the former
`grill-with-docs` skill) by Matt Pocock (@mattpocockuk),
<https://github.com/mattpocock/skills>. The skill text shipped here is our own.
See the package's `CREDITS.md`.

本技能的名称与访谈方法 —— 每个问题配一个推荐答案、仓库事实靠研究而非提问获得、决策在敲定之际即写下 —— 来自 Matt Pocock（@mattpocockuk）的 `grilling` 技能（以及先前的 `grill-with-docs` 技能），<https://github.com/mattpocock/skills>。此处随附的技能文本是我们自己写的。参见该包的 `CREDITS.md`。
