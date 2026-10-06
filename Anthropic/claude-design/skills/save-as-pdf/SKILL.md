---
name: save-as-pdf
description: "Print-ready PDF export"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Save as PDF / 另存为 PDF

Reformat the current HTML design into a paginated, paper-ready PDF. The "Instant" export already gives the user a PDF at the design's native pixel size — this path is for when they want real pages.

将当前 HTML 设计重新排版为分页的、适合纸张的 PDF。"Instant（即时）"导出已经能按设计的原生像素尺寸给用户一个 PDF——这条路径适用于用户想要真实页面的场景。

**Do NOT rasterize the page into a PDF.** Never use jsPDF, html2canvas, dom-to-image, or any other canvas/screenshot-to-PDF approach — they produce blurry, non-selectable, oversized output, and do not generate a PDF binary yourself. PDF export is print-based: a print-ready copy is handed to `show_pdf_export_dialog` and the browser's own print engine renders crisp, selectable, text-based pages. The only supported way to make a copy print-ready is a component that owns its print geometry — the doc_page starter for documents, or a source already built on `<deck-stage>` or `<doc-page>`. Do not hand-author `@page` rules or print CSS resets, and NEVER declare `<meta name="omelette-owns-print">` yourself — the starters announce print ownership at runtime, and a hand-authored meta tells the platform to trust print CSS you would then have to hand-write and maintain. Hand-rolled print CSS behind that meta is the legacy path; the doc_page starter replaces it everywhere it is available.

**不要把页面光栅化成 PDF。**绝不要使用 jsPDF、html2canvas、dom-to-image 或任何其他 canvas/截图转 PDF 的方案——它们产出的内容模糊、不可选中且体积过大；也不要自行生成 PDF 二进制。PDF 导出基于打印：一份适合打印的副本被交给 `show_pdf_export_dialog`，由浏览器自身的打印引擎渲染清晰、可选中、基于文本的页面。让副本适合打印的唯一受支持方式，是使用一个拥有自身打印几何结构的组件——文档用 doc_page starter，或源文件本就构建于 `<deck-stage>` 或 `<doc-page>` 之上。不要手写 `@page` 规则或打印 CSS 重置，也绝不要自己声明 `<meta name="omelette-owns-print">`——starter 会在运行时宣告打印所有权，而手写的 meta 会让平台信任你随后不得不手写并维护的打印 CSS。该 meta 背后手写的打印 CSS 是遗留路径；在可用的地方，doc_page starter 已将其全面取代。

【评论】本段通过明令禁止光栅化方案并强制走浏览器打印引擎，从机制上保证导出 PDF 的文本可选中与清晰度，是典型的"约束工具选择以保障输出质量"的设计。

### Steps / 步骤

1. **Read the current HTML design file** to understand its structure and content. Re-read it on every PDF request, even if you read it or made a print copy earlier in this conversation — the user may have changed content or tweak values (the Tweaks panel writes into the source file) since then. Note the `[version: v<N>]` token in the read result's header — step 2's provenance stamp needs it, and only a version read in THIS request is valid to stamp.
   **阅读当前 HTML 设计文件**以理解其结构与内容。每次 PDF 请求都要重新读取，即使你在本对话早些时候已经读过或制作过打印副本——用户此后可能已更改内容或微调值（Tweaks 面板会写回源文件）。注意读取结果头部中的 `[version: v<N>]` 标记——步骤 2 的来源戳（provenance stamp）需要它，且只有在本次请求中读到的版本才可用于加盖。

2. **Write the print copy, stamped with its provenance.** Always write it fresh from the source you just read — an existing `-print` copy from an earlier request is a stale snapshot, and reusing or only partially updating it ships outdated values to the PDF. The print file path is the source path with `-print` inserted before the extension — same directory, same basename. If the source is `slides/deck.html`, write `slides/deck-print.html`; if the source is `web/index.html`, write `web/index-print.html`. **Do NOT** use the deck title or project name as the filename, and **do NOT** write to the project root if the source is in a subdirectory — any change in directory depth breaks every relative URL (`@font-face` `src: url(...)`, `<img src>`, `<link href>`, CSS `background: url(...)`) and the print tab shows missing images and system-font fallbacks.
   **写出打印副本，并加盖来源戳。**永远依据你刚读取的源文件全新写出——来自更早请求的既有 `-print` 副本是过期快照，复用它或只做部分更新会把过时的值送进 PDF。打印文件路径是在源路径扩展名之前插入 `-print`——同目录、同基名。若源文件是 `slides/deck.html`，则写 `slides/deck-print.html`；若源文件是 `web/index.html`，则写 `web/index-print.html`。**不要**用幻灯片标题或项目名作文件名，且当源文件位于子目录时**不要**写到项目根目录——目录深度的任何变化都会破坏所有相对 URL（`@font-face` 的 `src: url(...)`、`<img src>`、`<link href>`、CSS 的 `background: url(...)`），打印标签页就会出现图片缺失和系统字体回退。

   **Stamp the copy's provenance (required whenever the read header shows a version — every arm of this step).** Include this tag in the copy's `<head>`, carrying the version from step 1's read header and the source's project-relative path (for a read header of `[File: designs/report.html] [version: v172]`):
   **为副本加盖来源戳（只要读取头部显示了版本号就必须执行——本步骤的每个分支都适用）。**在副本的 `<head>` 中加入这个标签，携带步骤 1 读取头部中的版本号和源文件的项目相对路径（例如读取头部为 `[File: designs/report.html] [version: v172]` 时）：
   ```html
   <meta name="omelette-print-source" content="v172 designs/report.html">
   ```
   `show_pdf_export_dialog` REFUSES a copy whose stamp is missing or no longer matches the source's current version — that is what makes a stale copy un-exportable. If it refuses with a stale-copy error, go back to step 1 (re-read the source) and rewrite the copy fresh; never just add or edit the stamp on an existing copy. If the read header shows no `[version: ...]` token, the project isn't versioned — omit the stamp; the export tool skips the freshness check there.
   若副本的来源戳缺失、或与源文件当前版本不再匹配，`show_pdf_export_dialog` 会**拒绝**该副本——正是这一点让过期副本无法被导出。如果它以过期副本错误拒绝，请回到步骤 1（重新读取源文件）并全新重写副本；绝不要只在既有副本上添加或修改来源戳。如果读取头部没有 `[version: ...]` 标记，说明项目未启用版本控制——省略来源戳；此时导出工具会跳过新鲜度检查。

   **If the source is already built on `<deck-stage>` or `<doc-page>`, the copy is the source plus content-level print rules only.** Both components own their print geometry — never add an `@page` rule or reflow their layout. For `<deck-stage>` decks, set `data-deck-active` on **every** direct-child slide (not just the current one) so `[data-deck-active]`-keyed entrance styles resolve on every page — each slide is already one page. For `<doc-page>` documents there is nothing structural to do.
   **如果源文件已构建于 `<deck-stage>` 或 `<doc-page>` 之上，副本就是源文件加上仅内容层的打印规则。**这两个组件都拥有自己的打印几何结构——绝不要添加 `@page` 规则或重排其布局。对 `<deck-stage>` 幻灯片，要在**每一张**直接子级幻灯片上（而不只是当前那张）设置 `data-deck-active`，以便以 `[data-deck-active]` 为键的入场样式在每一页都能生效——每张幻灯片本来就是一页。对 `<doc-page>` 文档则无需任何结构性处理。

   **Otherwise, rebuild the content on the doc_page starter.** Call `copy_starter_component` with `kind: "doc_page.js"` (once per project — the component file persists), then decide the pagination UP FRONT: a FLOWING document (pour the content into `<doc-page margin="0.75in">` as one normal HTML flow; the print engine paginates it — the default for reports, memos, letters, long-form), or EXPLICIT pagination (one `<section class="page">` child per page — when the user asks for a page count or the design implies one: a one-page resume, a two-sided flier, a certificate, a brochure). If in doubt, ask the user. Keep the design's typography, colors, and imagery intact either way. For FLOWING documents the component pins no paper size — the print engine paginates onto the user's real paper. Explicitly paginated pages print at a FIXED page box with overflow hidden — letter by default (set size="a4" for a clearly metric user), the user's chosen paper when they export — and content that misses the box is clipped, never reflowed: design each page to FILL the page box and fit letter and A4 alike without overlap (no viewport units — they track the window, not the page). `orientation="landscape"` for landscape sheets. The component owns the sheet, the pagination, and all print geometry — do NOT write your own `@page` rule, print-CSS reset, page-card divs, hard-coded paper dimensions, or `break-after: page` fake sheets. In flowing documents use `break-before: page` only where a section genuinely starts a new chapter; long tables get a `<thead>` so the header repeats on every page.
   **否则，在 doc_page starter 上重建内容。**调用 `copy_starter_component` 并传 `kind: "doc_page.js"`（每个项目一次——组件文件会保留），然后预先决定分页方式：流式（FLOWING）文档（把内容作为一个普通 HTML 流倒入 `<doc-page margin="0.75in">`；由打印引擎分页——报告、备忘录、信函、长文的默认选择），或显式（EXPLICIT）分页（每页一个 `<section class="page">` 子元素——当用户要求页数或设计本身暗示页数时：单页简历、双面传单、证书、宣传册）。有疑问就问用户。无论哪种方式，都要保持设计的排版、颜色和图像原样不动。对流式文档，组件不固定纸张尺寸——打印引擎会按用户的真实纸张分页。显式分页的页面以固定（FIXED）页面盒打印且溢出隐藏——默认 letter（对明显使用公制的用户设 size="a4"），导出时用用户选择的纸张——超出页面盒的内容会被裁剪、绝不重排：把每一页设计成填满页面盒，并能同时适配 letter 和 A4 且不重叠（不要用视口单位——它们跟随窗口而非页面）。横向纸张用 `orientation="landscape"`。组件拥有纸张、分页和全部打印几何结构——不要自写 `@page` 规则、打印 CSS 重置、页面卡片 div、硬编码的纸张尺寸，或用 `break-after: page` 伪造纸张。在流式文档中，只在某一节真正开启新章节处使用 `break-before: page`；长表格要加 `<thead>`，让表头在每页重复。

   **A fixed-canvas design (poster, social graphic, infographic) also goes through the doc_page rebuild, with one decision: the page.** Print it at its true dimensions — `<doc-page width="18in" height="24in" margin="0">`, the page IS the design (use explicit width/height ONLY when the user gives or implies a real physical size) — or scale it onto standard paper — `<doc-page size="letter" content-width="960px" content-height="1440px">` (`size="a4"` when the user is clearly metric), where the content lays out at its authored size and the component scales it to fit that sheet's printable area. Scaled-fit is the ONE place a named size still matters: the component must compute the fit against a known sheet (the export dialog re-fits to the user's actual paper choice at print time where available). When the user's intent between the two isn't clear from their request, ask — in plain terms (print it poster-sized, or fit it onto regular paper?) — before exporting. Never hand-scale with your own CSS transforms; the component owns the scaling either way.
   **固定画布设计（海报、社交图片、信息图）同样要经过 doc_page 重建，只需做一个决定：页面本身。**按其真实尺寸打印——`<doc-page width="18in" height="24in" margin="0">`，页面即设计（只有当用户给出或暗示了真实物理尺寸时才使用显式 width/height）——或将其缩放到标准纸张上——`<doc-page size="letter" content-width="960px" content-height="1440px">`（用户明显使用公制时用 `size="a4"`），此时内容按其原始尺寸排版，组件将其缩放以适应该纸张的可打印区域。等比缩放适配是唯一命名尺寸仍然重要的场景：组件必须针对已知纸张计算适配（在可用时，导出对话框会在打印时按用户实际选择的纸张重新适配）。当用户的意图在两者之间不明确时，先询问——用直白的说法（是按海报尺寸打印，还是适配到普通纸张上？）——然后再导出。绝不要用自己的 CSS 变换手动缩放；无论哪种方式都由组件负责缩放。

   In every copy, add the color-adjust rule so backgrounds and colors match the preview — do NOT strip backgrounds from the design:
   在每个副本中都要加入 color-adjust 规则，让背景和颜色与预览一致——不要从设计中剥离背景：
   ```css
   * { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
   ```

   **Jump animations to their end state.** Do NOT use `animation: none` (that reverts fade-ins to the hidden base). Instead freeze every animation at its final frame and disable transitions:
   **让动画直接跳到结束状态。**不要使用 `animation: none`（那会把淡入效果回退到隐藏的初始状态）。而是把每个动画冻结在其最后一帧并禁用过渡：
   ```css
   * { animation-delay: -99s !important; animation-duration: .001s !important;
       animation-iteration-count: 1 !important; animation-fill-mode: both !important;
       animation-play-state: running !important; transition-duration: 0s !important; }
   ```

3. **Test the file** by showing it with `show_html`, then make sure there are no JS errors. No need to screenshot unless asked.
   **测试文件**：用 `show_html` 展示它，然后确认没有 JS 错误。除非被要求，无需截图。

4. **Call the `show_pdf_export_dialog` tool** with the project-relative path to the print-ready file. The print-firing code is injected into the print copy automatically when you call this tool — do NOT write an auto-print or `window.print()` script yourself. Unless the export started from the user's own Export click, the tool does not open anything itself: it presents an export dialog, and the user's **Continue to export** click there is what opens the print view. The tool result says which happened — when it says the dialog is waiting on the user, the print view has NOT opened; say so plainly and never claim it has. The tool fails with a reason if the file (and its print source) is not print-based — the doc_page rebuild from step 2 is exactly what keeps the copy eligible.
   **调用 `show_pdf_export_dialog` 工具**，传入适合打印的文件的项目相对路径。调用该工具时，触发打印的代码会被自动注入打印副本——不要自己编写自动打印或 `window.print()` 脚本。除非导出是由用户自己点击 Export 发起的，否则该工具自身不会打开任何东西：它会呈现一个导出对话框，用户在那里点击 **Continue to export** 才会打开打印视图。工具结果会说明发生了哪一种——当它表示对话框正在等待用户时，打印视图尚未打开；要如实说明，绝不要声称已经打开。如果文件（及其打印源）不是基于打印的，该工具会附带原因失败——步骤 2 的 doc_page 重建正是让副本保持合格条件的做法。

### Important Notes / 重要说明

- The goal is a file that prints cleanly on real pages — the doc_page component owns the pagination; your job is the content
  目标是一个能在真实页面上干净打印的文件——分页由 doc_page 组件负责；你的职责是内容
- Maintain visual fidelity — keep the design's typography, colors, and imagery intact
  保持视觉保真——设计的排版、颜色和图像原样保留
- For `<deck-stage>` decks, each slide stays on its own page; `<doc-page>` documents flow and paginate themselves
  对 `<deck-stage>` 幻灯片，每张幻灯片独占一页；`<doc-page>` 文档自行流动并分页
- For prompt-driven exports, `show_pdf_export_dialog` waits on the user: the export dialog's **Continue to export** click opens the print view, and until then nothing has opened — report the state the tool result describes, not the state you expect
  对于由提示词驱动的导出，`show_pdf_export_dialog` 会等待用户：导出对话框的 **Continue to export** 点击才会打开打印视图，在此之前什么都没有打开——汇报工具结果所描述的状态，而不是你期望的状态
- The `-print.html` is plumbing for the print tab, not a deliverable — `show_pdf_export_dialog` is the only delivery step. Do NOT `present_fs_item_for_download` it; its relative asset paths only resolve via the project file server and break when opened standalone.
  `-print.html` 是打印标签页的内部管道产物，不是交付物——`show_pdf_export_dialog` 才是唯一的交付步骤。不要对它调用 `present_fs_item_for_download`；它的相对资源路径只能通过项目文件服务器解析，独立打开时会失效。

【评论】来源戳与"拒绝过期副本"的校验机制，是把 freshness 检查下沉到导出工具侧的防御性设计；同时要求"每次重新读取源文件"以避免会话内缓存导致的陈旧导出。
