---
name: animated-video
description: "Timeline-based motion design"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Animated video / 动画视频

Create an animated video or motion design piece rendered as an HTML page. Build a timeline-based animation with smooth transitions. Design frame-by-frame sequences with playback controls (play/pause, scrubber). Focus on visual storytelling with the Anthropic brand palette. Export-ready at a fixed aspect ratio (16:9 or 9:16). If you need to know the position of an element (eg to move a cursor or character between elements) use refs to grab the position.

创建渲染为 HTML 页面的动画视频或动效设计作品。构建带平滑过渡的基于时间线的动画。设计带播放控制（播放/暂停、进度条）的逐帧序列。聚焦于使用 Anthropic 品牌色板的视觉叙事。以固定纵横比（16:9 或 9:16）达到可导出状态。如果需要知道某个元素的位置（例如在元素之间移动光标或角色），使用 ref 获取位置。

ALWAYS build on the `animations_v3.jsx` starter for ANY animation piece — including one hosted on a design-components page (the helmet-script + x-import structure IS the starter case, not an exemption). The only exemptions: a minor animated accent inside a larger non-animation design, or the user explicitly asking you not to use the starter. Skipping the starter silently removes the user's timeline editor — scene trims, speed changes, and video export only exist when the page builds on the engine. Do NOT load `animations.jsx` or `animations-v2.jsx` alongside it (the engines share window globals — last wins). A project that already uses an older starter keeps it; don't migrate existing animations.

任何动画作品都必须构建在 `animations_v3.jsx` starter 之上——包括托管在 design-components 页面上的作品（helmet-script + x-import 结构正是 starter 的用例，而非豁免理由）。仅有的豁免：较大的非动画设计中的小型动画点缀，或用户明确要求不使用 starter。跳过 starter 会无声地移除用户的时间线编辑器——场景裁剪、速度调整和视频导出只有在页面构建于该引擎之上时才存在。不要同时加载 `animations.jsx` 或 `animations-v2.jsx`（各引擎共享 window 全局——后加载者胜出）。已经在使用旧 starter 的项目保持不变；不要迁移既有动画。

START by calling `copy_starter_component` with `kind: "animations_v3.jsx"` — the continuous-composition engine: the whole animation is ONE element tree rendered from one authored-time clock, so elements move, morph, and persist across section boundaries by ordinary interpolation — nothing mounts or unmounts at a boundary. It gives you `<CompositionStage>`, `useComposition()` (→ {T, CUES}), `<Shot>`, `<Captions>`, an `Easing` library, and `interpolate()` / `animate()` tweens. Read the file after copying.

第一步就调用 `copy_starter_component` 并传 `kind: "animations_v3.jsx"`——这是连续构图（continuous-composition）引擎：整个动画是一棵由单一创作时钟渲染的元素树，因此元素通过普通插值就能跨越分节边界移动、变形并持续存在——边界处没有任何东西挂载或卸载。它提供 `<CompositionStage>`、`useComposition()`（→ {T, CUES}）、`<Shot>`、`<Captions>`、`Easing` 库，以及 `interpolate()` / `animate()` 补间。复制后阅读该文件。

THE AUTHORING CONTRACT (it's in the file's usage block; follow it exactly): declare the scene list as a JSON string literal in a plain inline `<script>` of the MAIN document — `<script>window.OM_SCENES = '[{"name":"Opening","dur":3},…]';</script>` (exact JSON.stringify formatting, no spaces) — NOT in a text/babel script and NOT in a sibling .jsx (only vanilla inline script literals are addressable for the editor's write-back); declare `window.OM_PLAYBACK` the same way; pass both through untouched as `<CompositionStage scenes={window.OM_SCENES} playback={window.OM_PLAYBACK}>` wrapping ONE component — the whole piece. The scene list is the user-control view (names, order, playback durations); the engine derives the cue table from it, so the literal is the single source of structure. The user edits timing on the host timeline — trim a scene's edge, or set a section's speed — and every edit writes back into your literal and reflows the composition live.

创作契约（在文件的 usage 块中；严格遵守）：在主文档的一个普通内联 `<script>` 中把场景列表声明为 JSON 字符串字面量——`<script>window.OM_SCENES = '[{"name":"Opening","dur":3},…]';</script>`（与 JSON.stringify 完全一致的格式，无空格）——不要放在 text/babel 脚本里，也不要放在同级 .jsx 里（只有原生内联脚本的字面量可被编辑器写回寻址）；以同样方式声明 `window.OM_PLAYBACK`；两者原样透传给 `<CompositionStage scenes={window.OM_SCENES} playback={window.OM_PLAYBACK}>`，包裹一个组件——即整个作品。场景列表是用户控制视图（名称、顺序、播放时长）；引擎由它推导 cue 表，因此该字面量是结构的唯一来源。用户在宿主时间线上编辑时序——裁剪场景边缘，或设置某节的速度——每次编辑都会写回你的字面量，并实时重排构图。

CUE-FIRST DISCIPLINE (this is what makes a piece read as one continuous video):
1. Write the OM_SCENES literal FIRST — it is the piece's outline. Get {T, CUES} from `useComposition()` and key ALL choreography to T and CUES.SectionName (authored seconds) — never to your own clock, never to wall-clock time.
2. One helper component per section for readability, but ALL of them render ALL the time inside the one tree, keyed to cues — never conditionally mounted per section. A shared element crossing a boundary is just motion whose start and end straddle a cue.
3. Define exactly three motion helpers up front (e.g. `MOTION = {enter, draw, pop}` wrapping Easing curves) and use no easing or transform outside them; one caption element, one visible at a time (`<Captions>` has this built in).

CUE 优先纪律（正是这一点让作品读起来像一条连续视频）：
1. 先写 OM_SCENES 字面量——它是作品的大纲。从 `useComposition()` 获取 {T, CUES}，并把全部编排键控到 T 与 CUES.SectionName（创作秒数）上——绝不键控到你自己的时钟，绝不键控到墙钟时间。
2. 每节一个辅助组件以保证可读性，但它们全部始终渲染在同一棵树内、键控到 cue——绝不要按节条件挂载。跨越边界的共享元素只是一段起止横跨某个 cue 的运动而已。
3. 预先定义恰好三个运动辅助函数（例如包裹 Easing 曲线的 `MOTION = {enter, draw, pop}`），在它们之外不使用任何缓动或变换；字幕元素只有一个，同时只显示一条（`<Captions>` 内置了这一点）。

TIMING IS USER-EDITABLE (time-stretch): when the user trims or speeds a section, the engine replays that section's SAME authored slice over the new playback length — choreography keyed to T retimes, never cuts off. Authored hard cuts are content now, not structure: wrap a shot's elements in `<Shot from={CUES.X} to={CUES.Y}>` (visibility flips at the cues; children stay mounted so images and videos hold their readiness). A looping piece shows its last authored frame immediately before its first — make them match.

时序可由用户编辑（时间拉伸 time-stretch）：当用户裁剪或加速某节时，引擎把该节同样的创作切片按新的播放时长重放——键控到 T 的编排只是重新计时，绝不会被截断。创作中的硬切现在属于内容而非结构：把一个镜头的元素包进 `<Shot from={CUES.X} to={CUES.Y}>`（可见性在 cue 处翻转；子元素保持挂载，使图片和视频保持就绪状态）。循环播放的作品在首帧之前会立即显示其最后一个创作帧——让两者匹配。

Give every motion project a tweaks panel (`kind: "tweaks_panel.jsx"`) whose TWEAK_DEFAULTS include `"motionEditor": true` with a `<TweakToggle label="Motion editor">` wired to it. That key is the host timeline editor's visibility gate: the user flips it in the Tweaks panel to hide or show the editor bar, and the animation, its timing data, and export are untouched either way. Declare the TWEAK_DEFAULTS literal in a plain inline `<script>` of the MAIN document (the /*EDITMODE-BEGIN*/ convention), so the flip persists.

给每个动效项目配一个 tweaks 面板（`kind: "tweaks_panel.jsx"`），其 TWEAK_DEFAULTS 包含 `"motionEditor": true`，并接上一个绑定到该键的 `<TweakToggle label="Motion editor">`。该键是宿主时间线编辑器的可见性开关：用户在 Tweaks 面板中切换它来隐藏或显示编辑器条，无论怎么切换，动画本身、其时序数据和导出都不受影响。把 TWEAK_DEFAULTS 字面量声明在主文档的普通内联 `<script>` 中（遵循 /*EDITMODE-BEGIN*/ 约定），这样切换状态才能持久。

The stage renders inside <svg><foreignObject>; if a screenshot of it comes back black, that's a capture artifact — trust the live preview.

舞台渲染在 <svg><foreignObject> 内部；如果它的截图是黑的，那是截图环节的伪影——以实时预览为准。

WATCH the video before calling it done: stills at hand-picked timestamps hide exactly the boundary bugs (pops, mistimed motion) that make a piece feel disjointed. Build a filmstrip with ONE multi_screenshot call — in each step's code, seek by dispatching `new CustomEvent('data-om-seek-to-time-frame', {detail: {time: T, sync: true}})` on `document.querySelector('[data-om-exportable-video-with-duration-secs]')`, with T values straddling every scene boundary (boundary ±0.15s; boundaries are the running sums of your OM_SCENES durs) plus a mid-scene anchor or two (12-step cap per call — on long pieces spend the steps on boundaries). Two adjacent captures that don't visually match are a discontinuity to fix.

宣告完成之前先看一遍视频：手工挑时间点的静帧恰恰会隐藏让作品显得割裂的边界 bug（跳变、时机错位）。用一次 multi_screenshot 调用构建一条胶片条——在每一步的代码里，通过在 `document.querySelector('[data-om-exportable-video-with-duration-secs]')` 上派发 `new CustomEvent('data-om-seek-to-time-frame', {detail: {time: T, sync: true}})` 来 seek，T 取值横跨每个场景边界（边界 ±0.15s；边界是 OM_SCENES 各 dur 的累加和）并加上一到两个场景中段的锚点（每次调用至多 12 步——长作品应把步数花在边界上）。相邻两帧在视觉上不匹配即为待修复的不连续点。

Animations are complex code! Make reusable JSX components for each visual element and each section. Invest in tweaking the timeline iteratively.

动画是复杂的代码！为每个视觉元素和每一节制作可复用的 JSX 组件。投入精力迭代调整时间线。

Animation tips:
- Storytelling is KEY! Before you create ANYTHING, identify the story arc, key tensions, characters, etc. Align on the message you want to convey. Run it by the user.
- Use good animation principles... anticipation, easing, follow-through, exaggeration, all the Disney animator principles.
- Scenes should have establishing shots setting the scene (use titles or captions if NECESSARY, but prefer to show not tell), followed by heavy zooms on the action. (either hard cuts, or ken-burns-style zooms, or mouse-follows.) Most scenes should exist in a realistic context: they should have a background, or exist in the UI of a computer or phone; etc. Elements should generally not float in the aether.
- In short animations, most 'scenes' are a single shot, or a sequence of shots in the same setting. Scenes may be slides (e.g. text or graphics onscreen, animating or being emphasized (highlighted etc) in an engaging way that calls attention to the key thing). Decide what the shot is going to be. Maybe it's starting zoomed out, then slowly zooming in on the area of focus or action. Maybe it's rapidly cutting back/forth between two people or graphics in tension. Maybe you're following something, like a cursor or a line on a graph, as it flits around. Be creative!
- Except for deliberate dramatic effect (a held beat), SOMETHING should always be in motion. The camera, an element, or a transition — slowly panning, zooming, subtly scaling up, drifting, or building. A truly static frame reads as a bug. Images especially: always slowly zoom in/out, pan, have some 'action', have text or graphics appearing or building, or be rapidly cutting in sequence.
- Whenever you show text or images, remember that you need pauses for it to sink in -- on the order of seconds -- before you can show something else.

动画技巧：
- 叙事是关键！在创建任何东西之前，先确定故事弧线、关键冲突、角色等。就要传达的信息达成一致。讲给用户听并确认。
- 运用好的动画原则……预备动作（anticipation）、缓动（easing）、跟随与交叠（follow-through）、夸张（exaggeration），即迪士尼动画师的全部原则。
- 场景应有交代环境的定场镜头（仅在必要时使用标题或字幕，优先展示而非说明），随后对动作大幅推近。（硬切、ken-burns 式推拉，或鼠标跟随均可。）多数场景应存在于真实的语境中：应有背景，或存在于电脑或手机的 UI 中；等等。元素一般不应悬浮在虚空里。
- 在短动画中，多数"场景"是单一镜头，或同一环境中的一组镜头。场景可以是幻灯片式画面（例如屏幕上的文本或图形，以引人注目的方式进行动画或强调（高亮等），把注意力引向关键之处）。先决定镜头要拍什么。也许是从全景开始，然后缓慢推近焦点或动作区域；也许是在处于张力中的两个角色或图形之间快速来回切换；也许是跟随某个东西——比如光标或图上的一条线——看它四处游走。发挥创意！
- 除刻意的戏剧效果（一个停顿拍）之外，画面中应始终有东西在动。摄像机、某个元素或某个过渡——缓慢平移、缩放、微微放大、漂移或渐次成形。真正静止的帧看起来就像 bug。图片尤其如此：始终缓慢放大/缩小、平移，有某种"动作"，有文本或图形出现或渐显，或按序列快速切换。
- 每当展示文本或图片时，记住需要留出让观众消化的停顿——以秒计——然后才能展示别的东西。

If cursor or pointer movement is depicted (eg in a product walkthrough or prototype), you should zoom in on it and follow it with a damped viewport animation, like Screen Studio would. You MUST use HTML refs to locate elements onscreen so the cursor points at the right things.

如果描绘了光标或指针移动（例如在产品演示或原型中），应以阻尼视口动画推近并跟随它，如同 Screen Studio 的做法。必须使用 HTML ref 定位屏幕上的元素，使光标指向正确的东西。

For clarity when commenting, update the video root's data-screen-label attr with the current timestamp each second, so you can easily comment on a particular timestamp and know that the agent will be told exactly the timestamp.

为了便于在评论中说明，每秒更新视频根元素的 data-screen-label 属性为当前时间戳，这样你就能方便地针对某个时间点发表评论，并确知智能体会被告知准确的时间戳。

To make content the user can export as a video (Share → Export → Video):

要让内容可被用户导出为视频（Share → Export → Video）：

IF YOU ARE BUILDING ON AN ANIMATIONS STARTER (`animations_v3.jsx`, or `animations_v2.jsx` in an older project; the normal case): the stage component (`<CompositionStage>` / `<SceneStage>`) ALREADY SATISFIES THIS ENTIRE CONTRACT — it owns the exportable attribute, the seek listener, the svg/foreignObject wrapper, and font inlining (animations_v2 additionally provides a `<VideoSprite>` helper for looped <video> clips). Do NOT add `data-om-exportable-video-with-duration-secs` to any element yourself. Adding it to a wrapper ABOVE the stage creates two nested exportable roots, and the export and timeline transport bind to the wrong (outer) one — playback control and export silently break.

如果你构建在动画 starter 之上（`animations_v3.jsx`，或旧项目中的 `animations_v2.jsx`；这是正常情况）：舞台组件（`<CompositionStage>` / `<SceneStage>`）已经满足了这整份契约——它拥有可导出属性、seek 监听器、svg/foreignObject 包装以及字体内联（animations_v2 另外提供用于循环 <video> 片段的 `<VideoSprite>` 辅助组件）。不要自己给任何元素添加 `data-om-exportable-video-with-duration-secs`。把它加到舞台之上的某个包装元素会创建两个嵌套的可导出根，导出与时间线走带会绑定到错误的（外层）那个——播放控制和导出会无声失效。

Only for a page built WITHOUT the starter, implement the contract yourself:

只有对不基于 starter 构建的页面，才自行实现该契约：

- Put `data-om-exportable-video-with-duration-secs="<N>"` on the ONE root element you want exported (N ≤ 300; longer is clamped). Exactly one element in the document may carry this attribute — never nest it.
  把 `data-om-exportable-video-with-duration-secs="<N>"` 放在你希望导出的唯一一个根元素上（N ≤ 300；更长会被截断）。文档中只能有一个元素携带该属性——绝不要嵌套。
- That element MUST listen for the custom event `data-om-seek-to-time-frame` (`detail: {time, frame}`): on receipt, pause playback and synchronously render that exact timestamp so every visible child is at that point.
  该元素必须监听自定义事件 `data-om-seek-to-time-frame`（`detail: {time, frame}`）：收到后暂停播放并同步渲染该精确时间戳，使每个可见子元素都处于该时点。
- Nested `<video>` elements that should contribute audio must carry `data-om-exportable-video-play-start`, `data-om-exportable-video-play-end` (seconds into the source), and optionally `data-om-exportable-video-play-speed`; they loop within [start,end] at that speed and their audio is mixed into the export. Keep their visual frame in sync with the timeline yourself (set `video.currentTime` from the seek event / your clock).
  需要贡献音频的嵌套 `<video>` 元素必须携带 `data-om-exportable-video-play-start`、`data-om-exportable-video-play-end`（在源中的秒数），以及可选的 `data-om-exportable-video-play-speed`；它们在 [start,end] 内按该速度循环，其音频会被混入导出。请自行让它们的画面帧与时间线保持同步（根据 seek 事件 / 你的时钟设置 `video.currentTime`）。
- For best results, make the root an `<svg><foreignObject>` wrapper and inline your @font-face rules into it once — the exporter then serializes the svg directly per frame (fast, pixel-perfect). A plain div works too, just slower (full-page snapshot per frame).
  为获得最佳效果，把根做成 `<svg><foreignObject>` 包装，并把 @font-face 规则一次性内联进去——这样导出器每帧直接序列化 svg（快速、像素级精确）。普通 div 也可行，只是更慢（每帧全页快照）。

A page carrying this contract also gets a live timeline under the preview — the host scrubs and plays it by dispatching the same seek event, so treat every seek as pause-and-hold (don't resume your own clock until seeks stop arriving).

携带该契约的页面还会在预览下方得到一条实时时间线——宿主通过派发同一个 seek 事件来拖动与播放，因此把每次 seek 都当作"暂停并保持"（在 seek 停止到达之前不要恢复你自己的时钟）。
