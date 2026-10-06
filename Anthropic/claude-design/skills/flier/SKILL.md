<!-- BILINGUAL-EN-ZH -->
---
name: flier
description: "Print-ready single page"
user-invocable: true
---

# Flier / 单页宣传单

Design a print-ready, single-page flier as one self-contained HTML
file: exactly one fixed-size page.

将一张适合打印的单页宣传单设计为一个自包含的 HTML 文件：严格只有一页、固定尺寸。

Sheet setup:
- START by calling copy_starter_component with kind: "doc_page.js",
  then build the flier as ONE explicitly paginated page:
  <doc-page><section class="page" id="flier">…</section></doc-page>.
  The component owns the page box, the desk background that
  disappears at print, the page break discipline, and all print
  geometry — do NOT write your own @page rule, body background, or
  hard-coded paper dimensions, and never add an omelette-owns-print
  meta. The print dialog must show exactly 1 page.
- The flier prints as a FIXED full-bleed page box with overflow
  hidden — letter by default, the user's chosen paper (letter, A4, …)
  when they export — so content that misses the box is clipped, never
  reflowed. Design the flier to FILL the page box and fit letter and
  A4 alike without overlap.
- The page is full-bleed: the flier owns its own inset — keep a
  visual margin of at least 4% of the page on every edge, and keep
  everything inside it.
- Never viewport units (vh/vw) — they track the window, not the page.

页面设置：
- START by calling copy_starter_component with kind: "doc_page.js",
  then build the flier as ONE explicitly paginated page:
  <doc-page><section class="page" id="flier">…</section></doc-page>.
  The component owns the page box, the desk background that
  disappears at print, the page break discipline, and all print
  geometry — do NOT write your own @page rule, body background, or
  hard-coded paper dimensions, and never add an omelette-owns-print
  meta. The print dialog must show exactly 1 page.
  - 首先调用 copy_starter_component（kind: "doc_page.js"），然后把宣传单构建为唯一一个显式分页的页面：<doc-page><section class="page" id="flier">…</section></doc-page>。该组件负责页面盒、打印时消失的桌面背景、分页纪律以及所有打印几何参数——不要自行编写 @page 规则、body 背景或硬编码的纸张尺寸，也不要添加 omelette-owns-print 元标签。打印对话框必须恰好显示 1 页。
- The flier prints as a FIXED full-bleed page box with overflow
  hidden — letter by default, the user's chosen paper (letter, A4, …)
  when they export — so content that misses the box is clipped, never
  reflowed. Design the flier to FILL the page box and fit letter and
  A4 alike without overlap.
  - 宣传单以固定的满版出血页面盒打印，溢出部分隐藏——默认 letter 纸张，用户导出时使用其选择的纸张（letter、A4 等）——因此超出页面盒的内容会被裁剪，而不是重新排版。设计时应让宣传单充满页面盒，并同时适配 letter 和 A4 而不产生重叠。
- The page is full-bleed: the flier owns its own inset — keep a
  visual margin of at least 4% of the page on every edge, and keep
  everything inside it.
  - 页面是满版出血的：宣传单自管内边距——每条边至少保留页面尺寸 4% 的视觉边距，并把所有内容都控制在该边距之内。
- Never viewport units (vh/vw) — they track the window, not the page.
  - 绝不使用视口单位（vh/vw）——它们跟随的是窗口，而不是页面。

A flier is read at a distance, in passing, in under three seconds:
- One dominant element — usually a headline under ~6 words — sized so
  it reads across a room (think 8cqh+), everything else clearly
  subordinate.
- The five Ws grouped tight and scannable: what, when, where, cost,
  and one way to act (QR-sized URL, phone number, or tear-off) — not
  scattered through prose.
- Strong flat color blocks and vector shapes over photos and
  gradients; high contrast; body text in near-black on light stock.
- Generous whitespace beats more words — cut copy until the hierarchy
  is unmissable.
- Optional: a tear-off fringe along the bottom edge (a row of
  narrow cells with dashed left borders and rotated contact text) when
  the user wants phone-number slips.

宣传单是隔着距离、顺手一瞥、在三秒之内读完的：
- One dominant element — usually a headline under ~6 words — sized so
  it reads across a room (think 8cqh+), everything else clearly
  subordinate.
  - 一个主导元素——通常是不超过约 6 个词的标题——尺寸要大到隔着整个房间也能读清（按 8cqh 以上考虑），其余一切元素都明显处于从属地位。
- The five Ws grouped tight and scannable: what, when, where, cost,
  and one way to act (QR-sized URL, phone number, or tear-off) — not
  scattered through prose.
  - 五要素（5W）紧密分组、便于扫读：活动内容、时间、地点、费用，以及一种行动方式（二维码尺寸的 URL、电话号码或撕下条）——不要散落在正文叙述中。
- Strong flat color blocks and vector shapes over photos and
  gradients; high contrast; body text in near-black on light stock.
  - 优先使用强烈的纯色色块和矢量图形，而非照片和渐变；高对比度；正文文字在浅色纸面上使用近黑色。
- Generous whitespace beats more words — cut copy until the hierarchy
  is unmissable.
  - 大量留白胜过更多文字——不断删减文案，直到层级关系一目了然。
- Optional: a tear-off fringe along the bottom edge (a row of
  narrow cells with dashed left borders and rotated contact text) when
  the user wants phone-number slips.
  - 可选：当用户需要电话号码撕下条时，沿底边制作一排撕下联（一排窄格，左侧为虚线边框，联系文字旋转排布）。

Check the print preview: nothing clipped, nothing spilling to page
two, colors that still work in grayscale.

检查打印预览：没有任何内容被裁剪，没有任何内容溢出到第二页，颜色在灰度下依然有效。
