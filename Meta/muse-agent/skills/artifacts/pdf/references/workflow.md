---
description: HTML-first PDF generation. Build the HTML source, render it with render_audit.mjs, and validate the output pages.
---
<!-- BILINGUAL-EN-ZH -->

# PDF artifacts / PDF 工件

PDF generation is HTML-first. Build a self-contained HTML source, render it to PDF with
`render_audit.mjs` (a Playwright-driven render that also runs a font + overflow audit), then
validate before returning the link. Required validation tools (`pdfinfo`, `pdftoppm`, `pdftotext`, `fc-list`)
are preinstalled. `render_audit.mjs` depends on the Playwright runtime bundled with the
web Artifact runtime assets and uses the preinstalled Chromium at `/opt/meta-chromium/chrome`
when present (see the note at the end of this file).

PDF 生成以 HTML 为先。构建自包含的 HTML 源，用 `render_audit.mjs`（一次 Playwright 驱动的渲染，同时运行字体与溢出审计）渲染为 PDF，然后在返回链接之前完成验证。所需验证工具（`pdfinfo`、`pdftoppm`、`pdftotext`、`fc-list`）均已预装。`render_audit.mjs` 依赖随 web Artifact 运行时资产附带的 Playwright 运行时，并在存在时使用预装的 Chromium（`/opt/meta-chromium/chrome`）（见文末说明）。

**Core rules:**

**核心规则：**

- Write build sources under `project_dir/.src/` and the final PDF at `project_dir/<artifact-slug>.pdf`. Keep `.src/` after rendering.
  将构建源文件写在 `project_dir/.src/` 下，最终 PDF 写在 `project_dir/<artifact-slug>.pdf`。渲染后保留 `.src/`。
- Embed image assets as `data:` URIs, including CSS `background-image` URLs. Do not use external `http://`, `https://`, or `file://` image references.
  将图像资产以 `data:` URI 嵌入，包括 CSS `background-image` URL。不要使用外部 `http://`、`https://` 或 `file://` 图像引用。
- Photos must stay sharp at print resolution. The render gate measures this: an image with fewer pixels than its rendered width fails, and one below twice its rendered width gets an advisory. Source large (a cover or page-width hero wants a high-resolution original), then compress what you embed (JPEG for photos) so the `data:` URI stays small.
  照片在打印分辨率下必须保持清晰。渲染门会度量这一点：像素数少于其渲染宽度的图像会失败，低于其渲染宽度两倍的图像会收到一条建议（advisory）。选取大图（封面或整页宽的主图需要高分辨率原图），再压缩要嵌入的内容（照片用 JPEG），使 `data:` URI 保持较小。
- Whenever `media.generate_image` is used for artifact media, including PDF images, decorative/fictional map artwork, or any other illustrative asset, set the `output_dir` field to `artifact_media_dir` so generated files stay under `.src/media/` before embedding.
  凡是将 `media.generate_image` 用于工件媒体——包括 PDF 图像、装饰性/虚构的地图艺术图或任何其他插图资产——都把 `output_dir` 字段设为 `artifact_media_dir`，使生成的文件在嵌入前保持在 `.src/media/` 之下。
- Use one accurate image per concept. Omit an image rather than reusing a mismatched one.
  每个概念使用一张准确的图像。宁可省略图像，也不要复用不匹配的图像。
- Never disable TLS certificate verification to make a fetch succeed.
  绝不要为了让抓取成功而禁用 TLS 证书验证。
- When the document plots data, read `/opt/hatch/skills/artifacts/references/charts.md` first: it owns where the numbers come from, how to render a chart for a PDF, and the encoding rules that apply everywhere.
  当文档绘制数据图时，先阅读 `/opt/hatch/skills/artifacts/references/charts.md`：它负责数字的来源、如何为 PDF 渲染图表，以及适用于所有场合的编码规则。
- For maps of real places or geographic data, read `/opt/hatch/skills/artifacts/references/maps.md` first; it decides how a PDF's map is rendered and stored. `media.generate_image` is only for decorative or fictional map art.
  对于真实地点或地理数据的地图，先阅读 `/opt/hatch/skills/artifacts/references/maps.md`；它决定 PDF 中的地图如何渲染与存储。`media.generate_image` 仅用于装饰性或虚构的地图艺术图。
- Check content completeness before rendering: if a table or schedule names N items, the body should have N matching detail sections.
  渲染前检查内容完整性：如果表格或日程列出 N 个条目，正文应有 N 个对应的详情小节。
- Avoid emojis in PDF HTML; many PDF font stacks render them as boxes.
  避免在 PDF HTML 中使用 emoji；许多 PDF 字体栈会把它渲染成方框。
- Use ASCII hyphens instead of uncommon dash code points (U+2011 non-breaking hyphen and kin): the installed faces lack their glyphs and render boxes. En dashes in ranges are fine in Noto and Liberation, but any unusual glyph must be confirmed rendered in the PNG pass, never assumed.
  使用 ASCII 连字符，而不要使用不常见的连字符码点（U+2011 不换行连字符及同类）：已安装字体缺少对应字形，会渲染成方框。Noto 和 Liberation 中表示范围的 en dash 没有问题，但任何不常见的字形都必须在 PNG 检查环节确认已正确渲染，绝不能想当然。
- Use locally installed fonts. Verify with `fc-list` before naming one; reliable choices are `Noto Serif`, `Noto Sans`, `Liberation Serif`, and `Liberation Sans`. `DejaVu` is NOT installed on the VM image, so naming it silently renders in Chromium's default face instead. Do not rely on Google Fonts imports or Microsoft fonts like Arial, Calibri, Georgia, or Times New Roman.
  使用本地安装的字体。指定某个字体前先用 `fc-list` 验证；可靠的选择是 `Noto Serif`、`Noto Sans`、`Liberation Serif` 和 `Liberation Sans`。VM 镜像中并未安装 `DejaVu`，指定它只会静默地以 Chromium 默认字体渲染。不要依赖 Google Fonts 导入或 Arial、Calibri、Georgia、Times New Roman 等 Microsoft 字体。

**Revising a PDF this workspace built (the brief names an existing slug):** the PDF is a render
output, not the source. Recover `project_dir/.src/index.html`, make the change there, and
re-render through the same flow below. (A PDF the *user* supplied has no `.src/` and is a
different job: see `existing-pdfs.md`.)
Read the existing HTML first, and carry every section the current PDF has
through the re-render unless the brief asks to drop it. Cutting is a normal edit: when the brief
asks to shorten, trim an appendix, or hit a page count, remove whole sections deliberately and
carry the survivors through unchanged. What is not an edit is *unrequested* loss — content that
disappears because the document was rewritten from memory rather than modified. Never author a
replacement document from scratch and never write a second PDF beside the original. If `.src/` is
missing for that slug, say so and ask for direction — rebuilding from the rendered PDF silently
drops whatever the source held.

**修订本工作区构建过的 PDF（简报中给出既有 slug）：** PDF 是渲染输出，不是源文件。找回 `project_dir/.src/index.html`，在其中修改，然后通过下方同一流程重新渲染。（*用户*提供的 PDF 没有 `.src/`，属于另一类任务：参见 `existing-pdfs.md`。）
先阅读既有 HTML，除非简报要求删去，否则当前 PDF 已有的每个小节都要带入重新渲染。删减是正常编辑：当简报要求缩短、裁去附录或达到页数要求时，有意识地移除整个小节，其余原样带入。不是编辑的是*未经请求的丢失*——因凭记忆重写文档而非修改原文而消失的内容。绝不要从零编写替代文档，也绝不要在原文之外另写第二份 PDF。如果该 slug 缺少 `.src/`，如实说明并请求指示——凭渲染后的 PDF 重建会静默丢失源文件中的内容。

**HTML setup:**

**HTML 设置：**

1. Create `project_dir/.src/media/` and write `project_dir/.src/index.html`.
   创建 `project_dir/.src/media/` 并写入 `project_dir/.src/index.html`。
2. Include DOCTYPE, `lang`, charset, viewport, a descriptive `<title>`, and a single print-focused `<style>` block.
   包含 DOCTYPE、`lang`、charset、viewport、描述性的 `<title>`，以及单一的面向打印的 `<style>` 块。
3. Apply the user's styling direction from the verbatim request or parent conversation when provided. Map requested fonts to an installed local family before using them in CSS.
   如有提供，应用逐字请求或父对话中用户的样式指示。在 CSS 中使用所请求的字体之前，先把它映射到已安装的本地字体族。
4. Use this baseline print CSS, then adapt only as needed:
   使用以下基线打印 CSS，然后仅在需要时调整：

   ```css
   @page { size: A4; margin: 0; }
   * { box-sizing: border-box; print-color-adjust: exact; -webkit-print-color-adjust: exact; }
   body { margin: 0; font-family: 'Liberation Sans', 'Noto Sans', sans-serif; }
   .page { min-height: 297mm; padding: 2cm; }  /* Letter: min-height: 11in */
   .card, section, figure, table { break-inside: avoid; page-break-inside: avoid; }
   img { display: block; width: 100%; max-width: 100%; height: auto; max-height: 8cm; object-fit: contain; }
   .card, section { display: flow-root; }
   ```

   Keep `object-fit: contain` for charts, maps, diagrams, and screenshots. Combined with a `max-height`, `cover` crops the image to fill the box and silently cuts off content at the edges: axes, labels, legends, and outer data points disappear while the render report and `validate_pdf.sh` still pass, because a cropped image does not overflow its box. Reach for `cover` only on a decorative photo you mean to crop.

   对于图表、地图、示意图和截图，保持 `object-fit: contain`。与 `max-height` 组合时，`cover` 会裁切图像以填满容器，并静默切掉边缘内容：坐标轴、标签、图例和外围数据点会消失，而渲染报告和 `validate_pdf.sh` 仍然通过，因为被裁切的图像不会溢出其容器。只有对明确打算裁切的装饰性照片才使用 `cover`。

   Avoid full-bleed covers: bleeding a photo to the page edge stretches small sources, crops content, and leaves unintended margins. Default to an in-page hero, and go full-bleed only when the design calls for it and the source's pixels cover the page box.

   避免全出血封面：把照片铺到页面边缘会拉伸小尺寸原图、裁掉内容并留下意料之外的边距。默认使用页内主图，只有设计确有需要且原图像素足以覆盖页面盒时才做全出血。

**Render + audit:**

**渲染 + 审计：**

```sh
SLUG="<artifact-slug>"
# Your build task names `project_dir`. Use it. A goal's document is built under
# that goal's `files/` directory, so a hardcoded `your_files` path writes the
# PDF where nothing reads it.
# `project_dir` comes from your build task. Write its leading `~/` as
# `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
# `"~/workspace/..."` builds into a directory literally named `~`.
DIR="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>"
bun run "/opt/hatch/skills/artifacts/scripts/render_audit.mjs" \
  --html "$DIR/.src/index.html" \
  --pdf "$DIR/$SLUG.pdf" \
  --page-selector ".page" \
  --fonts "Liberation Sans:400,Liberation Sans:700" \
  --require-geometry --require-text-floor --require-image-resolution --hermetic --gate
```

`render_audit.mjs` loads the HTML in Chromium, honors the `@page { size: ... }` CSS rule
(`preferCSSPageSize`) instead of Chromium's default page size, writes the PDF, and prints a
single JSON report to stdout. Pass `--fonts "Family:weight,..."` for every webfont/face the
design relies on, and `--page-selector` matching your page wrapper (default
`.slide-container, .page, section.slide`) for overflow checks. Diagnostics go to stderr; the
process exits non-zero on a hard failure (missing Chromium, unresolvable Playwright, failed load).

`render_audit.mjs` 在 Chromium 中加载 HTML，遵循 `@page { size: ... }` CSS 规则（`preferCSSPageSize`）而非 Chromium 的默认页面尺寸，写出 PDF，并向 stdout 打印单份 JSON 报告。为设计依赖的每个 webfont/字体传入 `--fonts "Family:weight,..."`，并让 `--page-selector` 匹配你的页面包裹元素（默认 `.slide-container, .page, section.slide`）以进行溢出检查。诊断信息输出到 stderr；发生硬失败（缺少 Chromium、Playwright 无法解析、加载失败）时进程以非零码退出。

The JSON report has this shape:

JSON 报告具有如下结构：

```json
{ "ok": true, "pdf": "<abs>", "pages": 0, "pngs": [],
  "fonts": { "missing": [], "unused": [], "used": ["..."], "expected": ["..."] },
  "overflow": [ {"index": 0, "overflowY_px": 42, "overflowX_px": 0} ] }
```

**Gate on the report:** treat the render as failed and re-edit `.src/index.html` if `ok` is
false, if `fonts.missing` is non-empty (that face did not resolve under the requested policy;
`fonts.used` names what Chromium rasterized — fix the family spelling or choose an installed
local face), or if `overflow` is non-empty (content spills past a page box; reduce content or fix
the layout for the named element `index`). This
per-element overflow detection is more reliable than eyeballing the PNGs.

**以报告为准：** 若 `ok` 为 false、若 `fonts.missing` 非空（该字体在请求的策略下未能解析；`fonts.used` 列出了 Chromium 实际光栅化的字体——修正字体族拼写或改选已安装的本地字体）、或若 `overflow` 非空（内容溢出页面盒；减少内容或修复所列元素 `index` 的布局），都将渲染视为失败并重新编辑 `.src/index.html`。这种按元素检测溢出的方式比肉眼查看 PNG 更可靠。

Fix overflow by cutting content or letting it flow to another page, never by crushing the
design. Keep body text at 10pt or larger, nothing anywhere below 8pt, at least 12mm between
content and the paper edge (the page wrapper's padding, per the baseline CSS; do not add
`@page` margins for this), and visible gaps between blocks (3mm or more). `--require-text-floor` enforces
the type half of this: text below 8pt is a gate failure. A body that sits mostly below
10pt only raises an advisory, so the gate passing does not clear you: treat that advisory
as your own re-edit signal, restore the size and cut content, and leave it standing only
when the brief itself asked for a compact document. When the brief fixes the page count,
the content budget is what gives: drop the weakest items or tighten wording until the
page fits at full size.

修复溢出要靠删减内容或让它流到下一页，绝不要靠压缩设计。正文保持 10pt 或更大，任何地方都不低于 8pt，内容与纸张边缘之间至少留 12mm（按基线 CSS 由页面包裹元素的 padding 实现；不要为此添加 `@page` 边距），块与块之间留出可见间隙（3mm 或更多）。`--require-text-floor` 强制执行其中的字号下限：低于 8pt 的文本即门禁失败。正文大部分低于 10pt 只会引发一条建议，因此门禁通过不代表可以高枕无忧：把该建议当作自行重新编辑的信号，恢复字号并删减内容；只有简报本身要求紧凑文档时才可保留现状。当简报固定了页数时，让步的应是内容预算：删掉最弱的条目或收紧措辞，直到页面能以完整字号容纳。

`--require-geometry --gate` adds `geometry`: the page count and paper size read back from the
PDF that was written, reconciled against the page wrappers measured in the DOM under print media.
The PDF does not know which wrapper became which sheet, so it takes both to see a page split.

`--require-geometry --gate` 增加 `geometry`：从写出的 PDF 读回页数与纸张尺寸，并与在打印媒体下从 DOM 测得的页面包裹元素进行核对。PDF 不知道哪个包裹元素变成了哪张纸，因此需要两者共同判断分页。

`failures` must be fixed: a page element taller than the paper, or an authored page count that
does not match the produced page count. The page wrapper has to fit the paper it declares
(`min-height: 297mm` for A4, `11in` for Letter) with `box-sizing: border-box`, so padding sits
inside that height instead of adding to it.

`failures` 必须修复：某个页面元素高于纸张，或所写页数与产出的页数不一致。页面包裹元素必须适配其声明的纸张（A4 为 `min-height: 297mm`，Letter 为 `11in`）并使用 `box-sizing: border-box`，使 padding 位于该高度之内而不是叠加在其上。

`advisories` never gate: geometry that could not be measured, markup that reached the page as
visible text, and a body that sits mostly below the 10pt target. Leave the markup one alone in a
document that quotes code or markup on purpose; resolve the body one as the floor paragraph above
says, keeping it only for a deliberately compact brief.

`advisories` 从不构成门禁：无法测量的几何信息、以可见文本形式到达页面的标记，以及正文大部分低于 10pt 目标。在有意引用代码或标记的文档中，标记那条可以单独放行；正文那条按上文下限段落处理，只有刻意紧凑的简报才保留。

**Validation loop (mandatory):** Run this after every render. If a check fails, edit
`.src/index.html`, re-run the render+audit above, and re-validate. Maximum 3 iterations; after
that, report the specific failure and ask for direction. Never return the artifact link until
validation passes.

**验证循环（强制）：** 每次渲染后都要运行此步骤。若某项检查失败，编辑 `.src/index.html`，重新运行上面的渲染+审计，然后重新验证。最多 3 轮迭代；之后报告具体失败并请求指示。验证通过之前绝不返回工件链接。

```sh
PDF="$DIR/$SLUG.pdf"
HTML="$DIR/.src/index.html"
OUT="$DIR/.src/validate"
"/opt/hatch/skills/artifacts/scripts/validate_pdf.sh" "$PDF" "$HTML" "$OUT"
```

`validate_pdf.sh` checks PDF integrity, page metadata, and `<img>` and
CSS `url(...)` data-URI embedding via an HTML parser, then rasterizes every PDF page to
`page-*.png` in `.src/validate/` with `pdftoppm`. These PNGs are the visual review source
because they come from the actual PDF bytes.

`validate_pdf.sh` 通过 HTML 解析器检查 PDF 完整性、页面元数据以及 `<img>` 与 CSS `url(...)` 的 data-URI 嵌入，然后用 `pdftoppm` 把每一页 PDF 光栅化为 `.src/validate/` 中的 `page-*.png`。这些 PNG 是视觉审查的依据，因为它们来自实际的 PDF 字节。

After both the render report and `validate_pdf.sh` pass, **read every PNG from
`project_dir/.src/validate/` with the `read` tool before responding** (the build task names
`project_dir`; it is not always under `your_files`) and verify: images render, text
is legible and not clipped, text has readable margins, spacing is consistent across page
breaks, sections keep visible breathing room rather than running wall-to-wall,
sections/tables/figures do not split awkwardly across pages, headers/footers/page
numbers appear where expected for the artifact type, fonts look correct and do not fall back to
missing-glyph boxes, no accidental blank pages or giant gaps exist, covers are full-bleed only
when intended, images/charts match the adjacent content, and citations and references read as
human text with no tool tokens or placeholder strings left behind.

在渲染报告和 `validate_pdf.sh` 都通过之后，**响应前用 `read` 工具读取 `project_dir/.src/validate/` 中的每一张 PNG**（构建任务会给出 `project_dir`；它并不总在 `your_files` 之下），并验证：图像已渲染、文本清晰可读且未被裁切、文本有可读的边距、跨页断处间距一致、各小节保持可见的呼吸空间而非顶满页边、小节/表格/插图不会在跨页时被尴尬地拆开、页眉/页脚/页码按该工件类型的预期出现、字体正确且没有回退成缺字形方框、没有意外的空白页或巨大空隙、封面仅在有意时才全出血、图像/图表与相邻内容匹配，以及引文和参考文献呈现为人类可读的文本，没有残留的工具记号或占位字符串。

Reject any PNG showing a blank or white image box, text overlapping another element, or text
running off the page: fix the source and re-render. A missing image is a failed build, not a
cosmetic issue, and the render report will not always catch one, because an empty box does not
overflow its container.

任何 PNG 若显示空白或白色的图像框、文本与另一元素重叠、或文本跑出页面，都应拒绝：修复源文件并重新渲染。图像缺失是构建失败，不是外观问题，而且渲染报告不一定能发现它，因为空盒子不会溢出其容器。

Those PNGs carry the prose too, so run the read-back from
`/opt/hatch/skills/artifacts/references/prose.md` in this same pass, and
treat a page that renders perfectly but reads badly as a failed validation. Prose fixes go back
into `.src/index.html` and through the same render-and-revalidate loop as a layout fix.

这些 PNG 也承载正文文字，因此要在同一轮中执行 `/opt/hatch/skills/artifacts/references/prose.md` 的读回检查，并把渲染完美但读起来糟糕的页面视为验证失败。文字问题要回到 `.src/index.html`，走与布局修复相同的渲染并复验循环。

Common fixes: remove trailing page breaks for blank final pages, raise image `max-height` (or drop
it) when a chart or map renders too small to read, use `display: flow-root` or wrappers for margin
collapse, switch cover images to CSS backgrounds, and replace missing fonts with installed local
families. If content is missing from the edges of an image rather than the image being split, the
cause is `object-fit: cover` cropping it, not `max-height`.

常见修复：移除末尾分页符以消除空白末页；图表或地图渲染得太小无法阅读时，调大图像 `max-height`（或干脆去掉）；用 `display: flow-root` 或包裹元素处理外边距折叠；把封面图像改为 CSS 背景；用已安装的本地字体族替换缺失字体。如果是图像边缘内容缺失而非图像被拆开，原因是 `object-fit: cover` 裁切了它，而不是 `max-height`。

【评论】该流程把渲染验证设计为可度量的门禁（字号下限、几何核对、逐元素溢出检测），再以逐页 PNG 人工读回收尾，属于“失败关闭”式的产出质量控制，用以弥补自动检查无法覆盖的视觉缺陷。

## Content checks by artifact type / 按工件类型的内容检查

Use only the checks that match the artifact:

只使用与该工件匹配的检查项：

- **Reports and whitepapers:** Include an executive summary, page numbers, readable charts, and tables with headers.
  **报告与白皮书：** 包含执行摘要、页码、可读的图表和带表头的表格。
- **Data exports:** Include data source, extraction time, query/filter context, and record counts. Spot-check source values against the PDF.
  **数据导出：** 包含数据来源、提取时间、查询/过滤上下文和记录数。对照 PDF 抽查源值。
- **Invoices and receipts:** Verify required invoice fields, two-decimal currency formatting, and subtotal/tax/total math.
  **发票与收据：** 核验必需的发票字段、两位小数的货币格式以及小计/税额/总额的算术。
- **Travel guides:** Include complete addresses, hours, cost, nearest transit, and phone numbers as selectable text.
  **旅行指南：** 以可选中文本包含完整地址、营业时间、费用、最近的交通和电话号码。
- **Resumes:** Keep text selectable and in logical reading order. Use a single-column layout for ATS-focused resumes.
  **简历：** 保持文本可选且符合逻辑阅读顺序。面向 ATS 的简历使用单栏布局。
- **Manuals:** Include version/date, table of contents, figure captions, and code blocks that preserve indentation.
  **手册：** 包含版本/日期、目录、插图题注以及保留缩进的代码块。

> **Runtime dependency note (`render_audit.mjs`):** the render+audit script imports the  
> Playwright bundled with the web Artifact runtime assets (staged at  
> `/opt/hatch/skills/spaces/ts-runtime/dist/node_modules/playwright`) and launches the Muse  
> image-provisioned Chromium at `/opt/meta-chromium/chrome` when present. If that image path is  
> missing, it may use an already-present Playwright cached Chromium, but it does not install  
> Chromium at runtime. If resolution fails the script exits non-zero with a JSON  
> `{ok:false,error}` naming the missing piece.

> **运行时依赖说明（`render_audit.mjs`）：** 渲染+审计脚本导入随 web Artifact 运行时资产附带的  
> Playwright（暂存在  
> `/opt/hatch/skills/spaces/ts-runtime/dist/node_modules/playwright`），并在存在时启动 Muse  
> 镜像预置的 Chromium（`/opt/meta-chromium/chrome`）。如果该镜像路径缺失，它可能使用已存在的  
> Playwright 缓存 Chromium，但不会在运行时安装 Chromium。如果解析失败，脚本以非零码退出并输出 JSON  
> `{ok:false,error}`，指明缺失的部分。
