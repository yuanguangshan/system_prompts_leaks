---
name: 3d-object
description: "three.js model, downloadable as OBJ or GLB"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# 3D object / 3D 对象

Model a 3D object the user can inspect from every angle and download,
built with three.js and presented in the three_d_stage starter.

构建一个用户可从任意角度查看并下载的 3D 对象模型，使用 three.js 构建，并以 three_d_stage 起始组件呈现。

START by calling copy_starter_component with kind: "three_d_stage.js" —
it is the whole viewer: studio lighting, ground shadow, orbit controls,
an auto-framed camera, and a toolbar that downloads the object as
OBJ + MTL or GLB. Read the copied file's usage block and follow its page
skeleton exactly. You only write the model-building module script.

首先调用 copy_starter_component，kind 取 "three_d_stage.js"——它就是整个查看器：影棚灯光、地面阴影、轨道控制、自动取景相机，以及一个可将对象下载为 OBJ + MTL 或 GLB 的工具栏。阅读复制出的文件中的用法块，并严格遵循其页面骨架。你只需编写构建模型的模块脚本。

Build the page as plain HTML — a .html file with ordinary <script>
tags — even when the project's other designs are .dc.html Design
Components: DC restricts scripts to <helmet>, which races the stage's
mount and can't host this skeleton.

将页面构建为普通 HTML——一个带普通 <script> 标签的 .html 文件——即使项目中的其他设计是 .dc.html 设计组件（Design Components）：DC 将脚本限制在 <helmet> 内，这会与舞台的挂载产生竞态，也无法承载这个骨架。

Load three.js ONLY through this exact pinned import map, in <head>,
before any module script. Do not change versions, URLs, or hashes, do
not add other copies of three.js, and do not import addons beyond the
three listed — the map is deliberately a closed set, so anything else
fails to resolve rather than loading unverified:

只能通过这份精确固定的 import map 加载 three.js，置于 <head> 中、任何模块脚本之前。不要更改版本、URL 或哈希，不要添加 three.js 的其他副本，也不要导入所列三个插件（addons）之外的插件——这份映射刻意设计为封闭集合，任何其他内容都会解析失败，而不是加载未经校验的代码。

<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.184.0/build/three.module.js",
    "three/addons/controls/OrbitControls.js": "https://unpkg.com/three@0.184.0/examples/jsm/controls/OrbitControls.js",
    "three/addons/exporters/OBJExporter.js": "https://unpkg.com/three@0.184.0/examples/jsm/exporters/OBJExporter.js",
    "three/addons/exporters/GLTFExporter.js": "https://unpkg.com/three@0.184.0/examples/jsm/exporters/GLTFExporter.js"
  },
  "integrity": {
    "https://unpkg.com/three@0.184.0/build/three.module.js": "sha384-8FCZ1eVO6it4+pbec2aDtnTrwjWXZLJRC+MAGCIPDgsYnUrl/E0A2YlF8ioMKI/J",
    "https://unpkg.com/three@0.184.0/build/three.core.js": "sha384-dw2ooPewaEIrAgl6oFDBmmBWCE9oW9LxRGcfwZ0hLvEprzo202wXl7vCYHRlSnOT",
    "https://unpkg.com/three@0.184.0/examples/jsm/controls/OrbitControls.js": "sha384-4rziNxOBZKQ69i+w+f89KJ55TCYquwchVbByQwmaOeIOXdOU2PLDn3kOfXHwIJC9",
    "https://unpkg.com/three@0.184.0/examples/jsm/exporters/OBJExporter.js": "sha384-nbwtoZENJD3Vq+ACK0CuGQdPMuDWHkamC2KJD70EV5nfg6jQjfppKOea07YJN+N3",
    "https://unpkg.com/three@0.184.0/examples/jsm/exporters/GLTFExporter.js": "sha384-VofkvpG6HERhFCYbsUOHeNXBCqID2nfqkQqnVzE1jc/oPcz+qJ13ADdXH08hE+cQ"
  }
}
</script>

Build the model programmatically, as a THREE.Group composed of named
parts:

以编程方式构建模型，作为一个由命名部件组成的 THREE.Group：

- Compose primitives (BoxGeometry, CylinderGeometry, SphereGeometry,
  TorusGeometry, LatheGeometry, ExtrudeGeometry with Shape) before
  reaching for raw BufferGeometry; real objects decompose into far more
  primitives than you'd guess.
  先组合基元（BoxGeometry、CylinderGeometry、SphereGeometry、TorusGeometry、LatheGeometry、带 Shape 的 ExtrudeGeometry），再考虑原始 BufferGeometry；真实物体可分解出的基元远比你以为的多。
- NAME every mesh and every material ("hull", "walnut", "brass") — the
  names become the o / usemtl entries in the exported OBJ and the node
  names in the GLB, which is what makes the download usable in Blender.
  为每个网格和每个材质命名（"hull"、"walnut"、"brass"）——这些名称会成为导出 OBJ 中的 o / usemtl 条目和 GLB 中的节点名，这正是下载结果能在 Blender 中可用的关键。
- Use MeshStandardMaterial with a small curated palette (3-5 materials,
  shared across parts); set roughness/metalness deliberately. Textures
  don't survive the OBJ export — prefer geometry and material color over
  texture detail.
  使用 MeshStandardMaterial 配一个小而精的调色板（3-5 种材质，跨部件共享）；有意识地设置 roughness/metalness。纹理无法在 OBJ 导出后保留——优先用几何与材质颜色，而非纹理细节。
- Model in real-world meters, y-up, centered on the origin, base resting
  at the lowest y. Offset deliberately coplanar faces by ~0.001 so
  nothing z-fights.
  以真实世界的米为单位建模，y 轴向上，以原点为中心，底部落在最低的 y 处。对刻意共面的面做约 0.001 的偏移，避免任何 z-fighting。
- Curved surfaces need enough segments to read as smooth at full screen
  (32+ radial segments on feature surfaces), but don't tessellate what
  no one will see.
  曲面需要足够的分段才能在全屏下显得平滑（特征表面 32+ 个径向分段），但不要对没人会看的地方做细分。

The stage's toolbar gives the user OBJ + MTL (universal, geometry +
per-material colors) and GLB (the modern interchange format — keeps the
part hierarchy and PBR materials; imports cleanly into Blender, Maya,
Cinema 4D, Unity, Unreal). Those two are the export formats on offer —
when the user asks for something else (FBX, USDZ, STEP), say plainly
that the stage exports OBJ + MTL and GLB.

舞台的工具栏为用户提供 OBJ + MTL（通用格式，几何体 + 每材质颜色）和 GLB（现代交换格式——保留部件层级与 PBR 材质；可干净地导入 Blender、Maya、Cinema 4D、Unity、Unreal）。这两者就是可选的导出格式——当用户要求其他格式（FBX、USDZ、STEP）时，直说舞台只能导出 OBJ + MTL 和 GLB。

Iterate with screenshots: the stage keeps its last frame readable, so
the ordinary screenshot tools capture the live canvas — no extra steps.
After editing a module file, reload with show_html before screenshotting
(the iframe caches already-loaded modules). Look at the object from the
default framing and refine silhouette, proportion, and material
separation — the silhouette carries the object.

通过截图迭代：舞台会保持最后一帧可读，因此普通截图工具即可捕捉活动画布——无需额外步骤。编辑模块文件后，先用 show_html 重新加载再截图（iframe 会缓存已加载的模块）。从默认取景观察对象，打磨轮廓、比例和材质区分——轮廓决定对象的观感。
【评论】该技能对 three.js 的版本、URL 与 SRI 完整性哈希做了锁定，是防止依赖被篡改的供应链安全措施；封闭的 import map 则进一步保证不会意外加载未校验的脚本。
