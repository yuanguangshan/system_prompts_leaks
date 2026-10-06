---
name: artifact_testing
metadata: { "includeInPrompt": false }
description: Verify an artifact before delivering it - file deliverables (pdf, pptx, docx, xlsx, csv) and web artifacts alike. Use whenever a build is about to return a link, or a build task asks for validation, QA, or a visual check. Covers the per-kind gate scripts, render-and-look verification, and leftover-placeholder scanning.
---

<!-- BILINGUAL-EN-ZH -->
# Artifact verification / Artifact 验证

One home for verification across the artifact namespace. The rule every
kind shares: a deliverable is verified by looking at what the user will
see, freshly rendered, not by trusting the code that produced it. Never
return a link while a gate below fails; after three failed fix attempts,
report the specific failure and ask for direction instead of iterating.

这是 artifact 命名空间下所有验证工作的统一归属。各类共享的规则是：交付物要通过查看用户将看到的内容——即全新渲染的结果——来验证，而不是信任生成它的代码。在下述任何门禁（gate）未通过时绝不要返回链接；三次修复尝试均失败后，报告具体失败原因并询问用户方向，而不是继续迭代。

## File kinds: gates, then eyes / 文件类：先门禁，后目检

| Kind | Gate |
|---|---|
| pdf | `render_audit.mjs` with the flags the pdf skill's `workflow.md` names, then `/opt/hatch/skills/artifacts/scripts/validate_pdf.sh` |
| presentation | `assemble_deck.mjs`, then `render_audit.mjs` with the flags `workflow.md` names |
| docx | render to PDF with headless LibreOffice (`soffice --headless --convert-to pdf`), then rasterize (`pdftoppm -jpeg -r 100`) |
| xlsx | `/opt/hatch/skills/artifacts/scripts/validate_xlsx.py` |
| csv / md | parse it back (csv: a Python `csv` read; md: read the file) |

| 类型 | 门禁 |
|---|---|
| pdf | 按 pdf 技能 `workflow.md` 所列参数运行 `render_audit.mjs`，然后运行 `/opt/hatch/skills/artifacts/scripts/validate_pdf.sh` |
| presentation | 先 `assemble_deck.mjs`，再按 `workflow.md` 所列参数运行 `render_audit.mjs` |
| docx | 用无头 LibreOffice 渲染为 PDF（`soffice --headless --convert-to pdf`），再栅格化（`pdftoppm -jpeg -r 100`） |
| xlsx | `/opt/hatch/skills/artifacts/scripts/validate_xlsx.py` |
| csv / md | 把内容解析回去（csv：用 Python `csv` 读取；md：直接读文件） |

The shared render engine is `/opt/hatch/skills/artifacts/scripts/render_audit.mjs`;
renders outlast `muse.exec`'s default yield, so size `yield_ms` past the expected
runtime and read the report file the flags name rather than trusting a
backgrounded command's silence.

共享渲染引擎是 `/opt/hatch/skills/artifacts/scripts/render_audit.mjs`；渲染耗时超出 `muse.exec` 的默认让渡（yield）时长，因此要把 `yield_ms` 设置得超过预期运行时长，并读取各参数所指定的报告文件，而不是把后台命令没有输出当作没问题。

**Then look.** Read every validation PNG or page image with fresh eyes - the
generating context sees what it expects, not what rendered. Check first for
text overflow or cut-off content, then overlaps, collisions, cramped or
uneven spacing, low-contrast text, and template decoration left behind.

**然后目检。** 以全新眼光查看每一张校验 PNG 或页面图片——生成环境看到的往往是它期望的样子，而不是实际渲染出的样子。先检查文字溢出或内容被截断，再检查重叠、碰撞、局促或不均匀的间距、低对比度文字，以及残留的模板装饰。
【评论】这句点出了自我验证的常见盲区：生成方容易把预期结果当成实际渲染结果。

**Placeholder scan.** Before returning, search the deliverable's text for
leftover scaffolding: TODO, lorem, placeholder, [insert, xxx runs, and
sample rows the user never asked for. Anything found is fixed, not shipped.

**占位符扫描。** 返回之前，在交付物文本中搜索残留的脚手架内容：TODO、lorem、placeholder、[insert、成串的 xxx，以及用户从未要求的示例行。发现任何一项都要先修复，而不是照常交付。

## Web artifacts / Web artifact

Web builds keep their own audit tools (`web_artifacts.build` and
`web_artifacts.audit` run the same capture engine with enforcement); this
skill's render-fresh and placeholder rules apply to their output all the
same.

Web 构建保留其专属的审计工具（`web_artifacts.build` 和 `web_artifacts.audit` 运行同一套带强制执行的捕获引擎）；本技能的"全新渲染目检"与"占位符"规则同样适用于它们的输出。
