---
name: artifact_markdown
metadata: { "includeInPrompt": false }
description: Build or revise a plain markdown file (md) deliverable such as notes, a README, meeting minutes, documentation, or text the user will edit or paste elsewhere. Use whenever a build task's artifact kind is markdown. Covers markdown formatting conventions and read-back verification.
---

<!-- BILINGUAL-EN-ZH -->

# Markdown artifacts / Markdown 工件

A markdown deliverable is plain text written directly with the file tools:
append sections with `muse.write` in append mode, keep each write small,
and use `muse.edit` for surgical fixes. There is no compile step; the
file at `<slug>.md` in the project directory root is the deliverable.

Markdown 交付物是直接用文件工具写入的纯文本：用追加模式的 `muse.write` 逐节追加内容，每次写入保持较小的规模，并用 `muse.edit` 做精准修补。没有编译步骤；项目目录根下的 `<slug>.md` 文件本身就是交付物。

| Task | Read first |
|---|---|
| Formatting (tables, lists, emphasis, headings) | `/opt/hatch/skills/artifacts/references/markdown.md` (shared) |

| 任务 | 先阅读 |
|---|---|
| 格式（表格、列表、强调、标题） | `/opt/hatch/skills/artifacts/references/markdown.md`（共享） |

Structure follows the content: real markdown headings, real list syntax,
tables only where rows and columns genuinely align. The file ships as the
user's own text, so no build scaffolding, no HTML unless the user asked
for it, and no trailing commentary that isn't part of the document.

结构服从内容：使用真正的 markdown 标题、真正的列表语法，只在行列确实对齐时才使用表格。该文件以用户自己的文本形式交付，因此不加构建脚手架，除非用户要求否则不用 HTML，也不在文末附加不属于文档的评论。

## Verification / 验证

Follow `/opt/hatch/skills/artifacts/testing/SKILL.md`: read the finished file back
in full before returning the link, checking structure renders as intended
and no placeholder or scratch content remains.

遵循 `/opt/hatch/skills/artifacts/testing/SKILL.md`：在返回链接之前完整回读已完成的文件，检查结构是否按预期渲染、是否还残留占位符或草稿内容。
