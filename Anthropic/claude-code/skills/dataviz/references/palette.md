<!-- BILINGUAL-EN-ZH -->
# Reference palette / 参考调色板

This is the **reference instance** of the data-viz method: every parameter the
method needs, filled in with a validated default palette. The rest of the skill
is system-agnostic - **to target your brand, substitute this file's values** and
re-run the validator. Nothing else changes.

这是数据可视化方法的**参考实例**：方法所需的每一个参数，都以一套经过验证的默认调色板填好。技能的其余部分与设计系统无关——**要适配你的品牌，替换本文件中的值**并重新运行校验器即可。其余一切不变。

## How to use these values / 如何使用这些值

Everything below is plain hex. In an HTML chart, **define the slots you use as
CSS custom properties in a local `<style>` block** at the top of the file, then
reference them by role throughout - so the light/dark values swap in one place,
and the chart body is written against roles rather than raw hex:

下面的值都是普通十六进制色值。在 HTML 图表中，**把你要用的槽位定义为文件顶部局部 `<style>` 块中的 CSS 自定义属性**，然后全文按角色引用——这样浅色/深色值只需在一处切换，图表主体按角色而非原始色值编写：

```css
.viz-root {
  color-scheme: light;
  --surface-1:      #fcfcfb;   /* chart surface */
  --text-primary:   #0b0b0b;
  --text-secondary: #52514e;
  --series-1:       #2a78d6;   /* categorical slot 1 */
  /* ...only the roles this chart uses */
}
@media (prefers-color-scheme: dark) {
  :root:where(:not([data-theme="light"])) .viz-root {
    color-scheme: dark;
    --surface-1:      #1a1a19;
    --text-primary:   #ffffff;
    --text-secondary: #c3c2b7;
    --series-1:       #3987e5;
  }
}
:root[data-theme="dark"] .viz-root {
  color-scheme: dark;
  --surface-1:      #1a1a19;
  --text-primary:   #ffffff;
  --text-secondary: #c3c2b7;
  --series-1:       #3987e5;
}
```

Declare the dark values under both scopes as above - the media query covers
the OS setting; the `data-theme` scope covers the viewer's theme toggle,
which must win both ways (the `:not(...)` guard lets a light stamp beat
OS-dark; `:where()` keeps the media block below the toggle scope).

如上在两个作用域下都声明深色值——媒体查询覆盖操作系统设置；`data-theme` 作用域覆盖查看者的主题切换，且后者必须双向取胜（`:not(...)` 守卫让"浅色"标记能压过操作系统的深色设置；`:where()` 使媒体查询块的优先级保持在切换作用域之下）。

## Categorical palette / 类别调色板

Both modes are selected. The dark column is the same eight hues stepped for the
dark surface, not a separate palette:

两种模式都是被选定（selected）的。深色列是同样八种色相针对深色表面调整的档位，而不是另一套调色板：

| Slot | Hue | Light | Dark |
|------|-----|-------|------|
| 1 | blue | `#2a78d6` | `#3987e5` |
| 2 | orange | `#eb6834` | `#d95926` |
| 3 | aqua | `#1baf7a` | `#199e70` |
| 4 | yellow | `#eda100` | `#c98500` |
| 5 | magenta | `#e87ba4` | `#d55181` |
| 6 | green | `#008300` | `#008300` |
| 7 | violet | `#4a3aa7` | `#9085e9` |
| 8 | red | `#e34948` | `#e66767` |

| 槽位 | 色相 | 浅色 | 深色 |
|------|-----|-------|------|
| 1 | 蓝（blue） | `#2a78d6` | `#3987e5` |
| 2 | 橙（orange） | `#eb6834` | `#d95926` |
| 3 | 青绿（aqua） | `#1baf7a` | `#199e70` |
| 4 | 黄（yellow） | `#eda100` | `#c98500` |
| 5 | 品红（magenta） | `#e87ba4` | `#d55181` |
| 6 | 绿（green） | `#008300` | `#008300` |
| 7 | 紫（violet） | `#4a3aa7` | `#9085e9` |
| 8 | 红（red） | `#e34948` | `#e66767` |

This order passes every hard gate in both modes on the default *adjacent*
pairlist (stacks, bars, lines): worst adjacent CVD Delta E 9.1 light / 8.4 dark
(OKLab ×100, >=8 target), worst adjacent normal-vision Delta E 19.6 light / 19.3
dark (>=15 floor). Under `--pairs all` (scatter, bubble, choropleth, small
multiples) the full eight cannot clear the floors - with all 28 pairs in
play no ordering can (the pairlist no longer depends on order), and
re-stepping is off the table by the documented-palette rule - so those
chart forms carry a series cap: **the first three slots validate all-pairs
in both modes** (worst pair CVD Delta E 9.2 light / 9.4 dark, normal-vision 24.0
light / 20.9 dark - clear of the CVD warn band). Past three, fold to "Other" or
facet: the fourth slot puts yellow and orange on screen
together, and that pair fails the all-pairs floors (normal-vision 13.7
light; CVD 4.8 dark). Three light-mode slots (magenta, yellow, aqua)
sit below 3:1 contrast on the light surface: the **relief rule** applies (ship
visible direct labels or the table view). The dark steps were chosen for the
dark band (OKLCH L ~ 0.48-0.67, >= 3:1 on the dark surface) and validated as a
set. (Ordering history: adopted July 2026 for its more harmonious opening -
the same eight hues and steps as its predecessor, re-ordered, zero hex
changes. The predecessor validated its first FOUR slots all-pairs, with its
dark run in the 6-8 CVD warn band, so secondary encoding was required there;
this order deliberately trades that fourth slot - yellow now sits beside orange -
for better-looking leading colors. Revisit the trade if yellow<->orange
confusion shows up in real charts with four or more series; undoing it is a
pure re-order.) When you swap in your own ramps, hold your palette to the full
gate.

这个顺序在默认的*相邻*颜色对列表（堆叠图、柱状图、折线图）上，于两种模式下都通过了每一道硬性门槛：最差相邻 CVD Delta E 为浅色 9.1 / 深色 8.4（OKLab ×100，目标 >=8），最差相邻正常视觉 Delta E 为浅色 19.6 / 深色 19.3（下限 >=15）。在 `--pairs all` 下（散点图、气泡图、分级统计图、小组图），完整八色无法越过下限——当全部 28 个颜色对都在考察范围内时，任何排序都不行（颜色对列表不再依赖顺序），而按"只用文档化调色板"规则重新调阶梯也被排除——因此这些图型带有系列数上限：**前三个槽位在两种模式下都通过全对比校验**（最差颜色对 CVD Delta E 为浅色 9.2 / 深色 9.4，正常视觉浅色 24.0 / 深色 20.9——避开了 CVD 警告区间）。超过三条时，折叠为 "Other" 或分面：第四个槽位会让黄色与橙色同屏出现，而该颜色对在全对比下限上不合格（正常视觉浅色 13.7；CVD 深色 4.8）。浅色模式下有三个槽位（品红、黄、青绿）在浅色表面上对比度低于 3:1：适用**补救规则**（提供可见的直接标签或表格视图）。深色档位是为深色区间挑选的（OKLCH L 约 0.48-0.67，在深色表面上 >= 3:1），并作为整体通过校验。（顺序沿革：2026 年 7 月因其更和谐的开头配色而被采用——与前身使用同样的八种色相和档位，仅重新排序，零色值变更。前身通过了前*四个*槽位的全对比校验，但其深色模式落在 6-8 的 CVD 警告区间内，因此在该处要求次级编码；这个顺序有意拿第四个槽位做交换——黄色现在紧邻橙色——换取更好看的开头配色。如果在四条及以上系列的真实图表中出现黄<->橙混淆，再重新审视这一取舍；撤销它只是纯粹的重排序。）换入你自己的色阶时，也要让你的调色板接受完整门槛的检验。

The slot **ordering** is the CVD-safety mechanism, not cosmetic - candidate
orderings were enumerated and only those clearing every adjacent gate in both
modes kept (see `color-formula.md` § Themes); this default is one of the
passing orders, picked among them for its opening colors. When you swap in
your brand's hues, do the same: run the validator on candidate orderings and
choose only among the passing ones.

槽位**排序**是 CVD 安全机制，不是装饰——候选排序被逐一枚举，只有那些在两种模式下都通过所有相邻门槛的才被保留（见 `color-formula.md` § Themes）；这个默认值是通过的排序之一，依其开头几色的搭配从中选定。换入你品牌的色相时也照做：对候选排序运行校验器，只从通过者中挑选。

## Sequential hue / 连续色相

Default single hue: **blue**, light->dark. When two sequential contexts appear at
once, the second takes the next categorical slot's hue (orange), each as its own
one-hue ramp.

默认单一色相：**蓝色**，由浅到深。当同时出现两个连续编码场景时，第二个使用下一个类别槽位的色相（橙色），各自作为自己的单色相色阶。

| step | hex | step | hex | step | hex | step | hex |
|---|---|---|---|---|---|---|---|
| 100 | `#cde2fb` | 250 | `#86b6ef` | 400 | `#3987e5` | 550 | `#1c5cab` |
| 150 | `#b7d3f6` | 300 | `#6da7ec` | 450 | `#2a78d6` | 600 | `#184f95` |
| 200 | `#9ec5f4` | 350 | `#5598e7` | 500 | `#256abf` | 650 | `#104281` |
| | | | | | | 700 | `#0d366b` |

| 档 | 色值 | 档 | 色值 | 档 | 色值 | 档 | 色值 |
|---|---|---|---|---|---|---|---|
| 100 | `#cde2fb` | 250 | `#86b6ef` | 400 | `#3987e5` | 550 | `#1c5cab` |
| 150 | `#b7d3f6` | 300 | `#6da7ec` | 450 | `#2a78d6` | 600 | `#184f95` |
| 200 | `#9ec5f4` | 350 | `#5598e7` | 500 | `#256abf` | 650 | `#104281` |
| | | | | | | 700 | `#0d366b` |

The full 100->700 range is for **sequential** encoding (continuous magnitude -
heatmaps, choropleths) where the lightest step means "near zero" and is allowed
to recede toward the surface. For an **ordinal** ramp (discrete ordered marks -
funnel stages, tiers - validated with `--ordinal`), the step nearest the surface
must still clear 2:1: on light, start no lighter than **step 250** (`#86b6ef`,
2.06:1); on dark, go no darker than **step 600** (`#184f95`, 2.15:1).

完整的 100->700 范围用于**连续型**编码（连续的数量大小——热力图、分级统计图），其中最浅的一档表示"接近零"，允许向表面退隐。对**顺序型**色阶（离散的有序标记——漏斗阶段、层级——用 `--ordinal` 校验），最靠近表面的一档仍必须越过 2:1：浅色模式起点不浅于**第 250 档**（`#86b6ef`，2.06:1）；深色模式终点不深于**第 600 档**（`#184f95`，2.15:1）。

## Diverging pair / 发散色对

**blue <-> red** - warm/cool poles that read as opposite. Neutral midpoint is gray
(light `#f0efec`, dark `#383835`). Equal step count per arm. (blue<->aqua was
rejected - both cool, the midpoint doesn't read as "nothing".)

**蓝 <-> 红**——读起来相反的暖/冷两极。中性中点为灰色（浅色 `#f0efec`，深色 `#383835`）。每臂档数相等。（蓝<->青绿被否决——两者都是冷色，中点读不出"无"。）

## Status palette (fixed - never themed) / 状态色板（固定——绝不随主题）

| role | hex | light-surface contrast | dark-surface contrast |
|---|---|---|---|
| good | `#0ca30c` | 3.27 | 5.19 |
| warning | `#fab219` | 1.79 | 9.49 |
| serious | `#ec835a` | 2.57 | 6.60 |
| critical | `#d03b3b` | 4.68 | 3.62 |

| 角色 | 色值 | 浅色表面对比度 | 深色表面对比度 |
|---|---|---|---|
| 良好（good） | `#0ca30c` | 3.27 | 5.19 |
| 警告（warning） | `#fab219` | 1.79 | 9.49 |
| 严重（serious） | `#ec835a` | 2.57 | 6.60 |
| 危急（critical） | `#d03b3b` | 4.68 | 3.62 |

Dark: same four steps - all clear 3:1 on the dark surface (`#1a1a19`) and remain
distinct from the dark categorical slots. On the light surface, warning and
serious are sub-3:1 by design; the **icon + label** pairing is the mitigation, so
a status color never carries meaning alone. These steps are deliberately distinct
from the categorical slots so a status color never impersonates a series -
distinct enough that nothing collides at a glance, not enough for hue to
carry the distinction unaided: measured by the series floor's own bar
(unsimulated Delta E >= 15), around nine categorical-vs-status pairs per mode sit
below 15 - in light mode red vs critical and yellow vs warning both measure
4.8, slot-2 orange sits 5.8 from status-serious, and the light success text
green `#006300` sits 10.1 from the series green; green vs status-good (9.7)
holds in both modes, since both hexes are mode-invariant. The rule is general: any series color beside a
same-hue-family status or delta cue leans on the icon + label pairing and on
placement; never on hue alone.

深色：同样四档——全部在深色表面（`#1a1a19`）上越过 3:1，且与深色类别槽位保持区分。在浅色表面上，警告与严重两档按设计低于 3:1；**图标 + 标签**的成对出现是缓解手段，因此状态色绝不单独承载含义。这些档位刻意与类别槽位拉开距离，使状态色永远不会冒充某条系列——距离足够近以致没有东西一眼相撞，又不足以让色相在无辅助的情况下承载区分：按系列下限自己的标准（未模拟 Delta E >= 15）衡量，每种模式约有九对类别-状态组合低于 15——浅色模式下红对危急、黄对警告都只有 4.8，槽位 2 的橙色与状态-严重相距 5.8，浅色成功文字绿 `#006300` 与系列绿相距 10.1；绿对状态-良好（9.7）在两种模式下都成立，因为两个色值都不随模式变化。规则是通用的：任何与同色相族的状态或涨跌提示相邻的系列色，都依赖图标 + 标签的成对出现与位置安排，绝不依赖色相本身。

## Texture fill (the accessibility channel) / 纹理填充（无障碍通道）

One hand-drawn **"Lines"** fill, used at **45° and its 135° mirror only**. Inked
tone-on-tone (a darker step of the fill's own ramp). On value scales it is
*ordered* (rotation steps with magnitude; arm angle carries the diverging sign).
Triggered by the accessibility setting, print, or `forced-colors` - never
decorative, never on by default.

一种手绘的**"Lines"**（线条）填充，仅在 **45° 及其 135° 镜像**下使用。以同色调描线（填充自身色阶中更深的一档）。在数值刻度上它是*有序的*（旋转档位随数量递进；臂的角度承载发散符号）。由无障碍设置、打印或 `forced-colors` 触发——绝不作装饰，默认绝不开启。

## Surfaces (for the validator) / 表面色（供校验器使用）

- Light chart surface: `#fcfcfb`
  浅色图表表面：`#fcfcfb`
- Dark chart surface: `#1a1a19`
  深色图表表面：`#1a1a19`

These are the validator's built-in defaults. **When you swap in your own
palette, re-run against your own surfaces:**
`--surface <your-light> --mode light` and `--surface <your-dark> --mode dark` -
contrast and band results are only meaningful against the surface the chart
actually renders on.

这些是校验器的内置默认值。**换入你自己的调色板时，改用你自己的表面重新运行：**
`--surface <your-light> --mode light` 与 `--surface <your-dark> --mode dark`——对比度与区间结果只有针对图表实际渲染的表面才有意义。

## Chart chrome & ink / 图表边框与墨色

| Role | Light | Dark |
|---|---|---|
| Chart surface | `#fcfcfb` | `#1a1a19` |
| Page plane | `#f9f9f7` | `#0d0d0d` |
| Primary ink | `#0b0b0b` | `#ffffff` |
| Secondary ink | `#52514e` | `#c3c2b7` |
| Muted (axis/labels) | `#898781` | `#898781` |
| Gridline (hairline) | `#e1e0d9` | `#2c2c2a` |
| Baseline / axis | `#c3c2b7` | `#383835` |
| Delta up good (success text) | `#006300` | `#0ca30c` |
| Border (hairline ring) | `rgba(11,11,11,0.10)` | `rgba(255,255,255,0.10)` |

| 角色 | 浅色 | 深色 |
|---|---|---|
| 图表表面 | `#fcfcfb` | `#1a1a19` |
| 页面底面 | `#f9f9f7` | `#0d0d0d` |
| 主墨色 | `#0b0b0b` | `#ffffff` |
| 次墨色 | `#52514e` | `#c3c2b7` |
| 弱化色（坐标轴/标签） | `#898781` | `#898781` |
| 网格线（发丝线） | `#e1e0d9` | `#2c2c2a` |
| 基线 / 坐标轴 | `#c3c2b7` | `#383835` |
| 上涨向好（成功文字） | `#006300` | `#0ca30c` |
| 边框（发丝描边） | `rgba(11,11,11,0.10)` | `rgba(255,255,255,0.10)` |

## Filter controls / 筛选控件

Filters are standard UI, not chart components - the chart layer only adds the
composition rules in `interaction.md`. A date-range control is a list of preset
rows (today, last 7/30/90 days, month-to-date) with selection marked by a 16px
bold check, hover as a ghost wash, and custom range behind a hairline in the
footer. Dimension filters are a standard combobox.

筛选器是标准 UI，不是图表组件——图表层只补充 `interaction.md` 中的组合规则。日期范围控件是一列预设行（今天、最近 7/30/90 天、月初至今），选中项以 16px 粗体对勾标记，悬停显示幽灵色淡染，自定义范围藏在页脚一条发丝线下。维度筛选器是标准下拉组合框。

## Typeface & figures / 字体与数字呈现

Everything - including the hero figure - stays in the system sans: `system-ui,
-apple-system, "Segoe UI", sans-serif`. No display or serif face anywhere. Large
standalone numbers (hero figure, stat-tile values) use the default proportional
figures; reserve `font-variant-numeric: tabular-nums` for columns that must align
vertically (table rows, axis ticks). Substitute your brand's UI sans here.

一切——包括大数字（hero figure）——都使用系统无衬线字体：`system-ui,
-apple-system, "Segoe UI", sans-serif`。任何地方都不用展示型或衬线字体。较大的独立数字（大数字、统计瓦片的数值）使用默认的比例数字；把 `font-variant-numeric: tabular-nums` 留给必须纵向对齐的列（表格行、坐标轴刻度）。在这里替换为你品牌的 UI 无衬线字体。
