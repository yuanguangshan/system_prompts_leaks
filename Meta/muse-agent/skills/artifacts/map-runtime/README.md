<!-- BILINGUAL-EN-ZH -->

# Artifact map runtime / 制品地图运行时

Bundles `@meta/maps` into a standalone script that a generated web artifact can
load with two plain tags. Artifacts have no bundler and no React, so they cannot
use the component API directly.

将 `@meta/maps` 打包为一个独立脚本，生成的 Web 制品只需两个普通标签即可加载。制品没有打包器，也没有 React，因此无法直接使用组件 API。

Built output is vendored into the bundle at `/opt/hatch/skills/artifacts/map-runtime/dist/`:

构建产物以 vendored 方式放入 bundle 的 `/opt/hatch/skills/artifacts/map-runtime/dist/` 路径：

| File | Purpose |
|---|---|
| `hatch-maps.js` | IIFE exposing `window.HatchMaps` — `mountMap` and `isMetaMapSupported` |
| `hatch-maps.css` | MapLibre control and canvas styles |

| 文件 | 用途 |
|---|---|
| `hatch-maps.js` | IIFE，暴露 `window.HatchMaps` — `mountMap` 和 `isMetaMapSupported` |
| `hatch-maps.css` | MapLibre 控件与画布样式 |

**Both are required.** Loading the script without the stylesheet stacks the
tiles on top of each other, so the map looks broken rather than missing.

**两个文件都是必需的。**只加载脚本而不加载样式表会让瓦片彼此堆叠，地图看起来是显示错乱而非缺失。

## Usage from an artifact / 在制品中的用法

Copy both files into the artifact's own assets first. The bundle path is a
filesystem location on the build VM; a published artifact is served from its own
origin, so referencing `/opt/hatch/...` from the page 404s and leaves
`HatchMaps` undefined.

先把两个文件复制进制品自己的 assets。bundle 路径是构建虚拟机上的文件系统位置；已发布的制品从其自身源（origin）提供内容，因此页面中引用 `/opt/hatch/...` 会返回 404，并使 `HatchMaps` 保持未定义。

```bash
cp /opt/hatch/skills/artifacts/map-runtime/dist/hatch-maps.{js,css} <assets>/
```

```html
<link rel="stylesheet" href="assets/hatch-maps.css">
<script src="assets/hatch-maps.js"></script>
<div id="map" style="height: 420px"></div>
<script>
  HatchMaps.mountMap(document.getElementById('map'), {
    clientId: 'artifact_web',
    places: [
      {label: 'Chaotic Coffee', lat: 47.6588, lng: -117.426},
      {label: 'Mobius Discovery Center', lat: 47.6575, lng: -117.4231},
    ],
    onFatalError: () => {
      document.getElementById('map').textContent = 'Map unavailable';
    },
  });
</script>
```

The container needs a resolved height. A map in a zero-height element renders
nothing and reports no error.

容器需要有确定的高度。放在高度为零的元素中的地图不会渲染任何内容，也不会报错。

`onFatalError` is not optional in practice. It fires for the map control's own
fatal errors and for anything thrown while the map is being constructed — a
render-phase throw is caught by a boundary inside `mountMap` rather than
escaping to the window. Surfaces differ in what they let a map do, and the same
bundle renders normally on some and not others, so give the callback a real
fallback — the place list, or a short "Map unavailable" block — because a build
cannot tell in advance which surface it will get.

`onFatalError` 实际上并非可选。地图控件自身的致命错误以及地图构建过程中抛出的任何异常都会触发它——渲染阶段的抛出会被 `mountMap` 内部的边界捕获，而不会逃逸到 window。不同页面对地图的容许程度不同，同一个 bundle 在某些页面能正常渲染，在另一些则不能，所以要给回调一个真正的兜底方案——地点列表，或一个简短的"地图不可用"区块——因为构建时无法预知会落在哪种页面上。

`_nc_client_caller` is fixed at `Muse_Artifact`; pass the artifact kind as
`clientId`, which becomes `_nc_client_id`. Both are wire identifiers that
maps-side dashboards group by, so do not invent per-artifact values.

`_nc_client_caller` 固定为 `Muse_Artifact`；把制品种类作为 `clientId` 传入，它会成为 `_nc_client_id`。两者都是地图侧仪表盘用于分组的线上标识符，因此不要为单个制品杜撰取值。

`locale` and `politicalView` are overrides, and the example leaves them out on
purpose. The style, glyph and tile requests leave the reader's own browser, so
the maps backend resolves the values correct for whoever is reading; setting
them pins one view of every disputed border onto every reader of the artifact.
Set them only to correct a resolution observed to be wrong.

`locale` 和 `politicalView` 是覆盖项，示例有意省略了它们。样式、字形和瓦片请求从读者自己的浏览器发出，因此地图后端会为每位读者解析出适合其环境的取值；设置它们会把某一种争议边界视图强加给制品的每一位读者。只有当观察到解析结果确实错误时才设置它们。

【评论】politicalView 参数控制争议边界的政治视角渲染，该条款把默认视角的选择权留给读者侧环境，是地缘政治敏感内容常见的处理方式。

`rtlTextPluginUrl` defaults to `false`, so MapLibre's RTL text plugin is not
registered and Arabic, Hebrew and Persian labels render mis-shaped and reversed.
The library's own default resolves the vendored asset through `import.meta.url`,
which this build cannot emit — esbuild at `--format=iife` has no asset loader —
so declining is the honest default rather than a silent failure. A page that maps
an RTL-script region and can serve the asset from its own origin passes the URL.

`rtlTextPluginUrl` 默认为 `false`，因此 MapLibre 的 RTL 文本插件不会注册，阿拉伯语、希伯来语和波斯语标签会出现字形错误且顺序颠倒。该库自身的默认行为通过 `import.meta.url` 解析 vendored 资源，而这种构建无法产出——`--format=iife` 模式下的 esbuild 没有资源加载器——因此拒绝注册是诚实的默认值，而不是无声的失败。若页面要映射 RTL 文字地区且能从自身源提供该资源，则传入该 URL。

Attribution is mounted by the map control. It is mandatory — do not hide,
overlay, or reimplement it. See  
`hatch-skills/skills/artifacts/references/maps.md`.

署名信息由地图控件挂载。它是强制性的——不要隐藏、覆盖或重新实现。参见  
`hatch-skills/skills/artifacts/references/maps.md`。

【评论】强制保留地图署名通常是地图服务提供商的许可与合规要求，移除或覆盖该署名可能构成违约。

## Maps do not render in every artifact surface / 地图并非在每种制品页面中都能渲染

A vector map needs WebGL and a Web Worker, and not every surface an artifact
opens on provides them. Ask before mounting:

矢量地图需要 WebGL 和 Web Worker，而制品打开所在的页面并非都提供这两者。挂载前先探测：

```js
const {supported, reason} = HatchMaps.isMetaMapSupported();
if (supported) {
  HatchMaps.mountMap(el, {
    clientId: 'artifact_web',
    places,
    onFatalError: () => renderPlaceListOnly(),
  });
} else {
  renderPlaceListOnly(); // reason is 'webgl' or 'worker'
}
```

**The probe is not the whole story.** A surface can pass it and still refuse the
style, glyph and tile requests once the map has mounted, which arrives through
`onFatalError` rather than through the probe. `onFatalError` is also the backstop
for a lost GL context and for a throw during construction, which `mountMap`
catches rather than letting reach the window. Point both at the same fallback
and the page degrades identically whichever fires. Surface specifics live in
`references/maps.md`, under "Probe the surface before you mount".

**探测并不能说明全部情况。**页面可能通过探测，但在地图挂载后仍然拒绝样式、字形和瓦片请求，这种情况通过 `onFatalError` 而不是探测结果反馈。`onFatalError` 也是 GL 上下文丢失和构建期间抛出异常的兜底，`mountMap` 会捕获这些异常而不让其到达 window。让两者指向同一个兜底方案，无论哪个触发，页面都会以相同方式降级。页面相关的细节在 `references/maps.md` 的"Probe the surface before you mount"一节。

Test a map change on more than one surface -- a build that renders in one place
tells you nothing about the others.

在不止一个页面上测试地图改动——只在一个地方能渲染的构建结果说明不了其他地方的情况。

## Build / 构建

```bash
bun build.mjs
```

`@meta/maps` resolves from Metaccio. Both `.npmrc` and `bunfig.toml` scope only
the `@meta` prefix, so every other JS dependency in this repo keeps resolving
from the public registry.

`@meta/maps` 从 Metaccio 解析。`.npmrc` 和 `bunfig.toml` 都只对 `@meta` 前缀限定作用范围，因此本仓库中其他所有 JS 依赖仍从公共注册源解析。

Metaccio authenticates a corp host by network position — no token. Buildkite
agents qualify, and `bundle-setup/scripts/build.sh` gives the build container
host networking whenever `fwdproxy` resolves. A corp laptop running
`x2pagentd` reaches the same registry over http through `127.0.0.1:10054`
instead; keep those proxy lines out of the committed config, since CI has no
agent daemon listening.

Metaccio 通过网络位置验证公司主机——无需令牌。Buildkite 代理符合条件，且只要 `fwdproxy` 可解析，`bundle-setup/scripts/build.sh` 就会给构建容器启用宿主网络。运行 `x2pagentd` 的公司笔记本则通过 `127.0.0.1:10054` 经 http 访问同一注册源；不要把那些代理配置行提交进配置文件，因为 CI 上没有代理守护进程在监听。

Peer note: this package carries no passenger dependencies, and should not gain
one. `@meta/maps` declares `next` and `@opentelemetry/api` as optional peers and
imports neither at module scope — telemetry reads OTel off the global registry
(`Symbol.for('opentelemetry.js.api.1')`) when something else registered it. The
only reference is a type-only import in `mapTelemetry.d.ts`, which
`skipLibCheck` absorbs; if a change makes that type reachable from `src/`, it
belongs in `devDependencies` and never in `dependencies`.

对等依赖说明：本包不携带任何"顺带"依赖（passenger dependencies），也不应增加。`@meta/maps` 将 `next` 和 `@opentelemetry/api` 声明为可选对等依赖，且不在模块作用域导入它们——当有其他代码注册了遥测时，遥测会从全局注册表（`Symbol.for('opentelemetry.js.api.1')`）读取 OTel。唯一引用是 `mapTelemetry.d.ts` 中的纯类型导入，`skipLibCheck` 会将其吸收；如果某次改动使该类型从 `src/` 变得可达，它应放入 `devDependencies`，绝不能放入 `dependencies`。
