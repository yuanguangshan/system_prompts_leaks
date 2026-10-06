<!-- BILINGUAL-EN-ZH -->
# Working with existing PDFs / 处理已有的 PDF

Reading, reorganizing, and filling PDFs the user already has (uploads under
`~/workspace/your_files/` or attachments). A NEW pdf deliverable is still
authored as HTML and rendered; this reference is for operating on PDF files
that already exist. Poppler ships in the cell; `pypdf` does not, so install
it on demand when a task below names it:  
`pip install --break-system-packages pypdf`.

读取、重组并填写用户已有的 PDF（`~/workspace/your_files/` 下的上传文件或附件）。新的 PDF 交付物仍然以 HTML 编写并渲染；本参考文档针对的是对已存在 PDF 文件的操作。Poppler 已随单元格内置；`pypdf` 则没有，因此当下方任务提到它时按需安装：  
`pip install --break-system-packages pypdf`。

| Task | Path |
|---|---|
| Read text | `muse.read` opens a PDF directly (converted to markdown, paged); `pdftotext -bbox-layout` only when you need per-word coordinates as XML |
| Look at pages | `pdftoppm -png -r 150 file.pdf page` then read the images; `-f N -l M` bounds the range, `-r 300` for fine print |
| List or extract embedded images | `pdfimages -list file.pdf`; `pdfimages -all file.pdf out/img` |
| Merge documents | `pdfunite a.pdf b.pdf out.pdf` |
| Split into pages / extract a range | `pdfseparate -f 2 -l 5 file.pdf page-%d.pdf`, then `pdfunite` the kept pages |
| Fill a form, stamp an overlay, crop, decrypt with a known password | `pypdf` (install on demand) |

| 任务 | 路径 |
|---|---|
| 读取文本 | `muse.read` 直接打开 PDF（转换为 markdown、分页）；仅在需要按词坐标的 XML 时使用 `pdftotext -bbox-layout` |
| 查看页面 | `pdftoppm -png -r 150 file.pdf page` 然后读取图像；`-f N -l M` 限定页码范围，`-r 300` 用于细小文字 |
| 列出或提取内嵌图像 | `pdfimages -list file.pdf`；`pdfimages -all file.pdf out/img` |
| 合并文档 | `pdfunite a.pdf b.pdf out.pdf` |
| 拆分为单页 / 提取页码范围 | `pdfseparate -f 2 -l 5 file.pdf page-%d.pdf`，然后用 `pdfunite` 合并保留的页面 |
| 填写表单、加盖覆盖层、裁剪、用已知密码解密 | `pypdf`（按需安装） |

Text extraction never proves layout: for anything visual (alignment,
what sits next to what, whether a value actually landed in a box), look
at rendered page images, not extracted text.

文本提取永远不能证明版面：任何视觉层面的问题（对齐、元素相邻关系、某个值是否真的落在框内），都要查看渲染后的页面图像，而不是提取出的文本。

## Filling forms / 填写表单

First find out whether the PDF has real fillable (AcroForm) fields, and
inspect both representations: the canonical `/AcroForm/Fields` tree and
each page's `/Widget` annotations (follow `/Parent` and `/Kids`). A
widget can paint a value from its appearance stream while the canonical
field is missing or holds a stale value, so a clean render alone never
proves a fill.

首先确认 PDF 是否具有真正可填写的（AcroForm）字段，并检查两种表示形式：规范的 `/AcroForm/Fields` 树和每页的 `/Widget` 注解（顺着 `/Parent` 与 `/Kids` 追溯）。一个 widget 可以通过其外观流绘制出某个值，而规范字段却缺失或持有过期值，因此仅凭干净的渲染永远不能证明填写成功。

```python
from pypdf import PdfReader
reader = PdfReader("form.pdf")
fields = reader.get_fields()  # canonical /AcroForm/Fields tree
widgets = [
    annot
    for page in reader.pages
    for annot in (page.get("/Annots") or [])
    if annot.get_object().get("/Subtype") == "/Widget"
]
```

**Fillable fields.** Keep the result interactive by default; flatten only
when the user explicitly asks for a completed static form, and never
flatten a signed PDF without an explicit decision. Preserve the source
PDF, and keep the unflattened copy when the user may revise the form.
Inspect each field's type and states before writing: a text field takes a
string; a checkbox must be set to its own checked export value (read the
field's states; `/Off` is unchecked, the other state, often `/Yes` or
`/On`, checks it); a radio group takes one of its options' export values.

**可填写字段。** 默认保持结果可交互；仅在用户明确要求生成已完成的静态表单时才展平，且未经明确决定绝不展平已签名的 PDF。保留源 PDF，当用户可能还要修改表单时保留未展平的副本。写入前检查每个字段的类型与状态：文本字段接受字符串；复选框必须设置为它自身的选中导出值（读取该字段的状态；`/Off` 为未选中，另一个状态——通常是 `/Yes` 或 `/On`——表示选中）；单选组接受其某个选项的导出值。

Call `reattach_fields()` only when an expected field is actually missing
from `get_fields()`, never as routine repair: its orphan test is "widget
carrying `/FT` that is not in the TOP-LEVEL `/Fields` array", it never
follows `/Kids`, so on a well-formed form it appends properly parented
kid widgets (every radio kid, for instance) to the root `/Fields`,
shipping a spec-violating tree that value checks alone will not catch.
If a widget and a canonical field share a name but are distinct objects
with no `/Parent` relationship, do not reattach either: report the
ambiguity or deliver a flattened static result instead.

仅当预期字段确实在 `get_fields()` 中缺失时才调用 `reattach_fields()`，绝不要把它当作常规修复手段：它的孤儿判定是"携带 `/FT` 但不在顶层 `/Fields` 数组中的 widget"，它从不沿 `/Kids` 追溯，因此在格式良好的表单上，它会把本有正确父级的子 widget（例如每个单选子项）追加到根 `/Fields`，产出一个违反规范的树，仅靠值检查无法发现。若某个 widget 与规范字段同名但相互独立、无 `/Parent` 关系，则两者都不要重新挂接：报告这一歧义，或改为交付展平后的静态结果。

【评论】该段把一个第三方库方法限定为"仅限真正孤儿字段"的兜底手段，并详细说明误用会破坏单选组结构且难以通过值检查发现——属于对库边界行为的防御性说明。

Two verified pypdf flatten edges: radio-group kids share one flattened
appearance name (`/Fm_<field>`), so a flattened radio group paints every
position with the first kid's look and the selection is silently lost;
and the pre-paint walk crashes on a `/Btn` widget with no `/AP` (legal
when the form shipped `NeedAppearances`). Flatten only forms with no
radio groups whose buttons all carry `/AP`; otherwise deliver the
interactive fill and say why.

两个已验证的 pypdf 展平边界情形：单选组的子项共享同一个展平后的外观名称（`/Fm_<field>`），因此展平后的单选组会以第一个子项的外观绘制所有位置，且选中状态被静默丢失；另外，预绘制遍历会在没有 `/AP` 的 `/Btn` widget 上崩溃（当表单自带 `NeedAppearances` 时这是合法状态）。只展平没有单选组、且所有按钮都带 `/AP` 的表单；否则交付可交互的填写结果并说明原因。

```python
from pypdf import PdfReader, PdfWriter
from pypdf.generic import NameObject

writer = PdfWriter()
writer.clone_document_from_reader(PdfReader(input_pdf))
fields = writer.get_fields() or {}
if set(expected_values) - set(fields):
    writer.reattach_fields()  # recover genuinely orphaned widgets only
    fields = writer.get_fields() or {}
missing = set(expected_values) - set(fields)
if missing:
    raise ValueError(f"fields not found after repair: {sorted(missing)}")

values = dict(expected_values)
if flatten:
    # Paint every existing value before removing the widgets below.
    values = {
        name: field.get("/V", "/Off" if field.get("/FT") == "/Btn" else "")
        for name, field in fields.items()
    } | expected_values

# auto_regenerate=None leaves the input's NeedAppearances flag alone;
# True/False both overwrite it, and clearing it can stop untouched
# prefilled values from rendering in appearance-strict viewers.
writer.update_page_form_field_values(
    None, values, auto_regenerate=None, flatten=flatten
)
if flatten:
    # flatten=True paints appearances but leaves the widgets in place.
    writer.remove_annotations(subtypes="/Widget")
    writer.root_object.pop(NameObject("/AcroForm"), None)
with open(output_pdf, "wb") as stream:
    writer.write(stream)
```

**Verify on the reopened output, not the writer.** For an interactive
result: every expected field is present in `get_fields()` with the
expected `/V`, every page widget's effective value (its own `/V` or the
inherited `/Parent` one) agrees, each updated widget has a non-empty
`/AP` `/N` appearance, and no root `/Fields` entry carries `/Parent` (a
kid widget promoted to root is the reattach corruption above). Then
rasterize and read every page for stale or clipped appearances. Neither
`/NeedAppearances` nor a clean PNG render proves the logical field data
updated; the reopened field tree does. For a flattened result: zero
`/Widget` annotations and no `/AcroForm` field tree remain, and every
value, radio selections above all, is visible on the rendered pages.

**在重新打开的输出上验证，而不是在 writer 上。** 对可交互结果：每个预期字段都出现在 `get_fields()` 中且 `/V` 符合预期，每个页面 widget 的有效值（其自身 `/V` 或继承自 `/Parent` 的值）一致，每个被更新的 widget 都有非空的 `/AP` `/N` 外观，并且根 `/Fields` 中没有条目携带 `/Parent`（被提升到根的子 widget 就是上述 reattach 损坏）。然后将所有页面栅格化并逐页查看是否存在过期或被裁剪的外观。`/NeedAppearances` 与干净的 PNG 渲染都不能证明逻辑字段数据已更新；重新打开后的字段树才能证明。对展平结果：不再存在任何 `/Widget` 注解和 `/AcroForm` 字段树，且每个值——尤其是单选选中项——都在渲染出的页面上可见。

**No fillable fields.** The form is just page content, so place text on
top of it:

**无可填写字段。** 表单只是页面内容，因此在它上面叠放文本：

1. Locate each blank. `pdftotext -bbox-layout` gives every label's exact
   coordinates (origin top-left, PDF points), and the entry area starts
   where the label ends and runs to the next label or rule. For scanned
   PDFs with no text layer, rasterize at high resolution and read the
   images, cropping tight regions (`gm convert page.png -crop WxH+X+Y
   out.png`) to pin down positions; convert pixels back to points by the
   ratio of page size to image size.
   定位每个空白处。`pdftotext -bbox-layout` 给出每个标签的精确坐标（原点在左上角，单位为 PDF 点），填写区从标签结束处延伸到下一个标签或横线。对于没有文本层的扫描 PDF，以高分辨率栅格化并读取图像，裁剪出紧凑区域（`gm convert page.png -crop WxH+X+Y out.png`）以确定位置；按页面尺寸与图像尺寸之比把像素换算回点。
2. Build a transparent overlay: author an HTML page per filled page,
   sized exactly to the PDF page, with each value absolutely positioned;
   render it through the normal pdf pipeline.
   构建透明覆盖层：为每个要填写的页面各编写一个 HTML 页面，尺寸与 PDF 页面完全一致，每个值绝对定位；通过正常的 pdf 流水线渲染。
3. Stamp with pypdf: `page.merge_page(overlay_page)` for each page, then
   write.
   用 pypdf 盖章：对每一页执行 `page.merge_page(overlay_page)`，然后写出。
4. Verify by rasterizing the output and reading every page; text sitting
   on the wrong line or overlapping a label means the coordinates, not
   the approach, need fixing.
   通过栅格化输出并逐页查看来验证；文字落在错误的行上或与标签重叠，意味着需要修正的是坐标，而不是方法本身。

Do not guess unknown answers onto a form: fill what the request supplies
and leave the rest blank.

不要把未知答案猜测着填到表单上：只填写请求所提供的内容，其余留空。

【评论】这是一条反幻觉约束：禁止用推测值补全表单空缺，只允许填写请求中明确给出的数据。

## Encrypted or damaged inputs / 加密或损坏的输入

`PdfReader.is_encrypted` plus `reader.decrypt(password)` handles a file
whose password the user supplied; never attempt to bypass a password the
user does not have. If poppler tools reject a file as damaged, say so and
ask for a re-export rather than hand-repairing the binary.

`PdfReader.is_encrypted` 加上 `reader.decrypt(password)` 可以处理用户提供密码的文件；绝不尝试绕过用户没有的密码。如果 poppler 工具判定文件损坏而拒绝处理，就如实说明并要求重新导出，而不是手工修补二进制。

【评论】"绝不绕过用户没有的密码"是一条安全边界条款：允许处理用户提供凭据的加密文件，但禁止任何形式的破解。

## Verify / 验证

Any produced or modified PDF ends with the render check in
`/opt/hatch/skills/artifacts/testing/SKILL.md`: rasterize with pdftoppm and read
every page image before returning a link.

任何生成或修改过的 PDF 都要以 `/opt/hatch/skills/artifacts/testing/SKILL.md` 中的渲染检查收尾：用 pdftoppm 栅格化并逐页查看图像，然后才返回链接。
