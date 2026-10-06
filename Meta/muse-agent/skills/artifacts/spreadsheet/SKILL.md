---
name: artifact_spreadsheet
metadata: { "includeInPrompt": false }
description: Create, read, edit, fix, or clean spreadsheet files (.xlsx, .xlsm, .csv, .tsv). Use whenever a build task's artifact kind is spreadsheet, or the task names a spreadsheet file and wants something done to it or produced from it, including restructuring messy tabular data into a proper workbook. Covers openpyxl generation, formulas and recalculation, editing existing workbooks, and the validation gates. Not for tasks whose deliverable is a document, report, or web page that merely contains a table.
---
<!-- BILINGUAL-EN-ZH -->

# Spreadsheet artifacts / 电子表格工件

An xlsx is generated with the preinstalled `openpyxl` library from a
generator script. Keep the generator under `.src/`: it is the editable
source for future revisions, and the binary is always regenerated from it.

xlsx 由预装的 `openpyxl` 库通过一个生成脚本产生。把生成脚本保存在 `.src/` 下：它是日后修订时可编辑的源文件，二进制文件始终由它重新生成。

| Task | Path |
|---|---|
| Create, or edit with formulas and formatting | `openpyxl` |
| Quick look at an existing sheet | `muse.read` (converted to markdown, paged); it carries no cell coordinates, so never plan edits from it |
| Read a workbook's model (formulas AND their values) | two `load_workbook` passes; see `/opt/hatch/skills/artifacts/spreadsheet/references/formulas.md` |
| Bulk or messy tabular data in or out | Python `csv` from the standard library; `pip install --break-system-packages pandas` when a task genuinely needs it |
| Design, structure, number formats, model conventions | `/opt/hatch/skills/artifacts/spreadsheet/references/visual.md` |
| Formulas, recalculation, editing existing workbooks | `/opt/hatch/skills/artifacts/spreadsheet/references/formulas.md` |

| 任务 | 路径 |
|---|---|
| 创建工作簿，或带公式和格式进行编辑 | `openpyxl` |
| 快速查看既有工作表 | `muse.read`（转换为 markdown，分页）；它不含单元格坐标，绝不要据此规划编辑 |
| 读取工作簿的模型（公式及其值） | 两次 `load_workbook` 加载；参见 `/opt/hatch/skills/artifacts/spreadsheet/references/formulas.md` |
| 批量或杂乱的表格数据导入/导出 | 标准库的 Python `csv`；任务确实需要时再 `pip install --break-system-packages pandas` |
| 设计、结构、数字格式、模型约定 | `/opt/hatch/skills/artifacts/spreadsheet/references/visual.md` |
| 公式、重新计算、编辑既有工作簿 | `/opt/hatch/skills/artifacts/spreadsheet/references/formulas.md` |

Requirements on every delivered workbook:

对每个交付工作簿的要求：

- Set a non-empty `workbook.properties.title` and a human-readable
  filename. Data enters the workbook from the build's gathered content
  only; a value you do not have is a blank cell or a question, never an
  invented number.
  设置非空的 `workbook.properties.title` 和人类可读的文件名。数据只能来自构建所汇集的内容；你没有的值就留成空白单元格或向用户提问，绝不可编造数字。
- Write formulas, never precomputed results: `sheet["B10"] = "=SUM(B2:B9)"`,
  not the Python-computed total. The sheet must recalculate when its
  inputs change.
  写公式，而不是预先算好的结果：`sheet["B10"] = "=SUM(B2:B9)"`，而不是用 Python 算出的总和。工作表必须在输入变化时能够重新计算。
- Follow the user's spec literally: their exact tab names, exact column
  headers, and the formula they spelled out. A redesign that computes
  something else fails, however elegant.
  严格照用户的规格执行：完全一致的标签页名称、完全一致的列标题，以及他们明确给出的公式。任何计算出另一种结果的重新设计都算失败，无论多么优雅。
- Zero formula errors at delivery: run the recalculation gate below and
  fix what it names.
  交付时公式错误必须为零：运行下文的重新计算关卡，并修复它点名的问题。
- Document assumptions and hardcoded numbers where the reader will see
  them (a cell comment or an adjacent labeled cell), citing the real
  source when one exists and saying plainly when the number came from the
  user.
  在读者能看到的位置（单元格批注或相邻的带标签单元格）记录假设和硬编码的数字；有真实来源时注明来源，数字来自用户时也如实说明。
- A workbook created for someone to fill in gets a short legend naming
  the cells to edit and one example row of realistic values; never add an
  example row to a file you were asked to edit.
  为他人填写而创建的工作簿要附一段简短图例，指出需编辑的单元格，并给出一行符合实际的示例值；对被要求编辑的文件，绝不要添加示例行。

## Scripts / 脚本

| Script | What it does |
|---|---|
| `/opt/hatch/skills/artifacts/scripts/recalc_xlsx.py` | Recalculates the workbook in place through headless LibreOffice and reports every formula-error cell as JSON; mandatory whenever the file contains formulas. `errors_found` exits 0: read the JSON, not the exit code |
| `/opt/hatch/skills/artifacts/scripts/validate_xlsx.py` | Opens the workbook read-back: zip integrity, sheet parts, populated-cell counts; do not deliver a link while it fails. A csv output gets no reader check |

| 脚本 | 作用 |
|---|---|
| `/opt/hatch/skills/artifacts/scripts/recalc_xlsx.py` | 通过无头 LibreOffice 就地重新计算工作簿，并以 JSON 报告每个公式出错单元格；文件含公式时为必跑步骤。`errors_found` 的退出码是 0：请读 JSON，不要看退出码 |
| `/opt/hatch/skills/artifacts/scripts/validate_xlsx.py` | 回读打开工作簿：zip 完整性、工作表部件、非空单元格计数；校验失败期间不要交付链接。csv 输出不做读取检查 |

## Verification / 验证

Run `recalc_xlsx.py` (when formulas exist), then `validate_xlsx.py`, then
follow `/opt/hatch/skills/artifacts/testing/SKILL.md`. A clean recalculation proves
the formulas evaluate, not that they are right: spot-check two or three
formulas pull the values you expect before building out a grid.

先运行 `recalc_xlsx.py`（存在公式时），再运行 `validate_xlsx.py`，然后遵循 `/opt/hatch/skills/artifacts/testing/SKILL.md`。重新计算通过只能证明公式可以求值，不能证明公式是对的：在铺开整个网格之前，先抽检两三个公式是否取到你期望的值。
