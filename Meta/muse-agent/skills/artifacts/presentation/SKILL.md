---
name: artifact_presentation
metadata: { "includeInPrompt": false }
description: Build or revise a slide deck (pptx by default; pdf or html on request). Use whenever a build task's artifact kind is presentation, or the user asks for a deck, slides, or a presentation. Covers per-slide HTML authoring, the StylePlan theme system, font embedding, deck assembly, render gates, and the PPTX export.
---
<!-- BILINGUAL-EN-ZH -->

# Slide-deck artifacts / 幻灯片产物

A deck is authored as one standalone HTML file per slide plus a shared
`deck.css` and a `deck.json` manifest, assembled and gated deterministically,
then exported. The per-slide sources under `.src/slides/` are the editable
truth for every future revision; slide ids never renumber.

一份幻灯片（deck）的编写方式是：每张幻灯片一个独立 HTML 文件，外加共享的 `deck.css` 和 `deck.json` 清单文件，以确定性的方式组装并通过质量门检查，然后导出。`.src/slides/` 下的每张幻灯片源文件是后续所有修订的唯一可编辑事实来源；幻灯片 id 永不重新编号。

Three rules hold on every slide, before any reference is read. **Sentence case
everywhere** (including stat labels). Never use `text-transform:uppercase` or
`letter-spacing` on any text. No slide furniture: pills, subtitles, summary
lines, eyebrows, kickers, badges, chips, tags, and captions do not belong
anywhere on a slide. The cover is the deck title over its hero image and
nothing else.

三条规则适用于每一张幻灯片，且优先于任何参考文档的阅读。**全部使用句首大写**（包括数据标签）。绝不对任何文本使用 `text-transform:uppercase` 或 `letter-spacing`。不放幻灯片装饰元素：胶囊标签、副标题、摘要行、眉题、引导语、徽章、芯片、标签和图注一律不得出现在幻灯片上。封面就是叠在主视觉图上的幻灯片标题，除此之外没有任何其他元素。

| Task | Read first |
|---|---|
| New deck | `/opt/hatch/skills/artifacts/presentation/references/workflow.md` plus the references it names |
| Edit an existing deck | `/opt/hatch/skills/artifacts/presentation/references/editing.md`, plus the references it names |
| Design and layout | `/opt/hatch/skills/artifacts/presentation/references/visual.md` |
| Theme generation or application | `/opt/hatch/skills/artifacts/presentation/references/theme.md` |
| Slides that plot data | `/opt/hatch/skills/artifacts/references/charts.md` (shared) |
| Slides that show a place or a map | `/opt/hatch/skills/artifacts/references/maps.md` (shared) |

| 任务 | 先阅读 |
|---|---|
| 新建幻灯片 | `/opt/hatch/skills/artifacts/presentation/references/workflow.md` 及其提到的参考文档 |
| 编辑已有幻灯片 | `/opt/hatch/skills/artifacts/presentation/references/editing.md` 及其提到的参考文档 |
| 设计与布局 | `/opt/hatch/skills/artifacts/presentation/references/visual.md` |
| 主题的生成或应用 | `/opt/hatch/skills/artifacts/presentation/references/theme.md` |
| 绘制数据的幻灯片 | `/opt/hatch/skills/artifacts/references/charts.md`（共享） |
| 展示地点或地图的幻灯片 | `/opt/hatch/skills/artifacts/references/maps.md`（共享） |

## Scripts / 脚本

| Script | What it does |
|---|---|
| `/opt/hatch/bin/hatch-slide-style` | Compiles the StylePlan; a deck is never built on an invented substitute plan |
| `/opt/hatch/skills/artifacts/scripts/embed_deck_fonts.mjs` | Subsets and embeds webfonts into deck.css; a font that will not download warns, never blocks |
| `/opt/hatch/skills/artifacts/scripts/assemble_deck.mjs` | Validates slide files against the manifest, enforces CSS scoping, emits the combined document; non-zero exit is unshippable |
| `/opt/hatch/skills/artifacts/scripts/render_audit.mjs` (shared) | Renders slides and runs the deck gates; `workflow.md` names the flags |
| `/opt/hatch/skills/artifacts/scripts/build_pptx.py` | Exports the validated PNGs as the PPTX and carries each slide's title into its speaker notes as best-effort accessibility text; the daemon-side rebuild path preserves those notes after UI edits |

| 脚本 | 作用 |
|---|---|
| `/opt/hatch/bin/hatch-slide-style` | 编译 StylePlan；绝不允许基于自行编造的替代方案构建幻灯片 |
| `/opt/hatch/skills/artifacts/scripts/embed_deck_fonts.mjs` | 对 Web 字体做子集化并嵌入 deck.css；无法下载的字体只告警，不阻塞 |
| `/opt/hatch/skills/artifacts/scripts/assemble_deck.mjs` | 依据清单校验幻灯片文件，强制 CSS 作用域，生成合并文档；非零退出码即不可交付 |
| `/opt/hatch/skills/artifacts/scripts/render_audit.mjs`（共享） | 渲染幻灯片并运行幻灯片质量门；具体参数见 `workflow.md` |
| `/opt/hatch/skills/artifacts/scripts/build_pptx.py` | 把校验通过的 PNG 导出为 PPTX，并尽力将每张幻灯片的标题写入其演讲者备注作为无障碍文本；守护进程侧的重建路径会在 UI 编辑后保留这些备注 |

## Verification / 验证

Follow `/opt/hatch/skills/artifacts/testing/SKILL.md`: run the gates, then read every
slide PNG fresh before returning a link.

遵循 `/opt/hatch/skills/artifacts/testing/SKILL.md`：先运行质量门检查，然后在返回链接之前逐一重新查看每张幻灯片的 PNG。
