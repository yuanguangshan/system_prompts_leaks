<!-- BILINGUAL-EN-ZH -->
# Marks & anatomy / 图元与构造

The quiet, considered look is a few fixed specs plus two pieces of negative space.
The data is the only thing allowed to be loud.

这种沉静、克制的外观来自几条固定规格加上两处留白。
数据是唯一被允许"抢眼"的元素。

## Mark specs (fixed across every chart) / 图元规格（所有图表统一）

| Mark | Spec |
|---|---|
| Bar / column | **<= 24px thick** (cap it - never fill the slot; let the band's leftover be air); **4px rounded data-end, square at the baseline**; grows from a single baseline |
| Line | **2px**, round join/cap |
| Marker / end-dot | **>= 8px** (r >= 4), filled with the series color |
| Area fill | the series hue at **~10% opacity** (a wash, never a saturated block) |
| Gridlines / axes | one-step-off-surface gray, **hairline (1px), solid** (never dashed), recessive |

| 图元 | 规格 |
|---|---|
| 条形 / 柱形 | **粗细 <= 24px**（设上限 —— 不要填满整个槽位；让分带的剩余空间保持透气）；**数据端 4px 圆角，基线端直角**；从单一基线生长 |
| 线条 | **2px**，圆角连接/端帽 |
| 标记 / 端点 | **>= 8px**（r >= 4），以系列色填充 |
| 面积填充 | 系列色相、**约 10% 不透明度**（一层淡染，绝不做饱和色块） |
| 网格线 / 坐标轴 | 比底色深一档的灰色，**发丝线（1px）、实线**（绝不用虚线），保持低调 |

## The two spacers (white doing the separating) / 两种间隔物（用底色完成分隔）

- **Surface gap.** A **2px gap** in the surface color separates touching marks - every
  segment of a stacked bar, and every adjacent (touching) bar, the same width. Keep it
  one consistent width across a stack; neighbors one step apart read distinct because of
  the gap, not a stroke drawn around them.
  **底色间隙。** 用底色的 **2px 间隙**分隔相互接触的图元 —— 堆叠条形的每个分段、每根相邻（接触）的条形，间隙宽度一致。整个堆叠中保持同一宽度；相邻元素之所以能被区分，靠的是间隙，而不是绕着它们画的描边。
- **Surface ring.** Dots and end-markers carry a **2px ring in the surface color**,
  so they stay legible where they cross a line or overlap each other. The ring is part
  of the mark's hover/hit target, not just spacing - see `interaction.md` (small dots
  are easy to under-size for hover).
  **底色环。** 圆点与端点标记带有一圈 **2px 的底色描边环**，使它们在与线条交叉或相互重叠时仍清晰可辨。该环属于图元悬停/命中目标的一部分，而不只是间距 —— 参见 `interaction.md`（小圆点很容易被做得过小而难以悬停）。

Never draw a border around a mark to separate it. The gap and the ring are the
mechanism; a stroke adds data-weight ink that isn't data.

绝不要用给图元加边框的方式来分隔。间隙和底色环才是机制；描边会增加并非数据本身的"数据级重量"墨迹。

## Labels & legend / 标签与图例

A **legend is always present for two or more series** - the dependable identity
channel; never make the reader rely on color-matching alone. Direct labels then ride
the marks to *supplement* it. **A single series needs no legend box**: there is only
one color, so the chart's title or subtitle already says what is plotted. A box with
one swatch restates the title and costs space.

**两个及以上的系列必须始终有图例** —— 这是最可靠的身份识别通道；绝不让读者仅靠颜色比对。直接标签附着在图元上，起*补充*作用。**单一系列不需要图例框**：只有一个颜色，图表的标题或副标题已经说明了所绘内容。只有一个色块的图例框是在重复标题并浪费空间。

- **Label selectively - never a number on every point.** A value beside every dot or
  segment is chaos and goes unread. Label the endpoint, the extreme, or the one series
  the story is about; let the axis, the legend, and the tooltip/table carry the rest.
  Direct labels work *because* they are sparing - flood the chart and they stop working.
  **有选择地标注 —— 绝不在每个点上标数字。** 每个圆点或分段旁边都标数值只会造成混乱且无人会读。只标注端点、极值，或故事聚焦的那一个系列；其余交给坐标轴、图例和工具提示/数据表。直接标签之所以有效，*恰恰因为*它们克制 —— 满屏都是就失效了。
- **Direct labels before gridlines; gridlines before a second axis.**
  **优先直接标签，其次网格线；优先网格线，其次第二坐标轴。**
- **A label that won't fit doesn't get clipped - measure first.** Only place a label
  *inside* a bar or stacked segment when the rendered text fits with comfortable
  padding on both sides. If it doesn't fit: for a whole bar/column, move the label
  outside the bar end (or to the tooltip if there's no room outside either); for an
  *interior* stacked segment (which has no free end),
  skip the inline label and let the legend + tooltip carry it. Either way the value
  stays in the table view, so nothing is gated. Never use `overflow: hidden` on the
  segment to "solve" it - that crops the first/last characters and is worse than no
  label. Text never overflows or is clipped by its own mark.
  **放不下的标签不做裁切 —— 先测量。** 只有当渲染后的文字能在两侧留出舒适内边距时，才把标签放在条形或堆叠分段*内部*。如果放不下：对整根条形/柱形，把标签移到条形端部之外（若内外都放不下则放入工具提示）；对*内部*堆叠分段（没有可用的自由端），放弃内联标签，交给图例 + 工具提示。无论哪种方式，数值都保留在数据表视图中，信息不会丢失。绝不要用分段上的 `overflow: hidden` 来"解决"问题 —— 那会裁掉首尾字符，比没有标签更糟。文字绝不溢出或被自己的图元裁切。
- Bars -> value at the tip. Columns -> value on the cap. Lines -> value at the end.
  条形图 -> 数值放在条形末端。柱形图 -> 数值放在柱顶。折线 -> 数值放在线尾。
- Y-axis ticks: round to clean numbers (0 / 1,000 / 2,000), thousands-comma'd; they
  carry the values you didn't directly label, so keep them unless every value is labeled.
  Y 轴刻度：取整为干净的数字（0 / 1,000 / 2,000），千位加逗号；它们承载着你没有直接标注的数值，因此应保留，除非每个值都已标注。
- **Text never wears the data color.** Marks - bars, lines, dots, area fills - carry
  the series color; labels, values, legends, and axis text use **text tokens**
  (primary / secondary / muted). A light categorical hue (yellow, aqua) is illegible
  as text on the surface. Identity comes from the colored mark *beside* the text - a
  dot, a short line-key, a swatch - never from coloring the text itself. A label set
  *inside* a colored fill (a stacked segment, a map tile) is the one exception: pick
  white or ink by the fill's luminance so it always clears contrast.
  **文字永远不用数据色。** 图元 —— 条形、线条、圆点、面积填充 —— 承载系列色；标签、数值、图例和坐标轴文字使用**文字色 token**（primary / secondary / muted）。浅的分类色相（黄色、水色）作为底色上的文字无法阅读。身份识别来自文字*旁边*的彩色图元 —— 一个圆点、一小段线钥、一个色块 —— 绝不来自给文字本身上色。唯一例外是放在彩色填充*内部*的标签（堆叠分段、地图瓦片）：按填充色的亮度选择白色或墨色，确保始终满足对比度。
- **When end-labels collide, don't stack them.** Direct end-labels work when series
  separate at the right edge. When lines converge, nudging labels apart vertically
  detaches them from their lines and reads as noise - instead use **leader lines**
  (a thin connector from label to line-end), facet into **small multiples**, or fall
  back to the legend + tooltip. Past ~4 converging series, small multiples is usually right.
  **当末端标签相互碰撞时，不要上下堆叠。** 直接末端标签只在各系列在右缘彼此分开时有效。当线条汇聚时，垂直挪开标签会使标签脱离其线条，读起来像噪音 —— 应改用**引导线**（从标签到线尾的细连接线）、切分为**小倍数图**，或退回图例 + 工具提示。汇聚系列超过约 4 个时，小倍数图通常是正确选择。

## Figures - when the form is a number / 数字型图示 —— 当形态本身是一个数字

- **Stat tile** contract: `label` (sentence case, no trailing colon) · `value` (Sans
  semibold, auto-compact: 1,284 / 12.9K / $4.2M) · `delta` (optional; signed,
  vs a named period; color = direction × whether up is good) · `trend` (optional;
  12-point sparkline in the de-emphasis hue, current period in the accent).
  **统计瓦片（stat tile）** 契约：`label`（句首大写格式，结尾无冒号）· `value`（Sans 半粗体，自动紧凑：1,284 / 12.9K / $4.2M）· `delta`（可选；带正负号，对比某个具名时期；颜色 = 方向 × "上涨是否为好"）· `trend`（可选；以弱化色调绘制的 12 点迷你走势线，当前周期用强调色）。
- **Meter:** the fill carries severity (accent -> warning -> danger); the unfilled
  track is a **lighter step of the same ramp** (blue-on-blue, etc.) so state reads
  across the whole bar.
  **计量条（meter）：** 填充色承载严重程度（强调色 -> 警告 -> 危险）；未填充的轨道是**同一色阶的更浅一档**（蓝上蓝等），使状态在整个条上可读。
- **Hero figure.** The single number a dashboard leads with, >=48px, in the same
  sans as everything else (never a display or serif face - it reads as off-brand
  decoration). Exactly one per view.
  **主数字（hero figure）。** 仪表盘以之领衔的那个单一数字，字号 >=48px，与其他文字使用同一无衬线字体（绝不用展示型或衬线字体 —— 那会显得像偏离品牌的装饰）。每个视图恰好一个。
- **Proportional figures for big numbers; tabular only in columns.** A large
  standalone value (hero figure, stat-tile value) uses the font's default
  proportional figures - `tabular-nums` gives every digit the width of a `0`, so a
  number like `121` looks loose at display sizes. Reserve
  `font-variant-numeric: tabular-nums` for columns of numbers that must align
  vertically (table rows, axis ticks).
  **大数字用比例数字；等宽数字只用于列。** 大型独立数值（主数字、统计瓦片的数值）使用字体默认的比例数字 —— `tabular-nums` 会让每个数字都有 `0` 的宽度，使 `121` 这类数字在大字号下显得松散。`font-variant-numeric: tabular-nums` 保留给必须纵向对齐的数字列（表格行、坐标轴刻度）。

## Texture - the backup channel (opt-in) / 纹理 —— 备用通道（需主动启用）

Where hue fails - full-severity CVD, grayscale print, `forced-colors` - texture
carries identity. One directional hand-drawn fill, used at **45° and its 135° mirror
only** (never horizontal/vertical - those read as gridlines/bars). Inked tone-on-tone
(a step from the fill's own ramp), equal loudness across slots. On value scales the
texture is *ordered* (rotation steps with magnitude; arm angle carries the diverging
sign) so it never misstates the value. Triggered by an accessibility setting, print,
or `forced-colors` - never on by default. (See `palette.md`.)

在色相失效的场合 —— 重度色觉障碍（CVD）、灰度打印、`forced-colors` —— 由纹理承载身份识别。使用一种方向性手绘填充，且**只用 45° 及其 135° 镜像**（绝不用水平/垂直 —— 那会被看成网格线/条形）。以同色调描线（取自填充色自身色阶的一档），各槽位浓淡一致。在数值色标上，纹理是*有序的*（旋转级数随数值大小变化；臂角承载发散符号），因此绝不会误述数值。由无障碍设置、打印或 `forced-colors` 触发 —— 绝不默认开启。（参见 `palette.md`。）

【评论】这份规格把"留白"（间隙与底色环）而非描边作为唯一的分隔机制，是典型的设计系统约束式提示词：用少数固定数值换取跨图表的一致性。
