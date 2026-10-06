---
description: Formatting rules for artifact markdown, covering .md deliverables, reports, and content exported to PDF or Word.
builders: file
kinds: markdown, pdf, document
---
<!-- BILINGUAL-EN-ZH -->

# Markdown formatting for artifact files / 工件文件的 Markdown 格式

These rules apply to artifact markdown: `.md` deliverables, reports, and any artifact content
that may be exported to PDF or Word.

这些规则适用于工件 Markdown：`.md` 交付物、报告，以及任何可能导出为 PDF 或 Word 的工件内容。

**Tables:**
- Cap cell content at ~25 characters. If a cell needs more, abbreviate or move details to a footnote below the table.
  单元格内容控制在约 25 个字符以内。如果需要更多内容，进行缩写，或把细节移到表格下方的脚注中。
- No full sentences in cells. Use short phrases, keywords, or values only.
  单元格中不写完整句子。只使用短语、关键词或数值。
- No URLs in table cells. Put links in a "Sources" section or footnotes below.
  表格单元格中不放 URL。把链接放在下方的 "Sources"（来源）小节或脚注中。
- 3-4 columns max. If you need more, split into multiple tables or use a definition list.
  最多 3-4 列。如果需要更多列，拆分成多个表格或改用定义列表。
- Right-align numeric columns with `---:` for scannability.
  数值列用 `---:` 右对齐，便于扫读。

**Lists:**
- One blank line between top-level bullets when each item has a sub-description or is longer than a few words.
  当每个顶级列表项带有子描述或长度超过几个词时，项与项之间空一行。
- No blank line between sub-bullets; keep them tight under their parent.
  子列表项之间不空行；让它们紧贴各自的父项。
- **Bold the lead phrase**, then follow with the description: `**Thing**: explanation here`.
  **加粗引导短语**，后接描述：`**Thing**: explanation here`。
- Max 2 levels of nesting. If you need a third level, restructure into sections with headers.
  最多嵌套 2 层。如果需要第三层，改用带标题的小节结构。

**Emojis:**
- No emojis in exportable documents (`.md` files intended for PDF/Word conversion). They render as boxes or missing glyphs in most PDF engines.
  可导出的文档（用于转换为 PDF/Word 的 `.md` 文件）中不使用表情符号。在大多数 PDF 引擎中，它们会渲染成方框或缺字。
- Use text markers instead: `**Note:**` for callouts, `-->` for flow, plain `*` bullets, `---` for dividers.
  改用文本标记：提示用 `**Note:**`，流程用 `-->`，列表用普通 `*`，分隔线用 `---`。
- Section headers in `.md` files: use plain text (`## Design`), not emoji-prefixed (`## 🎨 Design`).
  `.md` 文件中的小节标题用纯文本（`## Design`），不要加表情符号前缀（`## 🎨 Design`）。

**General:**
- Use `---` horizontal rules between major sections for visual breathing room.
  在主要小节之间使用 `---` 水平分隔线，留出视觉呼吸空间。
- Use `> [!NOTE]` / `> [!WARNING]` / `> [!TIP]` for callouts on GitHub. For portable output, use `**Note:**` bold prefixes.
  在 GitHub 上使用 `> [!NOTE]` / `> [!WARNING]` / `> [!TIP]` 作为提示框。为保证可移植性，使用 `**Note:**` 粗体前缀。
- Sentence case for headings (`## Key findings`), not title case unless it's a proper title.
  标题使用句首大写格式（`## Key findings`），除非是正式名称，否则不用标题式大小写。

【评论】禁用表情符号源于 PDF 渲染的字体兼容性问题（缺字方框），属于工程性排版约束而非内容规范。
