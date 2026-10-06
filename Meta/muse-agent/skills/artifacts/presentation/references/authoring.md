<!-- BILINGUAL-EN-ZH -->
---
description: How to write each slide's HTML so the deck reads as designed rather than templated.
---

# Slide authoring rulebook / 幻灯片编写规则手册

How to write each slide's HTML so the deck reads like a designed presentation, not a
template. You write a self-contained `section.slide` per slide, images embedded as `data:`
URIs, validated by `render_audit.mjs`. Read this together with `design-system.md` (the CSS
tokens + canvas + hero/lockup/layout scaffolds) and `image-directive.md` (art-directing
`media.generate_image`).

如何编写每张幻灯片的 HTML，使整套幻灯片读起来像经过设计的作品而非套用模板。你为每张幻灯片编写一个自包含的 `section.slide`，图像以 `data:` URI 内嵌，并由 `render_audit.mjs` 校验。请与 `design-system.md`（CSS 令牌 + 画布 + hero/lockup/版式脚手架）和 `image-directive.md`（`media.generate_image` 的艺术指导）一起阅读。

When you were handed a `StylePlan`, it already picked the archetype, theme, palette, fonts,
a per-slide layout, and the deck's lockup pair: apply it; do not re-pick colors or fonts.
Without one, apply the design system you set in the workflow with the same discipline.

如果拿到了 `StylePlan`，它已经选定了原型（archetype）、主题、调色板、字体、每张幻灯片的版式以及整套的 lockup 组合：照着执行即可，不要重新挑选颜色或字体。若没有，则按同样的纪律应用你在工作流中设定的设计系统。

## Content strategy / 内容策略

- Decide what the slide *says* before how it looks; content drives layout.
  先决定幻灯片要*表达什么*，再考虑外观；内容驱动版式。
- One message per slide. Put that message in the heading as a **claim, not a topic**
  ("Proposal stage is driving growth", not "Pipeline overview"), and let the largest element
  on the slide prove it. If you can't name the one takeaway, the slide isn't ready.
  每张幻灯片只传递一个信息。把该信息作为**论断而非话题**写进标题（"提案阶段正在驱动增长"，而非"管道概览"），并让幻灯片上最大的元素来证明它。如果说不出唯一的核心结论，这张幻灯片还没准备好。
- Titles are short and contain no metrics. Lead with the so-what: text carries the
  interpretation, visuals carry the evidence.
  标题要短且不含数字指标。以"所以呢"开篇：文字承载解读，视觉承载证据。
- Give each fact one home. If a chart plots the per-item values, don't also repeat them as
  stat cards, bar labels, and a legend.
  每个事实只有一个安放之处。若图表已画出各条目的数值，就不要再用统计卡片、条形标签和图例重复它们。
- Highlight at most 3 metrics, chosen to build toward the takeaway.
  最多突出 3 个指标，且要为支撑核心结论而挑选。
- Let it breathe: cap a content slide at ~5 bullets (1-2 lines each) or 3-4 cards. When
  there's more, cut it or split across slides rather than cramming.
  留出呼吸感：内容页最多约 5 个要点（每条 1-2 行）或 3-4 张卡片。超出时宁可删减或拆分到多张幻灯片，不要硬塞。
- Use only real, verifiable data from the provided source material; never fabricate figures,
  dates, citations, or source names. Cite a real source at most once, as a small muted note;
  never cite a chart you just generated.
  只使用所提供源材料中真实、可验证的数据；绝不编造数字、日期、引文或来源名称。真实来源最多引用一次，以小的弱化注释形式呈现；绝不要引用你刚生成的图表。

【评论】"绝不引用你刚生成的图表"堵住了循环自证的来源标注——生成物给自己当出处是常见的幻觉形式。

## Simplicity and restraint / 简洁与克制

- **Strictly prohibited anywhere on a slide:** pills, subtitles, summary lines, eyebrows,
  kickers, badges, chips, tags, captions. Remove them. Do not write CSS for them either: a
  `.kicker` rule you never use, or one that only hides the class, is dead code in every deck.
  **幻灯片上任何位置都严格禁止：** 胶囊元素、副标题、摘要行、眉题（eyebrow）、引题（kicker）、徽章、chip、标签、图注。一律移除，也不要为其编写 CSS：一条从不使用的 `.kicker` 规则，或一条只负责隐藏该类的规则，在任何一套幻灯片里都是死代码。
- Every element must serve the point and must not repeat information. Information reads
  linearly in a logical order.
  每个元素都必须服务于要点且不得重复信息。信息按逻辑顺序线性展开。
- The cover carries only the title over its hero image: no subtitle, tagline, date/source
  line, icons, cards, or stat boxes. Save details for the content slides.
  封面在其 hero 图上只放标题：不放副标题、宣传语、日期/来源行、图标、卡片或统计框。细节留给内容页。

## Layout and composition / 版式与构图

- Aim for unified, full-canvas layouts, and vary the structure between slides (cover and
  closing are the exceptions). Use typography and spacing to guide attention.
  追求统一的全画布版式，并在各页之间变化结构（封面与结尾页除外）。用排版与间距引导视线。
- Build multi-element layouts with **CSS Grid**, giving every cell an explicit `grid-column`
  and `grid-row` so nothing overlaps and nothing relies on auto-placement. Do not use
  `position:absolute` for main containers, and keep containers aligned.
  用 **CSS Grid** 构建多元素版式，为每个单元格显式指定 `grid-column` 与 `grid-row`，使任何元素都不重叠、不依赖自动布局。不要对主容器使用 `position:absolute`，并保持容器对齐。
- Keep at least a 16px gap between content blocks and at least 48px between any content and
  every slide edge. No element's bounding box may intersect another's; text never sits over a
  busy part of a photo.
  内容块之间至少留 16px 间距，任何内容与幻灯片边缘之间至少留 48px。任何元素的包围盒不得与另一个相交；文字绝不能压在照片的繁杂区域上。
- Fill the canvas by **centering the content as a group** (`align-content:center` on a grid,
  `justify-content:center` on a flex column) so leftover space becomes balanced top/bottom
  margins. Do not inflate cards to fill vertical space, and do not dump the slack into one
  band at the bottom. If a layout is much shorter than 720px, enlarge the image, step up the
  type scale, or use fewer columns.
  通过**将内容作为整体居中**来填满画布（网格上用 `align-content:center`，flex 纵向布局上用 `justify-content:center`），让剩余空间变成均衡的上下边距。不要靠撑大卡片填充纵向空间，也不要把余量全堆到底部一条带上。若版面远矮于 720px，就放大图像、提高字号阶梯或减少列数。
- Keep prose to **at most two side-by-side columns** (~460px each so lines hold 8-14 words).
  If content seems to need three or more, drop to two, stack full-width rows under the
  heading, or cut. Only short stat lockups (a number + a ~3-word label) belong in three
  columns, never sentences or bullets.
  正文最多**两个并排列**（每列约 460px，使每行容纳 8-14 个词）。若内容似乎需要三列或更多，就减到两列、在标题下堆叠全宽行，或删减。只有简短的统计 lockup（一个数字 + 约 3 词的标签）可以用三列，句子和要点绝不可以。
- A numbered, ranked, or sequential list reads as a **left-aligned vertical list** (one item
  per row, number/label at the left, heading left-aligned above). Never lay a ranked sequence
  as a centered block or a 2x2 grid. Reserve centered alignment for a single statement slide
  or the closing slide; never center a multi-item list or a content slide's title.
  编号、排序或顺序列表应呈现为**左对齐的纵向列表**（每行一项，编号/标签在左，标题在上方左对齐）。绝不要把排序序列排成居中块或 2x2 网格。居中对齐只保留给单句观点页或结尾页；绝不要居中多项目列表或内容页标题。
- If content exceeds the space, **reduce content** (fewer bullets, shorter text) rather than
  shrinking fonts below the hierarchy minimums. Clipping is never acceptable.
  若内容超出空间，应**削减内容**（更少的要点、更短的文字），而不是把字号缩到层级下限以下。裁切在任何情况下都不可接受。

## Typography and color / 排版与色彩

- Content layouts use these sizes: heading `var(--slide-font-display)` 48-64px bold
  `var(--slide-fg)`; body `var(--slide-font-body)` 16-18px, line-height 1.4-1.5; stat number
  `var(--slide-font-display)` 48-72px bold `var(--slide-primary)`; stat label
  `var(--slide-font-body)` 11-13px sentence case. Stat lockups put the number above the label,
  left-aligned, ≥24px between adjacent lockups.
  内容版式使用以下尺寸：标题 `var(--slide-font-display)` 48-64px 粗体、颜色 `var(--slide-fg)`；正文 `var(--slide-font-body)` 16-18px，行高 1.4-1.5；统计数字 `var(--slide-font-display)` 48-72px 粗体、颜色 `var(--slide-primary)`；统计标签 `var(--slide-font-body)` 11-13px、句首大写。统计 lockup 中数字在标签上方，左对齐，相邻 lockup 间距 ≥24px。
- Two adjacent elements never share a font size; each level is visibly distinct.
  相邻两个元素不得共用同一字号；每个层级都要有可辨识的差异。
- **Sentence case everywhere** (including stat labels). Never use `text-transform:uppercase`
  or `letter-spacing` on any text.
  **一律使用句首大写**（包括统计标签）。绝不对任何文本使用 `text-transform:uppercase` 或 `letter-spacing`。
- **Use the CSS variables for all colors and fonts; never hardcode hex or font-family names.**
  `var(--slide-bg)` backgrounds, `var(--slide-fg)` primary text, `var(--slide-primary)` for
  emphasis (titles, key data, borders, icons) used sparingly, `var(--slide-accent)` for
  support, `var(--slide-font-display)` headings, `var(--slide-font-body)` body. For
  semi-transparent overlays use `color-mix` (e.g. `color-mix(in srgb, var(--slide-primary) 20%,
  transparent)`). Keep to 2-3 palette colors per slide. The `:root` block (from the StylePlan)
  defines all theme variables; do not redefine them.
  **所有颜色与字体都使用 CSS 变量；绝不硬编码十六进制色值或 font-family 名称。** 背景用 `var(--slide-bg)`，主文本用 `var(--slide-fg)`，强调（标题、关键数据、边框、图标）少而精地用 `var(--slide-primary)`，辅助用 `var(--slide-accent)`，标题字体 `var(--slide-font-display)`，正文字体 `var(--slide-font-body)`。半透明叠加用 `color-mix`（如 `color-mix(in srgb, var(--slide-primary) 20%, transparent)`）。每张幻灯片控制在 2-3 种调色板颜色内。`:root` 块（来自 StylePlan）定义全部主题变量；不要重新定义它们。
- **In a slide file, a `#` colour or an `rgb(` is a bug.** Two exceptions exist, and nothing
  else. A slide's own `--hero-scrim-color:` is one, sampled from that slide's image. A cover
  title over a photo is the other, and it keeps `#fff` or a fixed dark value against the scrim.
  Everywhere else the colour is `var(--slide-*)`, or
  `color-mix(in srgb, var(--slide-*) N%, transparent)` for a tint. A darker or lighter shade of
  a token is `color-mix(in srgb, var(--slide-fg) 90%, black)`, or the same with `white`. A wash
  laid over an image is a `color-mix` of `var(--slide-bg)` inside the gradient, not an `rgba()`
  of its channels. That covers a `style=` attribute, a slide's own `<style>` block, a gradient
  stop, a shadow and an SVG `fill`. It covers a badge, a border and a status chip, and a set you
  colour-code by level or risk. In `deck.css` a `#` belongs in the one `:root` block, plus the
  `html, body` backdrop and the `var(--hero-scrim-color, #1a1a2e)` fallback. An inverse card
  needs no new colour, because a dark fill with light text is `background:var(--slide-fg)` with
  `color:var(--slide-bg)`. A raw value looks correct today. Then it stays as it is while
  everything around it changes theme.
  **在幻灯片文件里，出现 `#` 色值或 `rgb(` 就是缺陷。** 例外只有两个，别无其他。其一是幻灯片自身的 `--hero-scrim-color:`，从该页图像取色。其二是压在照片上的封面标题，它在 scrim 之上保持 `#fff` 或固定的深色值。其他任何地方，颜色一律是 `var(--slide-*)`，淡色调用 `color-mix(in srgb, var(--slide-*) N%, transparent)`。令牌的更深或更浅变体用 `color-mix(in srgb, var(--slide-fg) 90%, black)`，或把 `black` 换成 `white`。盖在图像上的色罩是渐变内 `var(--slide-bg)` 的 `color-mix`，而不是对其通道的 `rgba()`。这涵盖 `style=` 属性、幻灯片自己的 `<style>` 块、渐变节点、阴影和 SVG `fill`；也涵盖徽章、边框、状态 chip，以及按等级或风险做颜色编码的一组元素。在 `deck.css` 里，`#` 只属于唯一的 `:root` 块，外加 `html, body` 背景和 `var(--hero-scrim-color, #1a1a2e)` 回退值。反色卡片不需要新颜色，因为深底浅字就是 `background:var(--slide-fg)` 加 `color:var(--slide-bg)`。写死的原始值今天看着正确，随后周围全部更换主题时它却保持原样。

  ```html
  <!-- wrong: every one of these survives a theme change unchanged -->
  <strong style="color:#008694">                                        <!-- copied token -->
  <div style="border:1px solid rgba(0,134,148,0.12)">                    <!-- token as rgba channels -->
  <div style="background:#FAF7F2">                                       <!-- the paper colour, written out -->
  <div style="background:color-mix(in srgb, #11181F 70%, transparent)">  <!-- copied inside a function -->

  <!-- right -->
  <strong style="color:var(--slide-primary)">
  <div style="border:1px solid color-mix(in srgb, var(--slide-primary) 12%, transparent)">
  <div style="background:var(--slide-bg)">
  ```
  【评论】禁止硬编码色值并只留两个例外，是为了保证整套幻灯片在换主题时全局一致；写死的颜色不会随 `:root` 令牌更新。

- **Inline SVG takes its colours through CSS, not through attributes.** A presentation
  attribute does not resolve `var()` dependably. So do not write `fill="var(--slide-primary)"`.
  Write `style="fill:var(--slide-primary)"`, or put the rule in `deck.css`. A diagram whose
  `fill` and `stroke` attributes hold hex values keeps those colours through every theme
  change. A diagram is usually the largest coloured thing on its slide.
  **内联 SVG 的颜色要通过 CSS 获得，而不是通过属性。** 表现属性不能可靠解析 `var()`。因此不要写 `fill="var(--slide-primary)"`，而要写 `style="fill:var(--slide-primary)"`，或把规则放进 `deck.css`。`fill` 与 `stroke` 属性中写死十六进制值的示意图会在每次换主题时保持原色。示意图通常是一页中最大的着色对象。
- Every slide sets an opaque `background` on `section.slide` (the deck's `var(--slide-bg)`, or
  a full-bleed cover image). An unset background shows the page grey and reads as broken. Don't
  give the cover or closing a different flat background color; a different look there comes from
  a full-bleed image or the layout, not from swapping the paper color.
  每张幻灯片都要在 `section.slide` 上设置不透明的 `background`（整套的 `var(--slide-bg)`，或全出血封面图）。未设置背景会露出页面灰底，看起来像坏了。不要给封面或结尾页换一种平涂背景色；它们的不同观感应来自全出血图像或版式，而不是更换"纸张"颜色。
- All readable text meets ≥4.5:1 contrast against whatever sits directly behind it (3:1 for
  text ≥24px or bold ≥19px). **Never create hierarchy or dim text with `opacity` or low-alpha
  `rgba`/`hsla`**: it composites against the background and quietly drops below AA; use
  `var(--slide-muted)`. A bold label and its continuation after a dash/colon are the same body
  tier at full contrast; don't gray the continuation.
  所有可读文本与其正后方背景的对比度须 ≥4.5:1（文本 ≥24px 或粗体 ≥19px 时为 3:1）。**绝不用 `opacity` 或低透明度 `rgba`/`hsla` 制造层级或弱化文字**：它会与背景混合，悄然跌破 AA 标准；要弱化就用 `var(--slide-muted)`。粗体标签与其后跟在破折号/冒号之后的延续文字属同一正文层级、保持全对比度；不要把延续部分变灰。
- Do not use `padding-bottom` anywhere; use `padding-top` for vertical spacing.
  任何地方都不要用 `padding-bottom`；纵向间距用 `padding-top`。

## Images / 图像

- By default every deck needs imagery: source a real subject (a logo, product, place, or person)
  with the image-search skill, or generate a concept subject with `media.generate_image`. An
  all-typography deck, or one that fills space with CSS gradients and inline SVG, is a miss unless
  imagery can't be sourced or generated, or the brief asks for a deck with no imagery at all.
  Charts carry the data, images carry the concept. A brief that forbids SYNTHETIC imagery is not
  that case: it removes the generate route, not the source route, so a real subject is still
  sourced with the image-search skill.
  默认情况下每套幻灯片都需要图像：用 image-search 技能获取真实主体（徽标、产品、地点或人物），或用 `media.generate_image` 生成概念主体。纯排版幻灯片，或用 CSS 渐变与内联 SVG 填充空间的幻灯片，都属于失误，除非图像无法获取或生成，或需求明确要求整套无图。图表承载数据，图像承载概念。禁止"合成（SYNTHETIC）"图像的需求不属于上述例外：它取消的是生成路径，而不是获取路径，因此真实主体仍要用 image-search 技能获取。
- Each major section or concept that can be shown visually gets one image that genuinely
  depicts it. **Do not** drop a decorative image on a pure-number, stat, or statement cell, or
  into a grid cell just to fill it or hit a count (a non-illustrative image is worse than
  empty space). Carry at least 2 illustrative images across the deck, sourced or generated per
  their subject, only where they illustrate content.
  每个可以视觉呈现的主要章节或概念，都应配一张切实描绘它的图像。**不要**把装饰性图像放到纯数字、统计或观点单元格上，也不要为了填满网格单元格或凑数而放图（不具说明性的图像比留白更糟）。整套幻灯片至少携带 2 张有说明性的图像，按各自主题获取或生成，且只放在确实能说明内容的位置。
- The **cover always carries a hero image**: source it with the image-search skill when its
  subject is a real thing (a company, product, place, or person), otherwise generate it with
  `media.generate_image`. A text-only cover is a bug; a CSS gradient, flat color, or inline SVG
  is not a substitute for a real hero image, except as a documented fallback when the image can't
  be obtained or the deck is intentionally image-free.
  **封面必须携带 hero 图**：主体是真实事物（公司、产品、地点或人物）时用 image-search 技能获取，否则用 `media.generate_image` 生成。纯文字封面是缺陷；CSS 渐变、平涂色或内联 SVG 不能替代真实 hero 图，除非作为有记录的回退——图像无法获取，或整套幻灯片刻意无图。
- Give images rounded corners (16-24px large, 12px thumbnails), except full-bleed hero and
  split-panel images, which bleed to the edge with no radius (see `design-system.md`).
  图像用圆角（大图 16-24px，缩略图 12px），但全出血 hero 图与分栏面板图除外——它们出血到边缘、无圆角（见 `design-system.md`）。
- A cell-filling image uses `width:100%; height:100%; object-fit:cover` so it fills its cell
  with no blank band. **Never** give such an image a fixed height or `max-height` inside a
  flexible (`1fr` / `height:100%` / `flex:1`) track: a fixed-height child can't fill a
  flexible parent and leaves a white bar. Request each image in its cell's aspect ratio
  (tall for a tall cell, wide for a wide cell). Use `object-fit:contain` (over a
  `var(--slide-bg)` backdrop) only for an image with embedded text or a diagram that must not
  be cropped.
  填满单元格的图像使用 `width:100%; height:100%; object-fit:cover`，使其填满单元格且无空白条带。**绝不**给这类图像在弹性轨道（`1fr` / `height:100%` / `flex:1`）内设置固定高度或 `max-height`：固定高度的子元素无法填满弹性父元素，会留下白色条带。按单元格的长宽比请求每张图像（高格用竖图，宽格用横图）。`object-fit:contain`（衬在 `var(--slide-bg)` 背景上）只用于内嵌文字的图像或不可裁切的示意图。
- Embed every image as a `data:` URI before validation, in the slide file that shows it, so each
  slide stands alone. Generate concept imagery with `media.generate_image` per `image-directive.md`
  and source real photos and logos with the image-search skill; place both under `.src/media/`. A deck carries **no** external reference at all, fonts
  included: you name the theme's families in `deck.css`'s `:root` tokens, and
  `embed_deck_fonts.mjs` writes those faces into `deck.css` as `@font-face` rules holding the font
  bytes. Never write a `fonts.googleapis.com` or `fonts.gstatic.com` URL in any file. Images and
  CSS `url(...)` stay `data:` URIs. Never leave any other `http(s)://`, `file://`, or relative
  `.src/media` refs in a slide file.
  校验之前，把每张图像以 `data:` URI 内嵌到展示它的幻灯片文件中，使每张幻灯片自成一体。概念图像按 `image-directive.md` 用 `media.generate_image` 生成，真实照片与徽标用 image-search 技能获取；两者都放在 `.src/media/` 下。整套幻灯片**完全不携带**任何外部引用，字体也不例外：在 `deck.css` 的 `:root` 令牌中写出主题字体族，`embed_deck_fonts.mjs` 会把这些字面以持有字体字节的 `@font-face` 规则写入 `deck.css`。绝不在任何文件中写 `fonts.googleapis.com` 或 `fonts.gstatic.com` URL。图像与 CSS `url(...)` 一律保持 `data:` URI。绝不在幻灯片文件中留下任何其他 `http(s)://`、`file://` 或相对 `.src/media` 引用。
  Omit an image rather than reuse a mismatched one.
  宁可省略图像，也不要复用不匹配的图像。
- Hero text legibility: prefer the image in its own panel with text on a separate solid
  `var(--slide-bg)` panel (no scrim needed). When text must sit on a photo, use a bottom
  gradient scrim scoped to the text band (never a full-slide scrim), a frosted-glass card, or
  a text-shadow stack. Never put unstyled text on a photo.
  Hero 文字可读性：优先让图像独占一个面板、文字放在另一个纯色 `var(--slide-bg)` 面板上（无需 scrim）。当文字必须压在照片上时，使用限定在文字带内的底部渐变 scrim（绝不要全页 scrim）、磨砂玻璃卡片或 text-shadow 叠层。绝不要把无样式的文字放在照片上。
- On a content slide, keep image edges straight or rounded-rect, never cut a text column with a
  diagonal `clip-path`/`skew` edge (an angled edge makes the title read off-center).
  Diagonal-split is a cover-only scaffold (see `design-system.md`).
  内容页上图像边缘保持平直或圆角矩形，绝不要用斜向 `clip-path`/`skew` 边切割文本列（斜边会让标题看起来偏离中心）。斜切分栏是封面专用脚手架（见 `design-system.md`）。
- Side-by-side panels must not overlap: their widths sum to 100% or less.
  并排面板不得重叠：其宽度之和不超过 100%。

## Charts and diagrams / 图表与示意图

- When a slide plots data, read `/opt/hatch/skills/artifacts/references/charts.md`
  first: it owns where the numbers come from, how to render a chart for a
  deck, and the encoding rules that apply everywhere.
  当幻灯片要绘制数据时，先读 `/opt/hatch/skills/artifacts/references/charts.md`：数字从哪来、如何为幻灯片渲染图表，以及处处适用的编码规则都由它管辖。
- Never stack a chart or image vertically with its text; pair them side by side.
  绝不要把图表或图像与文字纵向堆叠；要左右并排。

## Overflow and card sizing / 溢出与卡片尺寸

- `section.slide` is `1280px × 720px`, `overflow:hidden`, `box-sizing:border-box`. Overflow is
  **clipped silently**: content past 720px is hidden, not shown. You can't see overflow while
  authoring, so budget content to fit up front and rely on the `render_audit` `overflow` and
  `fill` checks (see `workflow.md` step 8) to catch it.
  `section.slide` 为 `1280px × 720px`、`overflow:hidden`、`box-sizing:border-box`。溢出会被**静默裁切**：超过 720px 的内容被隐藏而非显示。编写时看不见溢出，所以要预先按预算控制内容量，并依靠 `render_audit` 的 `overflow` 与 `fill` 检查（见 `workflow.md` 第 8 步）来发现它。
- Budget before writing: usable height ≈620px (720 minus margins). A slide fits roughly a
  heading + 10-12 body/row lines total; the ~5-bullet / 3-4-card caps are hard ceilings, not
  targets. A text column beside an image spans the full 720px but holds only ~a heading + 3-4
  short lines; an intro paragraph or 4+ rows there overflows.
  下笔前先做预算：可用高度约 620px（720 减去页边距）。一页大约容纳一个标题 + 共 10-12 行正文；约 5 要点 / 3-4 卡片的上限是硬性顶格，不是目标。图像旁的文本列纵贯全部 720px，却只装得下约一个标题 + 3-4 行短句；在那里放一段引言或 4 行以上内容就会溢出。
- Text and data cards **grow to fit their content** (`height:auto` or `min-height`); never
  give a stat card, callout, bento cell, or comparison block a fixed height or a line-clamp,
  and never clip text mid-line.
  文本卡与数据卡**随内容增高**（`height:auto` 或 `min-height`）；绝不给统计卡、提示块、bento 单元格或对比块设固定高度或 line-clamp，也绝不在行中间裁断文字。
- For card grids use `align-items:stretch` with auto row tracks (`grid-auto-rows:auto`) so
  every card in a row grows to match the tallest, without cropping. Keep `min-width:0`; don't
  set `min-height:0` on text cards, don't use `align-items:start`, don't use fixed-height or
  `1fr`-of-720px tracks. If the tallest card would exceed 720px, reduce the number of cards or
  shorten the text.
  卡片网格使用 `align-items:stretch` 配合自动行轨道（`grid-auto-rows:auto`），使一行中的每张卡片都长到与最高者齐平而不裁切。保持 `min-width:0`；不要在文本卡上设 `min-height:0`，不要用 `align-items:start`，也不要用固定高度或以 720px 为基准的 `1fr` 轨道。若最高卡片会超过 720px，就减少卡片数量或缩短文字。
- A **bento** reads as a balanced grid: adjacent cells equal height (`align-items:stretch`), a
  taller feature or image cell may span two rows or a wide one (`grid-column: span 2`), and the
  whole bento is centered (`align-content:center`). A text cell sizes to its content; only an
  image cell fills via `object-fit:cover`.
  **bento** 呈现为均衡网格：相邻单元格等高（`align-items:stretch`），较高的特性格或图像格可跨两行或横向加宽（`grid-column: span 2`），整个 bento 居中（`align-content:center`）。文本格按内容定尺寸；只有图像格用 `object-fit:cover` 填满。

## Layouts and lockups / 版式与图文组合（lockup）

- Each slide has an assigned layout class in the StylePlan's `layout_plan` (`cover`,
  `closing`, or one of the eight content layouts, see `design-system.md`). Treat it as the
  default, but let content drive the choice: when a slide is light, prefer a simpler layout
  over a denser grid; don't manufacture cards to fill an assigned grid.
  每张幻灯片在 StylePlan 的 `layout_plan` 中都有指定的版式类（`cover`、`closing`，或八个内容版式之一，见 `design-system.md`）。把它当作默认值，但让内容主导选择：内容较少时，优先更简单的版式而非更密的网格；不要为填满指定网格而硬造卡片。
- The **closing** is centered text on one uniform background (a single color or one full-bleed
  image over the whole 720px canvas), never a partial-height panel/band that leaves a two-tone
  strip.
  **结尾页**是统一背景上的居中文字（单一颜色，或覆盖整个 720px 画布的一张全出血图像），绝不能是留下一道双色条带的半高面板/色带。
- When a slide combines text and an image, use one of the deck's two assigned **lockup
  patterns** (`StylePlan.lockups`, drawn from L1/L2/L4, see `design-system.md`); fill the
  lockup's image cell edge-to-edge so no band of blank background shows under the image or
  text. Don't invent a new text-and-image arrangement.
  当幻灯片组合文字与图像时，使用整套幻灯片指定的两种 **lockup 模式**之一（`StylePlan.lockups`，取自 L1/L2/L4，见 `design-system.md`）；把 lockup 的图像格边到边填满，使图像或文字下方不露出空白背景条带。不要发明新的图文排布。
