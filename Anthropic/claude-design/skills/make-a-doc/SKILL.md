<!-- BILINGUAL-EN-ZH -->
---
name: make-a-doc
description: "Page-style document, printable out of the box"
user-invocable: true
---

# Make a doc / 制作文档

Create a document (resume, one-pager, memo, letter, report, guide,
paper). First decide which of two shapes the user wants — they export
completely differently:

创建一份文档（简历、单页、备忘录、信函、报告、指南、论文）。首先判断用户想要两种形态中的哪一种——两者的导出方式完全不同：

**Flowing pages** — text that pours onto standard sheets (Letter/A4)
and breaks wherever needed: reports, memos, letters, papers, guides.
START by calling `copy_starter_component` with
`kind: "doc_page.js"`, then write the whole document as normal
flowing HTML inside `<doc-page size="letter" margin="0.75in">`.
The component owns the sheet, the desk background, and all print
geometry — do NOT write your own `@page` rule, body background,
page-card divs, `break-after: page` fake sheets, or
`break-inside: avoid` on items inside multi-column grids (a grid only
breaks between rows, so a kept row that doesn't fit leaves a blank band).

**流式页面**——文本倾注到标准纸张（Letter/A4）上、在需要处换页：报告、备忘录、信函、论文、指南。首先调用 `copy_starter_component`（`kind: "doc_page.js"`），然后在 `<doc-page size="letter" margin="0.75in">` 内以普通流式 HTML 编写整个文档。该组件负责纸张、桌面背景和所有打印几何参数——不要自行编写 `@page` 规则、body 背景、页面卡片 div、用 `break-after: page` 伪造的分页，也不要对多列网格内的条目设置 `break-inside: avoid`（网格只能在行与行之间断开，被强制的行放不下时会留下一整条空白带）。

Print rules for flowing pages: multi-column text uses CSS columns
(`column-count` + `column-gap`; `column-span: all` on a heading
that spans; `hyphens: auto` in narrow columns — it needs `lang`
on the html element), never side-by-side
flex/grid columns — only real CSS columns flow and break across pages.
Use `break-before: page` on anything that must start a new page (a
chapter, an appendix); add custom kept-together blocks (callouts, stat
tiles, cards) to a `break-inside: avoid` rule and keep each shorter
than a page — the component already keeps headings with their content,
keeps figures/code/table rows whole, and suppresses orphans and widows
(extend `orphans: 3; widows: 3` to custom text blocks). Long tables
get a `<thead>` so the header repeats on every page. No
`position: fixed`/`sticky` and no viewport units in content —
fixed elements stamp every printed page (running headers/footers go in
the component's slots) and `100vh` mis-sizes at print.

流式页面的打印规则：多栏文本使用 CSS 多栏布局（`column-count` + `column-gap`；跨栏标题使用 `column-span: all`；窄栏中启用 `hyphens: auto`——这需要在 html 元素上设置 `lang`），绝不用并排的 flex/grid 列——只有真正的 CSS 多栏才能跨页流动和断开。对任何必须另起新页的内容（章节、附录）使用 `break-before: page`；把自定义的整体保持块（标注框、统计卡片、卡片）加入 `break-inside: avoid` 规则，并保证每块都短于一页——组件本身已保证标题与其内容同页、图形/代码/表格行保持完整，并抑制孤行与寡行（可将 `orphans: 3; widows: 3` 扩展到自定义文本块）。长表格使用 `<thead>`，使表头在每一页重复。内容中禁用 `position: fixed`/`sticky` 和视口单位——固定元素会印到每一页上（页眉/页脚应放入组件的插槽），而 `100vh` 在打印时尺寸会出错。

**Fixed sheet** — a design that must fill exactly one page of fixed
dimensions: poster, infographic, social graphic, certificate. No
starter component — build it at its true pixel size with an explicit
px `width` (and `height` if fixed) on the top-level element; the
export sizes the PDF page to it automatically. Do not write any
`@page` rule for it.

**固定版面**——必须恰好填满一页固定尺寸的设计：海报、信息图、社交媒体图、证书。不使用 starter 组件——以真实像素尺寸构建，在顶层元素上显式指定 px `width`（如尺寸固定则加 `height`）；导出时会自动按其尺寸生成 PDF 页面。不要为它编写任何 `@page` 规则。

Styling (both shapes): body type 14–16px with generous line-height
(1.55–1.7); clear heading hierarchy; restrained palette. Tables get a
header row and hairline borders; figures and code blocks each carry a
short caption. Open with the document's own h1 as the first body
element (use any header-shaped first line of pasted content as that h1
rather than rendering it as a separate masthead).

样式（两种形态通用）：正文字号 14–16px，行高宽松（1.55–1.7）；标题层级清晰；配色克制。表格有表头行和细边框；图形和代码块各带简短说明文字。以文档自身的 h1 作为 body 的第一个元素开场（若粘贴内容的首行形似标题，直接用作该 h1，而不要把它渲染成单独的报头）。
