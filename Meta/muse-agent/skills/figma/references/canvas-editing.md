<!-- BILINGUAL-EN-ZH -->

# Canvas editing / 画布编辑

Use `use_figma` for native canvas inspection and mutation. Its script executes
inside Figma; the value explicitly returned by the script is the usable output.

使用 `use_figma` 进行原生的画布检查与修改。其脚本在 Figma 内部执行；脚本显式返回的值才是可用输出。

## Before editing / 编辑之前

- Inspect the file with a read-only script first. Discover its editor type,
  pages, components, variables, fonts, naming, and layout conventions.
  先用只读脚本检查文件。摸清其编辑器类型、页面、组件、变量、字体、命名与布局惯例。
- Infer editor type from the URL: `/design/`, `/board/`, or `/slides/`. APIs and
  supported node types differ across Design, FigJam, and Slides.
  从 URL 推断编辑器类型：`/design/`、`/board/` 或 `/slides/`。Design、FigJam 与 Slides 的 API 和受支持节点类型各不相同。
- Prefer existing components, styles, and variables over parallel lookalikes.
  优先使用已有的组件、样式与变量，而不是另造外观相似的替代品。
- Batch related work when it remains understandable and safe to recover. Avoid
  many tiny mutation calls followed by redundant screenshots.
  在仍可理解且可安全恢复的前提下，把相关工作合并批处理。避免大量微小修改调用加上冗余截图。

## Mutation rules / 修改规则

- `return` a structured result containing every created or mutated node ID.
  Logging to the console is not a substitute for returning evidence.
  `return` 一个包含每个新建或被修改节点 ID 的结构化结果。输出到控制台不能替代返回证据。
- Await every asynchronous API call. Load the existing font data before any
  text mutation; never guess a font style name.
  对每个异步 API 调用都要 await。在任何文本修改之前先加载已有字体数据；绝不臆测字体样式名。
- Switch to a target page asynchronously and at most once per call. Split
  independent multi-page work into parallel calls.
  以异步方式切换到目标页面，且每次调用至多切换一次。把相互独立的多页工作拆分成并行调用。
- Use auto layout for structurally related children. Append a child to its
  auto-layout parent before assigning fill or hug sizing, and resize before
  setting final sizing modes.
  对结构上相关的子元素使用自动布局（auto layout）。先把子元素挂到其自动布局父级上，再设置 fill 或 hug 尺寸；先调整大小，再设定最终的尺寸模式。
- Figma color channels use values from 0 through 1. Put opacity on the paint,
  not inside its color object. Clone read-only fill and stroke arrays before
  modifying and reassigning them.
  Figma 的颜色通道取值范围为 0 到 1。不透明度放在 paint（涂料）上，而不是其 color 对象内部。修改并重新赋值只读的 fill 与 stroke 数组之前，先克隆它们。
- Position new page-level content in clear space instead of accepting the
  default origin.
  把新的页面级内容放到空白区域，而不是接受默认原点。

## Recovery and verification / 恢复与验证

Treat a timeout or error as an uncertain write. If the result says it is unsafe
to retry without reading the canvas, inspect first and reconcile created IDs
before attempting another mutation. Never replay the whole write blindly.

把超时或错误视为一次结果不确定的写入。如果结果表明不读取画布就重试是不安全的，先检查画布并核对已创建的 ID，再尝试下一次修改。绝不盲目重放整个写入。

Require structural evidence from each mutation: affected IDs plus useful
counts, names, or bounds. Take a screenshot after meaningful composition and
again only when validating a visual fix. Stop once structural and visual checks
pass; repeated unchanged screenshots waste the user's quota.

每次修改都要有结构化证据：受影响的 ID 加上有用的计数、名称或边界。在完成有意义的构图后截一次图，且仅在验证视觉修复时再截。结构与视觉检查通过后即停止；重复截取无变化的截图是在浪费用户的配额。
