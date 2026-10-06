---
name: save-as-standalone-html
description: "Single self-contained file that works offline"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Save as standalone HTML / 另存为独立 HTML

Export the current design as a single self-contained HTML file that works completely offline — no external dependencies.

将当前设计导出为一个完全可离线运行的单个自包含 HTML 文件 —— 无外部依赖。

### How it works / 工作原理

There is a deterministic bundler (super_inline_html tool) that can inline resources referenced directly in HTML attributes — img src/srcset, source src/srcset, video/audio/track src, video poster, SVG `<image href>`/`<use href>`, link href (stylesheets, favicons), script src, CSS url() and @import, inline style attributes. However, it CANNOT discover resources that are only referenced as strings in JavaScript or JSX code — for example:

存在一个确定性的打包器（super_inline_html 工具），它可以内联直接在 HTML 属性中引用的资源 —— img src/srcset、source src/srcset、video/audio/track src、video poster、SVG `<image href>`/`<use href>`、link href（样式表、favicon）、script src、CSS url() 与 @import、内联 style 属性。但它无法发现那些仅以字符串形式出现在 JavaScript 或 JSX 代码中的资源 —— 例如：

- An image src set in React: `<img src={"./hero.png"} />`
  在 React 中设置的图片 src：`<img src={"./hero.png"} />`
- A background URL in a styled-component: `background: url('./pattern.svg')`
  styled-component 中的背景 URL：`background: url('./pattern.svg')`
- A dynamically imported script
  动态导入的脚本

Your job is to prepare the HTML file so the bundler can capture everything, then run it.

你的任务是准备好 HTML 文件，使打包器能捕获所有资源，然后运行它。

### Step 1: Make a copy of the HTML file and update code-referenced resources / 步骤 1：复制 HTML 文件并更新代码引用的资源

Copy the current HTML file. Read it. Copy its dependencies. Look through ALL the code (inline scripts, imported JSX files, styled-components, etc) for any resource URL that is referenced as a string in code rather than as an HTML attribute. This includes:

复制当前 HTML 文件。阅读它。复制其依赖。检查全部代码（内联脚本、导入的 JSX 文件、styled-components 等），找出任何以字符串而非 HTML 属性形式在代码中引用的资源 URL。这包括：

- Image URLs in React/JSX (`<img src={...} />`, `style={{ backgroundImage: ... }}`)
  React/JSX 中的图片 URL（`<img src={...} />`、`style={{ backgroundImage: ... }}`）
- URLs in CSS-in-JS (styled-components, inline styles set via JS)
  CSS-in-JS 中的 URL（styled-components、通过 JS 设置的内联样式）
- Script tags that import other scripts which themselves reference resources
  导入其他脚本、且那些脚本自身又引用资源的 script 标签
- Any fetch() or XMLHttpRequest calls that load assets
  任何加载资源的 fetch() 或 XMLHttpRequest 调用
- Audio/video sources set programmatically
  以编程方式设置的音频/视频源

Note: if you use the Anthropic API in the project, it will not work standalone. If this is core to the project, STOP and tell the user!

注意：如果项目中使用了 Anthropic API，它将无法独立运行。如果这是项目的核心部分，立即停止并告知用户！

### Step 2: Add ext-resource-dependency meta tags / 步骤 2：添加 ext-resource-dependency meta 标签

For EACH resource found in step 1, add a `<meta>` tag in the `<head>`:

对步骤 1 中找到的每一个资源，在 `<head>` 中添加一个 `<meta>` 标签：

```html
<meta name="ext-resource-dependency" content="<url>" data-resource-id="<id>" />
```

Where:
- `content` is the URL of the resource (relative to the HTML file, or absolute)
  `content` 是资源的 URL（相对于 HTML 文件，或绝对路径）
- `data-resource-id` is a short, unique identifier (e.g. "heroImage", "patternSvg")
  `data-resource-id` 是简短且唯一的标识符（如 "heroImage"、"patternSvg"）

Then update the code to reference `window.__resources[id]` instead of the hardcoded URL. At runtime in the bundled file, `window.__resources[id]` will contain a blob URL pointing to the inlined resource data.

然后更新代码，改用 `window.__resources[id]` 而不是硬编码的 URL。在打包后的文件运行时，`window.__resources[id]` 将包含指向已内联资源数据的 blob URL。

Example:
```html
<!-- In <head>: -->
<meta name="ext-resource-dependency" content="./hero.png" data-resource-id="heroImg" />
<meta name="ext-resource-dependency" content="./pattern.svg" data-resource-id="patternBg" />

<!-- In code, replace: -->
<!-- <img src={"./hero.png"} /> -->
<!-- with: -->
<!-- <img src={window.__resources.heroImg} /> -->
```

IMPORTANT:
- The relative paths in `content` are relative to the HTML page itself
  `content` 中的相对路径相对于 HTML 页面本身
- You must also do this for any external script tags that are imported and themselves reference resources — those scripts will be inlined by the bundler, but their resource references need to be lifted too
  对被导入且自身引用资源的任何外部 script 标签也必须这样做 —— 这些脚本会被打包器内联，但它们的资源引用也需要被提取
- Be thorough! Missing even one resource means a broken image or missing asset in the final file
  务必彻底！哪怕漏掉一个资源，最终文件中就会出现破损的图片或缺失的资源

### Step 3: Create a thumbnail (REQUIRED — the bundler will reject the file without it) / 步骤 3：创建缩略图（必需 —— 缺少它打包器将拒绝该文件）

Create a lightweight SVG thumbnail that acts as a splash screen while the bundled file unpacks. This SVG should be a simplified, representative preview of the design — e.g. the key shapes, layout silhouette, or a branded loading visual. It doesn't need to be pixel-perfect, just visually representative so the user sees something meaningful instantly. It will be displayed TINY so a simple glyph on a vibrant color BG is enough.

创建一个轻量级 SVG 缩略图，在打包文件解包期间充当启动画面。该 SVG 应是设计的简化、有代表性的预览 —— 例如关键图形、布局轮廓，或带品牌感的加载视觉。它不需要像素级精确，只需视觉上有代表性，让用户能立刻看到有意义的内容。它会以很小的尺寸显示，因此鲜艳底色上的简单图形即可。

Add it as a `<template>` tag in the source HTML:

将其作为 `<template>` 标签添加到源 HTML 中：

```html
<template id="__bundler_thumbnail" data-bg-color="#0a5e3e">
  <svg viewBox="0 0 1200 800" xmlns="http://www.w3.org/2000/svg">
    <!-- Simplified icon -->
  </svg>
</template>
```

- Set `data-bg-color` to match the page's background color
  将 `data-bg-color` 设置为与页面背景色一致
- The SVG should use `viewBox` for proper aspect-fit scaling
  SVG 应使用 `viewBox` 以实现正确的等比缩放
- Keep it simple — this is just a loading placeholder, not a full reproduction
  保持简单 —— 这只是加载占位图，不是完整复刻
- Use the design's actual colors so the transition feels seamless
  使用设计的实际颜色，使过渡显得自然无缝

The bundler will extract this and display it fullscreen (aspect-fit with the background color) while unpacking assets, then replace it with the real page. It also remains visible as the permanent fallback when JavaScript is disabled.

打包器会提取它并在解包资源期间全屏显示（按背景色等比适配），然后用真实页面替换它。在 JavaScript 被禁用时，它还会作为永久回退内容保持可见。

### Step 4: Run the bundler / 步骤 4：运行打包器

If you made changes in steps 1-3, save the modified HTML file first. Then (or if no changes were needed) call:

如果在步骤 1-3 中做了更改，先保存修改后的 HTML 文件。然后（或如果无需更改）调用：

```
super_inline_html({ input_path: "<path-to-html>", output_path: "My Deck.html" })
```

Give the output file a friendly human name.

为输出文件起一个友好的、人类可读的名字。

### Step 5: Verify (internal check only) / 步骤 5：验证（仅限内部检查）

**Read the tool result first** — if any asset couldn't be resolved, super_inline_html lists it directly in its output ("N asset(s) could not be bundled: - asset not found: ./foo.png"). That's the authoritative miss list; fix those references and re-run before opening anything.

**先阅读工具结果** —— 如果有任何资源无法解析，super_inline_html 会直接在其输出中列出（"N asset(s) could not be bundled: - asset not found: ./foo.png"）。这就是权威的遗漏清单；修复那些引用并重新运行，然后再打开任何内容。

Then open the bundled output with show_html TO CHECK IT WORKS — this is a private verification step for YOU, not the delivery mechanism. Check get_webview_logs for runtime errors (JS exceptions, failed decodes). If there are issues, fix the source file and re-run.

然后用 show_html 打开打包后的输出，确认它能正常工作 —— 这是给你自己的私有验证步骤，不是交付机制。检查 get_webview_logs 中是否有运行时错误（JS 异常、解码失败）。如有问题，修复源文件并重新运行。

### Step 6: Present for download — MANDATORY / 步骤 6：提交下载 —— 强制要求

You MUST deliver the final file using **present_fs_item_for_download** pointing directly at the inlined HTML output. This is the ONLY correct way to hand off a standalone export.

你必须使用 **present_fs_item_for_download** 交付最终文件，直接指向内联后的 HTML 输出。这是交付独立导出结果的唯一正确方式。

- Do NOT use show_html / show_to_user as the delivery step — those are preview tools, not download tools. The user cannot save the file from them.
  不要将 show_html / show_to_user 用作交付步骤 —— 它们是预览工具，不是下载工具。用户无法通过它们保存文件。
- Do NOT ask whether they want to download it — just call present_fs_item_for_download.
  不要询问用户是否想下载 —— 直接调用 present_fs_item_for_download。
- If you skip this step, the user has no way to get the file. This step is non-negotiable.
  如果跳过此步骤，用户将无法拿到文件。此步骤不容妥协。
