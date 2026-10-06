---
name: docx
description: "Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx) or Word templates (.dotx). Triggers include: any mention of Microsoft Word Documents, such as 'Word doc', 'word document', '.docx', '.dotx', 'microsoft doc'. Also use when extracting or reorganizing content from .docx or .dotx files, inserting or replacing images in documents, find-and-replace in Word files, working with tracked changes or comments, or converting content into a polished Word document. If the user asks for a deliverable as a Word or .docx file (to download, email or print), use this skill. However, if they ask for a document, page, report, memo, or notes WITHOUT naming a file format and the session offers Claude's own dedicated document or page skill or connector, use that instead, even if they will email or print it. Do NOT use for PDFs, spreadsheets, Google Docs, or coding unrelated to document generation."
license: Proprietary. LICENSE.txt has complete terms
---
<!-- BILINGUAL-EN-ZH -->

# DOCX creation, editing, and analysis / DOCX 创建、编辑与分析

A `.docx` is a ZIP archive of XML files. Choose your approach by task:

`.docx` 是一个由 XML 文件组成的 ZIP 压缩包。按任务选择处理方式：

| Task | Approach |
|---|---|
| **Create** a new document | Write a `docx` (npm) script — see gotchas below |
| **Edit** an existing document | `unzip` → edit `word/document.xml` → `zip` (docx-js cannot open existing files) |
| **Read** content | `pandoc -t markdown file.docx` |

| 任务 | 方式 |
|---|---|
| **创建**新文档 | 编写 `docx`（npm）脚本——见下方注意事项 |
| **编辑**现有文档 | `unzip` → 编辑 `word/document.xml` → `zip`（docx-js 无法打开已有文件） |
| **读取**内容 | `pandoc -t markdown file.docx` |

> Script paths below are relative to this skill's directory.

> 下文的脚本路径均相对于本技能目录。

## Creating with docx-js — gotchas / 使用 docx-js 创建——注意事项

`docx` is preinstalled — do not run `npm install` first; write the script and `require('docx')` directly. Only if that require fails: `npm install docx`. The model knows the API; these are the footguns:

`docx` 已预装——不要先运行 `npm install`；直接编写脚本并 `require('docx')`。只有当该 require 失败时才执行：`npm install docx`。模型了解该 API；以下是容易踩的坑：

- **Page size defaults to A4.** For US Letter set `page: { size: { width: 12240, height: 15840 } }` (DXA; 1440 = 1″).
  **页面尺寸默认为 A4。** 如需 US Letter，设置 `page: { size: { width: 12240, height: 15840 } }`（DXA 单位；1440 = 1 英寸）。
- **Landscape:** pass portrait dimensions and `orientation: PageOrientation.LANDSCAPE` — docx-js swaps width/height internally.
  **横向：** 传入纵向尺寸并设置 `orientation: PageOrientation.LANDSCAPE`——docx-js 会在内部交换宽高。
- **Tables need dual widths:** set `columnWidths` on the table AND `width` on every cell, both in `WidthType.DXA` (PERCENTAGE breaks in Google Docs). Column widths must sum to the table width.
  **表格需要双重宽度：** 在表格上设置 `columnWidths`，并在每个单元格上设置 `width`，两者都使用 `WidthType.DXA`（PERCENTAGE 在 Google Docs 中会出问题）。各列宽度之和必须等于表格宽度。
- **Table shading:** use `ShadingType.CLEAR`, never `SOLID` (renders black).
  **表格底纹：** 使用 `ShadingType.CLEAR`，绝不用 `SOLID`（会渲染成黑色）。
- **Lists:** never insert `•` literally; use a `numbering` config with `LevelFormat.BULLET`.
  **列表：** 绝不直接插入 `•` 字符；使用带 `LevelFormat.BULLET` 的 `numbering` 配置。
- **`ImageRun` requires `type:`** (`"png"`, `"jpg"`, …).
  **`ImageRun` 需要 `type:`**（`"png"`、`"jpg"` 等）。
- **`PageBreak` must be inside a `Paragraph`.**
  **`PageBreak` 必须放在 `Paragraph` 内部。**
- **Never use `\n`** — use separate `Paragraph` elements.
  **绝不使用 `\n`**——使用独立的 `Paragraph` 元素。
- **TOC:** headings must use built-in `HeadingLevel.*`; custom heading styles need `outlineLevel` set or they won't appear.
  **目录（TOC）：** 标题必须使用内置的 `HeadingLevel.*`；自定义标题样式需要设置 `outlineLevel`，否则不会出现在目录中。
- **Don't use a table as a horizontal rule** — use a paragraph bottom border instead.
  **不要用表格充当水平分隔线**——应使用段落底边框。
- **Dot-leader / right-aligned-on-same-line:** use `PositionalTab` (`alignment: PositionalTabAlignment.RIGHT`, `leader: PositionalTabLeader.DOT`) inside a `TextRun`, not literal `.` or space padding.
  **点状前导符/同行右对齐：** 在 `TextRun` 内使用 `PositionalTab`（`alignment: PositionalTabAlignment.RIGHT`、`leader: PositionalTabLeader.DOT`），而不是字面的 `.` 或空格填充。

## Verify the output / 验证输出

After writing a `.docx`, render it and look at it:

写出 `.docx` 之后，将其渲染并查看：

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.docx
pdftoppm -jpeg -r 100 output.pdf page
ls page-*.jpg   # then Read the images
```

`pdftoppm` zero-pads page numbers to the width of the page count (`page-01.jpg`…`page-12.jpg`).

`pdftoppm` 会按总页数的位数对页码补零（`page-01.jpg`……`page-12.jpg`）。

## Editing existing documents / 编辑现有文档

Legacy `.doc` files must be converted first: `python scripts/office/soffice.py --headless --convert-to docx file.doc`.

旧式 `.doc` 文件必须先转换：`python scripts/office/soffice.py --headless --convert-to docx file.doc`。

```bash
unzip -q doc.docx -d unpacked/
find unpacked -type l -delete   # strip symlink entries — docx from external parties is untrusted
python scripts/merge_runs.py unpacked/   # coalesce fragmented runs so text is findable
# edit unpacked/word/document.xml in place — do NOT reformat or pretty-print
(cd unpacked && rm -f ../out.docx && zip -Xr ../out.docx .)
python scripts/office/validate.py out.docx --original doc.docx   # XSD checks; --auto-repair fixes common issues
# redlining? add --author "<the name you redlined under>" to check every edit is tracked
```

Word splits text across many `<w:r>` runs (revision ids, spell-check markers), so a phrase you can see in the document often doesn't exist as a contiguous string in the XML. `merge_runs.py` merges adjacent identically-formatted runs in `word/document.xml` without changing content or rendering; it also accepts a `.docx` directly (`python scripts/merge_runs.py doc.docx -o merged.docx`).

Word 会把文本拆分到许多 `<w:r>` run 中（修订 ID、拼写检查标记等），因此你在文档中看到的短语在 XML 里往往不是一个连续字符串。`merge_runs.py` 会在不改变内容与渲染效果的前提下合并 `word/document.xml` 中相邻的同格式 run；它也可以直接接受 `.docx` 文件（`python scripts/merge_runs.py doc.docx -o merged.docx`）。

**Tracked changes:** when redlining, validate with `--author "<the name you redlined under>"` (needs `--original`) — it reports any text you changed without a `<w:ins>`/`<w:del>` around it, which is easy to do by accident and invisible in the accepted view. Wrap runs in `<w:ins>`/`<w:del>` with `w:id`, `w:author`, `w:date` attributes. Inside `<w:del>`, the text element is `<w:delText>`, not `<w:t>`. A deleted paragraph mark (`<w:pPr><w:rPr><w:del w:id=".." w:author=".." w:date=".."/></w:rPr></w:pPr>`) means "merge this paragraph into the next" — so deleting a paragraph outright is that plus a `<w:del>` around every run. The `<w:del/>` must come before the rPr's other children; their order is schema-enforced.

**修订（tracked changes）：** 做红线修订时，使用 `--author "<你修订时所用的名字>"`（需要 `--original`）进行验证——它会报告你改动了却未用 `<w:ins>`/`<w:del>` 包裹的任何文本，这种情况很容易无意发生，且在"接受修订"视图中不可见。用带有 `w:id`、`w:author`、`w:date` 属性的 `<w:ins>`/`<w:del>` 包裹 run。在 `<w:del>` 内部，文本元素是 `<w:delText>` 而不是 `<w:t>`。被删除的段落标记（`<w:pPr><w:rPr><w:del w:id=".." w:author=".." w:date=".."/></w:rPr></w:pPr>`）表示"把此段落并入下一段"——因此彻底删除一个段落就是上述操作再加上给每个 run 包一层 `<w:del>`。`<w:del/>` 必须位于 rPr 的其他子元素之前；其顺序由 schema 强制。

To produce a clean copy with all tracked changes accepted: `python scripts/accept_changes.py in.docx out.docx`.

要生成一份接受全部修订后的干净副本：`python scripts/accept_changes.py in.docx out.docx`。

Accepting a deleted paragraph mark should join that paragraph to the one below it, so a paragraph whose runs are *all* deleted vanishes. Word does this; `accept_changes.py` and `pandoc --track-changes=accept` don't always. Both fail the same way — they strip the deleted text but leave the emptied paragraph behind, which reads as a stray empty bullet when it was auto-numbered:

接受被删除的段落标记时，应把该段落与其下一段合并，从而使 run *全部*被删除的段落消失。Word 会这样做；`accept_changes.py` 和 `pandoc --track-changes=accept` 则不一定。两者以相同方式失败——它们剥离了被删除的文本，却留下被清空的段落，当该段落是自动编号时，看起来就像一个多余的空项目符号：

- `pandoc --track-changes=accept` never joins the paragraphs.
  `pandoc --track-changes=accept` 从不合并段落。
- `accept_changes.py` (LibreOffice) joins them correctly, except when the deleted paragraph is followed by an empty spacer paragraph.
  `accept_changes.py`（LibreOffice）能正确合并，但当被删除段落后面跟着一个空的间隔段落时除外。

An empty bullet in either view is an artifact of that view, not a defect in the document. Check paragraph deletions in the XML.

任一视图中的空项目符号都是该视图的产物，不是文档本身的缺陷。段落删除要在 XML 中核查。

## Comments / 批注

Comments require six cross-linked files. Use the helper — directory mode when you'll also be editing `document.xml` (saves an unzip/rezip cycle), `.docx`-direct mode otherwise:

批注需要六个相互关联的文件。请使用辅助脚本——如果你还要编辑 `document.xml`，用目录模式（省去一次解压/重打包循环）；否则用 `.docx` 直连模式：

```bash
# Against an already-unpacked directory (preferred when also placing markers)
python scripts/comment.py unpacked/ "Fees & expenses cap is too low"
python scripts/comment.py unpacked/ "Agreed" --parent 0

# Against a .docx directly
python scripts/comment.py contract.docx "This cap is too low" -o annotated.docx
```

The script writes `comments.xml`, `commentsExtended.xml`, `commentsIds.xml`, `commentsExtensible.xml`, the relationships, and the content-type overrides. Comment IDs are auto-assigned. It then prints the `<w:commentRangeStart>`/`<w:commentRangeEnd>`/`<w:commentReference>` snippet to add to `word/document.xml` so the comment anchors to specific text — until you place those markers, the comment exists but is not visible.

该脚本会写入 `comments.xml`、`commentsExtended.xml`、`commentsIds.xml`、`commentsExtensible.xml`、关系文件以及内容类型覆盖项。批注 ID 自动分配。随后它会打印需要添加到 `word/document.xml` 的 `<w:commentRangeStart>`/`<w:commentRangeEnd>`/`<w:commentReference>` 片段，使批注锚定到具体文本——在你放置这些标记之前，批注虽已存在但不可见。

## Dependencies / 依赖

`docx` (npm, preinstalled — install only if `require('docx')` fails) · `pandoc` · LibreOffice (`soffice`) · `pdftoppm` (Poppler)

`docx`（npm，已预装——仅在 `require('docx')` 失败时安装）· `pandoc` · LibreOffice（`soffice`）· `pdftoppm`（Poppler）
