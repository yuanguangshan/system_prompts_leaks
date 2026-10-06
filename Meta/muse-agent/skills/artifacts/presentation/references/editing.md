<!-- BILINGUAL-EN-ZH -->
---
description: Editing an existing deck in place, from its artifact slug, without rebuilding from scratch.
---

# Slide deck editing / 幻灯片文稿编辑

How to change a deck the user already has, in place, without rebuilding from scratch. You are
given the deck's `<artifact-slug>` and the change request; work on the existing deck at the
`project_dir` your build task names. That is `~/workspace/your_files/<artifact-slug>/` for an
ordinary deck, and the goal's own `files/` directory for a goal document.

介绍如何就地修改用户已有的演示文稿（deck），而不从头重建。你会拿到文稿的 `<artifact-slug>` 和修改请求；请在构建任务指定的 `project_dir` 中处理现有文稿。普通文稿对应 `~/workspace/your_files/<artifact-slug>/`，目标文档（goal document）则对应其自身的 `files/` 目录。

## Load first (always) / 首先加载（始终如此）

The deck's source is per-slide under `.src/slides/` (`workflow.md` step 5). Read only what the
change needs:

文稿的源文件按幻灯片存放在 `.src/slides/` 下（`workflow.md` 第 5 步）。只读取本次修改所需的内容：

- `.src/slides/deck.json`: the deck's slides in order. Read this first — it tells you which file
  is which slide.
  `.src/slides/deck.json`：按顺序排列的文稿幻灯片。先读它——它会告诉你哪个文件对应哪张幻灯片。
- `.src/slides/deck.css`: the theme (`:root` tokens) and every shared rule, plus the theme's
  `@font-face` rules holding the font bytes, at the end of the file. Edit the rules above those
  faces. Leave the faces themselves alone.
  `.src/slides/deck.css`：主题（`:root` 令牌）与所有共享规则，文件末尾还有持有字体字节的主题 `@font-face` 规则。只编辑这些字体面（font face）之上的规则，不要动字体面本身。
- `.src/slides/<id>.html`: only the slide(s) you are about to change. Each holds one
  `<section class="slide">` with its images embedded as `data:` URIs.
  `.src/slides/<id>.html`：仅限你即将修改的幻灯片。每个文件包含一个 `<section class="slide">`，其图片以 `data:` URI 形式内嵌。
- `.src/deck_plan.md`: title, audience, narrative arc, slide list.
  `.src/deck_plan.md`：标题、受众、叙事主线、幻灯片清单。
- `.src/style_plan.json`: archetype, palette, fonts, per-slide `layout_plan`, and `lockups`.
  `.src/style_plan.json`：原型（archetype）、调色板、字体、每页 `layout_plan` 以及 `lockups`。
- `meta.json`: the deck's title and the exact set of output formats it was promoted to.
  `meta.json`：文稿标题，以及它已被导出的输出格式的确切集合。

On a deck with per-slide source, do **not** read or edit `.src/index.html`. It is assembled from
the files above, so reading it costs the whole deck's bytes to change one slide, and an edit there
is discarded by the next assemble. (A legacy deck has no per-slide source, so its `index.html` is
the only copy of the slides; the rebuild below reads it on purpose.)

对带按页源文件的文稿，**不要**读取或编辑 `.src/index.html`。它由上述文件组装而成，读取它意味着为改一张幻灯片付出整份文稿的字节代价，而且对它的修改会在下一次组装时被丢弃。（旧式文稿没有按页源文件，其 `index.html` 是幻灯片的唯一副本；下文的重建流程会刻意读取它。）

【评论】禁止读取组装产物是典型的上下文窗口成本控制：只操作页级源文件，避免整份文档的字节进入模型上下文。

**A deck with no `.src/slides/`, or one whose `.src/slides/` assemble refuses for a reason that
predates your edit** (most often a slide whose `<section>` id does not match its manifest id, which
the old splitter could produce), has no per-slide source you can edit in place: rebuild it in the current format through `workflow.md`, keeping the slug and carrying the
project's own material forward so it stays the same deck (`deck_plan.md` for the arc and slide
list, `style_plan.json` for the theme and layouts, not re-resolved, `.src/media/` for imagery,
and the old `index.html` for each slide's content). Apply the requested change as part of the
rebuild and say the deck was rebuilt, since that re-authors slides the user did not ask about.
Everything below is for a deck that has per-slide source.

**没有 `.src/slides/` 的文稿，或其 `.src/slides/` 组装因早于本次修改的原因而拒绝执行**（最常见是某张幻灯片的 `<section>` id 与清单 id 不匹配——旧的切分器可能产生这种情况）的文稿，没有可就地编辑的按页源文件：请通过 `workflow.md` 以当前格式重建，保留 slug 并沿用项目自身素材，使其仍是同一份文稿（`deck_plan.md` 提供叙事主线与幻灯片清单，`style_plan.json` 提供主题与布局——不重新解析，`.src/media/` 提供图像素材，旧 `index.html` 提供每页内容）。把请求的修改作为重建的一部分完成，并说明文稿经过了重建，因为重建会重新生成用户并未要求改动的幻灯片。下文所有内容都针对带按页源文件的文稿。

Never author a fresh deck from the brief on an edit; mutate the deck that already exists. When
you rewrite or add a slide body, author it per `authoring.md` + `design-system.md` (layout
classes, lockup grid, type scale) exactly as a fresh build, so it matches the surrounding deck.

编辑时绝不要根据简报从零新写一份文稿；只改动已存在的那份。当你重写或新增幻灯片正文时，按照 `authoring.md` + `design-system.md`（布局类、lockup 网格、字号阶梯）像全新构建一样撰写，使其与文稿整体风格一致。

## Apply the change by intent / 按意图应用修改

- **Change or retopic a slide** ("change slide 3 to X", "fix the last slide"): resolve the slide
  number to an id through `deck.json`'s order (slide 1 = first entry), then edit only that
  `<id>.html`. Touch no other slide file, and leave `deck.json` alone.
  **修改或更换某张幻灯片的主题**（"把第 3 张改成 X"、"修一下最后一张"）：通过 `deck.json` 的顺序把幻灯片编号解析为 id（第 1 张 = 第一项），然后只编辑对应的 `<id>.html`。不要碰其他幻灯片文件，也不要动 `deck.json`。
- **Shorten** ("cut it to 5 slides", "tighter"): shortening removes slides — keep the strongest
  points and the narrative arc and drop the weakest whole slides, carrying the survivors'
  content and existing imagery through unchanged (reuse their existing media; do not regenerate,
  re-source, or re-resolve the style plan). Remove each dropped slide's entry from
  `deck.json` and delete its `<id>.html`; the survivors' files and ids do not change. Never
  invent filler to pad a deck out. Only rewrite a slide's copy when the user also asks to
  re-tighten it ("make it punchier").
  **缩短**（"压到 5 张"、"更紧凑"）：缩短意味着删除幻灯片——保留最有力的要点和叙事主线，整张舍弃最弱的幻灯片，被保留幻灯片的内容与既有图像原样沿用（复用其既有媒体；不要重新生成、重新取材或重新解析样式方案）。从 `deck.json` 中移除每张被删幻灯片的条目并删除其 `<id>.html`；被保留幻灯片的文件与 id 均不变。绝不要编造填充内容来凑篇幅。只有当用户还要求把某张改得更精炼时（"更有力"），才重写该张幻灯片的文案。
- **Extend or add** ("add a slide on X"): write a new `<id>.html` that reuses the deck's existing
  layouts and `lockups` — give it a `layout` already used by a sibling of the same kind — and
  insert its entry in `deck.json` at the position it belongs. Give it a short stable
  `kebab-case` `id` for what it is about (per `workflow.md` step 5) and leave every existing
  slide's `id` and file exactly as they are: ids name slides for the rest of the deck's life, so
  never renumber them to match a new position.
  **扩展或新增**（"加一张关于 X 的幻灯片"）：编写一个新的 `<id>.html`，复用文稿既有的布局和 `lockups`——给它一个同类兄弟幻灯片已在使用的 `layout`——并在 `deck.json` 中把它的条目插入应属的位置。给它一个简短稳定、能说明其内容的 `kebab-case` `id`（依 `workflow.md` 第 5 步），并让所有既有幻灯片的 `id` 与文件保持原样：id 在文稿余生中都用于命名幻灯片，绝不要为匹配新位置而重新编号。
  Do not re-run the style CLI to place it; that re-resolves the whole theme and would drift the
  deck's current one. Add imagery only where an image illustrates the new slide's content, per
  `image-directive.md` and `authoring.md`'s restraint rule; a table, stat, or comparison slide
  stays image-free.
  不要为放置它而重跑样式 CLI；那会重新解析整个主题，使文稿当前主题发生漂移。只有当图像能说明新幻灯片内容时才添加图像，遵循 `image-directive.md` 与 `authoring.md` 的克制规则；表格、数据或对比类幻灯片保持无图。
- **Re-theme** ("make it darker", "more minimal", "a different vibe"): resolve a fresh
  `StylePlan` for the new direction through `visual.md`. Apply it exactly, and
  entirely inside `deck.css` — swap the `:root` `css_variables` for the new plan's — then
  overwrite `.src/style_plan.json` so the saved plan and the `:root` block stay in sync. You do
  not touch the fonts by hand. The `:root` tokens name the new families, and step 7's embed replaces
  the previous theme's bytes on its own. Leave the `@font-face` block at the end of `deck.css`
  exactly as it is, and never write a font URL back in. Never hand-pick colors or fonts yourself;
  the palette is the resolved plan's. Keep
  every slide's content and layout, so the slide files stay untouched. The deck's photos were
  graded to the old palette (per `image-directive.md`), so a hue shift (not just lighter/darker)
  leaves image slides looking off. If the cover hero was **generated**, regenerate it to the new
  palette and re-embed it in its slide file; if it is a **sourced real image** (a logo or photo
  from the image-search skill), never regenerate it: keep the image and its per-slide
  `--hero-scrim-color` sampled from the image (`design-system.md`), and re-tune only the
  surrounding `:root` palette from the resolved plan. Tell the user the other photos keep their
  prior grade (offer to regenerate the generated ones).
  **更换主题**（"调暗一些"、"更极简"、"换个风格"）：通过 `visual.md` 为新方向解析一份新的 `StylePlan`。严格按方案应用，且全部在 `deck.css` 内完成——把 `:root` 的 `css_variables` 换成新方案的——然后覆盖写入 `.src/style_plan.json`，使保存的方案与 `:root` 块保持同步。不要手工处理字体。`:root` 令牌会指定新字体族，第 7 步的嵌入会自行替换上一主题的字体字节。`deck.css` 末尾的 `@font-face` 块保持原样，也绝不要把字体 URL 写回去。绝不要自己手工挑颜色或字体；调色板以解析出的方案为准。保留每张幻灯片的内容与布局，因此幻灯片文件保持不动。文稿中的照片是按旧调色板调色的（依 `image-directive.md`），因此色相变化（而不只是变亮/变暗）会让含图幻灯片显得不协调。若封面主视觉是**生成**的，按新调色板重新生成并重新嵌入其幻灯片文件；若它是**取材的真实图像**（来自图像搜索技能的徽标或照片），绝不要重新生成：保留图像及其从图像采样的每页 `--hero-scrim-color`（`design-system.md`），只按解析出的方案微调周边的 `:root` 调色板。告知用户其余照片保留原有调色（并提议可重新生成那些生成图）。
- **Anything else** (reorder, swap one image, resize): apply the minimum edit and leave the rest
  as is. Reorder = move the entry in `deck.json`, every slide file unchanged. Image swap =
  re-source a real subject with the image-search skill, or regenerate a concept image per
  `image-directive.md`, then re-embed its `data:` URI in its own file. Aspect-ratio change is global: update `@page` and `.slide` in `deck.css`, then re-run
  the overflow audit.
  **其他一切**（重排、更换某张图像、调整尺寸）：应用最小修改，其余保持原样。重排 = 移动 `deck.json` 中的条目，所有幻灯片文件不变。换图 = 用图像搜索技能重新取材真实主体，或按 `image-directive.md` 重新生成概念图，然后在其自身文件中重新嵌入 `data:` URI。宽高比变更属于全局变更：更新 `deck.css` 中的 `@page` 与 `.slide`，然后重跑溢出审计。

## Keep the theme unless asked / 除非被要求，否则保留主题

For a content edit (change, shorten, add), do **not** re-resolve the style plan or touch the
`:root` block: keep the deck's existing palette and fonts. Only a re-theme request changes the
theme. Silently restyling a content edit is a bug.

对内容类修改（修改、缩短、新增），**不要**重新解析样式方案，也不要碰 `:root` 块：保留文稿既有的调色板与字体。只有明确的换主题请求才更改主题。对内容修改暗中改变样式属于缺陷。

【评论】把"内容修改不得附带样式变化"明确判定为缺陷，是为了约束代理不放大用户意图，保证行为可预期。

## Re-assemble, re-validate and re-export / 重新组装、重新校验并重新导出

**Re-assemble first** (`workflow.md` step 7). Your edit changed the deck's source, so
`.src/index.html` is stale until `assemble_deck.mjs` runs, and everything below reads that file:

**先重新组装**（`workflow.md` 第 7 步）。你的修改已改变文稿源文件，因此在 `assemble_deck.mjs` 运行之前 `.src/index.html` 是过期的，而下文所有步骤都会读取该文件：

```sh
SLUG="<artifact-slug>"
# `project_dir` comes from your build task. A goal document is built under
# that goal's `files/` directory, so a hardcoded `your_files` path is wrong.
# `project_dir` comes from your build task. Write its leading `~/` as
# `$JARVIS_HOME/`: the shell leaves a tilde literal inside quotes, so
# `"~/workspace/..."` builds into a directory literally named `~`.
SRC="$JARVIS_HOME/<project_dir from the build task, without its leading ~/>/.src"
bun run "/opt/hatch/skills/artifacts/scripts/embed_deck_fonts.mjs" --slides "$SRC/slides"
bun run "/opt/hatch/skills/artifacts/scripts/assemble_deck.mjs" \
  --slides "$SRC/slides" \
  --out "$SRC/index.html"
```

The embed step is a no-op unless the deck's fonts or characters changed, so it is cheap on a
content edit and it is what re-fonts a re-theme.

除非文稿的字体或字符发生变化，嵌入步骤是空操作，因此对内容修改而言代价很小；而换主题时正是靠它重新嵌入字体。

Then run the full `workflow.md` validation loop (steps 8-9) exactly as a fresh build — gate on the
render report (`ok`, `fonts.missing`, `overflow`), read every `.src/validate` PNG, up to 3
iterations; don't shortcut it to a single pass (a re-theme swaps fonts, so `fonts.missing`
matters). On every edit here, a re-theme included, drop `--require-restraint` from the step-8
commands: the probe reads every slide, including ones this edit must leave alone. Fix furniture,
uppercase, and tracking only on the slides you did edit. Drop `--require-generated-imagery` too,
unless this edit regenerated imagery: an existing sourced or chart-only deck has no sidecar to
satisfy it, so keeping the flag leaves a finished edit no legal move. `workflow.md` states the
rule. Mention the untouched slides only if you
actually saw those styles there; a deck built under the gate carries none. Each iteration edits the
slide source and re-assembles before re-rendering. Then
re-export only the formats the deck already has (from `meta.json` `outputs`) via `workflow.md`'s
promote step (step 10) — not a hardcoded PPTX — and recompute `meta.json` per step 11. After a
structural edit (shorten/add), also sync the slide list in `deck_plan.md` and the `layout_plan`
entries in `style_plan.json` to the final deck — metadata only, leave the palette and fonts as
they are. The deck keeps its slug and its links.

然后像全新构建一样完整运行 `workflow.md` 校验循环（第 8-9 步）——以渲染报告（`ok`、`fonts.missing`、`overflow`）为门禁，逐张查看 `.src/validate` PNG，最多 3 轮迭代；不要捷径化为一轮了事（换主题会更换字体，因此 `fonts.missing` 很重要）。对这里的每一次编辑（包括换主题），都要从第 8 步命令中去掉 `--require-restraint`：该探针会读取每一张幻灯片，包括本次编辑必须保持不动的那些。只在你确实编辑过的幻灯片上修整装饰元素、大写与字距。`--require-generated-imagery` 同样去掉，除非本次编辑重新生成了图像：既有的取材型或纯图表文稿没有可满足该要求的 sidecar，保留该标志会让一次已完成的编辑无合规路径可走。`workflow.md` 中写明了此规则。只有当你确实在未改动幻灯片上看到那些样式问题时才提及它们；在该门禁下构建的文稿不会带有此类问题。每轮迭代都先编辑幻灯片源文件并重新组装，再重新渲染。然后仅重新导出文稿已有的格式（来自 `meta.json` 的 `outputs`），经由 `workflow.md` 的 promote 步骤（第 10 步）——而不是硬编码的 PPTX——并按第 11 步重算 `meta.json`。结构性编辑（缩短/新增）之后，还要把 `deck_plan.md` 中的幻灯片清单和 `style_plan.json` 中的 `layout_plan` 条目同步为最终文稿——仅更新元数据，调色板与字体保持不变。文稿保留其 slug 与链接。
