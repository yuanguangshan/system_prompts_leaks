---
name: artifact_document
metadata: { "includeInPrompt": false }
description: Create, read, edit, or manipulate Word documents (.docx) and Word templates (.dotx). Use whenever a build task's artifact kind is document with the default docx output, or the task mentions a Word doc, .docx, or .dotx, extracts or reorganizes content from one, inserts or replaces images, does find-and-replace in one, or works with tracked changes (redlines) or comments. Covers python-docx generation, raw OOXML editing of existing files, document structure and formatting, and render verification. Not for PDFs, spreadsheets, or Google Docs.
---
<!-- BILINGUAL-EN-ZH -->

# Word-document artifacts / Word 文档产物

A docx is generated with the preinstalled `python-docx` library from a
generator script. Keep the generator under `.src/`: it is the editable
source for future revisions, and the binary is always regenerated from it.

docx 由预装的 `python-docx` 库通过生成脚本产出。把生成脚本保留在 `.src/` 下：它是后续修订时可编辑的源文件，二进制文件始终由它重新生成。

| Task | Read first |
|---|---|
| Design and structure | `/opt/hatch/skills/artifacts/document/references/visual.md` |
| What the words say: outline, headings, tone, the prose read-back | `/opt/hatch/skills/artifacts/references/prose.md` (shared) |
| Content formatting (tables, lists, emphasis) | `/opt/hatch/skills/artifacts/references/markdown.md` (shared) |
| The document shows a place or a map | `/opt/hatch/skills/artifacts/references/maps.md` (shared) |
| Edit an existing or uploaded .docx/.dotx, tracked changes, comments, extract/read content, legacy .doc | `/opt/hatch/skills/artifacts/document/references/editing.md` |

| 任务 | 先读 |
|---|---|
| 设计与结构 | `/opt/hatch/skills/artifacts/document/references/visual.md` |
| 文字内容：大纲、标题、语气、文稿朗读 | `/opt/hatch/skills/artifacts/references/prose.md`（共享） |
| 内容格式（表格、列表、强调） | `/opt/hatch/skills/artifacts/references/markdown.md`（共享） |
| 文档展示地点或地图 | `/opt/hatch/skills/artifacts/references/maps.md`（共享） |
| 编辑现有或上传的 .docx/.dotx、修订（tracked changes）、批注、提取/读取内容、旧式 .doc | `/opt/hatch/skills/artifacts/document/references/editing.md` |

Set a non-empty `document.core_properties.title`, use a human-readable
filename and visible title, and never fake structure: real numbering for
lists (never a literal bullet character), real heading styles for anything a
table of contents must see, a paragraph bottom border for a rule (never a
one-row table), and separate paragraphs instead of newlines inside a run.

设置非空的 `document.core_properties.title`，使用人类可读的文件名与可见标题，并且绝不伪造结构：列表用真实的编号（绝不用字面项目符号字符），凡目录需要识别的内容都用真实标题样式，分隔线用段落下边框（绝不用单行表格），run 内部用独立段落而不是换行符。

【评论】“绝不伪造结构”条款要求用真实语义结构（标题样式、真实编号）而非视觉模仿，这保证了目录等依赖文档结构的功能可以正常工作。

## Verification / 验证

Follow `/opt/hatch/skills/artifacts/testing/SKILL.md`: render the document to PDF with
headless LibreOffice, rasterize with pdftoppm, and read every page image
before returning a link.

遵循 `/opt/hatch/skills/artifacts/testing/SKILL.md`：用无头 LibreOffice 将文档渲染为 PDF，用 pdftoppm 栅格化，并在返回链接之前逐一查看每页图像。
