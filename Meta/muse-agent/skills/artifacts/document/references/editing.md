<!-- BILINGUAL-EN-ZH -->

# Editing existing Word documents / 编辑现有 Word 文档

A `.docx` (and a `.dotx` template, handled identically) is a ZIP archive of
XML parts. Route by what the edit needs:

`.docx`（以及 `.dotx` 模板，处理方式相同）是一个由 XML 部件组成的 ZIP 归档。按编辑所需选择路径：

| Edit | Path |
|---|---|
| Content changes on a doc you generated | Revise the `.src/` generator and regenerate |
| Simple content changes on an uploaded doc | Open it with `python-docx`, edit, save |
| Tracked changes (redlines), comments, format-preserving surgical edits | Raw XML: unpack, edit `word/document.xml`, repack (`python-docx` cannot express these) |
| Read or extract content | `muse.read` opens a .docx directly (converted to markdown, paged); iterate with `python-docx` when you need structure the markdown flattens |
| Legacy `.doc` | Convert first: `soffice --headless --convert-to docx file.doc`, then treat as above |

| 编辑类型 | 路径 |
|---|---|
| 对你生成的文档做内容修改 | 修改 `.src/` 生成器并重新生成 |
| 对上传文档做简单内容修改 | 用 `python-docx` 打开、编辑、保存 |
| 修订（红线批注）、批注、保持格式的精细修改 | 原始 XML：解包、编辑 `word/document.xml`、重新打包（`python-docx` 无法表达这些操作） |
| 读取或提取内容 | `muse.read` 直接打开 .docx（转换为 markdown 并分页）；当需要被 markdown 展平的结构时，用 `python-docx` 迭代 |
| 旧版 `.doc` | 先转换：`soffice --headless --convert-to docx file.doc`，然后按上述方式处理 |

## The raw-XML round trip / 原始 XML 往返

```bash
unzip -q doc.docx -d unpacked/
find unpacked -type l -delete   # ZIP entries from outside parties can be symlinks; strip before touching the tree
# edit unpacked/word/document.xml IN PLACE
(cd unpacked && rm -f ../out.docx && zip -Xr ../out.docx .)
```

- Never pretty-print or reindent the XML: added whitespace text nodes
  perturb `xml:space`-sensitive content and change rendering.
  绝不要美化打印或重新缩进 XML：新增的空白文本节点会干扰对 `xml:space` 敏感的内容并改变渲染。
- Repack from inside the unpacked directory so part paths are
  archive-root-relative (`word/document.xml`, not `unpacked/word/...`);
  Word refuses an archive with prefixed paths.
  从解包目录内部重新打包，使部件路径相对于归档根目录（`word/document.xml`，而不是 `unpacked/word/...`）；Word 会拒绝带前缀路径的归档。
- `rm -f` the output first: `zip` appends into an existing archive, so a
  part you deleted from the tree survives without it. `-X` drops
  uid/gid/timestamp extra fields.
  先 `rm -f` 删除输出文件：`zip` 会向已有归档追加内容，否则你从目录树中删除的部件会残留下来。`-X` 去除 uid/gid/时间戳等额外字段。
- When extracting with Python instead, reject symlink entries
  (`stat.S_ISLNK(info.external_attr >> 16)`) and entries that resolve
  outside the destination before extraction.
  如果改用 Python 解压，要在解压前拒绝符号链接条目（`stat.S_ISLNK(info.external_attr >> 16)`）以及解析到目标目录之外的条目。
  【评论】"先删除 ZIP 内符号链接条目"是针对外部来源归档的安全防御，用于防止解压时经符号链接逃逸到目标目录之外。

## Run fragmentation / 文本运行碎片化

Word splits visible text across many `<w:r>` runs (revision ids,
spell-check markers, editing history), so a phrase you can read in the
document often does not exist as a contiguous string in the XML, and a
find-and-replace silently misses. Before string edits, coalesce adjacent
runs whose `<w:rPr>` serialize byte-identically (both absent also
matches); that criterion provably leaves rendering unchanged. `rsid*`
attributes and `<w:proofErr>` elements are pure metadata and safe to
strip. Never merge across two different `<w:ins>`/`<w:del>` wrappers:
that rewrites tracked-change structure and collapses separate revisions.

Word 会把可见文本拆散到许多 `<w:r>` 运行（run）中（修订 ID、拼写检查标记、编辑历史），因此你在文档中能读到的短语在 XML 中往往不是连续字符串，查找替换会悄悄漏掉。在做字符串编辑之前，先把 `<w:rPr>` 序列化后字节相同的相邻运行合并（两者都缺失也算匹配）；该准则可以证明不会改变渲染。`rsid*` 属性和 `<w:proofErr>` 元素是纯元数据，可以安全去除。绝不要跨越两个不同的 `<w:ins>`/`<w:del>` 包装合并：那会改写修订结构并把不同修订折叠到一起。

## Tracked changes (redlining) / 修订（红线批注）

Wrap changed runs in `<w:ins>`/`<w:del>`, each carrying `w:id`,
`w:author`, and `w:date`.

把被修改的运行包进 `<w:ins>`/`<w:del>`，每个都带有 `w:id`、`w:author` 和 `w:date`。

- Inside `<w:del>` the text element is `<w:delText>`, never `<w:t>`;
  field instructions become `<w:delInstrText>`.
  在 `<w:del>` 内部，文本元素是 `<w:delText>`，绝不是 `<w:t>`；域指令变为 `<w:delInstrText>`。
- A deleted paragraph MARK
  (`<w:pPr><w:rPr><w:del .../></w:rPr></w:pPr>`) means "merge this
  paragraph into the next". Deleting a whole paragraph is that plus a
  `<w:del>` around every run; either half alone is a different edit.
  段落删除标记（`<w:pPr><w:rPr><w:del .../></w:rPr></w:pPr>`）表示"把本段并入下一段"。删除整段等于该标记加上把每个运行包进 `<w:del>`；只做其中一半是另一种编辑。
- Inside `w:rPr` the `<w:del/>` child comes BEFORE the other children;
  rPr child order is schema-enforced.
  在 `w:rPr` 内部，`<w:del/>` 子元素必须排在其他子元素之前；rPr 的子元素顺序由 schema 强制。
- To reject another author's insertion, nest your `<w:del>` inside their
  `<w:ins>`; never edit or unwrap their wrapper. To restore their
  deletion, add your own `<w:ins>` after their `<w:del>`. A change is
  identified by (kind, author, date, text), so rewriting someone else's
  wrapper reads as a brand-new change.
  要拒绝其他作者的插入，把你的 `<w:del>` 嵌套进他们的 `<w:ins>` 内；绝不要编辑或拆开他们的包装。要恢复他们的删除，在他们的 `<w:del>` 之后加上你自己的 `<w:ins>`。一个修订由（类型、作者、日期、文本）标识，因此改写别人的包装会被识别为全新的修订。
- The silent failure mode is an edit made OUTSIDE any wrapper: invisible
  in the accepted view and recorded nowhere. After redlining, re-read the
  XML and confirm every text difference against the original sits inside
  a `<w:ins>`/`<w:del>` you authored. The document body is where this
  discipline applies; headers, footers, and footnotes are separate parts,
  so check them separately if you touched them.
  静默失败模式是进行了任何包装之外的编辑：在接受视图中不可见，也无任何记录。做完红线批注后，重新读取 XML，确认相对原文的每一处文本差异都位于你撰写的 `<w:ins>`/`<w:del>` 之内。此纪律适用于文档正文；页眉、页脚和脚注是独立部件，如果改动过它们，需要单独检查。

To hand back a clean all-accepted copy, drive headless LibreOffice with a
StarBasic macro dispatching `.uno:AcceptAllTrackedChanges`, then verify by
unpacking the output and grepping that no `w:ins`/`w:del` remain. Two
gotchas: soffice can hang after storing (a timeout is not a failure;
verify the output content instead of the exit), and a fully-deleted
paragraph followed by an empty spacer paragraph can survive as an emptied
paragraph, showing up as a stray empty bullet when auto-numbered. That
bullet is an artifact of the accepted view; judge paragraph deletions in
the XML.

要交回一份接受全部修订的干净副本，用 StarBasic 宏驱动无头 LibreOffice 派发 `.uno:AcceptAllTrackedChanges`，然后通过解包输出并 grep 确认没有残留 `w:ins`/`w:del` 来验证。两个坑：soffice 可能在存储后挂起（超时不是失败；要验证输出内容而不是退出码）；一个被完全删除的段落后面跟着一个空的间隔段落时，可能以清空段落的形式残留，在自动编号时表现为一个多余的空项目符号。该项目符号是接受视图的产物；段落删除要在 XML 中判断。

## Comments / 批注

Comments span six cross-linked locations: `word/comments.xml`,
`word/commentsExtended.xml`, `word/commentsIds.xml`,
`word/commentsExtensible.xml`, their four `Relationship` entries in
`word/_rels/document.xml.rels`, and four `Override` entries in
`[Content_Types].xml`. The ID chain: `w:comment` carries `w:id` and a
paragraph `w14:paraId`; commentsExtended keys on that paraId;
commentsIds maps paraId to a `durableId`; commentsExtensible keys on the
durableId.

批注横跨六个相互链接的位置：`word/comments.xml`、`word/commentsExtended.xml`、`word/commentsIds.xml`、`word/commentsExtensible.xml`、它们在 `word/_rels/document.xml.rels` 中的四个 `Relationship` 条目，以及 `[Content_Types].xml` 中的四个 `Override` 条目。ID 链：`w:comment` 携带 `w:id` 和段落 `w14:paraId`；commentsExtended 以该 paraId 为键；commentsIds 把 paraId 映射到 `durableId`；commentsExtensible 以 durableId 为键。

- Parts alone show nothing: anchor with `<w:commentRangeStart w:id="N"/>`
  ... `<w:commentRangeEnd w:id="N"/>` plus a
  `<w:commentReference w:id="N"/>` run. Range markers are direct children
  of `<w:p>`, never inside a `<w:r>`.
  只有部件本身什么也显示不出来：用 `<w:commentRangeStart w:id="N"/>` ... `<w:commentRangeEnd w:id="N"/>` 加上一个 `<w:commentReference w:id="N"/>` 运行来锚定。范围标记是 `<w:p>` 的直接子元素，绝不在 `<w:r>` 内部。
- A reply's `w15:commentEx` sets `w15:paraIdParent` to the parent
  comment's paraId, and its markers nest inside the parent's range.
  回复的 `w15:commentEx` 把 `w15:paraIdParent` 设为父批注的 paraId，且其标记嵌套在父批注的范围之内。
- ID ceilings: `w14:paraId` is hex below `0x80000000`, `durableId` below
  `0x7FFFFFFF`; generate with `randint(0, 0x7FFFFFFE)` formatted `%08X`.
  durableId is decimal in `numbering.xml` and hex everywhere else.
  ID 上限：`w14:paraId` 是小于 `0x80000000` 的十六进制数，`durableId` 小于 `0x7FFFFFFF`；用 `randint(0, 0x7FFFFFFE)` 生成并格式化为 `%08X`。durableId 在 `numbering.xml` 中是十进制，在其他地方都是十六进制。
- Declare the full Word namespace set (plus `mc:Ignorable`) on each new
  part's root element up front, so appended children never hit
  undeclared-prefix errors.
  在每个新部件的根元素上预先声明完整的 Word 命名空间集合（外加 `mc:Ignorable`），这样追加的子元素就不会遇到未声明前缀错误。

## Verify / 验证

Every edit ends with the render gate in
`/opt/hatch/skills/artifacts/testing/SKILL.md`: convert to PDF with headless
LibreOffice, rasterize with pdftoppm, and read every page image before
returning a link.

每次编辑都以 `/opt/hatch/skills/artifacts/testing/SKILL.md` 中的渲染闸门收尾：用无头 LibreOffice 转换为 PDF，用 pdftoppm 栅格化，并在返回链接之前查看每一页图像。
