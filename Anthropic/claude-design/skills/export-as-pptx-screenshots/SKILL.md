---
name: export-as-pptx-screenshots
description: "Flat images — pixel-perfect but not editable"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Export as PPTX (screenshots) / 导出为 PPTX（截图方式）

Export an HTML slide deck to a `.pptx` as full-bleed PNG images. Pixel-perfect, not editable. One `gen_pptx` tool call.

把 HTML 幻灯片组导出为 `.pptx`，内容为满幅 PNG 图片。像素级还原，但不可编辑。只需一次 `gen_pptx` 工具调用。

### Steps / 步骤

1. `show_to_user` the deck.
   用 `show_to_user` 展示幻灯片组。
2. Call `gen_pptx`:
   调用 `gen_pptx`：

```jsonc
{
  "mode": "screenshots",
  "width": 1920, "height": 1080,
  "slides": [
    { "showJs": "goToSlide(0)", "selector": "body" },  // selector unused in screenshot mode but required
    { "showJs": "goToSlide(1)", "selector": "body" }
  ],
  "hideSelectors": [".nav", ".progress"],
  // If the deck wraps slides in a transform:scale() container, name it here so
  // the deck is forced to width × height inside the locked iframe.
  "resetTransformSelector": ".slide-container",
  "filename": "my-deck"
}
```

If — and only if — the user asked for Google Slides, also pass `"offer_google_slides": true`: the export dialog gains a 'Send to Google Slides' button, and the upload happens only if they click it.

当且仅当用户要求 Google Slides 时，才额外传入 `"offer_google_slides": true`：导出对话框会多出一个 'Send to Google Slides' 按钮，且只有用户点击该按钮才会真正执行上传。

`slides[].delay` defaults to 600ms — bump if transitions are slower.

`slides[].delay` 默认为 600ms——如果幻灯片切换动画更慢，请调大此值。

#### If the deck uses the `<deck-stage>` starter component / 如果幻灯片组使用了 `<deck-stage>` 起始组件

- `resetTransformSelector: "deck-stage"` — same as editable mode; the component drops its shadow-DOM `transform: scale()` so the slides fill the locked iframe.
  `resetTransformSelector: "deck-stage"`——与可编辑模式相同；该组件会去掉其 shadow-DOM 中的 `transform: scale()`，让幻灯片填满被锁定的 iframe。
- `slides[N].showJs`: `"document.querySelector('deck-stage').goTo(N)"` — 0-indexed, so slide 1 is `goTo(0)`.
  `slides[N].showJs`：`"document.querySelector('deck-stage').goTo(N)"`——索引从 0 开始，因此第 1 张幻灯片对应 `goTo(0)`。
- `hideSelectors` is unnecessary — the overlay and tap-zones live in shadow DOM and aren't captured.
  无需设置 `hideSelectors`——覆盖层和点击热区位于 shadow DOM 中，不会被截取。

### Validation / 校验

Same flags as editable mode. Watch for `duplicate_adjacent` (showJs didn't navigate) and `reset_selector_miss` / `slide_size_mismatch` (your `resetTransformSelector` matched nothing or didn't size to width × height).

使用的标志与可编辑模式相同。注意 `duplicate_adjacent`（showJs 没有完成导航）以及 `reset_selector_miss` / `slide_size_mismatch`（你的 `resetTransformSelector` 没有匹配到元素，或未把尺寸调整为宽 × 高）。

Speaker notes from `#speaker-notes` are attached automatically. Page reloads after.

来自 `#speaker-notes` 的演讲者备注会自动附上。随后页面会重新加载。
