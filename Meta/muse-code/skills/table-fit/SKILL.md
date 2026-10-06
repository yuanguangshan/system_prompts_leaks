---
name: table-fit
description: Keep a Markdown table readable in a narrow terminal of about 100 display columns - a wide table or one carrying prose in its cells wraps into unreadable ragged rows. Always call read_skill for bundled:table-fit before emitting a Markdown table in a final answer, and load it as soon as the answer starts forming as a comparison, a mapping, a feature matrix, a pros-and-cons, an options rundown, or a per-item summary across several dimensions. Do not load it for a table already inside a file being edited, for code, data, or test fixtures that merely look tabular, for tables the user pasted for you to read, or for an answer with no table and no tabular shape forming.
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Table Fit / 表格适配

The answer is read in a terminal about 100 display columns wide. A table wider
than that wraps every row, and wrapped rows lose the column alignment that made
the table worth using. So a table only pays for itself when it stays narrow.

回答是在约 100 个显示列宽的终端里阅读的。超过这个宽度的表格会让每一行都折行，而折行后的行会失去列对齐——那正是表格值得使用的理由。所以表格只有在保持窄幅时才物有所值。

Two budgets, both hard:

两条硬性预算：

- **At most 4 columns.** The label column counts as one.
  - **最多 4 列。** 标签列也算一列。
- **Every cell a short phrase.** Never a sentence, never prose.
  - **每个单元格都是短语。** 不放句子，不放成段文字。

If the content fits both, use the table. If it does not, do not shrink the font
of the problem by abbreviating into unreadable shorthand, and do not let the
table run wide. Switch shape instead.

如果内容同时满足两条，就用表格。如果不满足，既不要靠缩写成难以辨认的简写来给问题"缩小字号"，也不要任由表格变宽，而是改换表达形态。

## When the content will not compress / 当内容无法压缩时

Use one of these instead of a table. Neither is a lesser answer; for prose-heavy
content they are the better one.

改用以下形式之一替代表格。两者都不是次一等答案；对文字密集的内容，它们才是更好的选择。

**A list** — when each item needs a sentence or two:

**列表** —— 当每一项需要一两句话时：

```
- **tokio** — the default. Widest ecosystem, work-stealing scheduler, heaviest
  dependency tree. Pick it unless you have a specific reason not to.
- **smol** — small and readable. Fewer integrations, so you write more glue.
```

**Short per-item headings** — when each item needs several dimensions covered in
prose:

**简短的逐项小标题** —— 当每一项需要用文字覆盖多个维度时：

```
### tokio
Maturity: production-default across the ecosystem.
Tradeoff: large dependency graph and a heavier compile.

### smol
Maturity: stable, much smaller surface.
Tradeoff: fewer ready-made integrations.
```

## Choosing / 选择

Count the dimensions the answer actually needs, then look at the longest value
any cell would hold.

数一数回答实际需要的维度数量，再看任何一个单元格可能容纳的最长值。

- 4 or fewer dimensions, all values short phrases → table.
  - 维度不超过 4 个，且所有值都是短语 → 用表格。
- More than 4 dimensions → drop the least useful ones to reach 4, or use
  per-item headings.
  - 超过 4 个维度 → 砍掉最没用的维度凑到 4 个，或改用逐项小标题。
- Any dimension whose values run to sentences → list or per-item headings, even
  if there are only two columns.
  - 任何一个维度的值都是完整句子 → 用列表或逐项小标题，哪怕只有两列。

A mapping with short values on both sides — a key to its meaning, a type to its
size, a flag to its effect — is the case tables are for. Keep those as tables.

两侧都是短值的映射关系——键对含义、类型对大小、标志对效果——正是表格的用武之地。这类内容保留为表格。

## Do not / 禁止

- Do not measure or ask for the real terminal width. The budget is static.
  Width is per-connection and transient, and must not enter durable context.
  - 不要测量或询问真实终端宽度。预算是静态的。宽度是按连接而定的瞬态信息，不得进入持久上下文。
- Do not keep a table and simply truncate cells; losing the content is worse
  than losing the table.
  - 不要保留表格却简单截断单元格；丢失内容比丢掉表格更糟。
- Do not restate the table as prose underneath it. Pick one shape.
  - 不要在表格下面再用文字复述一遍。选定一种形态。
- Do not add a table to an answer that did not need a visualization at all. A
  single fact, a one-step action, or a simple edit stays prose.
  - 不要给根本不需要可视化的回答添加表格。单一事实、一步操作或简单编辑仍用普通文字。

【评论】该技能针对终端渲染的物理约束（约 100 列）设定表格预算，并明确要求"终端宽度是瞬态信息，不得进入持久上下文"——这是上下文窗口卫生的一个具体实例。
