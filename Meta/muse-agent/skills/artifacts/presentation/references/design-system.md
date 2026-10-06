---
description: The CSS contract slides are authored against, covering theme tokens, the canvas, and the layout scaffolds.
---
<!-- BILINGUAL-EN-ZH -->

# Slide design system / 幻灯片设计系统

The CSS contract every slide is authored against: the theme token set, the canvas, and the
hero / lockup / content-layout scaffolds. The `StylePlan` you were handed supplies the token
values (`css_variables`) and picks each slide's layout + the deck's lockup pair. All of it lives
in the deck's own `.src/slides/deck.css`, which every slide file links; no external foundation
CSS, and the one external reference the deck carries is the theme's Google Fonts stylesheet,
imported at the top of `deck.css` (see below).

每张幻灯片都依照这份 CSS 契约撰写：主题令牌集、画布，以及 hero / lockup / 内容布局脚手架。交给你的 `StylePlan` 提供令牌值（`css_variables`），并为每张幻灯片选定布局及文稿的 lockup 组合。所有内容都位于文稿自己的 `.src/slides/deck.css` 中，每个幻灯片文件都链接它；不使用外部基础 CSS，文稿携带的唯一外部引用是主题的 Google Fonts 样式表，在 `deck.css` 顶部导入（见下文）。

## Theme tokens (`:root`) / 主题令牌（`:root`）

Emit the `StylePlan.css_variables` verbatim into a single `:root` block, then reference the
variables everywhere. Never hardcode a theme hex or font-family name; never redefine these.
(The page backdrop, the hero-scrim fallback, and hero-overlay text below are the intended
literal-color exceptions.)

把 `StylePlan.css_variables` 逐字输出到一个 `:root` 块中，然后在各处引用这些变量。绝不要硬编码主题十六进制色或字族名；绝不要重新定义它们。（下文中的页面背景、hero 遮罩回退色与 hero 覆盖文字是预设的字面量颜色例外。）

```css
:root {
  --slide-bg: <paper>;           /* page background */
  --slide-fg: <ink>;             /* primary text */
  --slide-primary: <primary>;    /* emphasis: titles, key data, borders, icons (sparingly) */
  --slide-accent: <accent>;      /* supporting elements */
  --slide-muted: color-mix(in srgb, var(--slide-fg) 80%, var(--slide-bg));  /* de-emphasis; never opacity */
  --slide-font-display: "<display font>";  /* headings; quote multi-word names */
  --slide-font-body: "<body font>";        /* body */
}
```

`--hero-scrim-color` is set per-slide only on a full-bleed hero (a dark tone sampled from the
image), never in `:root`; the `var(--hero-scrim-color, #1a1a2e)` fallback keeps an unset value
safe.

`--hero-scrim-color` 只在满幅 hero 上按幻灯片设置（从图像采样的暗色调），绝不放在 `:root` 中；`var(--hero-scrim-color, #1a1a2e)` 回退值保证了未设置时也是安全的。

**Name the theme fonts in `:root`, and write no font URL anywhere.** Both `StylePlan` fonts are
Google Fonts families, but a deck never references Google. Write **no `@import`, no `<link>`, and no
`fonts.googleapis.com` or `fonts.gstatic.com`**, in `deck.css` or in any slide file. You name each
family in `--slide-font-display` and `--slide-font-body`. Then `embed_deck_fonts.mjs` (workflow.md
step 7) downloads those faces and writes them into `deck.css`. The render browser cannot fetch a
webfont at all. So an external reference is what makes the font gate fail, not what satisfies it.

**在 `:root` 中写出主题字体名，任何地方都不要写字体 URL。**`StylePlan` 的两个字体都是 Google Fonts 字族，但文稿绝不引用 Google。在 `deck.css` 或任何幻灯片文件中，**不要写 `@import`、不要写 `<link>`、不要写 `fonts.googleapis.com` 或 `fonts.gstatic.com`**。你只需在 `--slide-font-display` 与 `--slide-font-body` 中写出每个字族的名称。然后 `embed_deck_fonts.mjs`（workflow.md 第 7 步）会下载这些字体面并写入 `deck.css`。渲染浏览器完全无法抓取网络字体。因此外部引用只会让字体门禁失败，而不是满足它。

Those `:root` names are the download's only input. Spell each family exactly as Google publishes it.
Weights come from the CSS you write, so `font-weight: 900` is what fetches a real 900 face. Google
returns only the weights a family publishes; all current StylePlan families publish `400`,
while single-weight display families such as `Bebas Neue`, `Anton`, `DM Serif Display`, and
`Archivo Black` do not publish `600` or `700`. Never hand-write an `@font-face` rule or its base64.
One face is over 100 KB, and the script is the only thing that should write it.

`:root` 中的名称是下载的唯一输入。每个字族都要按 Google 发布时的拼写精确书写。字重来自你写的 CSS，因此 `font-weight: 900` 才会获取真正的 900 字面。Google 只返回字族发布的字重；当前所有 StylePlan 字族都发布 `400`，而 `Bebas Neue`、`Anton`、`DM Serif Display`、`Archivo Black` 这类单一字重的展示字族不发布 `600` 或 `700`。绝不要手写 `@font-face` 规则或其 base64。一个字体面超过 100 KB，只有脚本才应该写它。

**Keep each exact StylePlan family first in its `:root` font stack; never substitute a local
family for it.** A secondary local/generic fallback may follow for an HTML deliverable opened
offline, but it does not satisfy the slide render gate. The report distinguishes an unavailable
theme face (`fonts.missing`) from one that resolved but no element used (`fonts.unused`), and
`fonts.used` lists the families Chromium actually rasterized. Replacing the primary family with
`Liberation` or `Noto` does not fix a load failure; it permanently ships the deck in a generic
system face. The slide render uses `--require-webfonts`, so even an installed local face with the
same family name cannot satisfy the webfont policy. If the gate flags a face, check the family
spelling in `:root`, re-run `embed_deck_fonts.mjs`, and re-render. Gate both families at `400`;
add a heavier face to the gate only when the family publishes that exact weight.

**在 `:root` 字体栈中把 StylePlan 的确切字族放在第一位；绝不要用本地字族替代它。**针对离线打开的 HTML 交付物，可以在其后附加本地/通用回退，但它不满足幻灯片渲染门禁。报告会区分不可用的主题字体面（`fonts.missing`）与已解析但没有任何元素使用的字体面（`fonts.unused`），`fonts.used` 则列出 Chromium 实际光栅化的字族。用 `Liberation` 或 `Noto` 替换主字族并不能修复加载失败；那会让文稿永久以通用系统字体交付。幻灯片渲染使用 `--require-webfonts`，因此即使是已安装的同名本地字体面也无法满足网络字体策略。若门禁标记了某个字体面，检查 `:root` 中的字族拼写，重跑 `embed_deck_fonts.mjs`，再重新渲染。两个字族都以 `400` 门禁；只有当字族确实发布该字重时，才把更重的字面加入门禁。

## Canvas / 画布

Each slide is one `section.slide`, authored in its own file; assemble stacks them into the
deck's one self-contained document. These rules live in `deck.css`.

每张幻灯片是一个 `section.slide`，在自己的文件中撰写；assemble 把它们堆叠成文稿唯一的一份自包含文档。这些规则位于 `deck.css`。

```css
* { box-sizing: border-box; print-color-adjust: exact; -webkit-print-color-adjust: exact; }
html, body { margin: 0; background: #111; }
section.slide {
  width: 1280px; height: 720px;   /* 13.333in x 7.5in @96dpi; do NOT use min-height */
  overflow: hidden;               /* clips silently; budget content to ~620px usable */
  position: relative;
  background: var(--slide-bg);    /* opaque; an unset bg shows page grey and reads as broken */
  color: var(--slide-fg);
  font-family: var(--slide-font-body);
  page-break-after: always; break-after: page;
}
section.slide:last-child { page-break-after: auto; break-after: auto; }
```

Put the layout class on the section: `<section class="slide slide-hero-stats">`.

把布局类放在 section 上：`<section class="slide slide-hero-stats">`。

## Content layout classes / 内容布局类

Eight content layouts (the ninth/tenth are `cover` and `closing`). The StylePlan assigns one
per slide; treat it as the default and let content drive the final choice (a light slide can
drop to a simpler layout). Author each with CSS Grid per `authoring.md`; sizes come from the
typography rules there.

八种内容布局（第九/第十种是 `cover` 与 `closing`）。StylePlan 为每张幻灯片指定一种；把它当作默认值，由内容决定最终选择（内容较少的幻灯片可以降级为更简单的布局）。每种布局都按 `authoring.md` 用 CSS Grid 撰写；尺寸遵循其中的排版规则。

| Class | For |
|---|---|
| `slide-two-column` | two side-by-side prose/image columns |
| `slide-bento` | a balanced grid of mixed cells (stat, text, image) |
| `slide-hero-stats` | up to 3 big stat lockups (number over label) |
| `slide-headline-bullets` | a heading + ≤5 bullets |
| `slide-steps` | an ordered, left-aligned vertical sequence |
| `slide-comparison` | before/after or A-vs-B blocks |
| `slide-statement` | one centered editorial statement |
| `slide-gallery` | 2-3 subject-filling images in a row |

| 类 | 用途 |
|---|---|
| `slide-two-column` | 两个并排的正文/图像栏 |
| `slide-bento` | 混合单元格（数据、文字、图像）的均衡网格 |
| `slide-hero-stats` | 最多 3 个大号数据 lockup（数字在上、标签在下） |
| `slide-headline-bullets` | 标题 + ≤5 条要点 |
| `slide-steps` | 有序、左对齐的垂直序列 |
| `slide-comparison` | 前后对比或 A/B 对比块 |
| `slide-statement` | 一句居中的编辑性陈述 |
| `slide-gallery` | 一行 2-3 张充满主体的图像 |

`slide-statement`, `slide-hero-stats`, and the cover/closing use editorial-scale sizing (let
the heading/number be large); the rest follow the content sizes in `authoring.md`.

`slide-statement`、`slide-hero-stats` 与封面/结尾使用编辑级尺寸（让标题/数字做大）；其余布局遵循 `authoring.md` 中的内容尺寸。

## Cover hero scaffolds / 封面 hero 脚手架

The cover carries the hero image + the title only (see `authoring.md`). Pick the scaffold that
fits the image's composition. Fill the image edge-to-edge; never letterbox a hero; no
`border-radius`/`border`/`box-shadow`/padding on a hero image.

封面只承载 hero 图像与标题（见 `authoring.md`）。选择契合图像构图的脚手架。图像要铺满边缘；绝不要给 hero 加黑边（letterbox）；hero 图像上不要有 `border-radius`/`border`/`box-shadow`/padding。

**Full-bleed** (suits a 16:9 image, the usual choice; title sits in the image's empty region).
The scaffold is shared, so it goes in `deck.css`:  
**满幅（Full-bleed）**（适合 16:9 图像，是最常见的选择；标题放在图像的空白区域）。脚手架是共享的，因此放在 `deck.css` 中：  

```css
section.slide.cover {
  background-color: var(--slide-bg);
  background-size: cover; background-position: center;
}
section.slide.cover::after {              /* bottom-band scrim ONLY, never inset:0 */
  content: ''; position: absolute; left: 0; right: 0; bottom: 0; height: 55%;
  background: linear-gradient(transparent,
    color-mix(in srgb, var(--hero-scrim-color, #1a1a2e) 90%, transparent));
  z-index: 1;
}
.cover .title-block { position: absolute; bottom: 40px; left: 56px; right: 56px; z-index: 2; color: #fff; }
```

The embedded hero and its sampled scrim tone belong to that one slide, so they go in the slide
file's own `<style>`, scoped to its id (`workflow.md` step 5):  
内嵌的 hero 及其采样的遮罩色调只属于那一张幻灯片，因此放在该幻灯片文件自己的 `<style>` 中，并以其 id 限定作用域（`workflow.md` 第 5 步）：  

```html
<style>
  #cover {
    background-image: url('data:image/...');   /* the embedded hero */
    --hero-scrim-color: #1a1a2e;               /* a dark tone sampled from the image */
  }
</style>
```

**Split panel** (suits a tall/portrait image, image ≥50%, edge-bleed, text on its own panel).
Shared scaffold, in `deck.css`, image left out:  
**分栏面板（Split panel）**（适合竖幅图像，图像占 ≥50%，边缘出血，文字位于独立面板）。共享脚手架放在 `deck.css` 中，图像在左：  

```css
section.slide.cover { display: flex; padding: 0; }
.cover .image-panel {
  flex: 0 0 50%; position: relative;
  background-size: cover; background-position: center;
}
.cover .text-panel {
  flex: 0 0 50%; background: var(--slide-bg);
  display: flex; flex-direction: column; justify-content: center;
  padding: 64px 56px; color: var(--slide-fg);
}
```

The hero itself belongs to that one slide, so it goes in the slide file's own `<style>`, scoped:  
hero 本身属于那一张幻灯片，因此放在该幻灯片文件自己的 `<style>` 中并限定作用域：  

```html
<style>
  #cover .image-panel { background-image: url('data:image/...'); }
</style>
```

Put `.text-panel` first in DOM order to flip the image right. No standalone accent-bar div.

把 `.text-panel` 放在 DOM 顺序的第一位即可让图像换到右侧。不要有独立的强调条 div。

**Diagonal split** (one clean full-height diagonal; text on the plain `var(--slide-bg)`).
Shared scaffold, in `deck.css`, image left out:  
**对角分割（Diagonal split）**（一条干净的全高对角线；文字位于纯 `var(--slide-bg)` 上）。共享脚手架放在 `deck.css` 中，图像在左：  

```css
section.slide.cover { position: relative; padding: 0; background: var(--slide-bg); }
.cover .image-panel {
  position: absolute; inset: 0 0 0 0; width: 100%;
  background-size: cover; background-position: center;
  clip-path: polygon(0 0, 58% 0, 42% 100%, 0 100%);
}
.cover .text-panel {
  position: absolute; top: 0; right: 0; bottom: 0; width: 42%;
  display: flex; flex-direction: column; justify-content: center;
  padding: 56px 60px 56px 40px; z-index: 2; color: var(--slide-fg);
}
```

Hero in the slide file's own `<style>`, scoped the same way:  
hero 放在该幻灯片文件自己的 `<style>` 中，以相同方式限定作用域：  

```html
<style>
  #cover .image-panel { background-image: url('data:image/...'); }
</style>
```

Don't give `.text-panel` its own background/border (that creates a second straight edge).

不要给 `.text-panel` 自己的背景/边框（那会产生第二条直线边缘）。

Every cover scaffold splits the same way: structure shared in `deck.css`, the embedded image in the
slide's own scoped `<style>`. A `data:` hero in `deck.css` would make every slide carry one slide's
image, and an unscoped selector in a slide file is refused by assemble.

每种封面脚手架都以相同方式拆分：结构共享在 `deck.css`，内嵌图像放在幻灯片自己的限作用域 `<style>` 中。`deck.css` 里的 `data:` hero 会让每张幻灯片都携带某一张幻灯片的图像，而幻灯片文件中的未限定作用域选择器会被 assemble 拒绝。

【评论】把共享结构与单页资产严格分离（`deck.css` 放结构、幻灯片文件放 `data:` 图像），是为了控制组装体积并避免选择器串页，属于对组装流水线的工程约束。

## Lockups (text + image on a content slide) / Lockup（内容幻灯片上的文字 + 图像）

Use only the deck's two assigned lockups (`StylePlan.lockups`, drawn from L1/L2/L4). Fill the
image cell edge-to-edge (`width:100%; height:100%; object-fit:cover; border-radius:20px`) so no
blank band shows.

只使用文稿被指派的两个 lockup（`StylePlan.lockups`，从 L1/L2/L4 中选取）。图像单元格要铺满边缘（`width:100%; height:100%; object-fit:cover; border-radius:20px`），避免出现空白条带。

- **L1 left-heavy**: `grid-template-columns:55fr 45fr; grid-template-rows:auto 1fr;` image cell
  `grid-row:2; grid-column:1`, text cell `grid-column:2` vertically centered (`align-self:center`).
  **L1 左重**：`grid-template-columns:55fr 45fr; grid-template-rows:auto 1fr;` 图像单元格 `grid-row:2; grid-column:1`，文字单元格 `grid-column:2` 垂直居中（`align-self:center`）。
- **L2 right-heavy**: L1 mirrored (text column 1, image column 2).
  **L2 右重**：L1 的镜像（文字在第 1 列，图像在第 2 列）。
- **L4 asymmetric** (the strongest; prefer it): 60/40 grid, image fills its cell, text inset
  with 56px padding.
  **L4 非对称**（效果最强；优先使用）：60/40 网格，图像填满其单元格，文字内缩并留 56px padding。

## Closing / 结尾

Centered text on one uniform background (see `authoring.md` for the full closing rule). No
scrim unless text overlays a background photo.

居中文字置于单一均匀背景上（完整的结尾规则见 `authoring.md`）。除非文字叠加在背景照片上，否则不加遮罩。
