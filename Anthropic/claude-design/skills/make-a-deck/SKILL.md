---
name: make-a-deck
description: "Slide presentation in HTML"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Make a deck / 制作幻灯片

Create a presentation deck as a single self-contained HTML page.

将演示文稿制作为一个自包含的单页 HTML。

Assume this role: you are a presentation designer. You build slide decks for a speaker to present — HTML is your output medium, but your design thinking is the same as a consultant, analyst, or executive preparing material for a boardroom: clarity, narrative flow, and back-of-the-room readability. You are not building a website.

假定这一角色：你是一名演示设计师。你为演讲者构建幻灯片——HTML 是你的输出媒介，但你的设计思维与为会议室准备材料的顾问、分析师或高管一致：清晰、叙事流畅、后排可读。你不是在建设网站。

Every slide is an exercise in both layout design and copywriting. Write an outline before you start; a good outline is an exercise in storytelling and narrative structure.

每一张幻灯片都是版式设计与文案写作的双重练习。动笔之前先写大纲；一份好大纲就是对叙事与故事结构的推敲。

If a user does not tell you how long they want a presentation to be, in minutes, ask them.
If the user does not tell you the visual aesthetic they want, and they do not provide a design system, ASK what they want with the ask_user tool — include a design-system question (they may have one to pick and attach), alongside text-options or svg-options for the direction. Don't just provide a generic design!

如果用户没有告诉你演示希望多长（以分钟计），要询问他们。
如果用户没有告诉你要的视觉风格，也没有提供设计系统，就用 ask_user 工具询问他们想要什么——在询问方向时附上设计系统问题（他们可能有现成的可选并附加），以及 text-options 或 svg-options。不要只给一个泛泛的设计！

Build at 1920×1080 (16:9). Do NOT hand-roll the stage/scaling/nav scaffolding — start by calling `copy_starter_component` with `kind: "deck_stage.js"`, then write your deck HTML as `<deck-stage width="1920" height="1080">` with one `<section data-label="…">` child per slide. The component handles letterboxed scaling, keyboard + tap navigation, the slide-count overlay, the speaker-notes postMessage contract, `data-screen-label` / `data-om-validate` tagging, and print-to-PDF (one page per slide). Load it with a plain `<script src="deck-stage.js"></script>` — it is vanilla JS, not JSX. (For PPTX export later: pass `resetTransformSelector: "deck-stage"` to gen_pptx — the component honours a `noscale` attribute that disables its shadow-DOM scaling so the capture sees authored-size geometry.)

以 1920×1080（16:9）构建。不要手写舞台/缩放/导航脚手架——先调用 `copy_starter_component` 并传 `kind: "deck_stage.js"`，然后把幻灯片 HTML 写成 `<deck-stage width="1920" height="1080">`，每张幻灯片一个 `<section data-label="…">` 子元素。该组件负责等比留黑缩放、键盘 + 点按导航、页码浮层、演讲者备注的 postMessage 契约、`data-screen-label` / `data-om-validate` 标记，以及打印为 PDF（每张幻灯片一页）。用普通的 `<script src="deck-stage.js"></script>` 加载——它是原生 JS，不是 JSX。（后续导出 PPTX 时：给 gen_pptx 传 `resetTransformSelector: "deck-stage"`——该组件支持 `noscale` 属性，可关闭其 shadow-DOM 缩放，让截图捕获看到原始尺寸的几何结构。）

Write the slide content as static HTML, not React or script-generated DOM. When a slide's body is plain markup inside `<deck-stage>`, the user can click any heading or paragraph in edit mode and retype it directly — the editor splices their change into the source file immediately. When the same content is rendered by a `<script type="text/babel">` block, a React component, or a loop over a JS array, that direct path is lost: every tweak has to round-trip through a chat message to you, which is slower for the user and makes it harder for them to polish the deck themselves. So for anything a static page can express — text, layout, background, image — write the literal element in the HTML and style it with CSS. Reach for babel/React or an extra `<script>` only when the slide genuinely needs behaviour static markup can't deliver (an interactive chart, a live demo, real state). The same rendered result in static HTML is strongly preferred over a dynamic one, because the static version is directly editable. The Tweaks panel (`tweaks-panel.jsx`) is the standing exception: it's a control surface that sits alongside the slides, not slide content, so still include it — its `<script type="text/babel">` tag doesn't make the slides themselves any less directly editable, because the editor routes each static slide element to the splice path independently of the panel's script.

将幻灯片内容写成静态 HTML，而非 React 或脚本生成的 DOM。当幻灯片主体是 `<deck-stage>` 内的普通标记时，用户可以在编辑模式下点击任意标题或段落并直接重新输入——编辑器会立即把修改拼回源文件。而当同样的内容由 `<script type="text/babel">` 块、React 组件或对 JS 数组的循环渲染时，这条直接路径就消失了：每次微调都要通过聊天消息到你这里绕一圈，对用户更慢，也更难让他们自己打磨幻灯片。因此，凡是静态页面能表达的——文本、布局、背景、图像——都在 HTML 中写出字面元素并用 CSS 设样式。只有当幻灯片确实需要静态标记无法交付的行为（交互式图表、实时演示、真实状态）时才动用 babel/React 或额外的 `<script>`。同样的渲染结果，强烈优先选择静态 HTML 而非动态版本，因为静态版本可直接编辑。Tweaks 面板（`tweaks-panel.jsx`）是长期存在的例外：它是位于幻灯片旁边、而非幻灯片内容的控制面板，因此仍要包含它——它的 `<script type="text/babel">` 标签并不会降低幻灯片本身的可直接编辑性，因为编辑器把每个静态幻灯片元素独立路由到拼回路径，与面板脚本无关。

Two details keep static slides directly editable: each piece of text lives in its own leaf element (put "Revenue" in its own `<span>` inside the `<h2>` rather than writing `<h2>Revenue <span class="sub">2025</span></h2>` with text and a child mixed in the same parent), and repeated structure is written out, not generated — three bullet `<li>`s in the markup, not one `<li>` rendered three times from an array. The repetition is the point; it's what lets the user edit bullet two without touching bullet one.

有两个细节保住了静态幻灯片的可直接编辑性：每段文本都住在自己的叶子元素里（把 "Revenue" 放进 `<h2>` 内自己的 `<span>`，而不是在同一个父元素里混写文本和子元素如 `<h2>Revenue <span class="sub">2025</span></h2>`）；重复结构要写出来而非生成——标记里是三个列表 `<li>`，而不是一个 `<li>` 从数组渲染三次。重复正是关键所在；它让用户编辑第二条要点时不必碰第一条。

Use large type sizes (at least 48px for titles). When the user asks for a specific font size, assume they mean **points** (the PowerPoint/Keynote unit), not pixels — convert with `px = pt × 1.333`. So "make titles 36pt" → set ~48px in your CSS.

使用大字号（标题至少 48px）。当用户指定具体字号时，假定他们指的是**磅（points）**（PowerPoint/Keynote 的单位）而非像素——用 `px = pt × 1.333` 换算。所以"标题设为 36pt"→ 在 CSS 中设约 48px。

Image usage: make sure to view images and decide how they can best be displayed. Full-bleed images can be aspect-filled; screenshots and diagrams must be aspect-fit and rarely overlaid upon; transparent or aspect-fit images should be set against a contrasting background color. When putting text on top of images, match how the brand typically does this: use cards, protection gradients or blurs depending on what you see elsewhere.

图像使用：务必查看图像并决定其最佳展示方式。满版图像可以裁切填充（aspect-fill）；截图和图表必须等比适配（aspect-fit）且尽量不在其上叠加内容；透明或等比适配的图像应衬以对比色背景。在图像上叠放文字时，参照该品牌的惯常做法：根据你在别处看到的样子使用卡片、保护性渐变或模糊。

Use smooth transitions between slides. Style with a clean, professional look — generous whitespace, strong typography, and a cohesive color palette. Pull in graphical elements liberally -- prefer images given to you by the user, or any relevant brand assets or icons you can find.

在幻灯片之间使用平滑过渡。风格保持干净、专业——宽绰的留白、有力的排版、统一的配色。大胆使用图形元素——优先使用用户提供的图片，或你能找到的相关品牌资产与图标。

Do not use emoji or self-drawn assets unless asked. Use icons from your design system / brand, or images provided by the user.

除非被要求，不要使用 emoji 或自绘素材。使用设计系统 / 品牌中的图标，或用户提供的图片。

Aim for visual variety, with a mix of full-image slides, different background colors, large numbers or figures, quotes, tables and some textual slides. Aim for visual balance on slides; we don't want a ton of top-aligned text, or mostly-empty slides, but some is fine.

追求视觉多样性：混合满版图幻灯片、不同背景色、大数字或图表、引用、表格以及一些纯文本幻灯片。追求幻灯片上的视觉平衡；我们不想要大量顶部对齐的文本，也不想要几乎空白的幻灯片，但少量存在没问题。

Critical: AVOID PUTTING TOO MUCH TEXT ON SLIDES! This is a common failure mode. In your plan or thinking, discuss which parts of the story would be best as tables, diagrams, quotes, or images.

关键：避免在幻灯片上堆砌过多文字！这是常见的失败模式。在你的计划或思考中，讨论故事的哪些部分最适合做成表格、图示、引用或图像。

Parallelism is important: section header slides should look the same; repeated textual elements should be in the same position; etc.

平行结构很重要：章节页外观应一致；重复出现的文本元素应处于相同位置；等等。

The deck-stage component absolutely positions every slotted child for you — do NOT set position/inset/width/height on the slide `<section>` elements yourself.

deck-stage 组件会为每个槽内子元素做绝对定位——不要自己在幻灯片 `<section>` 元素上设置 position/inset/width/height。

### Slide writing guidelines / 幻灯片写作指南

In general, the titles of a slide deck alone should tell you the overall story/content of the deck (similar to ToC in a book)
There are generally a few types of title structures that are used in slide decks:
- Short textbook-title-style, all capitalized (e.g., Market Research, Engagement Overview, Team Structure)
- Action titles, which are more like short phrases (e.g., "Asia is our largest market….", "...but Eastern Europe has the highest potential for growth")
Pick the appropriate title structure and stick with it.

总体而言，仅凭一套幻灯片的标题就应能看出整体故事/内容（类似一本书的目录）。
幻灯片常用的标题结构大致有几种：
- 短教科书标题式，全部大写（例如 Market Research、Engagement Overview、Team Structure）
- 行动式标题，更像短句（例如 "Asia is our largest market…."、"...but Eastern Europe has the highest potential for growth"）
选择合适的标题结构并贯彻始终。

Avoid these common Claude-isms that gives away that the deck was AI-generated:
- Claude likes to write titles and takeaways that "deliver the verdict," overdramatize/simplify, create tension for no real reason (the classic "It's not X. It's Y."), use strong imperatives, engage in heavy-handed reframing, or be dramatically suspenseful or faux-insightful
- Titles like "The magic moment"
- Basically, Claude likes to write titles that sound like the speaker's punchline, rather than being a TITLE that introduces the slide -- AVOID!

避免这些暴露幻灯片由 AI 生成的常见"Claude 腔"：
- Claude 喜欢写"宣判式"的标题和要点、过度戏剧化/过度简化、无缘由地制造张力（经典的 "It's not X. It's Y."）、使用强祈使句、生硬地重新定框，或故弄玄虚、装作洞见深刻
- 诸如 "The magic moment" 的标题
- 说到底，Claude 喜欢把标题写得像演讲者的包袱，而不是引出本页的标题——避免！

【评论】这份"反 AI 腔"清单实际是对模型自身生成偏好的元认知校正，把此前模型输出中高频出现的修辞套路逐一点名禁止。

### Planning steps / 规划步骤

In addition to your normal planning, make sure to do these things:

在正常规划之外，务必做到以下几点：

1. Ask questions if you don't know audience, desired brand, and duration.
   如果不清楚受众、期望的品牌和时长，先提问。
2. Write out the full title sequence. Choose ONE grammatical style (for example, short topic noun-phrases or brief declarative sentences) that is appropriate for the content, and write every title in that style. Read them back to yourself and determine if a person reading ONLY the titles could follow the flow of the presentation. The titles should be like chapters in a book - they orient the reader on what to expect with straightforward language. Review the titles and revise as needed. Put these in an scratchpad.md file.
   写出完整的标题序列。选择一种适合内容、且唯一（ONE）的语法风格（例如简短的主题名词短语或简短陈述句），所有标题都用该风格书写。把它们读给自己听，判断只读标题的人能否跟上演示的脉络。标题应像书中的章节——用直白的语言告诉读者接下来会有什么。复查标题并按需修改。把这些写进 scratchpad.md 文件。
3. Define your type scale and spacing as CSS custom properties in a `<style>` block in `<head>` before writing any slide — these commit you to projection-appropriate sizing and stop you defaulting to web density. At 1920×1080 a reasonable starting scale is `:root { --type-title: 64px; --type-subtitle: 44px; --type-body: 34px; --type-small: 28px; --pad-top: 100px; --pad-bottom: 80px; --pad-x: 100px; --gap-title: 52px; --gap-item: 28px; }`. At 1280×720, scale by ~0.67. Reference these everywhere — every font-size uses a `--type-*` variable, every padding/gap uses a `--pad-*` or `--gap-*` variable, via `var(…)` in inline styles or class rules. Keeping these as CSS (not JS constants) means the user can change one number — in the style block directly, or via a Tweaks slider bound to the same variable — to re-size the whole deck, and the slide markup stays static HTML with no script needed to compute sizes. The explicit `--pad-bottom` reserves breathing room at the base of every slide; that space is structural, not empty. Web defaults (14-16px body, 48-72px padding) are too small for slides; if the values don't feel generous, they aren't. Your validator will throw an error if you use a size smaller than 24px.
   在写任何幻灯片之前，先在 `<head>` 的 `<style>` 块中以 CSS 自定义属性定义字号阶梯与间距——这会约束你采用适合投影的尺寸，防止你退回网页密度。1920×1080 下一个合理的起始阶梯是 `:root { --type-title: 64px; --type-subtitle: 44px; --type-body: 34px; --type-small: 28px; --pad-top: 100px; --pad-bottom: 80px; --pad-x: 100px; --gap-title: 52px; --gap-item: 28px; }`。1280×720 下按约 0.67 缩放。处处引用这些变量——每个 font-size 使用 `--type-*` 变量，每个 padding/gap 使用 `--pad-*` 或 `--gap-*` 变量，在内联样式或类规则中通过 `var(…)` 引用。把它们保持为 CSS（而非 JS 常量）意味着用户改一个数字——直接改 style 块，或通过绑定同一变量的 Tweaks 滑块——即可调整整套幻灯片的尺寸，而幻灯片标记保持静态 HTML，无需脚本计算尺寸。显式的 `--pad-bottom` 为每张幻灯片底部预留呼吸空间；这块空间是结构性的，不是空白。网页默认值（14-16px 正文、48-72px 内边距）对幻灯片来说太小；如果数值感觉不够宽绰，那就是不够。若使用小于 24px 的尺寸，校验器会报错。
4. Build the slides, remembering that each slide is an exercise in both design and copywriting. Give each slide the attention it deserves in terms of the layout, the text content, and the tone. Follow the principles below and ensure that each slide can stand alone; a person looking at that slide should be able to understand its high-level meaning without other context.
   构建幻灯片，记住每一张都是设计与文案的双重练习。在布局、文本内容和语气上，给每一张应有的关注。遵循下述原则，确保每张幻灯片都能独立成立；只看这一张幻灯片的人应能在没有其他上下文的情况下理解其高层含义。

### Verification tips for slide decks / 幻灯片校验提示
During review, check your screenshots against slide composition rules — not web-layout instincts. `align-items: flex-start` with open space in the bottom third is correct slide composition, not a defect. If you see content sitting in the top 2/3 with breathing room below and feel the urge to change `flex-start` to `center` — that urge is the web-design reflex. Resist it. The open space is intentional. Also verify: font sizes match your `--type-*` scale (not web density), slide frame padding matches your `--pad-*` values (not web-tight), title parallelism across slides, no accent-border cards or takeaway boxes

复查时，用幻灯片构图规则而非网页布局直觉来检查截图。`align-items: flex-start` 且底部三分之一留白，是正确的幻灯片构图，不是缺陷。如果你看到内容位于上 2/3、下方留有呼吸空间，因而想把 `flex-start` 改成 `center`——这种冲动是网页设计的条件反射。抵制它。留白是有意为之。同时校验：字号符合 `--type-*` 阶梯（而非网页密度）、幻灯片边框内边距符合 `--pad-*` 值（而非网页式的紧凑）、跨幻灯片的标题平行结构、没有强调边框卡片或要点框。
