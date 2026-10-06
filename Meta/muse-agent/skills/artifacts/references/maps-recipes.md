---
description: Use the bundled map helpers for URL generation, interactive maps, and data overlays.
---
<!-- BILINGUAL-EN-ZH -->

# Map helpers / 地图辅助工具

Follow [Maps and places](maps.md) for rendering and sourcing requirements.
The implementation is in `/opt/hatch/skills/artifacts/scripts/map_helpers.mjs`.
Use it instead of copying or rewriting URL and map-setup code.

渲染与数据来源方面的要求遵循 [Maps and places](maps.md)。实现位于 `/opt/hatch/skills/artifacts/scripts/map_helpers.mjs`。请直接使用它，不要复制或重写 URL 与地图配置代码。

## Place lookups / 地点查询

Use the existing `/opt/hatch/bin/local-search` and `/opt/hatch/bin/places` CLIs.
Set the task's absolute `project_dir`, expanding `~/` with `$JARVIS_HOME/`.
Save details to `.src/research/places.json` for files or
`client/src/research/places.json` for web artifacts. Pass the required flags from
[Resolve places](maps.md#resolve-places), redirect stdout to the file, and collect
command completion before reading it. An explicit `yield_ms: 60000` is useful;
collect the background result if the command outlives it.

使用现成的 `/opt/hatch/bin/local-search` 和 `/opt/hatch/bin/places` 命令行工具。设置任务的绝对 `project_dir`，并把 `~/` 展开为 `$JARVIS_HOME/`。将详细信息保存到 `.src/research/places.json`（文件类任务）或 `client/src/research/places.json`（Web 制品）。传入 [Resolve places](maps.md#resolve-places) 所列的必需标志，将 stdout 重定向到文件，并在读取之前等待命令完成。显式设置 `yield_ms: 60000` 很有用；如果命令超过该时长仍在运行，收集其后台结果。

## Location data / 位置数据

A `PlaceRef` has a required `label` and optional `venue`, `address`, `locality`,
`region`, `country`, `lat`, `lng`, and `source`. Coordinates are named numbers;
address fields are strings. Store facts and derive provider URLs with the helpers.

`PlaceRef` 包含必需的 `label`，以及可选的 `venue`、`address`、`locality`、`region`、`country`、`lat`、`lng` 和 `source`。坐标是有名称的数字；地址字段是字符串。存储事实数据，并使用辅助工具推导服务商 URL。

## Static map URL / 静态地图 URL

Run the URL script with a JSON file of options:

用一个 JSON 选项文件运行 URL 脚本：

```bash
bun /opt/hatch/skills/artifacts/scripts/map_urls.mjs static map-options.json
```

Required: `center: [lat, lng]` and `zoom`. Optional: `width`, `height`, `scale`,
`format`, `theme`, task-supplied `language` and `region`, and  
`markers: [{position: [lat, lng], scale: 1 | 2 | 3}]`.  
The script supplies the caller and attribution parameters. It prints the URL;
fetch the image at build time and render the stored copy.

必需：`center: [lat, lng]` 和 `zoom`。可选：`width`、`height`、`scale`、`format`、`theme`、任务提供的 `language` 与 `region`，以及  
`markers: [{position: [lat, lng], scale: 1 | 2 | 3}]`。  
脚本会补充调用方与署名参数。它打印 URL；构建时抓取图像并渲染存储的副本。

For circles and paths, import `buildStaticMapUrl`, then append one `circles` or
`paths[]` parameter per shape. The pipe-delimited grammar is optional
`color:0xRRGGBBAA`, `fillcolor:0xRRGGBBAA`, `weight:N`, `dash:N`, or `gap:N`,
followed by coordinates. A circle ends with a radius such as `500m`, `1k`, `1mi`,
or `2000ft`; a path lists its points. Use translucent fill alpha.

对于圆和路径，导入 `buildStaticMapUrl`，然后为每个形状追加一个 `circles` 或 `paths[]` 参数。管道分隔的语法是可选的 `color:0xRRGGBBAA`、`fillcolor:0xRRGGBBAA`、`weight:N`、`dash:N` 或 `gap:N`，后接坐标。圆以半径结尾，如 `500m`、`1k`、`1mi` 或 `2000ft`；路径则列出其各点。填充使用半透明 alpha。

## Interactive map / 交互式地图

Copy all three files from `/opt/hatch/skills/artifacts/map-runtime/dist/` into
the artifact's assets and load them as classic scripts. A static page may not
load a module, so use the generated `map_helpers.js` there, not the `.mjs`:

把 `/opt/hatch/skills/artifacts/map-runtime/dist/` 中的全部三个文件复制到制品的 assets 中，并以经典脚本方式加载。静态页面可能无法加载模块，因此在该场景使用生成的 `map_helpers.js`，而不是 `.mjs`：

```html
<link rel="stylesheet" href="assets/hatch-maps.css">
<script src="assets/hatch-maps.js"></script>
<script src="assets/map_helpers.js"></script>
```

```js
const map = MapHelpers.mountMap(el, { places, baseStyle: 'light' }, showPlaceList);
```

In a TypeScript Space, copy `map_helpers.mjs` into `client/src/` and import from
it instead. The names below are the same either way: on a static page reach them
through `MapHelpers`, in a Space import them.

在 TypeScript Space 中，把 `map_helpers.mjs` 复制到 `client/src/` 并从那里导入。以下名称在两种方式下相同：静态页面上通过 `MapHelpers` 访问，Space 中直接导入。

`mountMap` checks capabilities and connects
fatal errors to the required `onUnavailable` callback (the third argument). It
returns the control's map handle or `null`. Supply the callback yourself: show the
place list, or the data as a table or chart. The helper does not create page UI.

`mountMap` 会检查能力，并将致命错误接到必需的 `onUnavailable` 回调（第三个参数）上。它返回控件的地图句柄或 `null`。回调要自己提供：显示地点列表，或以表格或图表展示数据。辅助工具不会创建页面 UI。

For a short place list, call `mountPlaceListMap(el, {places, rows}, showPlaceList)`.
`rows` is an array of DOM elements in the same order as `places`. The helper
numbers pins, synchronizes clicks and highlights, scrolls to selected rows, and
clears selection. Style `.pin`, `.pin--on`, and `.map-selected`; change the last
name with `selectedClass`. Mount once per container and list.

对于简短的地点列表，调用 `mountPlaceListMap(el, {places, rows}, showPlaceList)`。`rows` 是与 `places` 顺序相同的 DOM 元素数组。辅助工具会为图钉编号、同步点击与高亮、滚动到选中行并清除选择。为 `.pin`、`.pin--on` 和 `.map-selected` 设置样式；最后一个类名可用 `selectedClass` 更改。每个容器和列表只挂载一次。

## Data overlays / 数据叠加层

`heatmapOverlay(id, geoJSON, colorStops, radius)` returns an overlay for `mountMap`.
`colorStops` alternates density thresholds and colors; include transparency at
zero and derive colors from the page. Radius defaults to 20. Pass it in
`overlays`, choose `baseStyle: 'grayscale'`, and use a chart or table callback.

`heatmapOverlay(id, geoJSON, colorStops, radius)` 返回一个供 `mountMap` 使用的叠加层。`colorStops` 交替给出密度阈值和颜色；在零值处使用透明，并从页面配色推导颜色。radius 默认为 20。把它传入 `overlays`，选择 `baseStyle: 'grayscale'`，并使用图表或表格回调。

Other encodings use the control's normal `overlays` and `tooltip` options.
Overlays remain below basemap labels; the control's pins remain above them.

其他编码方式使用控件常规的 `overlays` 和 `tooltip` 选项。叠加层保持在底图标注之下；控件的图钉保持在它们之上。

### Choropleth / 分级统计图

Call `mountChoropleth(el, options, showValueTable)`. Required options:

调用 `mountChoropleth(el, options, showValueTable)`。必需选项：

| Option | Value |
|---|---|
| `boundaryUrl` | A published geometry URL below |
| `rows` | Source records containing region identifiers and rates |
| `featureKey`, `rowKey`, `valueKey` | Geometry join key, row join key, and rate field |
| `colorStops` | Alternating rate thresholds and colors from the page palette |
| `outlineColor` | Outline color from the same palette |

| 选项 | 取值 |
|---|---|
| `boundaryUrl` | 下文已发布的几何数据 URL |
| `rows` | 包含区域标识符和比率的数据源记录 |
| `featureKey`、`rowKey`、`valueKey` | 几何连接键、行连接键和比率字段 |
| `colorStops` | 来自页面配色的比率阈值与颜色交替序列 |
| `outlineColor` | 同一配色中的轮廓颜色 |

The helper checks capabilities, fetches the geometry, joins values, and mounts
with grayscale, value tooltips, and the same callback for fetch or map failure.
Missing values remain `null` and are excluded from the fill; zero stays visible.
Add a legend explaining missing values and the scale.

辅助工具会检查能力、获取几何数据、连接数值，并以灰度底图、数值提示框以及同一个回调（用于抓取或地图失败）完成挂载。缺失值保持为 `null` 且不参与填充；零值保持可见。添加一个图例，说明缺失值与色标。

Hover text is the page's, so write it in the artifact's language. The default
line is `name: value`; pass `missingLabel` for the words a region with no value
shows, or `tooltip`, a function of the feature, to replace the line outright.

悬停文本属于页面，因此要用制品的语言书写。默认行是 `name: value`；为无数值区域显示的文字传入 `missingLabel`，或传入 `tooltip`（以要素为参数的函数）来整体替换该行。

| Geometry | URL | Feature keys |
|---|---|---|
| Countries | `https://external.xx.fbcdn.net/maps/static/boundaries/v1/countries.json` | `iso_2`, `iso_3` |
| US states | `https://external.xx.fbcdn.net/maps/static/boundaries/v1/us_states.json` | `postal_code`, `iso_3166` |

| 几何数据 | URL | 要素键 |
|---|---|---|
| 国家 | `https://external.xx.fbcdn.net/maps/static/boundaries/v1/countries.json` | `iso_2`、`iso_3` |
| 美国各州 | `https://external.xx.fbcdn.net/maps/static/boundaries/v1/us_states.json` | `postal_code`、`iso_3166` |

【评论】边界几何数据托管在 Meta 自己的 CDN（fbcdn.net）域名下，说明该地图能力依赖公司内部或关联基础设施，而非公开第三方服务。

## Navigation links / 导航链接

Call `mapsSearchUrl(place)` or `mapsDirectionsUrl(destination, options)` in the
page. `options` accepts an actual `origin: {lat, lng}` and `travelmode` of
`driving`, `walking`, `bicycling`, or `transit`. The helpers return a URL or `null`
when no location is available. Open links in a new tab.

在页面中调用 `mapsSearchUrl(place)` 或 `mapsDirectionsUrl(destination, options)`。`options` 接受实际的 `origin: {lat, lng}`，以及 `driving`、`walking`、`bicycling` 或 `transit` 的 `travelmode`。当没有可用位置时，这些辅助工具返回 `null`。在新标签页中打开链接。

For build-time generation, run `map_urls.mjs search place.json`, or
`map_urls.mjs directions route.json` with `{"destination": PlaceRef, "options": ...}`.
Use `bun /opt/hatch/skills/artifacts/scripts/map_urls.mjs --help` for CLI usage.

构建时生成可运行 `map_urls.mjs search place.json`，或运行 `map_urls.mjs directions route.json` 并配合 `{"destination": PlaceRef, "options": ...}`。命令行用法见 `bun /opt/hatch/skills/artifacts/scripts/map_urls.mjs --help`。
