---
name: maps-geography
description: "Accurate maps from real geo data — use for any map, or whenever geography would make a good graphic for a deliverable"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Maps & geography / 地图与地理

Geographic maps are data problems, not drawings: never freehand country outlines, coastlines, or street layouts — hand-drawn geography is reliably wrong, and users notice. Load real geometry and render it.

地理地图是数据问题，而不是绘画问题：绝不要徒手勾勒国家轮廓、海岸线或街道布局——手绘的地理形状必然是错的，而且用户看得出来。应当加载真实几何数据并进行渲染。

Build every map page as plain HTML — a .html file with ordinary <script> tags, NEVER a .dc.html Design Component, even when every other design in the project is one: DC confines scripts to <helmet>, whose mount timing races the map container — the same call the data-viz and 3D skills make.

每个地图页面都要构建为普通 HTML——即带常规 <script> 标签的 .html 文件，绝不要用 .dc.html 设计组件（Design Component），即便项目里其他设计都是 DC 也不例外：DC 把脚本限制在 <helmet> 中，其挂载时机与地图容器存在竞态——这与数据可视化技能和 3D 技能的要求一致。

For decks, docs, graphics, and animations — anything static or exported — render TopoJSON geometry with d3-geo: fetch https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json (Natural Earth data, public domain; the URL is version-pinned — use it exactly), convert with topojson.feature(topology, topology.objects.countries), and draw with d3.geoPath() under a projection chosen for the job (d3.geoNaturalEarth1 for the whole world; d3.geoMercator().fitSize(...) to zoom a region). d3-geo ships inside the d3 bundle below. Load the libraries ONLY through these exact pinned, hash-verified tags, in <head>. These tags fail closed if tampered with; any other script you add would load unverified — so do not change versions, URLs, or hashes, and add nothing else from a CDN:

对于幻灯片、文档、图形和动画——任何静态或导出的产物——用 d3-geo 渲染 TopoJSON 几何数据：获取 https://cdn.jsdelivr.net/npm/world-atlas@2.0.2/countries-110m.json （Natural Earth 数据，公有领域；该 URL 锁定了版本——必须原样使用），用 topojson.feature(topology, topology.objects.countries) 转换，并在为任务选定的投影下用 d3.geoPath() 绘制（全球用 d3.geoNaturalEarth1；缩放某区域用 d3.geoMercator().fitSize(...)）。d3-geo 包含在下方的 d3 捆绑包中。只能通过下面这些精确锁定且经哈希校验的标签、在 <head> 中加载这些库。这些标签一旦被篡改就会加载失败（fail closed）；你另行添加的任何脚本都无法得到校验——因此不要更改版本、URL 或哈希，也不要从 CDN 添加任何其他内容。

【评论】通过子资源完整性（SRI）哈希锁定 CDN 资源，属于供应链安全设计：被篡改的脚本会因哈希不匹配而拒绝加载。

<script src="https://unpkg.com/d3@7.9.0/dist/d3.min.js" integrity="sha384-CjloA8y00+1SDAUkjs099PVfnY2KmDC2BZnws9kh8D/lX1s46w6EPhpXdqMfjK6i" crossorigin="anonymous"></script>
<script src="https://unpkg.com/topojson-client@3.1.0/dist/topojson-client.min.js" integrity="sha384-Ukv1p/xTma6P4/2bY5KzWBw+ydSpXmhCMtyciIQVDJ1RmOxtCYNMF1uXT9T63H67" crossorigin="anonymous"></script>

Inline SVG from d3 also exports cleanly to PNG and PDF, which live map tiles do not — so exported deliverables always get d3 geometry, never an embedded tile map.

由 d3 生成的内联 SVG 还能干净地导出为 PNG 和 PDF，而实时地图瓦片做不到这一点——因此导出的交付物一律使用 d3 几何数据，绝不内嵌瓦片地图。

For street-level interactive maps — prototypes, websites, anything the user pans and zooms — use Leaflet with OpenStreetMap tiles, loaded ONLY through these exact tags (the stylesheet is required: without leaflet.css the tiles render scrambled):

对于街道级交互地图——原型、网站、任何需要用户平移和缩放的东西——使用 Leaflet 配 OpenStreetMap 瓦片，且只能通过下面这些精确的标签加载（样式表必不可少：没有 leaflet.css，瓦片会渲染错乱）：

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha384-sHL9NAb7lN7rfvG5lfHpm643Xkcjzp4jFvuavGOndn6pjVqS6ny56CAt3nsEVT4H" crossorigin="anonymous">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha384-cxOPjt7s7Iz04uaHJceBmS+qpjv2JkIHNVcuOrM+YHwZOmJGBXI00mdUXEq65HTH" crossorigin="anonymous"></script>

Create the map with L.map(...) and L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap contributors' }). The attribution string is OpenStreetMap's license requirement — never omit it.

用 L.map(...) 和 L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '© OpenStreetMap contributors' }) 创建地图。该署名字符串是 OpenStreetMap 的许可证要求——绝不可省略。
