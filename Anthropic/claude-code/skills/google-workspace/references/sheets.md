<!-- BILINGUAL-EN-ZH -->
# Google Sheets reference / Google Sheets 参考文档

As a first step, before you create a sheet or change one, you must read the spreadsheet design rules below in full. After them, this file covers the Sheets connector (`get_spreadsheet`, `get_values`, `update_values`, `update_formulas`, `insert_dimension`, and `update_spreadsheet`) and how to carry out the design rules with it. Most of the connector behavior here was tested against Google's Sheets API; the rest was seen through the connector, and a few rules have not been checked yet.

作为第一步，在创建或修改电子表格之前，你必须完整阅读下文的电子表格设计规则。在这些规则之后，本文件介绍 Sheets 连接器（`get_spreadsheet`、`get_values`、`update_values`、`update_formulas`、`insert_dimension` 与 `update_spreadsheet`），以及如何用它落实设计规则。此处关于连接器行为的大部分内容都对照 Google 的 Sheets API 做过测试；其余内容是通过连接器观察到的，还有少数规则尚未核实。

# Spreadsheet design / 电子表格设计

When the user asks for something specific, such as tab names, column headers, a formula or a number format, do exactly that, and don't replace it with a design of your own; these rules decide what the user left open. When you edit an existing workbook, its conventions come first (see "Editing an existing workbook"). The rules for wording sheet names, headers, labels and notes are under "Writing in the workbook".

当用户提出明确要求时，例如标签页名称、列标题、某个公式或数字格式，就严格照做，不要换成你自己的设计；这些规则只决定用户未加规定的部分。编辑既有工作簿时，其既有约定优先（见 "Editing an existing workbook"）。关于工作表名称、表头、标签与备注措辞的规则见 "Writing in the workbook"。

A reader judges a spreadsheet by whether its numbers are right and whether they can check them. Build it so that clicking any number shows either a formula they can trace or a labeled input they can change.

读者评判一份电子表格，看的是数字是否正确、以及他们能否自行核验。构建时应做到：点击任何一个数字，看到的要么是一条可追溯的公式，要么是一个带标签、可供修改的输入。

## Formulas, not typed results / 用公式，而不是键入的结果

- **Every derived number is a formula** that references the cells it comes from: totals, averages, ratios, growth rates, lookups. Write `=SUM(B2:B9)`, not a total you worked out yourself.
  - **每一个派生数字都是公式**，并引用其来源单元格：合计、平均、比率、增长率、查找。写 `=SUM(B2:B9)`，而不是你自己算出的总数。
- **This holds when you work in code.** Use code to read, explore and reshape data, but the results you put in the sheet are formulas over the source cells. If the task is "summarize this 5,000-row tab", the summary cells hold `=AVERAGE(Data!B2:B5001)` and `=SUMIFS(...)`, not numbers computed in pandas.
  - **在代码中工作时同样如此。** 可以用代码读取、探索和重塑数据，但放入工作表的结果应当是针对源单元格的公式。如果任务是"汇总这个 5000 行的标签页"，汇总单元格应包含 `=AVERAGE(Data!B2:B5001)` 与 `=SUMIFS(...)`，而不是用 pandas 算出的数字。
- **Typed values are for data brought in from outside**, such as rows from a file, a query or a web page (sorting, filtering or removing duplicates first is fine), and for the inputs listed under "Inputs and hardcoded values". A sum, average or ratio computed in code and pasted in is a hardcoded result.
  - **键入的数值只用于从外部引入的数据**，例如来自文件、查询或网页的行（可以先行排序、过滤或去重），以及 "Inputs and hardcoded values" 所列的输入。在代码中算出再粘贴进来的合计、平均或比率属于硬编码结果。
- **Put statistics and findings in labeled cells.** `=CORREL(B2:B100, C2:C100)` goes in a cell labeled "Correlation, price vs. volume". Any figure you report should be one the reader can find in the file.
  - **把统计结果与结论放进带标签的单元格。** `=CORREL(B2:B100, C2:C100)` 应放在一个标签为 "Correlation, price vs. volume"（价格与成交量相关性）的单元格中。你报告的任何数字都应是读者能在文件中找到的。
- **Reference data that is already in the workbook.** Don't rebuild it elsewhere as a pasted "clean" copy; point formulas at the source.
  - **引用工作簿中已有的数据。** 不要在别处重建一份粘贴出来的"干净"副本；让公式指向源头。
- **Chart data is formulas or references** to the source, not a pasted block of numbers.
  - **图表数据应是指向源数据的公式或引用**，而不是粘贴的一块数字。
- **Formulas cover most analysis:** aggregation (`SUM`, `SUMPRODUCT`), conditions (`SUMIFS`, `COUNTIFS`), lookups (`INDEX`/`MATCH`) and statistics (`CORREL`, `STDEV`, `SLOPE`). Which newer functions Sheets supports is under "Function support" below.
  - **公式能覆盖大部分分析：** 聚合（`SUM`、`SUMPRODUCT`）、条件（`SUMIFS`、`COUNTIFS`）、查找（`INDEX`/`MATCH`）与统计（`CORREL`、`STDEV`、`SLOPE`）。Sheets 支持哪些较新的函数见下文 "Function support"。
- **Syntax basics:** start every formula with `=`. Put text in double quotes (`=IF(A1="Yes",1,0)`); a bare word gives `#NAME?`. Arithmetic on a cell that holds text gives `#VALUE!`.
  - **语法基础：** 每个公式以 `=` 开头。文本放进双引号（`=IF(A1="Yes",1,0)`）；裸词会得到 `#NAME?`。对含文本的单元格做算术会得到 `#VALUE!`。

Before you finish, check that a reader can click any number in the analysis and see how it was derived. Replace any bare value that should be a formula.

收尾之前，检查读者能否点击分析中的任何一个数字并看到它的推导方式。把任何本应是公式的裸数值替换掉。

## Inputs and hardcoded values / 输入与硬编码值

- **Every business assumption lives in its own labeled cell**, and formulas reference it: `=B5*(1+$B$6)` where B6 is labeled "Revenue growth %", never `=B5*1.05`. This covers growth rates, margins, tax rates, multiples and thresholds. A comment explaining a number inside a formula does not fix it; the number still belongs in its own cell.
  - **每一条业务假设都住在自己带标签的单元格里**，公式引用它：`=B5*(1+$B$6)`，其中 B6 标注为 "Revenue growth %"，绝不要写 `=B5*1.05`。这适用于增长率、利润率、税率、倍数与阈值。用注释解释公式里的数字并不能解决问题；这个数字仍然应该放进自己的单元格。
- **Don't:**
  不要做的事：
  - repeat a number that is already in the workbook: `=A1*1.05` when 5% is in an assumptions cell, or `=500000+B2` when 500,000 is in another cell
    重复工作簿中已有的数字：5% 已在假设单元格中却写 `=A1*1.05`，或 500,000 已在另一单元格中却写 `=500000+B2`
  - type a value you calculated, such as 1,050,000 after working out 1,000,000 × 1.05
    键入你自己算出的值，例如算出 1,000,000 × 1.05 后填入 1,050,000
  - copy a number from another sheet into a formula instead of referencing it (`=Inputs!A5`)
    把另一张表中的数字复制进公式，而不是引用它（`=Inputs!A5`）
  - overwrite a formula with a typed value to force a result; fix the input or the logic that feeds it
    用键入值覆盖公式来强行得到某个结果；应当修的是输入或喂给它的逻辑
- **These hardcoded values are fine:**
  以下硬编码值可以接受：
  1. Designated inputs: values in a labeled inputs or assumptions section that formulas reference.
     1. 指定的输入：带标签的输入区或假设区中、被公式引用的值。
  2. True constants in formulas: 12 months a year, 7 days a week, 100 to convert a percentage. No label needed.
     2. 公式中的真正常量：一年 12 个月、一周 7 天、把百分比转换为小数的 100。无需标签。
  3. The first value of a calculated series when there is nothing earlier to reference, such as Year 1 revenue, placed in a labeled section.
     3. 计算数列的首值，且此前没有可引用的内容时，例如第 1 年收入，放在带标签的区域中。
  4. Structural values: row counts in `OFFSET`, sheet index numbers and other values that describe the spreadsheet rather than the business.
     4. 结构性数值：`OFFSET` 中的行数、工作表序号等描述电子表格本身而非业务的值。
  5. Small lookup tables: static reference data in a labeled range, such as tax brackets, that formulas elsewhere reference.
     5. 小型查找表：带标签区域内的静态参考数据，例如税级，供其他公式引用。
- **Say where every hardcoded input came from,** in a cell note or an adjacent cell: `Source: [system or document], [date], [specific reference], [URL if there is one]`, leaving out any part you don't have, for example `Source: Company 10-K, FY2024, page 45, revenue note, [SEC EDGAR URL]`. When the number came from the user, say so: `Source: user-provided assumption`. "Sources and citations" covers data brought in from outside.
  - **说明每个硬编码输入的来源**，写在单元格备注或相邻单元格中：`Source: [system or document], [date], [specific reference], [URL if there is one]`，缺少的部分省略即可，例如 `Source: Company 10-K, FY2024, page 45, revenue note, [SEC EDGAR URL]`。若数字来自用户，就写明：`Source: user-provided assumption`。从外部引入的数据见 "Sources and citations"。
- **Mark the inputs.** In a model you build, or a workbook you create for someone to fill in, put a short legend near the top that says which cells are inputs and how they are marked. A workbook for someone to fill in also gets one example row of realistic values that shows the expected format. Never add an example row to a file you were asked to edit.
  - **标记输入。** 在你构建的模型中，或你为他人填写而创建的工作簿中，在顶部附近放一段简短图例，说明哪些单元格是输入以及它们的标记方式。供他人填写的工作簿还应有一行贴近真实值的示例行，展示期望的格式。绝不要在你受命编辑的文件中添加示例行。

Before you write a value, ask: Is it a business assumption? Put it in a labeled cell. Is it derived? Write the formula. Is it an input with no source in the workbook? Label it and say where it came from.

写入任何一个值之前先问：它是业务假设吗？放进带标签的单元格。它是派生的吗？写公式。它是工作簿中没有来源的输入吗？加上标签并说明来源。

## Formulas a reader can follow / 读者能够看懂的公式

- **Keep each formula short enough to read at a glance.** Split logic with several conditions or lookups into labeled helper cells or columns, so each step can be checked.
  - **让每个公式都短到一眼能读完。** 把含多个条件或查找的逻辑拆进带标签的辅助单元格或辅助列，使每一步都可核查。
  - Good: the tax rate in a labeled helper cell B6, then `=B5*(1-B6)`.
    好：税率放在带标签的辅助单元格 B6 中，然后用 `=B5*(1-B6)`。
  - Bad: `=B5*(1-IF(AND(B3>100000,B4="US"),0.21,IF(B4="UK",0.25,0.15)))`
    差：`=B5*(1-IF(AND(B3>100000,B4="US"),0.21,IF(B4="UK",0.25,0.15)))`
  - Bad: `=SUMPRODUCT((A2:A100="East")*(B2:B100>50)*(C2:C100))/SUMPRODUCT((A2:A100="East")*(B2:B100>50))`
    差：`=SUMPRODUCT((A2:A100="East")*(B2:B100>50)*(C2:C100))/SUMPRODUCT((A2:A100="East")*(B2:B100>50))`
- **Write formulas that can be copied.** `$` locks what must not change when a formula is copied: `$A$1` locks both, `$A1` keeps column A when copied across, `A$1` keeps row 1 when copied down, and `A1` changes both.
  - **写可复制的公式。** `$` 用来锁定复制时不应改变的部分：`$A$1` 两者都锁，`$A1` 横向复制时保持 A 列，`A$1` 纵向复制时保持第 1 行，`A1` 两者都变。
- **Put dimension labels in a header, with one formula that references it.** When you summarize across months, quarters, regions or products, write the labels in a header row or column and one formula that refers to them.
  - **把维度标签放进表头，并用一条公式引用它。** 跨月份、季度、区域或产品汇总时，把标签写在表头行或列，再写一条引用它们的公式。
  - Bad: E2 is `=SUMIFS($D:$D,$B:$B,"Jan")`, F2 is `=SUMIFS($D:$D,$B:$B,"Feb")`, and so on, one hand-edited formula per month.
    差：E2 是 `=SUMIFS($D:$D,$B:$B,"Jan")`，F2 是 `=SUMIFS($D:$D,$B:$B,"Feb")`，依此类推，每个月一条手工修改的公式。
  - Good: E1:P1 hold "Jan" to "Dec"; E2 is `=SUMIFS($D:$D,$B:$B,E$1)`, copied to F2:P2.
    好：E1:P1 放 "Jan" 到 "Dec"；E2 是 `=SUMIFS($D:$D,$B:$B,E$1)`，复制到 F2:P2。
  - If the source has only dates, add a helper column first (`=TEXT(A2,"mmm")`) and reference it, rather than working out the period inside the `SUMIFS` criteria.
    如果源数据只有日期，先加一个辅助列（`=TEXT(A2,"mmm")`）并引用它，而不是在 `SUMIFS` 条件里现算期间。
  - If you can't copy a formula along its row without editing it, the literal in it belongs in a header cell.
    如果一条公式沿行复制时必须手工修改，那么其中的字面量就该放进表头单元格。
- **Use the same formula in every period of a row.** One edited cell in the middle of a row is the most common error that produces no error value.
  - 一行中的每个期间都使用同一条公式。行中间某处一条被单独改动的公式，是最常见的、不会产生错误值的错误。
- **Guard a division whose denominator can be zero:** `=IF(C5=0,0,B5/C5)`, which the zero format in "Financial models" shows as a dash. Use `IF` here rather than `IFERROR`, which also hides a broken reference.
  - **为分母可能为零的除法加保护：** `=IF(C5=0,0,B5/C5)`，"Financial models" 中的零值格式会把它显示为短横线。此处用 `IF` 而不是 `IFERROR`，后者还会把失效的引用一并隐藏。
- **Cross-sheet references use `!`:** `=Assumptions!B5`. Quote a sheet name that contains spaces or symbols, and double any apostrophe in it: `='Q3 Forecast'!B7`, `='Owner''s tab'!A1`. `=Assumptions.B5` (LibreOffice's notation), `=@Assumptions!B5` and `[Book2]Sheet1!A1` (Excel's reference to another file) are not valid in Sheets.
  - **跨表引用使用 `!`：** `=Assumptions!B5`。含空格或符号的表名要加引号，名称中的单引号要双写：`='Q3 Forecast'!B7`、`='Owner''s tab'!A1`。`=Assumptions.B5`（LibreOffice 记法）、`=@Assumptions!B5` 与 `[Book2]Sheet1!A1`（Excel 对另一文件的引用）在 Sheets 中无效。

## Data that grows / 会增长的数据

- **Formulas over a log that users will add rows to must include the new rows.** This applies to transaction lists, timesheets and any append-only records. A fixed range that covers today's rows, such as `=SUM(B2:B128)`, or `J2:J1001` for a 1,000-row table, leaves out the next row added. Decide first whether the data is a growing log or a fixed snapshot; a snapshot can use fixed ranges.
  - **针对用户会不断追加行的日志类数据的公式，必须把新增行包括进来。** 这适用于交易清单、工时表以及任何只追加的记录。覆盖今天已有行的固定区间，例如 `=SUM(B2:B128)`，或 1000 行表的 `J2:J1001`，会漏掉下一行新增。先判断数据是持续增长的日志还是固定快照；快照可以使用固定区间。
- **Use open-ended ranges** such as `=SUM(B2:B)`. A Sheets table (Format > Convert to table) also accepts `=SUM(Sales[Amount])`, but `[@Amount]`, `[[#Totals],[Amount]]` and the `@` operator are syntax errors in Sheets. For a row-by-row calculation, anchor the columns and fill the formula down (`=$B2*$C2`). A named range does not grow, so don't use one for growing data. `ARRAYFORMULA` can calculate a whole growing column in one formula; guard it against blank rows, as in `=ARRAYFORMULA(IF(LEN(B2:B), B2:B*C2:C, ))`, where the empty last argument leaves unused rows blank (`""` would be counted by `COUNTA`). `ARRAYFORMULA` exists only in Sheets, so if the file may go to Excel, fill a plain formula down instead.
  - **使用开放式区间**，例如 `=SUM(B2:B)`。Sheets 表格（Format > Convert to table）也接受 `=SUM(Sales[Amount])`，但 `[@Amount]`、`[[#Totals],[Amount]]` 与 `@` 运算符在 Sheets 中是语法错误。逐行计算时，锚定列并向下填充公式（`=$B2*$C2`）。命名区间不会增长，因此不要对增长数据使用命名区间。`ARRAYFORMULA` 可以用一条公式计算整个增长的列；要防范空白行，如 `=ARRAYFORMULA(IF(LEN(B2:B), B2:B*C2:C, ))`，其中留空的最后一个参数让未使用的行保持空白（`""` 会被 `COUNTA` 计入）。`ARRAYFORMULA` 仅存在于 Sheets，因此如果文件可能进入 Excel，请改为向下填充普通公式。
- **Band rows with something that extends to new rows:** alternating colors (Format > Alternating colors) over a range that runs past the last row, or a conditional format such as `=MOD(ROW(),2)=0` over the columns, reaching past the last row. Colors filled onto alternate rows by hand don't extend to new rows.
  - **用能延伸到新增行的机制为行加条纹：** 在超出最后一行的区间上使用交替颜色（Format > Alternating colors），或在各列上使用能延伸超出最后一行的条件格式如 `=MOD(ROW(),2)=0`。手工涂在隔行上的颜色不会延伸到新增行。

## Layout / 版面

- **Decide one style for a multi-sheet build** before you start: header fill, fonts, column widths and table style. Apply it the same way on every sheet, and check every sheet against it before you finish. Styling each sheet separately produces sheets that don't match each other.
  - **多表构建开工前先定一种样式：** 表头填充、字体、列宽与表格样式。在每张表上以相同方式应用，收尾前逐表对照检查。逐表单独设定样式会产出彼此不协调的表。
- **Use one professional font throughout,** such as Arial or Times New Roman, unless the user asks for another.
  - **全程使用一种专业字体**，例如 Arial 或 Times New Roman，除非用户要求其他字体。
- **Write cell text in the user's language and regional spelling:** headers, labels, notes and chart titles. If the workbook already follows a different variety, match the workbook. Never mix varieties in one workbook.
  - **单元格文本使用用户的语言与地区拼写：** 表头、标签、备注与图表标题。如果工作簿已经遵循另一种变体，则与工作簿保持一致。绝不在同一工作簿中混用变体。
- **Column widths:** size the row-label columns so labels are not cut off. Merge and center a header that sits over a group of columns rather than widening a column to fit it. In financial models, keep the number columns one uniform width, and indent with an extra narrow column rather than by varying widths.
  - **列宽：** 把行标签列调宽到标签不被截断。跨一组列的表头用合并居中处理，而不是把某一列加宽去迁就它。在财务模型中，数字列保持统一宽度，需要缩进时用一个额外的窄列，而不是靠改变列宽实现。
- **Group rows and columns instead of hiding them.** Don't hide rows or columns unless the user asks. A group shows a +/- control that tells the reader something is there; hidden rows are easy to miss and lead to mistakes. Before you collapse a group, check for charts placed over those rows or built from them: collapsing the rows hides the chart too. Keep a chart's data where collapsing detail rows won't affect it, such as a separate area or sheet.
  - **对行和列使用分组，而不是隐藏。** 除非用户要求，不要隐藏行或列。分组会显示一个 +/- 控件，告诉读者这里有内容；隐藏的行容易被忽略并导致错误。折叠分组之前，检查是否有图表覆盖在这些行上或由这些行构建：折叠行会连带隐藏图表。把图表的数据放在折叠明细行不会影响到的位置，例如独立的区域或工作表。

## Writing in the workbook / 在工作簿中写作

Text you write in the workbook (sheet names, headers, labels, notes, comments, text cells, chart titles) should read as though a person wrote it. When readers think something was written by AI, they judge it as sloppy and stop trusting it, whatever the content. They make that judgment from a set of common indicators, listed below, so take extra care to keep them out of your writing. These rules are for text you write, and a style the user or their style guide asks for takes priority. Do not rewrite the user's existing text to follow them unless the user asks you to.

写入工作簿的文本（工作表名称、表头、标签、备注、批注、文本单元格、图表标题）读起来应当像出自真人之手。当读者认为某段文字是 AI 写的，无论内容如何，他们都会视之为粗制滥造并不再信任它。他们依据一组常见特征做出这种判断，特征列举如下，因此要格外注意不让它们出现在你的文字里。这些规则针对你写的文本；用户或其风格指南要求的样式优先。除非用户要求，否则不要为了遵守这些规则而改写用户的既有文本。

【评论】这一节以"防止输出被识别为机器写作"为目标，列举了破折号滥用、"恰好三项"列表、空泛形容词等英文语料中常见的 LLM 文风特征，属于面向文风真实性的约束。

- Say what is true without first denying something else. "Revenue grew 12%, three times the US rate," not "This isn't a growth story, it's a market-share story." Do not open a note with "Here's the thing" or "The real story is".
  - 直接陈述事实，不要先否定别的说法。写 "Revenue grew 12%, three times the US rate,"，而不是 "This isn't a growth story, it's a market-share story."。不要以 "Here's the thing" 或 "The real story is" 开头写备注。
- Match the number of bullets, examples, and adjectives to the content, not to a default of three. Two drivers get two bullets; five get five. A list of exactly three ("fast, reliable, and scalable") usually means the third item was added for rhythm, not because there were three things to say, and readers read it as filler.
  - 要点、示例与形容词的数量与内容匹配，而不是默认凑成三个。两个驱动因素就写两条要点；五个就写五条。恰好三条的列表（"fast, reliable, and scalable"）通常意味着第三项是为了节奏感加上去的，而不是真有第三件事要说，读者会把它读成凑数。
- Do not use a metaphor where a literal word will do. If a plain description exists ("the same construction," "the same pattern," "slowed," "fell"), use it. Metaphors are for when the literal version would be longer or less precise, which is rare in analytical writing. Test: if the metaphor can be replaced by a plain word without losing meaning, replace it. A metaphor makes the reader translate it back into the plain claim and carries meaning you did not choose. The ones that appear most are "north star", "move the needle", "double-click" (meaning look closer), "unpack", "journey", and "landscape" (meaning a market). Common ones in business writing: "moat", "headwind", "drag" (meaning a cost on results), "safety net", "clears the bar", and "land" meaning finish or total ("lands $4.4k under budget"). Examples:
  - 能用直白字眼时就不要用比喻。如果存在平实的说法（"the same construction,"、"the same pattern,"、"slowed,"、"fell"），就用它。比喻只用于直白表述会更冗长或不更精确的场合，而这在分析性写作中很少见。检验方法：如果一个比喻可以换成平实字眼而不损失含义，就换掉。比喻迫使读者把它翻译回平实的主张，并夹带你未曾选择的含义。出现最多的是 "north star"、"move the needle"、"double-click"（意指细看）、"unpack"、"journey" 与 "landscape"（意指市场）。商业写作中常见的还有："moat"、"headwind"、"drag"（意指对结果的成本拖累）、"safety net"、"clears the bar"，以及表示完成或合计的 "land"（"lands $4.4k under budget"）。示例：
  - Bad: "The ones it has are the same species." Good: "The ones it has follow the same pattern."
    差："The ones it has are the same species." 好："The ones it has follow the same pattern."
  - Bad: "A coordinated digestion pause would be visible immediately." Good: "If several large customers cut capex in the same quarter, it would show up in the next guide."
    差："A coordinated digestion pause would be visible immediately." 好："If several large customers cut capex in the same quarter, it would show up in the next guide."
  - Bad: "Architecture transitions compressed margin on the way in and expanded it on the way out." Good: "Gross margin fell during the Hopper-to-Blackwell ramp and recovered once Blackwell shipped at volume."
    差："Architecture transitions compressed margin on the way in and expanded it on the way out." 好："Gross margin fell during the Hopper-to-Blackwell ramp and recovered once Blackwell shipped at volume."
  - Bad: "The lever that unlocks growth." Good: "The pricing change is what makes the target reachable."
    差："The lever that unlocks growth." 好："The pricing change is what makes the target reachable."
- Cut words that claim importance without giving evidence: "genuinely", "truly", "actually", "clearly", "significantly", "robust", "leverage", "delve", "actionable insights", "learnings". Where one of them stood in for a fact, put the fact there ("margins fell 4 points", not "margins fell significantly"); otherwise delete it. When "leverage" or "significant" carries its financial or statistical meaning ("net leverage", "statistically significant"), it is a literal term; keep it.
  - 删掉只宣称重要性却不出示证据的词："genuinely"、"truly"、"actually"、"clearly"、"significantly"、"robust"、"leverage"、"delve"、"actionable insights"、"learnings"。如果某个词原本顶替着一个事实，就把事实放回去（写 "margins fell 4 points"，而不是 "margins fell significantly"）；否则直接删除。当 "leverage" 或 "significant" 承载其金融或统计含义（"net leverage"、"statistically significant"）时，它是字面术语，保留。
- Use full stops and commas, and a colon before a list. No emoji in the workbook.
  - 使用句号和逗号，列表前用冒号。工作簿中不用表情符号。
- Use em dashes sparingly: at most one in a paragraph, and none in a heading, a title, or between a bold label and the text after it. Several em dashes in one paragraph is one of the first things readers use to spot machine writing. In place of one, use a comma, a colon, parentheses, or a new sentence, not an en dash or a spaced hyphen.
  - 少用破折号（em dash）：一段最多一个，标题、小标题中不用，粗体标签与其后文字之间也不用。一段中出现多个破折号是读者识别机器写作的首要线索之一。替代方案：逗号、冒号、括号或另起一句，而不是 en dash 或带空格的连字符。
- Sheet names and headers name the contents ("Revenue ($mm)", "Assumptions"), not how you made them ("New calc", "Fixed"). A line of text that sums up an analysis, such as the headline on a summary sheet, states the finding ("Europe missed plan", not "Analysis results").
  - 工作表名称与表头描述内容（"Revenue ($mm)"、"Assumptions"），而不是描述制作过程（"New calc"、"Fixed"）。概括分析的整行文字（如汇总表的标题行）应陈述结论（"Europe missed plan"，而不是 "Analysis results"）。
- Keep the conversation out of the workbook: do not name a tab "Summary (Revised)" or write "Updated per your feedback" in a cell. If two versions stay, name each for what it holds. Whoever opens the workbook next did not see the request, so those lines mean nothing to them. Say what changed in your reply, not in the workbook. Source notes and the Data Sources tab record where data came from; they are not conversation, so keep writing them where "Sources and citations" asks for them.
  - 不要把对话写进工作簿：不要把标签页命名为 "Summary (Revised)"，也不要在单元格里写 "Updated per your feedback"。如果两个版本并存，按各自内容命名。下一个打开工作簿的人没有看到那些请求，这些话对他们毫无意义。变更说明写在你的回复里，而不是工作簿里。来源备注与 Data Sources 标签页记录数据来自哪里，它们不是对话，因此仍按 "Sources and citations" 的要求书写。

## Financial models / 财务模型

Use these unless the user or the existing file does something else.

除非用户或既有文件另有做法，否则使用以下约定。

**Text colors:**
**文本颜色：**
- Blue (`#0000FF`): hardcoded inputs, and numbers users will change for scenarios.
  - 蓝色（`#0000FF`）：硬编码输入，以及用户会为情景分析而修改的数字。
- Black (`#000000`): all formulas and calculations.
  - 黑色（`#000000`）：所有公式与计算。
- Green (`#008000`): links to other sheets in the same workbook.
  - 绿色（`#008000`）：指向同一工作簿内其他工作表的链接。
- Red (`#FF0000`): links to other files.
  - 红色（`#FF0000`）：指向其他文件的链接。
- Yellow fill (`#FFFF00`): key assumptions that need attention, and cells the user should fill in or update.
  - 黄色填充（`#FFFF00`）：需要关注的关键假设，以及用户应填写或更新的单元格。

**Number formats:**
**数字格式：**
- Years are text: "2024", not 2,024.
  - 年份是文本："2024"，而不是 2,024。
- Currency is `$#,##0;($#,##0);"-"`, with the unit in the header: "Revenue ($mm)".
  - 货币格式为 `$#,##0;($#,##0);"-"`，单位写在表头："Revenue ($mm)"。
- Negative numbers go in parentheses: (123), not -123.
  - 负数放括号里：(123)，而不是 -123。
- Zeros show as a dash, percentages included (`0.0%;(0.0%);"-"`).
  - 零显示为短横线，百分比也一样（`0.0%;(0.0%);"-"`）。
- Percentages use one decimal place (`0.0%`) and are stored as fractions: 0.15 shows as 15.0%, while 15 would show as 1500.0%.
  - 百分比保留一位小数（`0.0%`）并以小数存储：0.15 显示为 15.0%，而 15 会显示为 1500.0%。
- Valuation multiples such as EV/EBITDA and P/E use `0.0"x"`, which shows 12.5x.
  - EV/EBITDA、P/E 等估值倍数使用 `0.0"x"`，显示为 12.5x。

**Sensitivity tables:**
**敏感性分析表：**
- Give the grid an odd number of rows and columns, such as 5×5 or 7×7, so the base case lands in the center cell, and highlight that cell (a yellow fill, for example). A WACC against terminal growth table should put the current WACC in the middle row and the current growth rate in the middle column.
  - 让网格的行数与列数为奇数，例如 5×5 或 7×7，使基准情形落在中心单元格，并高亮该单元格（例如用黄色填充）。WACC 对 terminal growth 的敏感性表应把当前 WACC 放在中间行、当前增长率放在中间列。
- Build every cell of the grid as a formula that recalculates the output from its row's and its column's input values. For a DCF, each cell discounts the same cash flows at its row's rate and adds a terminal value at its column's growth rate.
  - 网格的每个单元格都构建为一条公式，由其所在行与所在列的输入值重新计算输出。对 DCF 而言，每个单元格按其所在行的折现率对同一组现金流折现，并按其所在列的增长率加上终值。

## Charts / 图表

- **Lay chart data out as one block.** Headers in the first row become series names, and categories in the first column become the axis labels:
  - **把图表数据排成一个整块。** 第一行的表头成为系列名，第一列的类别成为坐标轴标签：

  |       | Q1  | Q2  | Q3  | Q4  |
  |-------|-----|-----|-----|-----|
  | North | 100 | 120 | 110 | 130 |
  | South | 90  | 95  | 100 | 105 |

  |      | Q1  | Q2  | Q3  | Q4  |
  |------|-----|-----|-----|-----|
  | 北区 | 100 | 120 | 110 | 130 |
  | 南区 | 90  | 95  | 100 | 105 |

- **Some chart types need a particular layout.** A pie or doughnut chart takes one column of values with labels. A scatter chart takes X values in the first column and Y values in the others. A candlestick chart takes Low, Open, Close, High.
  - **有些图表类型需要特定布局。** 饼图或环形图取一列带标签的数值。散点图第一列放 X 值，其余列放 Y 值。K 线图（candlestick）按 Low、Open、Close、High 排列。
- **Summarize before you chart.** To chart raw rows that need aggregating, build a summary first, either a pivot table or a table of `SUMIFS` formulas, and chart the summary range. A chart built on a pivot table follows the pivot table, so change the pivot table rather than the chart.
  - **先汇总再作图。** 要为需要聚合的原始行作图时，先建一个汇总，可以是数据透视表，也可以是 `SUMIFS` 公式表，然后对汇总区间作图。基于透视表的图表会跟随透视表，因此要改就改透视表，而不是图表。
- **Group dates into periods with a helper column.** When the user wants totals by month, quarter or year and the data has daily dates, add a column that turns each date into its period, such as `=EOMONTH(A2,-1)+1` for the first day of the month (format the column as a date, or it shows a serial number such as 45413) or `=YEAR(A2)&"-Q"&ROUNDUP(MONTH(A2)/3,0)` for the quarter. Give it a header, fill it for every row, and group by it instead of by the raw dates.
  - **用辅助列把日期归入期间。** 当用户想要按月、季度或年汇总而数据是逐日日期时，加一列把每个日期转成其所属期间，例如取当月第一天的 `=EOMONTH(A2,-1)+1`（把该列格式设为日期，否则会显示 45413 这类序列号），或取季度的 `=YEAR(A2)&"-Q"&ROUNDUP(MONTH(A2)/3,0)`。给它一个表头，为每一行填充，并按它而不是原始日期分组。

## Sources and citations / 来源与引用

Every value that comes into the workbook from outside it should be traceable without asking you: where it came from, how it reached you, and when it was pulled.

从工作簿外部进入工作簿的每一个值，都应能在不询问你的情况下被追溯：它来自哪里、如何到达你、何时抓取。

- **These need no source note:** data rows the user types or dictates with no system behind them, and the existing contents of a workbook the user gave you to edit. An assumption the user gave you still gets its note, as "Inputs and hardcoded values" says.
  - **这些无需来源备注：** 用户亲手键入或口述、背后没有系统的数据行，以及用户交给你编辑的工作簿的既有内容。用户给出的假设仍要按 "Inputs and hardcoded values" 所述加备注。
- **A figure you looked up on its own** (on a web page, or in a document or dashboard) and placed in your own layout gets a note on the value cell itself, not on its row label or header. If A8 is "Cash and cash equivalents" and B8 is $179,172, the note goes on B8. Use the source format from "Inputs and hardcoded values", with the URL of the page you actually read the figure from, not the index page you started at: `Source: Apple Investor Relations, https://investor.apple.com/sec-filings/annual-reports/2024`. This includes figures you found earlier in the conversation.
  - **你单独查到的一个数字**（在网页、文档或仪表板中查得）并放进自己的版式时，备注写在数值单元格本身上，而不是其行标签或表头上。如果 A8 是 "Cash and cash equivalents"、B8 是 $179,172，备注应写在 B8 上。使用 "Inputs and hardcoded values" 中的来源格式，URL 用你真正读取该数字的页面，而不是你出发的索引页：`Source: Apple Investor Relations, https://investor.apple.com/sec-filings/annual-reports/2024`。这包括你在对话中早先找到的数字。
- **A table you bring in whole** gets exactly one note, on its top-left header cell (C4 for a table at C4:F20), and no notes on the value cells or the other headers. This covers the rows of an uploaded file, a query result, query results the user pasted, and a table you transcribe from a document, such as a financial statement from a PDF.
  - **整体引入的一张表**只写一条备注，位于其左上角表头单元格（表在 C4:F20 时为 C4），数值单元格与其他表头不写备注。这涵盖上传文件的行、查询结果、用户粘贴的查询结果，以及你从文档转录的表，例如 PDF 中的财务报表。
  - For an uploaded file or document: `Source: [file name], [page or table, if it applies], uploaded [date]`.
    - 上传的文件或文档：`Source: [file name], [page or table, if it applies], uploaded [date]`。
  - For a query result: `Source: [system] ([object]), [N] rows, run [date, time and time zone]. [How obtained]. Full query on the Data Sources tab.` N counts data rows, not the header. "How obtained" is "Run by Claude" when you ran the query, or "Pasted into chat by user; not run by Claude" when the user supplied the results. Never word a note as if you ran a query whose results were given to you.
    - 查询结果：`Source: [system] ([object]), [N] rows, run [date, time and time zone]. [How obtained]. Full query on the Data Sources tab.`。N 计的是数据行，不含表头。"How obtained" 在你运行查询时写 "Run by Claude"，在用户提供结果时写 "Pasted into chat by user; not run by Claude"。绝不要把备注写成你运行过一个结果其实是别人提供的查询。
- **Query results also get a row on a "Data Sources" tab,** one row per result set. Make it the last tab and keep it visible. Its columns, in order: Written to (Sheet!Range, including the header row) | Source | Object (database.schema.table, endpoint, URL or file name) | Obtained via | Query or request (the exact query, or the user's request in their own words if no query was shown; never invent one) | Parameters | Run at | Requested by (the person who asked, if you know their name) | Row count | Changes after retrieval (none, or what you did: sorted, filtered, pivoted, converted units) | Notes. If you run a query again, add a new row and write "superseded" in the old row's Notes instead of overwriting it. If you remove a table, write "removed" in its row's Notes.
  - **查询结果还要在 "Data Sources" 标签页上占一行**，每个结果集一行。把它作为最后一个标签页并保持可见。其列依序为：Written to（Sheet!Range，含表头行）| Source | Object（database.schema.table、端点、URL 或文件名）| Obtained via | Query or request（确切的查询语句，或未展示查询时用户原话所述请求；绝不要编造）| Parameters | Run at | Requested by（提问者，若知道其姓名）| Row count | Changes after retrieval（无，或你所做的处理：排序、过滤、透视、单位换算）| Notes。若再次运行同一查询，新增一行并在旧行的 Notes 写 "superseded"，而不是覆盖它。若删除一张表，在其行的 Notes 写 "removed"。
- **Run at** is when the data was pulled, as the tool or the user reports it. Turn a relative day such as "yesterday" into a date, and type it as text; never use `NOW()` or `TODAY()`, which change every time the file opens.
  - **Run at** 是数据被抓取的时间，以工具或用户的报告为准。把 "yesterday" 这类相对日期转换为具体日期并以文本输入；绝不要使用 `NOW()` 或 `TODAY()`，它们在每次打开文件时都会变化。
- **Keep secrets and personal identifiers out of the audit record.** Credentials, API tokens, passwords and connection strings never go anywhere in the workbook. In the query text, the notes and the Data Sources tab, replace personal identifiers that appear as literal values with [REDACTED]: government ID numbers, full bank-account or payment-card numbers, the name or date of birth of a private individual, home addresses, and personal email addresses or phone numbers. Keep table and column names, company and product names, internal record keys and non-identifying filter values, so the query can still be read and run again. `WHERE ssn = '123-45-6789' AND region = 'EMEA'` becomes `WHERE ssn = '[REDACTED]' AND region = 'EMEA'`. The returned rows go into the sheet as the user asked; this rule is about not copying identifiers into the record a second time.
  - **把机密与个人身份信息挡在审计记录之外。** 凭据、API token、密码与连接字符串绝不出现在工作簿的任何位置。在查询文本、备注与 Data Sources 标签页中，把作为字面值出现的个人身份信息替换为 [REDACTED]：政府证件号、完整银行账号或支付卡号、私人个体的姓名或出生日期、家庭住址、个人邮箱或电话号码。保留表名与列名、公司与产品名、内部记录键以及不具识别性的过滤值，使查询仍可被阅读和重跑。`WHERE ssn = '123-45-6789' AND region = 'EMEA'` 变成 `WHERE ssn = '[REDACTED]' AND region = 'EMEA'`。返回的行照用户要求写入工作表；本规则针对的是不要把身份信息第二次复制进记录。

【评论】该节要求把个人身份信息在审计记录中替换为 [REDACTED]，同时保留查询的可读性与可重跑性，是在数据合规与可复现性之间做的折中。

## Editing an existing workbook / 编辑既有工作簿

- **Its conventions override these rules:** colors, number formats, fonts, layout and tab order. Find its input cells first (a distinct font color, fill or shading usually marks them), write only there unless the request needs more, and leave its existing formulas alone unless the user asks you to change them.
  - **其既有约定优先于这些规则：** 颜色、数字格式、字体、版面与标签页顺序。先找到它的输入单元格（通常以特别的字体颜色、填充或底纹标记），除非请求需要，否则只写在输入单元格中，且不动其既有公式，除非用户要求修改。
- **Keep its formatting.** Format a new row like the row above it, and a new column like the one beside it.
  - **保持其格式。** 新行的格式照上一行，新列的格式照相邻列。
- **Match the number format of what you summarize.** A total under a currency column gets the same currency format. If the new cell is unformatted, set the format rather than leaving a bare number.
  - **与你汇总的对象数字格式一致。** 货币列下方的合计用同样的货币格式。如果新单元格没有格式，就设置格式，而不是留下裸数字。
- **After inserting rows inside a summarized range,** check that the totals and averages include them. Rows inserted just below the last summed row or above the first are often left out.
  - **在汇总区间内部插入行之后，** 检查合计与平均是否把它们包含了进去。插在最后一个被求和行下方或第一行上方的行经常被漏掉。
- **Inserted rows and columns take their neighbors' formatting.** Rows inserted under a blue header row come out blue. Check the new cells and clear formatting that doesn't belong.
  - **插入的行和列会继承邻近单元格的格式。** 蓝色表头行下方插入的行会变成蓝色。检查新单元格并清除不属于那里的格式。

## Checking the result / 检查结果

- **Check what the reader will see.** Number formats and locale decide what a cell displays, so for formatting work check the displayed text, not only the stored value.
  - **检查读者将看到的内容。** 数字格式与区域设置决定单元格的显示，因此涉及格式的工作要检查显示文本，而不只是存储值。
- **No error values:** no `#VALUE!`, `#REF!`, `#NAME?`, `#DIV/0!` or `#N/A`, no circular references, and no range that stops short of the data.
  - **无错误值：** 没有 `#VALUE!`、`#REF!`、`#NAME?`、`#DIV/0!` 或 `#N/A`，没有循环引用，也没有停在数据中途的区间。
- **No error values does not mean correct.** Look for hardcoded numbers where a formula belongs, and for formulas that point at the wrong row but happen to give the right value today.
  - **没有错误值不等于正确。** 要查找本应是公式的硬编码数字，以及指向错误行、只是碰巧在今天给出正确值的公式。
- **Check every sheet.** List the sheets the workbook actually has, rather than the ones you remember creating, and check that each has its content. Finish or delete any empty tab you added.
  - **检查每一张表。** 列出工作簿实际拥有的表，而不是你记得创建的那些，并确认每张表都有其内容。把你添加的空表补齐或删除。
- **Check the sources.** Every figure from outside the workbook has the source note "Sources and citations" asks for.
  - **检查来源。** 来自工作簿之外的每个数字都有 "Sources and citations" 所要求的来源备注。
- **Check the formatting** against the user's request and the conventions above.
  - **检查格式**是否对照用户要求与上述约定。

# Sheets connector / Sheets 连接器

## Applying the design rules / 落实设计规则

- **Colors:** a `sheets_helper.py format` rule takes the hex values from the design rules above in `fg` and `bg`, such as `{"range": "B2:B8", "fg": "#0000FF"}` for inputs.
  - **颜色：** `sheets_helper.py format` 规则从上述设计规则取得十六进制值放入 `fg` 与 `bg`，例如输入用 `{"range": "B2:B8", "fg": "#0000FF"}`。
- **Number formats:** the helper's `currency` is the `$#,##0;($#,##0);"-"` from the design rules above and its `multiple` is `0.0"x"`. Its `percent` is plain `0.0%`, which shows zero as 0.0%; in a financial model, pass the pattern `0.0%;(0.0%);"-"` instead. *Untested through the connector.*
  - **数字格式：** 助手的 `currency` 即上述设计规则中的 `$#,##0;($#,##0);"-"`，其 `multiple` 为 `0.0"x"`。它的 `percent` 是普通 `0.0%`，会把零显示为 0.0%；在财务模型中，改传模式 `0.0%;(0.0%);"-"`。*未经连接器实测。*
- **Years as text:** write `'2024` with a leading apostrophe. Without it, both write tools store the number 2024.
  - **年份作为文本：** 写 `'2024`，带一个前导单引号。没有它，两个写入工具都会存成数字 2024。
- **Source notes:** an `updateCells` request sets a cell's note: `{"updateCells": {"rows": [{"values": [{"note": "Source: ..."}]}], "fields": "note", "start": {"sheetId": 0, "rowIndex": 3, "columnIndex": 2}}}` puts it on C4. *Untested through the connector.*
  - **来源备注：** `updateCells` 请求设置单元格备注：`{"updateCells": {"rows": [{"values": [{"note": "Source: ..."}]}], "fields": "note", "start": {"sheetId": 0, "rowIndex": 3, "columnIndex": 2}}}` 会把备注放到 C4。*未经连接器实测。*
- **Grouping rows or columns:** `{"addDimensionGroup": {"range": {"sheetId": 0, "dimension": "ROWS", "startIndex": 4, "endIndex": 12}}}` groups rows 5 to 12; the indexes are 0-based with an exclusive end, like grid ranges. *Untested through the connector.*
  - **行或列分组：** `{"addDimensionGroup": {"range": {"sheetId": 0, "dimension": "ROWS", "startIndex": 4, "endIndex": 12}}}` 对第 5 至 12 行分组；索引从 0 开始、结束索引不含端点，与网格区间一致。*未经连接器实测。*
- **Banding:** `addBanding` with a `bandedRange` whose `rowProperties` set `headerColor`, `firstBandColor` and `secondBandColor`. The banding extends to rows added inside the range, so make the range run past the last row of data. *Untested through the connector.*
  - **加条纹：** `addBanding`，其 `bandedRange` 的 `rowProperties` 设置 `headerColor`、`firstBandColor` 与 `secondBandColor`。条纹会延伸到区间内新增的行，因此让区间超出最后一行数据。*未经连接器实测。*

## Create / 创建

- **Blank sheet, then fill it, in four calls.** Use this for models and anything with formulas.
  - **先建空白表，再用四次调用填满它。** 模型以及一切含公式的内容都走这条路。
  1. Drive `create_file` with `contentMimeType: "application/vnd.google-apps.spreadsheet"`. The first tab of a new sheet is `Sheet1` with `sheetId` `0`, so there is nothing to read yet.
     1. 用 `contentMimeType: "application/vnd.google-apps.spreadsheet"` 驱动 `create_file`。新表的第一个标签页是 `sheetId` 为 `0` 的 `Sheet1`，此时还没有任何内容可读。
  2. One `update_formulas` call writes the whole table from its top-left cell: title, headers, labels, numbers and formulas in one 2D array, with `null` for cells left empty. Both write tools parse input the same way, so don't send the numbers and the formulas in separate calls.
     2. 一次 `update_formulas` 调用从左上角单元格写入整张表：标题、表头、标签、数字与公式放进一个二维数组，留空的单元格用 `null`。两个写入工具以相同方式解析输入，因此不要把数字与公式拆成多次调用。
  3. One `update_spreadsheet` call does everything else: the tab rename (`updateSheetProperties` with `fields: "title"`), formatting, frozen rows, merges, conditional formats, and widths for text columns (the format spec's `col_widths` key, or an `autoResizeDimensions` request) so labels aren't cut off. `sheets_helper.py format spec.json --sheet-id 0` builds the formatting without a metadata read; append the other requests to its output.
     3. 一次 `update_spreadsheet` 调用完成其余一切：重命名标签页（`updateSheetProperties` 配 `fields: "title"`）、格式、冻结行、合并、条件格式，以及文本列的列宽（格式 spec 的 `col_widths` 键，或 `autoResizeDimensions` 请求），以免标签被截断。`sheets_helper.py format spec.json --sheet-id 0` 无需读取元数据即可构建格式；把其余请求附加到它的输出之后。
  4. One `get_values` over the table to verify.
     4. 对整张表做一次 `get_values` 验证。
- **Upload:** upload a CSV or an .xlsx built with the `xlsx` skill, and Drive converts it. Then edit the result in place. The file's whole content goes inside the tool call, so an upload is slow and often fails: a 20 KB encoded file has taken about three minutes. Keep uploads small; for a large data set, create a blank sheet and write the data with `update_values`.
  - **上传：** 上传 CSV 或用 `xlsx` 技能构建的 .xlsx，由 Drive 完成转换，然后就地编辑结果。文件的全部内容都要放进工具调用，因此上传很慢且经常失败：一个 20 KB 的编码文件曾耗时约三分钟。保持上传体积小；对大数据集，先创建空白表再用 `update_values` 写入数据。

## How a sheet is addressed / 工作表的寻址方式

There are two coordinate systems, and each tool uses one of them.

这里有两套坐标系统，每个工具使用其中一套。

| Tools | Addresses cells by | Example |
|---|---|---|
| `get_values`, `update_values`, `update_formulas` | A1 notation with the tab name | `'P&L Model'!B4:D10` |
| `update_spreadsheet`, `insert_dimension` | Numeric `sheetId` plus 0-based indexes; end indexes are exclusive | B4:D10 on tab 1001 is `{"sheetId": 1001, "startRowIndex": 3, "endRowIndex": 10, "startColumnIndex": 1, "endColumnIndex": 4}` |

| 工具 | 单元格寻址方式 | 示例 |
|---|---|---|
| `get_values`、`update_values`、`update_formulas` | A1 记法加标签页名称 | `'P&L Model'!B4:D10` |
| `update_spreadsheet`、`insert_dimension` | 数字 `sheetId` 加 0 起始索引；结束索引不含端点 | 标签页 1001 上的 B4:D10 即 `{"sheetId": 1001, "startRowIndex": 3, "endRowIndex": 10, "startColumnIndex": 1, "endColumnIndex": 4}` |

- **Quote tab names** that contain spaces or symbols, and double any apostrophe inside: `'P&L Model'!F5`, `'Owner''s tab'!A1`.
  - **给含空格或符号的标签页名称加引号**，名称中的单引号双写：`'P&L Model'!F5`、`'Owner''s tab'!A1`。
- **The `sheetId` is not the tab name or its position.** Get it from `get_spreadsheet` (see Read). To choose a new tab's ID yourself, set `sheetId` in `addSheet`; the reply confirms it.
  - **`sheetId` 既不是标签页名称也不是其位置。** 从 `get_spreadsheet` 获取（见 Read）。要自己指定新标签页的 ID，在 `addSheet` 中设置 `sheetId`；回复会予以确认。
- **Leave the math to the helper.** `sheets_helper.py range` converts A1 to a grid range, including open ranges like `C:C`, and resolves tab names:
  - **把换算交给助手脚本。** `sheets_helper.py range` 把 A1 转成网格区间，包括 `C:C` 这类开放区间，并解析标签页名称：

```
python <skill>/scripts/sheets_helper.py range "'P&L Model'!B4:D10" --meta meta.json
{"sheetId": 1001, "startRowIndex": 3, "endRowIndex": 10, "startColumnIndex": 1, "endColumnIndex": 4}
```

`meta.json` is the (small) output of the metadata read below, written to a file.

`meta.json` 是下文元数据读取的（体积很小的）输出，写入文件保存。

## Read / 读取

- **Tabs and IDs:** `get_spreadsheet` with `fields: ["sheets.properties"]`. It returns each tab's `title`, `sheetId`, and grid size. Keep it small: never call `get_spreadsheet` without `fields`.
  - **标签页与 ID：** `get_spreadsheet` 配 `fields: ["sheets.properties"]`。它返回每个标签页的 `title`、`sheetId` 与网格大小。保持精简：绝不不带 `fields` 调用 `get_spreadsheet`。
- **Field masks: one path per array item, no parentheses.** This connector rejects `sheets.properties(sheetId,title)` and `"spreadsheetId,properties.title"` as one item, even though the tool's own description suggests the parenthesized form. Pass separate items instead: `["spreadsheetId", "properties.title", "sheets.properties"]`.
  - **字段掩码：每个数组项一条路径，不加括号。** 该连接器会把 `sheets.properties(sheetId,title)` 与 `"spreadsheetId,properties.title"` 作为单个条目拒绝，尽管工具自身的描述暗示了带括号的形式。请改为传独立条目：`["spreadsheetId", "properties.title", "sheets.properties"]`。
- **Displayed values:** `get_values` returns what the UI shows, as strings (`"$125"`, `"25.0%"`, `"#DIV/0!"`). It has no option for formulas or raw values. An empty row comes back as `[]`, trailing empty cells and rows are dropped, and a fully empty range comes back with no `values` key at all.
  - **显示值：** `get_values` 返回 UI 所示内容，均为字符串（`"$125"`、`"25.0%"`、`"#DIV/0!"`）。它没有读取公式或原始值的选项。空行返回 `[]`，行尾的空单元格与空行会被丢弃，完全为空的区间返回时根本没有 `values` 键。
- **Read to the end of the data with an open range.** On a small tab, `'Log'!A:F` returns the header and every filled row, and nothing past the last one, so one read gives both the layout and where the data ends. Don't guess a fixed end such as `A1:Z1000`: a range past the tab's last row can fail with "exceeds grid limits", and deleting rows makes the tab shorter. To format a column down to the end, leave `endRowIndex` out of the grid range (`sheets_helper.py range "D2:D"` does this).
  - **用开放区间读到数据末尾。** 在小标签页上，`'Log'!A:F` 返回表头与每个已填行，不多于最后一行，一次读取同时拿到版式与数据终点。不要猜测固定终点如 `A1:Z1000`：超出标签页最后一行的区间可能报 "exceeds grid limits" 而失败，且删除行会让标签页变短。要把格式一直应用到列尾，把 `endRowIndex` 从网格区间中省去（`sheets_helper.py range "D2:D"` 就是这么做的）。
- **Formulas, raw values, and errors:** read the grid instead. Use exactly this call, which is small and has everything the helper needs:
  - **公式、原始值与错误：** 改为读取网格。使用与下面完全一致的调用，它体积小且包含助手所需的一切：

```json
{"spreadsheetId": "...", "includeGridData": true, "ranges": ["'Model'!A1:H40"],
 "fields": ["sheets.properties.title", "sheets.data.startRow", "sheets.data.startColumn",
            "sheets.data.rowData.values.userEnteredValue",
            "sheets.data.rowData.values.formattedValue",
            "sheets.data.rowData.values.effectiveValue"]}
```

Then list the cells, with each formula next to its displayed result:

然后列出单元格，每条公式旁标注其显示结果：

```
python <skill>/scripts/sheets_helper.py cells grid.json
Sheet1!D5   '=B5*(1+C5)'   -> '$125'
Sheet1!D7   '=D5/0'        -> '#DIV/0!'  <-- ERROR DIVIDE_BY_ZERO
errors: 1
```

Add `--errors-only` to list only the error cells. Add `sheets.data.rowData.values.userEnteredFormat` to the fields when you need to read formatting, and keep the range tight: grid reads grow fast.

加 `--errors-only` 只列出错误单元格。需要读取格式时，把 `sheets.data.rowData.values.userEnteredFormat` 加进 `fields`，并保持区间紧凑：网格读取的体积增长很快。

## Write values and formulas / 写入值与公式

- **Both write tools parse input the way the Sheets UI does.** `update_values` and `update_formulas` both turn `"=A1+B1"` into a formula, `"0.25"` into a number, `"3/14/2026"` into a date, and `"00123"` into the number 123. `"1/2"` becomes a date that still displays as 1/2, and a `SUM` over it adds the date's serial number, a five-digit number such as 46024. Period labels such as `"Jan 2027"`, `"2027-01"` or `"Jan 1"` become dates too; a bare month abbreviation such as `"Jan"` stayed text. Because both parse the same way, one `update_formulas` call can write a block's labels, numbers and formulas together.
  - **两个写入工具都按 Sheets UI 的方式解析输入。** `update_values` 与 `update_formulas` 都会把 `"=A1+B1"` 变成公式、`"0.25"` 变成数字、`"3/14/2026"` 变成日期、`"00123"` 变成数字 123。`"1/2"` 会变成一个仍显示为 1/2 的日期，对它求 `SUM` 会把日期的序列号加进来，即 46024 这样的五位数。`"Jan 2027"`、`"2027-01"` 或 `"Jan 1"` 这类期间标签也会变成日期；`"Jan"` 这种裸月份缩写则保持为文本。因为两者解析方式相同，一次 `update_formulas` 调用可以把一个区块的标签、数字与公式一并写入。
- **Keep text as text with a leading apostrophe.** `"'00123"` stores the text `00123`, and `"'=not a formula"` stores that literal text. Use it for IDs, ZIP codes, account numbers, and any text that looks like a number, date, or formula. Cells formatted as `TEXT` beforehand also keep input as text; other number formats, such as currency, don't.
  - **用前导单引号让文本保持为文本。** `"'00123"` 存储文本 `00123`，`"'=not a formula"` 存储那段字面文本。对 ID、邮编、账号以及任何看起来像数字、日期或公式的文本使用它。预先格式化为 `TEXT` 的单元格也会把输入保持为文本；其他数字格式（如货币）则不会。
- **Send numbers as numbers**, not strings. Store percentages as fractions, such as `0.25` for 25%, and format them as percent.
  - **数字以数字类型发送**，而不是字符串。百分比以小数存储，例如 25% 存 `0.25`，并格式化为百分比。
- **`null` skips a cell; `""` clears it.** Booleans become `TRUE` / `FALSE`.
  - **`null` 跳过单元格；`""` 清空它。** 布尔值变为 `TRUE` / `FALSE`。
- **Write formulas for every calculated cell.** Do not write results you computed yourself. Formulas keep the sheet live when the user changes an input.
  - **每个计算单元格都写公式。** 不要写入你自己算出的结果。公式能让工作表在用户修改输入时保持联动。
- **The range should fit the data.** Give the full extent (`'Model'!A4:D6` for 3 rows of 4) or just the top-left cell. Writing a 2D array into a smaller range fails with "Requested writing within range …".
  - **区间应与数据吻合。** 给出完整范围（3 行 4 列用 `'Model'!A4:D6`）或只给左上角单元格。把二维数组写入更小的区间会报 "Requested writing within range …" 而失败。
- **Clearing values leaves the cells' number formats.** A cell that held a date shows the next number you write as a date. Reset the format with `repeatCell` before reusing the cells.
  - **清除值会保留单元格的数字格式。** 原本存日期的单元格会把下一个写入的数字显示为日期。复用这些单元格之前先用 `repeatCell` 重置格式。
- **Inserting rows or columns shifts formulas for you.** `insert_dimension` (or `insertDimension` in a batch) updates references in existing formulas, so re-read before writing to rows below an insert. Set `inheritFromBefore: true` to copy the formatting of the row above. Directly under a header row, pass `false`: `true` copies the header's formatting. A row inserted between the last summed row and its total row is not added to the `SUM`, and neither is a row inserted at or above the first summed row: the range shifts down past it. Insert inside the range, or rewrite the total as a formula over the new rows; never replace it with a typed number.
  - **插入行或列时会自动平移公式。** `insert_dimension`（批处理中为 `insertDimension`）会更新既有公式中的引用，因此在插入点下方的行写入之前要重新读取。设置 `inheritFromBefore: true` 可复制上一行的格式。紧挨表头行下方时传 `false`：`true` 会复制表头的格式。插在最后一个被求和行与其合计行之间的行不会进入 `SUM`，插在第一个被求和行之上或其位置的行也不会：区间会整体下移越过它。要么插在区间内部，要么把合计改写为覆盖新行的公式；绝不要把它换成键入的数字。

## Batch updates: structure, formatting, charts / 批量更新：结构、格式与图表

`update_spreadsheet` sends raw `spreadsheets.batchUpdate` requests. The reads in this file return no `revisionId`, so send these requests without `writeControl`, and read again right before a write that depends on positions.

`update_spreadsheet` 发送原始的 `spreadsheets.batchUpdate` 请求。本文件中的读取不返回 `revisionId`，因此发送这些请求时不带 `writeControl`，并在执行依赖位置的写入之前先重新读取。

- **Field masks inside requests have the same rule:** comma-separated full paths, no parentheses. `"fields": "userEnteredFormat.numberFormat,userEnteredFormat.textFormat.foregroundColor"` works; `"userEnteredFormat(numberFormat,textFormat)"` is rejected.
  - **请求内部的字段掩码遵循同一规则：** 逗号分隔的完整路径，不加括号。`"fields": "userEnteredFormat.numberFormat,userEnteredFormat.textFormat.foregroundColor"` 有效；`"userEnteredFormat(numberFormat,textFormat)"` 会被拒绝。
- **Colors are `red`, `green`, `blue` from 0 to 1**, not hex.
  - **颜色是取值 0 到 1 的 `red`、`green`、`blue`**，不是十六进制。
- **Formatting: use the helper.** Describe the formatting in a short spec and `sheets_helper.py format` builds the requests, with correct ranges, colors, and field masks:
  - **格式：使用助手脚本。** 把格式描述写成简短的 spec，`sheets_helper.py format` 会构建请求，区间、颜色与字段掩码都正确无误：

```json
{"sheet": "Model",
 "rules": [
   {"range": "A1:F1", "bold": true, "bg": "#1F3864", "fg": "#FFFFFF", "align": "center"},
   {"range": "B2:B8", "fg": "#0000FF"},
   {"range": "C2:F20", "number": "currency"},
   {"range": "G2:G20", "number": "percent"},
   {"range": "A1:G20", "borders": "#BFBFBF"}],
 "freeze": {"rows": 1, "columns": 1},
 "col_widths": {"A": 220, "B:G": 110},
 "row_heights": {"1": 30}}
```

```
python <skill>/scripts/sheets_helper.py format spec.json --meta meta.json
```

For one tab whose `sheetId` you already know, such as `0` on a new sheet, pass `--sheet-id 0` instead of `--meta` and leave `sheet` out of the spec. The output is the full `requests` for `update_spreadsheet`. Named number formats: `currency`, `currency2`, `percent`, `multiple`, `integer`, `decimal`, `date`, `text`; any other string is used as a Sheets pattern. Rule keys: `bold`, `italic`, `size`, `font`, `fg`, `bg`, `align`, `valign`, `wrap`, `number`, `borders`.

对已知道 `sheetId` 的单个标签页（例如新表的 `0`），传 `--sheet-id 0` 代替 `--meta`，并在 spec 中省去 `sheet`。输出即 `update_spreadsheet` 所需的完整 `requests`。命名数字格式：`currency`、`currency2`、`percent`、`multiple`、`integer`、`decimal`、`date`、`text`；其他任何字符串都会作为 Sheets 模式使用。规则键：`bold`、`italic`、`size`、`font`、`fg`、`bg`、`align`、`valign`、`wrap`、`number`、`borders`。

- **One conditional-format rule can cover several columns.** `addConditionalFormatRule` takes a list of `ranges`. Write a `CUSTOM_FORMULA` for the top-left cell of the first range; Sheets shifts its relative references for every other cell, including cells in the other ranges. With ranges D5:D15, F5:F15 and H5:H15, `=D5>C5` compares each actual with the plan in the column to its left. Add `$` to fix a column: `=$J5>$I5` on K5:L15 colors both cells of a row by one comparison.
  - **一条条件格式规则可以覆盖多列。** `addConditionalFormatRule` 接受一个 `ranges` 列表。为第一个区间的左上角单元格写 `CUSTOM_FORMULA`；Sheets 会为其他每个单元格平移其相对引用，包括其他区间中的单元格。区间为 D5:D15、F5:F15 与 H5:H15 时，`=D5>C5` 会把每个实际值与其左侧计划列比较。加 `$` 锁定列：K5:L15 上的 `=$J5>$I5` 用一次比较给一行的两个单元格着色。
- **Labels over a group of columns:** an unmerged label such as "July" above a Plan and an Actual column sits at the left edge of its first cell, away from the right-aligned numbers under it. Merge the group's label cells with `mergeCells` and center them, or drop the group row and name each column ("Jul plan", "Jul actual").
  - **横跨一组列的标签：** Plan 列与 Actual 列上方未合并的 "July" 之类标签停在其第一个单元格的左缘，偏离其下方右对齐的数字。用 `mergeCells` 合并该组的标签单元格并居中，或删掉组行、为每列单独命名（"Jul plan"、"Jul actual"）。
- **Other common requests:** `addSheet` (with `properties.sheetId` and `title`), `updateSheetProperties` (rename with `fields: "title"`; freeze rows with `gridProperties.frozenRowCount`), `deleteDimension`, `mergeCells`, `setDataValidation` (dropdowns, checkboxes), `addConditionalFormatRule`, `sortRange`, `findReplace`, `autoResizeDimensions`.
  - **其他常用请求：** `addSheet`（配 `properties.sheetId` 与 `title`）、`updateSheetProperties`（用 `fields: "title"` 重命名；用 `gridProperties.frozenRowCount` 冻结行）、`deleteDimension`、`mergeCells`、`setDataValidation`（下拉框、复选框）、`addConditionalFormatRule`、`sortRange`、`findReplace`、`autoResizeDimensions`。
- **Charts work.** `addChart` creates a native chart; the reply returns its `chartId`, which Slides can embed. A minimal column chart:
  - **图表可用。** `addChart` 创建原生图表；回复返回其 `chartId`，Slides 可嵌入它。一个最小的柱状图：

```json
{"addChart": {"chart": {
  "spec": {"title": "Revenue", "basicChart": {"chartType": "COLUMN",
    "domains": [{"domain": {"sourceRange": {"sources": [
      {"sheetId": 0, "startRowIndex": 0, "endRowIndex": 6, "startColumnIndex": 0, "endColumnIndex": 1}]}}}],
    "series": [{"series": {"sourceRange": {"sources": [
      {"sheetId": 0, "startRowIndex": 0, "endRowIndex": 6, "startColumnIndex": 1, "endColumnIndex": 2}]}},
      "targetAxis": "LEFT_AXIS"}],
    "headerCount": 1}},
  "position": {"overlayPosition": {"anchorCell": {"sheetId": 0, "rowIndex": 8, "columnIndex": 0}}}}}}
```

Use `LINE` for trends over time, `BAR` or `COLUMN` for comparisons, and set `headerCount: 1` when the first row is a label.

趋势随时间变化用 `LINE`，比较用 `BAR` 或 `COLUMN`，首行是标签时设 `headerCount: 1`。

- **A chart that will be embedded in a Slides deck** follows the deck's existing chart style. When the deck has none: no chart title, since the slide carries the label; no legend; and a value label on every bar. For those, leave `title` out of the `spec`, set `"legendPosition": "NO_LEGEND"` on `basicChart`, and add `"dataLabel": {"type": "DATA"}` to each bar or column series. Leave `dataLabel` off a line series: a line's few point labels and its name go on the slide as text boxes. To color a single bar, use the series' `styleOverrides` (`[{"index": 3, "colorStyle": {"rgbColor": {...}}}]`). *Untested through the connector.*
  - **将被嵌入 Slides 演示文稿的图表**应遵循该文稿既有的图表样式。若文稿没有：不要图表标题，因为幻灯片自带标签；不要图例；每个柱子上加数值标签。为此，在 `spec` 中省去 `title`，在 `basicChart` 上设 `"legendPosition": "NO_LEGEND"`，并给每个柱状或条形系列加 `"dataLabel": {"type": "DATA"}`。折线系列不要加 `dataLabel`：折线为数不多的点标签及其名称会以文本框形式放在幻灯片上。要给单独一根柱子着色，使用系列的 `styleOverrides`（`[{"index": 3, "colorStyle": {"rgbColor": {...}}}]`）。*未经连接器实测。*

## Verify / 验证

1. Read the displayed values of everything you built with `get_values`. Scan for `#REF!`, `#DIV/0!`, `#NAME?`, `#VALUE!`, `#N/A`, and `#ERROR!`, and check that totals match the numbers you expect.
   1. 用 `get_values` 读取你构建的一切的显示值。扫描 `#REF!`、`#DIV/0!`、`#NAME?`、`#VALUE!`、`#N/A` 与 `#ERROR!`，并核对合计与你预期的数字一致。
2. For models, run the grid read and `sheets_helper.py cells --errors-only`, then spot-check that calculated cells hold formulas (`'=...'`), not typed numbers.
   2. 对模型，运行网格读取与 `sheets_helper.py cells --errors-only`，再抽查计算单元格持有的是公式（`'=...'`）而不是键入的数字。
3. After an upload, confirm the tab names and sizes with the metadata read before editing.
   3. 上传之后，编辑前先用元数据读取确认标签页名称与大小。

## Function support / 函数支持

Sheets supports `XLOOKUP`, `FILTER`, `UNIQUE`, and `SORT`, so the `xlsx` skill's LibreOffice limits do not apply here. If the user may export the file to Excel, avoid Sheets-only functions such as `QUERY`, `ARRAYFORMULA`, `IMPORTRANGE`, and `GOOGLEFINANCE`. Formulas that fetch an outside URL (`IMAGE`, `IMPORTDATA`, `IMPORTXML`, `IMPORTHTML`, `IMPORTFEED`) show `#REF!` until someone opens the sheet in a browser and clicks Allow access, so tell the user.

Sheets 支持 `XLOOKUP`、`FILTER`、`UNIQUE` 与 `SORT`，因此 `xlsx` 技能中的 LibreOffice 限制在此不适用。如果用户可能把文件导出到 Excel，避免使用 Sheets 专属函数如 `QUERY`、`ARRAYFORMULA`、`IMPORTRANGE` 与 `GOOGLEFINANCE`。抓取外部 URL 的公式（`IMAGE`、`IMPORTDATA`、`IMPORTXML`、`IMPORTHTML`、`IMPORTFEED`）在有人于浏览器中打开该表并点击 Allow access 之前会显示 `#REF!`，所以要告知用户。

## Sheets failures / Sheets 故障排查

| Symptom | Cause | Fix |
|---|---|---|
| `Invalid field: sheets.properties(sheetId,title)` | The connector rejects parenthesized field masks | Pass one full path per array item: `["sheets.properties"]`. |
| `Invalid field: user_entered_format(number_format` | Parentheses in a request's `fields` | Use comma-separated full paths. `sheets_helper.py format` does this for you. |
| An ID lost its leading zeros, or text became a date | Both write tools parse input like the UI | Prefix the value with `'`. |
| A formula shows up as a value you typed | A computed result was written instead of a formula | Write the formula string starting with `=`. |
| `get_values` result has no `values` key | The range is empty | Treat it as empty; it is not an error. |
| Formatting landed on the wrong rows | 1-based rows used as 0-based indexes, or an inclusive end index | Build the range with `sheets_helper.py range`. |
| Formatting landed on the wrong tab | Tab position or name used as `sheetId` | Read `sheets.properties` and use the numeric `sheetId`. |
| Formulas point at the wrong rows after an insert | Rows were written using positions from before the insert | Re-read after `insertDimension`; Sheets already shifted existing formulas. |
| `Unable to parse range: <tab>!A1` | Usually a tab name that doesn't exist, not bad A1 syntax | Read `sheets.properties` and use the exact tab title. |

| 症状 | 原因 | 修复 |
|---|---|---|
| `Invalid field: sheets.properties(sheetId,title)` | 连接器拒绝带括号的字段掩码 | 每个数组项传一条完整路径：`["sheets.properties"]`。 |
| `Invalid field: user_entered_format(number_format` | 请求的 `fields` 中出现括号 | 使用逗号分隔的完整路径。`sheets_helper.py format` 会替你这样做。 |
| ID 丢失了前导零，或文本变成了日期 | 两个写入工具都按 UI 的方式解析输入 | 给值加前缀 `'`。 |
| 公式显示成了你键入的值 | 写入的是计算结果而不是公式 | 写以 `=` 开头的公式字符串。 |
| `get_values` 结果没有 `values` 键 | 区间为空 | 当作空处理；这不是错误。 |
| 格式落在了错误的行上 | 把 1 起始的行号当成了 0 起始索引，或把结束索引当成了含端点 | 用 `sheets_helper.py range` 构建区间。 |
| 格式落在了错误的标签页上 | 把标签页位置或名称当成了 `sheetId` | 读取 `sheets.properties` 并使用数字 `sheetId`。 |
| 插入行之后公式指向错误的行 | 用插入前的位置写入了行 | `insertDimension` 之后重新读取；Sheets 已平移既有公式。 |
| `Unable to parse range: <tab>!A1` | 通常是标签页名称不存在，而不是 A1 语法错误 | 读取 `sheets.properties` 并使用确切的标签页标题。 |
