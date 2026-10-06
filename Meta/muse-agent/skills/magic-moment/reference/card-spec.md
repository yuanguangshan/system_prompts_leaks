<!-- BILINGUAL-EN-ZH -->
# Card Spec — proof artifact cards + the web-artifact process / 卡片规格 — 证明工件卡片与网页工件流程

`../cmm/card.py` is the implementation and the source of truth for every
number below; this file explains intent. Where they disagree, the code
wins — fix this file.

`../cmm/card.py` 是下面所有数值的实现与唯一事实来源；本文件只解释意图。两者不一致时，以代码为准 —— 应修正本文件。

## Internals (maintainer reference — NOT an agent recipe) / 内部实现（维护者参考 — 不是智能体操作指南）

`make_card(inner_img, hero=...)` wraps a tightly cropped content image in
the white rounded shell (`hero` is accepted for the final-reveal beat but
renders the same shell — the gold border + FINAL PICK badge is retired);
the card then enters the video as a thread message animated by
`cmm/overlay.py`'s stack renderer. `cmm/compose.py` and `cmm/script.py`
are the only callers — as an agent you reach cards solely through
screenplay `visual` beats and `./mm render`, and the verify guards
already run inside compose. Do not write driver scripts against them.

`make_card(inner_img, hero=...)` 把一张紧密裁剪的内容图片包进白色圆角外壳（`hero` 参数仍被最终揭示节拍接受，但渲染的是同样的外壳 —— 金色边框加 FINAL PICK 徽章已弃用）；随后卡片作为线程消息进入视频，由 `cmm/overlay.py` 的堆叠渲染器做动画。`cmm/compose.py` 和 `cmm/script.py` 是仅有的调用方 —— 作为智能体，你只能通过剧本的 `visual` 节拍和 `./mm render` 使用卡片，验证守卫已经在 compose 内部运行。不要针对它们编写驱动脚本。

## What is locked, and why / 哪些被锁定，以及原因

**Single-source truth.** The proof card must be one pre-composed PNG from the
real capture. Never live-stack a body layer plus a faint layer plus a bold
layer — that doubles the text and produces a second ghost thumbnail peeking
outside the card. If you must replace a thumbnail, clear the full region to
opaque white first; a partial clear leaves a visible second cluster. Safest
option is to use the real captured body untouched.

**单一事实来源。** 证明卡片必须是从真实截屏预合成的一张 PNG。绝不要把正文层、淡层和加粗层实时堆叠 —— 那会使文字加倍，并产生第二个探出卡片之外的幽灵缩略图。如果必须替换缩略图，先把整个区域清为不透明白色；部分清除会留下一团可见的第二簇。最稳妥的做法是原样使用真实截取的正文。

**Content-driven height.** Card height follows the content, clamped only to
avoid absurd extremes. The bug this replaces forced a minimum height, which
produced cards that were half empty white when the crop was short. Exact
clamp values live in `make_card` in the code — they have drifted from prose
before, so read them there.

**由内容决定高度。** 卡片高度跟随内容，只在避免荒谬极端值时施加钳制。本规则取代的旧缺陷曾强制最小高度，导致裁剪内容偏短时卡片有半截空白。精确的钳制值在代码的 `make_card` 里 —— 它们此前就与文字描述发生过漂移，所以请到代码中读取。

**Rounded corners are load-bearing.** The shell is rounded and `verify_card`
proves it stayed that way (transparent extreme corner, opaque inset).
Compositing anything square over the shell's top corners is the most
visible way a proof card looks wrong — bake content INTO the inner image
before `make_card`, never onto the finished card.

**圆角是承重特性。** 外壳是圆角的，`verify_card` 会证明它保持如此（极端角透明、内嵌区域不透明）。把任何方形内容合成到外壳顶角之上，是证明卡片外观出错的最为显眼的方式 —— 在调用 `make_card` 之前就把内容烘焙进内部图片，绝不要画在成品卡片上。

**Cards enter on the shared stage.** A card uses the renderer's entry and
backward arc motion. After its authored hold, it remains in the thread and
shrinks and becomes more transparent as later entries push it into depth. Use a tap for a detailed artifact reveal.

**卡片在共享舞台上登场。** 卡片使用渲染器的入场和后退弧线运动。在其设定的停留之后，它仍留在画面线程中，并随着后续登场者把它推向纵深而缩小、变得更透明。需要详细展示工件时使用点按（tap）。

## Web-artifact process — specificity by full-block crop / 网页工件流程 — 以完整块裁剪保证具体性

Before calling something an Amazon or FedEx card, fetch it for real, then crop
to what the transcript is talking about at that second.

在把某样东西称作 Amazon 或 FedEx 卡片之前，先真实抓取页面，然后裁剪到剧本在该秒所谈论的内容。

**Never slice by percentage.** A naive "top 28%" slice cuts through baselines
and clips glyphs mid-line. Crop to the smallest *full block* that contains the
complete thought, plus ~12px padding, complete lines only. Validate that the
bottom edge lands on a clear whitespace row; if it doesn't, expand downward
until it does.

**绝不要按百分比切片。** 朴素的"顶部 28%"切片会切穿基线，把字形截断在行中间。裁剪到包含完整语义的最小*完整块*，外加约 12px 的留白，只保留完整的行。确认底边落在一条干净的空白行上；若不是，向下扩展直到满足。

Build the crop map per site, keyed by the noun phrase the transcript uses:

按站点构建裁剪映射表，以剧本使用的名词短语为键：

```python
CROP_MAP = {
  "hero_full":      {"box": (x0, y0, x1, y1), "note": "nav + title + dek, complete lines"},
  "model_panel":    {"box": (x0, y0, x1, y1), "note": "spec panel + all rows"},
  "hold_telemetry": {"box": (x0, y0, x1, y1), "note": "control + ring + live signal grid"},
  "punchline":      {"box": None,             "note": "final-reveal card, full frame"},
}
```

Then: identify the transcript noun phrase at that timestamp → map it to a rule
key → crop that box. If no box fits, re-capture via `browser.spawn_task` using
`getBoundingClientRect()` on the target element, then pad.

然后：识别该时间戳处剧本的名词短语 → 映射到规则键 → 裁剪对应的框。若没有框适用，通过 `browser.spawn_task` 用目标元素上的 `getBoundingClientRect()` 重新截取，再加留白。

**Each proof must be visibly different.** If two crops come out identical you
picked the same rule twice — that is a failure, not a shortcut.

**每张证明必须 visibly 不同。** 若两张裁剪完全相同，说明你把同一条规则选用了两次 —— 那是失败，不是捷径。

The boxes above are per-site and must be derived fresh for each capture —
never reuse a crop map measured against a different page.

上面的框是按站点定义的，必须为每次截屏重新推导 —— 绝不要复用针对另一个页面测得的裁剪映射表。

**Use real tokens, never invented ones.** Pull the actual palette off the page
with `getComputedStyle` rather than guessing brand colors. A page that fails
to load is still useful: an invalid FedEx tracking number returns a
system-error page, which proves the flow and still yields the real FedEx
purple, orange, fonts, and radii for a faithful replica. Never substitute a
plausible blue for a brand you didn't measure.

**使用真实的 token，绝不臆造。** 用 `getComputedStyle` 从页面上提取实际调色板，而不是猜测品牌色。加载失败的页面仍有用处：一个无效的 FedEx 运单号会返回系统错误页，这既证明了流程，又能提供真实的 FedEx 紫、橙、字体和圆角半径，用于忠实复刻。绝不要用一只看似合理的蓝色顶替你未实际测量的品牌色。

A tight portrait image (a product shot, a generated hero) skips cropping
entirely — use the real body untouched.

紧凑的竖版图片（产品照、生成的 hero 图）完全跳过裁剪 —— 原样使用真实正文。

## Card shapes to reach for / 可选用的卡片形态

These are DESIGN PATTERNS for the HTML you author per beat, not code — there
is no template library; every synthetic card is fresh HTML. The look of that
HTML (the Muse Moments Kit components, scaling, and padding) is owned by
`design.md` — read it before authoring. Five shapes that have worked,
sharing the same shell logic and differing only in content:

这些是你为每个节拍编写 HTML 时的设计模式，而不是代码 —— 没有模板库；每张合成卡片都是全新的 HTML。该 HTML 的外观（Muse Moments Kit 组件、缩放与留白）由 `design.md` 掌管 —— 编写前先读它。以下五种形态已被验证有效，共享同一套外壳逻辑，仅内容不同：

1. **product** — thumb, title, star rating, price, delivery line, buy button
   **product** —— 缩略图、标题、星级评分、价格、配送行、购买按钮
2. **order** — dark header, order number, date, item list
   **order** —— 深色页眉、订单号、日期、商品列表
3. **tracking** — carrier-purple header, status, timeline
   **tracking** —— 承运商紫页眉、状态、时间线
4. **call-dark** — dark background, waveform, speaker labels, timestamp
   **call-dark** —— 深色背景、波形、说话人标签、时间戳
5. **confirmation-green** — white card, green check, amount, confirmation code
   **confirmation-green** —— 白色卡片、绿色对勾、金额、确认码

## Verification / 验证

`verify_card(card, name)` and `verify_proofs(cards)` live in
`../cmm/card.py` and compose runs them on every SHELL'D visual (`"shell"`
or `"hero"` beats). Standalone visuals — the default — are guarded by
render_html's blank/height checks and the deterministic photo treatment
instead. Do not hand-roll any of these checks.

`verify_card(card, name)` 和 `verify_proofs(cards)` 位于 `../cmm/card.py`，compose 会对每个 SHELL'D 视觉（`"shell"` 或 `"hero"` 节拍）运行它们。独立视觉 —— 默认情形 —— 则由 render_html 的空白/高度检查和确定性的照片处理来守护。不要手工重复实现这些检查中的任何一个。

Three checks, each naming its own failure via `CardVerificationError`:

三项检查，每项都通过 `CardVerificationError` 指明自身的失败原因：

1. **top-left corner transparent** — an opaque corner means a square header
   was composited over the rounded shell
   **左上角透明** —— 角不透明意味着有方形页眉被合成到了圆角外壳之上
2. **r16 inset opaque** — proves the shell rendered at all
   **r16 内嵌区域不透明** —— 证明外壳确实渲染出来了
3. **card not blank** — catches a render that produced nothing
   **卡片非空白** —— 捕捉什么都没渲染出来的情况

There is deliberately **no fill-ratio check**. The half-empty card it would
guard against was caused by `make_card` forcing a 300px minimum height, and
`make_card` now derives height from the content itself
(`inner_h + 2*INNER_PAD`, clamped to 180–620). Every card taller than the
180px floor therefore fills exactly 100% of its usable height by
construction, so the check could not fire on any input.

这里有意**不设填充率检查**。它本要防范的半空卡片，源于 `make_card` 曾强制 300px 最小高度，而现在 `make_card` 直接从内容推导高度（`inner_h + 2*INNER_PAD`，钳制在 180–620）。因此每张高于 180px 下限的卡片在构造上就恰好填满 100% 的可用高度，该检查在任何输入上都不可能触发。

Note also why a *pixel* version of that check would be wrong: the crop rule
above tells you to expand until the bottom edge lands on clear whitespace, so
a correctly cropped card ends in whitespace *by design*. Shell padding and
content whitespace are the same pixels; no pixel test can tell them apart.

还要说明为什么该检查的*像素级*版本是错误的：上面的裁剪规则要求向下扩展直到底边落在干净的空白上，所以裁剪正确的卡片在设计上就以空白收尾。外壳留白与内容空白是同一批像素，任何像素测试都无法区分它们。

`verify_card` returns its measurements so a caller can log them rather than
just asserting.

`verify_card` 会返回它的测量值，调用方可以将其记录下来，而不只是做断言。
