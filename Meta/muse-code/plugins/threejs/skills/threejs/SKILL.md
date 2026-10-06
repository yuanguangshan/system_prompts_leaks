---
name: threejs
description: Three.js 3D reference hub. Use when working with Three.js: pick the topic below and read its file for patterns and API guidance.
source: https://github.com/CloudAI-X/threejs-skills
license: MIT
---

<!-- BILINGUAL-EN-ZH -->

# Three.js

This skill is a hub: one catalog row pointing at ten topic references.
When the task touches a topic, read that topic's file in one hop: join
its `references/threejs-<topic>.md` path against the `skill-dir` value in
this body's wrapper header and pass the absolute path to `read_file`.

此技能是一个中心（hub）：一行目录指向十个主题参考文件。当任务涉及某个主题时，一步读取该主题的文件：将它的 `references/threejs-<topic>.md` 路径与本正文包装头（wrapper header）中的 `skill-dir` 值拼接，并把绝对路径传给 `read_file`。

- `references/threejs-fundamentals.md` — scene setup, cameras, renderer, Object3D hierarchy, coordinate systems.
  `references/threejs-fundamentals.md` — 场景设置、相机、渲染器、Object3D 层级、坐标系。
- `references/threejs-geometry.md` — built-in shapes, BufferGeometry, custom geometry, instancing.
  `references/threejs-geometry.md` — 内置几何体、BufferGeometry、自定义几何体、实例化。
- `references/threejs-materials.md` — PBR, basic, phong, shader materials, material properties.
  `references/threejs-materials.md` — PBR、基础材质、phong、着色器材质及材质属性。
- `references/threejs-lighting.md` — light types, shadows, environment lighting.
  `references/threejs-lighting.md` — 灯光类型、阴影、环境光照。
- `references/threejs-textures.md` — texture types, UV mapping, environment maps, texture settings.
  `references/threejs-textures.md` — 纹理类型、UV 映射、环境贴图、纹理设置。
- `references/threejs-animation.md` — keyframe and skeletal animation, morph targets, animation mixing.
  `references/threejs-animation.md` — 关键帧与骨骼动画、变形目标（morph targets）、动画混合。
- `references/threejs-loaders.md` — GLTF, textures, images, models, async loading patterns.
  `references/threejs-loaders.md` — GLTF、纹理、图像、模型、异步加载模式。
- `references/threejs-shaders.md` — GLSL, ShaderMaterial, uniforms, custom effects.
  `references/threejs-shaders.md` — GLSL、ShaderMaterial、uniforms、自定义效果。
- `references/threejs-postprocessing.md` — EffectComposer, bloom, depth of field, screen effects.
  `references/threejs-postprocessing.md` — EffectComposer、泛光（bloom）、景深、屏幕特效。
- `references/threejs-interaction.md` — raycasting, controls, mouse/touch input, object selection.
  `references/threejs-interaction.md` — 射线拾取（raycasting）、控制器、鼠标/触摸输入、对象选择。
