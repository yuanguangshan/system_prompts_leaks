<!-- BILINGUAL-EN-ZH -->
# Design Spec — the Muse Moments Kit is the design system / 设计规格——Muse Moments Kit 即设计系统

Use `/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html`
for product components, fonts, and palette. Preserve the anatomy of recognizable
product controls. Adapt informational layouts to the narrated action using  
`/opt/hatch/skills/magic-moment/reference/visual-storytelling.md`.

产品组件、字体与调色板以 `/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html` 为准。可辨识的产品控件要保留其结构解剖。信息型版面则要配合旁白动作进行调整，方法见  
`/opt/hatch/skills/magic-moment/reference/visual-storytelling.md`。

Use one outer surface per card. Preserve meaningful inner structure: calendar
cells, chart marks, route lines, media viewports, and selected states. Flatten
only redundant decorative containers. Do not flatten a calendar into bullet
points or a chart into a list of numbers.

每张卡片只用一个外表面。保留有意义的内部结构：日历单元格、图表标记、路线线条、媒体视口以及选中态。只拍平冗余的装饰性容器。不要把日历拍平成要点列表，也不要把图表拍平成一串数字。

## The look is the Muse product's own / 外观即 Muse 产品本来的样子

The kit mirrors the shipping Muse clients: native and simple, mostly
typographic. Hierarchy comes from font weight and size plus the text
color ramp — primary `#111112`, secondary `rgba(0,4,9,.59)`, tertiary
`rgba(0,7,17,.37)` — never from decoration. The rules that keep a card
reading as the product:

该套件映射已上线的 Muse 客户端：原生、简洁、以排版为主。层级来自字重与字号，加上文字色阶——primary `#111112`、secondary `rgba(0,4,9,.59)`、tertiary `rgba(0,7,17,.37)`——绝不来自装饰。以下是让卡片读起来仍是产品本身规则：

- Flat surfaces only. Cards are a solid fill (`#FFFFFF`, or `#111112`
  for a deliberate dark card) with 22-24px corners at kit scale. NO
  borders, NO side strokes, NO box-shadows, NO elevation anywhere; the
  product's card components do not even have a shadow parameter. Rows
  separate with 1px hairlines at `rgba(0,0,0,.06-.10)`, never with
  outlined boxes.
  只用平面。卡片是纯色填充（`#FFFFFF`，或刻意深色卡片用 `#111112`），按套件比例取 22-24px 圆角。不要边框、不要描边、不要盒阴影、任何地方都不要立体投影；产品的卡片组件甚至没有阴影参数。行与行之间用 `rgba(0,0,0,.06-.10)` 的 1px 发丝线分隔，绝不用描边盒子。
- Color is rare and means something. Use one accent family per
  card: accent `#0064D4` (tint `rgba(0,100,212,.08)`), success
  `#25B159` fills / `#008200` text, error `#C01F37`. Everything else is
  the neutral ramp. Repeat the accent on related marks to show a relationship.
  颜色稀少且各有含义。每张卡片只用一族强调色：强调 `#0064D4`（淡色 `rgba(0,100,212,.08)`）、成功填充 `#25B159` / 成功文字 `#008200`、错误 `#C01F37`。其余一切都是中性色阶。在相关标记上重复同一强调色以表达关联。
- Uppercase micro-labels (11px kit scale, 600, `.08em` tracking,
  tertiary color) are the section-header idiom inside cards.
  大写微标签（套件比例 11px、字重 600、`.08em` 字距、tertiary 色）是卡片内部的小节标题惯用法。
- Chips, pills, and buttons are fully rounded (`border-radius:999px`)
  with low-alpha tint fills, never outlines. The primary button is a
  black `#111112` pill with white text; the `#0064D4 → #4553E4`
  gradient pill (the mobile canonical blue-indigo-650 stops) is for
  primary actions on product-parity cards and at most one hero moment
  elsewhere.
  芯片、胶囊与按钮全圆角（`border-radius:999px`），用低透明度淡色填充，绝不用描边。主按钮是黑 `#111112` 胶囊配白字；`#0064D4 → #4553E4` 渐变胶囊（移动端规范的蓝靛-650 端点色）只用于产品对等卡片上的主操作，以及别处至多一个高光时刻。
- Messages live in bubbles, radius ~18-22 at kit scale, canonical
  default chat theme: outgoing (user) `#CBE5FF` with `#111112` text,
  incoming (Muse) white with `#111112` text — the same bubbles the
  renderer draws for the thread itself.
  消息放在气泡里，套件比例圆角约 18-22，规范的默认聊天主题：发出（用户）`#CBE5FF` 配 `#111112` 文字，收到（Muse）白色配 `#111112` 文字——与渲染器为会话线程本身绘制的气泡相同。

## Fonts: one face, three narrow exceptions / 字体：一款字面，三个狭窄例外

Optimistic AI is THE face. Every piece of text on every card — titles,
body, captions, timestamps, counts, meta lines, badges — is Optimistic
AI at weights 400, 500, or 600, sized on the ramps this file names.
Meta and timestamps are NOT mono: they are Optimistic AI captions
(12/16 or 11/13) in secondary or tertiary color, exactly like the
shipping clients. The only sanctioned departures:

Optimistic AI 是唯一字面。每张卡片上的每一段文字——标题、正文、说明、时间戳、计数、元信息行、徽标——都是 Optimistic AI，字重 400、500 或 600，字号按本文件给出的色阶。元信息与时间戳不是等宽字体：它们是 secondary 或 tertiary 色的 Optimistic AI 说明文字（12/16 或 11/13），与已上线客户端完全一致。唯一获批的例外：

- **Optimistic Mono** appears where the product itself uses it and
  nowhere else: the scheduled-list group headers (13px SemiBold
  uppercase — the one mono moment in the UI), file paths and
  extension labels (`memory/people/mom.md`, "PDF"-style technical
  labels may also stay AI), and genuinely masked/tabular values
  (`•••• 4412`). A mono timestamp, count, or caption is
  off-language.
  **Optimistic Mono** 只出现在产品本身使用它的地方，别处不用：日程列表的分组标题（13px SemiBold 大写——全 UI 中唯一一处等宽时刻）、文件路径与扩展名标签（`memory/people/mom.md`；"PDF" 一类的技术标签也可继续用 AI 字面），以及真正的遮蔽/表格数值（`•••• 4412`）。等宽的时间戳、计数或说明文字都是脱离语言规范的。
- **MuseLetterCaveat** exists ONLY inside the letter card (its
  handwriting title and byline). Never elsewhere.
  **MuseLetterCaveat** 只存在于信件卡片内部（其手写标题与署名）。绝不出现在别处。
- **Real-product recreations keep their own stack**: the Gmail
  composer renders in Gmail's sans, an artifact's interior uses the
  artifact's own fonts, a real screenshot is whatever it is. The
  Muse faces govern the Muse chrome around them.
  **真实产品的重建保留其自身字体栈**：Gmail 撰写窗口用 Gmail 的无衬线字体渲染，制品内部使用该制品自己的字体，真实截图是什么样就是什么样。Muse 字面只管它们周围的 Muse 框架。
- Weights above 600 appear only where a canonical trace demands them
  (the connect sheet's 22/32 Bold name, Caveat's 700 letter title).
  高于 600 的字重只在规范描摹要求之处出现（connect 面板 22/32 Bold 的姓名、Caveat 700 的信件标题）。

All faces ship in `/opt/hatch/skills/magic-moment/assets/fonts/` and
are registered by the renderer, so kit markup renders as designed with
no font stack changes.

所有字面都随 `/opt/hatch/skills/magic-moment/assets/fonts/` 交付，并由渲染器注册，因此套件标记无需改动字体栈即可按设计渲染。

## Artifact cards are portraits, not templates / 制品卡片是肖像，不是模板

When the story's artifact exists — a page or app the user actually
built — the artifact card shows THAT artifact, not the kit's skeleton.
FIRST capture it for real: `./mm snap <its entry html> --run <name>`
renders the artifact's own document (relative assets and all) through
the capture browser and saves a true screenshot — crop it per
`reference/card-spec.md` and put it in the viewport, exactly what the
product would show. When the artifact cannot render the narrated state (missing session
data, server-only state, or missing assets), fall back to reading its code and recreating
its real design inside the viewport — its actual name, its own colors
and typography, its real layout and content, shrunk to a believable
mini-render. The kit's artifact
components fix only the anatomy around the viewport (browser chrome,
title row, actions); everything inside the viewport is
the user's app, uniquely designed per what they created. Two videos
about two different apps must show two visibly different apps; a
skeleton or generic mock where a real artifact exists is a failed
card (and for a live page, a real screenshot crop beats any
recreation). The Muse design language governs the card around the
viewport; the artifact's own look governs the inside — same rule as
the browser card's site viewport.

当故事中的制品真实存在——用户实际构建的页面或应用——制品卡片就该显示那个制品，而不是套件的骨架。首先真实地捕获它：`./mm snap <its entry html> --run <name>` 会通过捕获浏览器渲染制品自己的文档（相对资源在内）并保存真实截图——按 `reference/card-spec.md` 裁剪后放进视口，正是产品会显示的样子。当制品无法渲染旁白所述状态（缺少会话数据、仅服务端状态、或缺失资源）时，退而读取其代码，在视口内重建其真实设计——它真实的名称、它自己的颜色与字体、它真实的版面与内容，缩小成可信的迷你渲染。套件的制品组件只固定视口周围的结构（浏览器框架、标题行、操作）；视口之内的一切都是用户的应用，按其创作内容独特设计。关于两个不同应用的两支视频必须呈现两个肉眼可见不同的应用；真实制品存在时却放骨架或通用模型，就是一张失败的卡片（而对在线页面而言，真实截图裁剪胜过任何重建）。Muse 设计语言管视口周围的卡片；制品自己的外观管视口内部——与浏览器卡片的网站视口同一规则。

## Product-parity components copy the product exactly / 产品对等组件精确复制产品

These kit components are traced from the canonical mobile design and specification Figma files for Muse, not composed from the palette: the Sentinel approval card
(centered, gradient Allow), the secure
storage pair (log-in details list + add sheet), the connect pair
(in-chat `#0064D4` Medium link line + the r38/58 disclosure sheet),
the avatar options grid, the Goals surface (hairline list with the
concentric-dot section header, checkbox rows, activity rail; a goal
checking off gets the product's SMALL confetti burst on the checkbox
— box fills, ~6 tiny particles pop outward once and fade, title
drops to the 37% done tint — the kit's checked subgoal row is the
timed reference), the  
Feed edition (kicker/headline newspaper rows), Ideas rows (44px wash
squircle chip, dividers inset 72), the scheduled-tasks agent list
(Optimistic Mono uppercase group headers, 64px icon-tile rows),
shopping (compact product card, inline product chips, more-like-this
minis, the "Order placed" status chip), Muse
Wallet (tableview screen + the purchase-approval HITL card), the
browser activity island (glass 308×60 with the screenshot deck), and
the black full-bleed Title Card chapter divider.
Preserve their recognizable controls and change only the supported facts.
Retain inner geometry that explains the interaction. Remove redundant outer
wrappers when embedding a full screen in a card. Their anatomy IS the
product, and inventing a different vault, wallet, or approval layout
ships a screen that does not exist. Canonical quirks these surfaces
carry (product truth outranks the palette defaults): app surfaces sit
on `#FCFCFC`, not white or wash; the tertiary tier is
`rgba(0,7,17,.37)` for icons/times/disabled; feed links are
`#1793FF`; scheduled group headers are the one Optimistic Mono
moment in the UI; payment/connect sheets are r38-top; and — on FLOATING
chat-layer surfaces only (the approval card, the
disclosure sheet) — the product's soft elevation (`0 3px 9px
rgba(0,0,0,.02), 0 10px 50px rgba(0,0,0,.10)`). In-card content
stays flat everywhere else.

这些套件组件是从 Muse 的规范移动端设计与规格 Figma 文件描摹而来，不是用调色板拼凑的：Sentinel 审批卡片（居中、渐变 Allow）、安全存储组合（登录信息列表 + 添加面板）、connect 组合（会话内 `#0064D4` Medium 链接行 + r38/58 披露面板）、头像选项网格、Goals 界面（发丝线列表配同心圆点小节标题、复选框行、活动栏；目标勾选时复选框上呈现产品的小型彩带迸发——方格填充、约 6 颗微粒向外弹出一次并消散、标题降为 37% 完成度的淡色——套件中已勾选的子目标行就是计时参照）、  
Feed 版式（引题/标题的报纸行）、Ideas 行（44px 淡色方圆角芯片、分隔线内缩 72）、日程任务代理列表（Optimistic Mono 大写分组标题、64px 图标瓷贴行）、购物（紧凑产品卡、行内产品芯片、more-like-this 迷你卡、"Order placed" 状态芯片）、Muse Wallet（tableview 屏幕 + 购买审批 HITL 卡片）、浏览器活动岛（玻璃质感 308×60 配截图组）以及黑色全出血的 Title Card 章节分隔卡。
保留其可辨识控件，只更改受支持的事实。保留能解释交互的内部几何。把整块屏幕嵌入卡片时移除冗余外层包装。它们的结构本身就是产品；另造一个不同的保管库、钱包或审批版面，等于上线一个不存在的屏幕。这些界面携带的规范细节（产品真相优先于调色板默认值）：应用界面底色是 `#FCFCFC`，不是白色或淡染；tertiary 层对图标/时间/禁用态取 `rgba(0,7,17,.37)`；feed 链接为 `#1793FF`；日程分组标题是全 UI 唯一的 Optimistic Mono 时刻；支付/connect 面板顶部圆角 r38；以及——仅在浮动聊天层界面上（审批卡、披露面板）——产品本身的柔和投影（`0 3px 9px rgba(0,0,0,.02), 0 10px 50px rgba(0,0,0,.10)`）。其余地方的卡片内容保持扁平。

## Shipping widget replicas outrank composed cards / 线上组件复刻优先于拼装卡片

Chat moments should look like the widgets Muse ACTUALLY renders,
not like cards in the same palette. The kit's "shipping widget
replica" components are traced from the web client's own renderers —
copy them verbatim and change only the facts:

聊天时刻应当看起来像 Muse 真正渲染的组件，而不是同一调色板下拼出的卡片。套件的"线上组件复刻"组件是从网页客户端自己的渲染器描摹而来——逐字复制，只更改事实：

- **File/artifact card**: 320px, radius 16, hairline border, a 40px
  file-type icon beside a 13/18/600 name over the UPPERCASED
  extension in tertiary, then a 16:9 preview well (bottom corners
  only) split off by a 1px inset top hairline. The preview is the
  real document. Slides variant: wider, amber 40px icon tile,
  "Slides · N". User uploads: the 256px chip, accent-tinted tile.
  **文件/制品卡片**：320px、圆角 16、发丝线边框，40px 文件类型图标旁是 13/18/600 的名称，其下是 tertiary 色的全大写扩展名，再往下是由 1px 内嵌顶部发丝线分隔出的 16:9 预览井（仅底部两角有圆角）。预览就是真实文档。Slides 变体：更宽、琥珀色 40px 图标瓷贴、"Slides · N"。用户上传：256px 芯片、强调色淡染瓷贴。
- **Shopping results**: the grid lives INSIDE the assistant bubble
  (white radius-22, 14px padding) — borderless tiles, square image
  on wash radius-12, "brand · domain" caption, 2-line 12/16/600
  name, price with struck original.
  **购物结果**：网格放在助手气泡内部（白色、圆角 22、内边距 14px）——无边框瓷贴、淡染底上的方形图片（圆角 12）、"brand · domain" 说明、两行 12/16/600 名称、价格附划线原价。
- **Dashed choices**: the option widget is a transparent stack of
  radius-6 DASHED-border buttons (chosen goes solid, losers dim to
  50%). Choose-an-image moments (avatar looks, generated options)
  use the MOBILE picker anatomy, not the web one: a white radius-22
  card holding a bare 2×2 grid of radius-8 images — no in-card
  title, no dashed tiles, no confirm button — and the pick plays as
  a user click that answers by QUOTE-REPLY: the touch ring lands on
  a tile, it dips, then "↩ You replied to `<agent>`" lands
  right-aligned above the chosen image as its own bubble with the
  black "Option N" corner chip. The kit's "image picker · mobile
  canonical" card is the timed reference. The dashed seam also
  ends text_with_button: prose in the bubble shell, then a
  full-bleed footer CTA behind a dashed top hairline.
  **虚线选项**：选项组件是透明堆叠的圆角 6 虚线边框按钮（选中者变实线，落选者暗至 50%）。选图时刻（头像造型、生成选项）使用移动端选择器结构，而不是网页版：一张白色圆角 22 的卡片，装着一个裸露的 2×2 圆角 8 图片网格——卡内无标题、无虚线瓷贴、无确认按钮——选择以用户点击的方式呈现，并以引用回复作答：触控环落在一个瓷贴上，它下压，随后 "↩ You replied to `<agent>`" 右对齐落在被选图片上方，作为自己的气泡并带黑色 "Option N" 角标芯片。套件的 "image picker · mobile canonical" 卡片是计时参照。虚线接缝也用于收束 text_with_button：气泡壳内是正文，其下是虚线顶部发丝线之后的全出血页脚 CTA。
- **Letter**: warm paper gradient (#fbf6ec→#f1e7d3), Caveat
  handwriting title in #a4623f, a 5°-rotated emoji stamp, three grey
  fake-text bars, an optional −3° photo sticker with a tape strip.
  Theme-independent; the single most recognizable Muse card.
  **信件**：暖纸渐变（#fbf6ec→#f1e7d3）、#a4623f 的 Caveat 手写标题、旋转 5° 的 emoji 邮票、三道灰色假文条、可选的 −3° 旋转照片贴纸加胶带条。与主题无关；是全系列最易辨认的 Muse 卡片。
- **Idea card**: radius-24 bordered card, 16:9 preview, 16/22/600
  title, 14/20 secondary description, an UPPERCASE tertiary meta
  line with a 4px dot separator, and the accent radius-14 "Let's do
  it" pill with the wand glyph.
  **Idea 卡片**：圆角 24 描边卡片、16:9 预览、16/22/600 标题、14/20 secondary 描述、带 4px 圆点分隔符的全大写 tertiary 元信息行，以及带魔杖图形的强调色圆角 14 "Let's do it" 胶囊。
- **External link**: radius-16 bordered unfurl, hero capped ~208px,
  the hand-drawn globe tile when bare, "siteName · host" footer,
  trailing external arrow. **Audio**: the 280px fully-round pill —
  32px play circle on wash, 3px waveform bars (played = accent,
  unplayed = 20% black), tabular m:ss. **Browser**: the renderer
  draws this card itself (shell, address bar, touring cursor, click
  ring, page swap) from the rebuilt pages on the browser beat, so
  there is no markup here to copy and no "Take over" pill ("Browser
  cards rebuild the real page" owns the full rule).
  **外部链接**：圆角 16 描边展开卡，主图上限约 208px，无图时用手绘地球瓷贴，"siteName · host" 页脚，尾部外链箭头。**音频**：280px 全圆角胶囊——淡染底上 32px 播放圆、3px 波形条（已播放 = 强调色，未播放 = 20% 黑），等宽数字 m:ss。**浏览器**：渲染器在浏览器节拍上从重建页面自行绘制这张卡片（外壳、地址栏、游走光标、点击环、页面切换），因此这里没有可复制的标记，也没有 "Take over" 胶囊（完整规则见 "Browser cards rebuild the real page" 一节）。

Widget replicas use the client's own type ramp (16/22 body, 14/20,
13/18, 12/16, 11/13) rather than the mobile 17/22 ramp the bubbles
use — that mismatch is faithful, not a bug. Scaling: ×3.75 is the
minimum and the CANVAS is the ceiling — a card's outer width never
exceeds the 1240px body (the capture crops anything wider, shearing
borders and corners without a validator error). When a replica's
largest line lands under the validator's 56px dominant floor at
×3.75 (the 13px file-card name, the 14px link title), scale the
INNER metrics — type, paddings, icons, radii — uniformly by
56 ÷ largest-px, and pin the outer width to fill the 1240px canvas
instead of scaling past it: the card reads like the product at a
large accessibility text size. Never inflate one line alone; a
single bumped size breaks the anatomy the floor exists to protect. The client's card
entrance is a spring pop (scale .8→1); approximate it with a ~.35s
overshoot cubic-bezier when a widget card enters mid-video.

组件复刻使用客户端自己的字号阶梯（16/22 正文、14/20、13/18、12/16、11/13），而不是气泡所用的移动端 17/22 阶梯——这种不一致是忠实的，不是缺陷。缩放：×3.75 是下限，画布是上限——卡片外宽绝不超过 1240px 正文宽度（更宽的内容会被捕获裁掉，边框与圆角被削去而不报校验错误）。当某个复刻在 ×3.75 下最大一行仍低于校验器 56px 的主导字号下限时（13px 的文件卡名称、14px 的链接标题），把内部度量——字号、内边距、图标、圆角——按 56 ÷ 最大像素值统一缩放，并把外宽钉在填满 1240px 画布而不是越过它：卡片读起来就像产品的大号辅助功能文字。绝不只放大单独一行；单独调大一个字号会破坏这条下限所要保护的结构。客户端的卡片入场是弹簧弹出（scale .8→1）；当组件卡片在视频中途入场时，用约 .35s 的过冲 cubic-bezier 近似它。

Never invent system or security chrome. The product has no "Muse
is driving" pill, no "Vault" badge, and no "Encrypted token sent"
caption — copy like that reads as fake UI and is banned. Browser
presence is expressed only by the browser beat's rebuilt pages and
the activity island; credential moments use the canonical secure-storage
surfaces with, at most, the plain "Filled from secure storage"
caption; encryption and token mechanics never render as UI copy at
all — the security story is told by the approval card, not by
badges.

绝不发明系统或安全类界面文案。产品没有 "Muse is driving" 胶囊、没有 "Vault" 徽标、也没有 "Encrypted token sent" 说明——这类文案读起来像假 UI，一律禁止。浏览器在场感只通过浏览器节拍重建的页面与活动岛表达；凭据时刻使用规范的安全存储界面，至多加一句平实的 "Filled from secure storage" 说明；加密与令牌机制绝不作为 UI 文案渲染——安全故事由审批卡讲述，而不是由徽标讲述。

【评论】该条款把安全表达收敛到用户已熟悉的既有组件（审批卡、安全存储界面）上，避免自创徽标式文案让界面显得像伪造 UI。

## Placeholders never ship / 占位块绝不出厂

Every media slot in the kit — image and screenshot blocks — is a
neutral striped block carrying `data-ph="image"` or
`data-ph="screenshot"`. A copied component keeps
that attribute until you fill the slot, and `validate` REJECTS any card
that still carries one (or the striped/dashed look without it). Before
a card ships, every slot is either filled with real media or removed:

套件中的每个媒体槽——图片与截图块——都是携带 `data-ph="image"` 或 `data-ph="screenshot"` 的中性条纹块。复制来的组件在填入内容前保留该属性，而 `validate` 会拒绝任何仍带该属性的卡片（或带条纹/虚线外观却无该属性的卡片）。卡片出厂前，每个槽要么填入真实媒体，要么移除：

- Screenshots and photos: real material from the thread, a crop per
  `reference/card-spec.md`, or generated media through its own beat.
  Generated avatar media counts as real media only when the moment is
  literally about the avatar (the avatar-generation celebration card);
  no other card carries an avatar.
  截图与照片：线程中的真实素材、按 `reference/card-spec.md` 的裁剪、或经各自节拍生成的媒体。生成的头像媒体只有在时刻本身就是关于头像时（头像生成庆祝卡片）才算真实媒体；其他任何卡片都不得携带头像。
- A slot with nothing real to fill it gets removed, and the card
  re-balanced — a placeholder in a final video is a failed build.
  没有真实内容可填的槽一律移除，并重新平衡卡片——最终视频中出现占位块就是一次失败的构建。

【评论】用校验器在构建期拒绝占位块，把"最终产物不得含占位内容"从口头约定变成机器可执行的约束。

Local `file://` images are the one non-inline resource cards may
reference; network fetches stay banned.

本地 `file://` 图片是卡片唯一可引用的非内联资源；网络抓取仍然禁止。

## Scale kit markup to the 1240px canvas / 把组件标记缩放到 1240px 画布

Kit components are authored around a 330px card width; our canvas is a
1240px-wide body. When you copy a component, multiply every px value in
it (font sizes, paddings, radii, widths, heights) by 3.75 and round.
A kit 11px label becomes 41px; a 13px body line becomes 49px; a 17px
title becomes 64px; a 16px inner padding becomes 60px. Scaling
everything by the same factor is what preserves the kit's proportions;
scaling only the text is what makes cards look inflated.

套件组件按 330px 卡片宽度编写；我们的画布是 1240px 宽的正文。复制组件时，把其中每个 px 值（字号、内边距、圆角、宽、高）乘以 3.75 并取整。套件的 11px 标签变成 41px；13px 正文行变成 49px；17px 标题变成 64px；16px 内边距变成 60px。所有值按同一因子缩放才保得住套件的比例；只放大文字正是卡片显得虚胀的原因。

## One card per component, flat inside, nothing bare on the video / 每个组件一张卡片，内部扁平，视频上不允许裸露内容

Every component is exactly one solid card: a single filled, rounded
surface from the kit palette, sitting on the creator's footage.
Everything inside that card lies flat on its surface — text rows,
icons, labels — with chips, tabs, badges, and buttons as the only
inner elements that carry their own small fills. Two failure modes,
both banned:

每个组件恰好是一张实心卡片：来自套件调色板的单一填充圆角表面，落在创作者的实拍素材上。卡片内部的一切都平铺在其表面上——文字行、图标、标签——只有芯片、标签页、徽标与按钮这些内部元素可以自带小型填充。两种失败模式，一律禁止：

- The double card: a padded, rounded, filled panel inside the card (a
  message shown as a filled bubble, a highlight row in a tinted box).
  Show the message as a flat row instead: sender label, the text, a
  small status chip.
  双层卡片：卡片内部又出现一块带内边距、圆角、填充的面板（把消息画成填充气泡、把高亮行装进淡色盒子）。应把消息显示为扁平行：发送者标签、文字、一个小状态芯片。
- Bare content: any text or element sitting directly on the video with
  no card under it. Footage is a busy background; content without a
  solid surface reads as broken and washes out on light clothing.
  裸露内容：任何直接落在视频上、底下没有卡片的文字或元素。实拍素材是嘈杂的背景；没有实心表面承托的内容读起来像坏了，在浅色衣物上会被洗白。

The validator enforces all three edges: a surface on the page body is
rejected (the kit page's display chrome never ships), a card-sized
filled element inside another filled element is rejected, and visible
text with no filled surface anywhere above it is rejected.

校验器强制全部三条边界：页面正文上的表面会被拒绝（套件页面的展示框架绝不出厂）、一张卡片大小的填充元素嵌在另一个填充元素内会被拒绝、可见文字上方任何位置都没有填充表面会被拒绝。

Anatomy for a conversation preview: one card. A header row ("Reply to
Angel"), each message as a flat row — bold sender label, then the
text, the draft distinguished by an accent-colored label or a small
chip ("draft ready"), never by wrapping it in its own filled bubble. A
digest or confirmation is the same shape: one card, flat rows, chips
for state.

对话预览的结构：一张卡片。一行表头（"Reply to Angel"），每条消息是一行扁平行——加粗发送者标签、然后是文字；草稿用强调色标签或小芯片（"draft ready"）区分，绝不用自带的填充气泡包裹。摘要或确认也是同一形状：一张卡片、扁平行、用芯片表达状态。

## Breathing room / 呼吸空间

After scaling, give content slightly more air than the kit default:
inner padding of at least 64px on every side at 1240px width, and keep
text and controls clear of the rounded corners (nothing within the
corner radius arc). A cramped corner is the first thing that reads as
machine-generated.

缩放之后，给内容比套件默认值稍多的留白：1240px 宽度下每侧内边距至少 64px，并让文字与控件远离圆角（圆角弧线以内不放任何东西）。局促的角落是最先暴露机器生成痕迹的地方。

## Card motion / 卡片动效

Leave card entry, exit, and movement through the thread to the renderer.
Do not add CSS fades, entrance transforms, or exit animations to the whole
card. Keep card text readable. Animate controls and label changes only to
depict an interaction supported by the source, following the interaction
guidance below.

卡片的进场、退场与沿线程的移动交给渲染器。不要给整张卡片添加 CSS 淡入淡出、入场变换或退场动画。保持卡片文字可读。只在描绘源素材支持的交互时给控件与标签变化做动画，遵循下文的交互指引。

## Browser cards rebuild the real page / 浏览器卡片重建真实页面

A browser beat depicts Muse driving a website, and you build the pages
it shows. Ground the beat before you author a page. `./mm webshots`
lists the browser tool's own action captures with each one's page URL
and timestamp, and `./mm webshots --run <run> --copy N [--url-contains
<site>]` pulls the newest captures into the run directory alongside a
`webshots.json` holding their true URLs. Run it first on any browser
story, and run it early, because the capture store is
garbage-collected. Check `~/workspace/your_files/` and the parent
conversation's attachments for the same material. Read those captures
for the site's real layout, the story's real item, and its real price.
Rebuild the pages from what you read. When no capture survives on this VM,
`./mm snap <https url>` re-visits the live page through the cell's
egress and screenshots it, so you can source the same details from
that.

浏览器节拍描绘 Muse 驾驶一个网站，而页面由你来构建。动笔写页面之前先落实素材。`./mm webshots` 列出浏览器工具自己的动作捕获及每条的页面 URL 与时间戳，`./mm webshots --run <run> --copy N [--url-contains <site>]` 把最新的捕获拉入运行目录，并附上记录其真实 URL 的 `webshots.json`。任何浏览器故事都先运行它，而且要趁早，因为捕获存储会被垃圾回收。同时检查 `~/workspace/your_files/` 与父会话的附件是否有同样的素材。读这些捕获，获取网站的真实版面、故事中的真实商品及其真实价格。依据读到的内容重建页面。当本 VM 上没有捕获留存时，`./mm snap <https url>` 会经单元格的出口重新访问在线页面并截图，同样可以从中取材。

Author one browser beat per journey: `{"type": "browser", "address":
"rei.com", "reference": ["<run>/webshot_00_www.rei.com.png"], "pages":  
[{"html": "<page 1>", "click": "#add-to-cart"}, {"html": "<page 2>"}],  
"start": 12.0, "end": 22.0}`. Give the beat at
least two pages, because a journey with one page has nowhere to go.
Set `"address"` to the real site's domain, as one string for the whole
journey or as a list matching `pages` one to one. Name the captures your
pages rebuild from in `"reference"`, as one path or a list of paths. The
validator rejects a browser beat that names no `"reference"` or names a
path that does not exist. The run's provenance record lists the captures
you rebuilt from. With no capture and no live snap you have no
`"reference"`, and therefore no browser beat. Present the tool that did
the work instead, per the last paragraph of this section. Do not split
one journey into a card per page.

每段行程编写一个浏览器节拍：`{"type": "browser", "address": "rei.com", "reference": ["<run>/webshot_00_www.rei.com.png"], "pages":  
[{"html": "<page 1>", "click": "#add-to-cart"}, {"html": "<page 2>"}],  
"start": 12.0, "end": 22.0}`。给节拍至少两页，因为只有一页的行程无处可去。`"address"` 设为真实站点的域名，整段行程用一个字符串，或与 `pages` 一一对应的列表。在 `"reference"` 中写明页面重建所依据的捕获，用单个路径或路径列表。校验器会拒绝未指名 `"reference"` 或指名了不存在路径的浏览器节拍。运行的溯源记录会列出你重建所依据的捕获。既无捕获又无在线截图时就没有 `"reference"`，因而也没有浏览器节拍。此时按本节最后一段，改为呈现实际完成工作的工具。不要把一段行程拆成每页一张卡片。

Rebuild each page as a simplified version of a page the browser really
drove. Give it that site's own layout, header, colors, and font stacks,
per this file's "Real-product recreations keep their own stack" rule.
Cut it down to the elements the story needs: the header, the item, its
price, and the control the cursor presses. Simplified means fewer
sections than the real page. It does not mean lower fidelity in the
sections you keep. Do not put placeholder
stripes, gray boxes, or lorem text on a page; the skeleton ban covers
rebuilt pages too.

把每个页面重建为浏览器真实驾驶过的页面的简化版。按本文件 "Real-product recreations keep their own stack" 规则，给它该站点自己的版面、页头、颜色与字体栈。裁剪到故事需要的元素：页头、商品、价格、以及光标按下的控件。简化指比真实页面更少的区块；不指保留区块里的保真度降低。不要在页面上放占位条纹、灰盒或 lorem 文字；骨架禁令同样覆盖重建页面。

Write each page with its reference capture open. Work from what the
capture shows, not from what you remember about the site. Reproduce
that page's anatomy exactly as the capture shows it: the header lockup,
the nav labels, the breadcrumb, the layout proportions, the type scale
and weights, and the button copy. Sample the page's colors out of the
capture with PIL, including the background washes, the brand accents,
and the sale or CTA colors. Do not guess a brand's colors from memory.
Someone who uses the site should recognize the rebuilt page at a
glance.

写每一页时都把参照捕获摆在手边。依据捕获显示的内容工作，而不是凭你对站点的记忆。精确复现捕获所示页面结构：页头组合、导航标签、面包屑、版面比例、字号阶梯与字重，以及按钮文案。用 PIL 从捕获中取样页面颜色，包括背景淡染、品牌强调色，以及促销或 CTA 颜色。不要凭记忆猜品牌颜色。常用该站点的人应当一眼认出重建的页面。

Put the story's real item on the page. Its name, its price, and its
image come from the trajectory you just read. Crop the item's image out
of a real capture with PIL per
`/opt/hatch/skills/magic-moment/reference/card-spec.md`, and reference
the crop in the page markup by absolute `file://` path. Crop the site's
logo lockup out of the capture the same way and place it by absolute
`file://` path; a redrawn logo reads as a knock-off. The capture
engine loads no external URLs, so a web image URL renders as a broken
box. Every specific on a page (a price, a product name, a time, a code)
comes from your fact sheet, and the validator lints page text with the
same rule it applies to bubbles and cards.

把故事中的真实商品放到页面上。它的名称、价格与图片来自你刚读过的轨迹。按 `/opt/hatch/skills/magic-moment/reference/card-spec.md`，用 PIL 从真实捕获中裁出商品图片，并在页面标记中以绝对 `file://` 路径引用该裁剪。同样方式裁出站点的标志组合并按绝对 `file://` 路径放置；重画的标志读起来像山寨。捕获引擎不加载外部 URL，网络图片 URL 会渲染成破图。页面上的每个具体信息（价格、商品名、时间、代码）都来自你的事实清单，校验器以与气泡和卡片相同的规则审查页面文字。

Author each page as a fixed desktop viewport: `body` width 1240px with
640px of visible height, the aspect of the browser's own ~1919×992
captures. Content below 640px is cut, the way a real browser folds a
page.

每页按固定桌面视口编写：`body` 宽 1240px、可见高度 640px，即浏览器自身约 1919×992 捕获的宽高比。640px 以下的内容被裁掉，就像真实浏览器折叠页面那样。

Give every page except the last a `"click"`: a CSS selector naming the
element the cursor presses to reach the next page, like
`"#add-to-cart"`. The renderer measures that element's rendered box on
the page you wrote, lands the cursor on its center, pulses the click
ring there, and swaps to the next page. A selector that matches nothing
fails the render with the selector named. An element whose center sits
below the 640px fold fails the render too. A `[x_frac, y_frac]` pair is accepted
when no selector names the spot you want pressed.

除最后一页外，每页都给 `"click"`：一个 CSS 选择器，指明光标按下以进入下一页的元素，如 `"#add-to-cart"`。渲染器在你写的页面上度量该元素的渲染盒，把光标落到其中心，在那里脉动一次点击环，然后切换到下一页。匹配不到任何元素的选择器会使渲染失败，并指名该选择器。中心落在 640px 折叠线以下的元素同样使渲染失败。当没有选择器能指名你想按的位置时，接受 `[x_frac, y_frac]` 一对坐标。

The renderer owns every bit of motion on a browser beat: the shell, the
address bar, the touring cursor, the click ring, and the load-flash
page swap, across the whole 5.0 to 20.0 second beat. Do not write
`@keyframes` into a page, because the renderer captures each page as
one still. Browser beats tap by default, so the session plays at stage
size where the cursor work reads on a phone;
`/opt/hatch/skills/magic-moment/guide/visuals.md` owns the tap rule.

浏览器节拍上的每一点动效都归渲染器所有：外壳、地址栏、游走光标、点击环与加载闪光式页面切换，贯穿整个 5.0 到 20.0 秒的节拍。不要把 `@keyframes` 写进页面，因为渲染器把每页捕获为一帧静止画面。浏览器节拍默认为点按，会话以舞台尺寸播放，光标动作在手机上清晰可读；点按规则归 `/opt/hatch/skills/magic-moment/guide/visuals.md` 所有。

Do not put a "Take over" pill or any other invented affordance on a
browser card. When a skill or connector did the work instead of the
browser (Duffel for flights, a search API, any tool), present that tool
as itself with the skill task status chip (the canonical 316×64 chip
naming the skill, progressing to done, in the kit) or the plain thread.
Author no browser beat for that moment.

不要在浏览器卡片上放 "Take over" 胶囊或任何其他发明的控件。当是技能或连接器而非浏览器完成了工作（机票用 Duffel、搜索 API、任何工具），就把那个工具如实呈现，配上技能任务状态芯片（套件中规范的 316×64 芯片，写明技能名、推进到完成）或朴素的线程。那一刻不要编写浏览器节拍。

Interactive components PLAY their interaction. Anything with a
button or a choosable row shows one press: the touch ring lands
first — the same `#0064D4` ring the driving card's cursor clicks
with (a ~30px circle, 2.5px border over `rgba(0,100,212,.14)`,
scaling in and fading over ~0.6s), centered on the control ~0.3s
before the dip, so the viewer sees WHAT got pressed — then a ~0.3s
dip (`scale(.85-.96)` and back), then the consequence — the send button
dips and the draft lands as a sent bubble, Allow dips and resolves
to "Allowed ✓" while Deny fades, an option tile dips and takes the
selection ring while the others dim, a checkbox dips before it
fills. The kit's composer, Gmail, approval, connect, add-login,
avatar-grid, ideas, fullstack-app, goals, feed-heart, music,
completion, and browser-island cards are the timed references —
copy their grammar. One press per card; label swaps use two
stacked layers cross-fading (never text morphs); everything is
`1 forwards` so the card plays once and holds the resolved state. A
card full of buttons where nothing ever gets pressed reads as a
mockup, not a product.

交互组件要演出其交互。任何带按钮或可选行的组件都要呈现一次按压：触控环先落——与驾驶卡片光标点击所用的同一个 `#0064D4` 环（约 30px 圆、`rgba(0,100,212,.14)` 之上的 2.5px 描边，约 0.6s 内放大并淡出），在按压前约 0.3s 居中于该控件，让观众看清按的是什么——然后约 0.3s 的下压（`scale(.85-.96)` 再回弹），随后是后果——发送按钮下压、草稿作为已发送气泡落位；Allow 下压并定格为 "Allowed ✓" 而 Deny 淡出；选项瓷贴下压并取得选中环而其余变暗；复选框先下压再填充。套件的 composer、Gmail、approval、connect、add-login、avatar-grid、ideas、fullstack-app、goals、feed-heart、music、completion 与 browser-island 卡片是计时参照——复制它们的语法。每卡一次按压；标签切换用两层堆叠交叉淡化（绝不做文字变形）；一切都是 `1 forwards`，让卡片播放一次并保持解决后的状态。满卡按钮却无一处被按的卡片读起来像模型，不是产品。

The press finishes its story. Author the touch ring, the dip, and the
resolved state as finite animations
(`animation: <name> 4s ease-in-out 1 forwards`), not `infinite`. A
control that keeps re-pressing itself reads as a stuck robot. A card
whose animations are all finite plays once and holds its final frame; a
card that mixes a finite narrative with looping ambience captures both on one timeline through the beat endpoint. The finite action stays complete while the ambient animation continues.

按压要讲完自己的故事。把触控环、下压与解决后的状态写成有限动画（`animation: <name> 4s ease-in-out 1 forwards`），而不是 `infinite`。不断自己重复按压的控件读起来像卡住的机器人。动画全为有限的卡片播放一次并停在最后一帧；把有限叙事与循环氛围混在一张卡片上的，两者会在同一条时间线上经节拍端点一起捕获。氛围动画继续的同时，有限动作保持完整。

Tap sparingly on cards that are not browser beats: one or two beats per
video, reserved for the payoff
(`/opt/hatch/skills/magic-moment/guide/visuals.md` owns the rule).

非浏览器节拍的卡片上节制使用点按：每支视频一至两个节拍，只留给高潮时刻（规则归 `/opt/hatch/skills/magic-moment/guide/visuals.md` 所有）。

## The full-screen Muse finisher / 全屏 Muse 收尾

Let the renderer append the Muse close after the source footage. Do not author an ending card or avatar header. Keep authored beats within the source duration and preserve the concluding narration.

让渲染器在源素材之后追加 Muse 收尾。不要编写结尾卡片或头像页头。已编写的节拍保持在源素材时长之内，并保留结尾旁白。

## Mechanics (machine-enforced, unchanged by the kit) / 机制（机器强制，套件不可更改）

- Author at `body` width 1240px; card height under ~600px — or ~1100px
  on a `"tap": true` beat, where tall is what makes the growth land
  (`render_html` rejects taller than the beat's budget). Do not set
  body height, and keep the body itself bare: the card surface lives
  on the component's root element, never on the page body (the
  validator rejects it).
  以 `body` 宽 1240px 编写；卡片高度低于约 600px——或 `"tap": true` 节拍上的约 1100px——正是更高的卡片让增长镜头成立（`render_html` 拒绝超过节拍预算的高度）。不要设置 body 高度，并保持 body 本身裸露：卡片表面住在组件的根元素上，绝不在页面 body 上（校验器会拒绝）。
- Inline CSS only; no JavaScript; no external fetches (local `file://`
  images for filled media slots are the one exception); no headless
  browser renders.
  只用内联 CSS；不用 JavaScript；不做外部抓取（已填媒体槽的本地 `file://` 图片是唯一例外）；不做无头浏览器渲染。
- Font sizes in px only. Floors after scaling: nothing under 32px, and
  every card's largest rendered line at least 56px (`validate` rejects
  violations; only sizes the markup actually uses count).
  字号只用 px。缩放后的下限：不小于 32px，且每张卡片最大的一行渲染文字至少 56px（`validate` 拒绝违规；只有标记实际用到的字号才计数）。
- Every specific (name, price, code, time) needs source evidence in your
  fact sheet; the validator lints the card's visible text with the same
  rule as bubbles.
  每个具体信息（名称、价格、代码、时间）都要在你的事实清单中有来源证据；校验器以与气泡相同的规则审查卡片可见文字。
- Generated images and videos render as-is through their own beat
  types; the kit frames the moment around real media, never a synthetic
  stand-in for it.
  生成的图片与视频经各自的节拍类型原样渲染；套件围绕真实媒体构建时刻，绝不用合成替身。

Components in the kit, by section (plus a thread-chrome reference
section at the top — bubbles, chips, typing — which the renderer draws
itself and you never author):

套件组件按小节列出（顶部另有一个线程框架参照小节——气泡、芯片、输入中——由渲染器自行绘制，你绝不编写）：

- Artifacts & documents: web artifact (finished hero), fullstack app
  (two states: resting = the app's beveled space icon + name in a
  small card, exactly the Figma icon idiom; tapped = the tap effect
  fills the chat window with the LEGIT full web app — its own
  chrome, sections, and state, recreated from the real app or
  captured with `./mm snap`, never a widget-sized summary), finished
  files (one card per kind), completion card, sharing (link out,
  someone opens). There is no build/loader card anymore — an
  artifact enters the story finished (the hero's real screenshot);
  show the work through the browser card or the thread, never a
  progress skeleton.
  制品与文档：web artifact（完成态主图）、fullstack app（两个状态：静止 = 应用的小型空间图标 + 名称装在小卡片里，正是 Figma 图标惯用法；点按 = 点按效果让聊天窗口充满名正言顺的完整 Web 应用——它自己的框架、区块与状态，从真实应用重建或用 `./mm snap` 捕获，绝不是组件尺寸的摘要）、完成态文件（每种一张卡片）、completion card、分享（链接出去、有人打开）。不再有构建/加载卡片——制品以完成态进入故事（主图的真实截图）；工作过程通过浏览器卡片或线程展示，绝不用进度骨架。
- Media: generated image, generated video, avatar generation
  (celebration), music (found and playing), media library (organized).
  Generated images and videos render as-is; the kit frames the moment
  around them, never a synthetic stand-in.
  媒体：生成图片、生成视频、头像生成（庆祝）、音乐（找到并播放）、媒体库（整理好的）。生成的图片与视频原样渲染；套件围绕它们构建时刻，绝不用合成替身。
- Browser & shopping: live browser control (progress + proof),
  browser activity island (canonical glass row), web search
  (citations), shopping (compact product card, inline chips,
  more-like-this, canonical), purchase (order-placed status chip,
  canonical).
  浏览器与购物：实时浏览器控制（进度 + 证据）、浏览器活动岛（规范玻璃行）、web search（引用）、购物（紧凑产品卡、行内芯片、more-like-this，规范）、购买（order-placed 状态芯片，规范）。
- Trust & security: connect an account (in-chat link + disclosure
  sheet, canonical), secure storage pair (canonical), sentinel (the
  product approval card), secure form fill (card number, password).
  信任与安全：连接账户（会话内链接 + 披露面板，规范）、安全存储组合（规范）、sentinel（产品审批卡）、安全表单填写（卡号、密码）。
- Communication: texting on your behalf (a real message thread —
  contact header, date stamp, grouped bubbles, and the composer bar
  with the + circle, rounded-full field, and one send button; the
  send plays and the sent bubble lands in the thread),
  email triage, email draft (the legit Gmail compose window with its
  blue Send), phone call (live + outcome), voice (listening state).
  沟通：代你发短信（真实消息线程——联系人页头、日期戳、分组气泡，以及带 + 圆、全圆角输入框和一个发送按钮的撰写栏；发送动作演出、已发送气泡落进线程）、邮件分诊、邮件草稿（名正言顺的 Gmail 撰写窗口及其蓝色 Send）、电话（实时 + 结果）、语音（聆听状态）。
- Automation & ambient: scheduled tasks (agent list, canonical),
  standing watch (ambient guardian), proactive (unprompted nudge),
  morning brief (feed edition, canonical), ideas (idea rows,
  canonical).
  自动化与常驻：日程任务（代理列表，规范）、长期守望（环境守护者）、主动提醒（未经请求的提示）、晨报（feed 版式，规范）、ideas（idea 行，规范）。
- Personal intelligence: memory (recalled from weeks ago), person
  page (markdown memory doc, product design), goals surface +
  tracking check-in (canonical hairline lists), calendar (conflict
  resolved), device sync (flowing in).
  个人智能：memory（从数周前回忆）、person page（markdown 记忆文档，产品设计）、goals 界面 + 追踪打卡（规范发丝线列表）、calendar（冲突已解决）、设备同步（正在流入）。
- Work & analysis: deep research (multi-source report), uploaded file
  (analyzed), data analysis (chart from a spreadsheet), long thread
  (digest).
  工作与分析：深度研究（多来源报告）、上传文件（已分析）、数据分析（由电子表格生成的图表）、长线程（摘要）。
- Meta-capabilities: subagents (fanning out in parallel), wallet
  (Muse Wallet screen + purchase approval, canonical), title card
  (chapter divider, canonical).
  元能力：子代理（并行展开）、wallet（Muse Wallet 屏幕 + 购买审批，规范）、title card（章节分隔，规范）。
