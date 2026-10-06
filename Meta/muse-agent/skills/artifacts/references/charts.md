---
description: Building a chart on any surface: where its numbers come from, how data maps to marks, and the technique each surface needs.
---
<!-- BILINGUAL-EN-ZH -->

# Charts / 图表

Read this before writing any chart, on any surface. Decide one thing first: is
the chart a rendered image placed on a fixed page, or live elements in a page
that reflows? That choice picks your technique section below. The data and
encoding rules apply everywhere.

在为任何承载面编写任何图表之前，先阅读本文。首先要决定一件事：图表是放置在固定页面上的渲染图片，还是页面中会随回流（reflow）变化的活性元素？这一选择决定你使用下文哪个技术章节。数据规则与编码规则适用于所有场合。

## Where the numbers come from / 数字从何而来

- If the data already exists (a file the user gave you, a document you just
  produced, a spreadsheet in the workspace), read it and build from it. Never
  regenerate numbers you already produced: two generations of "the same" data
  will not match, and the user cannot tell which is right.
  若数据已存在（用户给你的文件、你刚生成的文档、工作区中的电子表格），读取它并据此构建。绝不重新生成你已产出过的数字：两代"相同"的数据不会一致，用户也无法分辨哪个是对的。
- Never claim two artifacts show the same data unless you read one and built
  the other from it.
  除非你读取了其中之一并根据它构建了另一个，否则绝不声称两个工件展示的是同一份数据。
- Use only real, verifiable data from the source. Never fabricate figures,
  dates, citations, or source names, and never cite a chart you generated as a
  source.
  只使用来自数据源的真实、可验证的数据。绝不编造数字、日期、引用或数据源名称，也绝不把自己生成的图表当作数据源来引用。
- If a source's summary disagrees with its own rows, recompute from the rows
  and say so on the artifact.
  若数据源的汇总与其自身各行不一致，从行数据重新计算，并在工件上注明这一点。
- No randomness in a shipped artifact: `Math.random()` means no two loads
  agree. If you must synthesize sample data, generate it once, persist it to a
  file, and have every artifact read that file.
  交付的工件中不允许存在随机性：`Math.random()` 意味着两次加载的结果互不相同。若必须合成示例数据，只生成一次，持久化到一个文件，并让每个工件都读取该文件。

**Every displayed number is computed from the plotted data.** That covers
captions, headings, stat tiles, annotations, and summary lines; superlatives
("peak", "highest", "fastest-growing"), which come from an argmax over the
series, never from memory; comparisons and counts stated in prose ("up 40%",
"eight above the threshold"); and deltas, whose two endpoints must both exist
in the dataset. A literal typed into prose drifts the moment the data changes.
Never floor or cap a displayed statistic: `max(22, computed)` guarantees a
number, not a fact. Never claim a reconciliation you did not compute.

**每一个显示出来的数字都必须由所绘制的数据计算得出。** 这涵盖图注、标题、统计卡片、注释和汇总行；涵盖最高级表述（"峰值"、"最高"、"增长最快"）——它们来自对序列取 argmax，绝不出自记忆；涵盖行文中给出的比较与计数（"上涨 40%"、"超出阈值八个"）；也涵盖差值——其两个端点必须都存在于数据集中。写进行文的字面量在数据一变时就发生漂移。绝不对显示的统计量做下限或上限截断：`max(22, computed)` 保证的是一个数字，而不是一个事实。绝不声称做了你没有计算过的核对。

【评论】该节把"显示的每个数字必须能由所绘数据复现"立为硬性约束，并点名 `Math.random()` 与写死字面量两类典型出错源，属于数据可信性方面的防护设计。

Totals and shares: compute a total from the parts shown beside it, never from
a different source than the parts. Percentages share one denominator and the
parts must account for the whole; if they sum to 90%, a part is missing or the
total is wrong, and the artifact says which. State a count only if it matches
what you drew. Name the remainder; an unlabeled slice is unreadable.

总计与占比：总计要从其旁边展示的各部分计算得出，绝不用与各部分不同的来源。百分比共用同一个分母，且各部分必须覆盖整体；若合计只有 90%，说明缺了一个部分或总计有误，工件上要写明是哪种情况。只有与你绘制的内容相符时才能给出计数。要为剩余部分命名；没有标签的扇区是无法解读的。

## How data maps to marks / 数据如何映射为图形标记

These rules apply on every surface. Where you hand-author the chart they are
the arithmetic you are about to write; where a library computes them, check
its output, because library defaults happily draw a truncated baseline or an
axis that clips your data.

这些规则适用于每个承载面。在你手工编写图表时，它们就是你要亲手写下的算术；在由库计算时，要检查其输出，因为库的默认设置会很自然地画出被截断的基线或裁掉数据的坐标轴。

Axes:

坐标轴：

- Derive the domain from the data; never hardcode a limit. A literal max will
  eventually be smaller than the data, and points past it clip without
  warning. Verify every plotted value falls inside whatever you set.
  从数据推导定义域；绝不硬编码界限。写死的最大值终有一天会小于数据，超出的点会在无警告的情况下被裁掉。要核验每个绘制值都落在你所设的范围内。
- Bar and area charts start at zero: length encodes magnitude, and a truncated
  baseline misstates it. If the variation only shows on a truncated scale, use
  a line chart. When truncation is unavoidable, disclose it on the chart
  itself (a visible axis break), not in prose beside it, which does not travel
  with a screenshot.
  条形图与面积图从零开始：长度编码数量大小，截断的基线会错误陈述它。若差异只有在截断刻度上才看得出来，就改用折线图。当截断不可避免时，在图表自身上披露（可见的断轴标记），而不是在旁边的文字里说明——文字不会跟着截图走。
  【评论】"条形图从零开始"针对的是数据可视化中经典的截断基线误导：条形长度编码数量，截断基线会在视觉上夸大差异。
- Match the range to the question: an axis so wide that meaningfully different
  bars render identical, or so narrow that noise reads as a mountain range,
  both misreport.
  让范围与问题相称：坐标轴过宽，会使本有明显差异的条形渲染得一模一样；过窄，则会让噪声看起来像连绵山脉——两者都是误报。
- One labeled scale per encoding. Two series in different units need two
  labeled axes or two charts; rescaling one series by a constant so it can
  share the axis is the same defect with extra steps.
  每种编码只配一个带标签的刻度。单位不同的两个序列需要两条带标签的坐标轴或两张图；用一个常数缩放某个序列以共用坐标轴，是同一个缺陷又多绕了几步。
- Give every axis readable ticks; value labels alone are not a scale.
  给每条坐标轴可读的刻度；只有数值标签不算刻度。

Labels:

标签：

- Never truncate label text in code: a sliced label reads as data ("Utiliti").
  Rotate, wrap, shrink the type, or show fewer ticks, and measure the longest
  label against the space before choosing.
  绝不在代码中截断标签文本：被切掉的标签会被读成数据（"Utiliti"）。可以旋转、换行、缩小字号或减少刻度数量，选择前先用最长标签与可用空间比一比。
- Drop value labels uniformly or keep them uniformly; clipping only the
  longest one hides the largest value.
  数值标签要么统一去掉，要么统一保留；只裁掉最长的一个，恰好隐藏了最大的值。
- Check text against its container (card, plot area, donut hole), not just the
  viewport, and keep labels from overlapping each other or the marks.
  对照文本的容器（卡片、绘图区、环形图内孔）检查文本，而不只是视口，并避免标签彼此重叠或与图形标记重叠。

Legends and annotations:

图例与注释：

- Every plotted series appears in the legend, with each swatch built from the
  same value the renderer uses; two series must never resolve to one color.
  每个绘制的序列都出现在图例中，每个色样都由渲染器实际使用的同一个值生成；两个序列绝不能解析为同一颜色。
- A reference line or annotation sits at its statistic: a line labeled
  "median" is at the median.
  参考线或注释要位于其统计量处：标注"median"的线就在中位数的位置。
- Decorative marks are proportional or they are not marks.
  装饰性标记要么成比例，要么就不配称为标记。

Data shape:

数据形态：

- Missing data renders as a gap. Never interpolate across a hole; a reader
  cannot tell absent from zero.
  缺失数据渲染为空缺。绝不在空洞处插值；读者无法区分"缺席"与"零"。
- Unequal intervals need a real scale: 2021, 2023, and 2026 as three evenly
  spaced categories tell the reader the gaps were equal.
  不相等的间隔需要真实的刻度：把 2021、2023 和 2026 画成三个等距类别，等于告诉读者这些年份的间隔是相等的。
- Label a partial period as partial, or it reads as a collapse in volume.
  不完整的时期要标注为不完整，否则会被读作数量骤降。
- A part-to-whole shows every part; a capped slice list still names and counts
  the remainder.
  部分对整体图要展示每个部分；被截断的扇区列表仍要指名剩余部分并给出其数量。
- Never clamp values to the top of a range: clamping to 100% makes 108% and
  166% identical.
  绝不把数值钳制到范围顶端：钳制到 100% 会让 108% 和 166% 变得毫无区别。

## Technique by surface / 按承载面区分的技术

### Slide deck / 幻灯片

- Render the chart as an image from real data with deterministic Python
  (matplotlib). Never fake one with CSS shapes, gradients, styled divs, inline
  SVG, or Mermaid, and never use `media.generate_image` for a chart: generated
  imagery is for concepts, and charts carry the data.
  用确定性的 Python（matplotlib）从真实数据把图表渲染为图片。绝不用 CSS 形状、渐变、带样式的 div、内联 SVG 或 Mermaid 伪造图表，也绝不对图表使用 `media.generate_image`：生成式图像用于概念表达，而图表承载数据。
- Color it from the deck: pass `--slide-primary`, `--slide-accent`,
  `--slide-fg`, and `--slide-muted` into the chart script.
  配色取自幻灯片：把 `--slide-primary`、`--slide-accent`、`--slide-fg` 与 `--slide-muted` 传给图表脚本。
- Place it with `object-fit: contain`, never `cover`, which crops the axes and
  labels. This overrides the usual cell `cover` default.
  用 `object-fit: contain` 放置，绝不用 `cover`——后者会裁掉坐标轴和标签。这条规则覆盖通常的单元格 `cover` 默认值。
- The library computes scales, ticks, and legend, but check them against the
  encoding rules above; matplotlib will happily truncate a baseline.
  库会计算刻度、刻度值和图例，但要对照上述编码规则检查它们；matplotlib 会毫不犹豫地截断基线。
- Slide layout and the one-fact-one-home rule stay in `slides/authoring.md`.
  幻灯片版式与"一个事实一个归属"规则见 `slides/authoring.md`。

### PDF or printed document / PDF 或印刷文档

- Build charts with a Python script (matplotlib, plotly, or seaborn). Never
  use `media.generate_image` or any other media generation tool for a data
  visualization.
  用 Python 脚本（matplotlib、plotly 或 seaborn）构建图表。绝不对数据可视化使用 `media.generate_image` 或任何其他媒体生成工具。
- Page placement (`object-fit`, `max-height`, the `cover` trap that crops axes
  while validation passes) is owned by the pdf skill's `workflow.md`. If a chart renders too small to read, raise its `max-height`
  or drop it.
  页面摆放（`object-fit`、`max-height`，以及"校验通过却裁掉坐标轴"的 `cover` 陷阱）由 pdf 技能的 `workflow.md` 负责。若图表渲染得小到无法阅读，调高它的 `max-height` 或干脆去掉它。

### Self-contained page with no build step / 无构建步骤的自包含页面

Covers `runtime: static` and any live page built without a bundler.

覆盖 `runtime: static` 以及任何不经打包器构建的活性页面。

- Hand-author the chart as inline SVG rather than loading a charting library
  from a CDN at read time; that keeps the chart's correctness in the file you
  write instead of in a dependency that must resolve before anything renders.
  以手写内联 SVG 的方式编写图表，而不是在读取时从 CDN 加载图表库；这样图表的正确性就保存在你写的文件里，而不是在一个必须先解析成功才能渲染任何东西的依赖里。
- No library is computing scales, ticks, labels, or legend, so every rule in
  "How data maps to marks" is arithmetic you write. Settle the domain, the
  baseline, the label strategy, the legend, and the annotation positions
  before you draw.
  没有库替你计算刻度、刻度值、标签或图例，所以"数据如何映射为图形标记"中的每条规则都是你要亲手写的算术。落笔之前先定好定义域、基线、标签策略、图例与注释位置。
- Inline the data once as a literal array; that array is the single source for
  the marks, the labels, the legend, and any total you display.
  把数据一次性以内联字面量数组写入；该数组是图形标记、标签、图例以及你展示的任何总计的唯一来源。
- The page scrolls, so let cards size to content (`align-items: start`); never
  stretch a chart card to match a taller sibling. Reserve real height for the
  plot: squeezed into a wide, short box, the series flattens.
  页面会滚动，所以让卡片按内容决定尺寸（`align-items: start`）；绝不把图表卡片拉伸去匹配更高的兄弟元素。为绘图区保留真实的高度：被塞进又宽又矮的盒子里，序列会被压平。
- Position tooltips in the coordinate space you apply them in: values computed
  in an SVG `viewBox` and set as CSS pixels land wrong once the SVG scales,
  and `overflow: hidden` on the container clips tooltips at the edges, exactly
  where they appear.
  提示框要定位在它所作用的坐标空间中：在 SVG `viewBox` 里算出却按 CSS 像素设置的值，一旦 SVG 缩放就会落错位置；容器上的 `overflow: hidden` 会在边缘裁掉提示框，而边缘恰恰是它们出现的地方。
- This section governs only the chart. The page's other assets (fonts, images,
  allowed CDN tags, maps) follow the asset rules already in your instructions;
  never strip an allowed asset to satisfy something you read here.
  本节只管辖图表。页面的其他资产（字体、图片、允许的 CDN 标签、地图）遵循你指令中已有的资产规则；绝不为满足本节内容而移除某个被允许的资产。

### Web artifact on the TypeScript runtime / TypeScript 运行时上的 Web 工件

- Bind charts to the data file the source document already reads. A web
  artifact usually follows a document, and a hand-typed copy of its dataset is
  a second dataset: it will differ, and both artifacts will claim to be the
  same numbers. If no source file exists yet, produce one first and have both
  read it.
  把图表绑定到源文档已在读取的数据文件上。Web 工件通常伴随一份文档而来，而手工键入的数据集副本就成了第二份数据集：两者必然有出入，但两个工件却都声称是同一批数字。若尚无源文件，先产出一份，让两者都读取它。
- Build charts as real DOM with Recharts. No matplotlib PNGs (they cannot
  reflow, are invisible to assistive tech, and blur on scaling), no styled-div
  or Mermaid fakes, no canvas libraries (Chart.js, uPlot, ECharts' default
  renderer: the chart becomes a bitmap the page cannot inspect or style), and
  no D3/visx unless the chart is genuinely custom, since they hand back the
  scale and axis math this reference exists to avoid.
  用 Recharts 以真实 DOM 构建图表。不要 matplotlib PNG（无法回流、对辅助技术不可见、缩放时发糊），不要带样式 div 或 Mermaid 的仿制品，不要 canvas 类库（Chart.js、uPlot、ECharts 的默认渲染器：图表会变成页面无法检查或设置样式的位图），也不要 D3/visx——除非图表确实是定制需求，因为它们会把刻度与坐标轴的计算交还给你，而本参考文档的存在正是为了避免这种计算。
- Check the artifact's own `package.json` first: a new scaffold pins
  `"recharts": "2.15.3"`, an older artifact does not, and importing it without
  the dependency fails the build. Add the scaffold's exact pin, not a range,
  then run `bun install` before building: auto-install is off, so the entry
  alone does not put the module on disk.
  先检查工件自己的 `package.json`：新脚手架固定了 `"recharts": "2.15.3"` 版本，旧工件则没有，而在缺少该依赖时导入它会导致构建失败。添加脚手架的精确固定版本，而非版本区间，然后在构建前运行 `bun install`：自动安装是关闭的，所以仅有依赖声明并不会把模块装到磁盘上。
- Recharts derives scales, legend entries, and tooltip positioning from the
  series, the three things most often shipped wrong. Two things it does not
  do: it auto-scales, so set a zero domain yourself for bars and areas, and it
  will not stop you plotting two units against one axis.
  Recharts 从序列推导刻度、图例项和提示框定位——这三样恰是最常出错的地方。它不做两件事：它会自动缩放，所以条形图和面积图的零点定义域要你自己设定；它也不会阻止你把两种单位画到同一条轴上。
- Let cards size to content in a scrolling grid (`align-items: start`);
  stretching belongs on fixed slides. Verify text fits at the widths the
  artifact really renders, including narrow ones; a horizontally scrolling
  table hides its right-hand columns, usually the totals.
  在滚动网格中让卡片按内容决定尺寸（`align-items: start`）；拉伸只属于固定幻灯片。在工件实际渲染的各个宽度（包括窄宽度）下核验文本是否放得下；横向滚动的表格会藏起右侧的列，而那通常正是总计。
- Every control described anywhere in the page or your reply re-renders every
  panel it claims to affect. Keep control state and displayed state in one
  place, and make exports reflect the current view.
  页面或你回复中描述的每个控件，都要真正重新渲染它声称影响的每个面板。把控件状态与显示状态保存在一处，并让导出内容反映当前视图。

## Before you hand it over / 交付之前

Re-derive each headline figure from the source and compare it to what the
artifact displays: the total, the percentages, the named extreme, any growth
figure, any count you state. A number you cannot reproduce from the data is
wrong, and the fix is the number, not the caption. Then read the artifact as
its reader will, starting from first load with no interaction: a chart that
only appears after a click has not been delivered.

从数据源重新推导每个头条数字，并与工件所显示的内容比对：总计、百分比、点名的极值、任何增长数字、你陈述的任何计数。一个无法从数据中复现的数字就是错的，要修正的是数字，而不是图注。然后以读者的视角阅读工件，从首次加载、无任何交互开始：需要点击之后才出现的图表，等于没有交付。
