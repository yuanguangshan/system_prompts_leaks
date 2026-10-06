---
name: artifact_pdf
metadata: { "includeInPrompt": false }
description: Build, revise, or manipulate a fixed-layout PDF (report, guide, one-pager, printable document). Use whenever a build task's artifact kind is pdf, a document build's output format is pdf, or the task reads, merges, splits, crops, or fills an existing PDF, including fillable AcroForms. Covers authoring the print-CSS HTML source, rendering, the geometry and validation gates, existing-PDF manipulation and form filling, and delivery under workspace/your_files.
---
<!-- BILINGUAL-EN-ZH -->

# PDF artifacts / PDF 产物

A PDF is authored as HTML with print CSS and rendered through the
shared capture engine. The kept source under `.src/` is the editable truth
for every future revision; the PDF binary is always regenerated, never
patched.

PDF 以带打印 CSS 的 HTML 编写，并通过共享的截图（capture）引擎渲染。`.src/` 下保留的源文件是后续每次修订时可编辑的事实来源；PDF 二进制文件总是重新生成，绝不打补丁。

【评论】“始终重新生成、绝不打补丁”的约定让产物状态可复现，避免了在二进制上做增量修改带来的不可追溯性。

| Task | Read first |
|---|---|
| Any PDF build or edit | `/opt/hatch/skills/artifacts/pdf/references/workflow.md` (the workflow: authoring, render, gates, validation loop) |
| Design and layout | `/opt/hatch/skills/artifacts/pdf/references/visual.md` |
| What the words say: outline, headings, tone, the prose read-back | `/opt/hatch/skills/artifacts/references/prose.md` (shared) |
| The document plots data | `/opt/hatch/skills/artifacts/references/charts.md` (shared) |
| The document shows a place or a map | `/opt/hatch/skills/artifacts/references/maps.md` (shared) |
| Read, merge, split, or extract from an existing PDF; fill a PDF form | `/opt/hatch/skills/artifacts/pdf/references/existing-pdfs.md` |

| 任务 | 先读 |
|---|---|
| 任何 PDF 构建或编辑 | `/opt/hatch/skills/artifacts/pdf/references/workflow.md`（工作流：编写、渲染、门禁、验证循环） |
| 设计与布局 | `/opt/hatch/skills/artifacts/pdf/references/visual.md` |
| 文字内容：大纲、标题、语气、文稿朗读 | `/opt/hatch/skills/artifacts/references/prose.md`（共享） |
| 文档绘制数据图表 | `/opt/hatch/skills/artifacts/references/charts.md`（共享） |
| 文档展示地点或地图 | `/opt/hatch/skills/artifacts/references/maps.md`（共享） |
| 读取、合并、拆分或提取现有 PDF；填写 PDF 表单 | `/opt/hatch/skills/artifacts/pdf/references/existing-pdfs.md` |

## Scripts / 脚本

| Script | What it does |
|---|---|
| `/opt/hatch/skills/artifacts/scripts/render_audit.mjs` (shared) | Renders the HTML source to PDF and PNGs and runs the render gates; `workflow.md` names the flags |
| `/opt/hatch/skills/artifacts/scripts/validate_pdf.sh` | Integrity, page metadata, data-URI embedding, full rasterization; run until it passes |

| 脚本 | 作用 |
|---|---|
| `/opt/hatch/skills/artifacts/scripts/render_audit.mjs`（共享） | 将 HTML 源渲染为 PDF 与 PNG，并运行渲染门禁；各标志的说明见 `workflow.md` |
| `/opt/hatch/skills/artifacts/scripts/validate_pdf.sh` | 完整性、页面元数据、data-URI 嵌入、完整栅格化检查；运行直到通过 |

## Verification / 验证

Follow `/opt/hatch/skills/artifacts/testing/SKILL.md`: run the gates, then read every
validation PNG before returning a link.

遵循 `/opt/hatch/skills/artifacts/testing/SKILL.md`：先运行各门禁，然后在返回链接之前逐一查看每张验证 PNG。

【评论】将“在返回链接前查看每张验证 PNG”设为强制步骤，属于对自动化渲染结果的人工兜底质检设计。
