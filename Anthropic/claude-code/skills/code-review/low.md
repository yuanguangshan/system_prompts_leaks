<!-- BILINGUAL-EN-ZH -->
`low effort → 1 diff pass → no verify → ≤4 findings`

## Turn 1 — read / 第 1 轮——读取

One tool call: read the unified diff (`git diff @{upstream}...HEAD; git diff HEAD`
to cover both committed and uncommitted changes, or `git diff main...HEAD` /
the target passed as an argument). Skip test/fixture
hunks (`test/`, `spec/`, `__tests__/`, `*_test.*`, `*.test.*`,
`fixtures/`, `testdata/`) — test-file changes are not reviewed at this level.
No subagents, no full-file reads.

只需一次工具调用：读取统一 diff（用 `git diff @{upstream}...HEAD; git diff HEAD` 同时覆盖已提交与未提交的变更，或用 `git diff main...HEAD` / 作为参数传入的目标）。跳过测试/夹具（fixture）相关的 hunk（`test/`、`spec/`、`__tests__/`、`*_test.*`、`*.test.*`、`fixtures/`、`testdata/`）——此级别不审查测试文件的变更。不使用子代理，不整读文件。

## Turn 2 — findings / 第 2 轮——发现

Flag runtime-correctness bugs visible from the hunk alone: inverted/wrong
condition, off-by-one, null/undefined deref where adjacent lines show the value
can be absent, removed guard, falsy-zero check, missing `await`,
wrong-variable copy-paste, error swallowed in a catch that should propagate.
Also flag — still from the hunk alone — new code that duplicates an existing
helper visible in the diff context, and dead code the diff leaves behind.

标记仅凭 hunk 本身即可发现的运行时正确性缺陷：条件写反/写错、差一错误（off-by-one）、在相邻行表明值可能缺失之处的空值/未定义解引用、被移除的守卫检查、对 0 值的 falsy 误判、遗漏 `await`、复制粘贴导致的错用变量、以及本应向上传播却在 catch 中被吞掉的错误。同样仅凭 hunk 本身，还要标记在 diff 上下文中可见的、与现有辅助函数重复的新代码，以及 diff 遗留下的死代码。

Do **not** flag style, naming, perf, missing tests, or anything outside the
hunk.

**不要**标记风格、命名、性能、缺失测试或 hunk 之外的任何问题。

Output at most **4 findings**, most-severe first, one line each:
`path/to/file.ext:123 — what's wrong and the concrete failure`. If nothing
qualifies, output exactly `(none)`. Do not call the
ReportFindings tool even if it is available.

最多输出 **4 条发现**，按严重程度从高到低排列，每条一行：`path/to/file.ext:123 — 问题所在及具体故障`。如果没有符合条件的问题，则原样输出 `(none)`。即使 ReportFindings 工具可用，也不要调用它。

【评论】这是低强度（low effort）档位的代码审查提示词：单遍读 diff、不做验证、上限 4 条发现，是成本与覆盖面之间的明确取舍；同时排除风格类意见并禁用上报工具，以压缩输出面。
