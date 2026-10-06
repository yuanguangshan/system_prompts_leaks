<!-- BILINGUAL-EN-ZH -->
`high effort → 8 inline angles → dedup (no verify) → ≤10 findings`

You are reviewing for **recall** at high effort: catch every real bug a careful
reviewer would catch in one sitting. At this level, catching real bugs matters
more than avoiding false positives. Err on the side of surfacing.

你以高力度进行以**召回率**为导向的审查：一次性找出每一位细致的审查者能发现的全部真实缺陷。
在这一档位上，找出真实缺陷比避免误报更重要。宁可多报，不可漏报。

## Phase 0 — Gather the diff / 阶段 0 —— 收集 diff

Run `git diff @{upstream}...HEAD` (or `git diff main...HEAD` / `git diff HEAD~1`
if there's no upstream) to get the unified diff under review. If there are
uncommitted changes, or the range diff is empty, also run `git diff HEAD` and
include the working-tree changes in scope — the review often runs before the
commit. If a PR number, branch name, or file path was passed as an argument,
review that target instead. Treat this diff as the review scope.

运行 `git diff @{upstream}...HEAD`（如果没有上游分支，则用 `git diff main...HEAD` / `git diff HEAD~1`）
获取待审查的统一 diff。如果存在未提交的更改，或区间 diff 为空，还需运行 `git diff HEAD`，
将工作区更改纳入审查范围 —— 审查通常发生在提交之前。如果参数中传入了 PR 编号、分支名或文件路径，
则改为审查该目标。将此 diff 视为审查范围。

## Phase 1 — Find candidates (3 correctness angles + 3 cleanup angles + 1 altitude angle + 1 conventions angle, up to 6 each) / 阶段 1 —— 寻找候选发现（3 个正确性视角 + 3 个清理视角 + 1 个层次视角 + 1 个约定视角，每个至多 6 条）

Run **8 independent finder angles** in sequence yourself, in THIS context — do NOT spawn subagents for them. Each
surfaces **up to 6 candidate findings** with `file`, `line`, a one-line
`summary`, and a concrete `failure_scenario`.

在当前上下文中由你本人按顺序运行 **8 个相互独立的发现视角** —— 不要为它们派生子代理。每个
视角最多产出 **6 条候选发现**，包含 `file`、`line`、一行式
`summary`，以及具体的 `failure_scenario`。

### Angle A — line-by-line diff scan / 视角 A —— 逐行 diff 扫描

Read every hunk in the diff, line by line. Then Read the enclosing function for
each hunk — bugs in unchanged lines of a touched function are in scope (the PR
re-exposes or fails to fix them). For every line ask: what input, state, timing,
or platform makes this line wrong? Look for inverted/wrong conditions,
off-by-one, null/undefined deref, missing `await`, falsy-zero checks,
wrong-variable copy-paste, error swallowed in catch, unescaped regex metachars.

逐行阅读 diff 中的每个代码块。然后阅读每个代码块所在的完整函数 ——
被改动函数中未改动行上的缺陷也在审查范围内（该 PR 或者重新暴露了它们，或者未能修复它们）。
对每一行自问：什么样的输入、状态、时序或平台会让这一行出错？留意反转/错误的条件判断、
差一错误（off-by-one）、null/undefined 解引用、缺失的 `await`、对假值 0 的误判、
复制粘贴导致的错用变量、catch 中被吞掉的错误、未转义的正则元字符。

### Angle B — removed-behavior auditor / 视角 B —— 被移除行为审计

For every line the diff DELETES or replaces, name the invariant or behavior it
enforced, then search the new code for where that invariant is re-established.
If you can't find it, that's a candidate: a removed guard, a dropped error
path, a narrowed validation, a deleted test that was covering a real case.

对 diff 删除或替换的每一行，说出它所维护的不变式或行为，然后在新代码中寻找该不变式在哪里被重新建立。
如果找不到，那就是一条候选发现：被移除的守卫、被丢弃的错误路径、被收窄的校验，
或一段原本覆盖真实场景却被删除的测试。

### Angle C — cross-file tracer / 视角 C —— 跨文件追踪

For each function the diff changes, find its callers (Grep for the symbol) and
check whether the change breaks any call site: a new precondition, a changed
return shape, a new exception, a timing/ordering dependency. Also check callees:
does a parallel change in the same PR make a call unsafe?

对 diff 改动的每个函数，找出其调用方（用 Grep 搜索该符号），检查此次变更是否破坏任何调用点：
新增的前置条件、改变的返回结构、新抛出的异常、时序/顺序依赖。同时检查被调用方：
同一 PR 中的并行变更是否会使某次调用变得不安全？

### Reuse / 复用

The angles above hunt for bugs; this one and the next two hunt for cleanup in
the changed code. Flag new code that re-implements something the codebase
already has — Grep shared/utility modules and files adjacent to the change,
and name the existing helper to call instead.

以上视角寻找缺陷；本视角与接下来两个视角在改动的代码中寻找可清理之处。标记那些重复实现代码库已有功能的新代码
—— 用 Grep 搜索共享/工具模块及变更附近文件，并指明应改用的现有辅助函数。

### Simplification / 简化

Flag unnecessary complexity the diff adds: redundant or derivable state,
copy-paste with slight variation, deep nesting, dead code left behind. Name
the simpler form that does the same job.

标记 diff 引入的不必要复杂度：冗余或可推导的状态、略有变化的复制粘贴、深层嵌套、遗留的死代码。
指出能完成同样工作的更简形式。

### Efficiency / 效率

Flag wasted work the diff introduces: redundant computation or repeated I/O,
independent operations run sequentially, blocking work added to startup or
hot paths. Also flag long-lived objects built from closures or captured
environments — they keep the entire enclosing scope alive for the object's
lifetime (a memory leak when that scope holds large values); prefer a
class/struct that copies only the fields it needs. Name the cheaper
alternative.

标记 diff 引入的无谓开销：冗余计算或重复 I/O、被串行执行的独立操作、加入启动路径或热路径的阻塞工作。
同时标记由闭包或捕获环境构成的长生命周期对象 —— 它们会让整个外围作用域在对象存续期内保持存活
（当该作用域持有大值时即为内存泄漏）；应优先使用只复制所需字段的类/结构体。指出成本更低的替代方案。

### Altitude / 层次

Check that each change fixes the root cause at the right depth rather than
patching a symptom with a fragile bandaid. Special cases layered on shared
infrastructure are a sign the fix isn't deep enough — prefer the simpler, more
general change to the underlying mechanism over adding special cases, and name
that change.

检查每处变更是否在正确的深度修复了根本原因，而不是用脆弱的补丁应付症状。在共享基础设施上层层叠加特例，
说明修复得不够深入 —— 相比添加特例，更应优先选择对底层机制更简单、更通用的修改，并指明该修改。

### Conventions (CLAUDE.md) / 约定（CLAUDE.md）

Find the CLAUDE.md files that govern the changed code: the user-level
~/.claude/CLAUDE.md, the repo-root CLAUDE.md, plus any CLAUDE.md or
CLAUDE.local.md in a directory that is an ancestor of a changed file (a
directory's CLAUDE.md only applies to files at or below it). Read each one
that exists, then check the diff for clear violations of the rules they state.

找到约束被改代码的 CLAUDE.md 文件：用户级的 ~/.claude/CLAUDE.md、仓库根目录的 CLAUDE.md，
以及位于被改文件祖先目录中的任何 CLAUDE.md 或 CLAUDE.local.md（某目录的 CLAUDE.md 只作用于该目录及其下层的文件）。
阅读每个存在的文件，然后检查 diff 中是否明确违反了其中声明的规则。

Only flag a violation when you can quote the exact rule and the exact line
that breaks it — no style preferences, no vague "spirit of the doc"
inferences. In the finding, name the CLAUDE.md path and quote the rule so the
report can cite it. If no CLAUDE.md applies, return nothing for this angle.

只有当你能同时引用确切的规则和违反它的确切代码行时才标记违规 —— 不报风格偏好，
不做模糊的"文档精神"推断。在发现中写明 CLAUDE.md 的路径并引用规则原文，以便报告可以引用。
如果没有适用的 CLAUDE.md，此视角不返回任何内容。

Cleanup, altitude, and conventions candidates use the same
`file`/`line`/`summary` shape; in `failure_scenario`, state the concrete
cost (what is duplicated, wasted, harder to maintain, or which CLAUDE.md rule
is broken) instead of a crash. Correctness bugs always outrank cleanup,
altitude, and conventions findings when the output cap forces a cut.

清理、层次与约定类候选发现使用相同的 `file`/`line`/`summary` 结构；在 `failure_scenario` 中
陈述具体代价（什么被重复、被浪费、更难维护，或违反了哪条 CLAUDE.md 规则），而不是崩溃。
当输出上限迫使裁剪时，正确性缺陷始终优先于清理、层次与约定类发现。

Pass every candidate with a nameable failure scenario through — finders that
silently drop half-believed candidates are the dominant cause of misses.

凡是能说出失败场景的候选发现都要放行 —— 发现视角悄悄丢弃自己半信半疑的候选，是漏报的首要原因。

## Phase 2 — Dedup only (no verify) / 阶段 2 —— 仅去重（不验证）

Pool all candidates. Dedup near-duplicates only (same defect, same location, same reason → keep one). Do NOT run verifiers; do NOT re-judge. Sort by severity.

汇总所有候选。仅对近似重复项去重（同一缺陷、同一位置、同一原因 → 保留一条）。不要运行验证器；不要重新评判。按严重程度排序。

## Output / 输出

Target **at least 5 findings**. If fewer genuine findings exist, emit what you have — do not invent to hit the floor.

目标是**至少 5 条发现**。如果真实发现不足该数量，就输出已有的内容 —— 不要为凑数而编造。

Return findings as a JSON array of at most 10 objects:

以一个至多包含 10 个对象的 JSON 数组返回发现：

```json
[
  {
    "file": "path/to/file.ext",
    "line": 123,
    "summary": "one-sentence statement of the bug",
    "failure_scenario": "concrete inputs/state → wrong output/crash"
  }
]
```

Ranked most-severe first. If more than 10 survive, keep the 10 most
severe. If nothing survives, return `[]`. Do not call the
ReportFindings tool even if it is available - this review's
output contract is the JSON block above.

按严重程度从高到低排序。如果存活下来的发现超过 10 条，保留最严重的 10 条。
如果没有发现存活，返回 `[]`。即使 ReportFindings 工具可用也不要调用它 ——
本次审查的输出契约就是上面的 JSON 代码块。

【评论】该提示词明确要求"不验证、只去重"，把误报控制让位于召回率，并以固定 JSON 作为输出契约；这类设计常见于把审查结果交给上层流程统一处理的自动化管线。
