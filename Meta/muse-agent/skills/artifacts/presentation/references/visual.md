---
description: Resolving user-provided visual direction into the style plan for slide decks.
---
<!-- BILINGUAL-EN-ZH -->
# Presentation Visual Guidance / 演示文稿视觉指南

User-provided visual direction in `verbatim_request` or the parent conversation
is authoritative. Before styling, take one pass against generic defaults:
make each visual choice because it fits this deliverable, not because it
is the easiest template (one accent everywhere, a single font doing all
the work, no hierarchy between title and body, emoji as icons).

用户在 `verbatim_request` 或父会话中提供的视觉方向具有权威性。在样式处理之前，先对照通用默认值做一次审视：每一个视觉选择都应因为适合本交付物而做出，而不是因为它是最省事的模板（到处使用同一个强调色、让单一字体承担所有工作、标题与正文之间没有层级、用 emoji 当图标）。

- Resolve the StylePlan before authoring a new deck. First resolve the slide
  count using the rule in `workflow.md`, then derive one stable
  kebab-case id per slide from the planned content (grounded in the verbatim
  request and the requesting conversation's supplied content), with `cover` first and `closing` last. Write strict JSON to
  `project_dir/.src/style-plan-input.json` containing `title`, `brief`, and
  `slide_ids`. The brief is the verbatim request plus any explicit visual
  direction from the parent conversation.
  在创作新幻灯片组之前先解析 StylePlan。首先按 `workflow.md` 中的规则确定幻灯片数量，然后根据计划的内容（以逐字请求和请求会话所提供的内容为依据）为每张幻灯片推导一个稳定的 kebab-case id，`cover` 在最前，`closing` 在最后。将包含 `title`、`brief` 和 `slide_ids` 的严格 JSON 写入 `project_dir/.src/style-plan-input.json`。brief 即逐字请求加上父会话中任何明确的视觉方向。
- Run `/opt/hatch/bin/hatch-slide-style plan --input` with the exact input file
  path you wrote as its argument. Never put the title or brief in shell
  arguments. The command's JSON output is the deck's `style_plan`; write it
  unchanged to `project_dir/.src/style_plan.json`. A StylePlan is required: if
  resolution fails, report the build failure instead of inventing a substitute
  plan.
  运行 `/opt/hatch/bin/hatch-slide-style plan --input`，把你写入的确切输入文件路径作为其参数。绝不要把标题或 brief 放进 shell 参数。该命令的 JSON 输出就是幻灯片组的 `style_plan`；将其原样写入 `project_dir/.src/style_plan.json`。StylePlan 是必需的：如果解析失败，报告构建失败，而不要自行编造替代方案。
- On a deck edit, keep the existing `.src/style_plan.json` unless the user asks
  to re-theme it. For a re-theme, run the same command with the existing deck
  title, the requested direction, and the existing slide ids, then replace the
  saved plan with the command's JSON output.
  编辑幻灯片组时，保留现有 `.src/style_plan.json`，除非用户要求重新定制主题。重新定制主题时，用现有的幻灯片组标题、所请求的方向和现有幻灯片 id 运行同一命令，然后用该命令的 JSON 输出替换已保存的方案。
- The resolved `style_plan` is the sole styling authority. Take every color,
  font, layout, and lockup from it; do not restyle the deck directly from
  the request text, the parent conversation, or a saved document theme.
  解析得到的 `style_plan` 是唯一的样式权威。所有颜色、字体、布局和组合（lockup）都取自它；不要直接依据请求文本、父会话或已保存的文档主题重新设计样式。
- Prefer image-led layouts, strong hierarchy, and concise text. At most ~5 bullets or 3-4
  cards per slide; avoid paragraph-heavy slides.
  优先使用以图像为主导的布局、清晰的层级和简洁的文本。每张幻灯片最多约 5 个要点或 3-4 张卡片；避免充满段落的幻灯片。
- Avoid generic corporate slides, stock-looking compositions, decorative filler, and repeated
  title-plus-bullets layouts.
  避免千篇一律的企业风幻灯片、素材图式的构图、装饰性填充物，以及重复的"标题加要点"布局。
- No slide furniture: pills, subtitles, summary lines, eyebrows, kickers, badges, chips, tags,
  and captions do not belong anywhere on a slide. Do not put a small label or section eyebrow
  (e.g. a `3 — The Problem` line) above a heading; put the point in the heading itself.
  幻灯片上不放任何装饰性附属元素（slide furniture）：胶囊标签、副标题、摘要行、眉标、引题（kicker）、徽章、chip、标签和说明文字都不应出现在幻灯片的任何位置。不要在标题上方放置小标签或章节眉标（例如 `3 — The Problem` 这样的行）；把要点放进标题本身。
- The cover is the deck title over its hero image and nothing else: no subtitle, tagline, date
  or source line, kicker, pill, stat box, or button. Save the details for the content slides.
  封面只是主视觉图上的幻灯片组标题，别无其他：没有副标题、宣传语、日期或来源行、引题、胶囊标签、数据框或按钮。细节留给内容页。
- Never shrink text to fit. Body, bullet, and card text stay 16-18px and never drop below
  14px; only short stat labels may sit at 11-13px. If content does not fit at these sizes,
  cut it or split it across slides rather than shrinking the text or tightening spacing.
  绝不为了适应空间而缩小文字。正文、要点和卡片文字保持 16-18px，绝不低于 14px；只有简短的数据标签可以用 11-13px。如果内容在这些字号下放不下，就删减内容或拆分到多张幻灯片，而不是缩小文字或压缩间距。
- Use one theme across the whole deck: take every color from the StylePlan `--slide-*`
  variables (`--slide-bg`, `--slide-fg`, `--slide-primary`, `--slide-accent`, `--slide-muted`)
  rather than raw hex, and give every slide the same background. The deck should read as one
  designed system, not a mix of themes.
  整个幻灯片组使用同一主题：所有颜色都取自 StylePlan 的 `--slide-*` 变量（`--slide-bg`、`--slide-fg`、`--slide-primary`、`--slide-accent`、`--slide-muted`），而不是直接写十六进制色值，并让每张幻灯片使用相同背景。整个幻灯片组应呈现为一个统一的设计系统，而不是多种主题的混搭。
- One idea per slide, and cut rather than cram. Put the slide's point in the heading as a
  claim, highlight at most 3 metrics, and keep a content slide to about 5 bullets or 3-4
  cards. Give each fact one home (do not repeat a chart's numbers as stat cards too; `/opt/hatch/skills/artifacts/references/charts.md` owns charts). When the
  source has more than fits at full size, leave most of it off the slides; a slide with less
  content reads better and lets the text breathe.
  每张幻灯片只讲一个观点，宁可删减也不要塞满。把幻灯片的论点作为主张写进标题，最多突出 3 个指标，内容页保持在约 5 个要点或 3-4 张卡片。让每个事实只有一个落点（不要把图表中的数字再做成数据卡片重复一遍；图表由 `/opt/hatch/skills/artifacts/references/charts.md` 负责）。当素材内容超出满版所能容纳的量时，把大部分内容留在幻灯片之外；内容较少的幻灯片读起来更好，也让文字有呼吸感。
- Do not place unstyled text directly over photos. Use scrims, panels, masks, or split layouts
  for contrast.
  不要把未加样式的文字直接叠放在照片上。使用遮罩（scrim）、面板、蒙版或分栏布局来形成对比。
- Keep text readable at presentation distance. Use clear hierarchy: title, body, stat number,
  stat label. There is no section-label tier; a section's point goes in its heading.
  保证文字在演示距离下可读。使用清晰的层级：标题、正文、数据数字、数据标签。不存在"章节标签"这一层级；章节的要点写进其标题。
- **Sentence case everywhere** (including stat labels). Never use `text-transform:uppercase`
  or `letter-spacing` on any text.
  **一律使用句首大写（sentence case）**（包括数据标签）。绝不对任何文字使用 `text-transform:uppercase` 或 `letter-spacing`。
- Prevent overlap and clipping. Prefer reducing content over shrinking fonts.
  防止元素重叠和文字被裁切。宁可减少内容，也不要缩小字体。
- Every deck carries real imagery, a cover hero and images on content slides, and never an
  empty image box or "visual goes here" placeholder, unless the brief is intentionally
  image-free (step 6).
  每个幻灯片组都配有真实图像——封面主视觉和内容页上的图片，绝不出现空的图像框或"此处放图"占位符，除非 brief 明确要求不使用图像（步骤 6）。
- Keep images relevant to the adjacent claim. Omit an image rather than using a mismatched one.
  保证图像与相邻的论点相关。宁可省略图像，也不要使用不匹配的图像。
- Do not invent facts, dates, metrics, citations, or source names; ground every claim in
  the verbatim request and the requesting conversation's user-stated facts.
  不要编造事实、日期、指标、引用来源或来源名称；每个论点都要以逐字请求和请求会话中用户陈述的事实为依据。
- Read `authoring.md`, `design-system.md`, and
  `image-directive.md` for the detailed visual contract. The slide
  workflow and editing references own planning, validation, and export
  mechanics.
  阅读 `authoring.md`、`design-system.md` 和 `image-directive.md` 了解详细的视觉规范。幻灯片工作流和编辑参考文档负责规划、校验和导出机制。

If a delivered deck has a reported visual problem, re-render the affected pages
to PNG and read them before changing the source. Do not guess at layout or
styling defects without visual evidence.

如果交付的幻灯片组被报告存在视觉问题，先将有问题的页面重新渲染为 PNG 并查看，然后再修改源文件。不要在没有视觉证据的情况下猜测布局或样式缺陷。

【评论】将 style_plan 设为唯一样式权威、禁止依据请求文本直接改样式，是"计划与执行分离"的设计；同时"Do not invent facts"条款用于抑制生成环节的事实幻觉。
