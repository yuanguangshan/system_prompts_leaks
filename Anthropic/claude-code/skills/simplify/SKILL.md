---
name: simplify
description: |-
  Review the changed code for reuse, simplification, efficiency, and altitude cleanups, then apply the fixes. Quality only — it does not hunt for bugs; use /code-review for that.
---
<!-- BILINGUAL-EN-ZH -->

`/simplify → 4 cleanup agents in parallel → apply the fixes`

`/simplify → 4 个清理 agent 并行 → 应用修复`

You are improving the quality of the changed code, not hunting for bugs. Review
it for reuse, simplification, efficiency, and altitude issues, then fix what you
find. Do not look for correctness bugs — that is what `/code-review` is for.

你的任务是改进已变更代码的质量，而不是寻找缺陷。从复用、简化、效率和抽象层次（altitude）几个角度审查代码，然后修复发现的问题。不要寻找正确性缺陷——那是 `/code-review` 的职责。

## Phase 0 — Gather the diff / 阶段 0 — 收集 diff

Run `git diff @{upstream}...HEAD` (or `git diff main...HEAD` / `git diff HEAD~1`
if there's no upstream) to get the unified diff under review. If there are
uncommitted changes, or the range diff is empty, also run `git diff HEAD` and
include the working-tree changes in scope — the review often runs before the
commit. If a PR number, branch name, or file path was passed as an argument,
review that target instead. Treat this diff as the review scope.

运行 `git diff @{upstream}...HEAD`（若无上游分支则用 `git diff main...HEAD` / `git diff HEAD~1`）获取待评审的统一 diff。如果存在未提交的变更，或范围 diff 为空，再运行 `git diff HEAD`，把工作区变更也纳入范围——评审常常发生在提交之前。如果参数中传入了 PR 编号、分支名或文件路径，则改为评审该目标。将此 diff 视为评审范围。

## Phase 1 — Review (4 cleanup agents in parallel) / 阶段 1 — 评审（4 个清理 agent 并行）

Launch **4 independent review agents** via the Agent tool, all in a
single message so they run concurrently. Pass each agent the diff and one of
the four angles below. Each returns its findings with `file`, `line`, a
one-line `summary`, and the concrete cost (what is duplicated, wasted, or
harder to maintain).

通过 Agent 工具启动 **4 个独立的评审 agent**，全部放在同一条消息中发出，使它们并发运行。给每个 agent 传入 diff 以及下列四个视角之一。每个 agent 返回其发现，包含 `file`、`line`、一行 `summary`，以及具体的代价（什么被重复了、什么被浪费了、什么变得更难维护）。

### Reuse / 复用

Flag new code that re-implements something the codebase
already has — Grep shared/utility modules and files adjacent to the change,
and name the existing helper to call instead.

标记那些重新实现了代码库已有功能的新代码——用 Grep 检索共享/工具模块以及与变更相邻的文件，并指名应改为调用的现有辅助函数。

### Simplification / 简化

Flag unnecessary complexity the diff adds: redundant or derivable state,
copy-paste with slight variation, deep nesting, dead code left behind. Name
the simpler form that does the same job.

标记 diff 引入的不必要复杂度：冗余或可推导的状态、略有改动的复制粘贴、深层嵌套、遗留的死代码。指名能完成同样工作的更简形式。

### Efficiency / 效率

Flag wasted work the diff introduces: redundant computation or repeated I/O,
independent operations run sequentially, blocking work added to startup or
hot paths. Also flag long-lived objects built from closures or captured
environments — they keep the entire enclosing scope alive for the object's
lifetime (a memory leak when that scope holds large values); prefer a
class/struct that copies only the fields it needs. Name the cheaper
alternative.

标记 diff 引入的浪费性工作：冗余计算或重复 I/O、被串行执行的相互独立的操作、加到启动路径或热路径上的阻塞工作。同时标记由闭包或捕获环境构建的长生命周期对象——它们会让整个外围作用域在对象存续期内一直保持存活（当该作用域持有大值时即为内存泄漏）；应优先使用只复制所需字段的类/结构体。指名开销更低的替代方案。

### Altitude / 抽象层次

Check that each change fixes the root cause at the right depth rather than
patching a symptom with a fragile bandaid. Special cases layered on shared
infrastructure are a sign the fix isn't deep enough — prefer the simpler, more
general change to the underlying mechanism over adding special cases, and name
that change.

检查每处变更是否在正确的深度修复了根因，而不是用脆弱的创可贴去打症状的补丁。在共享基础设施上层层叠加特例，说明修复得不够深入——应优先选择对底层机制做更简单、更通用的修改，而不是添加特例，并指名该修改。

【评论】"altitude"（抽象层次）是这套评审方法论的核心概念：判断改动是否落在正确的抽象层，避免在错误的层面打补丁。

## Phase 2 — Apply the fixes / 阶段 2 — 应用修复

Wait for all four agents to complete, dedup findings that point at the same
line or mechanism, and fix each remaining one directly. Skip any finding whose
fix would change intended behavior, require changes well outside the reviewed
diff, or that you judge to be a false positive — note the skip rather than
arguing with it. Finish with a brief summary of what was fixed and what was
skipped (or confirm the code was already clean).

等待全部四个 agent 完成，对指向同一行或同一机制的发现去重，然后直接修复其余每一项。跳过任何满足以下条件的发现：修复会改变预期行为、需要改动远超评审 diff 范围之外的代码，或你判定为误报——记录跳过即可，不要与之争辩。最后简要总结修复了什么、跳过了什么（或确认代码本已干净）。
