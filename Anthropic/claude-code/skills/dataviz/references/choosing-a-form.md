<!-- BILINGUAL-EN-ZH -->
# Choosing a form / 选择图表形式

Decide this **before** color. The data's job picks the form - and sometimes the
right form is not a chart.

要**先于**配色做出这一决定。数据的任务决定形式——有时正确的形式根本不是图表。

## Is it even a chart? / 这究竟是不是图表？

| The data is... | Use | Not |
|---|---|---|
| A single current value (+ maybe a trend) | **Stat tile** (value + delta + sparkline) | A one-bar bar chart |
| A handful of headline numbers | **KPI row** of stat tiles | A grouped bar chart |
| The one number a dashboard leads with | **Hero figure** (>=48px, sans) | - |
| A single ratio against a limit | **Meter** (same-ramp track) | A pie of 2 slices |
| More than ~7 classes that all carry meaning | A **table** (or table + chart) | More colors |

| The data is... / 数据是…… | Use / 使用 | Not / 而非 |
|---|---|---|
| 单个当前值（+ 可能有趋势） | **统计磁贴**（数值 + 变化量 + 迷你走势图） | 只有一根柱子的柱状图 |
| 少数几个头条数字 | 由统计磁贴组成的 **KPI 行** | 分组柱状图 |
| 仪表盘主打的那个数字 | **英雄数字**（>=48px，无衬线体） | - |
| 相对某个上限的单一比率 | **量规**（同色阶轨道） | 两片的饼图 |
| 超过约 7 个且都承载含义的类别 | **表格**（或表格 + 图表） | 更多颜色 |

If a chart *is* right, pick the type by the job:

如果图表*确实*是正确形式，就按任务选择类型：

## The job -> the type / 任务 -> 类型

| Job (what the reader must do) | Default form | Color job |
|---|---|---|
| Compare magnitude, low -> high | bar / column; **heatmap** for a grid | sequential (one hue) |
| Trend over time | line; area for a single series | sequential or 1 categorical |
| Tell distinct series apart | grouped/stacked bar, multi-line | **categorical** |
| One series is the point, rest are context | **emphasis** (highlight one, gray the rest) | 1 hue + gray |
| Above/below a baseline; delta to target | diverging bar, or line vs baseline | diverging |
| Part-to-whole | **stacked bar** (go horizontal for many / long-named categories) | categorical |
| Ordered-scale share (Likert, sentiment, agree<->disagree) | **diverging stacked bar**, centered on neutral | diverging |
| Before -> after per item | dumbbell | 1 hue, 2 shades |

| Job (what the reader must do) / 任务（读者要做的事） | Default form / 默认形式 | Color job / 配色任务 |
|---|---|---|
| 比较数值大小，从低到高 | 条形图/柱状图；网格用**热力图** | 顺序型（单一色相） |
| 随时间的趋势 | 折线图；单一系列用面积图 | 顺序型或 1 个分类色 |
| 区分不同的系列 | 分组/堆叠条形图、多折线图 | **分类型** |
| 一个系列是重点，其余是背景 | **强调型**（突出一个，其余置灰） | 1 个色相 + 灰色 |
| 高于/低于某基线；相对目标的差值 | 发散型条形图，或折线对基线 | 发散型 |
| 部分到整体 | **堆叠条形图**（类别多或名称长时改为水平方向） | 分类型 |
| 有序量表上的占比（李克特量表、情感倾向、同意<->反对） | **发散型堆叠条形图**，以中立项为中心 | 发散型 |
| 每个条目的之前 -> 之后 | 哑铃图 | 1 个色相、2 个深浅 |

【评论】该参考用"读者必须完成的任务"而非数据形状来决定图表类型，属于以任务为导向的可视化选型方法论；配色栏与流行的"色彩仅承载一种含义"原则一致。

## The rules behind the table / 表格背后的规则

- **Sequential is the safe default.** One hue, more-is-darker. It stays legible and
  consistent and is hard to misread. Reach for it unless the data's job is
  specifically *identity* or *polarity*.
  - **顺序型是安全的默认选择。**单一色相，数值越大颜色越深。它保持清晰易读、前后一致，且很难被误读。除非数据的任务明确是*身份识别*或*极性*，否则就用它。
- **Categorical is for when the series ARE the subject** - and it has a real cost:
  it can bury the one data point that actually matters. If the story is "this one
  went up," that's **emphasis**, not categorical.
  - **分类型用于"系列本身就是主角"的场合**——而且它有实际代价：可能把真正重要的那个数据点埋掉。如果故事是"这一项涨上去了"，那应当用**强调型**，而不是分类型。
- **Emphasis** = the most underused form. One series in the accent hue, the rest in
  the de-emphasis gray. Often the honest answer to "make this chart clearer."
  - **强调型** = 最被低估的形式。一个系列用强调色相，其余用弱化灰。对"把这张图弄清楚些"这类要求，这往往才是诚实的答案。
- **Texture is an opt-in expression, not a default form.** It earns its place only
  for accessibility (full CVD), print/export, and `forced-colors`. Never decorative.
  -> see `marks-and-anatomy.md`.
  - **纹理是可选的表达手段，不是默认形式。**只有无障碍（全色盲）、打印/导出和 `forced-colors` 场景才配得上用它。绝不做装饰用途。-> 参见 `marks-and-anatomy.md`。

## Series-count ladder (categorical) / 系列数量阶梯（分类型）

| Series | Treatment |
|---|---|
| 1-3 | color alone is comfortable for everyone; direct-label |
| 4 | adjacent forms (stacks, bars, lines) stay gate-safe, but direct labels become mandatory - yellow and orange now share the screen; all-pairs forms (scatter, bubble, choropleth, small multiples) cap at **three** - fold to "Other" or facet rather than seat a 4th |
| 5-6 | soft cap; legend or small multiples |
| 7-8 | token ceiling; past it, fold the tail into "Other," facet into small multiples, or use composite encoding (hue × shape) |

| Series / 系列数 | Treatment / 处理方式 |
|---|---|
| 1-3 | 仅靠颜色对所有人都清晰可辨；直接标注 |
| 4 | 相邻比较形式（堆叠、条形、折线）仍在"门槛"安全线内，但直接标注变为强制——黄色与橙色会同屏出现；全两两比较形式（散点图、气泡图、分级统计地图、小倍数图）上限为 **3**——宁可归入"其他"或分面，也不要塞入第 4 个 |
| 5-6 | 软上限；用图例或小倍数图 |
| 7-8 | 记号上限；超过之后，把尾部归入"其他"、分面成小倍数图，或使用复合编码（色相 × 形状） |

Never solve "too many series" by generating more hues. A generated 9th hue is
indistinguishable from an existing one under CVD and breaks every check.

绝不要靠生成更多色相来解决"系列太多"的问题。在色觉障碍（CVD）条件下，生成的第 9 个色相与已有色相无法区分，并且会破坏所有检查。
