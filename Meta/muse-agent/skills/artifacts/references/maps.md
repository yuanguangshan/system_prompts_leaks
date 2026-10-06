<!-- BILINGUAL-EN-ZH -->
---
description: Choose and verify Meta maps, resolve places, present geographic data, and build navigation links.
builders: web, file
---

# Maps and places / 地图与地点

Read this when an artifact shows places, maps, routes, directions, or geographic
data. Use the bundled scripts described in [Map helpers](maps-recipes.md) for URL
generation and map setup.

当产物要展示地点、地图、路线、导航或地理数据时阅读本文。URL 生成与地图设置请使用 [Map helpers](maps-recipes.md) 中描述的内置脚本。

## Choose the rendering approach / 选择渲染方式

Meta's map services render every map. Use the bundled Meta control for interactive
maps and Meta's static endpoint for map images. Do not substitute another renderer
or tile source, including vendored libraries or downloaded tile pyramids. Satellite
and aerial imagery are unavailable. Preserve the required attribution below.

所有地图都由 Meta 的地图服务渲染。交互式地图使用内置的 Meta 控件，地图图像使用 Meta 的静态端点。不得替换为其他渲染器或瓦片源，包括内置的第三方库或下载的瓦片金字塔。卫星与航空影像不可用。须保留下文要求的署名信息。

| Surface and data | What to show |
|---|---|
| Live web page with trusted coordinates | An interactive vector map. If the control is unavailable or cannot render, show the place list without a map. |
| Fixed-layout export or headless rasterization with trusted coordinates | A stored static map image with the place list beside it. |
| Places without trusted coordinates | A place list with maps search links, without a map. |

| 承载面与数据 | 应展示的内容 |
|---|---|
| 带可信坐标的实时网页 | 交互式矢量地图。若控件不可用或无法渲染，则只展示地点列表、不放地图。 |
| 带可信坐标的固定版式导出或无头栅格化 | 一张存储的静态地图图像，旁边附地点列表。 |
| 没有可信坐标的地点 | 带地图搜索链接的地点列表，不放地图。 |

A live page uses a vector map or no map; a static image is not its fallback.
Missing coordinates do not justify choosing a different renderer or inventing
location data. Static maps take `[lat, lng]`; GeoJSON and interactive map coordinates
use `[lng, lat]`. Keep latitude and longitude named and convert at the boundary.

实时页面要么用矢量地图、要么不放地图；静态图像不是它的回退选项。缺少坐标不构成更换渲染器或编造位置数据的理由。静态地图使用 `[lat, lng]`；GeoJSON 与交互式地图坐标使用 `[lng, lat]`。保持经纬度命名清晰，并在边界处做转换。

The second row is different on a confidential VM; see
[Exports on a confidential VM](#exports-on-a-confidential-vm).

在机密虚拟机上，第二行有所不同；参见 [Exports on a confidential VM](#exports-on-a-confidential-vm)。

For geographic data, choose a map when location carries the meaning, such as
adjacency, distance, or clustering. Use a chart for comparisons and rankings; a
page can include both. See [Data maps](#data-maps) for supported encodings.

对地理数据而言，当位置本身承载含义（如相邻、距离或聚集）时选用地图。比较与排名用图表；一页可以两者兼有。支持的编码方式见 [Data maps](#data-maps)。

## Resolve places / 解析地点

Resolve places through Meta's places graph when it is reachable. Use web search
for candidate names and editorial context; it does not replace place resolution.
For a resolved venue, take its facts and media from the graph record, including on
later edits. Do not fill missing fields from review sites, web images, or stock
photos.

在可达时通过 Meta 的地点图谱解析地点。网络搜索用于候选名称与编辑性背景；它不能替代地点解析。对已解析的场所，其事实与媒体一律取自图谱记录，后续编辑亦然。不要用点评网站、网络图片或图库照片填补缺失字段。

Run the bundled `local-search` and `places details` CLIs through `exec`:

通过 `exec` 运行内置的 `local-search` 与 `places details` CLI：

- For a well-scoped request, use `local-search --search-type discovery`. Start with
  a broad query that matches the request. Add queries only for different facets,
  at most six. Put the area in `--location`, use one call per area, and add
  `--radius` when the request states or implies a distance bound.
  对范围明确的需求，使用 `local-search --search-type discovery`。先发出一个与需求匹配的宽泛查询。只为不同侧面追加查询，最多六个。区域放在 `--location` 中，每个区域一次调用；当需求明示或暗示距离边界时加上 `--radius`。
- For a best, insider, or trending request, find candidate names with web search,
  then resolve each with `local-search --search-type known_place`.
  对"最佳"、"内行推荐"或"热门"类需求，先用网络搜索找出候选名称，再用 `local-search --search-type known_place` 逐一解析。
- Fetch details in a batch with `--motivation` describing the artifact's purpose.
  Always pass `--product-source IG --product-source FB`; other sources can return
  review-site quotes and media. Bound `--num-quotes` and `--num-gallery-media` to
  what the artifact will show.
  批量获取详情，并用 `--motivation` 说明产物的用途。始终传 `--product-source IG --product-source FB`；其他来源可能返回点评网站的引文与媒体。`--num-quotes` 与 `--num-gallery-media` 以产物实际会展示的数量为上限。
- Save output to a file and read it after the command finishes. Avoid consumers
  that close the output pipe early. Set an explicit `yield_ms` and collect the
  completed result if the call continues in the background.
  把输出保存到文件，等命令结束后再读取。避免会提前关闭输出管道的消费方。显式设置 `yield_ms`；若调用转入后台继续，稍后收取完成的结果。
- A failed details batch returns no records. Retry its IDs individually and keep
  the successful results. A place whose details remain unavailable can still
  appear with the information already obtained.
  失败的详情批次不返回任何记录。对其 ID 逐个重试并保留成功的结果。详情始终拿不到的地点，仍可带着已获得的信息出现。

Keep lookup outcomes distinct:

区分不同的查询结果：

| Evidence | Place handling |
|---|---|
| Graph resolves the place | Use its coordinates and record. A permanently closed place is not plotted. |
| Graph rejects a candidate you found | Do not name or plot that candidate. |
| User named the place, but resolution fails | Retain it. Plot user-supplied coordinates if present; otherwise list it with a search link. |
| Graph is unreachable | Use the information already held: plot user-supplied coordinates; list names or addresses without coordinates with search links. |

| 证据 | 地点处理方式 |
|---|---|
| 图谱解析出该地点 | 使用其坐标与记录。永久歇业的地点不予绘制。 |
| 图谱否决了你找到的候选 | 不得提名或绘制该候选。 |
| 用户点名了该地点，但解析失败 | 保留它。若有用户提供的坐标则绘制；否则带搜索链接列入清单。 |
| 图谱不可达 | 使用已掌握的信息：绘制用户提供的坐标；没有坐标的名称或地址配搜索链接列入清单。 |

An unreachable lookup is not a rejection. Do not geocode a place name, approximate
from a locality, or invent coordinates to fill a lookup gap. Never drop a
user-named place because a lookup missed. These place-resolution rules do not
replace the source data for geographic datasets and overlays.

查询不可达不等于被否决。不要对地名做地理编码、不要从街区近似推算、也不要编造坐标来填补查询空缺。绝不要因为查询未命中就丢弃用户点名的地点。这些地点解析规则不取代地理数据集与叠加层的源数据。

【评论】把"查不到"与"不存在/被否决"区分开、并禁止用地理编码自行补坐标，是防止模型编造位置数据的典型约束。

Save the details payload under the task's `project_dir`: `.src/research/places.json`
for file artifacts or `client/src/research/places.json` for web artifacts. Reuse it
on later edits or fetch details again. It is build-time input: never import the
payload into the page. Inline only the fields used and store images locally.

把详情载荷保存在任务的 `project_dir` 下：文件产物用 `.src/research/places.json`，网页产物用 `client/src/research/places.json`。后续编辑时复用它，或重新获取详情。它是构建期输入：绝不要把载荷导入页面。只内联用到的字段，图像在本地存储。

## Present place data / 呈现地点数据

Store structured location facts using `PlaceRef` from the helper reference and
derive links from them. Keep display labels separate from source coordinates. Store the street
line in `address`, the city in `locality`, and the state in `region`; do not repeat
address parts. Preserve coordinate provenance where the artifact depends on it.
Never invent addresses, neighborhoods, coordinates, or drive times.

使用助手参考中的 `PlaceRef` 存储结构化位置事实，并从它们派生链接。展示标签与源坐标分开保存。街道行存入 `address`，城市存入 `locality`，州存入 `region`；不要重复地址组成部分。当产物依赖坐标来源时保留其溯源信息。绝不编造地址、街区、坐标或车程时间。

- Select useful fields for the artifact: address, hours, price level, rating and
  count, category, description, known-for items, features, booking links, or photos.
  Chat-reply limits on displaying these fields do not apply to artifacts.
  为产物挑选有用的字段：地址、营业时间、价位、评分与评论数、类目、描述、知名特色、设施、预订链接或照片。聊天回复中对这些字段展示的限制不适用于产物。
- When the payload has suitable review quotes, include one per place with the
  author's handle and a link to the original. Preserve the quote; do not copy the
  author's avatar.
  当载荷中有合适的评论引文时，每个地点收录一条，附作者 handle 与原文链接。保持引文原样；不要复制作者头像。
- Check the language of returned strings. Reword categories in the page's language;
  omit quotes whose language you cannot verify. Do not translate review quotes.
  检查返回字符串的语言。类目用页面语言改写；无法核实语言的引文直接舍弃。不要翻译评论引文。
- Report ratings, counts, and hours as returned. Show scheduled hours rather than
  a build-time "open now" claim or other stale live state.
  评分、评论数与营业时间按返回原样报告。展示排定的营业时间，而不是构建时刻的"正在营业"断言或其他会过时的实时状态。
- Download place photos at build time and render stored copies. Link to reels in
  a new tab rather than embedding them; [Social embeds](social-embeds.md) applies.
  地点照片在构建期下载并渲染本地存储的副本。Reels 用新标签页链接而非内嵌；适用 [Social embeds](social-embeds.md)。

## Static maps / 静态地图

Use the [static URL script](maps-recipes.md#static-map-url), which supplies the
required caller and attribution parameters. Fetch the image at build time and store it in
`.src/media/` for file artifacts. Where web asset storage is needed, use
`client/src/assets/` or `ctx.blobs`. Never ship the static endpoint as an image `src`.

使用 [static URL script](maps-recipes.md#static-map-url)，它会提供必需的 caller 与署名参数。在构建期获取图像，文件产物存入 `.src/media/`。需要网页资产存储时，用 `client/src/assets/` 或 `ctx.blobs`。绝不把静态端点直接当作图像 `src` 发布。

- Center on the points' bounding box and choose a zoom that contains them; the
  static service has no fit-bounds operation.
  以各点的包围盒为中心并选择能容纳它们的缩放级别；静态服务没有 fit-bounds 操作。
- Request `scale: 2` for high-DPI display and set the image's width and height to
  the unscaled dimensions.
  高 DPI 显示请求 `scale: 2`，并把图像的宽高设置为未缩放的尺寸。
- Give the image descriptive alt text and keep the place list beside it.
  为图像写描述性 alt 文本，并在旁边保留地点列表。
- Static markers carry no text. Circles and paths use the grammar in the helper
  reference; use translucent fill alpha so the underlying map remains visible.
  静态标记不带文字。圆与路径使用助手参考中的语法；填充 alpha 用半透明，使底图保持可见。
- If the build-time image fetch fails, show the place list without a map.
  若构建期图像获取失败，则只展示地点列表、不放地图。
- Preserve the baked-in credit and add the notices link in [Attribution](#attribution).
  保留烧录在图中的署名，并按 [Attribution](#attribution) 添加声明链接。

## Exports on a confidential VM / 机密虚拟机上的导出

A static map request carries its markers in the URL, so asking for the image
tells the service where the reader's places are. On a confidential VM that is
the one recipient the VM exists to keep this data from, and an attested
transport would not change it, so do not call the static endpoint there at all.

静态地图请求把标记点带在 URL 里，因此请求图像就等于告诉服务读者的地点在哪里。在机密虚拟机上，地图服务恰恰是这台虚拟机要防止其接触该数据的那一方，即便走可证明的传输通道也不会改变这一点，因此在机密虚拟机上完全不要调用静态端点。

【评论】这是一条由威胁模型直接推出的约束：请求 URL 本身就是泄露通道，目标服务正是数据要隔离的对象，故整条调用路径被禁止而非加密了事。

Draw the export's map in the guest instead, from coordinates or GeoJSON the
build already holds, with the same deterministic plotting the charts use. Fetch
no basemap, no tiles and no rendered map image, and do not substitute a
third-party renderer or tile service to make up the difference. The result is
plainer than a Meta basemap; that is the trade. Published boundary geometry is
still available, because that request carries no reader location of its own.

改为在 guest 内绘制导出地图，使用构建已持有的坐标或 GeoJSON，采用与图表相同的确定性绘制。不获取底图、瓦片或任何渲染好的地图图像，也不要用第三方渲染器或瓦片服务来弥补。结果会比 Meta 底图朴素；这就是代价。已发布的边界几何仍可使用，因为该请求本身不携带读者位置。

The rest of this file still holds. Do not invent coordinates to fill the
drawing, keep the place list beside it, and where there are no trusted
coordinates the third row still applies: a place list and no map.

本文件其余规则仍然有效。不要为绘图编造坐标，旁边保留地点列表；没有可信坐标时第三行仍然适用：只有地点列表、不放地图。

## Interactive maps / 交互式地图

Use `/opt/hatch/skills/artifacts/map-runtime/dist/hatch-maps.js` and
`hatch-maps.css`. Confirm both exist, copy them into the artifact's assets, and
reference the local copies. A build-VM path is not a served URL. Do not install
another renderer, load one from a public CDN, or import `@meta/maps` directly.
Give the map container a resolved height.

使用 `/opt/hatch/skills/artifacts/map-runtime/dist/hatch-maps.js` 与 `hatch-maps.css`。确认两者存在，复制进产物资产并引用本地副本。构建虚拟机路径不是可访问的 URL。不要安装其他渲染器、不要从公共 CDN 加载、也不要直接 import `@meta/maps`。给地图容器一个确定的（resolved）高度。

Use the [mount helpers](maps-recipes.md#interactive-map), which check capabilities
and route fatal errors to your `onUnavailable` callback. That callback shows
"Map unavailable" with a place list, or a table or chart of the values for a data
map. Boundary-data fetch failures use the same callback. Leave a map mounted
after non-fatal tile, sprite, or glyph errors.

使用 [mount helpers](maps-recipes.md#interactive-map)，它们会检查能力并把致命错误路由到你的 `onUnavailable` 回调。该回调展示"Map unavailable"加地点列表，或为数据地图展示数值的表格/图表。边界数据获取失败也走同一回调。非致命的瓦片、sprite 或字形错误发生时，地图保持挂载。

Choose `baseStyle` by name: `light` (default), `dark`, or `grayscale`. The runtime
owns style URLs, caller identifiers, and viewer locale. Do not pass a style URL or
set `locale` or `pv`. Do not add custom headers to map-resource requests: the map
servers do not support the resulting CORS preflight.

按名称选择 `baseStyle`：`light`（默认）、`dark` 或 `grayscale`。样式 URL、caller 标识与查看者区域设置（locale）由运行时管理。不要传样式 URL，也不要设置 `locale` 或 `pv`。不要给地图资源请求添加自定义头：地图服务器不支持由此触发的 CORS 预检。

Keep overlays beneath basemap labels and the control's pin layers above overlays.
The runtime supplies these defaults. If authoring a symbol layer, keep icons while
thinning colliding labels with `text-allow-overlap: false` and `text-optional: true`;
place it last so its labels take priority over basemap labels.

叠加层保持在底图标注之下，控件的图钉层在叠加层之上。运行时提供这些默认值。若自行编写符号层，用 `text-allow-overlap: false` 与 `text-optional: true` 在保留图标的同时稀疏化冲突标注；把它放在最后，使其标注优先于底图标注。

For a map with a place list:

对带地点列表的地图：

- Number pins to match rows using `marker` for lists of a few dozen places. Escape
  any source text inserted as HTML. Use the default circle layer for larger lists
  and overlays for thousands of points.
  用 `marker` 让图钉编号与行号对应，适用于几十个地点的清单。任何以 HTML 插入的源文本都要转义。更大的清单用默认圆形图层，数千个点用叠加层。
- Connect `onSelectPlace` to row highlighting and scrolling, and row clicks to
  `selectPlace`. Clear the selection when the callback receives `null`.
  把 `onSelectPlace` 接到行高亮与滚动，把行点击接到 `selectPlace`。回调收到 `null` 时清除选中状态。

Read the [interactive helpers](maps-recipes.md#interactive-map) for these APIs.

这些 API 详见 [interactive helpers](maps-recipes.md#interactive-map)。

## Data maps / 数据地图

Use GeoJSON sources with layer specifications through `overlays` on the vector
map, built by the [data overlay helpers](maps-recipes.md#data-overlays). The
static tier supports only markers, circles, and paths; choose from those
capabilities for fixed-layout maps.

在矢量地图上通过 `overlays` 使用带图层规格的 GeoJSON 源，由 [data overlay helpers](maps-recipes.md#data-overlays) 构建。静态层只支持标记、圆和路径；固定版式地图从这些能力中选择。

| Encoding | Layer type | Data |
|---|---|---|
| Choropleth | `fill` | Rates or ratios for enumeration units, with a meaningful value across the represented area |
| Proportional or graduated symbols | `circle` | Counts or totals; scale symbol area with the value |
| Heatmap | `heatmap` | Density of many points |
| Dot density | Small `circle` symbols | One dot per item or per stated quantity |
| Category points | `circle` with a `match` expression | Unordered classes |
| Outlines and routes | `line` | Boundaries and paths |

| 编码方式 | 图层类型 | 数据 |
|---|---|---|
| 分级统计图（Choropleth） | `fill` | 枚举单位的比率或比例，且该值在所表示的区域内有意义 |
| 比例符号或分级符号 | `circle` | 计数或总量；符号面积随数值缩放 |
| 热力图 | `heatmap` | 大量点的密度 |
| 点密度图 | 小型 `circle` 符号 | 每个项目一个点，或每个声明的数量一个点 |
| 类别点 | 带 `match` 表达式的 `circle` | 无序类别 |
| 轮廓与路线 | `line` | 边界与路径 |

Do not shade raw counts as a choropleth or treat absence of a phenomenon as a low
value. Use point encodings for phenomena that exist only at specific locations.
Scale proportional symbols with `r = k * sqrt(value)` and show two or three
reference sizes in the legend. Graduated symbols may group values into classes.
A dot-density legend must say whether a dot represents one item or a quantity.

不要把原始计数画成分级统计图，也不要把现象缺席当作低数值。只存在于特定位置的现象用点编码。比例符号按 `r = k * sqrt(value)` 缩放，并在图例中展示两三个参考尺寸。分级符号可以把数值分组为等级。点密度图的图例必须说明一个点代表一个项目还是一个数量。

For choropleths, use the [choropleth helper](maps-recipes.md#choropleth), which
fetches the published geometry, joins the data onto its features, and mounts the
result. Join countries on `iso_2` or `iso_3` and US states on `postal_code` or
`iso_3166`. Keep missing values as `null`, leave those regions unpainted, and
explain them in the legend.

分级统计图使用 [choropleth helper](maps-recipes.md#choropleth)，它会获取已发布的几何数据、把数据连接到要素上并挂载结果。国家按 `iso_2` 或 `iso_3` 连接，美国州按 `postal_code` 或 `iso_3166` 连接。缺失值保持 `null`，这些区域不上色，并在图例中说明。

Use a `grayscale` basemap under data overlays. Choose a sequential palette for
ordered values, a diverging palette around a meaningful midpoint, or a categorical
palette for unordered classes. Derive hues from the page's palette while preserving
legible distinctions; choose a nearby hue if the page accent cannot support the
required range. Do not use rainbow ramps. See [Charts](charts.md) for shared
encoding guidance.

数据叠加层下使用 `grayscale` 底图。有序数值选用顺序（sequential）调色板，围绕有意义中点的数据选用发散（diverging）调色板，无序类别选用分类（categorical）调色板。色相从页面调色板派生，同时保持可辨识的差异；若页面强调色撑不起所需范围，选邻近色相。不要使用彩虹色带。共享编码指南见 [Charts](charts.md)。

Give overlays that carry values a `tooltip` showing the relevant values, and say
a region has no data in the artifact's own language rather than leaving the
helper's English default. A data map that cannot render shows its values as a
table or chart, not a place list.

承载数值的叠加层要配置 `tooltip` 展示相关数值；"该区域无数据"要用产物自身语言表述，而不是保留助手的英文默认值。无法渲染的数据地图把数值展示为表格或图表，而不是地点列表。

## Language and political view / 语言与政治视角

Political view controls disputed borders. Do not infer it from language or choose
a default.

政治视角（political view）控制争议边界的画法。不要从语言推断它，也不要擅自选默认值。

| Rendering | Parameters |
|---|---|
| Interactive | Leave `locale` and `pv` unset so the runtime resolves them per reader. |
| Static | Pass `language` (IETF tag) and `region` (political-view ccTLD) unchanged only when supplied by the task or context; omit missing values. |

| 渲染方式 | 参数 |
|---|---|
| 交互式 | 保持 `locale` 与 `pv` 不设置，由运行时按读者解析。 |
| 静态 | 仅当任务或上下文提供了 `language`（IETF 标签）与 `region`（政治视角 ccTLD）时原样传递；缺失的值直接省略。 |

【评论】地图的政治视角是产品层面的合规参数：同一地理数据在不同视角下争议边界画法不同，因此被要求显式提供而不是让模型推断。

## Navigation links / 导航链接

Use Google Maps only for the outbound search and directions handoff, with neutral
labels such as "Open in maps" or "Directions". Use the
[navigation builders](maps-recipes.md#navigation-links).

Google Maps 只用于向外的搜索与路线交接，标签保持中性，如"Open in maps"或"Directions"。使用 [navigation builders](maps-recipes.md#navigation-links)。

- Show a resolved place's facts in the artifact instead of a maps search link.
  Unresolved or name-only places retain that link. Directions links remain useful
  for navigation from the reader's current location.
  已解析地点的事实直接在产物中展示，而不是给地图搜索链接。未解析或只有名称的地点保留该链接。路线链接对于从读者当前位置出发的导航仍然有用。
- Prefer a name plus street address or locality; use coordinates when a name has
  no locating detail. Deduplicate address parts.
  优先"名称 + 街道地址或区域"；名称没有定位细节时用坐标。地址组成部分要去重。
- If a name matches multiple listings, accept the chooser. Query the street address
  alone only when the named query would lead to the wrong place.
  若名称匹配多个条目，接受其选择器。仅当按名称查询会导向错误地点时，才单独用街道地址查询。
- Pass a real coordinate pair for `origin`, or omit it so the maps app can request
  the device location. Never pass a label such as "Your location".
  `origin` 传真实坐标对，或省略它让地图应用自行请求设备位置。绝不传"Your location"这类标签。
- Keep coordinate commas literal in generated URLs and encode text queries.
  生成的 URL 中坐标逗号保持字面量，文本查询做编码。
- Open outbound links with `target="_blank" rel="noopener"` so the destination
  opens outside the artifact's frame.
  向外链接使用 `target="_blank" rel="noopener"` 打开，使目的地在产物框架之外打开。

## Attribution / 署名

Leave the interactive control's attribution visible and intact. Do not hide,
cover, shrink, or replace it.

交互式控件的署名保持可见且完整。不要隐藏、遮盖、缩小或替换它。

For static maps, keep `show_attribution=1` and leave the baked-in credit uncropped.
Render this notices link beneath the image:

静态地图保持 `show_attribution=1`，烧录署名不裁切。在图像下方渲染这个声明链接：

```html
<a href="https://www.facebook.com/maps/attribution_terms"
   target="_blank" rel="noopener">© OpenStreetMap contributors, Map Data Legal Notices</a>
```

A map drawn in the guest has no baked-in credit to keep, so render that same
notices link beneath it whenever it draws published boundary geometry. A drawing
made only from the build's own coordinates credits nothing.

在 guest 内绘制的地图没有烧录署名可保留，因此只要它绘制了已发布的边界几何，就在其下方渲染同一声明链接。仅用构建自有坐标绘制的图形无需任何署名。

## Verify / 校验

- Check the rendered artifact: correct places and coordinate order, readable data,
  descriptive image alt text, visible attribution, and useful content without a map.
  检查渲染后的产物：地点与坐标顺序正确、数据可读、图像 alt 文本具描述性、署名可见、无地图时内容仍然有用。
- Confirm the surface uses the required map approach. For interactive maps, check
  that both control files load, the map has height, and the probe and fatal-error
  handler use the same replacement. Exercise the available viewing surfaces.
  确认承载面使用了规定的地图方案。交互式地图要检查两个控件文件都能加载、地图有高度、探针与致命错误处理使用同一个替代展示。可用的查看承载面都要实际运行一遍。
- Check that no alternative renderer or tile source is shipped. Tile, style, and
  glyph requests use `external.xx.fbcdn.net`; select basemaps by name.
  检查没有携带任何替代渲染器或瓦片源。瓦片、样式与字形请求使用 `external.xx.fbcdn.net`；底图按名称选择。
- For static maps, inspect the stored image, required caller parameters, task-supplied
  locale values, uncropped credit, and notices link. No remote image `src`.
  静态地图要检查存储的图像、必需的 caller 参数、任务提供的 locale 值、未裁切的署名与声明链接。不得有远程图像 `src`。
- On a confidential VM, confirm the export drew its own map and that the build
  made no static-endpoint, tile, or basemap request at all.
  机密虚拟机上要确认导出自行绘制了地图，且构建全程没有发起任何静态端点、瓦片或底图请求。
- Match resolved-place facts, quotes, and photos to the retained payload. Check
  search links against the resolution rules rather than rejecting every search URL.
  已解析地点的事实、引文与照片要与保留的载荷一致。按解析规则检查搜索链接，而不是一概否决所有搜索 URL。
- Check data joins, missing values, encoding, palette, legends, tooltips, layer
  ordering, and the table or chart shown when a data map cannot render.
  检查数据连接、缺失值、编码方式、调色板、图例、tooltip、图层顺序，以及数据地图无法渲染时展示的表格或图表。
- Exercise map/list selection in both directions, including clearing it.
  地图/列表双向选中都要实际操作，包括清除选中。
- Open a navigation link and verify its destination, new tab, query encoding,
  coordinate commas, and omitted or coordinate-backed origin.
  打开一个导航链接，验证其目的地、新标签页、查询编码、坐标逗号，以及省略或以坐标兜底的 origin。
