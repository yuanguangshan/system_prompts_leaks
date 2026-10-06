<!-- BILINGUAL-EN-ZH -->
# Color formula / 颜色公式

Color is **not hand-picked**. Every chart color does exactly one of four jobs, and a
palette is legal only if it passes six checks. The checks are the product - they are
what makes a palette safe to change and what lets the same method run on any design
system's ramps.

颜色**绝非手工挑选**。每个图表颜色恰好只承担四种职责中的一种，而一套调色板只有通过六项检查才算合法。这些检查本身就是这套方法的核心——正是它们让调色板可以安全地更换，也让同一套方法能运行在任何设计系统的色阶上。

## The four jobs / 四种职责

| Job | What it encodes | Structure |
|---|---|---|
| **Categorical** | identity (which series) | 8 hues, fixed order, assigned in sequence, never cycled |
| **Ordinal** | position in a sequence (funnel stage, tier, bucket) | one hue, monotone lightness steps; light end still >= 2:1 on surface |
| **Sequential** | magnitude (how much) | one hue, steps 100->700, light->dark; flips anchor in dark |
| **Diverging** | polarity (which side of a baseline) | two hues + a neutral gray midpoint; equal steps per arm |
| **Status** | state (good->critical) | a small fixed scale, reserved meaning, always icon+label |

| 职责 | 编码的内容 | 结构 |
|---|---|---|
| **类别型（Categorical）** | 身份（哪条系列） | 8 种色相，固定顺序，按序分配，绝不循环 |
| **顺序型（Ordinal）** | 序列中的位置（漏斗阶段、层级、分桶） | 单一色相，明度单调递进；浅端在表面上仍 >= 2:1 |
| **连续型（Sequential）** | 数量大小（多少） | 单一色相，阶梯 100->700，由浅到深；深色模式下翻转锚点 |
| **发散型（Diverging）** | 极性（基线的哪一侧） | 两种色相 + 中性灰中点；每臂等距阶梯 |
| **状态（Status）** | 状态（良好->危急） | 小型固定刻度，含义保留，始终图标+标签 |

**Categorical or ordinal?** If swapping the category order would change the
meaning - funnel stages, size tiers (S/M/L), age bands, cohort buckets - it is
**ordinal** and takes a one-hue ramp so the reader sees the order in the color.
If swapping would not - product names, teams, regions, endpoints - it is
**nominal categorical** and each bar takes the *same* slot-1 hue (one series,
so no legend box - the title names it), or slots 1..N when there are N separate
series. Never color nominal bars by their value: that spends the identity channel
re-encoding what bar length already shows.

**类别型还是顺序型？**如果交换类别顺序会改变含义——漏斗阶段、尺码档（S/M/L）、年龄段、同期群分桶——那它就是**顺序型**，应使用单一色相的色阶，让读者从颜色中直接看出顺序。如果交换不会改变含义——产品名、团队、地区、端点——那它就是**名义类别型**，每根柱子使用*同一个*槽位 1 的色相（只有一条系列，因此不需要图例框——标题已说明系列名），或在有 N 条独立系列时使用槽位 1..N。绝不按数值给名义柱着色：那会把身份通道浪费在重复编码柱长已经表达的信息上。

【评论】正文称“四种职责”，但表格列出了五行（含 Status）；结合下文“Status is fixed”一节可知，Status 在该方法中被视为独立于四种数据编码职责之外的固定刻度，表格只是将它一并列出。

## The six checks / 六项检查

Every categorical color - current or proposed - must pass all six.

每一个类别型颜色——无论是现有的还是拟议的——都必须通过全部六项。

1. **Fixed hue anchors.** Eight families in a fixed order. The order is the CVD-safety mechanism; it never changes. *(structural - enforced, not measured)*

   1. **固定色相锚点。**八个色相族按固定顺序排列。这一顺序本身就是色觉缺陷（CVD）安全机制；它永不改变。*（结构性检查——靠约束执行，而非测量）*
2. **Lightness band per mode.** OKLCH L ~ 0.43-0.77 light; ~ 0.48-0.67 dark. *(validator)*

   2. **每种模式各有明度区间。**浅色模式下 OKLCH L 约 0.43-0.77；深色模式约 0.48-0.67。*（校验器检查）*
3. **Chroma floor.** OKLCH C >= ~0.10 - below it a hue reads as gray and stops doing
   identity work. *(validator)*

   3. **彩度下限。**OKLCH C >= 约 0.10——低于该值时色相会被读成灰色，不再承担身份区分工作。*（校验器检查）*
4. **CVD separation.** Delta E here and everywhere in this method is Euclidean distance
   in OKLab ×100. Target >= 8 / floor >= 6 (floor legal only with secondary encoding),
   under protanopia & deuteranopia simulated with Machado-Oliveira-Fernandes 2009 at
   severity 1.0 - the thresholds are calibrated to that simulation model, so the
   model is part of the standard, not an implementation detail. A companion
   **normal-vision floor** gates the same pairs under unsimulated vision: worst
   pair Delta E >= 15, so neighbors stay easy to tell apart for full-color readers too.
   This floor is a hard gate - secondary encoding does not excuse it.
   (This floor is what forced the first of the July 2026 re-orders of the
   documented default palette - same hues and steps, re-ordered; the current
   default clears it at 19.6 light / 19.3 dark; see `palette.md`.)
   *Adjacent* pairs for
   stacks/bars/lines (only neighbors touch - assignment never skips); **all pairs for
   scatter, bubble, choropleth, and small-multiples**, where any two marks can sit side
   by side - pass `--pairs all` there or a real collapse stays hidden. All-pairs is
   a strictly harder test, and it caps how many series those chart forms can carry:
   the documented default validates all-pairs with its **first three slots** in both
   modes, and no ordering of the full eight can pass (the all-pairs pairlist doesn't
   depend on order). More than three series in an all-pairs form means fewer series
   (fold to "Other") or facets - not a palette change. *(validator)*

   4. **CVD 区分度。**本方法中所有地方的 Delta E 均指 OKLab 空间中的欧氏距离 ×100。目标 >= 8 / 下限 >= 6（只有在提供次级编码时下限才合法），在以 Machado-Oliveira-Fernandes 2009 模型、严重度 1.0 模拟的 protanopia（红色盲）与 deuteranopia（绿色盲）下测得——阈值是针对该模拟模型校准的，因此该模型本身是标准的一部分，而非实现细节。配套的**正常视觉下限**对同一组颜色对在未模拟视觉下设置门槛：最差颜色对 Delta E >= 15，使全色觉读者也能轻松区分相邻颜色。该下限是硬性门槛——次级编码不能豁免。（正是这道下限促成了 2026 年 7 月对文档默认调色板的第一次重排——色相与阶梯不变，仅重新排序；当前默认值在浅色模式 19.6 / 深色模式 19.3 通过该门槛；参见 `palette.md`。）对堆叠图/柱状图/折线图使用*相邻*颜色对（只有相邻者接触——分配绝不跳槽）；对散点图、气泡图、分级统计图（choropleth）与小组图（small-multiples）则要求**全部颜色对**，因为任意两个标记都可能并排出现——这些图型必须传 `--pairs all`，否则真实的塌陷会被隐藏。全对比是严格更难的测试，也限制了这些图型能承载的系列数：文档默认调色板在两种模式下都以**前三个槽位**通过全对比校验，而完整八色的任何排序都无法通过（全对比的颜色对列表与顺序无关）。在全对比图型中超过三条系列就意味着减少系列（折叠为 "Other"）或使用分面——而不是更换调色板。*（校验器检查）*
5. **Contrast vs surface.** >= 3:1 for marks; conditionally relaxed where values are
   readable another way (visible labels or the table view). *(validator)*

   5. **与表面的对比度。**标记需 >= 3:1；当数值可以通过其他方式读取（可见标签或表格视图）时有条件放宽。*（校验器检查）*
6. **Documented palette only.** Every slot is a hex from the instance file
   (`palette.md` or its equivalent) - no eyeballed values. *(structural; for a
   customer's ramps, snap to nearest - below)*

   6. **只允许使用文档化调色板。**每个槽位都必须取自实例文件（`palette.md` 或其等价物）中的十六进制值——不允许目测取值。*（结构性检查；对客户的色阶则吸附到最近档——见下文）*

【评论】第 4 项明确把色觉模拟模型（Machado-Oliveira-Fernandes 2009）纳入标准本身，这是一种使阈值可复现的工程化做法：换用其他 CVD 模拟模型时，同一组阈值未必等价。

## Run the checks - never eyeball them / 运行检查——绝不靠目测

```
node scripts/validate_palette.js \
  "#2a78d6,#eb6834,#1baf7a,#eda100,#e87ba4,#008300,#4a3aa7,#e34948" --mode light
```

(`scripts/` is relative to this skill's base directory, shown at the top of the prompt.)

（`scripts/` 相对于本技能的基础目录，即提示词顶部所示的目录。）

(or load it as `<script type="module">` in the chart's own page - it reads
`data-palette` off `<body>` and logs a `console.table` report)

（或者在图表所在页面中以 `<script type="module">` 加载它——它会从 `<body>` 读取 `data-palette`，并输出 `console.table` 报告。）

Reports each computable check (2-5) with PASS / WARN / FAIL plus the worst CVD pair.
Exit 0 = no hard FAIL (WARN bands - floor-band CVD 6-8 and sub-3:1 contrast
relief - still exit 0 and require secondary encoding); exit 1 on any FAIL,
including a normal-vision floor below 15, which is a hard gate. Run once per mode
(`--mode dark --surface "#1a1a19"`), and add
`--pairs all` for scatter / bubble / map / small-multiples charts (where any two marks
can be neighbors - the default adjacent check would hide a collapse). For an
**ordinal** ramp pass `--ordinal` - it switches to the ramp checks (monotone L,
adjacent delta L >= 0.06, light-end contrast >= 2.0:1, single hue) instead of the
categorical six.
A WARN on CVD (6-8 floor) is legal **only** if you also ship secondary encoding
(direct labels, gaps, or texture). A FAIL on the normal-vision floor says
full-color readers will struggle to tell the flagged neighbors apart.
On the *adjacent* pairlist, re-step one of the pair; secondary encoding does
not excuse this one. On `--pairs all`, a floor FAIL over many series is the
series cap binding (check 4): cut the series count, facet, or switch chart
form - re-ordering or re-stepping cannot make eight colors pairwise-distinct
at this floor. A WARN on contrast is **not dismissable** - it
obligates a relief channel (visible direct labels or the table view); shipping the
sub-3:1 fill with neither is a fail.

对每项可计算的检查（2-5）给出 PASS / WARN / FAIL 结果，并列出最差的 CVD 颜色对。退出码 0 表示没有硬性 FAIL（WARN 区间——下限区间 CVD 6-8 与低于 3:1 的对比度豁免——仍退出 0，但要求提供次级编码）；出现任何 FAIL 则退出码为 1，包括正常视觉下限低于 15 的情况，那是硬性门槛。每种模式各运行一次（`--mode dark --surface "#1a1a19"`），并为散点图 / 气泡图 / 地图 / 小组图添加 `--pairs all`（这些图型中任意两个标记都可能相邻——默认的相邻检查会掩盖塌陷）。对**顺序型**色阶传 `--ordinal`——它会切换为色阶检查（L 单调、相邻 ΔL >= 0.06、浅端对比度 >= 2.0:1、单一色相），而不是类别型六项。CVD 上的 WARN（6-8 下限区间）**只有**在同时提供次级编码（直接标签、间隙或纹理）时才合法。正常视觉下限上的 FAIL 表示全色觉读者将难以区分被标记的相邻颜色。在*相邻*颜色对列表上，应对该颜色对中的一方重新调整阶梯；这一项次级编码不能豁免。在 `--pairs all` 下，多系列时的下限 FAIL 说明系列数上限已经生效（检查 4）：应削减系列数、使用分面或更换图型——重新排序或重新调阶梯都无法让八种颜色在该下限下两两可区分。对比度上的 WARN **不可忽略**——它要求提供补救通道（可见的直接标签或表格视图）；两者皆无的低对比填充即视为失败。

**Scope - what the validator does and doesn't cover.** These six checks validate a
*categorical* palette (series identity). They do **not** judge a lone status/text
color or a sequential ramp. For a single status or text color, run a WCAG *text*-
contrast check (4.5:1 normal, 3:1 large) - `validate_palette.js` exports
`contrast(a, b)` for exactly this. For sequential/diverging, the check is lightness
monotonicity across the ramp, not adjacency CVD - running the categorical validator on
a sequential ramp **will FAIL by design** (it spans the band; steps sit close), which
is expected, not a real failure; don't "fix" a good ramp to satisfy it.

**适用范围——校验器覆盖与不覆盖的内容。**这六项检查校验的是*类别型*调色板（系列身份）。它们**不**评判单独的状态色/文字色或连续色阶。对单一状态色或文字色，应运行 WCAG *文字*对比度检查（普通文字 4.5:1，大号文字 3:1）——`validate_palette.js` 正是为此导出了 `contrast(a, b)`。对连续型/发散型，检查内容是整个色阶的明度单调性，而非相邻 CVD——把类别型校验器跑在连续色阶上**会按设计 FAIL**（它跨越明度区间；各档彼此接近），这是预期行为，不是真实失败；不要为了迎合它而“修复”一条好的色阶。

## Snap-to-passing (any design system) / 吸附到达标（适用于任何设计系统）

Given a customer's ramps and a desired order:

给定客户的色阶和期望的顺序：

1. For each slot, pick the step whose OKLCH L sits in the mode's band and C >= floor.

   1. 对每个槽位，挑选 OKLCH L 落在该模式区间内且 C >= 下限的那一档。
2. Run the validator. For any adjacent pair below the Delta E 8 target, nudge one slot
   ± a step (hold its hue, move its lightness) and re-run.

   2. 运行校验器。对任何低于 Delta E 8 目标的相邻颜色对，将其中一个槽位上下移动一档（保持色相，调整明度），再重新运行。
3. Repeat until the worst adjacent pair clears the floor. Function preserved, the
   customer's hues kept.

   3. 重复上述过程，直到最差的相邻颜色对越过下限。功能得以保留，客户的色相也得以保留。

## Themes / 主题

The slot **order** is a separable, named choice - a *theme* - on the same hues and
the same six checks. Each design system names a default order and any alternates;
swapping themes tunes the mood without touching the method. A surface adopts one
theme and freezes it; never mix themes within a dashboard. (See `palette.md`.)

槽位**顺序**是一个可分离的、有名字的选择——即*主题*——建立在相同色相与相同六项检查之上。每个设计系统会指定一个默认顺序及若干备选；更换主题只调整气质，不触动方法本身。一个产品面采用一个主题并冻结它；绝不在同一个仪表盘内混用主题。（参见 `palette.md`。）

**Deriving an order when a system has no theme yet:** don't guess. Enumerate candidate
orderings of the system's hues, run the validator on each, and pick the one that
maximizes the *minimum adjacent* CVD Delta E. (Seeding from a known-good order by hue-family
analogy, then optimizing, is fine - the default in `palette.md` came out of
exactly that enumeration, as one of the tied top orders under the gates,
picked among them for its opening.)

**当系统尚无主题时如何推导顺序：**不要靠猜。枚举该系统色相的所有候选排序，逐一运行校验器，选出使*最小相邻*CVD Delta E 最大化的那个。（从已知良好的顺序按色相族类比取得种子、再行优化是可以的——`palette.md` 中的默认值正是出自这样的枚举：它是各门槛下并列最优的排序之一，再依其开头几色的搭配从中选定。）

## Status is fixed / 状态色固定不变

Status never follows the theme - it is a small fixed scale (good -> warning -> serious
-> critical) with reserved meaning, on steps deliberately distinct from the categorical
slots so a status color never impersonates a series, and always paired with an
icon + label (on a light surface warning and serious sit below 3:1 by design -
the pairing is the mitigation). (Exact steps in `palette.md`.) The collision rule: when a series *means* good/bad (error rate, pass/fail) it wears
status tokens; when it's just "series 4" it wears categorical - never both in one chart.

状态色从不跟随主题——它是一个小型固定刻度（良好 -> 警告 -> 严重 -> 危急），含义保留，所选档位刻意与类别型槽位拉开距离，使状态色永远不会冒充某条系列，并且始终与图标 + 标签成对出现（在浅色表面上，警告与严重两档按设计低于 3:1——成对出现即是缓解手段）。（具体档位见 `palette.md`。）冲突规则：当一条系列*表达*好坏含义（错误率、通过/失败）时，它使用状态色 token；当它只是“第 4 条系列”时，它使用类别色——绝不在同一张图表中混用两者。
