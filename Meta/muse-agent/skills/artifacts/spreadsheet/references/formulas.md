<!-- BILINGUAL-EN-ZH -->

# Formulas, recalculation, and editing existing workbooks / 公式、重新计算与编辑既有工作簿

## Why recalculation is mandatory / 为什么重新计算是强制步骤

openpyxl writes a formula as a bare string with no cached value. Until a
real engine recalculates the file, every formula cell reads back as empty
to previewers, pandas, and `load_workbook(data_only=True)`: an
un-recalculated deliverable looks blank to the user. Run  
`python3 "/opt/hatch/skills/artifacts/scripts/recalc_xlsx.py" output.xlsx`  
after every save that touches formulas; it rewrites the file in place and returns JSON with
`status` (`success` or `errors_found`), `total_formulas`, `total_errors`,
and an `error_summary` naming error cells (locations cap at 100 per error
type with a `locations_truncated` count, so trust `total_errors`, not the
list length). `errors_found` exits 0; an `error` key instead of a `status`
means nothing was recalculated. Never deliver while it reports errors, and
never blame a pre-existing error without proving it: load the original
with `data_only=True` and look at that cell first.

openpyxl 把公式写成不带缓存值的裸字符串。在真实计算引擎重新计算该文件之前，每个公式单元格在预览器、pandas 和 `load_workbook(data_only=True)` 读回来都是空的：未重新计算的交付物在用户眼中就是一片空白。每次涉及公式的保存之后都要运行  
`python3 "/opt/hatch/skills/artifacts/scripts/recalc_xlsx.py" output.xlsx`  
；它会就地重写文件，并返回包含 `status`（`success` 或 `errors_found`）、`total_formulas`、`total_errors` 以及 `error_summary`（指出出错的单元格）的 JSON（每种错误类型的位置列表上限为 100 条，并附 `locations_truncated` 计数，所以应相信 `total_errors` 而不是列表长度）。`errors_found` 的退出码是 0；如果返回的是 `error` 键而不是 `status`，说明什么都没有重新计算。报告错误时绝不要交付文件，也绝不要在未证实的情况下把错误归咎于既有问题：先用 `data_only=True` 加载原始文件，查看那个单元格。

【评论】"先证实再归咎既有错误"是一条防止模型推卸责任的核查要求，避免把自身引入的问题说成文件原有缺陷。

A workbook that links to another FILE is a special case: the linked file
is not on this machine, so its cells' cached values are the only data
present, and an openpyxl re-save strips them; recalculation then turns
each into an error. The recalc script refuses such files without
`--force`. Copy the linked cells' values out before saving over them.

链接到另一个文件的工作簿是特殊情况：被链接的文件不在本机上，其单元格的缓存值是唯一存在的数据，而 openpyxl 重新保存会剥掉这些缓存值；重新计算随后会把每个都变成错误。不带 `--force` 时，重算脚本会拒绝处理这类文件。在覆盖保存之前，先把被链接单元格的值复制出来。

## Choosing formulas that survive / 选择能存活的公式

The engine that recalculates here is LibreOffice, which implements fewer
functions than Excel; a function it cannot evaluate ships as a literal  
`#NAME?`.

这里用于重新计算的引擎是 LibreOffice，它实现的函数比 Excel 少；它无法求值的函数会以字面量  
`#NAME?` 的形式交付。

- Prefer the classic set: `SUM`, `SUMIFS`, `INDEX`, `MATCH`, `IFERROR`,
  `SUMPRODUCT`, and their generation.
  优先使用经典函数集：`SUM`、`SUMIFS`、`INDEX`、`MATCH`、`IFERROR`、`SUMPRODUCT` 及其同代函数。
- Post-2007 names are stored prefixed in the XML, and openpyxl writes your
  string verbatim, so write `_xlfn.TEXTJOIN`, `_xlfn.CONCAT`, `_xlfn.IFS`,
  `_xlfn.SWITCH`, `_xlfn.MAXIFS`, `_xlfn.MINIFS`; written bare each one
  yields `#NAME?`.
  2007 之后新增的函数名在 XML 中以带前缀的形式存储，而 openpyxl 会原样写入你给出的字符串，所以要写成 `_xlfn.TEXTJOIN`、`_xlfn.CONCAT`、`_xlfn.IFS`、`_xlfn.SWITCH`、`_xlfn.MAXIFS`、`_xlfn.MINIFS`；裸写任何一个都会得到 `#NAME?`。
- Never use the spilling array functions: `XLOOKUP`, `XMATCH`, `SORT`,
  `FILTER`, `UNIQUE`, `SEQUENCE`. Even where an engine evaluates them, an
  openpyxl-written file carries no spill metadata, so only the top-left
  cell gets a value and the recalculation gate reads the truncated result
  as zero errors. Use `INDEX`/`MATCH` for lookups, and sort, filter, and
  de-duplicate in Python before writing cells.
  绝不使用溢出型数组函数：`XLOOKUP`、`XMATCH`、`SORT`、`FILTER`、`UNIQUE`、`SEQUENCE`。即使在引擎能求值的地方，openpyxl 写出的文件也不带溢出元数据，结果只有左上角单元格有值，而重算关卡会把这种被截断的结果读成零错误。查找请用 `INDEX`/`MATCH`，排序、筛选和去重则在写单元格之前用 Python 完成。
- Quote a sheet name containing a space in cross-sheet references:  
  `='Assumptions Inputs'!$B$5`; unquoted it evaluates to an error.
  跨表引用中，含空格的工作表名要加引号：  
  `='Assumptions Inputs'!$B$5`；不加引号会求值出错。

## openpyxl gotchas / openpyxl 的陷阱

- Reading a model takes two loads: `data_only=True` gives cached values
  with the formulas gone; the default gives formula strings with no
  values. One pass cannot give both.
  读取一个模型需要两次加载：`data_only=True` 给出缓存值但公式丢失；默认模式给出公式字符串但没有值。一次加载无法兼得两者。
- `data_only=True` is destructive if you save: that in-memory workbook has
  no formulas left, so saving replaces every one with a literal.
  如果保存，`data_only=True` 是破坏性的：内存中的工作簿已不含公式，保存会把每个公式替换成一个字面量。
- `data_only=True` on a file openpyxl just wrote returns `None`
  everywhere; recalculate first. A formula whose result is an empty string
  also reads back as `None`, so `None` alone proves nothing.
  对 openpyxl 刚写出的文件使用 `data_only=True` 会到处返回 `None`；请先重新计算。结果为空字符串的公式读回来也是 `None`，所以单凭 `None` 证明不了任何事。
- Merged ranges: write the top-left anchor only; every other cell in the
  range is read-only.
  合并区域：只写左上角锚点单元格；区域内的其他所有单元格都是只读的。
- An `.xlsm` loses its macros unless loaded with `keep_vba=True`.
  `.xlsm` 文件除非以 `keep_vba=True` 加载，否则会丢失宏。

## Editing an existing workbook / 编辑既有工作簿

The file's own conventions override every guideline here. Find its
designated input cells first (a distinct font color, fill, or shading
marks them), write only there, and leave every existing formula untouched.
Match the existing number formats and fonts rather than restyling.

文件自身的约定优先于本文的所有准则。先找出文件指定的输入单元格（通常以独特的字体颜色、填充或底纹标出），只在这些位置写入，并且不改动任何既有公式。匹配既有的数字格式和字体，而不是重新设计样式。
