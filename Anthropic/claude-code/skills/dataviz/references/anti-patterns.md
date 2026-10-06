<!-- BILINGUAL-EN-ZH -->
# Anti-patterns - what goes wrong / 反模式 —— 哪些做法会出错

Check every chart against this list. If your output matches an entry, it is wrong -
fix it before shipping. These are real failure modes, each caught in shipping
dashboards.

用这份清单检查每一张图表。如果你的输出命中其中任何一条，那就是错的 —— 交付前先修好。
这些都是真实的失败模式，每一条都是在已上线的仪表盘中抓到过的。

## Color & encoding / 颜色与编码

**Bad: Dual-axis charts (two y-scales on one plot).**
Why it misleads: the alignment of the two scales is arbitrary, so the chart invents a
correlation that isn't in the data. Real example: an "Adoption" chart plotting Users
(0-30k) against Sessions (0-800k) - a reviewer flagged it as looking "hallucinated."
Good: Do instead: two charts, small multiples, or index both series to a common base
(=100 at t0) on **one** axis.

**差：双轴图表（同一绘图区上两个 y 轴刻度）。**
为何误导：两个刻度的对齐方式是任意的，因此图表编造了数据中并不存在的相关性。真实案例：一张"Adoption"图表把 Users（0-30k）与 Sessions（0-800k）画在一起 —— 评审者认为它看起来像"幻觉出来的"。
**好：** 改用：两张图表、小倍数图，或将两个系列都指数化到同一基点（t0 时 =100）画在**同一个**坐标轴上。

**Bad: Recolor-on-filter.** Assigning colors by current rank, so filtering out a series
repaints the survivors.
Why: a reader who learned "Acme is blue" is now misled.
Good: Color follows the entity, not its row number. Survivors keep their hue.

**差：过滤时重新着色。** 按当前排名分配颜色，导致过滤掉一个系列后，幸存系列被重新上色。
为何：已经记住"Acme 是蓝色"的读者会被误导。
**好：** 颜色跟随实体本身，而不是它的行号。幸存系列保留原色相。

**Bad: Cycling / generating hues past 8.** A 9th categorical color, generated or reused.
Why: indistinguishable from an existing slot under CVD; breaks the order check.
Good: Fold the tail into "Other," facet into small multiples, or use composite encoding.

**差：超过 8 个后循环/生成色相。** 第 9 个分类色，无论是新生成还是复用。
为何：在色觉障碍（CVD）下会与现有槽位无法区分；破坏顺序检查。
**好：** 把尾部归入"Other"，切分为小倍数图，或使用复合编码。

**Bad: Eyeballing colorblind-safety.** "These look different enough."
Good: Run `scripts/validate_palette.js`. Adjacent Delta E >= 8 (OKLab ×100), or 6-8 WITH secondary encoding.

**差：靠目测判断色盲安全性。** "这些看起来差别够大。"
**好：** 运行 `scripts/validate_palette.js`。相邻色 Delta E >= 8（OKLab ×100），或 6-8 但必须搭配次级编码。

**Bad: A value-ramp on nominal categories.** Coloring each bar darker-where-bigger
when the categories have no natural order (products, teams, endpoints).
Why: it double-encodes bar length as hue, burns the only free channel on
information the chart already shows, and fails the categorical checks by design
(a ramp spans the lightness band and drops below the chroma floor).
Good: One series -> one color (slot 1) for every bar. Ordered categories (funnel,
tiers, age bands) -> the ordinal ramp, validated with `--ordinal`.

**差：在名义类别上使用数值色阶。** 在没有自然顺序的类别（产品、团队、端点）上，把每根柱子按数值大小涂成深浅不一。
为何：它把条形长度用色相双重编码，把唯一的空闲通道消耗在图表已经表达的信息上，而且按设计就通不过分类色检查（色阶横跨明度带并跌破色度下限）。
**好：** 一个系列 -> 每根柱子用同一颜色（槽位 1）。有序类别（漏斗、层级、年龄段）-> 使用顺序色阶，并用 `--ordinal` 校验。

**Bad: Rainbow / non-neighbor sequential.** A multi-hue ramp for magnitude.
Good: One hue, light->dark. (Analogous neighbors or semantic heat are the only multi-hue
sequential exceptions, always with a scale legend.)

**差：彩虹色/非相邻色的顺序色标。** 用多色相色阶表示数值大小。
**好：** 单色相、由浅到深。（相邻类似色或语义化热度色是仅有的多色相顺序色标例外，且必须配刻度图例。）

**Bad: A hue at the diverging midpoint, or two cool hues as the two poles.**
Why: the midpoint must read as "nothing"; poles must read as opposite. blue<->aqua
fails this (both cool); blue<->red or blue<->orange succeed (warm/cool).
Good: Two hues that read as opposite + a neutral gray midpoint.

**差：发散色标的中间点用有色相的颜色，或两个冷色作为两端。**
为何：中点必须读作"没有值"；两端必须读作对立。blue<->aqua 不合格（都是冷色）；blue<->red 或 blue<->orange 合格（暖/冷对峙）。
**好：** 两个读起来对立的色相 + 中性的灰色中点。

**Bad: Status color used for a non-status series** (or a series color used for status).
Good: Status tokens only when the color *means* good/bad; categorical when it's identity.

**差：把状态色用于非状态的系列**（或把系列色用于状态）。
**好：** 只有当颜色*表示*好坏时才用状态 token；表示身份时用分类色。

## Form / 形态

**Bad: Eight categorical hues when the story is one number.** The most common way a
chart misses its point.
Good: Emphasis (highlight one, gray the rest), or a stat tile / hero number.

**差：故事只是单个数字，却用了八个分类色相。** 这是图表偏离主旨最常见的方式。
**好：** 强调（突出一个，其余置灰），或改用统计瓦片/主数字。

**Bad: A one-bar bar chart, or a 2-slice pie.**
Good: A stat tile. The number is the chart.

**差：只有一根柱子的条形图，或只有两块的饼图。**
**好：** 用统计瓦片。数字本身就是图表。

**Bad: A donut/pie for comparing close values.**
Good: A bar, or the numbers. Part-to-whole at a glance only, <= 6 segments.

**差：用环形图/饼图比较接近的数值。**
**好：** 用条形图，或直接给数字。部分-整体关系仅限一眼可辨的场景，分段 <= 6。

**Bad: More than ~7 color classes carrying meaning.**
Good: A table, or table + chart. Past ~7 bins, adjacent classes blur.

**差：超过约 7 个承载含义的颜色类别。**
**好：** 用表格，或表格 + 图表。超过约 7 个分箱后，相邻类别会难以分辨。

## Marks & chrome / 图元与外观

**Bad: Thick saturated blocks, heavy gridlines, no breathing room.** Reads loud, even
childish, at scale.
Good: Thin marks, hairline recessive grid/axes, generous padding. Saturated fills are
for small marks and accents, never large blocks.

**差：粗重的饱和色块、浓重的网格线、没有呼吸空间。** 放大看显得吵闹，甚至幼稚。
**好：** 细图元、发丝级的低调网格/坐标轴、充裕的留白。饱和填充只用于小图元和强调点，绝不做大色块。

**Bad: Dashed gridlines or axis rules.** Dashing adds visual noise and reads as
"projection" or "threshold" when it's just a grid.
Good: Gridlines and axes are solid hairlines, one shade off the surface.

**差：虚线网格线或轴线。** 虚线增加视觉噪音，而且明明只是网格，读起来却像"预测"或"阈值"。
**好：** 网格线和坐标轴用实线发丝线，比底色深一档。

**Bad: A number on every data point.** A value beside every dot or segment is chaos and goes unread.
Good: A legend is always present for >= 2 series; direct-label *selectively* (the endpoint, the extreme, the one series that matters) and let the axis + tooltip carry the rest.

**差：每个数据点都标数字。** 每个圆点或分段旁都标数值只会造成混乱且无人会读。
**好：** >= 2 个系列时始终有图例；*有选择地*直接标注（端点、极值、重要的那个系列），其余交给坐标轴 + 工具提示。

**Bad: A border drawn around marks to separate them.**
Good: A 2px surface gap between fills (stacked segments and adjacent bars alike) and a 2px surface ring (on overlapping markers).

**差：用给图元加边框的方式分隔。**
**好：** 填充之间留 2px 底色间隙（堆叠分段和相邻条形一视同仁），重叠的标记上加 2px 底色环。

**Bad: A label clipped by, or overflowing, a too-small bar or stacked segment** -
including `overflow: hidden` cropping the first/last characters of an in-segment label.
Good: Only render a label inside a mark when it fits with padding; otherwise move it
outside the bar end, or drop it to the tooltip/legend (the value stays in the table view).

**差：标签被过小的条形或堆叠分段裁切或溢出** —— 包括用 `overflow: hidden` 裁掉分段内标签的首尾字符。
**好：** 只有当标签连同内边距放得下时才渲染在图元内部；否则移到条形端部之外，或降级到工具提示/图例（数值仍保留在数据表视图中）。

**Bad: A chart container whose fixed height excludes the x-axis band** - the plot
fits, the axis labels don't, so the card gets a tiny nested vertical scroll.
Good: Size the container to include the axis labels (plot height + x-axis band),
or let the container grow with its content instead of fixing a height.

**差：图表容器的固定高度没有把 x 轴区域算进去** —— 绘图区放得下，坐标轴标签放不下，卡片里出现一个微小的嵌套纵向滚动条。
**好：** 容器尺寸应包含坐标轴标签（绘图高度 + x 轴区域），或让容器随内容增长而不是固定高度。

**Bad: A display or serif face on the hero figure.** It reads as off-brand decoration.
Good: The hero figure uses the same sans as everything else.

**差：主数字使用展示型或衬线字体。** 那会显得像偏离品牌的装饰。
**好：** 主数字与其他文字使用同一无衬线字体。

**Bad: `tabular-nums` on a large standalone number.** Equal-width digits make `121`
look loose at display sizes.
Good: Proportional figures on hero and stat-tile values; `tabular-nums` only where
numbers align vertically (table rows, axis ticks).

**差：在大型独立数字上使用 `tabular-nums`。** 等宽数字会让 `121` 在大字号下显得松散。
**好：** 主数字与统计瓦片数值使用比例数字；`tabular-nums` 只用于需要纵向对齐的数字（表格行、坐标轴刻度）。

**Bad: Texture on by default, or as decoration.** Dense angled fields are a vestibular
risk and read as noise on value scales.
Good: Texture is opt-in (a11y setting, print, forced-colors), 45°/135° only, ordered on
value scales.

**差：默认开启纹理，或把纹理当装饰。** 密集的斜纹对前庭系统是风险，且在数值色标上读起来像噪音。
**好：** 纹理需主动启用（无障碍设置、打印、forced-colors），只用 45°/135°，在数值色标上保持有序。

## Interaction & accessibility / 交互与无障碍

**Bad: A tooltip as the only way to read a value.**
Good: Tooltips enhance, never gate - every value is also reachable via direct labels or
the table view; keyboard focus shows the same as hover.

**差：把工具提示作为读取数值的唯一途径。**
**好：** 工具提示只做增强，绝不做门槛 —— 每个数值也应能通过直接标签或数据表视图获取；键盘聚焦显示的内容与悬停一致。

**Bad: Pinpoint hover targets - an 8px scatter dot you must land on dead-center.**
Good: The hit area includes the 2px gap and meets a ~24px minimum; dense scatter uses a nearest-point / Voronoi layer.

**差： pinpoint 级的悬停目标 —— 必须正中命中的 8px 散点。**
**好：** 命中区域包含 2px 间隙并达到约 24px 的最小尺寸；密集散点使用最近点/Voronoi 层。

**Bad: Per-chart filters, or filters inside a chart card.**
Good: One filter row above everything it scopes; all charts re-render against the same slice.

**差：每个图表各带过滤器，或在图表卡片内放过滤器。**
**好：** 在其作用范围之上放一行统一的过滤器；所有图表基于同一数据切片重渲染。

**Bad: Skeleton flash on refetch.**
Good: Hold the previous render at reduced opacity - no layout jump.

**差：重新请求时闪烁骨架屏。**
**好：** 以降低不透明度的方式保留上一次的渲染 —— 布局不跳动。

**Bad: No table view / color-only encoding on a continuous scale.**
Good: Every chart has a table-view twin (the WCAG-clean equivalent).

**差：没有数据表视图/在连续色标上只用颜色编码。**
**好：** 每张图表都有一个数据表视图孪生（即满足 WCAG 的等价形式）。
