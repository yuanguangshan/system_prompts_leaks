---
name: xlsx
description: "Use this skill any time a spreadsheet file is the primary input or output. This means any task where the user wants to: open, read, edit, or fix an existing .xlsx, .xlsm, .xltx, .csv, or .tsv file (e.g., adding columns, computing formulas, formatting, charting, cleaning messy data); create a new spreadsheet from scratch or from other data sources; or convert between tabular file formats. Trigger especially when the user references a spreadsheet file by name or path — even casually (like \"the xlsx in my downloads\") — and wants something done to it or produced from it. Also trigger for cleaning or restructuring messy tabular data files (malformed rows, misplaced headers, junk data) into proper spreadsheets. The deliverable must be a spreadsheet file. Do NOT trigger when the primary deliverable is a Word document, HTML report, standalone Python script, database pipeline, or Google Sheets API integration, even if tabular data is involved."
license: Proprietary. LICENSE.txt has complete terms
---
<!-- BILINGUAL-EN-ZH -->

# XLSX creation, editing, and analysis / XLSX 创建、编辑与分析

| Task | Approach |
|---|---|
| **Create** or **edit** with formulas/formatting | `openpyxl` — see gotchas below |
| **Bulk data** in or out | `pandas` (`read_excel`, `to_excel`) |
| **Quick look** at a sheet | `markitdown file.xlsx` — `## SheetName` per sheet; reads `.xlsm` too. No cell coordinates, so don't plan edits from it |
| **Read** a model (formulas *and* values) | two `load_workbook` passes — see gotchas |

| 任务 | 方法 |
|---|---|
| 带公式/格式地**创建**或**编辑** | `openpyxl`——见下方的注意事项 |
| **批量数据**导入或导出 | `pandas`（`read_excel`、`to_excel`） |
| **快速查看**工作表 | `markitdown file.xlsx`——每个工作表输出 `## SheetName`；也能读 `.xlsm`。没有单元格坐标，因此不要据它规划编辑 |
| **读取**模型（公式*和*值） | 两次 `load_workbook`——见注意事项 |

> `openpyxl`, `pandas`, and `markitdown` are preinstalled — do not run `pip install` first; write the script and import directly. Only if an import fails (or the `markitdown` command is missing): `pip install` the missing package.

> `openpyxl`、`pandas` 与 `markitdown` 均已预装——不要先运行 `pip install`；直接编写脚本并导入。只有当导入失败（或 `markitdown` 命令缺失）时：才 `pip install` 缺失的包。

> Script paths below are relative to this skill's directory.

> 下文中的脚本路径均相对于本技能所在目录。

## Requirements for every output / 每个产出的要求

- **Professional font** (Arial, Times New Roman) throughout, unless the user says otherwise.
  全文使用**专业字体**（Arial、Times New Roman），除非用户另有要求。
- **Zero formula errors.** Never ship while `recalc.py` reports `errors_found`. If you think an error predates you, prove it: load the *original* with `data_only=True` and look at that cell. An error you introduced looks exactly like one you inherited.
  **公式错误为零。**只要 `recalc.py` 报告 `errors_found` 就绝不交付。如果你认为某个错误在你接手之前就存在，请证明它：用 `data_only=True` 加载*原始文件*并查看那个单元格。你引入的错误与你继承的错误看起来一模一样。
- **Use formulas, never hardcoded results.** Write `sheet['B10'] = '=SUM(B2:B9)'`, not the Python-computed total. The sheet must recalculate when its inputs change.
  **使用公式，绝不硬编码结果。**写 `sheet['B10'] = '=SUM(B2:B9)'`，而不是 Python 算出的总和。工作表必须在输入变化时能够重算。
- **Follow the user's spec literally.** Exact tab names, exact column headers, and the formula they spelled out. A redesign that computes something else fails, however elegant.
  **严格照字面遵循用户的规格。**精确的工作表名、精确的列标题、以及他们明确写出的公式。一个计算出其他东西的重新设计就是失败，无论多么优雅。
- **Document every assumption and hardcoded number** where the reader will see it — a cell comment, or an adjacent cell at a table's end. Cite a real source when one exists (`Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]`); when the number came from the user, say so plainly.
  **记录每一个假设和硬编码数字**，写在读者能看到的位置——单元格批注，或表格末尾的相邻单元格。存在真实来源时引用它（`Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]`）；数字来自用户时，就直说是用户提供的。
- **A workbook *you create* for someone to fill in** needs a short legend naming which cells to edit, and one example row of realistic values showing the expected format. Never add such a row to a file you were asked to edit.
  **你为他人填写而创建的工作簿**需要一段简短说明，指明哪些单元格可编辑，并提供一行符合真实格式的示例值。绝不要把这样的示例行加进别人要求你编辑的文件。
- **Editing an existing file: match its conventions exactly.** They override every guideline here. Find its designated input cells first — a distinct font color, fill, or shading marks them — write only there, and leave every existing formula untouched.
  **编辑现有文件：完全遵循其既有约定。**这些约定覆盖本文件中的所有准则。先找到文件指定的输入单元格——通常以独特的字体颜色、填充或底纹标出——只在那里写入，并保持所有既有公式原样不动。

## Recalculate (mandatory whenever the file contains formulas) / 重算（文件含公式时为必做步骤）

openpyxl writes formulas as strings with **no cached values**. Until you recalculate, every
formula cell reads back as `None` to anything reading cached values — `pandas`,
`load_workbook(data_only=True)`, and most previewers.

openpyxl 把公式写成字符串，**不带缓存值**。在你重算之前，对所有读取缓存值的东西来说，每个公式单元格读回来都是 `None`——包括 `pandas`、`load_workbook(data_only=True)` 以及大多数预览器。

```bash
python scripts/recalc.py output.xlsx [timeout_seconds]   # default 30
```

LibreOffice computes every formula, the file is **rewritten in place**, and you get JSON:
`status` (`success` | `errors_found`), `total_formulas`, `total_errors`, and an
`error_summary` naming up to 100 cells per error type (`locations_truncated` says how many it
withheld — trust `total_errors`, not the length of the list). Fix what it names and run it
again. **JSON with an `error` key instead of a `status` means nothing was recalculated**, and
only that case exits non-zero — `errors_found` exits 0, so never treat a clean exit as a clean
workbook.

LibreOffice 会计算每一个公式，文件被**就地重写**，你会得到 JSON：`status`（`success` | `errors_found`）、`total_formulas`、`total_errors`，以及一个 `error_summary`，为每种错误类型最多列出 100 个单元格（`locations_truncated` 说明它省略了多少——要相信 `total_errors`，而不是列表的长度）。修复它点名的问题并再次运行。**JSON 里出现 `error` 键而不是 `status`，表示什么都没有重算**，且只有这种情况才以非零码退出——`errors_found` 退出码是 0，因此绝不要把干净的退出码当成干净的工作簿。

**A green recalc proves your formulas *evaluate*, not that they are *right*.** An off-by-one
range or a reference to the wrong row yields a clean, error-free file with wrong numbers.
Write 2–3 formulas first and check they pull the values you expect, before building out a grid.

**重算通过只证明你的公式*能求值*，不证明它们是*对的*。**差一（off-by-one）的范围或引用错行，都会产出一个干净、无错误但数字错误的文件。先写 2–3 个公式，确认它们取到的值符合预期，再铺开整个网格。

**A workbook that links to another file loses those links** if you re-save it with openpyxl and
then recalculate. Such a formula reads `='[1]Returns Analysis'!$B$2` — the `[1]` is an index
into the workbook's external-reference list, naming a *separate file on disk*, not a sheet.
That file is rarely present here, so the cell's cached value is the only thing holding its
data. openpyxl strips that value on save; LibreOffice then has to resolve the reference for
real, fails, writes `#NAME?`, and deletes every link. `recalc.py` refuses to run in that state
— copy those cells' values out of the original before you save over them (`--force` overrides,
and accepts the loss).

如果一个工作簿链接到另一个文件，用 openpyxl 重新保存后再重算，这些链接就会**丢失**。这样的公式形如 `='[1]Returns Analysis'!$B$2`——`[1]` 是工作簿外部引用列表中的索引，指向*磁盘上一个单独的文件*，而不是某个工作表。该文件在这里通常不存在，因此单元格的缓存值是唯一保存其数据的东西。openpyxl 保存时会剥掉该值；LibreOffice 随后必须真正解析该引用、解析失败、写入 `#NAME?`，并删除所有链接。`recalc.py` 在这种状态下拒绝运行——在你覆盖保存之前，先把这些单元格的值从原文件中复制出来（`--force` 可强制覆盖并接受损失）。

## Choosing formulas that survive verification / 选择经得起校验的公式

LibreOffice implements fewer functions than Excel, and one it cannot evaluate becomes a
literal `#NAME?` baked into the file you deliver.

LibreOffice 实现的函数比 Excel 少，而它无法求值的函数会变成字面的 `#NAME?`，直接烙进你交付的文件。

- **Prefer Excel-2007-era functions** — `SUMIFS`, `INDEX`, `MATCH`, `IFERROR`, `SUMPRODUCT` — which need no prefix.
  **优先使用 Excel 2007 时代的函数**——`SUMIFS`、`INDEX`、`MATCH`、`IFERROR`、`SUMPRODUCT`——它们不需要前缀。
- **Six post-2007 functions work, but only with an `_xlfn.` prefix**, because openpyxl writes your formula into the XML verbatim and Excel stores post-2007 names prefixed (its UI hides the prefix): `_xlfn.TEXTJOIN`, `_xlfn.CONCAT`, `_xlfn.IFS`, `_xlfn.SWITCH`, `_xlfn.MAXIFS`, `_xlfn.MINIFS`. Written bare, each yields `#NAME?`.
  **六个 2007 之后的函数可用，但必须带 `_xlfn.` 前缀**，因为 openpyxl 会把公式原样写进 XML，而 Excel 存储 2007 之后的函数名时自带前缀（其界面隐藏了前缀）：`_xlfn.TEXTJOIN`、`_xlfn.CONCAT`、`_xlfn.IFS`、`_xlfn.SWITCH`、`_xlfn.MAXIFS`、`_xlfn.MINIFS`。裸写则每个都产生 `#NAME?`。
- **Never use `XLOOKUP`, `XMATCH`, `SORT`, `FILTER`, `UNIQUE`, or `SEQUENCE`.** The runtime's LibreOffice cannot evaluate them under *any* prefix. Newer builds do evaluate them, but they are spilling array functions and an openpyxl-written file has no spill metadata, so only the top-left cell of the range gets a value — and `recalc.py` reports `total_errors: 0` on the truncated result. Use `INDEX`/`MATCH` for lookups, and sort, filter, and de-duplicate in Python before writing the cells.
  **绝不要使用 `XLOOKUP`、`XMATCH`、`SORT`、`FILTER`、`UNIQUE` 或 `SEQUENCE`。**运行时的 LibreOffice 在*任何*前缀下都无法对它们求值。较新的构建确实能求值，但它们是溢出（spilling）数组函数，而 openpyxl 写出的文件没有溢出元数据，于是只有区域的左上角单元格会得到值——而且 `recalc.py` 会在被截断的结果上报告 `total_errors: 0`。查找用 `INDEX`/`MATCH`，排序、筛选与去重在 Python 中完成后再写入单元格。
- A formula LibreOffice could not parse is written back **lowercased** — a quick tell beside a `#NAME?`.
  LibreOffice 无法解析的公式会被**转成小写**写回——这是伴随 `#NAME?` 的一个快速识别标志。

## openpyxl gotchas / openpyxl 注意事项

- **Reading a model takes two loads.** `data_only=True` yields cached values with the formulas gone; the default yields formula strings with no values. One pass cannot give you both.
  **读取模型需要加载两次。**`data_only=True` 给出缓存值但没有公式；默认方式给出公式字符串但没有值。一次加载无法两者兼得。
- **`data_only=True` is destructive if you save.** That workbook has no formulas left, so saving replaces every one with a literal — permanently.
  **`data_only=True` 在保存时是破坏性的。**那个工作簿已不含任何公式，保存会用字面值永久替换每一个公式。
- **`data_only=True` on a file openpyxl just wrote returns `None` everywhere** — run `recalc.py` first. (A formula whose result is `""` also reads back as `None`.)
  **对 openpyxl 刚写出的文件用 `data_only=True` 会处处返回 `None`**——先运行 `recalc.py`。（结果为 `""` 的公式同样会读回 `None`。）
- **Merged cells: write the top-left anchor only.** Every other cell in the range is a `MergedCell` whose `.value` is read-only.
  **合并单元格：只写左上角锚点。**区域内的其他单元格都是 `MergedCell`，其 `.value` 是只读的。
- **`.xlsm` loses its macros unless you pass `keep_vba=True`** to `load_workbook`.
  **`.xlsm` 会丢失宏**，除非给 `load_workbook` 传入 `keep_vba=True`。
- **A sheet name containing a space must be quoted** in a cross-sheet reference: `='Assumptions Inputs'!$B$5`. Unquoted, it evaluates to `#VALUE!`.
  **含空格的工作表名在跨表引用中必须加引号**：`='Assumptions Inputs'!$B$5`。不加引号会求值为 `#VALUE!`。

## Financial models / 财务模型

Unless the user says otherwise, or the existing file already does something else.

除非用户另有要求，或现有文件已经在使用其他做法。

**Color:** blue text (`0,0,255`) for hardcoded inputs and scenario levers · black for formulas ·
green (`0,128,0`) for links to another sheet · red (`255,0,0`) for links to another file ·
yellow fill (`255,255,0`) for key assumptions and cells the user should fill in.

**颜色：**硬编码输入与情景调节变量用蓝色文字（`0,0,255`）· 公式用黑色 · 链接到其他工作表用绿色（`0,128,0`）· 链接到其他文件用红色（`255,0,0`）· 关键假设与需要用户填写的单元格用黄色填充（`255,255,0`）。

**Numbers:** currency `$#,##0`, with the unit named in the header (`Revenue ($mm)`) · zeros
render as `-`, including in percentages (`$#,##0;($#,##0);-`) · negatives in parentheses ·
percentages `0.0%`, **stored as fractions** (`0.15` renders `15.0%`; storing `15` renders
`1500.0%`) · valuation multiples `0.0x` · years as text (`"2024"`, never `2,024`).

**数字：**货币用 `$#,##0`，单位写在列标题里（`Revenue ($mm)`）· 零显示为 `-`，百分比中也一样（`$#,##0;($#,##0);-`）· 负数加括号 · 百分比用 `0.0%`，**以小数存储**（存 `0.15` 显示 `15.0%`；存 `15` 会显示 `1500.0%`）· 估值倍数用 `0.0x` · 年份作为文本（`"2024"`，绝不写成 `2,024`）。

**Structure:** every assumption in its own labeled cell, referenced by the formulas that use it
(`=B5*(1+$B$6)`, never `=B5*1.05`) · formulas consistent across every projection period, since a
lone edited cell mid-row is the commonest silent error · guard denominators that can be zero.

**结构：**每个假设放在自己带标签的单元格中，由使用它的公式引用（`=B5*(1+$B$6)`，绝不写 `=B5*1.05`）· 公式在每个预测期间保持一致，因为行中间被单独编辑的单元格是最常见的静默错误 · 为可能为零的分母加保护。

## Dependencies / 依赖

`openpyxl`, `pandas`, `markitdown` (pip, preinstalled — install only if an import fails or the command is missing) · LibreOffice (`soffice`, auto-configured for sandboxed environments via `scripts/office/soffice.py`)

`openpyxl`、`pandas`、`markitdown`（pip 包，已预装——仅在导入失败或命令缺失时安装）· LibreOffice（`soffice`，经 `scripts/office/soffice.py` 为沙箱环境自动配置）
