---
name: export-as-pptx-editable
description: "Native text & shapes — editable in PowerPoint"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Export as PPTX (editable) / 导出为 PPTX（可编辑）

Export an HTML slide deck to a `.pptx` with native PowerPoint objects (editable text, shapes, images). One `gen_pptx` tool call does everything: capture, font handling, generation, download.

将 HTML 幻灯片导出为带有原生 PowerPoint 对象（可编辑文本、形状、图片）的 `.pptx`。一次 `gen_pptx` 工具调用完成所有事情：捕获、字体处理、生成、下载。

### What you do / 你要做的事

1. **Know the deck.** You probably wrote it. If not, `read_file` the HTML to find: the slide selector, how to navigate (function name? class toggle?), what fonts it uses, whether there's a scaling wrapper.

   **了解这套幻灯片。** 它大概率是你写的。如果不是，用 `read_file` 读取 HTML，找出：幻灯片选择器、导航方式（函数名？类名切换？）、使用的字体、是否存在缩放包装容器。

2. **`show_to_user`** the deck so it's in the user's preview.

   用 **`show_to_user`** 展示幻灯片，使其出现在用户预览中。

3. **Call `gen_pptx`** with the inputs below.

   使用下面的输入**调用 `gen_pptx`**。

4. **Read the validation flags** in the result and decide if you need to retry.

   **阅读结果中的校验标志**，判断是否需要重试。

### gen_pptx inputs / gen_pptx 输入

```jsonc
{
  "width": 1920, "height": 1080,   // CSS px — match the deck's slide size
  "slides": [                      // one entry per slide, in order
    { "showJs": "goToSlide(0)", "selector": ".slide.active" },
    { "showJs": "goToSlide(1)", "selector": ".slide.active" }
    // For decks where all slides are in DOM at once and you don't need to navigate:
    //   { "selector": ".slide:nth-child(1)" }, { "selector": ".slide:nth-child(2)" }
  ],
  "hideSelectors": [".nav", ".progress", "[data-omelette-chrome]", "[data-noncommentable]"],
  // If the deck wraps slides in a transform:scale() container, name it here.
  // gen_pptx clears the transform AND forces width/height onto this element.
  "resetTransformSelector": ".slide-container",
  // Font handling — pick ONE strategy based on the directive at the bottom.
  // Substitution happens BEFORE capture so layout reflows correctly.
  "googleFontImports": ["Poppins", "Lora"],
  "fontSwaps": [{ "from": "BrandSans", "to": "Poppins" }],
  // Or fontSwaps: [{from:"BrandSans", to:"Arial"}] for web-safe.
  // Or omit both to keep brand fonts as-is.
  "filename": "my-deck"
}
```

If — and only if — the user asked for Google Slides, also pass `"offer_google_slides": true`: the export dialog gains a 'Send to Google Slides' button, and the upload happens only if they click it.

当且仅当用户要求 Google Slides 时，才额外传入 `"offer_google_slides": true`：导出对话框会多出一个"Send to Google Slides"按钮，且只有用户点击时才会上传。

`slides[].showJs` runs inside the iframe as a sync expression — don't `await`. If your deck's nav function is async, call it without await; the per-slide `delay` (default 600ms) covers the transition. Bump `delay` for decks with longer CSS transitions.

`slides[].showJs` 在 iframe 内作为同步表达式运行 —— 不要 `await`。如果幻灯片的导航函数是异步的，直接不带 await 地调用它；每张幻灯片的 `delay`（默认 600ms）会覆盖过渡时间。对于 CSS 过渡更长的幻灯片，调大 `delay`。

#### If the deck uses the `<deck-stage>` starter component / 如果幻灯片使用了 `<deck-stage>` 起始组件

- `resetTransformSelector: "deck-stage"` — the exporter sets the `noscale` attribute on it, which the component observes and responds to by dropping its shadow-DOM `transform: scale()`. You cannot reach the scaled canvas any other way.
  `resetTransformSelector: "deck-stage"` —— 导出器会在其上设置 `noscale` 属性，该组件会观察并响应此属性，移除其 shadow-DOM 中的 `transform: scale()`。除此之外你无法触达被缩放的画布。
- `slides[N].showJs`: `"document.querySelector('deck-stage').goTo(N)"` — 0-indexed, so slide 1 is `goTo(0)`.
  `slides[N].showJs`：`"document.querySelector('deck-stage').goTo(N)"` —— 从 0 开始计数，因此第 1 张幻灯片是 `goTo(0)`。
- `slides[N].selector`: `"deck-stage > [data-deck-active]"`.
  `slides[N].selector`：`"deck-stage > [data-deck-active]"`。
- `hideSelectors` is unnecessary — the overlay and tap-zones live in shadow DOM and aren't captured.
  `hideSelectors` 不需要 —— 覆盖层和点击区位于 shadow DOM 中，不会被捕获。

### Speaker notes / 演讲者备注

Read automatically from `<script type="application/json" id="speaker-notes">` and attached by index. You don't pass them.

自动从 `<script type="application/json" id="speaker-notes">` 读取，并按索引附加。你无需传入它们。

### Validation flags / 校验标志

The result lists flags. **These are warnings, not errors** — read each message and decide if it's expected for THIS deck:

结果会列出标志。**这些是警告，不是错误** —— 阅读每条消息，判断它对这套幻灯片而言是否属于预期：

- `duplicate_adjacent` / `duplicate_majority` — slides captured identically. Almost always means `showJs` didn't navigate. Check the function name, try a longer `delay`, or check if the deck uses 0-indexed vs 1-indexed slides.
  `duplicate_adjacent` / `duplicate_majority` —— 幻灯片被捕获为完全相同。几乎总是意味着 `showJs` 没有执行导航。检查函数名、尝试更长的 `delay`，或检查幻灯片使用的是 0 起始还是 1 起始索引。
- `slide_size_mismatch` — captured rect doesn't match width/height. The selector is probably matching a wrapper, or you need a `resetTransformSelector`.
  `slide_size_mismatch` —— 捕获的矩形与 width/height 不符。选择器可能匹配到了包装元素，或者你需要 `resetTransformSelector`。
- `notes_uniform_nonempty` — every speaker note is the same string. Likely a placeholder. Fine if intentional.
  `notes_uniform_nonempty` —— 每条演讲者备注都是同一字符串。很可能是占位内容。若属有意为之则无妨。
- `notes_count_mismatch` — #speaker-notes length ≠ slides length. Notes attach by index so the tail will be wrong.
  `notes_count_mismatch` —— #speaker-notes 的数量 ≠ 幻灯片数量。备注按索引附加，因此尾部会错位。
- `no_speaker_notes` — deck has no #speaker-notes tag. Expected if there are no notes.
  `no_speaker_notes` —— 幻灯片没有 #speaker-notes 标签。若本来就没有备注则属预期。
- `fonts_timeout` — fonts.ready took >8s. Font URLs may be unreachable.
  `fonts_timeout` —— fonts.ready 耗时超过 8 秒。字体 URL 可能不可达。
- `font_swap_failed` — one or more `fontSwaps` targets never loaded (misspelled family, or Google Fonts doesn't serve it), so the deck was laid out with a fallback while the file names the swap font. Retry with a corrected or different family, or fall back to web-safe fonts. Whatever you do next, tell the user plainly which fonts couldn't be applied — e.g. "Heads up: Poppins couldn't be loaded during export, so the deck uses a stand-in font and text may wrap differently. Want me to try a different font?"
  `font_swap_failed` —— 一个或多个 `fontSwaps` 目标始终未加载（字体族名拼写错误，或 Google Fonts 未提供该字体），幻灯片以回退字体完成排版，而文件中标注的却是替换字体。用更正后的或不同的字体族重试，或回退到 Web 安全字体。无论下一步做什么，都要坦率告诉用户哪些字体未能应用 —— 例如"注意：导出期间 Poppins 未能加载，因此幻灯片使用了替代字体，文本换行可能有所不同。要我试试别的字体吗？"
- `images_failed` — images didn't decode before capture. Usually a 404 or CORS.
  `images_failed` —— 图片在捕获前未能解码。通常是 404 或 CORS 问题。
- `reset_selector_miss` — your `resetTransformSelector` matched nothing.
  `reset_selector_miss` —— 你的 `resetTransformSelector` 没有匹配到任何元素。

If the flags look like real problems, fix the inputs and retry. If they're expected (deck genuinely has no notes, two slides really are identical), tell the user the download fired and move on.

如果标志指示真实问题，修复输入并重试。如果它们属于预期（幻灯片确实没有备注，两张幻灯片确实相同），告诉用户下载已触发，然后继续。

**Talking to the user about flags:** these names and messages are internal diagnostics — do NOT relay them verbatim. If everything is expected, don't mention validation at all; just confirm the download. If something looks genuinely wrong, describe it in plain language without the flag identifier or technical specifics — e.g. "Uh oh, the speaker notes may not be exporting properly." rather than "I received the no_speaker_notes flag", or "A couple of slides may have captured identically — let me fix navigation and retry." rather than quoting `duplicate_adjacent`.

**与用户谈论标志：** 这些名称和消息属于内部诊断信息 —— 不要原样转述。如果一切符合预期，完全不要提及校验；只需确认下载完成。如果确实有问题，用平实语言描述，不带标志标识符或技术细节 —— 例如"哎呀，演讲者备注可能没有正常导出"，而不是"我收到了 no_speaker_notes 标志"；"有两张幻灯片可能捕获成了相同的内容 —— 我来修复导航并重试"，而不是引用 `duplicate_adjacent`。

The page reloads automatically after capture — DOM mutations (hidden chrome, font swaps) are reverted.

捕获完成后页面会自动重新加载 —— DOM 变更（隐藏的界面元素、字体替换）会被还原。

### Font strategy / 字体策略

Read the directive at the end of this prompt and translate it to inputs:

阅读本提示词末尾的指令，并将其转换为输入：

| Directive | Inputs |
|---|---|
| brand fonts as-is | omit `googleFontImports` and `fontSwaps` |
| web-safe substitutes | `fontSwaps: [{from:"EachCustomFont", to:"Arial"}]` (or Georgia for serifs, Courier New for monospace) |
| Google Fonts substitutes | `googleFontImports: ["Poppins","Lora"]` + `fontSwaps: [{from:"EachCustomFont", to:"Poppins"}]` |

| 指令 | 输入 |
|---|---|
| 品牌字体原样保留 | 省略 `googleFontImports` 与 `fontSwaps` |
| Web 安全替代字体 | `fontSwaps: [{from:"EachCustomFont", to:"Arial"}]`（衬线体用 Georgia，等宽体用 Courier New） |
| Google Fonts 替代字体 | `googleFontImports: ["Poppins","Lora"]` + `fontSwaps: [{from:"EachCustomFont", to:"Poppins"}]` |

System fonts (Arial, Helvetica, Georgia, Times, Courier, sans-serif, etc.) — leave alone.

系统字体（Arial、Helvetica、Georgia、Times、Courier、sans-serif 等）—— 保持不动。
