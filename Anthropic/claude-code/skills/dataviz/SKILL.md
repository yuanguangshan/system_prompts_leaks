---
name: dataviz
description: >
  Use this skill whenever you are about to create ANY chart, graph, plot,
  dashboard, or data visualization, in ANY output medium — an HTML or React
  artifact, inline SVG, plotting code in any library (matplotlib, plotly, d3,
  Recharts, …), an image/PNG you will render and upload, or a chart shared into
  Slack. Read it BEFORE writing the first line of chart code, choosing chart
  colors, building a stat tile / meter / KPI row, or laying out a dashboard.
  When the destination is a first-party document connector (host-designated,
  never self-described) that renders live charts, hand it the rows (inline, or
  as an uploaded data file the chart cites) rather than a rendered PNG/SVG — a
  picture of a chart loses hover, data inspection and per-value comments.
  Produces visualizations that read as one system — elegant, accessible,
  consistent in light and dark — using a brand-neutral placeholder palette you
  swap for your own. Teaches a design-system-agnostic method: a form heuristic,
  a color formula with a runnable validator, mark specs, and interaction rules.
  A validated default palette is documented in `references/palette.md` — swap
  that file's values for your brand's. Triggers on: "chart", "graph", "plot",
  "data viz", "visualization", "dashboard", "analytics", "visualize data",
  "categorical colors", "sequential / diverging palette", "stat tile",
  "sparkline", "heatmap", "legend", "axis", "tooltip", "chart colors", "color
  by series".
---
<!-- BILINGUAL-EN-ZH -->

# Data Visualization / 数据可视化

A chart is **read by people and executed by you**. This skill turns "make it look
good" into a procedure with checks, so the result is right by construction rather
than by taste.

图表是**由人来读、由你来执行**的。本技能把"让它好看"变成一套带检查项的流程，使结果是"构造上正确"，而不是"凭品味正确"。

**The method here is design-system-agnostic.** Nothing in the procedure, the form
heuristic, the six checks, or the mark specs is specific to one product. A design
system supplies a small set of *parameters* (its ramps, a categorical order, a
diverging pair, a status palette, a texture, its surfaces, its filter components);
the method consumes them unchanged. A **validated default palette** is the
reference instance, fully specified in `references/palette.md`. To target your
brand, read that file's structure and substitute its values - touch nothing else.

**这里的方法与设计系统无关。**流程、图型启发式、六项检查、标记规格中没有一项是针对某个具体产品的。设计系统只需提供一小组*参数*（它的色阶、类别色顺序、发散色对、状态色板、纹理、表面色、筛选组件）；方法原样消费这些参数。一套**经过验证的默认调色板**是参考实例，在 `references/palette.md` 中完整给出。要适配你的品牌，读取该文件的结构并替换其中的值——其余一概不动。

> The single most important habit: **the color part is computable, so compute it.**
> Never eyeball whether a palette is colorblind-safe - run `scripts/validate_palette.js`.

> 最重要的唯一习惯：**颜色部分是可计算的，所以就去计算它。**
> 绝不靠目测判断调色板是否色觉安全——运行 `scripts/validate_palette.js`。

## The procedure - do these in order / 流程——按顺序执行

Color comes LAST. Most bad charts pick colors first.

颜色是最后一步。大多数糟糕的图表都先挑颜色。

1. **Pick the form.** What is the data's job - magnitude, identity, polarity, a
   single headline, change-over-time? The job picks the chart type, and sometimes
   the answer is *not a chart* (a stat tile or hero number). -> `references/choosing-a-form.md`

   1. **选定图型。**数据的职责是什么——数量大小、身份、极性、单个头条数字、随时间的变化？由职责决定图表类型，有时答案是*不用图表*（用统计瓦片或大数字）。-> `references/choosing-a-form.md`
2. **Assign color by the job it does.** Categorical (identity), sequential
   (magnitude), diverging (polarity), or status (state) - each has one rule.
   Assign categorical hues in fixed order, never cycled. -> `references/color-formula.md`

   2. **按颜色承担的职责分配颜色。**类别型（身份）、连续型（数量）、发散型（极性）或状态（状态）——每种各有一条规则。类别色相按固定顺序分配，绝不循环。-> `references/color-formula.md`
3. **VALIDATE the palette - run the script, don't reason about Delta E.**
   `node scripts/validate_palette.js "<hex,hex,...>" --mode light` (relative to
   this skill's base directory - or load it as `<script type="module">` in the
   chart's own page, where it reads
   `data-palette` off `<body>` and logs a `console.table` report). It returns
   pass/fail on the lightness band, chroma floor, adjacent-pair CVD separation,
   the normal-vision floor, and contrast. Fix anything that FAILs before continuing. Re-run for
   `--mode dark` with that mode's surface.

   3. **校验调色板——运行脚本，不要靠推理 Delta E。**
   `node scripts/validate_palette.js "<hex,hex,...>" --mode light`（相对于本技能的基础目录——或在图表所在页面中以 `<script type="module">` 加载，它会从 `<body>` 读取 `data-palette` 并输出 `console.table` 报告）。它对明度区间、彩度下限、相邻颜色对 CVD 区分度、正常视觉下限与对比度给出 pass/fail。先修复所有 FAIL 项再继续。再以 `--mode dark` 配合该模式的表面色重跑一次。
4. **Apply mark specs & spacers.** Thin marks, 4px rounded data-ends anchored to
   the baseline, 2px lines, >=8px markers, a 2px surface gap between fills (stacked
   segments and adjacent bars alike) and a 2px surface ring on overlapping marks,
   selective direct labels. -> `references/marks-and-anatomy.md`

   4. **应用标记规格与间隔。**细标记、锚定在基线上的 4px 圆角数据端、2px 线条、>=8px 的标记点、填充之间的 2px 表面色间隙（堆叠段与相邻柱都适用）、重叠标记上的 2px 表面色描边、有选择性的直接标签。-> `references/marks-and-anatomy.md`
5. **Add the hover layer - by default.** An HTML/SVG chart *is* interactive; ship
   a crosshair+tooltip on line/area and a per-mark hover tooltip on bar/dot/cell.
   The only form that skips it is a bare stat tile with no plot. Hit targets bigger
   than the mark; filters in one row above the charts. -> `references/interaction.md`

   5. **添加悬停层——默认添加。**HTML/SVG 图表*天生*是交互式的；折线/面积图提供十字线+提示框，柱/点/单元格提供逐标记悬停提示框。唯一可以省略的图型是不带绘图区的纯统计瓦片。命中区域要大于标记本身；筛选器放在图表上方的一行内。-> `references/interaction.md`
6. **Final accessibility pass.** For >= 2 series a legend is always present and <= 4
   are also direct-labeled (a single series needs no legend box - the title names
   it), so identity is never color-alone; a table view exists; dark mode is **selected** - its own
   steps from the same ramps, validated against the dark surface, not an automatic
   flip; texture is available for the CVD/print/forced-colors case.

   6. **最终无障碍检查。**>= 2 条系列时图例始终存在，且 <= 4 条时同时直接标注（单条系列不需要图例框——标题已说明系列名），因此身份识别绝不只靠颜色；提供表格视图；深色模式是**被选定的**——从同一批色阶中选取自己的档位、对照深色表面校验，而不是自动翻转；为 CVD/打印/强制配色场景提供纹理。
7. **Render it and look at it.** The validator checks color, not layout - open or
   screenshot the output and eyeball it for label collisions, geometry, and overflow
   before calling it done.

   7. **渲染并查看它。**校验器检查的是颜色，不是布局——打开或截图输出结果，目测检查标签碰撞、几何形状与溢出，然后才算完成。

Then check the result against **`references/anti-patterns.md`** - it is the catalog
of what goes wrong. If your chart matches an entry, it's wrong.

然后对照 **`references/anti-patterns.md`** 检查结果——它是"会出什么错"的目录。如果你的图表命中其中条目，那它就是错的。

## Non-negotiables (true in every design system) / 不可妥协项（在任何设计系统中都成立）

- **Assign categorical hues in fixed order, never cycled.** A 9th series is never a
  generated hue - it folds into "Other," small multiples, or composite encoding.
  **类别色相按固定顺序分配，绝不循环。**第 9 条系列绝不使用生成的新色相——它折叠进 "Other"、小组图或复合编码。
- **One axis.** Never a dual-axis chart (two y-scales). Two measures of different
  scale -> two charts, small multiples, or indexed to a common base. *(This is the
  #1 chart mistake - see anti-patterns.)*
  **单坐标轴。**绝不使用双轴图（两个 y 刻度）。两个量纲不同的度量 -> 两张图、小组图，或按共同基线归一化。*（这是第一大图表错误——见 anti-patterns。）*
- **Color follows the entity, never its rank.** A filter that changes the series
  count must not repaint the survivors.
  **颜色跟随实体，绝不跟随其排名。**改变系列数量的筛选器不得给幸存的系列重新着色。
- **Sequential = one hue, light->dark. Diverging = two hues + a neutral gray
  midpoint.** Never a rainbow; never a hue at the diverging midpoint.
  **连续型 = 单一色相，由浅到深。发散型 = 两种色相 + 中性灰中点。**绝不用彩虹色；发散中点绝不用色相。
- **Run the validator before shipping any categorical palette.** CVD Delta E >= 8 is the
  target (OKLab ×100); 6-8 is a floor that is legal ONLY with secondary encoding. A
  normal-vision floor below 15 is a hard FAIL - full-color readers can't tell the
  pair apart; re-step it on the adjacent pairlist (secondary encoding does not excuse
  this one); under `--pairs all` cut series or facet instead - see check 4. A contrast WARN
  obligates visible labels or a table view - it is not dismissable.
  **交付任何类别型调色板之前先运行校验器。**CVD Delta E >= 8 是目标（OKLab ×100）；6-8 是下限区间，只有提供次级编码时才合法。正常视觉下限低于 15 是硬性 FAIL——全色觉读者无法区分该颜色对；在相邻颜色对列表上重新调整其中一方的阶梯（这一项次级编码不能豁免）；在 `--pairs all` 下则改为削减系列或分面——见检查 4。对比度 WARN 必须以可见标签或表格视图补救——不可忽略。
- **Thin marks; a legend always present for >= 2 series (none for one), with
  selective direct labels (never a number on every point); recessive grid/axes.**
  **细标记；>= 2 条系列时图例始终存在（单条不需要），并配以有选择性的直接标签（绝不在每个点上标数字）；网格/坐标轴保持低调。**
- **Text wears text tokens, never the series color** - values, labels, and legends
  stay in primary/secondary/muted ink; a colored mark beside them carries identity.
  **文字使用文字 token，绝不使用系列色**——数值、标签与图例保持在 primary/secondary/muted 墨色；旁边的彩色标记承载身份信息。
- **Status colors are reserved** (good/warning/serious/critical) and never reused
  for "series 4"; they ship with an icon + label, never color alone.
  **状态色是保留的**（良好/警告/严重/危急），绝不复用于"第 4 条系列"；它们始终与图标 + 标签一起出现，绝不只靠颜色。

## Plugging in a design system / 接入一个设计系统

The method is invariant; only these parameters change per system. The reference
instance - every value filled in - is `references/palette.md`.

方法本身不变；每个系统只改变以下参数。参考实例——每个值都已填好——是 `references/palette.md`。

| Parameter | What the system provides |
|---|---|
| **Ramps** | the hue scales (named steps) the palette draws from |
| **Categorical theme** | the fixed hue order (a named theme); default + alternates |
| **Sequential hue** | the default single hue for magnitude |
| **Diverging pair** | two warm/cool poles + a neutral midpoint |
| **Status palette** | good / warning / serious / critical - steps distinct from categorical |
| **Texture fill** | one directional hand-drawn fill, used at 45° / 135° |
| **Surfaces** | light & dark chart-surface colors (the validator needs these) |
| **Filter controls** | date-range & dimension controls (behavioral spec in `interaction.md`) |

| 参数 | 系统需要提供的内容 |
|---|---|
| **色阶（Ramps）** | 调色板取色用的色相刻度（有名字的档位） |
| **类别主题（Categorical theme）** | 固定色相顺序（一个具名主题）；默认 + 备选 |
| **连续色相（Sequential hue）** | 表达数量大小的默认单一色相 |
| **发散色对（Diverging pair）** | 暖/冷两极 + 中性中点 |
| **状态色板（Status palette）** | 良好 / 警告 / 严重 / 危急——档位与类别色明显区分 |
| **纹理填充（Texture fill）** | 一种方向性手绘填充，以 45° / 135° 使用 |
| **表面色（Surfaces）** | 浅色与深色图表表面色（校验器需要它们） |
| **筛选控件（Filter controls）** | 日期范围与维度控件（行为规格见 `interaction.md`） |

To onboard a new system: fill those rows, feed its ramps to the validator, and let
it snap each slot to the nearest passing step. Structure and rules stay as written.

接入新系统的方法：填好这些行，把它的色阶喂给校验器，让校验器把每个槽位吸附到最近的达标档位。结构与规则保持原文所述不变。

## Reference files / 参考文件

| File | What it answers |
|------|-----------------|
| `references/choosing-a-form.md` | Which chart type / is it even a chart? |
| `references/color-formula.md` | The four jobs, the six checks, snap-to-passing |
| `references/marks-and-anatomy.md` | Mark specs, spacers, labels, figures, hero number |
| `references/interaction.md` | Tooltips & hover, filters & time ranges |
| `references/components.md` | The pieces a chart is made of - build each in plain HTML |
| `references/anti-patterns.md` | **What goes wrong - check every chart against this** |
| `references/palette.md` | **The reference palette instance** - every parameter, filled in; swap for your brand's |
| `scripts/validate_palette.js` | Runnable six-checks validator (run it; don't eyeball) |

| 文件 | 回答什么问题 |
|------|-----------------|
| `references/choosing-a-form.md` | 选哪种图表类型 / 这到底该不该用图表？ |
| `references/color-formula.md` | 四种职责、六项检查、吸附到达标 |
| `references/marks-and-anatomy.md` | 标记规格、间隔、标签、数字呈现、大数字（hero number） |
| `references/interaction.md` | 提示框与悬停、筛选器与时间范围 |
| `references/components.md` | 图表的组成部件——每一件都用纯 HTML 构建 |
| `references/anti-patterns.md` | **会出什么错——每张图表都要对照检查** |
| `references/palette.md` | **参考调色板实例**——每个参数都已填好；替换为你品牌的值 |
| `scripts/validate_palette.js` | 可运行的六项检查校验器（运行它；不要目测） |
