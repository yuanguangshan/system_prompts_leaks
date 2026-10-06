<!-- BILINGUAL-EN-ZH -->
# Interaction - tooltips & filters / 交互 - 工具提示与筛选器

An HTML chart is interactive by default - the hover layer is part of the deliverable,
not an upgrade. Omitting it is the exception (a bare stat tile), never the default.
Design it with the same care as the static render.

HTML 图表默认即是交互式的——悬停层是交付物的一部分，而非可选升级。省略它是例外（如纯数字统计磁贴），绝不是默认做法。请以与静态渲染同等的用心来设计它。

## Tooltips & hover / 工具提示与悬停

Tooltips **enhance, they never gate**: every value a tooltip shows is also reachable
without it, through direct labels or the table view. Same details on keyboard focus
as on hover.

工具提示**只做增强，绝不做门槛**：工具提示展示的每个数值，无需它也能通过直接标注或表格视图获取。键盘聚焦时展示的细节与悬停时相同。

- **The crosshair finds the X.** A vertical hairline tracks the pointer and snaps to
  the nearest data position. Readers aim at a date, never at a 2px line.
  十字线负责定位 X。一条垂直细线跟随指针并吸附到最近的数据位置。读者瞄准的是一个日期，而不是一条 2px 的线。
- **On bars and cells, the mark is the hit target.** No crosshair - each bar, segment,
  dot, or heat-cell carries its own `pointermove`/`focus` tooltip showing category and
  value, and the hovered mark lifts (slight lighten or outline) so the reader sees it respond.
  在条形与单元格上，图形本身就是命中目标。不使用十字线——每个条形、分段、圆点或热力单元格都自带 `pointermove`/`focus` 工具提示，显示类别与数值，且被悬停的图形会抬升（轻微变亮或加轮廓），让读者看到它的响应。
- **One tooltip, every series.** The readout lists every series at that X - the
  pointer never has to land on a line or a fill to get a value.
  一个工具提示，覆盖全部系列。读数区列出该 X 位置上的每个系列——指针不必落在某条线或某个填充区域上也能取得数值。
- **Labels are untrusted data - use `textContent`.** Series and category names
  often come from CSV headers, tool output, or API responses. Insert them into
  tooltip/legend/table DOM with `textContent` or `createTextNode`, never via
  `innerHTML` string concatenation.
  标签是不可信数据——请使用 `textContent`。系列名与类别名通常来自 CSV 表头、工具输出或 API 响应。请用 `textContent` 或 `createTextNode` 将它们插入工具提示/图例/表格的 DOM，绝不要通过 `innerHTML` 字符串拼接。
  【评论】把标签一律当作不可信数据并通过安全 API 插入 DOM，是典型的 XSS 防御要求，也侧面说明图表数据源可能包含不可控内容。
- **Values lead, labels follow.** In the tooltip the value is the Strong,
  high-contrast element and the series name is secondary - the legend's hierarchy
  inverted, because here the reader has the series and wants the number.
  数值在前，标签居后。在工具提示中，数值是加粗、高对比度的主角，系列名退居其次——恰好是图例层级的倒置，因为此刻读者已知道系列，想要的是数字。
- **Line keys, not boxes.** Tooltip rows key their series with a short stroke of the
  series color; at tooltip density a filled box is data-weight ink doing a label's
  job. (Legends still mirror the mark: rect for bars/areas, line for lines.)
  线形色标，而非色块。工具提示的各行用一段该系列颜色的短笔画作色标；在工具提示的密度下，填充色块是"用数据级墨量去做标签的活"。（图例仍与图形保持一致：条形/面积用矩形，折线用线段。）
- **The hit target is bigger than the mark.** A mark's hover/focus area includes its
  2px surface gap and then some - never only the painted pixels. An 8px scatter dot is a
  pinpoint nobody hits reliably; give each point a transparent hit area of at least
  **24px**, or - for dense scatter - a nearest-point / Voronoi layer so the pointer only
  has to be *closest*, not dead-center. (The crosshair already does this for the X on
  line and bar charts; scatter and bubble need the per-point version.)
  命中目标大于图形本身。图形的悬停/聚焦区域包含其 2px 的表面间隙并再向外扩展——绝不仅限于绘制出的像素。8px 的散点是一个没人能稳定点中的针尖；请给每个点一个至少 **24px** 的透明命中区域，或者——对密集散点图——加一层最近点 / Voronoi 层，使指针只需*最接近*即可，无需正中。(折线图与条形图的 X 轴已由十字线实现了这一点；散点图与气泡图需要逐点版本。)
- **A value pushed off its mark lives in the tooltip.** When a label won't fit inside a
  small bar (see `marks-and-anatomy.md`), that bar's hit area carries the value on hover
  and focus - the tooltip is its overflow home, and the table view keeps it reachable
  without hovering at all.
  放不进图形的数值住在工具提示里。当标签放不进一个较小的条形时（见 `marks-and-anatomy.md`），该条形的命中区域在悬停与聚焦时都会携带该数值——工具提示是它的溢出之家，而表格视图则保证完全无需悬停也能取到它。

## Filters & time ranges / 筛选器与时间范围

Every monitoring dashboard needs the same controls. These are **standard UI, not
chart marks** - build them with ordinary HTML form controls styled to match the
chart chrome. Dataviz only adds composition rules:

每个监控仪表盘都需要同样的控件。它们是**标准 UI，而非图表图形**——用普通 HTML 表单控件构建，样式与图表外观相匹配。Dataviz 只补充构图规则：

- **One row, above the charts.** Filters sit in a single left-aligned row above the
  content they scope - never inside a chart card, never per-chart. If one chart needs
  its own range, it's a different dashboard.
  单行，置于图表上方。筛选器以单行左对齐的形式置于其所辖内容上方——绝不在图表卡片内部，也绝不逐图设置。如果某个图表需要自己的时间范围，那它就属于另一个仪表盘。
- **Date range first.** It's the filter every reader reaches for; presets (today,
  last 7 / 30 / 90 days) before a custom range.
  日期范围优先。它是每个读者都会首先使用的筛选器；预设项（今天、最近 7 / 30 / 90 天）放在自定义范围之前。
- **Filters scope everything below them.** Every chart, stat, and table re-renders
  against the same slice, so the numbers always agree.
  筛选器统辖其下的一切。每个图表、统计数字和表格都基于同一切片重新渲染，因此数字之间总是相互一致。
- **Refetch keeps the frame.** While data reloads, charts hold their previous render
  at reduced opacity - no skeleton, no layout jump, no flash.
  重新取数时保持画面。数据重新加载期间，图表以降低的不透明度保留上一次的渲染——不用骨架屏、不发生布局跳动、也不闪烁。

A good date picker lists presets as rows (nobody fights a calendar grid for "last 30
days"), marks selection with a 16px bold check, keeps hover a ghost wash so it never
competes with selection, and tucks the custom range behind a hairline in the footer.
(See `palette.md` for the reference spec.)

一个好的日期选择器把预设项列成行（没有谁会为了"最近 30 天"去折腾日历网格），用 16px 的加粗对勾标记选中项，让悬停只是一层极淡的底色从而绝不与选中态争夺注意力，并把自定义范围收进页脚的一条细线之后。（参考规范见 `palette.md`。）
