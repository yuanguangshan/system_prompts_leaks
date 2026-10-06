---
name: pptx
description: "Use this skill any time a .pptx or .potx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations as PowerPoint (.pptx) files; reading, parsing, or extracting text from any .pptx or .potx file (even if the extracted content will be used elsewhere, like in an email, summary, or creating a different type of slide deck); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates (.potx), layouts, speaker notes, or comments. Trigger whenever the user asks for a PowerPoint or .pptx file, or references a .pptx or .potx filename, regardless of what they plan to do with the content afterward. However, when the user asks for a deck, slides, a slide deck, or a presentation without naming a file format, default to using a dedicated slide-deck artifact type or a separate slides skill if this session offers one; otherwise, use this skill."
license: Proprietary. LICENSE.txt has complete terms
---
<!-- BILINGUAL-EN-ZH -->

# PPTX creation, editing, and analysis / PPTX 创建、编辑与分析

If this session offers a dedicated slide-deck artifact type or a separate slides skill, and the user has neither asked for a PowerPoint/.pptx file nor supplied a .pptx/.potx file to edit, fill in, or convert, build the deck with that type or skill instead. A .pptx/.potx file given only as source material or as an example for a new deck ("make another deck like this one") does not count as supplied: read it with this skill, then build the new deck with that type or skill. This skill remains the right tool for producing .pptx files and for reading, editing, templating, or converting existing .pptx/.potx files.

如果本次会话提供了专用的幻灯片组 artifact 类型或独立的 slides 技能，而用户既未要求 PowerPoint/.pptx 文件，也未提供需要编辑、填充或转换的 .pptx/.potx 文件，则应改用该类型或技能来构建幻灯片组。仅作为素材或新幻灯片组范例提供的 .pptx/.potx 文件（"照这份再做一份"）不算"已提供"：先用本技能读取它，再用该类型或技能构建新幻灯片组。在生成 .pptx 文件，以及读取、编辑、套用模板或转换既有 .pptx/.potx 文件方面，本技能仍是正确的工具。

【评论】这是一条技能路由限定规则：把泛泛的"要一份幻灯片"请求让渡给会话中的专用幻灯片 artifact 类型或其他技能，避免本技能被过度触发。

A `.pptx` is a ZIP archive of XML files. Choose your approach by task:

`.pptx` 是一个由 XML 文件组成的 ZIP 归档。请根据任务选择处理方式：

| Task | Approach |
|---|---|
| **Create** a new deck | Write a `pptxgenjs` script — see gotchas below |
| **Edit** an existing deck, or build from a template | unzip → edit `ppt/slides/slideN.xml` → zip |
| **Read** content | `markitdown deck.pptx` (one block per slide under `<!-- Slide number: N -->` markers); visual grid: `python scripts/thumbnail.py deck.pptx` |

| 任务 | 方法 |
|---|---|
| **创建**新幻灯片组 | 编写 `pptxgenjs` 脚本 — 见下文注意事项 |
| **编辑**既有幻灯片组，或从模板构建 | 解压 → 编辑 `ppt/slides/slideN.xml` → 重新打包 |
| **读取**内容 | `markitdown deck.pptx`（每张幻灯片对应 `<!-- Slide number: N -->` 标记下的一个区块）；可视化网格：`python scripts/thumbnail.py deck.pptx` |

## Scripts / 脚本

Paths are relative to this skill's directory. Everything else is plain Python, `node`, or shell.

路径相对于本技能所在目录。其余脚本均为普通 Python、`node` 或 shell。

| Script | What it does |
|---|---|
| `scripts/thumbnail.py deck.pptx [prefix]` | Labeled grid of every slide, for picking template layouts. `.pptx` only. Pass `prefix` — it defaults to `thumbnails`, which overwrites the grids of any other deck done in the same directory |
| `scripts/add_slide.py unpacked/ slide2.xml [--after slideN.xml]` | Duplicate a slide (or a `slideLayoutN.xml`) with all the package bookkeeping. Also takes a `.pptx` directly with `-o out.pptx` |
| `scripts/clean.py unpacked/` | Delete slides, media, and rels no longer referenced. Run **after** `<p:sldIdLst>` is final |
| `scripts/office/validate.py deck.pptx [--original src.pptx]` | Schema, relationship, content-type, chart and slide checks; each failure names its fix. Pass `--original` for any template-derived deck — it baselines the schema checks against the template, so the template's own XSD errors don't read as yours |
| `scripts/office/soffice.py --headless --convert-to pdf deck.pptx` | LibreOffice wrapper — bare `soffice` hangs in this sandbox |

| 脚本 | 作用 |
|---|---|
| `scripts/thumbnail.py deck.pptx [prefix]` | 生成每张幻灯片的带标签网格图，用于挑选模板布局。仅支持 `.pptx`。可传 `prefix` — 默认为 `thumbnails`，会覆盖同目录中其他幻灯片组已生成的网格 |
| `scripts/add_slide.py unpacked/ slide2.xml [--after slideN.xml]` | 复制幻灯片（或 `slideLayoutN.xml`）并完成所有包级登记事项。也可通过 `-o out.pptx` 直接作用于 `.pptx` 文件 |
| `scripts/clean.py unpacked/` | 删除不再被引用的幻灯片、媒体和 rels。请在 `<p:sldIdLst>` 定稿**之后**运行 |
| `scripts/office/validate.py deck.pptx [--original src.pptx]` | 架构、关系、内容类型、图表与幻灯片检查；每项失败都会注明修复方法。凡由模板派生的幻灯片组都请传 `--original` — 它以模板为基线执行架构检查，模板自身的 XSD 错误就不会被算到你头上 |
| `scripts/office/soffice.py --headless --convert-to pdf deck.pptx` | LibreOffice 封装 — 在本沙箱中直接运行 `soffice` 会挂起 |

## Creating with pptxgenjs — gotchas / 使用 pptxgenjs 创建 — 注意事项

`pptxgenjs` is preinstalled — do not run `npm install` first; write the script and `require('pptxgenjs')` directly. Only if that require fails: `npm install pptxgenjs`. The model knows the API; these are the footguns:

`pptxgenjs` 已预装 — 不要先运行 `npm install`；直接编写脚本并 `require('pptxgenjs')`。仅当 require 失败时才执行：`npm install pptxgenjs`。模型了解该 API；以下是需要提防的坑：

【评论】这份"坑清单"以命令式断言罗列第三方库的具体缺陷行为，本质是用人工经验补足模型对库版本细节的知识缺口，属于典型的技能层知识外置设计。

- **Set `pres.layout` before adding slides.** The default canvas is `LAYOUT_16x9` = **10" × 5.625"**, not 13.3" wide. Coordinates past the edge are written, not clamped — the shape just isn't on the slide. (`LAYOUT_WIDE` is 13.3" × 7.5".)
  **在添加幻灯片之前先设置 `pres.layout`。** 默认画布是 `LAYOUT_16x9` = **10" × 5.625"**，宽度并非 13.3"。超出边界的坐标会被照实写入而不截断 — 形状只是不在幻灯片上而已。（`LAYOUT_WIDE` 为 13.3" × 7.5"。）
- **Hex colors: never `#`, never 8 digits.** `color: "FF0000"`. Both `"#FF0000"` and alpha baked into the hex (`"00000020"`) **corrupt the file**. For translucency: `transparency: 0-100` on fills and images, `opacity: 0.0-1.0` on shadows — each is silently ignored on the other.
  **十六进制颜色：永远不要带 `#`，也不要 8 位。** 应写作 `color: "FF0000"`。`"#FF0000"` 和把透明度并入十六进制（`"00000020"`）都会**损坏文件**。半透明效果：填充和图片用 `transparency: 0-100`，阴影用 `opacity: 0.0-1.0` — 用错位置会被静默忽略。
- **pptxgenjs mutates option objects in place** (converts values to EMU on first use). Never share one `shadow`/options object across two `add*` calls — build a fresh object each time.
  **pptxgenjs 会就地修改选项对象**（首次使用时把值转换为 EMU）。绝不要在两个 `add*` 调用之间共享同一个 `shadow`/选项对象 — 每次都新建对象。
- **Shadow `offset` must be ≥ 0** — a negative offset corrupts the file. To cast a shadow upward, use `angle: 270` with a positive offset.
  **阴影 `offset` 必须 ≥ 0** — 负值会损坏文件。若要阴影朝上，用 `angle: 270` 配正的 offset。
- **`letterSpacing` is silently ignored** — the real option is `charSpacing`.
  **`letterSpacing` 会被静默忽略** — 真正的选项是 `charSpacing`。
- **Lists:** `bullet: true` on each item, never a literal `•` (renders double bullets). Set `breakLine: true` on every array item except the last. Space bulleted paragraphs with `paraSpaceAfter`, not `lineSpacing` (huge gaps).
  **列表：** 每一项都要设 `bullet: true`，绝不要用字面 `•`（会渲染出双重项目符号）。除最后一项外，每个数组项都要设 `breakLine: true`。项目符号段落之间用 `paraSpaceAfter` 控制间距，不要用 `lineSpacing`（会产生巨大空隙）。
- **One `new pptxgen()` per output file** — never reuse an instance.
  **每个输出文件对应一个 `new pptxgen()`** — 绝不复用实例。
- **`rectRadius` only works on `ROUNDED_RECTANGLE`**, not `RECTANGLE`.
  **`rectRadius` 只对 `ROUNDED_RECTANGLE` 生效**，对 `RECTANGLE` 无效。
- **Gradient fills aren't supported** — use a gradient image as the background instead.
  **不支持渐变填充** — 请改用渐变图片作为背景。
- **Every `addText` call needs `isTextBox: true`** — without it the shape lacks `txBox="1"`, so screen readers announce the text as a "graphic" instead of a text box. No visual change.
  **每次 `addText` 调用都需要 `isTextBox: true`** — 否则形状缺少 `txBox="1"`，屏幕阅读器会把文字播报为"图形"而非文本框。视觉上没有变化。
- **Text boxes have built-in internal padding** — set `margin: 0` whenever text must align with a shape, line, or icon at the same x.
  **文本框自带内边距** — 只要文字需要与形状、线条或图标在同一 x 位置对齐，就设置 `margin: 0`。
- **Speaker notes go in `slide.addNotes("...")`** (plain text, once per slide), never in a text box on the slide.
  **演讲者备注写入 `slide.addNotes("...")`**（纯文本，每张幻灯片一次），绝不要放在幻灯片上的文本框里。
- **Keep charts native.** Use `addChart()` for everything PowerPoint can chart (pass an array of `{type, data, options}` for combos). For PowerPoint-native features the library doesn't expose (trendlines, error bars), compute the extra series yourself or post-process the generated OOXML — do not fall back to a rendered image. Only chart types PowerPoint has no native form for (Sankey, network, chord) go in as images.
  **图表保持原生。** PowerPoint 能绘制的图表一律用 `addChart()`（组合图传 `{type, data, options}` 数组）。对于库未暴露的 PowerPoint 原生特性（趋势线、误差线），自行计算额外数据序列或对生成的 OOXML 做后处理 — 不要退回使用渲染图片。只有 PowerPoint 没有原生形式的图表类型（桑基图、网络图、弦图）才以图片形式插入。
- **Default charts render bare** — no title, no data labels, dated palette. Set `showTitle` + `title`, `showValue: true` + `dataLabelPosition`, `chartColors: [...]` from your palette, and quiet the frame (`catAxisLabelColor`/`valAxisLabelColor`, `valGridLine: { color, size }`, `catGridLine: { style: "none" }`, `showLegend: false` for a single series).
  **默认图表渲染出来很朴素** — 无标题、无数据标签、配色过时。请设置 `showTitle` + `title`、`showValue: true` + `dataLabelPosition`、取自你调色板的 `chartColors: [...]`，并压低边框存在感（`catAxisLabelColor`/`valAxisLabelColor`、`valGridLine: { color, size }`、`catGridLine: { style: "none" }`、单序列时 `showLegend: false`）。
- **On a stacked bar or column chart, `dataLabelPosition` must be `ctr`, `inEnd`, or `inBase`.** `outEnd` **corrupts the file**.
  **在堆积条形图或柱状图上，`dataLabelPosition` 必须是 `ctr`、`inEnd` 或 `inBase`。** `outEnd` 会**损坏文件**。
- **A combo series using `secondaryValAxis`/`secondaryCatAxis` needs both `valAxes` and `catAxes` on the chart options, two entries each.** Without them pptxgenjs writes axis *ids* it never declares, and PowerPoint **discards that chart** and reports the file as corrupt. Supplying only `valAxes` is not enough.
  **使用 `secondaryValAxis`/`secondaryCatAxis` 的组合图序列需要在图表选项上同时提供 `valAxes` 和 `catAxes`，各两项。** 缺了它们，pptxgenjs 会写入从未声明的坐标轴 *id*，PowerPoint 会**丢弃该图表**并报告文件损坏。只提供 `valAxes` 是不够的。
- **After `writeFile()`, run `python scripts/office/validate.py deck.pptx`.** It reports the two chart faults above and the slide-XML defects PowerPoint refuses, and names the fix for each. Fix them in your generator, not by hand-editing the packed XML.
  **`writeFile()` 之后，运行 `python scripts/office/validate.py deck.pptx`。** 它会报告上述两类图表错误以及 PowerPoint 拒绝接受的幻灯片 XML 缺陷，并注明每项的修复方法。请在生成器中修复，不要手工编辑打包后的 XML。
- **Never reorder the children of `<p:presentation>`.** pptxgenjs writes `<p:notesMasterIdLst>` right after `<p:sldIdLst>` and points both masters at one theme part. PowerPoint reads that happily — move the element and the same deck becomes unopenable.
  **绝不重排 `<p:presentation>` 的子元素顺序。** pptxgenjs 会把 `<p:notesMasterIdLst>` 紧跟在 `<p:sldIdLst>` 之后写入，并让两个 master 指向同一个 theme 部分。PowerPoint 能正常读取这种结构 — 一旦移动该元素，同一份幻灯片组就会打不开。
- **Icons:** render `react-icons` to SVG (`ReactDOMServer.renderToStaticMarkup`), rasterize with `sharp` at ≥256px, and insert via `addImage({ data: "image/png;base64," + buf.toString("base64") })` — the `image/png;base64,` prefix is required (`react-icons`, `react`, `react-dom`, and `sharp` are preinstalled — `npm install react-icons react react-dom sharp` only if a require fails).
  **图标：** 把 `react-icons` 渲染为 SVG（`ReactDOMServer.renderToStaticMarkup`），用 `sharp` 以 ≥256px 栅格化，再通过 `addImage({ data: "image/png;base64," + buf.toString("base64") })` 插入 — `image/png;base64,` 前缀不可省略（`react-icons`、`react`、`react-dom` 和 `sharp` 均已预装 — 仅当 require 失败时才 `npm install react-icons react react-dom sharp`）。

## Editing existing decks and templates / 编辑既有幻灯片组与模板

Pick layouts first: `python scripts/thumbnail.py template.pptx template-thumbs` writes a labeled grid of every slide and prints the file(s) it created — `template-thumbs.jpg`, split into `template-thumbs-N.jpg` past 12 slides. **Always pass that second argument, named after the deck.** It defaults to `thumbnails`, so two decks thumbnailed in one directory silently overwrite each other's grids — the first deck's are simply gone (template analysis only — visual QA needs the full-resolution renders from [Converting to Images](#converting-to-images); it only accepts `.pptx`, so copy a `.potx` to a `.pptx` name first). Use it with `markitdown` to map each content section onto a template slide, and vary the layouts — don't put every section on the same title-and-bullets slide.

先挑选布局：`python scripts/thumbnail.py template.pptx template-thumbs` 会生成每张幻灯片的带标签网格图，并打印其创建的文件 — `template-thumbs.jpg`，超过 12 张幻灯片时拆分为 `template-thumbs-N.jpg`。**务必传第二个参数，并以幻灯片组命名。** 其默认值为 `thumbnails`，因此同一目录下对两个幻灯片组截图时，二者的网格会互相静默覆盖 — 第一份的直接丢失（仅限模板分析 — 视觉 QA 需用 [Converting to Images](#converting-to-images) 中的全分辨率渲染图；该命令只接受 `.pptx`，故需先把 `.potx` 复制为 `.pptx` 文件名）。将其与 `markitdown` 配合使用，把每个内容小节映射到模板幻灯片上，并变换布局 — 不要把每个小节都放在同一张"标题+要点"幻灯片上。

```bash
python3 -c "import sys,zipfile; zipfile.ZipFile(sys.argv[1]).extractall('unpacked')" deck.pptx
python scripts/add_slide.py unpacked/ slide2.xml --after slide2.xml   # duplicate a slide (or slideLayoutN.xml); prints the new slide's path
# reorder / delete slides = edit <p:sldIdLst> in ppt/presentation.xml
python scripts/clean.py unpacked/                                     # after deletions: removes orphaned slides, media, rels
# edit slide content in ppt/slides/slideN.xml
(cd unpacked && rm -f ../out.pptx && zip -Xr ../out.pptx .)           # zip from INSIDE the dir; rm first or deleted parts survive
python scripts/office/validate.py out.pptx --original deck.pptx
```

- **Do all structural work — add, delete, reorder — before editing any slide's content.** `add_slide.py` copies a slide file verbatim, so duplicating after you edit clones the edited content; and `clean.py` deletes any slide missing from `<p:sldIdLst>`, including one you just wrote.
  **先完成所有结构性操作 — 添加、删除、重排 — 再编辑任何幻灯片的内容。** `add_slide.py` 会原样复制幻灯片文件，因此编辑之后再复制会把修改一并克隆；而且 `clean.py` 会删除 `<p:sldIdLst>` 中不存在的任何幻灯片，包括你刚写好的那张。
- **Never copy a slide file by hand** — `add_slide.py` does every registration a new slide needs and reports what it made (`Created ppt/slides/slide17.xml from slide2.xml`). It also works directly on a file: `add_slide.py deck.pptx slide2.xml -o out.pptx` — **pass `-o`, or it rewrites the input deck in place.** A duplicated slide still *references* its source's chart/SmartArt/embedded-object parts rather than cloning them, so editing one slide's chart changes the other's.
  **绝不要手工复制幻灯片文件** — `add_slide.py` 会完成新幻灯片所需的全部登记并报告其产物（`Created ppt/slides/slide17.xml from slide2.xml`）。它也可以直接作用于文件：`add_slide.py deck.pptx slide2.xml -o out.pptx` — **务必传 `-o`，否则它会就地重写输入的幻灯片组。** 复制出的幻灯片对源幻灯片的图表/SmartArt/嵌入对象部分仍是*引用*而非克隆，因此编辑其中一张的图表会同时影响另一张。
- **If you use `python-pptx`**, three things it won't do: duplicate a slide (its only entry point is `add_slide(layout)`), preserve formatting through `text_frame.text = "..."` (that collapses the paragraph to a single unstyled run — assign `run.text` instead), or read the SVG/EMF most template art uses (`add_picture` raises `UnidentifiedImageError`).
  **如果你使用 `python-pptx`**，有三件事它做不到：复制幻灯片（其唯一入口是 `add_slide(layout)`）、通过 `text_frame.text = "..."` 保留格式（这会把段落折叠为单一无样式 run — 应改为对 `run.text` 赋值）、以及读取大多数模板插图所用的 SVG/EMF（`add_picture` 会抛出 `UnidentifiedImageError`）。
- Legacy `.ppt` must be converted first: `python scripts/office/soffice.py --headless --convert-to pptx file.ppt`. `.potx` templates unpack and pack identically — keep the `.potx` extension on the output.
  旧版 `.ppt` 必须先转换：`python scripts/office/soffice.py --headless --convert-to pptx file.ppt`。`.potx` 模板的解包与打包方式相同 — 输出请保留 `.potx` 扩展名。
- To reuse a template icon or image, duplicate a slide or layout that already contains it.
  要复用模板中的图标或图片，请复制已包含它的幻灯片或布局。

When filling in a template:

填充模板时：

- If you script an XML transform, parse with `defusedxml.minidom` — round-tripping OOXML through `xml.etree.ElementTree` rewrites namespace prefixes and corrupts the deck.
  如果用脚本做 XML 变换，请用 `defusedxml.minidom` 解析 — 用 `xml.etree.ElementTree` 往返处理 OOXML 会重写命名空间前缀并损坏幻灯片组。
- **Template slots ≠ source items.** If the template shows 4 team members and you have 3, delete the 4th member's entire group (image + text boxes), not just its text — then check for orphaned visuals in QA.
  **模板槽位 ≠ 素材条目。** 若模板展示 4 名团队成员而你只有 3 人，应删除第 4 名成员的整个组合（图片 + 文本框），而不只是其文字 — 然后在 QA 中检查有无孤立的视觉元素。
- One `<a:p>` per list item — never concatenate items into a single paragraph. Copy the sibling `<a:pPr>` to preserve spacing, and put `b="1"` on the `<a:rPr>` of titles, section headers, and inline labels (`Status:`, `Owner:`).
  每个列表项一个 `<a:p>` — 绝不要把多个条目拼接进单个段落。复制同级的 `<a:pPr>` 以保留间距，并在标题、小节标题和行内标签（`Status:`、`Owner:`）的 `<a:rPr>` 上加 `b="1"`。
- Let bullets inherit from the layout; only add `<a:buChar>`, `<a:buAutoNum>` (numbered), or `<a:buNone>` to override — never a literal `•` in the text.
  项目符号应从布局继承；仅在需要覆盖时添加 `<a:buChar>`、`<a:buAutoNum>`（编号）或 `<a:buNone>` — 绝不要在文本里写体面的 `•` 字符。
- Text with leading or trailing spaces needs `xml:space="preserve"` on its `<a:t>`.
  含前导或尾随空格的文本需要在 `<a:t>` 上加 `xml:space="preserve"`。

## Design Ideas / 设计思路

**Don't create boring slides.** Plain bullets on a white background won't impress anyone. Consider ideas from this list for each slide.

**不要做出乏味的幻灯片。** 白底上的普通要点列表打动不了任何人。每张幻灯片都请从本列表中借鉴点子。

### Before Starting / 开始之前

- **Pick a bold, content-informed color palette**: The palette should feel designed for THIS topic. If swapping your colors into a completely different presentation would still "work," you haven't made specific enough choices.
  **选择大胆、贴合内容的调色板**：调色板应让人感觉是专为"这个"主题设计的。如果把你的配色换到一份完全不同的演示里依然"成立"，说明你的选择还不够具体。
- **Dominance over equality**: One color should dominate (60-70% visual weight), with 1-2 supporting tones and one sharp accent. Never give all colors equal weight.
  **主次分明而非平均分配**：一种颜色应占主导（60-70% 的视觉权重），配 1-2 种辅助色调和一种醒目的点缀色。绝不要让所有颜色平分权重。
- **Dark/light contrast**: Dark backgrounds for title + conclusion slides, light for content ("sandwich" structure). Or commit to dark throughout for a premium feel.
  **深浅对比**：标题页和结尾页用深色背景，内容页用浅色（"三明治"结构）。或者全程使用深色以营造高级感。
- **Commit to a visual motif**: Pick ONE distinctive element and repeat it — rounded image frames, icons in colored circles. Carry it across every slide. **Do not use a color bar or accent stripe as your motif** (see Avoid list).
  **坚持一个视觉母题**：选定"一个"独特元素并反复使用 — 圆角图片框、彩色圆圈中的图标，并贯穿每一张幻灯片。**不要把色条或装饰条纹用作母题**（见"避免事项"列表）。

### Color Palettes / 配色方案

Choose colors that match your topic — don't default to generic blue. Use these palettes as inspiration:

选择与主题匹配的颜色 — 不要默认用千篇一律的蓝色。以下调色板可供参考：

| Theme | Primary | Secondary | Accent |
|-------|---------|-----------|--------|
| **Midnight Executive** | `1E2761` (navy) | `CADCFC` (ice blue) | `FFFFFF` (white) |
| **Forest & Moss** | `2C5F2D` (forest) | `97BC62` (moss) | `F5F5F5` (cream) |
| **Coral Energy** | `F96167` (coral) | `F9E795` (gold) | `2F3C7E` (navy) |
| **Warm Terracotta** | `B85042` (terracotta) | `E7E8D1` (sand) | `A7BEAE` (sage) |
| **Ocean Gradient** | `065A82` (deep blue) | `1C7293` (teal) | `21295C` (midnight) |
| **Charcoal Minimal** | `36454F` (charcoal) | `F2F2F2` (off-white) | `212121` (black) |
| **Teal Trust** | `028090` (teal) | `00A896` (seafoam) | `02C39A` (mint) |
| **Berry & Cream** | `6D2E46` (berry) | `A26769` (dusty rose) | `ECE2D0` (cream) |
| **Sage Calm** | `84B59F` (sage) | `69A297` (eucalyptus) | `50808E` (slate) |
| **Cherry Bold** | `990011` (cherry) | `FCF6F5` (off-white) | `2F3C7E` (navy) |

| 主题 | 主色 | 次色 | 点缀色 |
|-------|---------|-----------|--------|
| **Midnight Executive（午夜行政）** | `1E2761`（海军蓝） | `CADCFC`（冰蓝） | `FFFFFF`（白） |
| **Forest & Moss（森林与苔藓）** | `2C5F2D`（森林绿） | `97BC62`（苔藓绿） | `F5F5F5`（奶油白） |
| **Coral Energy（珊瑚活力）** | `F96167`（珊瑚红） | `F9E795`（金黄） | `2F3C7E`（海军蓝） |
| **Warm Terracotta（暖陶土）** | `B85042`（陶土红） | `E7E8D1`（沙色） | `A7BEAE`（鼠尾草绿） |
| **Ocean Gradient（海洋渐变）** | `065A82`（深海蓝） | `1C7293`（青绿） | `21295C`（午夜蓝） |
| **Charcoal Minimal（炭黑极简）** | `36454F`（炭灰） | `F2F2F2`（灰白） | `212121`（黑） |
| **Teal Trust（青绿信赖）** | `028090`（青绿） | `00A896`（海泡绿） | `02C39A`（薄荷绿） |
| **Berry & Cream（浆果奶油）** | `6D2E46`（浆果紫红） | `A26769`（灰玫瑰） | `ECE2D0`（奶油白） |
| **Sage Calm（鼠尾草宁静）** | `84B59F`（鼠尾草绿） | `69A297`（桉叶绿） | `50808E`（岩板灰） |
| **Cherry Bold（樱桃浓烈）** | `990011`（樱桃红） | `FCF6F5`（灰白） | `2F3C7E`（海军蓝） |

### For Each Slide / 每张幻灯片

**Every slide needs a visual element** — image, chart, icon, or shape. Text-only slides are forgettable.

**每张幻灯片都需要一个视觉元素** — 图片、图表、图标或形状。纯文字的幻灯片让人过目即忘。

**Layout options:**

**布局选项：**

- Two-column (text left, illustration on right)
  双栏（左侧文字，右侧插图）
- Icon + text rows (icon in colored circle, bold header, description below)
  图标 + 文字行（彩色圆圈中的图标、加粗标题、下方描述）
- 2x2 or 2x3 grid (image on one side, grid of content blocks on other)
  2x2 或 2x3 网格（一侧放图片，另一侧放内容块网格）
- Half-bleed image (full left or right side) with content overlay
  半出血图片（占满左侧或右侧），内容叠加其上

**Data display:**

**数据展示：**

- Large stat callouts (big numbers 60-72pt with small labels below)
  大号数据标注（60-72pt 的大数字，下方配小标签）
- Comparison columns (before/after, pros/cons, side-by-side options)
  对比栏（前后对比、优缺点对比、并排选项）
- Timeline or process flow (numbered steps, arrows)
  时间线或流程图（编号步骤、箭头）

**Visual polish:**

**视觉打磨：**

- Icons in small colored circles next to section headers
  小节标题旁放彩色小圆圈中的图标
- Italic accent text for key stats or taglines
  关键数据或标语用斜体点缀文字

### Typography / 字体排印

**Font names you write into the .pptx are rendered by the user's PowerPoint, not by this environment.** Your visual QA renders via LibreOffice, which substitutes fonts it doesn't have — and for some fonts the substitute has different widths, so your QA preview can show text overflow (or fit) that the real deck won't have. To keep your QA trustworthy:

**你写进 .pptx 的字体名称由用户的 PowerPoint 渲染，而不是由本环境渲染。** 你的视觉 QA 通过 LibreOffice 渲染，它会对没有的字体做替换 — 而某些字体的替换字体宽度不同，因此 QA 预览可能显示出真实幻灯片中并不存在（或相反情形）的文字溢出。为了让 QA 可信：

- **Safe fonts** (render true-to-width in QA *and* ship with Office): **Arial, Calibri, Cambria, Times New Roman, Courier New, Bookman Old Style, Century Schoolbook**. Use these for body text and anything where fit matters.
  **安全字体**（在 QA 中宽度保真*且*随 Office 附带）：**Arial, Calibri, Cambria, Times New Roman, Courier New, Bookman Old Style, Century Schoolbook**。正文以及任何对排版适配敏感的内容都使用这些字体。
- **Headers with personality at zero QA risk**: pair a safe-list serif header (Cambria, Bookman Old Style, Century Schoolbook) with a safe-list sans body (Calibri or Arial). You get visual contrast without giving up reliable overflow checks.
  **零 QA 风险又有个性的标题**：用安全列表中的衬线标题（Cambria、Bookman Old Style、Century Schoolbook）搭配安全列表中的无衬线正文（Calibri 或 Arial）。既有视觉对比，又不放弃可靠的溢出检查。
- **If the user asks for a font outside the safe list** (e.g. Georgia or Trebuchet MS): use it where the user asked, but size those containers with extra slack (~10%) and don't trust QA text-fit on those elements — the preview of that font is approximate. If the user hasn't specified, prefer safe-list fonts for body text.
  **如果用户要求安全列表之外的字体**（如 Georgia 或 Trebuchet MS）：在用户要求之处照用，但为这些容器预留额外余量（约 10%），且不要相信 QA 对这些元素的文字适配判断 — 该字体的预览只是近似。用户未指定时，正文优先使用安全列表字体。
- **QA-unreliable fonts** (substitute has different widths — overflow checks can be wrong): Georgia, Trebuchet MS, Impact, Arial Black, Garamond, Consolas, Palatino Linotype. Calibri Light substitution varies by environment; treat as QA-unreliable. Fine for titles/accents with slack; don't trust QA text-fit on these.
  **QA 不可靠字体**（替换字体宽度不同 — 溢出检查可能出错）：Georgia、Trebuchet MS、Impact、Arial Black、Garamond、Consolas、Palatino Linotype。Calibri Light 的替换情况因环境而异，同样视为 QA 不可靠。预留余量后用于标题/点缀没有问题；但不要相信 QA 对它们的文字适配判断。
- **Never default to Aptos** — Office's post-2023 default has no metric-compatible substitute here *and* is missing from older Office installs, so it's unreliable on both ends.
  **绝不要默认使用 Aptos** — Office 2023 年之后的默认字体在此环境没有度量兼容的替换字体，*而且*旧版 Office 安装中也没有它，两端都不可靠。

| Element | Size |
|---------|------|
| Slide title | 36-44pt bold |
| Section header | 20-24pt bold |
| Body text | 14-16pt |
| Captions | 10-12pt muted |

| 元素 | 字号 |
|---------|------|
| 幻灯片标题 | 36-44pt 加粗 |
| 小节标题 | 20-24pt 加粗 |
| 正文 | 14-16pt |
| 说明文字 | 10-12pt 弱化色 |

### Spacing / 间距

- 0.5" minimum margins
  边距至少 0.5"
- 0.3-0.5" between content blocks
  内容块之间间隔 0.3-0.5"
- Leave breathing room—don't fill every inch
  留出呼吸空间 — 不要塞满每一寸

### Avoid (Common Mistakes) / 避免事项（常见错误）

- **Don't repeat the same layout** — vary columns, cards, and callouts across slides
  **不要重复同一布局** — 在各幻灯片间变换栏式、卡片和标注
- **Don't center body text** — left-align paragraphs and lists; center only titles
  **不要居中正文** — 段落和列表左对齐；只有标题才居中
- **Don't skimp on size contrast** — titles need 36pt+ to stand out from 14-16pt body
  **不要吝啬字号对比** — 标题需要 36pt 以上才能从 14-16pt 的正文中凸显出来
- **Don't default to blue** — pick colors that reflect the specific topic
  **不要默认用蓝色** — 选择能反映具体主题的颜色
- **Don't mix spacing randomly** — choose 0.3" or 0.5" gaps and use consistently
  **不要随意混用间距** — 选定 0.3" 或 0.5" 的间隔并统一使用
- **Don't style one slide and leave the rest plain** — commit fully or keep it simple throughout
  **不要只装饰一张幻灯片而让其余保持朴素** — 要么全程贯彻风格，要么全程从简
- **Don't create text-only slides** — add images, icons, charts, or visual elements; avoid plain title + bullets
  **不要做纯文字幻灯片** — 加入图片、图标、图表或其他视觉元素；避免朴素的"标题 + 要点"
- **Don't forget text box padding** — when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
  **不要忘记文本框内边距** — 将线条或形状与文字边缘对齐时，在文本框上设 `margin: 0`，或为形状设置偏移以补偿内边距
- **Don't use low-contrast elements** — icons AND text need strong contrast against the background; avoid light text on light backgrounds or dark text on dark backgrounds
  **不要使用低对比元素** — 图标和文字都需要与背景形成强对比；避免浅底浅字或深底深字
- **NEVER use accent lines under titles** — these are a hallmark of AI-generated slides; use whitespace or background color instead
  **绝不要在标题下加装饰线** — 这是 AI 生成幻灯片的典型特征；改用留白或背景色

【评论】把"标题下加线"视为 AI 生成痕迹并明令禁止，是针对 AI 产物审美同质化的反向设计考量，说明该技能同时承担"生成"与"去 AI 味"两个目标。

- **NEVER add decorative color bars or accent stripes** — this includes: header/footer bars spanning the slide width, vertical sidebar stripes down one edge of the slide, thin accent stripes along one edge of a card or content block, and "single-side borders" on rectangles. These read as AI-generated filler. If you want to set a card apart, use a subtle background tint, a drop shadow, or an icon — not an edge stripe.
  **绝不要添加装饰性色条或点缀条纹** — 包括：横贯幻灯片宽度的页眉/页脚色条、沿幻灯片一边的竖向侧栏条纹、沿卡片或内容块一边的细点缀条纹，以及矩形的"单侧边框"。这些都会被看作 AI 生成的填充物。想让卡片突出，可用淡雅的底色、投影或图标 — 而不是边缘条纹。
- **Don't default to cream/beige backgrounds** — when no background is specified, use white (`FFFFFF`) or the user's brand palette; avoid warm-neutral defaults like `F5F5DC`, `FAF0E6`, `FAEBD7`, `FFF8E1`
  **不要默认使用奶油色/米色背景** — 未指定背景时使用白色（`FFFFFF`）或用户品牌色；避免 `F5F5DC`、`FAF0E6`、`FAEBD7`、`FFF8E1` 这类暖中性色默认值
- **Don't ship text that overflows its shape** — if text doesn't fit, reduce font size, split across slides, or enlarge the container; never leave content cut off or spilling past bounds
  **不要让交付的文本溢出其形状** — 文字放不下就缩小字号、拆分到多张幻灯片或扩大容器；绝不要留下被截断或超出边界的内容

## QA (Required) / QA（必需）

Your first render usually has a few real issues — overlaps, overflow, misalignment. Find and fix those, re-render only the slides you changed, and stop.

首次渲染通常都会有几个真实问题 — 重叠、溢出、错位。找出并修复它们，只重新渲染你改过的幻灯片，然后停止。

### Content QA / 内容 QA

```bash
markitdown output.pptx
```

Check for missing content, typos, wrong order.

检查内容缺失、错别字和顺序错误。

**When using templates, check for leftover placeholder text:**

**使用模板时，检查是否残留占位文本：**

```bash
markitdown output.pptx | grep -iE "\bx{3,}\b|lorem|ipsum|\bTODO|\[insert|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

如果 grep 有结果，先修复再宣布完成。

### File QA (required) / 文件 QA（必需）

```bash
python scripts/office/validate.py output.pptx                      # built from scratch
python scripts/office/validate.py output.pptx --original src.pptx  # built from a template
```

**If the deck came from a template, always pass `--original`.** A template may itself
contain parts the XSD rejects, so a bare run can report failures you never caused — and
a genuine regression can hide among them. `--original` baselines
the schema and slide checks against the template, suppressing errors it already had.
The structural checks — relationships, content types, charts — ignore `--original` and
report template-inherited problems either way, so read those on their own merits.

**如果幻灯片组来自模板，务必传 `--original`。** 模板自身可能含有 XSD 拒绝的部分，因此不加参数运行时报告的失败可能并非你造成 — 而真正的回归问题也可能藏在其中。`--original` 以模板为基线执行架构与幻灯片检查，抑制模板本来就有的错误。结构检查（关系、内容类型、图表）不受 `--original` 影响，无论是否传参都会报告模板遗留的问题，因此需要单独审视这些报告。

pptxgenjs emits chart XML PowerPoint refuses to open, and every other tool
accepts: python-pptx opens those decks, LibreOffice renders them, the XSD
passes them. Every failure names its fix. Fix it in the generator and rebuild.

pptxgenjs 生成的图表 XML 会被 PowerPoint 拒绝打开，而其他所有工具都接受：python-pptx 能打开这类幻灯片组，LibreOffice 能渲染，XSD 校验也能通过。每项失败都会注明修复方法。请在生成器中修复并重新构建。

### Visual QA / 视觉 QA

Convert the slides to images (see [Converting to Images](#converting-to-images)) and inspect every one. After staring at the generating code you tend to see what you expect rather than what rendered, so look at the images fresh (a subagent works well for this if you have one). User-visible defects to look for:

把幻灯片转换为图片（见 [Converting to Images](#converting-to-images)）并逐张检查。盯着生成代码看久了，你往往看到的是自己预期的东西而非实际渲染的结果，因此要以全新眼光查看图片（如果有子代理，用它来做这件事很合适）。需要查找的用户可见缺陷：

【评论】建议用子代理以"新眼光"检查渲染结果，是对抗自产自检确认偏差的一种机制设计。

- **Text overflow or text cut off at a box or slide boundary — check this first.** It is the most common defect and always user-visible. (For a font the previewer renders unreliably per Typography, the preview is approximate: trust the ~10% slack you left, not its apparent fit.)
  **文本溢出或在文本框/幻灯片边界处被截断 — 首先检查这一项。** 这是最常见且必然被用户看到的缺陷。（对按"字体排印"一节所述预览不可靠的字体，预览只是近似：相信你预留的约 10% 余量，而不要相信表面上的适配效果。）
- Overlapping elements (text through shapes, lines through words, stacked elements)
  元素重叠（文字穿过形状、线条穿过文字、元素堆叠）
- Source citations or footers colliding with content above
  来源引用或页脚与上方内容相撞
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
  元素间距过近（< 0.3"）或卡片/小节几乎相触
- Uneven gaps (large empty area in one place, cramped in another)
  间隔不均（一处大片空白，另一处却很拥挤）
- Insufficient margin from slide edges (< 0.5")
  距幻灯片边缘的边距不足（< 0.5"）
- Columns or similar elements not aligned consistently
  栏或类似元素对齐不一致
- Low-contrast text (e.g., light gray text on cream-colored background)
  低对比文字（如奶油色背景上的浅灰文字）
- Template decoration mispositioned after text replacement — e.g., a title underline positioned for one line, but the replaced title wrapped to two
  替换文字后模板装饰错位 — 例如标题下划线按一行定位，而替换后的标题折成了两行
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
  低对比图标（如深色背景上的深色图标又没有对比色圆圈衬托）
- Text boxes too narrow causing excessive wrapping
  文本框过窄导致过度折行
- Leftover placeholder content
  残留的占位内容

## Converting to Images / 转换为图片

Convert presentations to individual slide images for visual inspection:

将演示文稿转换为单张幻灯片图片以便进行视觉检查：

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
rm -f slide-*.jpg
pdftoppm -jpeg -r 150 output.pdf slide
ls -1 "$PWD"/slide-*.jpg
```

**Pass the absolute paths printed above directly to the view tool.** The `rm` clears stale images from prior runs. `pdftoppm` zero-pads based on page count: `slide-1.jpg` for decks under 10 pages, `slide-01.jpg` for 10-99, `slide-001.jpg` for 100+.

**把上面打印出的绝对路径直接传给查看工具。** `rm` 用于清除先前运行留下的旧图片。`pdftoppm` 按页数补零：10 页以下为 `slide-1.jpg`，10-99 页为 `slide-01.jpg`，100 页以上为 `slide-001.jpg`。

**After fixes, rerun all four commands above** — the PDF must be regenerated from the edited `.pptx` before `pdftoppm` can reflect your changes.

**修复之后，重新运行上述全部四条命令** — 必须先从编辑后的 `.pptx` 重新生成 PDF，`pdftoppm` 才能反映你的修改。

## Dependencies / 依赖

`pptxgenjs` (npm, preinstalled — install only if `require('pptxgenjs')` fails) · `markitdown[pptx]`, `Pillow`, `defusedxml`, `lxml` (pip — text dump, thumbnail, clean, validate) · LibreOffice (`soffice`, auto-configured for sandboxed environments via `scripts/office/soffice.py`) · `pdftoppm` (Poppler)

`pptxgenjs`（npm，已预装 — 仅当 `require('pptxgenjs')` 失败时才安装）· `markitdown[pptx]`、`Pillow`、`defusedxml`、`lxml`（pip — 文本导出、缩略图、清理、校验）· LibreOffice（`soffice`，通过 `scripts/office/soffice.py` 为沙箱环境自动配置）· `pdftoppm`（Poppler）
