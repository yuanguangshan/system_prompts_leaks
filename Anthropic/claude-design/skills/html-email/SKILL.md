<!-- BILINGUAL-EN-ZH -->
---
name: html-email
description: "Send-ready single-file email"
user-invocable: true
---

# HTML email / HTML 邮件

Design an HTML email as ONE self-contained .html file that survives
real email clients. Email rendering is not browser rendering — Gmail,
Outlook, and Apple Mail each strip or mangle different things, and the
rules below are what reliably survives all three. When a rule here
conflicts with normal web-design instincts, the rule wins.

将 HTML 邮件设计为一个自包含的 .html 文件，使其能在真实邮件客户端中完好呈现。邮件渲染不等于浏览器渲染——Gmail、Outlook 和 Apple Mail 各自会剥离或破坏不同的东西，下述规则正是经证实能在三者中同时存活的写法。当这里的规则与常规网页设计直觉冲突时，以规则为准。

Layout and styling:
- Structure with nested `<table role="presentation" cellpadding="0"
  cellspacing="0" border="0">` — no flexbox, no grid, no floats, no
  position. One centered wrapper table, max-width 600px, single-column
  flow (stacked rows beat side-by-side columns).
- Inline EVERY style on the element it styles. A `<style>` block in
  `<head>` may additionally carry only what can't inline (media queries,
  dark-mode tweaks) — several clients drop it entirely, so the email
  must read correctly from inline styles alone.
- No JavaScript anywhere (universally stripped). No external
  stylesheets. No web fonts — use email-safe stacks (Arial, Helvetica,
  Georgia, Verdana, Tahoma, 'Courier New') with generic fallbacks.
- Build the visual design out of colored table cells, borders, spacer
  cells, and type — not images. There is nowhere to host project
  images from here: a referenced project file will not exist for
  recipients. If imagery is essential, leave a clearly-marked
  placeholder cell with alt text and tell the user to swap in a hosted
  https URL before sending.
- Buttons are "bulletproof": a padded `<td>` with bgcolor and inline
  border-radius, the `<a>` filling it with display:block and inline
  color — never an image, never a styled `<button>`.

布局与样式：
- Structure with nested `<table role="presentation" cellpadding="0"
  cellspacing="0" border="0">` — no flexbox, no grid, no floats, no
  position. One centered wrapper table, max-width 600px, single-column
  flow (stacked rows beat side-by-side columns).
  - 使用嵌套的 `<table role="presentation" cellpadding="0"
    cellspacing="0" border="0">` 构建结构——不用 flexbox、不用 grid、不用浮动、不用 position。使用一个居中的外层包裹表格，max-width 600px，单列流式布局（堆叠行优于并排列）。
- Inline EVERY style on the element it styles. A `<style>` block in
  `<head>` may additionally carry only what can't inline (media queries,
  dark-mode tweaks) — several clients drop it entirely, so the email
  must read correctly from inline styles alone.
  - 每一条样式都内联在它所作用的元素上。`<head>` 中的 `<style>` 块只可额外承载无法内联的内容（媒体查询、深色模式微调）——若干客户端会将其完全丢弃，因此邮件仅凭内联样式也必须能正确呈现。
- No JavaScript anywhere (universally stripped). No external
  stylesheets. No web fonts — use email-safe stacks (Arial, Helvetica,
  Georgia, Verdana, Tahoma, 'Courier New') with generic fallbacks.
  - 任何地方都不用 JavaScript（会被普遍剥离）。不用外部样式表。不用网络字体——使用邮件安全字体栈（Arial、Helvetica、Georgia、Verdana、Tahoma、'Courier New'）并附带通用回退字体。
- Build the visual design out of colored table cells, borders, spacer
  cells, and type — not images. There is nowhere to host project
  images from here: a referenced project file will not exist for
  recipients. If imagery is essential, leave a clearly-marked
  placeholder cell with alt text and tell the user to swap in a hosted
  https URL before sending.
  - 视觉设计由着色表格单元格、边框、占位空白单元格和文字构成——不要用图片。从这里无法托管项目图片：被引用的项目文件对收件人来说并不存在。如果图像不可或缺，就留下一个标记清晰的占位单元格并附上 alt 文本，并告知用户发送前替换为托管的 https URL。
- Buttons are "bulletproof": a padded `<td>` with bgcolor and inline
  border-radius, the `<a>` filling it with display:block and inline
  color — never an image, never a styled `<button>`.
  - 按钮采用"防弹（bulletproof）"写法：带内边距的 `<td>`，设置 bgcolor 和内联 border-radius，`<a>` 以 display:block 填满并使用内联颜色——绝不用图片，绝不用样式化的 `<button>`。

【评论】"bulletproof button"是邮件开发领域的标准术语：由于 Outlook（Word 渲染引擎）对背景图片和按钮元素的支持不可靠，业界通行做法是用表格单元格加背景色来模拟按钮，以保证各客户端下的点击区域完整。

Client quirks that matter:
- Outlook (Word engine): give every table/cell explicit widths; set
  line-height with mso-line-height-rule:exactly; wrap Outlook-only
  fixes in `<!--[if mso]> … <![endif]-->` conditionals.
- Gmail clips messages beyond ~100KB of HTML — stay well under.
- Add `<meta name="color-scheme" content="light dark">` and pick colors
  that survive dark-mode inversion (avoid pure `#000/#fff` backgrounds;
  test text on mid-tone fills).

值得注意的客户端特性：
- Outlook (Word engine): give every table/cell explicit widths; set
  line-height with mso-line-height-rule:exactly; wrap Outlook-only
  fixes in `<!--[if mso]> … <![endif]-->` conditionals.
  - Outlook（Word 引擎）：为每个表格/单元格显式指定宽度；用 mso-line-height-rule:exactly 设置行高；把仅针对 Outlook 的修补包在 `<!--[if mso]> … <![endif]-->` 条件注释中。
- Gmail clips messages beyond ~100KB of HTML — stay well under.
  - Gmail 会裁剪 HTML 超过约 100KB 的邮件——务必远低于该阈值。
- Add `<meta name="color-scheme" content="light dark">` and pick colors
  that survive dark-mode inversion (avoid pure `#000/#fff` backgrounds;
  test text on mid-tone fills).
  - 添加 `<meta name="color-scheme" content="light dark">`，并选用能在深色模式反色下依然成立的颜色（避免纯 `#000/#fff` 背景；在中间色调底色上测试文字）。

Deliverability and accessibility:
- First element in `<body>`: a hidden preheader span (~85 chars) that
  previews next to the subject line.
- alt text on any image, lang on `<html>`, real `<a href>` links (no
  dead # anchors), and a footer with a plausible unsubscribe line and
  postal address for anything marketing-shaped.

送达率与无障碍：
- First element in `<body>`: a hidden preheader span (~85 chars) that
  previews next to the subject line.
  - `<body>` 中的第一个元素：一个隐藏的预览头（preheader）span（约 85 个字符），它会显示在主题行旁边。
- alt text on any image, lang on `<html>`, real `<a href>` links (no
  dead # anchors), and a footer with a plausible unsubscribe line and
  postal address for anything marketing-shaped.
  - 任何图片都要有 alt 文本，`<html>` 上要有 lang 属性，使用真实的 `<a href>` 链接（不用失效的 # 锚点）；凡外观接近营销邮件的，页脚要包含一条可信的退订说明和邮政地址。

Show the design at 600px; mention in your reply that the file is
send-ready HTML the user can drop into their email tool.

以 600px 宽度展示设计；并在回复中说明该文件是可直接放入用户邮件工具发送的 HTML。
