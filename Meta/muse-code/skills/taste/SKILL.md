---
name: taste
description: Mandatory preflight for web frontend visual design. If available, call read_skill for bundled:taste before the first frontend create or edit tool call whenever creating or visually styling a human-viewed web page, component, app, dashboard, or interactive report. Do not load it for non-visual frontend logic or machine-only HTML. If unavailable, continue without warning. Negative constraints only; user requests and other explicit styles still win.
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Anti-Slop: Frontend DON'Ts / 反“AI 味”：前端设计禁忌

What NOT to do when designing a UI or page. This is a filter, not a style; it never prescribes a look. Stacking several of the below is what reads as AI-made on sight. Adaptive, so if the user or another skill asks for something specific, do that instead; these are defaults, not overrides.

设计 UI 或页面时不要做的事。这是一道过滤器，而不是一种风格；它从不规定某种外观。同时堆叠以下多条特征，正是“一眼看上去像 AI 生成”的原因。本清单是自适应的：如果用户或其他技能明确要求某种做法，就按要求的做；这些只是默认值，不是压倒性规则。

【评论】该清单以负向约束（只列“不要什么”）排除一批高频“AI 生成感”视觉套路，从而间接塑造产出风格，而非提供正向设计规范。

- purple / indigo / violet accent, or a purple-to-blue hero gradient (the top AI tell)
  紫色/靛蓝/紫罗兰色点缀，或紫到蓝的首屏渐变（最典型的 AI 痕迹）
- `bg-clip-text` gradient headline text
  使用 `bg-clip-text` 的渐变大标题文字
- cream (`#faf8f4`) as the whole-page background
  以米白（`#faf8f4`）作为整页背景
- aurora blobs / glowing colored shadows, or an identical glow on every card
  极光光斑/彩色发光阴影，或每张卡片上完全相同的辉光
- Inter everywhere, or Geist / Space Grotesk / Instrument Serif / Roboto / Arial as your only identity
  到处使用 Inter，或仅以 Geist / Space Grotesk / Instrument Serif / Roboto / Arial 作为唯一视觉识别
- gray-400 / gray-500 body text
  gray-400 / gray-500 的正文文字
- one font weight and size throughout (no hierarchy)
  全篇只有一种字重和字号（没有层级）
- tracked all-caps micro-kickers above every section
  每个区块上方都加字距拉开的全大写小眉题
- centered hero of eyebrow pill + headline + two CTAs
  居中首屏：眉题胶囊 + 大标题 + 两个 CTA
- the identical 3-card feature grid with a lucide icon in a rounded square
  千篇一律的三卡片功能网格，配圆角方块内的 lucide 图标
- an 8+ word, 48px+ headline that says nothing
  8 词以上、48px 以上却毫无信息量的大标题
- bento grid for everything
  什么都用便当格（bento grid）
- the same eyebrow-headline-3cards section repeated back to back
  “眉题-标题-三卡片”区块连续重复出现
- `rounded-2xl`, or any single radius, on every element
  所有元素都用 `rounded-2xl` 或其他任何单一圆角
- glassmorphism blur + border + shadow on every surface
  每个表面都堆玻璃拟态的模糊 + 边框 + 阴影
- `hover:scale-105` lift on cards
  卡片使用 `hover:scale-105` 悬浮抬升
- emoji as feature icons (🚀 ⚡ ✨ 🎯 💡)
  用 emoji 充当功能图标（🚀 ⚡ ✨ 🎯 💡）
- left-border accent cards, a colored border on a rounded element, or cards nested inside cards
  左侧色条卡片、圆角元素上的彩色边框，或卡片套卡片
- fade-up / slide-up on everything as it scrolls in
  所有元素滚入时都淡入上移/上滑进场
- motion that ignores `prefers-reduced-motion`
  无视 `prefers-reduced-motion` 的动画
- scribbly hand-drawn SVG mascots with no purpose
  毫无用途的潦草手绘 SVG 吉祥物
