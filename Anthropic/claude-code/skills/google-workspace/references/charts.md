<!-- BILINGUAL-EN-ZH -->
# Charts / 图表

Read the slide design rules in `references/slides.md` first; this file covers charts on slides. Assume each chart will be projected, printed or forwarded without you there to explain it, so everything the reader needs is on the chart.

先阅读 `references/slides.md` 中的幻灯片设计规则；本文件讲述幻灯片上的图表。请假定每张图表都会在被投影、打印或转发时没有你在场解释，因此读者需要的一切都应体现在图表上。

## Choose the display / 选择展示形式

| The reader needs to... | Use |
| :---- | :---- |
| See one or two numbers | No chart: plain words in a bullet, or a big number, the figure set large ("15%") with a line of smaller text under it saying what it is ("revenue growth last quarter"). Clearer than a one-bar chart or a two-slice pie |
| Compare magnitudes across categories | Bar chart: vertical columns by default, horizontal bars when the category names are long |
| See a trend, or anything else where the points are connected | Line: for time series and other connected data, never for categories; add error bars or a shaded range when the spread matters alongside an average |
| See how a few items moved between two points | Slope graph with no gridlines or value axis (say, education rate by region, 2015 against 2025), not a multi-series bar chart. Few lines, the one that moved most in the accent color; the start value alone on the left, the end value and the name once on the right |
| See one series against context | That series in the accent color, everything else gray |
| See parts of a whole | Stacked column: absolute to compare totals and their components, 100% to compare mix. Stacks get overwhelming fast; they show the gist, not precise comparisons of middle segments |
| Compare an ordered scale across categories (Likert responses) | 100% stacked horizontal bar: consistent baselines at the far left and far right |
| Follow a starting point, the changes, and the result | Waterfall: breaks a complicated calculation into pieces |
| See share on two dimensions at once (segment size and the mix within it) | Mekko |
| See a schedule | Gantt |
| See the relationship between two quantities | Scatterplot: mostly for samples and studies; fine in business when there is a real relationship to show |
| See data that normal charts don't accommodate | Shapes as illustration: a funnel, a grid of squares, an illustrative bar, when the data doesn't fit a chart and exact units don't matter, as long as it is obviously not to scale |

| 读者需要…… | 使用 |
| :---- | :---- |
| 看一两个数字 | 不用图表：项目符号中的平实文字，或一个大数字——数值放大显示（"15%"），下面配一行小字说明其含义（"上季度营收增长"）。比单柱柱状图或两块扇区的饼图更清晰 |
| 比较各类别的数量级 | 柱状图：默认用竖直柱形，类别名称较长时用水平条形 |
| 看趋势或任何点与点相连的数据 | 折线图：用于时间序列及其他相连数据，绝不用于类别；当离散度与平均值同样重要时，添加误差线或阴影区间 |
| 看少数项目在两个时点间的变化 | 斜率图，不带网格线和数值轴（例如 2015 年与 2025 年对比的各地区受教育率），而不是多系列柱状图。线条要少，变化最大的一条用强调色；左侧只标起始值，右侧标一次结束值和名称 |
| 在背景衬托下看某一个系列 | 该系列用强调色，其余全部用灰色 |
| 看整体的部分构成 | 堆积柱状图：比较总量及其构成用绝对值，比较构成比例用 100%。堆积层数一多就难以辨认；它展示的是大致格局，而非中间各分段的精确比较 |
| 比较各类别上的有序量表（Likert 应答） | 100% 堆积水平条形图：最左端和最右端基线保持一致 |
| 看清起点、变化过程和结果 | 瀑布图：把复杂的计算拆解为多个部分 |
| 同时看两个维度上的份额（分部规模及其内部构成） | Mekko 图 |
| 看日程安排 | 甘特图 |
| 看两个数量之间的关系 | 散点图：主要用于样本和研究数据；在商业场景中，当确实存在需要展示的关系时也可使用 |
| 看常规图表容纳不了的数据 | 形状示意：漏斗、方格阵、示意性条形——当数据不适合常规图表且精确单位不重要时使用，前提是明显不成比例 |

- Avoid unless the user explicitly asks: pie and donut charts (people judge angles badly; use a sorted bar chart instead); area charts (hard to read; the one exception is the square-area graph, where each square's area is a quantity, useful for very different magnitudes such as TAM, SAM and SOM); 3D; and secondary axes (the alignment of two scales is arbitrary and invents a correlation; label the units on each line, stack two charts on the same horizontal axis, or index both series to a common base).
  除非用户明确要求，否则避免使用：饼图和环形图（人对角度的判断很差；改用排序后的条形图）；面积图（难以阅读；唯一例外是方形面积图，即每个方块的面积代表一个数量，适用于 TAM、SAM 和 SOM 这类量级差异极大的场景）；3D 图表；以及次坐标轴（两个刻度的对齐方式是任意的，会凭空制造相关性；可在每条线上标注单位、在同一水平轴上叠放两个图表，或将两个系列都换算到共同基准）。
- One to three colored series read clearly; at four or more, label each series directly even if the deck's charts use a legend; five or six is the most one chart can hold. Past that, combine the smallest into "Other", or use small multiples (a grid of small identical charts on shared scales). Never fix "too many series" by adding colors.
  一到三个彩色系列清晰可读；达到四个或以上时，即使整套幻灯片的其他图表使用图例，也应直接为每个系列标注名称；五到六个是单张图表的上限。再多时，把最小的几个合并为 "Other"，或使用小倍数图（共享刻度的小型同构图表网格）。绝不要用增加颜色来解决"系列过多"的问题。

Combinations are sometimes right, such as a stacked bar with one segment broken out beside it (in a chart, a small table or a text box) when the audience will want that detail.

组合有时是合理的，例如当受众需要某个细节时，可在堆积条形图旁边把该分段单独拆出（以图表、小表格或文本框的形式呈现）。

## Which standard the chart follows / 图表遵循哪套标准

If the deck already contains charts and they share a consistent style (axis treatment, gridlines, legend or direct labels, label placement, whether the title sits inside the chart or above it, fonts, colors), that is the standard, and every new chart follows it, even where it differs from the master's theme. Charts that disagree with each other are not a standard. Otherwise follow the house standard below.

如果幻灯片中已包含图表且它们共享一致的风格（坐标轴处理、网格线、图例或直接标注、标签位置、标题位于图表内还是图表上方、字体、颜色），那么这套风格就是标准，每张新图表都遵循它，即使它与母版主题不同。彼此不一致的图表不构成标准。否则，遵循下文的内部标准。

Under a deck standard, the deck's charts decide everything the house standard decides, including where the title sits and how it is worded. Where you cannot reproduce their style directly, copy one of the deck's charts and replace its data and labels, which is the surest way to get an identical look.

在幻灯片自有标准之下，幻灯片中的既有图表决定内部标准所决定的一切事项，包括标题的位置和措辞。在无法直接复现其风格之处，可复制幻灯片中的一张现有图表并替换其数据和标签，这是获得完全一致外观的最可靠方法。

## The house standard / 内部标准

- **No legend**, except where Series names (Finishing touches) allows one.
  **不用图例**，除非 Series names（系列名称，见 Finishing touches 收尾细节）一节允许使用。
- **Bar and column charts:** no gridlines and no value axis; every bar carries its value, and the category axis names the bars. Keep a thin gray category axis line (about #BFBFBF), so the bars don't float.
  **条形图与柱状图：** 不用网格线和数值轴；每根条都标注数值，类别轴标注条形名称。保留一条细灰色类别轴线（约 #BFBFBF），使条形不致悬空。
- **Line charts:** faint horizontal gridlines at four or five round values, so the reader can read values across (faint means close to the background: light gray on a white slide, a dark gray just off black on a dark one); value-axis labels in a muted color that carry the unit ("$0B", "$25B"), with no axis line; no vertical gridlines. A small marker on each labeled point and none on the others, since markers on every point clutter a long line. Lines run straight from point to point, never smoothed.
  **折线图：** 在四五个整数值处绘制淡淡的水平网格线，便于读者横向读取数值（"淡"指接近背景色：白色幻灯片上用浅灰，深色幻灯片上用略深于纯黑的深灰）；数值轴标签使用低调颜色并带有单位（"$0B"、"$25B"），不画轴线；不用竖直网格线。每个有标注的点上放一个小标记，其余点不放，因为每个点都有标记会使长线条显得杂乱。折线在点与点之间直线连接，绝不做平滑处理。
- **Scatter charts:** no gridlines; both value axes, four or five ticks at round numbers, bare numbers in a muted color.
  **散点图：** 不用网格线；两条数值轴各设四五个整数值刻度，刻度为纯数字，使用低调颜色。
- **One accent color** for the point that is the message, gray for everything else. Use the deck's accent: the theme's first accent color, the accent the deck's finished slides use, or the accent you chose when setting up a new deck. Use this fallback palette only when the deck has no accent of its own (default theme, and no finished slide uses one), so decks for different companies don't share a look: accent #1A6BB8, bars #BFBFBF, lines #A6A6A6, text #1A1A1A, secondary text #595959.
  **一种强调色** 用于承载信息的要点，其余一律用灰色。使用幻灯片自身的强调色：主题的第一个强调色、幻灯片已完成页面所用的强调色，或你新建幻灯片时选择的强调色。仅当幻灯片没有自己的强调色时（使用默认主题且已完成的页面未使用任何强调色）才使用以下备用配色，以免不同公司的幻灯片外观雷同：强调色 #1A6BB8，条形 #BFBFBF，折线 #A6A6A6，文本 #1A1A1A，次级文本 #595959。
- **Transparent background.** The chart area and the plot area have no fill and no border, so the chart sits on the slide background.
  **透明背景。** 图表区和绘图区不设填充和边框，使图表融入幻灯片背景。
- **Units in the number format.** Every data label on a bar or column carries its unit ("$42M", "38%", "12K"); only a plain count such as 355 goes bare. On stacked columns only the column total carries the unit ("$40.5M"); the segments are bare numbers (18.2, 12.5, 9.8), and the total sits above each stack.
  **单位写入数字格式。** 条形图或柱状图上的每个数据标签都带单位（"$42M"、"38%"、"12K"）；只有 355 这类纯计数才不带单位。在堆积柱状图上只有柱总计带单位（"$40.5M"），各分段为纯数字（18.2、12.5、9.8），总计位于每个堆积柱上方。
- **Bar proportions.** The gap between bars is 50% of a bar's width, so bars are twice as wide as the gaps. On stacked bars, set the series overlap to 100% so the segments stack on one bar.
  **条形比例。** 条形之间的间距为条宽的 50%，使条形宽度为间距的两倍。在堆积条形图上，将系列重叠设为 100%，使各分段堆叠在同一根条上。
- **Headroom.** Set the value axis maximum about 15% above the tallest bar, so labels above the bars fit; about 25% when totals or growth arrows sit above the bars.
  **顶部留白。** 将数值轴最大值设为最高条形之上约 15% 处，使条形上方的标签放得下；当总计或增长箭头位于条形上方时留约 25%。
- **Sorting.** Horizontal bars are sorted with the largest at the top.
  **排序。** 水平条形图按从大到小排列，最大值在最上方。
- **Likert and other ordered scales** are tints of the one accent, darkest at the top of the scale (such as "Strongly agree") and lightest at the bottom, never a red-to-green ramp. This overrides the two-hue rule for values above and below a baseline in Finishing touches.
  **Likert 及其他有序量表** 使用单一强调色的深浅变化，量表顶端（如 "Strongly agree"）最深、底端最浅，绝不使用红到绿的渐变。此规则覆盖 Finishing touches（收尾细节）中关于基线上下数值的双色规则。
- **Waterfalls.** The start and end totals are dark gray, increases are the accent, and decreases are one contrasting warm color, such as orange #E07B39. Each step is labeled with its signed change, and the totals carry the unit.
  **瀑布图。** 起始与结束总计用深灰色，增加量用强调色，减少量用一种对比的暖色，例如橙色 #E07B39。每一步都标注带符号的变化量，总计带单位。
- **Text.** Axis and data labels are 14pt in the deck's body font; the year row and callouts are smaller (Finishing touches). Text set in a series color (line names, callouts) uses the secondary text color when that series is a light gray.
  **文本。** 坐标轴标签和数据标签使用幻灯片正文字体的 14pt；年份行和标注文字更小（见 Finishing touches 收尾细节）。以系列颜色显示的文本（折线名称、标注文字）在该系列为浅灰色时使用次级文本颜色。

## The chart label / 图表标签

The story is in the slide title (or in the slide's commentary when the deck uses label titles), never in the chart. A story title states the takeaway as a full sentence with the number ("Sales grew every quarter for two years and finished 2025 at $12.4M"). The chart carries a descriptive label only, with these parts in this order, separated by commas: what is plotted, the scope if there is one, the unit, the timeframe.

叙事内容写在幻灯片标题中（当幻灯片采用标签式标题时写在备注中），绝不在图表里。叙事式标题以完整句子陈述结论并包含数字（"Sales grew every quarter for two years and finished 2025 at $12.4M"）。图表只携带一个描述性标签，各部分按以下顺序以逗号分隔：所绘内容、适用范围（如有）、单位、时间范围。

- The unit is a short token, never spelled out. Currency scale letters follow the deck's own convention when it has one, otherwise the deck's language: $K, $M and $B for US English (also the default when the language is unclear), £k, £m and £bn for UK English, and so on. Do not mix conventions in one deck.
  单位是简短符号，绝不拼写完整。货币量级字母优先遵循幻灯片自身的约定，若无约定则遵循幻灯片语言：美式英语用 $K、$M 和 $B（语言不明时也以此为默认），英式英语用 £k、£m 和 £bn，依此类推。同一套幻灯片中不要混用不同约定。
- Parentheses hold only a second unit or a data basis, never the primary unit.
  括号内只放第二单位或数据口径，绝不放主单位。
- What is plotted always stays.
  所绘内容始终保留。
- Drop the timeframe when the axis shows the year, in the tick labels ("FY22") or on a year row. Keep it when the axis shows only bare periods ("Q1", "Jan"), as on a single-year chart, so the year appears somewhere on the slide.
  当坐标轴以刻度标签（"FY22"）或年份行显示年份时，省略时间范围。当坐标轴只显示不带年份的期间（"Q1"、"Jan"）时——例如单年图表——保留时间范围，使年份在幻灯片某处出现。
- The unit is on every data label of a bar or column chart and on the value-axis labels of a line chart, so it drops out of the chart label for both. On stacked columns the totals carry it. On scatter charts the axes show bare numbers, so the unit stays in the label.
  单位出现在条形图或柱状图的每个数据标签上，以及折线图的数值轴标签上，因此这两种图表的标签中可省略单位。堆积柱状图的总计带单位。散点图的坐标轴只显示纯数字，因此单位保留在标签中。
- Decide at the first chart which parts this deck's labels carry (scope, unit, timeframe), and apply that and the drop rules the same way on every chart, checking the earlier labels before each new one. Half the charts labeled "$M, FY25" and half not looks careless.
  在第一张图表上确定本套幻灯片的标签包含哪些部分（范围、单位、时间范围），并在每张图表上以同样方式应用该决定和省略规则，每次新建标签前先核对先前的标签。一半图表标着 "$M, FY25" 而另一半不标，会显得漫不经心。
- Never in the label: the takeaway, a figure (a growth rate, total or change goes in the slide title, a data label, a growth arrow or a callout), a colon or a dash between parts, a spelled-out unit.
  标签中绝不出现：结论、具体数字（增长率、总量或变化量应写入幻灯片标题、数据标签、增长箭头或标注文字）、部分之间的冒号或连字符、拼写完整的单位。

Examples:

示例：

- Column chart, bars labeled "$42M", x-axis "Q1 … Q4 | Q1 … Q4" with "2024" and "2025" on a year row → **"Quarterly revenue"**, not "Quarterly revenue, $M, Q1 2024–Q4 2025".
  柱状图，条形标注 "$42M"，x 轴为 "Q1 … Q4 | Q1 … Q4" 且 "2024" 和 "2025" 位于年份行 → **"Quarterly revenue"**，而不是 "Quarterly revenue, $M, Q1 2024–Q4 2025"。
- Column chart, bars labeled "$42M", x-axis "Q1 … Q4" (one year, no year row) → **"Quarterly revenue, FY25"**: the timeframe stays because the axis has none.
  柱状图，条形标注 "$42M"，x 轴为 "Q1 … Q4"（单年，无年份行）→ **"Quarterly revenue, FY25"**：坐标轴上没有年份，因此保留时间范围。
- Line chart, axis labels "0K … 250K", x-axis "Jan … Dec | Jan … Jun" with "2025" and "2026" on a year row → **"Monthly active users"**: the axis carries the unit and the year row the timeframe.
  折线图，轴标签 "0K … 250K"，x 轴为 "Jan … Dec | Jan … Jun" 且 "2025" 和 "2026" 位于年份行 → **"Monthly active users"**：单位由坐标轴承载，时间范围由年份行承载。
- Column chart with growth arrows, bars labeled "$42M", growth labels "+21%" → **"Quarterly revenue, FY25 (% growth QoQ)"**: a second unit goes in parentheses at the end.
  带增长箭头的柱状图，条形标注 "$42M"，增长标签 "+21%" → **"Quarterly revenue, FY25 (% growth QoQ)"**：第二单位放在末尾的括号内。
- Bar chart of segment shares, bars labeled "58%" → **"Regional share of revenue, FY25 (based on reported segments)"**: a data basis goes in parentheses.
  分部份额条形图，条形标注 "58%" → **"Regional share of revenue, FY25 (based on reported segments)"**：数据口径放在括号内。

The label is a text box in the slide's caption style directly above the chart, so its typography matches the slide; the chart's own built-in title stays off. Line the label's first character up with the slide title's: the same left position and the same left text inset (internal padding). A title placeholder usually has a nonzero inset from the master and a new text box has an inset of 0.1 in (7.2pt), so copy the title's inset rather than setting zero. Keep the label to one line when the subject is short. When the subject runs to about six words or more ("Quarterly sales by department, Product X"), put the unit and timeframe on a second line about two points smaller and muted ("FY25", or "$M, FY25" where the label keeps the unit) rather than letting the label wrap or run into the chart. A slide with two or more charts gives each its own descriptive label, and the slide title carries the message they make together.

标签是位于图表正上方、采用幻灯片说明文字样式的文本框，因此其排版与幻灯片一致；图表自带的内置标题保持关闭。使标签的首字符与幻灯片标题的首字符对齐：相同的左端位置和相同的左侧文本内边距（内部填充）。标题占位符通常继承母版的非零内边距，而新建文本框的内边距为 0.1 英寸（7.2pt），因此应复制标题的内边距而不是设为零。主题较短时标签保持一行。当主题达到约六个词或更长时（"Quarterly sales by department, Product X"），将单位和时间范围放到第二行，字号约小两磅并使用低调颜色（"FY25"，或标签保留单位时的 "$M, FY25"），而不是让标签折行或延伸到图表上。包含两张或以上图表的幻灯片为每张图表配一个描述性标签，幻灯片标题承载它们共同构成的信息。

## Placing the chart / 图表的放置

- A chart on its own takes the whole content area: the full content width (the title's left edge to its right edge), from just under the title to just above the footnotes. The common mistake is a chart that stops halfway down the slide, not one that is too big.
  单独放置的图表占据整个内容区域：整个内容宽度（从标题左缘到标题右缘），上起标题正下方，下至脚注正上方。常见错误是图表在幻灯片半腰处就停住，而不是图表过大。
- Small multiples: no panel shorter than about 120pt, or the labels have nowhere to go. Share one scale across the panels only when the comparison is about magnitude; when a shared scale flattens the smaller panels into a line, give each panel its own scale and say so in the caption.
  小倍数图：每个面板高度不小于约 120pt，否则标签无处安放。仅当比较的对象是量级时才在各面板间共享同一刻度；当共享刻度把较小的面板压成一条线时，给每个面板各自的刻度，并在说明文字中注明。

## Finishing touches / 收尾细节

Once the chart exists, adjust it so the message is visible:

图表完成后，对其进行调整以使信息清晰可见：

- **Color does one job per chart.** A second accent is a second message; if two things matter, make two charts. Use distinct hues only when telling the series apart is the point, and then the same hue for the same entity on every chart in the deck. Use a single hue from light to dark for magnitude, and two opposed hues around gray for above and below a baseline. Never a rainbow, and never color as the only carrier of a distinction (red and green least of all): a label or position always backs it up.
  **每种颜色在单张图表中只承担一项任务。** 第二种强调色意味着第二条信息；如果有两件事重要，就做两张图表。只有当区分系列本身就是重点时才使用不同色相，且同一实体在整套幻灯片的每张图表中都使用同一色相。表示量级时使用单一色相由浅到深，表示基线上下时在灰色两侧使用两种对立色相。绝不用彩虹色，也绝不让颜色成为区分的唯一载体（红绿搭配尤甚）：始终有标签或位置作为佐证。
- **Source line** below the chart, at footnote size (10–12pt, muted), following "Footnotes and sources" in `references/slides.md`, including citing only sources you have.
  图表下方的**来源行**，字号与脚注相同（10–12pt，低调颜色），遵循 `references/slides.md` 中的 "Footnotes and sources"（脚注与来源），包括只引用你确实拥有的来源。
- **Period axes.** Tick labels show only the short period (Q1 … Q4, Jan … Dec). The year sits on a second row under each year's first period, at 11pt in the secondary text color, and is not repeated on the other ticks; "Q1 FY25, Q2 FY25, …" on every tick is the defect to avoid. A linked Sheets chart has no tested way to draw it, so put small text boxes under the axis. A single-year chart has no year row and carries the year in its label.
  **期间坐标轴。** 刻度标签只显示短期间（Q1 … Q4、Jan … Dec）。年份位于每个年份首个期间下方的第二行，11pt，使用次级文本颜色，且不在其他刻度上重复；每个刻度都写成 "Q1 FY25, Q2 FY25, …" 是要避免的缺陷。关联的 Sheets 图表没有经过验证的绘制方法，因此可在坐标轴下方放置小的文本框。单年图表没有年份行，年份由其标签承载。
- **Series names.** Give each line a short name, one or two words ("Data center"), in its series color, right of its last point on the same line as the end value ("$89.0B Data center"), with room left on the right of the plot. On a chart of three lines or fewer other than a slope graph, there are two exceptions: when the deck standard uses a legend, use the deck's legend; otherwise a name longer than about 14 characters gets a small legend instead of a wrapped end label. With four or more lines, or on a slope graph, shorten long names instead. A chart with two or more lines is not done until every line is named on the chart, by an end label or a legend.
  **系列名称。** 为每条折线起一个简短名称，一到两个词（"Data center"），使用其系列颜色，放在其末点右侧、与结束值同一行（"$89.0B Data center"），并在绘图区右侧留出空间。三条或更少折线的图表（斜率图除外）有两个例外：当幻灯片标准使用图例时，使用幻灯片的图例；否则名称超过约 14 个字符时改用小图例，而不是折行的末端标签。四条或更多折线时，或斜率图上，改为缩短长名称。包含两条或以上折线的图表，只有当每条折线都通过末端标签或图例在图上命名后才算完成。
- **Point labels on a line:** the start, the end, and the peak, trough or break the title or bullets refer to. Three labels on a ten-point line is right; more than four is crowding, and the first and last alone are too few for a long line. Each label except the end label sits above or below its point, off the line.
  **折线上的点标签：** 起点、终点，以及标题或项目符号所指的峰值、谷值或转折点。十个点的折线上放三个标签是合适的；超过四个显得拥挤，而只有首尾两个对长线来说又太少。除末端标签外，每个标签位于其点的上方或下方，离开折线。
- **Annotate the conclusion:** a shaded band over the downturn, or a callout when Callouts below calls for one. Where actuals turn into a projection, mark the boundary so nobody reads the forecast as history: on a line, the forecast portion is dashed; on columns, forecast bars are hatched or a visibly lighter fill, and their category labels carry an E suffix ("2026E", "Q3E"). A short label at the boundary ("Actual" | "Forecast") helps when the deck will be read on its own.
  **为结论添加注释：** 在下行区间上加盖阴影带，或在下方 Callouts（标注文字）一节要求时添加标注。在实际值转为预测值之处标出分界，避免任何人把预测读成历史：折线图的预测部分用虚线；柱状图的预测柱用斜线填充或明显更浅的填充，且其类别标签带 E 后缀（"2026E"、"Q3E"）。当幻灯片会被独立阅读时，在分界处加一个简短标签（"Actual" | "Forecast"）会有帮助。
- **Growth arrows** show growth or decline where it is significant, not between every pair of bars. The default is an elbow: up from one bar, across at a height that clears the bars between and their labels, and down onto the other. A straight line from one bar top to the next works only between adjacent bars with room above them. The third shape is a difference marker (a short horizontal mark at each bar top and a vertical line between the two levels with the change on it). Whichever shape, the change number sits centered on the arrow (on an elbow, on its horizontal part), in front of it, on a fill matching the slide background. Growth arrows and CAGR brackets are shapes drawn over the chart, never chart elements; position them from the chart's geometry.
  **增长箭头** 只在增长或下降显著之处显示，而不是每对条形之间都画。默认形状是肘形：从一根条形向上，在高于中间条形及其标签的高度横向移动，再落到另一根条形上。从一根条形顶端直线连到下一根的做法只适用于上方有空间的相邻条形。第三种形状是差值标记（每根条形顶端一个短横线标记，两个水平之间一条竖线，变化量标在竖线上）。无论哪种形状，变化数字都居中放在箭头之上（肘形箭头放在其水平部分上），位于箭头前景，衬底与幻灯片背景色一致。增长箭头和 CAGR 括号是覆盖在图表之上的形状，绝不是图表元素；应根据图表的几何结构定位它们。
- **Callouts:** when the slide title or commentary claims a change in trajectory (an acceleration, a break, a launch, a one-off), put a callout naming the cause at the point where it happens, on the chart rather than only suggested in your reply. If nothing you were given names the cause, ask, or leave the callout out; never guess one. Otherwise most charts need none. At most one, occasionally two; more than that is clutter, and the explanation belongs in a text block beside the chart instead. The callout is a short phrase ("Supply chain outage", "Product X launch"), never a full sentence, at 11–12pt in the color of the series it annotates, with no box, border or fill, and a thin leader line in the same color from the text to the point. Place the text in the nearest open area of the plot, never over a bar, the line, or a data label.
  **标注文字（Callouts）：** 当幻灯片标题或备注声称轨迹发生变化（加速、断点、产品发布、一次性事件）时，在发生该变化的点上放置一个说明原因的标注，显示在图表上而不是只在回复中暗示。如果拿到的材料中没有说明原因，就询问，或不加标注；绝不要凭空猜测。除此之外，多数图表不需要标注。至多一个，偶尔两个；再多就是杂乱，解释应改为放在图表旁边的文本块中。标注文字是一个简短短语（"Supply chain outage"、"Product X launch"），绝不是完整句子，11–12pt，使用其注释对象系列的颜色，无框、无边框、无填充，并有一条同色的细引导线从文字连到该点。文字放在绘图区最近的空白区域，绝不要压在条形、折线或数据标签上。

【评论】"拿不到原因就询问或省略、绝不凭空猜测"的条款是典型的反幻觉设计：宁可让图表少一个标注，也不让模型编造归因。
