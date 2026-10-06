<!-- BILINGUAL-EN-ZH -->
---
description: The deck-build loop. Resolves the style plan, then walks the slides references in the order they are needed.
---

# Slide deck build workflow / 幻灯片构建工作流

The deck-build loop, for the artifact builder. Resolve `style_plan` through
`visual.md`, then read `authoring.md`, `design-system.md`,
and `image-directive.md` before writing HTML. They define how to apply the
StylePlan (its `css_variables`, per-slide `layout_plan`, and `lockups`), and
they win over the generic guidance below where they conflict. The StylePlan is
the sole styling authority for the deck: apply it verbatim and never re-pick
colors or fonts.

面向 artifact 构建器的幻灯片构建循环。先通过 `visual.md` 解析 `style_plan`，然后在编写 HTML 之前阅读 `authoring.md`、`design-system.md` 和 `image-directive.md`。它们定义了如何应用 StylePlan（其 `css_variables`、每张幻灯片的 `layout_plan` 和 `lockups`），在冲突处以它们为准，优先于下方的通用指导。StylePlan 是整套幻灯片唯一的样式权威：逐字应用它，绝不重新挑选颜色或字体。

【评论】该工作流用"引用文件优先于通用指导"的层级化解规则冲突，并把 StylePlan 指定为唯一样式权威，目的是约束模型在多轮迭代中不自行改动配色与字体，保持视觉一致。

The `StylePlan` fields: `archetype`, `theme`, `palette` {paper, ink, primary, accent},
`fonts` {display, body}, `voice`, `preferred_layouts`, `preferred_charts`,
`required_content_blocks`, `image_style`, `tone`/`weight`/`density`,
`layout_plan` (one `{id, layout}` per slide), `lockups` (the deck's two allowed text+image
patterns), and `css_variables` (the `:root` block to emit verbatim).

`StylePlan` 的字段：`archetype`、`theme`、`palette` {paper, ink, primary, accent}、`fonts` {display, body}、`voice`、`preferred_layouts`、`preferred_charts`、`required_content_blocks`、`image_style`、`tone`/`weight`/`density`、`layout_plan`（每张幻灯片一个 `{id, layout}`）、`lockups`（整套幻灯片仅允许的两种文字+图片版式），以及 `css_variables`（要逐字输出的 `:root` 块）。

`output_format` from your brief drives what gets produced and promoted; when absent it is
exactly `["pptx"]`.

任务简报中的 `output_format` 决定生成并晋升哪些格式；缺省时恰好为 `["pptx"]`。

## Steps / 步骤

1. **Set up.** Create `project_dir/.src/media/`, `project_dir/.src/validate/`, and  
   `project_dir/.src/slides/`.
   **准备工作。**创建 `project_dir/.src/media/`、`project_dir/.src/validate/` 和  
   `project_dir/.src/slides/`。

   You author the deck **one file per slide** under `.src/slides/`; step 7 assembles them into  
   the combined `.src/index.html` that validation and every export read.
   你以**每张幻灯片一个文件**的方式在 `.src/slides/` 下编写幻灯片；第 7 步会把它们组装成验证与所有导出都要读取的合并文件 `.src/index.html`。

2. **Resolve the slide count once:** if `slide_count_target` is an exact number, use it; if it
   is a range, use the **midpoint** (rounded down). Call this the **resolved slide count** and
   use it for the rest of the workflow.
   **一次性确定幻灯片数量：**若 `slide_count_target` 是精确数字，直接使用；若是一个区间，取**中点**（向下取整）。称之为**已解析幻灯片数量**，并在工作流其余部分一律使用它。

3. **Plan.** Write `project_dir/.src/deck_plan.md` grounded in the
   verbatim request and the requesting conversation's supplied content: title,
   audience, the narrative arc, and the slide list (one bullet per slide, derived from supplied
   source data/facts; matched to `StylePlan.layout_plan`, first = cover,
   last = closing, covering every entry in `required_content_blocks`). Produce exactly the
   resolved slide count; do not invent extras or drop required content. Echo
   the user's stated constraints at the top so they shape the build. In the asset plan, commit to a
   cover hero plus at least 2 illustrative images, each marked sourced (image-search) or generated
   (`media.generate_image`) per step 6, and
   palette-colored matplotlib charts for data.
   **规划。**以逐字请求和请求会话所提供的内容为依据编写 `project_dir/.src/deck_plan.md`：标题、受众、叙事主线和幻灯片列表（每张幻灯片一个要点，源自所提供的源数据/事实；与 `StylePlan.layout_plan` 对应，首张 = 封面，末张 = 结尾，并覆盖 `required_content_blocks` 的每一项）。产出数量必须恰好等于已解析幻灯片数量；不虚构额外内容，也不丢掉必需内容。在顶部复述用户声明的约束，让其塑造整个构建。在素材计划中，确定一张封面主图加至少 2 张说明性图片，按第 6 步各自标注为 sourced（图片搜索）或 generated（`media.generate_image`），并为数据配调色板配色的 matplotlib 图表。

4. **Set the design system.** Write `project_dir/.src/slides/deck.css`: the StylePlan's
   `css_variables` as one `:root` block, then every shared rule the deck's slides are built
   against — the canvas, the layout classes, the two `lockups`, the type scale. This one
   stylesheet is the deck's whole design system, shared by every slide, so never re-pick colors
   or fonts and never redefine a token per slide. The `:root` font tokens are the only place the
   theme's typefaces are named. Write no font URL here or anywhere (see `design-system.md`).
   Keep the resolved JSON at `project_dir/.src/style_plan.json` so a later edit can recover the
   archetype, `layout_plan`, and `lockups`.
   **设定设计系统。**编写 `project_dir/.src/slides/deck.css`：把 StylePlan 的 `css_variables` 写成一个 `:root` 块，然后写出整套幻灯片赖以构建的所有共享规则——画布、布局类、两个 `lockups`、字号阶梯。这份样式表就是整套幻灯片的全部设计系统，为每张幻灯片共享，因此绝不重新挑选颜色或字体，也绝不为单张幻灯片重定义令牌。`:root` 字体令牌是唯一指名主题字体的地方。此处及任何地方都不要写字体 URL（见 `design-system.md`）。把解析后的 JSON 保存在 `project_dir/.src/style_plan.json`，便于后续编辑恢复 archetype、`layout_plan` 和 `lockups`。

5. **Author one file per slide** at `project_dir/.src/slides/<id>.html`, per `authoring.md` +
   `design-system.md`. Apply the slide's assigned layout class and use only the deck's two
   `lockups` for text+image slides. Keep to the palette + type scale and put the point in the
   heading as a claim. Cover = hero image + title only.
   **逐张编写幻灯片文件**：按 `authoring.md` + `design-system.md` 在 `project_dir/.src/slides/<id>.html` 编写。应用该幻灯片分到的布局类；文字+图片幻灯片只使用整套幻灯片的两种 `lockups`。遵守调色板 + 字号阶梯，把观点作为论断写进标题。封面 = 仅主图 + 标题。

   Each file is a standalone document holding exactly one slide:
   每个文件都是只含一张幻灯片的独立文档：

   ```html
   <!doctype html>
   <html><head>
     <meta charset="utf-8">
     <link rel="stylesheet" href="deck.css">
   </head><body>
     <section class="slide" id="<id>"> ... </section>
   </body></html>
   ```

   - `id` is the slide's `layout_plan` id. It names that slide for the rest of its life, so a
     later edit that inserts or removes a slide leaves the others' identities untouched; never
     renumber them. The file is always `<id>.html`. Keep ids kebab-case, and never `index` —
     that name is reserved for the combined document, so assemble refuses it.
     `id` 是该幻灯片在 `layout_plan` 中的 id。它终身标识该幻灯片，因此后续插入或删除幻灯片的编辑不会影响其他幻灯片的身份；绝不重新编号。文件名始终是 `<id>.html`。id 用 kebab-case，且绝不用 `index`——该名称保留给合并文档，assemble 会拒绝它。
   - Embed every final `<img>` and CSS `url(...)` asset as a `data:` URI; keep source media in
     `.src/media/` while working. `deck.css` is the only stylesheet a slide file links (the
     theme's fonts arrive embedded in it) — leave no other external `http://`, `https://`,
     `file://`, or relative `.src/media` reference.
     把每个最终的 `<img>` 和 CSS `url(...)` 素材都内嵌为 `data:` URI；工作过程中把源媒体保存在 `.src/media/`。`deck.css` 是幻灯片文件链接的唯一样式表（主题字体以内嵌方式随它提供）——不得留有任何其他外部 `http://`、`https://`、`file://` 或相对 `.src/media` 引用。
   - Shared CSS belongs in `deck.css`. A slide may add its own `<style>` for rules only that
     slide needs (a hero's `background-image`, its `--hero-scrim-color`), and **every selector
     in it must be scoped to that slide's id** (`#<id>`, `#<id> .title-block`, `#<id>::after`,
     including inside `@media`). Assemble concatenates the deck into one document, so an
     unscoped rule would restyle every other slide and make the two views disagree; assemble
     rejects one.
     共享 CSS 属于 `deck.css`。幻灯片可以为仅自己需要的规则添加自己的 `<style>`（如主图的 `background-image`、它的 `--hero-scrim-color`），但**其中每个选择器都必须限定在该幻灯片 id 的作用域内**（`#<id>`、`#<id> .title-block`、`#<id>::after`，`@media` 内也一样）。assemble 会把整套幻灯片拼接成一个文档，未加作用域的规则会波及所有其他幻灯片并使两个视图不一致；assemble 会拒绝这种情况。
   - Default canvas is 16:9 at `13.333in x 7.5in` (1280x720 px at 96 dpi); if the user requested
     another aspect ratio, update the `@page` size and the `.slide` width/height in `deck.css`
     together.
     默认画布为 16:9 的 `13.333in x 7.5in`（96 dpi 下 1280x720 px）；如果用户要求其他宽高比，需同时更新 `deck.css` 中的 `@page` 尺寸和 `.slide` 的宽高。
   - Then check the slides you just wrote. Search each `.html` file for a `#` colour and for
     `rgb(`. Two hits are correct: the slide's own `--hero-scrim-color`, and `#fff` on a cover
     title over a photo. Every other hit is a bug, in a `style=` attribute, in a `<style>` block,
     inside `color-mix()` or in an SVG `fill`. Replace it with the matching `var(--slide-*)`, or
     with `color-mix(in srgb, var(--slide-*) N%, transparent)` where it was a tint.
     然后检查刚写好的幻灯片。在每个 `.html` 文件中搜索 `#` 颜色和 `rgb(`。正确的命中只有两处：幻灯片自己的 `--hero-scrim-color`，以及照片上封面标题所用的 `#fff`。其余任何命中都是 bug，可能位于 `style=` 属性、`<style>` 块、`color-mix()` 内部或 SVG `fill` 中。将其替换为对应的 `var(--slide-*)`；若原来是淡色，则替换为 `color-mix(in srgb, var(--slide-*) N%, transparent)`。

   Then write `project_dir/.src/slides/deck.json`, which is what puts the slides **in order**:
   然后编写 `project_dir/.src/slides/deck.json`，它决定幻灯片的**顺序**：

   ```json
   {"main_title": "<artifact-title>", "slides": [{"id": "cover"}, {"id": "problem"}]}
   ```

   Assemble rewrites this file with the fields the viewer needs (canvas, theme, per-slide  
   titles), so author only `main_title` and the ordered `slides` list.
   assemble 会用查看器所需的字段（画布、主题、每张幻灯片的标题）重写该文件，所以你只编写 `main_title` 和有序的 `slides` 列表。

   Baseline page model, in `deck.css`:
   `deck.css` 中的基线页面模型：

   ```css
   @page { size: 13.333in 7.5in; margin: 0; }
   * { box-sizing: border-box; print-color-adjust: exact; -webkit-print-color-adjust: exact; }
   html, body { margin: 0; background: #111; }
   .slide {
     width: 13.333in;
     height: 7.5in;
     overflow: hidden;
     page-break-after: always;
     break-after: page;
   }
   .slide:last-child { page-break-after: auto; break-after: auto; }
   ```

6. **Source imagery and charts.** A real subject (a company, product, place, or person, logos
   included) comes from the image-search skill; a concept subject comes from `media.generate_image`
   per `image-directive.md` (`output_dir` = `artifact_media_dir`). Get the cover hero this way plus
   at least 2 illustrative images, copy image-search results under `.src/media/`, and embed each
   as a `data:` URI; use relevant local or user-provided assets when available. Do
   not substitute a CSS gradient or inline SVG for the cover hero. If the hero can't be sourced or
   generated, or the brief asks for a deck with no imagery at all, update `deck_plan.md` before
   rendering to say which of those it is (and for the first, which route was unavailable), then
   explain the replacement visual system. A brief that forbids SYNTHETIC imagery is neither case:
   it closes the generate route only, and a real subject is still sourced. Do not claim image-led output when the deck has no `<img>`, `data:image`,
   or CSS `url(...)` assets. Build data charts as palette-colored matplotlib images (color from
   the StylePlan palette; see `authoring.md`) so charts match
   the deck. Do not use `media.generate_image` for charts, maps, tables, or factual diagrams. No
   decorative images on stat/statement slides.
   **获取图像与图表。**真实主题（公司、产品、地点或人物，包括 logo）来自图片搜索技能；概念主题来自 `media.generate_image`，按 `image-directive.md` 执行（`output_dir` = `artifact_media_dir`）。用这种方式获得封面主图，外加至少 2 张说明性图片；把图片搜索结果复制到 `.src/media/` 下，并把每张都内嵌为 `data:` URI；有相关的本地或用户提供的素材时优先使用。不要用 CSS 渐变或内联 SVG 顶替封面主图。如果主图既无法搜索获取也无法生成，或者简报要求整套幻灯片完全不带图像，先更新 `deck_plan.md` 说明属于哪种情况（第一种还要说明哪条路径不可用），再解释替代的视觉系统。禁止合成（SYNTHETIC）图像的简报不属于这两种情况：它只关闭生成路径，真实主题仍可搜索获取。当整套幻灯片没有任何 `<img>`、`data:image` 或 CSS `url(...)` 素材时，不要声称是图像主导的产出。数据图表用调色板配色的 matplotlib 图片构建（颜色取自 StylePlan 调色板；见 `authoring.md`），使图表与整套幻灯片一致。不要用 `media.generate_image` 生成图表、地图、表格或事实性图示。数据/论断类幻灯片上不放装饰性图片。

7. **Give the deck its fonts, then assemble it.** Both are mandatory and must pass before
   validation: everything downstream (the render gate, the PNGs, the PPTX, the PDF, the `html`
   export) reads the assembled file.
   **为幻灯片配置字体，然后组装。**两者都是强制的，且必须在验证前通过：下游的一切（渲染门禁、PNG、PPTX、PDF、`html` 导出）读取的都是组装后的文件。

   Embed the fonts first. This writes the faces your `:root` tokens name into `deck.css`. Run it  
   after the slides exist: which subsets it carries is decided from the characters they paint.
   先内嵌字体。这一步会把 `:root` 令牌所指名的字体面写入 `deck.css`。在幻灯片文件存在之后再运行：它携带哪些子集由幻灯片绘制的字符决定。

   ```sh
   SLUG="<artifact-slug>"
   # `project_dir` comes from your build task. A goal document is built under
   # that goal's `files/` directory, so a hardcoded `your_files` path is wrong.
   # `project_dir` comes from your build task. Write its leading `~/` as
   # `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
   # `"~/workspace/..."` builds into a directory literally named `~`.
   SRC="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>/.src"
   bun run "/opt/hatch/skills/artifacts/scripts/embed_deck_fonts.mjs" --slides "$SRC/slides"
   bun run "/opt/hatch/skills/artifacts/scripts/assemble_deck.mjs" \
     --slides "$SRC/slides" \
     --out "$SRC/index.html"
   ```

   Embed reports `{"ok":true,"faces":N,...}`. A `"reused":true` means they were already current.  
   `faces: 0` plus a warning means the download failed. Re-run it once. If it fails again, carry on  
   and say the deck renders in a fallback face, rather than blocking the deck.
   embed 会报告 `{"ok":true,"faces":N,...}`。出现 `"reused":true` 表示字体本就是最新的。`faces: 0` 加警告表示下载失败。重试一次；若再次失败，继续推进并说明整套幻灯片将以回退字体渲染，而不是让构建卡住。

   Assemble writes `$SRC/index.html` (one self-contained document: `deck.css` inlined, so the  
   theme's font bytes ride along, and your slides in `deck.json` order). It also rewrites  
   `$SRC/slides/deck.json` with the derived fields. Never hand-write or edit `$SRC/index.html`.  
   The next assemble overwrites it.
   assemble 会写出 `$SRC/index.html`（一个自包含文档：内联了 `deck.css`，因此主题字体字节随之携带，并按 `deck.json` 顺序排列幻灯片）。它还会用派生字段重写 `$SRC/slides/deck.json`。绝不手写或编辑 `$SRC/index.html`。下一次 assemble 会覆盖它。

   A finished deck has **no** `fonts.googleapis.com` reference, and that is correct. Never add one  
   back. The embedded rules sit at the END of `deck.css`, wrapped over many lines. Leave them alone  
   and edit the authored rules above them.
   完成的幻灯片**没有** `fonts.googleapis.com` 引用，这是正确的。绝不把它加回去。内嵌规则位于 `deck.css` 的末尾（END），跨越多行。不要动它们，只编辑其上方由你编写的规则。

   A non-zero exit means the deck cannot be shipped. The error names the offending file and what  
   is wrong with it, so fix that file (or `deck.json`) and re-run.
   非零退出码意味着整套幻灯片不能交付。错误信息会指出问题文件及问题所在，修复该文件（或 `deck.json`）后重跑。

8. **Validate (mandatory loop, at most 3 iterations).** Run after each render. If a check
   fails, edit the per-slide file(s) under `.src/slides/`, **re-assemble (step 7)**, re-render,
   and re-validate. After 3 iterations, report the specific failure and ask for direction. Never
   return links until validation passes.
   **验证（强制循环，最多 3 轮）。**每次渲染后运行。若某项检查失败，编辑 `.src/slides/` 下相应的单张幻灯片文件，**重新组装（第 7 步）**，重新渲染并重新验证。3 轮之后，报告具体失败并请求指示。验证通过之前绝不返回链接。

   ```sh
   SLUG="<artifact-slug>"
   # `project_dir` comes from your build task. A goal document is built under
   # that goal's `files/` directory, so a hardcoded `your_files` path is wrong.
   # `project_dir` comes from your build task. Write its leading `~/` as
   # `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
   # `"~/workspace/..."` builds into a directory literally named `~`.
   SRC="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>/.src"
   HTML="$SRC/index.html"
   # Gate both StylePlan Google Font families at 400 (Family:weight,
   # comma-separated), which every catalog family publishes. Add another weight
   # only after confirming the returned Google Fonts CSS contains that exact face;
   # single-weight display families ignore requested 600/700 faces. Pass the theme
   # families, never a local fallback: naming the fallback makes a wrong deck green.
   # Omitting --fonts still gates the families read from the deck's own `:root`, but
   # cannot validate exact weights.

   # For PPTX-only / HTML-only output: validate the slide render directly and
   # produce the PNGs consumed by the PPTX export. No intermediate PDF is produced.
   bun run "/opt/hatch/skills/artifacts/scripts/render_audit.mjs" \
     --html "$HTML" \
     --png-dir "$SRC/validate" \
     --page-selector "section.slide" --structure-check \
     --require-webfonts \
     --fonts "<Display Family>:400,<Body Family>:400" \
     --style-plan "$SRC/style_plan.json" \
     --require-fill --require-cover-image --require-generated-imagery --require-restraint \
     --hermetic --gate --report-out "$SRC/render_report.json"

   # Only when the user explicitly requested pdf in output_format: render the PDF
   # and validate the actual PDF bytes. validate_pdf.sh writes page PNGs to
   # .src/validate/.
   PDF="$SRC/$SLUG.pdf"
   bun run "/opt/hatch/skills/artifacts/scripts/render_audit.mjs" \
     --html "$HTML" \
     --pdf "$PDF" \
     --page-selector "section.slide" --structure-check \
     --require-webfonts \
     --fonts "<Display Family>:400,<Body Family>:400" \
     --style-plan "$SRC/style_plan.json" \
     --require-fill --require-cover-image --require-generated-imagery --require-restraint \
     --hermetic --gate --report-out "$SRC/render_report.json"
   "/opt/hatch/skills/artifacts/scripts/validate_pdf.sh" "$PDF" "$HTML" "$SRC/validate"
   ```

   `render_audit.mjs` renders the deck in Chromium, optionally writes a PDF (honoring the  
   `@page` size, tagged for accessibility and carrying a bookmark outline built from the  
   deck's headings), optionally screenshots each `section.slide` to a zero-padded PNG in  
   `.src/validate/`, and prints a JSON report  
   (`{ ok, pdf, pages, pngs, fonts: { missing, unused, used, expected }, overflow: [{index, overflowY_px, overflowX_px}], cover_no_image, broken_images: [...] }`)  
   to stdout, plus `fill` and `plan` (see below). A `browser_failures` array (console  
   errors, broken images, non-2xx sub-resources) means the page itself misbehaved;  
   it always sets `ok:false`, and the PNGs and report are still written so you can  
   see what broke.
   `render_audit.mjs` 在 Chromium 中渲染整套幻灯片，可选地写出 PDF（遵循 `@page` 尺寸、带无障碍标签，并携带按幻灯片标题构建的书签大纲），可选地把每个 `section.slide` 截图为 `.src/validate/` 下零填充编号的 PNG，并向 stdout 打印 JSON 报告（`{ ok, pdf, pages, pngs, fonts: { missing, unused, used, expected }, overflow: [{index, overflowY_px, overflowX_px}], cover_no_image, broken_images: [...] }`），外加 `fill` 和 `plan`（见下文）。出现 `browser_failures` 数组（控制台错误、损坏的图片、非 2xx 子资源）说明页面本身行为异常；它总会把 `ok` 置为 `false`，且 PNG 和报告仍会写出，便于你查看哪里出了问题。

   **`--gate` makes the verdict real, so read it.** With `--gate` the script sets `ok:false`,  
   lists reasons in `gate_failures`, and **exits non-zero**. A non-zero exit means the deck is not  
   finished: fix what is named and re-render. Do not export or return a link on a failed gate.  
   `--report-out` writes the same report to `.src/render_report.json`, so the verdict survives a  
   backgrounded `exec` — if your render backgrounded, read that file instead of assuming it passed.  
   It sits outside `.src/validate/` on purpose: `validate_pdf.sh` clears that directory before  
   rasterizing, which would delete the report on the PDF branch before you could read it.
   **`--gate` 使结论真实生效，所以要读它。**带 `--gate` 时，脚本会置 `ok:false`，在 `gate_failures` 中列出原因并**以非零码退出**。非零退出意味着整套幻灯片尚未完成：修复被点名的项并重新渲染。门禁失败时不要导出或返回链接。`--report-out` 会把同一份报告写到 `.src/render_report.json`，使结论在后台化的 `exec` 中也能留存——如果你的渲染放进了后台，读该文件而不要想当然认为通过了。它特意放在 `.src/validate/` 之外：`validate_pdf.sh` 在栅格化前会清空该目录，否则 PDF 分支的报告会在你读到之前被删掉。

   Gating (non-zero exit): `overflow`, `broken_images`, a slide carrying furniture,  
   uppercase, or letter tracking under `--require-restraint`, a slide under  
   `--require-fill`, a deck the media image tool generated nothing for under  
   `--require-generated-imagery`, and a  
   coverless deck when `--require-cover-image` was passed. **`advisories` never gate** — fix them  
   when they are real, but they do not block delivery. `fonts.missing` is an advisory because the  
   probe depends on the Google Fonts fetch and can flap between runs on the same deck; act on it  
   as before, but it will not fail the build.
   触发门禁（非零退出）的情况：`overflow`、`broken_images`、在 `--require-restraint` 下带装饰元素、全大写或字距调整的幻灯片、在 `--require-fill` 下不达标的幻灯片、在 `--require-generated-imagery` 下媒体图像工具一无所出的整套幻灯片，以及传入 `--require-cover-image` 时的无封面幻灯片。**`advisories` 永不触发门禁**——确属问题就去修，但不阻塞交付。`fonts.missing` 属于 advisory，因为该探测依赖 Google Fonts 拉取，同一套幻灯片多次运行的结果可能摇摆；照旧处理即可，但它不会让构建失败。

   `fill` is `[{index, top_gap_pct, v_fill_pct, h_fill_pct}]`: where each slide's content starts  
   and stops inside its own canvas. `--require-fill` (default 75%) fails a slide only when it is  
   both under that share **and top-packed** — content jammed against the top with the remainder  
   dead, which is what a fixed-height slide authored with no vertical distribution produces. A  
   deliberately sparse slide is centred, with a comparable gap above and below, and passes on any  
   layout, so a cover, statement or closing slide needs no exemption and none is applied. Fix a  
   failure by distributing the slide's content over its full height (the lockup grids in  
   `design-system.md` use `grid-template-rows: auto 1fr` and `align-self: center` for exactly this)  
   or by giving the slide more content — never by deleting the `height` or switching to  
   `min-height`, and never by padding the copy in ways `authoring.md` forbids. Every fix lands in  
   `deck.css` or the named slide's own file, then re-assemble before re-rendering.
   `fill` 是 `[{index, top_gap_pct, v_fill_pct, h_fill_pct}]`：每张幻灯片的内容在自己的画布内从哪里开始、到哪里结束。`--require-fill`（默认 75%）只有当幻灯片既低于该占比**又顶部堆叠**时才判失败——内容挤在顶部、其余空间死掉，这正是完全没有做纵向分布的定高幻灯片的产物。刻意留白的幻灯片是居中的，上下留白相当，在任何布局下都通过，因此封面、论断或结尾幻灯片无需豁免，机制也不提供豁免。修复方式是把幻灯片内容分布到整个高度（`design-system.md` 中的 lockup 网格正是为此使用 `grid-template-rows: auto 1fr` 和 `align-self: center`），或给幻灯片补充更多内容——绝不删除 `height` 或改用 `min-height`，也绝不用 `authoring.md` 禁止的方式填充文案。每次修复都落在 `deck.css` 或被点名幻灯片自己的文件里，然后在重新渲染前重新组装。

   **Drop `--require-cover-image`** on either step-6 branch: the hero could not be sourced or  
   generated, or the brief asks for a deck with no imagery at all, and you documented the  
   replacement visual system. That deck is allowed to be coverless, and the flag would make it  
   unshippable. A brief that forbids SYNTHETIC imagery is neither branch on its own: a sourced logo  
   or photo is not synthetic, so if the hero's subject is a real thing, source it and keep the flag.
   **在任一第 6 步分支上去掉 `--require-cover-image`**：主图无法搜索获取或生成，或简报要求整套幻灯片完全不带图像，而你已记录替代视觉系统。这样的幻灯片允许没有封面，该旗标会让它无法交付。禁止合成（SYNTHETIC）图像的简报本身不属于任一分支：搜索获取的 logo 或照片不是合成物，因此如果主图主题是真实事物，就去搜索获取并保留该旗标。

   **Drop `--require-generated-imagery`** on the same two step-6 branches, on a deliberately  
   chart-only or fully sourced deck, and on any in-place edit that does not regenerate imagery. It  
   requires one image `media.generate_image` actually returned, counted by the sidecar the tool  
   writes, so a hand-drawn placeholder cannot satisfy it. A sourced photo cannot either: it has no  
   sidecar, and nothing on disk separates a sourced file from a drawn one, which is exactly why a  
   fully sourced deck drops the flag rather than trying to pass it.
   **在相同的两个第 6 步分支上、在刻意只做图表或完全搜索获取的幻灯片上，以及在任何不重新生成图像的就地编辑上，去掉 `--require-generated-imagery`。**它要求 `media.generate_image` 真实返回过一张图片，并以该工具写出的 sidecar 文件计数，因此手绘占位图无法满足它。搜索获取的照片同样不行：它没有 sidecar，磁盘上也没有任何东西能区分搜索文件与生成文件，这正是完全搜索获取的幻灯片直接去掉该旗标而不是试图通过它的原因。

   **Drop `--require-restraint`** on any in-place edit of an existing deck, a re-theme and a  
   reorder included. The probe reads every slide, so on a deck built before this gate it reports  
   furniture, uppercase, or tracking on slides the edit must leave alone. `editing.md` keeps the  
   slide files untouched on a re-theme, applies the minimum edit on a reorder or image swap, and  
   calls a silent restyle a bug. Keeping the flag there leaves no legal move and no deck to  
   return. Fix the rules on the slides you did edit. Mention the untouched slides only if you  
   actually saw furniture, uppercase, or tracking on one of them; a deck built under the gate  
   carries none, so say nothing. Only a fresh build keeps the flag.
   **在对现有幻灯片的任何就地编辑（包括换主题和重排序）上去掉 `--require-restraint`。**该探测会读取每张幻灯片，因此在门禁出现之前构建的幻灯片上，它会在本次编辑必须保持不动的幻灯片上报告装饰元素、全大写或字距调整。`editing.md` 规定换主题时不改动幻灯片文件，重排序或换图时只做最小编辑，并把静默改样式视为 bug。在那里保留旗标将无合规动作可走，也没有幻灯片可交付。只需修复你确实编辑过的幻灯片上的规则。仅当你确实在未触碰的幻灯片上看到装饰元素、全大写或字距调整时才提及；在门禁之下构建的幻灯片不会有这些，所以不要多嘴。只有全新构建才保留该旗标。

   `plan` compares the built deck against the StylePlan and is **advisory only**: `missing_slides`  
   are `layout_plan` ids not built and `extra_slides` are ids the plan never asked for. Neither  
   blocks, because a shorten edit legitimately trims slides the saved plan still lists. A missing  
   `style_plan.json` is fine: the comparison is skipped and the run continues. The fill gate does  
   not depend on it.
   `plan` 将构建出的幻灯片与 StylePlan 比对，**仅供参考（advisory）**：`missing_slides` 是未构建的 `layout_plan` id，`extra_slides` 是计划从未要求的 id。两者都不阻塞，因为缩短编辑合法地裁掉已保存计划仍列出的幻灯片。缺少 `style_plan.json` 没关系：跳过比对并继续运行。fill 门禁不依赖它。

   Older findings still apply: re-edit and re-render if `fonts.missing` is  
   non-empty, or if `overflow` is non-empty. `fonts.used` is the set of families Chromium  
   actually rasterized (measured, not the CSS list); `fonts.unused` lists available faces that no  
   element used and is advisory. A missing face means the download failed or the family name is  
   wrong, so fix the spelling of the family in **`.src/slides/deck.css`'s `:root` tokens** (exact  
   spelling as Google publishes it), then re-run step 7 (embed, then assemble) and re-render.  
   Editing `.src/index.html` fixes nothing: assemble rewrites it from `deck.css` on the next run,  
   so the edit is thrown away. Do not rewrite `:root` to a  
   substituted family either, which only hides the failure. If only a heavier face is missing while  
   `fonts.used` contains the family,  
   check the returned Google Fonts CSS: when that weight is not published, remove it from the  
   gate and keep the family-level `400` check. `--require-webfonts` also rejects an installed  
   local face with the same name: slide themes must resolve through a custom `@font-face`, not an  
   image-dependent system font. Each overflow entry is a slide whose content spills past its own  
   client box (`scrollHeight > clientHeight`); fix the named slide `index` by  
   **reducing content**, not shrinking fonts. Also re-render if `cover_no_image` is true  
   (the cover has no real image; generate a hero, unless the deck is intentionally image-free  
   per step 6), or if `broken_images` is non-empty (an `<img>` whose src is not image bytes;  
   embed the generated file as a `data:image` URI, not the generate-image tool's JSON result).  
   These signals are more reliable than reading the PNGs blind.
   旧有结论仍然适用：若 `fonts.missing` 非空或 `overflow` 非空，重新编辑并重新渲染。`fonts.used` 是 Chromium 实际栅格化的字体族集合（实测值，不是 CSS 列表）；`fonts.unused` 列出没有任何元素使用的可用字体面，属 advisory。缺少某个字体面意味着下载失败或字体族名写错，因此修正 **`.src/slides/deck.css` 的 `:root` 令牌**中的字体族拼写（与 Google 发布的精确拼写一致），然后重跑第 7 步（先 embed 后 assemble）并重新渲染。编辑 `.src/index.html` 毫无用处：assemble 下次运行会从 `deck.css` 重写它，编辑会被丢弃。也不要把 `:root` 改写成替代字体族，那只是掩盖失败。若只缺较重的字重而 `fonts.used` 包含该字体族，检查返回的 Google Fonts CSS：当该字重未发布时，把它从门禁中去掉并保留字体族级 `400` 检查。`--require-webfonts` 也会拒绝同名的本地已安装字体面：幻灯片主题必须通过自定义 `@font-face` 解析，而不是依赖图像的系统字体。每个 overflow 条目对应一张内容溢出自身客户盒的幻灯片（`scrollHeight > clientHeight`）；修复被点名幻灯片 `index` 的办法是**削减内容**，而不是缩小字体。若 `cover_no_image` 为 true（封面没有真实图片；生成一张主图，除非按第 6 步整套幻灯片刻意不用图像），或 `broken_images` 非空（某个 `<img>` 的 src 不是图片字节；把生成文件内嵌为 `data:image` URI，而不是生成图像工具的 JSON 结果），也要重新渲染。这些信号比盲目看 PNG 更可靠。

   When `pdf` is in `output_format`, `validate_pdf.sh` verifies PDF integrity, parses page  
   count, checks `<img>` and CSS `url(...)` data-URI embedding, checks image-heavy PDF size,  
   and rasterizes the actual PDF pages to `.src/validate/page-*.png` with `pdftoppm`. If `pptx`  
   is also in `output_format`, build the PPTX from those PDF-derived PNGs.
   当 `pdf` 在 `output_format` 中时，`validate_pdf.sh` 校验 PDF 完整性、解析页数、检查 `<img>` 和 CSS `url(...)` 的 data-URI 内嵌、检查图像密集 PDF 的大小，并用 `pdftoppm` 把实际 PDF 页面栅格化为 `.src/validate/page-*.png`。如果 `pptx` 也在 `output_format` 中，就用这些由 PDF 派生的 PNG 构建 PPTX。

9. **Eyeball.** PNG filenames match `page-*.png`; do not assume a specific padding style. After
   the render report passes (and after `validate_pdf.sh` passes when a PDF is requested),
   **read every generated PNG with the `read` tool** and verify: rendered page count equals the
   resolved slide count; the palette and fonts are visibly applied; at least 3 distinct layouts
   when 5 or more slides; the cover establishes the visual language; images render and match
   their claim; heroes are legible; sufficient contrast; no missing images, blank slides,
   clipped or unreadable text, or repeated weak layouts; charts render correctly.
   **目检。**PNG 文件名形如 `page-*.png`；不要假设特定的填充（padding）样式。在渲染报告通过之后（如请求了 PDF，还需 `validate_pdf.sh` 通过之后），**用 `read` 工具读取每一张生成的 PNG** 并核对：渲染页数等于已解析幻灯片数量；调色板和字体可见地得到应用；幻灯片达到 5 张及以上时至少有 3 种不同布局；封面确立了视觉语言；图片正常渲染且与所述相符；主图清晰可读；对比度充足；没有缺失图片、空白幻灯片、被裁切或不可读的文字，也没有重复的弱势布局；图表渲染正确。

10. **Export and promote.** Promote each requested format to the slug root; files not in
   `output_format` stay under `.src/`.
   **导出并晋升。**把每种被请求的格式晋升到 slug 根目录；不在 `output_format` 中的文件留在 `.src/` 下。
   - `pptx`: build the deck from the validated PNGs with the bundled `build_pptx.py`. It
     requires `python-pptx`; if the import is missing, install it once with
     `python3 -m pip install --break-system-packages python-pptx` (PEP 668 systems require the
     flag; `-m pip` targets the same interpreter the script runs under). Then run and verify
     (substitute the real slug and title; do not paste literal `<artifact-slug>`):
     `pptx`: 用随附的 `build_pptx.py` 从验证过的 PNG 构建幻灯片。它依赖 `python-pptx`；若缺少该导入，用 `python3 -m pip install --break-system-packages python-pptx` 安装一次（PEP 668 系统需要该旗标；`-m pip` 指向脚本运行所用的同一解释器）。然后运行并校验（代入真实的 slug 和标题；不要原样粘贴 `<artifact-slug>`）：

     ```sh
     SLUG="<artifact-slug>"
     TITLE="<artifact-title>"
     # `project_dir` comes from your build task, and is not always under
     # `your_files`. Pass it to the script so the deck lands with its source.
     # `project_dir` comes from your build task. Write its leading `~/` as
     # `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
     # `"~/workspace/..."` builds into a directory literally named `~`.
     DIR="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>"
     "/opt/hatch/skills/artifacts/scripts/build_pptx.py" "$SLUG" "$TITLE" --project-dir "$DIR" || { echo "PPTX build failed" >&2; exit 1; }
     unzip -l "$DIR/$SLUG.pptx" | grep -q "ppt/media/" || { echo "PPTX has no embedded slide images" >&2; exit 1; }
     unzip -p "$DIR/$SLUG.pptx" docProps/core.xml | grep -qi "<dc:title>[^<]" || { echo "PPTX title metadata is empty" >&2; exit 1; }
     ```

`build_pptx.py <slug> [title]` reads the validated `.src/validate/page-*.png`, sizes each  
     slide to the PNG aspect ratio (full-bleed, no distortion), embeds one image per slide,  
     sets the deck title, and writes `<slug>.pptx` at the slug root. The image-based PPTX has  
     no editable text or shapes; each slide's title rides its speaker notes as best-effort  
     accessibility text.
     `build_pptx.py <slug> [title]` 读取验证过的 `.src/validate/page-*.png`，把每张幻灯片按 PNG 宽高比定尺寸（满幅、不变形），每张幻灯片内嵌一张图片，设置幻灯片标题，并在 slug 根目录写出 `<slug>.pptx`。这种基于图像的 PPTX 没有可编辑的文字或形状；每张幻灯片的标题随讲者备注保存，作为尽力而为的无障碍文本。
   - `pdf`: copy the validated `.src/<artifact-slug>.pdf` to `<artifact-slug>.pdf` at the slug
     root.
     `pdf`: 把验证过的 `.src/<artifact-slug>.pdf` 复制为 slug 根目录的 `<artifact-slug>.pdf`。
   - `html`, only when `html` is in `output_format`: copy the **assembled** `.src/index.html` to
     `<artifact-slug>.html` at the slug root (one self-contained file; it carries the theme's
     font bytes inline, so it renders in the branded face offline and fetches nothing when
     opened). Never promote the per-slide files:  
     `.src/slides/` is deck source, not a deliverable.
     `html`（仅当 `html` 在 `output_format` 中）：把**组装后的** `.src/index.html` 复制为 slug 根目录的 `<artifact-slug>.html`（单个自包含文件；它内联携带主题字体字节，因此离线也能以品牌字体渲染，打开时不发起任何请求）。绝不晋升单张幻灯片文件：  
     `.src/slides/` 是幻灯片源码，不是交付物。

   **If `pptx` cannot be produced** (`python-pptx` will not install, or the build/verify checks  
   fail), do not silently substitute PDF or HTML for a default deck. Drop `pptx` only when  
   other formats were explicitly requested in `output_format` and continue with those requested  
   outputs. If a default `["pptx"]` deck cannot produce PPTX, report that PowerPoint export  
   failed and ask whether PDF or HTML is wanted instead.
   **如果无法产出 `pptx`**（`python-pptx` 装不上，或构建/校验检查失败），不要默默用 PDF 或 HTML 顶替默认幻灯片。只有当 `output_format` 中明确请求了其他格式时才去掉 `pptx`，并继续产出那些被请求的输出。如果默认 `["pptx"]` 的幻灯片无法产出 PPTX，就报告 PowerPoint 导出失败，并询问用户是否改为要 PDF 或 HTML。

11. **Write `project_dir/meta.json`** after every requested output has been verified at the
    slug root. Include entries in `outputs` only for files that both exist at the slug root and
    were explicitly requested by `output_format`; `primary_output` is the highest-priority
    produced requested file (`pptx > pdf > html`). `description` is a one-line summary of the
    deck. Only `title` and `description` are read by the runtime today; the remaining fields
    are informational bookkeeping.
    **编写 `project_dir/meta.json`**：在每种被请求的输出都已在 slug 根目录验证之后。`outputs` 中只收录既存在于 slug 根目录、又被 `output_format` 明确请求的文件；`primary_output` 是已产出且被请求的文件中优先级最高者（`pptx > pdf > html`）。`description` 是整套幻灯片的一行摘要。目前运行时只读取 `title` 和 `description`；其余字段只是信息性记录。

Compute the dynamic fields in the same step that writes `meta.json` (after the final  
    validation render); do not reuse values cached from an earlier validation iteration, since  
    each run rewrites `.src/validate/`. Use the absolute `.src/validate/` path (set  
    `SRC="$JARVIS_HOME/<project_dir without its leading ~/>/.src"`) so the commands do not  
    depend on the current directory. **Use the command output, not the literal example values  
    below:**
    在写出 `meta.json` 的同一步中计算动态字段（在最终验证渲染之后）；不要复用先前验证迭代缓存的值，因为每次运行都会重写 `.src/validate/`。使用绝对的 `.src/validate/` 路径（设置 `SRC="$JARVIS_HOME/<project_dir without its leading ~/>/.src"`），使这些命令不依赖当前目录。**使用命令输出，而不是下方字面示例值：**
    - `slide_count` = `ls "$SRC"/validate/page-*.png | wc -l`
      `slide_count` = `ls "$SRC"/validate/page-*.png | wc -l`（统计页数）
    - `thumbnail` = the slug-root-relative path `.src/validate/<name>`, where `<name>` is the  
      first page PNG's filename: `ls "$SRC"/validate/page-*.png | sort | head -1 | xargs -n1 basename`.  
      Store it relative (e.g. `.src/validate/page-1.png`), matching the `outputs` paths, not  
      the absolute path the `ls` prints. Do not assume a specific padding style; use the actual  
      filename.
      `thumbnail` = 相对 slug 根目录的路径 `.src/validate/<name>`，其中 `<name>` 是第一页 PNG 的文件名：`ls "$SRC"/validate/page-*.png | sort | head -1 | xargs -n1 basename`。以相对形式存储（如 `.src/validate/page-1.png`），与 `outputs` 路径一致，而不是 `ls` 打印的绝对路径。不要假设特定的填充样式；使用实际文件名。

    ```json
    {
      "title": "<artifact-title>",
      "description": "<one-line summary of the deck>",
      "type": "presentation",
      "primary_output": "<artifact-slug>.pptx",
      "outputs": [
        {"kind": "pptx", "path": "<artifact-slug>.pptx"}
      ],
      "slide_count": 8,
      "thumbnail": "<first .src/validate/page-*.png filename>",
      "status": "complete"
    }
    ```

Add `{"kind": "pdf", "path": "<artifact-slug>.pdf"}` and/or  
    `{"kind": "html", "path": "<artifact-slug>.html"}` to `outputs` only when those formats were  
    explicitly requested and promoted to the slug root.
    仅当这些格式被明确请求并已晋升到 slug 根目录时，才把 `{"kind": "pdf", "path": "<artifact-slug>.pdf"}` 和/或 `{"kind": "html", "path": "<artifact-slug>.html"}` 加入 `outputs`。

## Quality gate / 质量门禁

A deck is not complete until it passes these checks:
整套幻灯片通过以下检查之前不算完成：
- Slide count equals the resolved slide count (the exact target, or the midpoint of a supplied
  range).
  幻灯片数量等于已解析幻灯片数量（精确目标值，或给定区间的中点）。
- It has a clear narrative arc, not a stack of unrelated pages.
  有清晰的叙事主线，而不是一堆互不相关的页面。
- It uses at least 3 distinct slide layouts when the resolved slide count is 5 or more.
  已解析幻灯片数量达到 5 张及以上时，使用至少 3 种不同的幻灯片布局。
- At most `floor(resolved slide count * 0.4)` slides are simple title-plus-bullets, where
  "title-plus-bullets" means a slide whose only non-title content is a single `<ul>` or `<ol>`.
  简单的"标题+列表"幻灯片至多为 `floor(resolved slide count * 0.4)` 张；"标题+列表"指除标题外唯一内容是单个 `<ul>` 或 `<ol>` 的幻灯片。
- Every non-appendix slide has a visual anchor: image, chart, timeline, diagram, callout
  system, or strong typographic composition.
  每张非附录幻灯片都有视觉锚点：图片、图表、时间线、图示、标注系统，或强有力的排版构图。
- If generated or sourced imagery was requested, attempted, or created, the slide files embed
  the selected images (`<img src="data:image...">` or CSS `url(data:image...)`) and the
  validation PNGs visibly show them. Any unused generated files in `.src/media/` are either
  deleted or documented in `deck_plan.md` with a reason they were rejected.
  如果请求过、尝试过或创建过生成/搜索获取的图像，幻灯片文件要内嵌选定的图片（`<img src="data:image...">` 或 CSS `url(data:image...)`），且验证 PNG 能明显看到它们。`.src/media/` 中任何未使用的生成文件要么删除，要么在 `deck_plan.md` 中记录被否弃的原因。
- The title slide establishes the deck's visual language immediately.
  标题页立即确立整套幻灯片的视觉语言。
- Dense text has been rewritten, split, or removed instead of being shrunk until unreadable.
  密集文字已被改写、拆分或删除，而不是被缩小到无法阅读。
- The PPTX, when produced, is exported from the validated rendered slides. Do not build a
  direct native-shape PPTX.
  产出的 PPTX 从验证过的渲染幻灯片导出。不要构建直接的本地形状（native-shape）PPTX。
- Data charts are palette-colored matplotlib; imagery follows step 6 (a cover hero plus at
  least 2 illustrative images, sourced or generated per their subject, or the documented
  fallback); every claim is grounded in the provided source.
  数据图表是调色板配色的 matplotlib；图像遵循第 6 步（一张封面主图加至少 2 张说明性图片，按各自主题搜索获取或生成，或采用已记录的回退方案）；每个论断都有所提供来源的支撑。
- The deck exists as per-slide source under `.src/slides/` and the combined `.src/index.html`
  was produced from it by `assemble_deck.mjs` after the last edit, so the two agree.
  幻灯片以单张源文件形式存在于 `.src/slides/` 下，且合并的 `.src/index.html` 是在最后一次编辑之后由 `assemble_deck.mjs` 从它生成的，两者保持一致。

If a check fails after 3 iterations, report the specific slide + failure and ask for direction
rather than shipping a clipped or off-theme deck.
如果 3 轮迭代后仍有检查失败，报告具体的幻灯片与失败项并请求指示，而不是交付被裁切或偏离主题的幻灯片。

【评论】把检查分为"gating（硬性门禁，非零退出、阻塞交付）"与"advisory（建议，不阻塞）"是典型的自动化质检分级设计；配合"最多 3 轮迭代后上报"的止损规则，避免模型陷入无休止的修复循环。
