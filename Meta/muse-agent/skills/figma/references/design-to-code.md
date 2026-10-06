<!-- BILINGUAL-EN-ZH -->

# Design to code / 设计稿转代码

Use this workflow when implementing a Figma screen or component in a codebase.

在代码库中实现 Figma 屏幕或组件时，使用此工作流。

1. Inspect the target repository before editing. Identify its framework,
   styling conventions, installed design-system packages, existing components,
   assets, and tokens.
   编辑前先检查目标仓库：确认其框架、样式约定、已安装的设计系统包、现有组件、资源和设计令牌（tokens）。
2. Call `get_design_context` for the requested node and request a screenshot in
   the same call when its live schema supports that option. If the response has
   no screenshot, call `get_screenshot` before editing.
   对目标节点调用 `get_design_context`；当其实时 schema 支持该选项时，在同一次调用中请求截图。如果响应中没有截图，则在编辑前调用 `get_screenshot`。
3. If the context response says it is sparse, do not implement from that
   summary. Correlate its visible child IDs with the screenshot and request
   high-fidelity context for those children in one batch.
   如果上下文响应表明内容稀疏，不要基于该摘要直接实现。把其中的可见子节点 ID 与截图对应起来，并一次性批量请求这些子节点的高保真上下文。
4. Treat generated React or Tailwind as a visual specification, not code that
   must be pasted verbatim. Translate it into the repository's stack, layout
   primitives, component boundaries, and interaction patterns.
   把生成的 React 或 Tailwind 代码当作视觉规格说明，而不是必须原样粘贴的代码。将其转换为仓库所用技术栈、布局原语、组件边界和交互模式。
5. Reuse Code Connect mappings at their mapped nodes. Also reuse suitable local
   components and tokens; recreate them only when they cannot express the
   design accurately.
   在映射到的节点处复用 Code Connect 映射。同时复用合适的本地组件和设计令牌；只有当它们无法准确表达设计时才重建。
6. Preserve every visible static asset in its intended slot and geometry.
   Download provider-supplied assets to durable project paths and remove all
   temporary Figma URLs. Never substitute the screenshot for implementation
   assets or redraw an available asset by hand.
   保留每个可见静态资源在其应有的位置和几何尺寸。把提供方给出的资源下载到持久的项目路径，并移除所有临时 Figma URL。绝不要用截图代替实现所需的资源，也不要手工重绘本可获取的资源。
7. Render the requested screen or component and compare it with the Figma
   screenshot. Fix in-scope layout, typography, color, interaction, and asset
   mismatches; report pre-existing out-of-scope differences without changing
   them.
   渲染所请求的屏幕或组件，并与 Figma 截图比对。修复范围内的布局、排版、颜色、交互和资源不一致；对先前已存在的范围外差异只做报告，不做改动。

Keep API- or prop-provided images dynamic. Avoid absolute positioning when the
design can be represented with the project's responsive layout system.

由 API 或 props 提供的图片要保持动态。当设计可以用项目的响应式布局系统表达时，避免使用绝对定位。

【评论】第 3 条是针对上下文稀疏（模型凭低保真摘要臆造界面）的防护：强制先用截图交叉核对子节点，再批量索取高保真数据后才动手实现。
