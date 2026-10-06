<!-- BILINGUAL-EN-ZH -->
`xhigh effort → 10 inline angles → dedup (no verify) → sweep → ≤15 findings`

`xhigh effort → 10 个内联视角 → 去重（不验证）→ 补漏扫描 → ≤15 条发现`

You are reviewing for **recall** at extra-high effort: catch every real bug. At
this level, catching real bugs matters more than avoiding false positives — a
missed bug ships. Err on the side of surfacing.

你以超高强度进行面向**召回率**的评审：抓住每一个真实缺陷。在这个级别，抓住真实缺陷比避免误报更重要——漏掉的缺陷会随版本发布。宁可多报，不可漏报。

## Phase 0 — Gather the diff / 阶段 0——收集 diff

Run `git diff @{upstream}...HEAD` (or `git diff main...HEAD` / `git diff HEAD~1`
if there's no upstream) to get the unified diff under review. If there are
uncommitted changes, or the range diff is empty, also run `git diff HEAD` and
include the working-tree changes in scope — the review often runs before the
commit. If a PR number, branch name, or file path was passed as an argument,
review that target instead. Treat this diff as the review scope.

运行 `git diff @{upstream}...HEAD`（如果没有上游，则用 `git diff main...HEAD` / `git diff HEAD~1`）获取待评审的统一 diff。如果有未提交的更改，或范围 diff 为空，再运行 `git diff HEAD` 并把工作区更改纳入范围——评审常常发生在提交之前。如果传入了 PR 编号、分支名或文件路径作为参数，则改为评审该目标。将此 diff 视为评审范围。

## Phase 1 — Find candidates (5 correctness angles + 3 cleanup angles + 1 altitude angle + 1 conventions angle, up to 8 each) / 阶段 1——寻找候选（5 个正确性视角 + 3 个清理视角 + 1 个层次视角 + 1 个约定视角，每个最多 8 条）

Run **10 independent finder angles** in sequence yourself, in THIS context — do NOT spawn subagents for them. Each
surfaces **up to 8 candidate findings**. Do NOT let one angle's conclusions
suppress another's — if two angles flag the same line for different reasons,
record both.

在当前上下文中亲自依次运行 **10 个独立的发现视角**——不要为它们派生子代理。每个视角产出**最多 8 条候选发现**。绝不让一个视角的结论压制另一个视角——如果两个视角因不同原因标记同一行，两条都要记录。

### Angle A — line-by-line diff scan / 视角 A——逐行扫描 diff

Read every hunk in the diff, line by line. Then Read the enclosing function for
each hunk — bugs in unchanged lines of a touched function are in scope (the PR
re-exposes or fails to fix them). For every line ask: what input, state, timing,
or platform makes this line wrong? Look for inverted/wrong conditions,
off-by-one, null/undefined deref, missing `await`, falsy-zero checks,
wrong-variable copy-paste, error swallowed in catch, unescaped regex metachars.

逐行阅读 diff 中的每个补丁块。然后阅读每个补丁块所在的完整函数——被改动的函数中未改动行上的缺陷也在范围内（该 PR 重新暴露了它们或未能修复它们）。对每一行都要问：什么输入、状态、时序或平台会使这一行出错？寻找反转/错误的条件、差一错误、null/undefined 解引用、缺失的 `await`、把 0 误判为假值的检查、复制粘贴导致的错用变量、catch 中被吞掉的错误、未转义的正则元字符。

### Angle B — removed-behavior auditor / 视角 B——被删除行为审计

For every line the diff DELETES or replaces, name the invariant or behavior it
enforced, then search the new code for where that invariant is re-established.
If you can't find it, that's a candidate: a removed guard, a dropped error
path, a narrowed validation, a deleted test that was covering a real case.

对 diff 删除或替换的每一行，说出它曾维护的不变量或行为，然后在新代码中搜索该不变量在哪里被重新建立。如果找不到，那就是一条候选：被移除的守卫、被丢弃的错误路径、被收窄的校验、被删除却覆盖真实场景的测试。

### Angle C — cross-file tracer / 视角 C——跨文件追踪

For each function the diff changes, find its callers (Grep for the symbol) and
check whether the change breaks any call site: a new precondition, a changed
return shape, a new exception, a timing/ordering dependency. Also check callees:
does a parallel change in the same PR make a call unsafe?

对 diff 改动的每个函数，找到其调用方（用 Grep 搜索该符号），检查改动是否破坏了任何调用点：新的前置条件、变化的返回结构、新的异常、时序/顺序依赖。同时检查被调用方：同一 PR 中的并行改动是否使某个调用变得不安全？

### Angle D — language-pitfall specialist / 视角 D——语言陷阱专家

Scan for the classic pitfalls of the diff's language/framework — for example:
JS falsy-zero, `==` coercion, closure-captured loop var; Python mutable default
args, late-binding closures; Go nil-map write, range-var capture; SQL injection;
timezone/DST drift; float equality. Flag any instance the diff introduces.

扫描 diff 所用语言/框架的经典陷阱——例如：JS 的 0 被当作假值、`==` 强制类型转换、闭包捕获循环变量；Python 的可变默认参数、迟绑定闭包；Go 的向 nil map 写入、range 变量捕获；SQL 注入；时区/夏令时漂移；浮点数相等比较。标记 diff 引入的任何实例。

### Angle E — wrapper/proxy correctness / 视角 E——包装器/代理正确性

When the PR adds or modifies a type that wraps another (cache, proxy, decorator,
adapter): check that every method routes to the wrapped instance and not back
through a registry/session/global — e.g. a caching provider holding a
`delegate` field that resolves IDs via `session.get(...)` instead of
`delegate.get(...)` will re-enter the cache or recurse. Also check that the
wrapper forwards all the methods the callers actually use.

当 PR 新增或修改了一个包装另一对象的类型（缓存、代理、装饰器、适配器）时：检查每个方法都路由到被包装实例，而不是经由注册表/会话/全局变量绕回去——例如一个持有 `delegate` 字段的缓存提供方，若通过 `session.get(...)` 而不是 `delegate.get(...)` 来解析 ID，就会重新进入缓存或造成递归。同时检查包装器是否转发了调用方实际使用的所有方法。

### Reuse / 复用

The angles above hunt for bugs; this one and the next two hunt for cleanup in
the changed code. Flag new code that re-implements something the codebase
already has — Grep shared/utility modules and files adjacent to the change,
and name the existing helper to call instead.

以上视角寻找缺陷；本视角与接下来两个视角在改动的代码中寻找可清理项。标记那些重新实现了代码库中已有功能的新代码——用 Grep 搜索共享/工具模块以及改动相邻的文件，并指出应改用的现有辅助函数。

### Simplification / 简化

Flag unnecessary complexity the diff adds: redundant or derivable state,
copy-paste with slight variation, deep nesting, dead code left behind. Name
the simpler form that does the same job.

标记 diff 引入的不必要复杂度：冗余或可推导的状态、带细微变化的复制粘贴、深层嵌套、遗留的死代码。指出能完成同样工作的更简形式。

### Efficiency / 效率

Flag wasted work the diff introduces: redundant computation or repeated I/O,
independent operations run sequentially, blocking work added to startup or
hot paths. Also flag long-lived objects built from closures or captured
environments — they keep the entire enclosing scope alive for the object's
lifetime (a memory leak when that scope holds large values); prefer a
class/struct that copies only the fields it needs. Name the cheaper
alternative.

标记 diff 引入的浪费性工作：冗余计算或重复 I/O、被串行执行的独立操作、加入启动路径或热路径的阻塞工作。同时标记由闭包或捕获环境构建的长生命周期对象——它们会在对象存续期间让整个外层作用域保持存活（当该作用域持有大值时就是内存泄漏）；优先使用只复制所需字段的类/结构体。指出更廉价的替代方案。

### Altitude / 层次

Check that each change fixes the root cause at the right depth rather than
patching a symptom with a fragile bandaid. Special cases layered on shared
infrastructure are a sign the fix isn't deep enough — prefer the simpler, more
general change to the underlying mechanism over adding special cases, and name
that change.

检查每处改动是否在正确的深度修复了根因，而不是用脆弱的创可贴遮掩症状。在共享基础设施上层层叠加特例是修复不够深入的信号——优先选择对底层机制做更简单、更通用的改动，而不是添加特例，并说出那个改动。

### Conventions (CLAUDE.md) / 约定（CLAUDE.md）

Find the CLAUDE.md files that govern the changed code: the user-level
~/.claude/CLAUDE.md, the repo-root CLAUDE.md, plus any CLAUDE.md or
CLAUDE.local.md in a directory that is an ancestor of a changed file (a
directory's CLAUDE.md only applies to files at or below it). Read each one
that exists, then check the diff for clear violations of the rules they state.

找到约束被改动代码的 CLAUDE.md 文件：用户级的 ~/.claude/CLAUDE.md、仓库根目录的 CLAUDE.md，以及位于被改文件祖先目录中的任何 CLAUDE.md 或 CLAUDE.local.md（某目录的 CLAUDE.md 只适用于该目录及其下层文件）。阅读每个存在的文件，然后检查 diff 是否明显违反了其中声明的规则。

Only flag a violation when you can quote the exact rule and the exact line
that breaks it — no style preferences, no vague "spirit of the doc"
inferences. In the finding, name the CLAUDE.md path and quote the rule so the
report can cite it. If no CLAUDE.md applies, return nothing for this angle.

只有能同时引用确切的规则和违反它的确切代码行时才标记违规——不要报风格偏好，不要做"文件精神"式的模糊推断。在发现中写明 CLAUDE.md 路径并引用规则，使报告可以引用它。如果没有适用的 CLAUDE.md，此视角不返回任何内容。

Cleanup, altitude, and conventions candidates use the same
`file`/`line`/`summary` shape; in `failure_scenario`, state the concrete
cost (what is duplicated, wasted, harder to maintain, or which CLAUDE.md rule
is broken) instead of a crash. Correctness bugs always outrank cleanup,
altitude, and conventions findings when the output cap forces a cut.

清理、层次与约定类候选使用相同的 `file`/`line`/`summary` 结构；在 `failure_scenario` 中陈述具体代价（什么被重复、被浪费、更难维护，或违反了哪条 CLAUDE.md 规则），而不是崩溃。当输出上限迫使裁剪时，正确性缺陷始终优先于清理、层次与约定类发现。

## Phase 2 — Dedup only (no verify) / 阶段 2——仅去重（不验证）

Pool all candidates. Dedup near-duplicates only (same defect, same location, same reason → keep one). Do NOT run verifiers; do NOT re-judge. Sort by severity. Do NOT drop on uncertainty.

汇总所有候选。仅对近似重复项去重（同一缺陷、同一位置、同一原因 → 保留一条）。不要运行验证器；不要重新评判。按严重程度排序。不要因不确定而丢弃。

## Phase 3 — Sweep for gaps / 阶段 3——补漏扫描

Take one more pass (same context — no subagent) as a fresh reviewer who has the deduplicated list. Re-read
the diff and enclosing functions looking ONLY for defects not already listed.
Do not re-derive or re-confirm anything already there — the job is gaps. Focus
on what the first pass tends to miss: moved/extracted code that dropped a guard
or anchor; second-tier footguns (dataclass default evaluated once, `hash()`
non-determinism, lock-scope shrink, predicate methods with side effects);
setup/teardown asymmetry in tests; config defaults flipped.

以一名手握去重后清单的全新评审者身份再过一遍（同一上下文——不用子代理）。重读 diff 及相关函数，只寻找尚未列入清单的缺陷。不要重新推导或重新确认已有的内容——任务只在于补漏。聚焦第一遍容易漏掉的东西：移动/抽取代码时丢失了守卫或锚点；二线陷阱（dataclass 默认值只求值一次、`hash()` 非确定性、锁作用域收窄、带副作用的谓词方法）；测试中 setup/teardown 的不对称；被翻转的配置默认值。

Surface **up to 8 additional candidates**, each naming a defect not already on
the list. If nothing new, return nothing from this phase — do not pad.

产出**最多 8 条额外候选**，每条都要指出清单上尚不存在的缺陷。如果没有新发现，此阶段不返回任何内容——不要凑数。

## Output / 输出

Target **at least 7 findings**. If fewer genuine findings exist, emit what you have — do not invent to hit the floor.

目标为**至少 7 条发现**。如果真实发现不足，就输出实际拥有的——不要为凑数而编造。

Return findings as a JSON array of at most 15 objects:

以最多包含 15 个对象的 JSON 数组返回发现：

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

Ranked most-severe first. If more than 15 survive, keep the 15 most
severe. If nothing survives, return `[]`. Do not call the
ReportFindings tool even if it is available - this review's
output contract is the JSON block above.

按严重程度从高到低排序。如果存活项超过 15 条，保留最严重的 15 条。如果没有存活项，返回 `[]`。即使 ReportFindings 工具可用也不要调用它——本次评审的输出契约就是上面的 JSON 块。
