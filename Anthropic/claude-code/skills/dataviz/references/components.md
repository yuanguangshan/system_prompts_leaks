<!-- BILINGUAL-EN-ZH -->
# Components - the pieces a chart is made of / 组件——构成图表的各个部分

A chart is built from these parts, assembled in plain HTML/SVG. Tier 0 is the
foundation everything mounts on; the System tier is what makes the method
portable (and is, itself, this skill).

图表由以下部分构成，以纯 HTML/SVG 组装。第 0 层是一切的挂载基础；System 层则让这套方法具备可移植性（它本身就是这个技能）。

## Tier 0 - Foundations / 第 0 层——基础

- **Color roles** - categorical (8 × light/dark), sequential ramps, diverging pairs,
  status (4), de-emphasis / "Other", grayscale chart furniture (axis/grid/label/surface).
  Defined as CSS custom properties at the top of the HTML - see `palette.md`.
  - **颜色角色** - 分类色（8 × 浅色/深色）、顺序色带、发散色对、状态色（4 种）、弱化 / "其他"色、灰度图表结构件（坐标轴/网格/标签/底面）。以 CSS 自定义属性的形式定义在 HTML 顶部——参见 `palette.md`。
- **Texture fill** - the directional fill + 45°/135° rotations.
  - **纹理填充** - 方向性填充 + 45°/135° 旋转。
- **Chart container** - a `<figure>` (or card `<div>`) that owns responsive
  sizing, title/caption, and the **table-view toggle** (the accessibility twin
  of every chart). **Any fixed height includes the x-axis band** (plot height
  + axis labels) so the card never gets a nested vertical scroll; prefer
  letting the container grow with its content.
  - **图表容器** - 一个 `<figure>`（或卡片 `<div>`），负责响应式尺寸、标题/说明文字，以及**表格视图切换**（每张图表的无障碍孪生版本）。**任何固定高度都必须包含 x 轴区域**（绘图高度 + 轴标签），使卡片永远不会出现嵌套的垂直滚动；优先让容器随内容自然增长。
- **Legend** (toggle-to-isolate, texture-aware swatches) · **Tooltip** · **Axis** · **Data label**.
  - **图例**（点击切换以隔离显示、纹理感知色块）· **工具提示** · **坐标轴** · **数据标签**。

## Tier 1 - The charts people ask for / 第 1 层——用户最常要求的图表

- **Bar chart** - grouped + stacked, thin-bar default, horizontal + vertical.
  - **柱状图** - 分组 + 堆叠，默认细柱，横 + 纵两个方向。
- **Line chart** - multi-series, soft-fill area variant, accessibility markers.
  - **折线图** - 多系列、软填充面积变体、无障碍标记。
- **Stat tile** - value + delta + optional sparkline (the figure contract).
  - **统计卡片** - 数值 + 变化量 + 可选迷你走势图（figure 契约）。
- **Meter / progress track** - same-ramp tracks.
  - **量表 / 进度条** - 使用同一色带的轨道。

## Tier 2 - Rounding out the kit / 第 2 层——补全工具箱

- **Area chart** (stacked, band-edge = line) · **Sparkline** · **Heatmap**
  - **面积图**（堆叠式，带边缘 = 折线）· **迷你走势图** · **热力图**
- **Scale legend** (sequential / diverging) · **Chart filters / time range** · **Empty state**
  - **标尺图例**（顺序 / 发散）· **图表筛选器 / 时间范围** · **空状态**

## System tier - becomes the skill / System 层——最终成为技能本身

- **Six-checks validator** - `scripts/validate_palette.js` (palette validation).
  - **六项检查校验器** - `scripts/validate_palette.js`（调色板校验）。
- **Theming engine** - snap a customer's ramps to passing values (color-formula.md).
  - **主题引擎** - 将客户的色带吸附到可通过校验的取值上（color-formula.md）。
- **Chart-type heuristic** - pick the form (choosing-a-form.md).
  - **图表类型启发式** - 选择合适的形式（choosing-a-form.md）。
- **Table-view generator** - the WCAG-clean equivalent of any chart.
  - **表格视图生成器** - 任何图表的符合 WCAG 的等价表格。

Notes: part-to-whole rides on the stacked bar chart; donut stays deprioritized.
Small multiples is a layout pattern over these, not a separate piece. Scatter
joins Tier 2 if scatter-heavy surfaces land.

说明：部分与整体（part-to-whole）关系依托堆叠柱状图实现；环形图保持低优先级。小型多图（small multiples）是叠加在这些组件之上的一种布局模式，而非独立组件。如果散点密集型界面落地，散点图将加入第 2 层。
