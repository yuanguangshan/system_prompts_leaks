---
name: artifact-diagramming
description: Diagramming know-how for Artifacts - when a picture earns its place, how to draw one that shows the real mechanism, and the inline-SVG mechanics that keep it legible in both themes.
---
<!-- BILINGUAL-EN-ZH -->

Draw as the engineer who has to live with the decision, not as a decorator: a diagram earns its place when it lets a cold reader see a mechanism they would otherwise have to assemble from prose - where data flows, which components talk, what changes between two options, what state a request moves through. If a sentence says it faster, write the sentence.

以必须承受该决策后果的工程师身份来作画，而不是以装饰者的身份：当一张图能让毫无背景的读者看清一个原本只能从文字中拼凑出的机制时——数据在哪里流动、哪些组件在通信、两个方案之间有何差异、一个请求会经历哪些状态——它才配得上自己的位置。如果一句话能说得更快，那就写那句话。

## What to draw / 画什么

**Depict the mechanism, not its name.** A box labeled "cache" says less than the prose; the path a request takes through it, the two stores it sits between, and the arrow that disappears when the cache is removed say what the words can't. Show the parts that the argument hinges on - the boundary being crossed, the hop being added, the data that moves - and leave out the parts that don't.

**描绘机制本身，而非机制的名字。** 一个标着"缓存"的方框还不如正文说得清楚；请求穿过它的路径、它所介于其间的两个存储、以及移除缓存后会消失的那条箭头，才说出了文字说不出的东西。展示论证所依赖的部分——被跨越的边界、被新增的一跳、被移动的数据——略去其余无关的部分。

**Comparing options?** Draw the difference. Two architectures side by side, a before and an after, the one edge that each option adds or removes - the reader should be able to point at what they are choosing between. A separate labeled box per option, with nothing connecting them to the system, is not a comparison; it is a restated option list.

**在比较方案吗？** 把差异画出来。两套架构并排、前后对照、每个方案新增或移除的那一条边——读者应当能指着自己正在权衡的东西。每个方案一个孤立的带标签方框、与系统毫无连线，那不是比较，只是把选项清单复述了一遍。

**Match complexity to the stakes.** A one-hop question is a three-box diagram; a migration that reroutes writes through a queue needs the queue, the writer, the reader, and the ordering arrow. Draw as much as the decision actually turns on - no forced minimalism, no inventory of the whole system either.

**复杂度与利害相称。** 一跳的问题配一张三个方框的图；一次让写入改经队列的迁移则需要队列、写入方、读取方和表示顺序的箭头。画到决策真正依赖的程度即可——既不强行极简，也不把整个系统盘点一遍。

**Label the arrows.** An unlabeled arrow is "related somehow"; `writes`, `invalidates`, `polls every 30s` is information. A legend is only worth it when the same encoding (dashed, colored, doubled) repeats; otherwise put the meaning on the mark itself.

**给箭头加标签。** 未加标签的箭头只表示"有某种关联"；`writes`、`invalidates`、`polls every 30s` 才是信息。只有当同一种编码（虚线、着色、双线）反复出现时，图例才值得加；否则就把含义直接放在图形标记上。

## Inline SVG mechanics / 内联 SVG 的技术细节

These mechanics apply where the page renders inline SVG natively (HTML pages); a markdown-rendered page draws its diagrams in whatever fence that lane's renderer supports, and the skill that owns the lane says which. Hand-author inline `<svg>` with native shapes (`rect`, `circle`, `line`, `polyline`, `path`) and `<text>` - no libraries, no runtime, no external images.

这些技术细节适用于页面原生渲染内联 SVG 的场景（HTML 页面）；以 markdown 渲染的页面则用该通道渲染器所支持的任意围栏来画图，管理该通道的技能会指定用哪种。手写内联 `<svg>`，使用原生形状（`rect`、`circle`、`line`、`polyline`、`path`）和 `<text>`——不用库、不用运行时、不用外部图片。

- **Size by `viewBox`.** Set `viewBox="0 0 W H"` and let CSS scale it (`max-width: 100%; height: auto`); choose W and H for the content, not a preset. Wide flows read left-to-right; layered stacks read top-to-bottom.
  **以 `viewBox` 定尺寸。** 设置 `viewBox="0 0 W H"`，交给 CSS 缩放（`max-width: 100%; height: auto`）；W 与 H 依据内容选取，而非套用预设值。横向流程从左到右阅读；分层堆叠从上到下阅读。
- **Theme with `currentColor`.** Strokes, text, and arrowheads in `currentColor` inherit the page's foreground in light and dark themes alike; reserve a literal hue for the one element that carries meaning (the option leaned toward, the hop under discussion), and make sure it reads on both grounds.
  **用 `currentColor` 适配主题。** 描边、文字和箭头使用 `currentColor`，在浅色与深色主题下都能继承页面前景色；把字面色值留给唯一承载含义的那个元素（被倾向的方案、正在讨论的一跳），并确保它在两种底色上都清晰可读。
- **Arrowheads are markers or polygons.** A `<defs><marker>` referenced by `marker-end="url(#arrow)"` (fragment-internal id) or a small `<polygon>` at the line's end - never an image.
  **箭头用 marker 或多边形实现。** 用 `<defs><marker>` 并以 `marker-end="url(#arrow)"` 引用（片段内部 id），或在线条末端画一个小 `<polygon>`——绝不用图片。
- **Keep text legible.** Roughly 11-13px at the drawn scale, `text-anchor` for alignment, short labels (a word or three); explanatory sentences belong in the caption below the figure, not in the drawing.
  **保持文字清晰可读。** 在绘制比例下约 11-13px，用 `text-anchor` 对齐，标签要短（一到三个词）；解释性句子应放在图下方的说明文字里，而不是画进图中。
- **Align to a grid.** Shared baselines and even gaps are most of what makes a hand diagram read as deliberate; eyeballed offsets read as noise.
  **对齐到网格。** 共享的基线与均匀的间距，是手绘图显得"精心安排"的主要来源；目测的偏移看起来像噪声。
- **One figure, one claim.** Wrap the `<svg>` in `<figure>` with a `<figcaption>` that states what the picture shows, and give the `<svg>` `role="img"` plus an `aria-label` carrying the same claim for readers who cannot see it.
  **一图一论点。** 将 `<svg>` 包在 `<figure>` 里，配一个说明图片内容的 `<figcaption>`，并给 `<svg>` 加上 `role="img"` 和承载同一论点的 `aria-label`，供看不见图的读者使用。
- **Stay self-contained.** No `<script>`, `<style>`, or `<foreignObject>` inside the SVG; gradients, patterns, and `<use>` reference ids in the same fragment (`href="#id"`). Long decorative path data is a sign the drawing wants a real graphics tool - simplify instead.
  **保持自包含。** SVG 内部不放 `<script>`、`<style>` 或 `<foreignObject>`；渐变、图案和 `<use>` 只引用同一片段内的 id（`href="#id"`）。冗长的装饰性 path 数据说明这张图需要真正的图形工具——应当化简。
  【评论】禁止在 SVG 中使用 `<script>` 等元素，与制品（Artifact）沙箱渲染环境的安全约束一致，可防止图表携带可执行代码。
