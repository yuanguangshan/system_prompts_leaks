---
description: Visual rules for spreadsheets. Titles, hierarchy, alignment, number formats, deterministic charts, and the workbook validation a sheet must pass before delivery.
---
<!-- BILINGUAL-EN-ZH -->

# Spreadsheet Visual Guidance / 电子表格视觉规范

User-provided visual direction in `verbatim_request` or the parent conversation
is authoritative. Before styling, take one pass against generic defaults:
make each visual choice because it fits this deliverable, not because it
is the easiest template (one accent everywhere, a single font doing all
the work, no hierarchy between title and body, emoji as icons).

用户在 `verbatim_request` 或父对话中提供的视觉指示具有最高效力。在样式化之前，先对照通用默认值过一遍：每一个视觉选择都应因为适合本次交付物而做出，而不是因为它是最省事的模板（到处用同一个强调色、让单一字体包打天下、标题与正文毫无层级、拿 emoji 当图标）。

- Do not apply a saved PDF theme unless the user asked for styling.
  - 除非用户要求样式化，否则不要套用已保存的 PDF 主题。
- Use a visible title, clear hierarchy, consistent spacing, and restrained
  color. Tables need readable headers, stable alignment, and number formats
  appropriate to their data.
  - 使用醒目的标题、清晰的层级、一致的间距和克制的配色。表格需要可读的表头、稳定的对齐方式，以及与其数据相称的数字格式。
- Charts, plots, and other factual graphics are generated
  deterministically from their source data.
  - 图表、绘图和其他事实性图形都要从其源数据确定性地生成。

## Validation / 校验

Read the workbook back before returning its link:

返回链接之前先把工作簿读回来校验：

```sh
# Your build task names `project_dir`. Use it, not a hardcoded `your_files`
# path: an artifact built under a goal lives in that goal's `files/` directory.
# `project_dir` comes from your build task. Write its leading `~/` as
# `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
# `"~/workspace/..."` builds into a directory literally named `~`.
DIR="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>"
python3 "/opt/hatch/skills/artifacts/scripts/validate_xlsx.py" \
  "$DIR/<file_name>.xlsx" --json-out "$DIR/.src/validate/xlsx.json"
```

- It exits non-zero on a file that is not a real workbook, a corrupt archive,
  or one whose sheets are all empty. Fix and rerun; do not deliver a link while
  it fails. Pass `--allow-empty` when a blank template is the request.
  - 如果文件不是真正的工作簿、归档已损坏，或所有工作表都为空，脚本会以非零码退出。先修复再重跑；校验未通过时不要交付链接。当需求本身就是空白模板时，传入 `--allow-empty`。
- Quote its sheet, cell, and formula counts in your summary. A `csv` output
  gets no reader check.
  - 在总结中引用脚本报告的工作表、单元格和公式数量。`csv` 输出不做读取校验。

## Financial-model conventions / 财务模型惯例

Defaults for models and calculators, unless the user says otherwise or an
existing file already does something else:

模型和计算器的默认约定，除非用户另有说明，或已有文件已经采用了其他做法：

- Color code by role: blue text for hardcoded inputs and scenario levers,
  black for formulas, green for links to another sheet, red for links to
  another file, yellow fill for key assumptions and the cells the user
  should fill in.
  - 按角色配色：蓝色文字表示硬编码输入和情景调节项，黑色表示公式，绿色表示指向其他工作表的链接，红色表示指向其他文件的链接，黄色填充表示关键假设和应由用户填写的单元格。
- Number formats: currency `$#,##0` with the unit named in the header
  (`Revenue ($mm)`); negatives in parentheses and zeros rendered as a dash
  (`$#,##0;($#,##0);-`); percentages `0.0%` stored as fractions (`0.15`
  renders 15.0%, storing `15` renders 1500.0%); valuation multiples
  `0.0x`; years as text so `2024` never renders `2,024`.
  - 数字格式：货币用 `$#,##0`，单位写在表头中（如 `Revenue ($mm)`）；负数放括号内、零显示为短横线（`$#,##0;($#,##0);-`）；百分比用 `0.0%` 且以小数存储（`0.15` 渲染为 15.0%，存 `15` 会渲染成 1500.0%）；估值倍数用 `0.0x`；年份按文本存储，以免 `2024` 被渲染成 `2,024`。
- Structure: every assumption in its own labeled cell, referenced by the
  formulas that use it (`=B5*(1+$B$6)`, never `=B5*1.05`); formulas
  consistent across every projection period, since a lone hand-edited cell
  mid-row is the commonest silent error; guard denominators that can be
  zero.
  - 结构：每个假设都放在自己带标签的单元格里，由使用它的公式引用（如 `=B5*(1+$B$6)`，绝不用 `=B5*1.05`）；公式在所有预测期间保持一致，因为行中间孤立的手工改动单元格是最常见的隐性错误；对可能为零的分母加防护。
- A professional default font throughout (the cell ships Liberation Sans
  and Liberation Serif as the Arial and Times stand-ins).
  - 全文使用专业的默认字体（环境内置 Liberation Sans 和 Liberation Serif，分别作为 Arial 和 Times 的替代品）。

【评论】"颜色编码按角色区分输入/公式/跨表链接"是财务建模界的传统规范，写入系统提示词等于把行业惯例固化为代理的默认行为；百分比以小数存储的提醒则针对最常见的电子表格语义错误。
