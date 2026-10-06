<!-- BILINGUAL-EN-ZH -->
---
name: artifact-design
description: Design guidance and fundamentals for Artifacts.
when_to_use: Load before writing any artifact, including a skill-instructed Markdown one - Markdown is never a shortcut past the design pass.
user-invocable: false
---

Work the way the design lead at a small, versatile studio would: give each client a visual identity at the level of treatment the task calls for. Make deliberate choices about palette, typography, and layout that are specific to this subject, and avoid templated designs.

像一个小型多面手工作室的设计负责人那样工作：按任务所需的处理力度，为每位客户赋予视觉识别。在调色板、字体排印和布局上做出针对该主题的审慎选择，避免模板化的设计。

## Read the request first / 先读懂请求

Decide the treatment; designing is a given. A doc gets the same craft as a landing page; only the treatment differs. Format is a separate matter: author HTML, and publish Markdown only when a loaded skill explicitly instructs it. A Markdown publish keeps its filename as its title, uses almost none of the craft below, and is never a way to save time.

先决定处理方式；设计是必然要做的事。一份文档与一个落地页享有同样的工艺；不同的只是处理方式。格式是另一回事：以 HTML 创作，只有当已加载的 skill 明确指示时才发布 Markdown。以 Markdown 发布会把文件名当作标题，几乎用不到下文的任何工艺，也绝不是节省时间的手段。

Many requests call for a more utilitarian treatment: a plan, a memo, a demo. Make it polished, with real typographic hierarchy, considered spacing, and a proper palette, but avoid over-designing. Most pages don't need a flashy, gigantic hero. Keep flourishes tasteful and limited.

许多请求需要更实用的处理方式：一份计划、一份备忘录、一个演示。要做到精致：有真正的字体层级、经过考量的间距、得体的调色板，但要避免过度设计。大多数页面不需要炫目而巨大的 hero 区。装饰要保持克制而有品位。

Some requests call for an editorial treatment: a landing page, a game, an app or tool they'll keep or share.

另一些请求需要编辑型的处理方式：落地页、游戏，或用户会保留、分享的应用或工具。

If unsure: a well-composed page is always acceptable; an over-designed visual identity sometimes isn't.

拿不准时：构图良好的页面总是可以接受的；过度设计的视觉识别有时则不然。

Fundamentals below apply to everything. Follow the editorial process after them only when that reading calls for it.

下面的基础规范适用于一切。只有当对请求的解读有此需要时，才在其之后遵循编辑流程。

## Fundamentals for every artifact / 每个作品的基础规范

**Respect what already exists.** Look for an existing design system first: CLAUDE.md, a tokens or theme file, existing component styles. When one exists, apply it; everything below fills gaps and never overrides. Precedence is always the user's own words, then the project's existing system, then your choices.

**尊重已有之物。**先寻找现有的设计系统：CLAUDE.md、令牌或主题文件、已有的组件样式。若存在，就应用它；下文的一切只用于填补空白，绝不覆盖。优先级始终是：用户自己的话，然后是项目现有系统，然后才是你的选择。

**Ground it in the subject.** If the subject isn't already clear, define it: one concrete subject, its audience, and the page's single job. Distinctive choices come from the subject's own world: its materials, instruments, and vernacular. Whatever the treatment, include at least one detail only this subject would have (its real units and scales, its document conventions, its terms of art) as content rather than ornament; it costs nothing even on a plain page. Use real content throughout and never lorem ipsum.

**扎根于主题。**如果主题尚不明晰，就先定义它：一个具体的主题、它的受众，以及页面的唯一职责。有辨识度的选择来自主题自身的世界：它的材料、工具与行话。无论采用何种处理方式，都要包含至少一个只有这个主题才会有的细节（它真实的单位与量级、它的文档惯例、它的术语），作为内容而非装饰；即使在素净的页面上这也毫无成本。通篇使用真实内容，绝不用 lorem ipsum。

**Pair typefaces.** Typography determines how the page reads even when the page isn't about typography. Google Fonts is the only font host the Artifact CSP allows; link it directly (`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...&display=swap">`). A face from anywhere else must be inlined as a @font-face data URI, or the browser silently uses a fallback. In both cases, declare a real fallback stack. Keep running text near 65 characters wide. Set a type scale and keep to it. Give headings `text-wrap: balance`, give body text comfortable spacing, and give uppercase labels a little letter-spacing.

**搭配字体。**即使页面与字体排印无关，字体排印也决定页面读起来的感受。Google Fonts 是 Artifact CSP 允许的唯一字体宿主；直接链接它（`<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=...&display=swap">`）。来自其他任何地方的字体必须内联为 @font-face data URI，否则浏览器会静默使用回退字体。两种情况下都要声明真实的回退字体栈。正文宽度保持在 65 字符左右。设定一个字阶并严格遵守。给标题加 `text-wrap: balance`，给正文舒适的间距，给大写标签加一点字距。

**Load libraries instead of inlining them.** When the page really needs a library (React, a charting or highlighting package), load its UMD build from cdnjs with one pinned `<script src="https://cdnjs.cloudflare.com/ajax/libs/...">` placed before the inline script that uses its global; don't inline the library's source or hand-write a substitute. Only the script loads this way; a library's stylesheet still has to be inlined, and the Artifact tool's description lists the few other script hosts the CSP admits. The page's own CSS and JS, its images, and its data ship with the page. Most pages don't need any library; use one only when it does substantial work for the page.

**加载库而不是内联它们。**当页面确实需要某个库（React、图表或代码高亮包）时，从 cdnjs 以一个固定版本的 `<script src="https://cdnjs.cloudflare.com/ajax/libs/...">` 加载其 UMD 构建，并放在使用其全局变量的内联脚本之前；不要内联库的源码，也不要手写替代品。只有脚本可以这样加载；库的样式表仍须内联，Artifact 工具的描述列出了 CSP 允许的其他少数脚本宿主。页面自身的 CSS 和 JS、图片和数据随页面一同交付。大多数页面不需要任何库；只有当它能为页面做实质性工作时才使用。

**Choose neutrals deliberately.** A pure mid-grey looks unconsidered; a grey with a slight hue bias toward the page's accent looks intentional. Pure white and near-black are fine backgrounds when they suit the subject, as long as you chose the neutral deliberately instead of inheriting a default.

**有意地选择中性色。**纯正的中灰显得欠考虑；带一点向页面强调色偏移的灰则显得有意图。纯白与近黑在适合主题时是很好的背景，前提是你有意选择了这个中性色，而不是继承了默认值。

**Design both themes.** The page renders in the viewer's theme, and the viewer has three states: an explicit choice sets `data-theme="dark"` or `data-theme="light"` on the root element, and the default "system" setting sets *nothing*. Most viewers see the document with no `data-theme` attribute, where only `prefers-color-scheme` distinguishes light from dark. Structure the CSS at the token level for all three. The bare `:root` block defines the complete light palette (for a deliberately dark-first design, swap light and dark consistently through this whole pattern); `@media (prefers-color-scheme: dark)` redefines only the tokens, guarded as `:root:not([data-theme="light"])` so an explicit light choice overrides a dark OS setting; `:root[data-theme="dark"]` redefines them again so the toggle also overrides in the other direction; wherever the dark palette applies - both dark blocks, or bare `:root` in a dark-first or single-dark design - also set `color-scheme: dark` (the skeleton pins `light` on `:root`), so native form controls and scrollbars follow the palette. Style components through the tokens, never directly inside a media or `[data-theme]` block: a color defined only inside `[data-theme]` never applies when no `data-theme` attribute is set, and the page then shows one theme's text on the other theme's background. Two more rules keep each theme consistent. First, the artifact is composited over a background the viewer paints in *its own* theme, so `body` must set an explicit `background` from a token; a transparent body shows the host's background with no warning. Second, every element that sets a color takes it from the same token set as the surface behind it, never from a literal that works in only one theme. Declare every token in the bare `:root` block before any media or `[data-theme]` block redefines it; a color that exists only inside one of those blocks is the classic unreadable-artifact bug. Give the second theme the same care as the first: don't simply invert it; keep contrast legible and keep the accent working on both backgrounds. A design that deliberately commits to one visual world (a neon arcade screen, a letterpress invitation) may stay single-theme: then omit the media query and `[data-theme]` blocks entirely but still set the background and every color explicitly, so the page looks right on either host background; do this by choice, never by omission.

**为两种主题都做设计。**页面按查看者的主题渲染，而查看者有三种状态：显式选择会在根元素上设置 `data-theme="dark"` 或 `data-theme="light"`，默认的 "system" 设置则*什么都不设*。大多数查看者看到的文档没有 `data-theme` 属性，此时只有 `prefers-color-scheme` 区分明暗。针对全部三种状态，在令牌层面组织 CSS。裸 `:root` 块定义完整的浅色调色板（若是有意的暗色优先设计，则在整个模式中一致地对调明暗）；`@media (prefers-color-scheme: dark)` 只重定义令牌，并以 `:root:not([data-theme="light"])` 加以防护，使显式的浅色选择能覆盖深色的操作系统设置；`:root[data-theme="dark"]` 再次重定义它们，使切换开关在另一个方向同样能覆盖；凡深色调色板生效之处——两个深色块，或暗色优先/单深色设计中的裸 `:root`——还要设置 `color-scheme: dark`（骨架在 `:root` 上固定了 `light`），让原生表单控件和滚动条跟随调色板。组件一律通过令牌设置样式，绝不在媒体查询或 `[data-theme]` 块内直接写：只定义在 `[data-theme]` 内的颜色在没有设置 `data-theme` 属性时永远不会生效，页面就会出现一种主题的文字配另一种主题的背景。还有两条规则保证每个主题的一致性。第一，作品合成在查看者以*其自身*主题绘制的背景之上，因此 `body` 必须用令牌设置显式的 `background`；透明的 body 会毫无提示地露出宿主的背景。第二，每个设置颜色的元素都从与它背后表面相同的令牌集取值，绝不用只在单一主题下可行的字面量。每个令牌都要先在裸 `:root` 块中声明，再由媒体查询或 `[data-theme]` 块重定义；只存在于其中某个块内的颜色正是经典的"作品不可读"缺陷。给第二种主题与第一种同等的用心：不要简单反相；保持对比度可读，让强调色在两种背景上都成立。有意承诺单一视觉世界的设计（霓虹街机屏、凸版印刷请柬）可以保持单一主题：此时完全省略媒体查询和 `[data-theme]` 块，但仍要显式设置背景与每个颜色，使页面在任一宿主背景上都好看；这样做必须是主动选择，绝不能是疏漏。

【评论】这一段是典型的"三态主题"工程模式（显式亮、显式暗、系统跟随），规则实质是把所有颜色收敛到 CSS 令牌并在 `:root` 兜底声明，以避免宿主主题与页面主题错配这一类高频缺陷。

**Use layout for spacing.** Lay out sibling groups with flex or grid and `gap` instead of per-element margins, which collapse or double without warning. Keep a side gutter of at least 16px at every width: set it once as side padding on `body` or one outer wrapper, whose vertical padding uses `padding-block` and never a `padding` shorthand that zeroes the sides. Let rows wrap or stack to one column at phone width (about 400px). Give images and any `aspect-ratio` box `max-width: 100%`, and don't give anything a `min-width` wider than the screen. Only wide tables, code, and diagrams may exceed it; give each `overflow-x: auto` on its own container so the page body never scrolls sideways. The publish skeleton pads `:root` top and bottom by the phone's safe-area insets (zero everywhere except a phone app) so the page runs edge to edge while its content stays clear of the system bars; keep that padding. A bar fixed to the top or bottom stays at `0` and adds `env(safe-area-inset-top, 0px)` or `env(safe-area-inset-bottom, 0px)` to its own padding. A sticky page header uses `top: env(safe-area-inset-top, 0px)` and never `0`. Size a one-screen app with `height: 100%` on `html` and `body` instead of `100vh`, so it fits inside that padding. A page that includes its own viewport meta gets this padding only when that meta declares `viewport-fit=cover`. Use `font-variant-numeric: tabular-nums` wherever digits line up in columns.

**用布局实现间距。**用 flex 或 grid 加 `gap` 来排布同级分组，而不是逐元素设 margin——后者会毫无预警地塌缩或叠加。任何宽度下都保持至少 16px 的侧边留白：在 `body` 或一个外层包装上一次性设置侧向内边距，其纵向内边距用 `padding-block`，绝不用会把两侧清零的 `padding` 简写。在手机宽度（约 400px）让行换行或堆叠为单列。给图片和任何 `aspect-ratio` 盒子设 `max-width: 100%`，不给任何东西设超过屏幕宽度的 `min-width`。只有宽表格、代码和图示可以超宽；给每个超宽者在其自身容器上设 `overflow-x: auto`，使页面主体永不横向滚动。发布骨架会按手机安全区嵌入（safe-area insets，手机 app 之外处处为零）给 `:root` 上下加内边距，使页面通栏铺满而内容避开系统栏；保留这个内边距。固定在顶部或底部的栏保持 `0`，并在自身 padding 上加 `env(safe-area-inset-top, 0px)` 或 `env(safe-area-inset-bottom, 0px)`。吸顶页头用 `top: env(safe-area-inset-top, 0px)` 而绝不用 `0`。单屏应用用 `html` 和 `body` 上的 `height: 100%` 定尺寸而不用 `100vh`，以便装进该内边距。自带 viewport meta 的页面只有在该 meta 声明 `viewport-fit=cover` 时才获得这个内边距。凡数字按列对齐之处使用 `font-variant-numeric: tabular-nums`。

**Make repeated elements consistent.** For cards in a row, label/value pairs down a list, or badges on sibling items, use the same edges, baselines, and inner padding on each, and put any recurring element in the same place on each. Let content set a container's height and pick a column count the items fill, so nothing stretches over empty space or sits alone in a row. Make text that can outgrow its track wrap or scroll in its own container; clipped text is a bug.

**让重复元素保持一致。**对一排卡片、一列标签/值对，或同级条目上的徽章，每项都使用相同的边缘、基线和内边距，且把任何重复出现的元素放在每项的同一位置。让内容决定容器高度，并选择条目能填满的列数，使任何东西都不在空白上拉伸，也不落单成行。让可能溢出其轨道的文字在自己的容器内换行或滚动；被裁剪的文字是缺陷。

**Use card styling selectively.** Border, fill, radius, and shadow each mark an element as a separate object. Apply them by role, to set off the one element that needs it; applying the same radius and shadow to every block flattens the hierarchy. Open with big-number tiles only when those figures are the point of the page.

**有选择地使用卡片样式。**边框、填充、圆角和阴影各自都会把一个元素标记为独立对象。按角色应用它们，以衬托唯一需要它的元素；把同样的圆角和阴影套在所有块上会压平层级。只有当大数字本身就是页面重点时，才以大数字磁贴开场。

**Draw charts to scale.** Place marks, ticks, and labels with one scale, and make every label name a value the chart actually reaches. Color chart text from the theme tokens so it is readable in both themes. Keep marks, labels, and edges clear of one another and inside the drawing's bounds; in SVG, leave room in the viewBox for the outermost labels and give every drawn shape an explicit fill.

**按比例绘制图表。**用同一比例放置标记、刻度和标签，并让每个标签所指的都是图表实际到达的值。图表文字从主题令牌取色，使其在两种主题下都可读。让标记、标签和边界互不干扰且处于绘图范围之内；在 SVG 中，为最外侧标签在 viewBox 里留出空间，并给每个绘制的形状显式填充。

**Make the page complete at rest.** Everything meant to be read is visible once the page has loaded, with no scrolling to trigger it; that first still frame is what a thumbnail, a shared link, and a skimming reader all see. A section may animate in, but from a visible resting state, never left at `opacity: 0` waiting for an observer. Size a hero to its content instead of to the viewport; a `100vh` opener pushes the rest of the page out of that first frame. A tool or app opens in a realistic working state: the user's real data where it exists, otherwise example rows, a loaded sample, or a plausibly filled form, clearly marked as examples and never presented as the user's own figures. The first view shows what the tool does; an empty shell waiting for input shows nothing.

**让页面在静止时就完整。**所有旨在被阅读的内容在页面加载后即应可见，无需滚动触发；第一幅静止画面就是缩略图、分享链接和略读者看到的全部。某个区块可以有入场动画，但必须从可见的静止状态出发，绝不停留在 `opacity: 0` 等待观察器。hero 尺寸随其内容而非视口而定；`100vh` 的开场会把页面其余部分挤出第一帧。工具或应用应以逼真的工作状态打开：有用户真实数据就用真实数据，否则用示例行、已加载的样例或填得合乎情理的表单，并明确标注为示例，绝不冒充用户自己的数字。首屏展示工具能做什么；空壳等待输入则什么也展示不了。

**Avoid AI-generated design.** AI-generated design currently clusters around a few looks: warm cream (#F4F1EA) with a serif display and terracotta accent; near-black with a lone acid-green or vermilion pop; broadsheet hairline rules with dense columns; a purple-to-blue gradient hero on white; Inter or Space Grotesk as the "safe" face; emoji as section markers; everything centered; `rounded-lg` everywhere; accent bar/rail on rounded cards. When the user specifies a visual direction, follow it exactly; their words always win, including when they ask for one of these looks. When nothing is specified, don't use that freedom on one of these defaults.

**避免 AI 生成式设计。**当前的 AI 生成设计聚集在几种面孔上：暖奶油色（#F4F1EA）配衬线展示字体和陶土色强调；近黑底配孤零零的酸性绿或朱红点睛；报纸式发丝线配密排分栏；白底上紫到蓝渐变的 hero；把 Inter 或 Space Grotesk 当"安全"字体；emoji 作区块标记；一切居中；满屏 `rounded-lg`；圆角卡片上的强调色条/边轨。当用户指定视觉方向时，严格遵循；用户的话总是赢，即使他们要的正是其中某种面孔。当没有任何指定时，不要把这份自由浪费在这些默认套路之一上。

【评论】这是一份"负面风格清单"，用于规避当前 AI 生成内容的高频视觉套路；同时明确用户显式要求例外，属于典型的"默认约束 + 用户意愿优先"双层结构。

**Build cleanly.** Watch for overlapping elements, cascade collisions, and silent font fallbacks. Close every non-void element, double-quote attributes, give keyboard focus a visible state, and respect `prefers-reduced-motion`. Give every form control a stable `id` (the platform preserves form values, focus, and scroll position across a republish). For generative or decorative graphics, use Canvas or WebGL instead of hand-writing long SVG path data.

**干净地构建。**留意元素重叠、层叠冲突和静默的字体回退。闭合每个非 void 元素，属性用双引号，给键盘焦点可见状态，尊重 `prefers-reduced-motion`。给每个表单控件稳定的 `id`（平台会在重新发布时保留表单值、焦点和滚动位置）。对生成式或装饰性图形，用 Canvas 或 WebGL，而不是手写冗长的 SVG path 数据。

**CSS rules.** When writing the CSS, watch your selector specificity. It is easy to generate classes that cancel each other out, e.g. a type-based selector like `.section` and an element-based one like `.cta` both setting padding and margins between sections. Structure the cascade so it doesn't undo your spacing unnoticed.

**CSS 规则。**编写 CSS 时留意选择器优先级。很容易生成互相抵消的类，例如 `.section` 这类类型化选择器和 `.cta` 这类元素化选择器同时设置区块之间的 padding 和 margin。把层叠组织好，别让它悄无声息地抵消你的间距。

**Writing the copy.** Treat words as design material and never as decoration. Write from the user's side of the screen: name things by what people recognize instead of how the system is built (a person manages *notifications*; they don't manage *webhook config*). Use active voice; a control states exactly what happens ("Publish", then a toast that says "Published"). Errors explain what went wrong and how to fix it, without apologies or vagueness. Prefer specific to clever. Write plainly, the way a knowledgeable person would talk. Avoid mannered devices: asides set off by em-dashes, "not X, but Y" framing, colon-then-reveal sentences, scare quotes around invented labels, and stock phrases such as "worth noting" or "honest caveat". Prefer short, direct sentences over compressed or clever phrasing.

**撰写文案。**把文字当作设计材料，而不是装饰。从用户一侧的屏幕来写：按人们认知的方式命名事物，而不是按系统构建的方式（一个人管理的是*通知*，而不是 *webhook 配置*）。用主动语态；控件应准确说明会发生什么（"发布"，随后一条提示"已发布"）。错误信息要说明出了什么问题、如何修复，不道歉也不含糊。宁可具体，不要耍巧。写得平实，像一个懂行的人说话那样。避免做作的修辞：破折号引出的插语、"不是 X，而是 Y" 的句式、冒号后揭晓式的句子、给生造标签加引号，以及"值得注意的是"、"坦诚地说"这类套话。宁可简短直白的句子，不要压缩或耍巧的措辞。

【评论】此处列举的"避免句式"（破折号插语、"不是 X 而是 Y"、套话）与许多反 AI 腔调写作指南高度重合，说明该 skill 把文风也纳入了去 AI 化的范围。

**Name the page like a product; don't caption it.** The `<title>` is the artifact's name in the gallery and the browser tab, and it gives the reader a first impression of the care taken. Give the page a real name: a short noun phrase, typically two to four words, specific to the subject; or, for a page that exists to answer one question, that question itself, which then is the page's name. Stop at the name; a title that adds its own explanation after a dash or colon reads as generated filler. The name must also identify the page among many: in the gallery it appears beside dozens of other artifacts, and a generic category label that could apply to any of them fails as a name just as an appended explanation does. When a candidate title combines the name with a generic word (a greeting, a category, a page-type label), keep the name; a trim that drops the identifying part and keeps the generic word produces a title that could apply to any page. This rule removes explanations and doesn't require brevity: a multi-word title that already reads as one specific name is finished, and shortening it further only makes it generic. Put the explanation in the one-sentence publish `description`; the gallery shows it directly under the title.

**像给产品命名那样命名页面；不要给它配图说。**`<title>` 是作品在画廊和浏览器标签页中的名字，它给读者关于这份用心程度的第一印象。给页面起一个真正的名字：一个简短的名词短语，通常两到四个词，紧扣主题；或者，对一个只为回答某个问题而存在的页面，就用那个问题本身——它就是页面的名字。止步于名字；在破折号或冒号后追加自我解释的标题读起来像生成的填充物。名字还必须能把该页面从众多页面中识别出来：在画廊里它与几十个其他作品并列，一个适用于任何作品的通用类别标签作为名字会失败，正如追加解释一样。当候选标题把名字与通用词（问候语、类别、页面类型标签）组合在一起时，保留名字；把识别性的部分裁掉、只留通用词，得到的标题会适用于任何页面。这条规则移除的是解释，并不要求简短：一个多词标题若读起来已经是一个具体名字，就已完成，再缩短只会使它泛化。把解释放进发布时的一句话 `description`；画廊会把它直接显示在标题下方。

**Structure is information.** Structural devices (numbering, eyebrows, dividers, labels) should encode something true about the content instead of decorating it. Many generic designs use numbered markers (01 / 02 / 03), but those fit only if the content actually is a sequence, such as a real process or a typed timeline where the order is information the reader needs. Before adding numbered markers or similar devices, check that they actually make sense.

**结构即信息。**结构性手法（编号、眉题、分隔线、标签）应当编码关于内容的真实信息，而不是装饰它。许多平庸设计使用编号标记（01 / 02 / 03），但只有当内容确实是序列时才合适，例如真实流程或类型化时间线，其顺序是读者需要的信息。在添加编号标记或类似手法之前，确认它们确实说得通。

**When the page is a UI (dashboard, tool).** It is scanned and operated instead of read top to bottom, so the craft shifts from typography to information design. Put the summary before the detail. Encode state in form as well as in numbers (a pill, a chip, a severity stripe) so that whatever needs attention is visible at a glance. Semantic color (good / warning / critical) is separate from the accent hue and doesn't count as your accent. Give sparklines and charts the same care as type: an area fill, a faint grid, an emphasized endpoint. Interactive elements should look interactive.

**当页面是 UI（仪表盘、工具）时。**它被扫视和操作，而不是自上而下阅读，因此工艺从字体排印转向信息设计。摘要置于细节之前。状态既用数字也用形态编码（药丸、筹码、严重度条纹），使任何需要注意的东西一目了然。语义色（正常/警告/严重）与强调色分离，不计入你的强调色。给迷你图和图表与字体同等的用心：面积填充、浅淡网格、强调的端点。交互元素应当看起来可交互。

**When adding charts or diagrams** The craft shifts from identity to honesty — pick the form the data's shape calls for, keep encodings from exaggerating, title the finding rather than the axes. Load the `dataviz` skill for the specifics; this skill continues to govern the page the chart sits in.

**添加图表或图示时**工艺从识别转向诚实——选择数据的形状所要求的形式，防止编码夸大，标题写结论而不是坐标轴。加载 `dataviz` skill 获取细节；本 skill 继续管辖图表所在的页面。

## Process / 流程

Start from what the viewer should be able to do on the page, in addition to what they will read. If the page should take input, keep what people change for whoever opens it next, show live data, or ask Claude something, load the `artifact-capabilities` skill now and design around what it makes available to this user. A page that is only read doesn't need that.

除了读者将读到什么，还要从查看者应能在页面上做什么出发。如果页面应接受输入、把人们改动的内容留给下一个打开者、展示实时数据，或向 Claude 提问，现在就加载 `artifact-capabilities` skill，并围绕它向该用户提供的能力进行设计。纯阅读的页面不需要这些。

Before writing code, sketch a short design plan (a compact token system with color, type, and layout):

写代码之前，先拟一份简短的设计计划（一个紧凑的令牌系统，涵盖颜色、字体和布局）：

- **Color**: describe the palette as 4-6 named hex values.
  **颜色**：以 4-6 个命名的十六进制值描述调色板。
- **Type**: typefaces for 2+ roles: a characterful display face used with restraint, a complementary body face, and a utility face for captions or data if needed.
  **字体**：2 种以上角色的字体：一个有个性、克制使用的展示字体，一个互补的正文字体，以及需要时用于说明文字或数据的工具字体。
- **Layout**: a layout concept in one or two sentences.
  **布局**：一两句话的布局概念。

Then build, following the plan and deriving every color and type decision from it.

然后开始构建，遵循该计划，并让每个颜色与字体决策都由它推导而来。

**Write, look once, publish.** Before publishing you may look at the rendered page once: one screenshot of the local file, or the `ArtifactCheck` tool's preview (or the Artifact tool's own `action: "preview"` where there is no separate `ArtifactCheck` tool) where this session offers it. Then make one pass of edits for what it shows, without a second look. For a page that charts real numbers, do take that look instead of skipping it, and use it on the chart. Don't build a test loop around your own file: no repeated screenshots, no pulling the script out to run it through node, no scripts that probe the DOM. Such a loop spends the session re-checking what a careful write already got right, while the user waits for a link. Then publish. A page whose point is logic or stored data takes its one check here, not in a render loop: exercise once any `window.claude` call that the preview couldn't run (read the stored data back, for example), and stop. Review happens on the live page, and further polish is for the user to request. That check is for runtime code you wrote into this page, not for content you fill into an Artifact made from an Artifact type (a Slides deck, a Design canvas): there the type's own instructions say whether to check, and if they say nothing, don't. If the user reports something visibly broken (a clipped column, unreadable text, a control that does nothing), fix that and republish once.

**写完，看一次，发布。**发布前你可以查看渲染后的页面一次：本地文件的一张截图，或 `ArtifactCheck` 工具的预览（在没有单独 `ArtifactCheck` 工具的环境中用 Artifact 工具自身的 `action: "preview"`），前提是本会话提供该功能。然后针对它显示的问题做一轮修改，不再看第二次。对绘制真实数字的图表页面，务必要看这一次而不是跳过，并把目光用在图表上。不要围绕自己的文件搭建测试循环：不重复截图，不把脚本拽出来跑 node，不写探测 DOM 的脚本。这样的循环会把会话耗在反复检查一次用心书写本来就能做对的事情上，而用户还在等链接。然后发布。以逻辑或存储数据为卖点的页面在这里做它唯一的一次检查，而不是在渲染循环里：把预览无法运行的 `window.claude` 调用演练一次（例如把存储的数据读回来），然后停下。评审发生在上线后的页面上，进一步打磨由用户提出。这项检查针对的是你写进页面的运行时代码，而不是你填充进由 Artifact 类型生成的 Artifact（Slides 演示、Design 画布）的内容：在那里由该类型自身的说明决定是否检查，若它只字未提，就不检查。如果用户报告了明显损坏的东西（被裁剪的列、无法阅读的文字、点了没反应的控件），修好它并重新发布一次。

【评论】"只看一次"是对模型过度自检循环的显式约束，用单次预览加单轮修改来控制时延；这与许多智能体框架中"避免无界验证循环"的工程考量一致。

**Open viewers.** You don't need to do anything for viewers who already have the page open: published changes are delivered to them automatically at their next quiet moment, with state preserved where possible. If your page has state a viewer would miss (a game, a long form), register `window.claude?.hot?.snapshot(...)` and boot through `window.claude?.hot?.ready ? window.claude.hot.ready(start) : start(window.claude?.hot?.data ?? {})`.

**已打开页面的查看者。**对已经打开页面的查看者你无需做任何事：已发布的更改会在他们下一个空闲时刻自动送达，并尽可能保留状态。如果你的页面有查看者会错失的状态（游戏、长表单），注册 `window.claude?.hot?.snapshot(...)`，并通过 `window.claude?.hot?.ready ? window.claude.hot.ready(start) : start(window.claude?.hot?.data ?? {})` 启动。

## When the request is editorial / 当请求属于编辑型时

The stance changes here: the client has already rejected proposals that felt templated, and is paying for a distinctive point of view. Make opinionated decisions, and take one real aesthetic risk where it serves the work.

立场在此改变：客户已经否决了那些感觉模板化的方案，并为一个有辨识度的观点付费。做出有主见的决策，并在有助于作品之处冒一次真正的美学风险。

Review the design plan against the subject before building: if any part of it reads like the generic default you would produce for any similar page, revise that part, and note what you changed and why. Write the code only after confirming the plan is unique, following the revised plan exactly.

构建之前，对照主题审视设计计划：如果其中任何部分读起来像你会为任何类似页面产出的通用默认，就修改该部分，并记下你改了什么、为什么改。确认计划独一无二之后才写代码，并严格遵循修改后的计划。

**Principles** / **原则**

- The hero states the thesis: open with the most characteristic thing in the subject's world (headline, image, live demo, interactive moment). 
  hero 陈述论点：以主题世界中最具特征的事物开场（大标题、图片、现场演示、交互时刻）。
- Typography sets the personality of the page. Pair the display and body faces deliberately, avoiding the families you would use on any other project, and set a clear type scale with intentional weights, widths, and spacing. Make the type treatment itself a memorable part of the design instead of a neutral container for the content. 
  字体排印奠定页面的个性。有意图地搭配展示与正文字体，避开你在任何其他项目上都会用的字族，设定清晰的字阶，并配上有想法的字重、字宽与间距。让字体处理本身成为设计中令人难忘的部分，而不是内容的中性容器。
- Use motion deliberately. Think about whether and where animation can serve the subject: a page-load sequence, hover micro-interactions, ambient atmosphere. One orchestrated moment is usually more effective than scattered effects; choose what the direction calls for. However, sometimes less is more, and extra animation adds to the impression that the design is AI-generated. 
  有意图地使用动效。思考动画能否、在何处服务于主题：页面加载序列、悬停微交互、环境氛围。一个精心编排的时刻通常比散落的效果更有效；选择方向所要求的东西。不过，有时少即是多，额外的动画会加深"这设计是 AI 生成的"印象。
- Match complexity to the vision. Maximalist directions need elaborate execution; minimal directions need precision in spacing, type, and detail. Elegance is executing the chosen vision well.
  让复杂度与愿景匹配。极繁方向需要精细的执行；极简方向需要在间距、字体和细节上的精确。优雅就是把选定的愿景执行到位。
- Put your boldness in one place; keep everything around it quiet. If the accent clashes with the background, shift it toward an analogous hue or desaturate it instead of replacing it.
  把大胆集中在一处；让它周围的一切保持安静。如果强调色与背景冲突，把它移向邻近色相或降低饱和度，而不是更换它。
