<!-- BILINGUAL-EN-ZH -->
---
name: create-design-system
description: "Skill to use if user asks you to create a design system or UI kit"
user-invocable: true
---

# Create design system / 创建设计系统

Design system creation instructions:
设计系统创建说明：

Design systems are folders on the file system containing typography guidelines, colors, assets, brand style and tone guides, css styles, and React recreations of UIs, decks, etc. They give design agents the ability to create designs against a company's existing products, and create assets using that company's brand. Design systems should contain real visual assets (logos, brand illustrations, etc), low-level visual foundations (e.g. typography specifics; color system, shadow, border, spacing systems), reusable UI components, and high-level UI kits (full screens).

设计系统是文件系统上的文件夹，其中包含排版规范、颜色、素材、品牌风格与语气指南、CSS 样式，以及用 React 复刻的 UI、幻灯片等。它们让设计智能体能够参照某公司现有产品进行设计，并使用该公司的品牌创作素材。设计系统应包含真实的视觉素材（logo、品牌插画等）、底层视觉基础（如排版细节；颜色系统、阴影、边框、间距系统）、可复用的 UI 组件，以及高层级的 UI kit（完整屏幕）。

No need to invoke the create_design_system skill; this is it.

无需调用 create_design_system skill；本文件就是它。

An automated compiler reads this project, bundles the components into a runtime library, and indexes the styles. It discovers everything from file content and sibling relationships — not from folder names — so the only fixed location is:

一个自动化编译器会读取本项目，把组件打包进运行时库，并为样式建立索引。它从文件内容和同级文件关系中发现一切——而不是从文件夹名称——因此唯一固定的位置是：

- `styles.css` at the project root (or `index.css` / `globals.css` / `global.css` / `main.css` / `theme.css` / `tokens.css` — first match wins). This is the global-CSS entry point; consumers link this one file. Keep it as a list of `@import` lines only. Everything it transitively `@import`s is shipped to consumers; `@font-face` rules anywhere in that closure declare the webfonts.
  项目根目录下的 `styles.css`（或 `index.css` / `globals.css` / `global.css` / `main.css` / `theme.css` / `tokens.css`——按首个匹配生效）。这是全局 CSS 入口；使用方只链接这一个文件。保持它只含 `@import` 行。它直接或间接 `@import` 的所有文件都会交付给使用方；该闭包内任何位置的 `@font-face` 规则声明 web 字体。

Organize everything else however suits the brand. A sensible default layout (use it unless the attached codebase or brand has its own convention):

其余内容按适合该品牌的方式任意组织。一个合理的默认布局（除非所附代码库或品牌有自己的约定，否则请使用它）：

- `tokens/` — CSS custom properties, one file per concern (`colors.css`, `typography.css`, `spacing.css`, …), each `@import`ed from `styles.css`.
  `tokens/`——CSS 自定义属性，每个关注点一个文件（`colors.css`、`typography.css`、`spacing.css` 等），均由 `styles.css` `@import`。
- `components/<group>/` — reusable React UI primitives.
  `components/<group>/`——可复用的 React UI 基元。
- `ui_kits/<product>/` — full-screen click-through recreations of real product views.
  `ui_kits/<product>/`——真实产品视图的全屏可点击复刻。
- `guidelines/` — foundation specimen cards and deeper-dive prose.
  `guidelines/`——基础样本卡片与深入说明文字。
- `assets/` — logos, icons, illustrations, imagery.
  `assets/`——logo、图标、插画、图像。
- `readme.md` (root) — the design guide and manifest.
  `readme.md`（根目录）——设计指南与清单。

What the compiler looks for, regardless of path:

编译器关注的内容（与路径无关）：

- A **component** is any `<Name>.jsx` / `<Name>.tsx` (PascalCase stem) with a sibling `<Name>.d.ts` in the same directory. Add `<Name>.prompt.md` alongside, and one `@dsCard`-tagged `.html` per directory (its first line is `<!-- @dsCard group="…" -->`; details under "Components" below).
  **组件**是指同目录下带有同级 `<Name>.d.ts` 的任意 `<Name>.jsx` / `<Name>.tsx`（文件名主干为 PascalCase）。同时添加 `<Name>.prompt.md`，且每个目录一张带 `@dsCard` 标记的 `.html`（其首行为 `<!-- @dsCard group="…" -->`；详见下文 "Components" 一节）。
- A **token** is any `--*` custom property declared under `:root` (or a single-selector theme scope) in a file reachable from `styles.css`.
  **令牌（token）**是指从 `styles.css` 可达的文件中，声明在 `:root` 下（或单一选择器主题作用域内）的任意 `--*` 自定义属性。
- A **font** is any `@font-face` rule in that same closure; its `src: url(…)` targets are the binaries shipped to consumers.
  **字体**是指同一闭包内的任意 `@font-face` 规则；其 `src: url(…)` 指向的二进制文件会交付给使用方。

To begin, create a todo list with the tasks below, then follow it:

首先，把下面的任务创建为待办清单，然后按清单执行：

- Explore provided assets and materials to gain a high-level understanding of the company/product context, the different products represented, etc. Read each asset (codebase, figma, file etc) and see what they do. Find some product copy; examine core screens; find any design system definitions.
  浏览所提供的素材与资料，从高层级理解公司/产品背景、涉及的不同产品等。逐个阅读每份素材（代码库、figma、文件等），了解其用途。找一些产品文案；查看核心界面；寻找任何设计系统定义。
- Create a readme.md (root) with the high-level understanding of the company/product context, the different products represented, etc. Mention the sources you were given: full Figma links, GitHub repos, codebase paths, etc. Do not assume the reader has access, but store in case they do.
  创建 readme.md（根目录），写入对公司/产品背景、涉及的不同产品等的高层级理解。提及你拿到的来源：完整 Figma 链接、GitHub 仓库、代码库路径等。不要假设读者拥有这些访问权限，但仍要记录在案以备不时之需。
- Call set_project_title with a short name derived from the brand/product (e.g. "Acme Design System"). This replaces the generic placeholder so the project is findable.
  调用 set_project_title，传入从品牌/产品派生的短名称（如 "Acme Design System"）。这会替换通用占位符，使项目可被检索。
- IF any slide decks attached, use your repl tool to look at them, extract key assets + text, write to disk.
  如果附有幻灯片文档，用 repl 工具查看它们，提取关键素材与文字并写入磁盘。
- Explore the codebase and/or figma design contexts and write the token CSS files — CSS custom properties on `:root`, both base values (`--fg-1`, `--font-serif-display`) and semantic aliases (`--text-body`, `--surface-card`). Copy any webfonts/ttfs into the project and write the `@font-face` rules in a CSS file. Then write the root `styles.css` as a list of `@import` lines only (never inline rules there) that reaches every token and font-face file.
  探索代码库和/或 figma 设计上下文，编写令牌 CSS 文件——`:root` 上的 CSS 自定义属性，既包括基础值（`--fg-1`、`--font-serif-display`），也包括语义别名（`--text-body`、`--surface-card`）。把 web 字体/ttf 复制进项目，并在某个 CSS 文件中编写 `@font-face` 规则。然后把根 `styles.css` 写成只含 `@import` 行的列表（绝不在其中内联规则），并确保它能到达每个令牌文件和 font-face 文件。
- Explore, then update readme.md with a CONTENT FUNDAMENTALS section: how is copy written? What is tone, casing, etc? I vs you, etc? are emoji used? What is the vibe? Include specific examples
  探索后更新 readme.md，添加 CONTENT FUNDAMENTALS（内容基础）章节：文案如何撰写？语气、大小写等如何处理？用"我"还是"你"等？是否使用 emoji？整体氛围是什么？并附上具体示例
- Explore, update readme.md with VISUAL FOUNDATIONS section that talks about the visual motifs and foundations of the brand. Colors, type, spacing, backgrounds (images? full-bleed? hand-drawn illustrations? repeating patterns/textures? gradients?), animation (easing? fades? bounces? no anims?), hover states (opacity, darker colors, lighter colors?), press states (color? shrink?), borders, inner/outer shadow systems, protection gradients vs capsules, layout rules (fixed elements), use of transparency and blur (when?), color vibe of imagery (warm? cool? b&w? grain?), corner radii, what do cards look like (shadow, rounding, border), etc. whatever else you can think of. answer ALL these questions.
  探索后更新 readme.md，添加 VISUAL FOUNDATIONS（视觉基础）章节，阐述品牌的视觉母题与视觉基础。颜色、字体、间距、背景（图片？全出血？手绘插画？重复图案/纹理？渐变？）、动画（缓动？淡入淡出？回弹？无动画？）、悬停状态（透明度、更深的颜色、更浅的颜色？）、按压状态（变色？缩小？）、边框、内/外阴影系统、保护性渐变与胶囊形、布局规则（固定元素）、透明与模糊的使用（何时用？）、图像的色调氛围（暖？冷？黑白？颗粒感？）、圆角半径、卡片的外观（阴影、圆角、边框）等，以及你能想到的其他方面。回答所有这些问题。
- If you are missing font files, find the nearest match on Google Fonts. Flag this substitution to the user and ask for updated font files.
  如果缺少字体文件，在 Google Fonts 上找最接近的替代。向用户标明这一替换，并索要更新的字体文件。
- As you work, create foundation specimen cards (small HTML files) that populate the Design System tab. Target ~700×150px each (400px max) — err toward MORE small cards, not fewer dense ones. Split at the sub-concept level: separate cards for primary vs neutral vs semantic colors; display vs body vs mono type; spacing tokens vs a spacing-in-use example. A typical foundations set is 12–20+ cards. Skip titles and framing — the card name renders OUTSIDE the card, so just show the swatches/specimens/tokens directly with minimal decoration. Each card links `styles.css` (relative path from wherever you put it) so it picks up the real tokens. Tag each card with `<!-- @dsCard group="<Group>" viewport="700x<height>" subtitle="<one line>" name="<Card name>" -->` as its first line — the Design System tab renders every tagged `.html` in the project, grouped verbatim by `group`. Suggested groups: "Type", "Colors", "Spacing", "Brand" — title-cased, consistent.
  工作过程中，创建基础样本卡片（小型 HTML 文件）来填充 Design System 标签页。每张目标约 700×150px（最大 400px）——宁可更多的小卡片，也不要更少的高密度卡片。在子概念层面拆分：primary 色与 neutral 色与语义色各分卡片；display 字型与 body 与 mono 各分卡片；间距令牌与间距实际用例各分卡片。一套典型的基础卡片是 12–20+ 张。不要标题和边框装饰——卡片名称渲染在卡片之外，因此直接以最少的装饰展示色板/样本/令牌即可。每张卡片链接 `styles.css`（从卡片所在位置出发的相对路径），从而取到真实令牌。每张卡片首行加上 `<!-- @dsCard group="<Group>" viewport="700x<height>" subtitle="<one line>" name="<Card name>" -->` 标记——Design System 标签页会渲染项目中每个带标记的 `.html`，并按 `group` 原样分组。建议分组："Type"、"Colors"、"Spacing"、"Brand"——首字母大写、保持一致。
- Copy logos, icons and other visual assets into `assets/`. **If the provided sources contain no logo, do not create one**: render the brand name in plain type wherever a mark would go and note the absence in readme.md. Never draw, reconstruct, or approximate a company's real logo or brand mark from memory — even when the company seems identifiable from font names or sample content — and never rebrand the design system with a company identity the user didn't provide. Update readme.md with an ICONOGRAPHY section describing the brand's approach to iconography. Answer ALL these and more: are certain icon systems used? is there a builtin icon font? are there SVGs used commonly, or png icons? (if so, copy them in!) Is emoji ever used? Are unicode chars used as icons? Make sure to copy key logos, background images, maybe 1-2 full-bleed generic images, and ALL generic illustrations you find. NEVER draw your own SVGs or generate images; COPY icons programmatically if you can.
  将 logo、图标和其他视觉素材复制到 `assets/`。**如果所给来源中没有 logo，就不要创造一个**：在需要标识的地方用普通字体呈现品牌名，并在 readme.md 中注明缺失。绝不凭记忆绘制、重构或近似公司的真实 logo 或品牌标识——即使从字体名称或示例内容看公司似乎可辨认——也绝不用用户未提供的公司身份为设计系统重新冠名。更新 readme.md，添加 ICONOGRAPHY（图标）章节，描述品牌的图标使用方式。回答以下所有问题及更多：是否使用了特定图标体系？是否内置图标字体？常用的是 SVG 还是 png 图标？（如果是，复制进来！）是否用过 emoji？是否用 unicode 字符充当图标？务必复制关键 logo、背景图片，可能再加 1-2 张全出血的通用图片，以及你找到的所有通用插画。绝不自己画 SVG 或生成图片；能复制就编程式地复制图标。
  【评论】"绝不凭记忆重绘公司 logo"是针对模型幻觉与商标/知识产权风险的防护条款；配套的"缺失就注明"策略把诚实性置于视觉完整性之上。
- For icons: FIRST copy the codebase's own icon font/sprite/SVGs into `assets/` if you can. Otherwise, if the set is CDN-available (e.g. Lucide, Heroicons), link it from CDN. If neither, substitute the closest CDN match (same stroke weight / fill style) and FLAG the substitution. Document usage in ICONOGRAPHY.
  图标方面：首先尽量把代码库自带的图标字体/sprite/SVG 复制到 `assets/`。否则，如果该图标集有 CDN 版本（如 Lucide、Heroicons），就从 CDN 链接。两者都不可行时，用最接近的 CDN 替代品（相同笔画粗细/填充风格）并明确标出该替换。在 ICONOGRAPHY 中记录用法。
- Author the reusable components (see the Components section). Each directory's card HTML must carry `<!-- @dsCard group="Components" … -->` on line 1.
  编写可复用组件（见 Components 一节）。每个目录的卡片 HTML 首行必须带 `<!-- @dsCard group="Components" … -->`。
- For each product given (e.g. app and website), create a UI kit — `{README.md, index.html, Screen1.jsx, …}` in its own directory; see the UI kits section. Verify visually. Make one todo list item for each product/surface.
  对给出的每个产品（如 app 和网站），创建一个 UI kit——`{README.md, index.html, Screen1.jsx, …}` 放在各自的目录中；见 UI kits 一节。进行视觉验证。为每个产品/界面在待办清单中单列一项。
- If you were given a slide template, create sample slides — `{index.html, TitleSlide.jsx, ComparisonSlide.jsx, BigQuoteSlide.jsx, …}` in their own directory. If no sample slides were given, don't create them. Create an HTML file per slide type; if decks were provided, copy their style. Use the visual foundations and bring in logos + other assets. Tag each slide HTML with `<!-- @dsCard group="Slides" viewport="1280x720" -->` on line 1 so the 16:9 frame scales to fit the card.
  如果给了幻灯片模板，创建示例幻灯片——`{index.html, TitleSlide.jsx, ComparisonSlide.jsx, BigQuoteSlide.jsx, …}` 放在它们自己的目录中。如果没有给示例幻灯片，就不要创建。每种幻灯片类型各建一个 HTML 文件；若提供了幻灯片文档，则复制其风格。运用视觉基础，并引入 logo 与其他素材。每张幻灯片 HTML 首行加 `<!-- @dsCard group="Slides" viewport="1280x720" -->` 标记，使 16:9 画框能缩放适配卡片。
- Tag each UI kit's index.html with `<!-- @dsCard group="<Product>" viewport="<design width>x<above-fold height>" -->` — the declared height caps what's shown, so pick the portion worth previewing.
  给每个 UI kit 的 index.html 加上 `<!-- @dsCard group="<Product>" viewport="<design width>x<above-fold height>" -->` 标记——声明的高度限定了展示范围，因此要选出值得预览的部分。
- Update readme.md with a short "index" pointing the reader to the other files available. This should serve as a manifest of the root folder, plus a list of components, ui kits, etc.
  更新 readme.md，加一段简短的 "index"，指引读者查看其余可用文件。它应充当根文件夹的清单，并列出组件、ui kit 等。
- Create SKILL.md file (details below)
  创建 SKILL.md 文件（细节见下文）
- You are done! The Design System tab shows every registered card. Do NOT summarize your output; just mention CAVEATS (e.g. things you were unable to do or unsure) and have a CLEAR, BOLD ASK for the user to help you ITERATE to make things PERFECT.
  完成了！Design System 标签页会显示每张已注册的卡片。不要总结你的产出；只提及注意事项（如你无法完成或不确定的事项），并向用户提出清晰、大胆的请求，请他们帮你迭代打磨至完美。

Components / 组件

- These are the brand's reusable UI primitives. **When a concrete source defines the inventory (a mounted .fig file, a Figma link, a component library in an attached codebase), that inventory IS the component list** — build exactly the families the source defines, nothing more. Do not add primitives a design system "usually" has (Toast, Avatar, Tabs, …) when the source doesn't define them; a component with no counterpart in the source is an invention consumers will trust and designers won't recognize. If an addition is genuinely needed (e.g. an Icon wrapper for a glyph set), list it in readme.md under "Intentional additions" with a one-line reason. Only when NO source defines components (brand-guidelines-only or from-scratch runs) should you author a standard set — Button, IconButton, Input, Select, Checkbox, Radio, Switch, Card, Badge, Tag, Tabs, Dialog, Toast, Tooltip — sized to the brand's needs. Either way, group by concern (e.g. `forms/`, `feedback/`, `navigation/` under whatever parent directory you choose); a single `core/` group is fine for a small set.
  这些是品牌可复用的 UI 基元。**当某个具体来源定义了组件清单（挂载的 .fig 文件、Figma 链接、所附代码库中的组件库）时，该清单就是组件列表**——严格构建来源定义的组件族，不多不少。当来源未定义时，不要添加设计系统"通常都有"的基元（Toast、Avatar、Tabs 等）；来源中没有对应物的组件是一种发明，使用方会信任它而设计师却认不出。如果确实需要新增（如为某字形集加一个 Icon 包装器），在 readme.md 的 "Intentional additions" 下列出并附一行理由。只有在完全没有任何来源定义组件（仅品牌指南或从零开始）时，才编写一套标准组件——Button、IconButton、Input、Select、Checkbox、Radio、Switch、Card、Badge、Tag、Tabs、Dialog、Toast、Tooltip——并按品牌的需要确定规模。无论哪种方式，都按关注点分组（如在你选择的父目录下分 `forms/`、`feedback/`、`navigation/`）；小组件集用单个 `core/` 分组也可以。
- Enumerate before you build: list the source's FULL component inventory FIRST (for a mounted .fig, read /METADATA.md's "Component families" section; for a Figma link, list the file's pages and components via get_design_context), put every family on your todo list, and build ALL of them, tracking progress against that list. Do NOT stop at a "core subset". If you cannot finish, end your turn by reporting exactly which families remain unbuilt and ask the user whether to continue — never end silently incomplete.
  先枚举后构建：首先列出来源的完整组件清单（挂载的 .fig 读 /METADATA.md 的 "Component families" 部分；Figma 链接则通过 get_design_context 列出文件的页面与组件），把每个组件族加入待办清单，并全部构建，对照清单跟踪进度。不要止步于"核心子集"。如果无法完成，结束回合时准确报告哪些组件族尚未构建，并询问用户是否继续——绝不默默烂尾。
- Each component is one file `<Name>.jsx` (or `.tsx`) with `export function <Name>(props) {…}` — a named, PascalCase export; that name becomes the public API and the literal `export` keyword is required so the bundler picks it up. Keep them self-contained: import React only, reference styling via the CSS custom properties (no CSS-in-JS libs, no npm packages). Siblings may import each other with relative paths.
  每个组件一个文件 `<Name>.jsx`（或 `.tsx`），内含 `export function <Name>(props) {…}`——具名的 PascalCase 导出；该名称即公共 API，且必须使用字面的 `export` 关键字，打包器才能识别。保持组件自包含：只导入 React，通过 CSS 自定义属性引用样式（不用 CSS-in-JS 库，不用 npm 包）。同级组件可用相对路径互相导入。
- In the same directory, write `<Name>.d.ts` with the props interface — the sibling `.d.ts` is what gives a component its props contract, adherence rules, and starting-point eligibility; a `.jsx` without one is still bundled and exported under the namespace but gets none of those — and `<Name>.prompt.md` (first line is a one-sentence "what & when", then a small JSX usage example, then notable variants/props).
  在同一目录编写 `<Name>.d.ts`，包含 props 接口——正是同级的 `.d.ts` 赋予组件 props 契约、遵循规则和起始点资格；没有它的 `.jsx` 仍会被打包并在命名空间下导出，但上述待遇一概没有——另外再写 `<Name>.prompt.md`（首行是一句"是什么、何时用"，随后是一小段 JSX 用法示例，再列出值得注意的变体/props）。
- One card HTML per directory (name it whatever you like — e.g. `buttons.card.html`): first line is `<!-- @dsCard group="Components" viewport="700x<height>" name="<Directory label>" -->`. Link `styles.css` via the correct relative path, load the bundle via `<script src="…/_ds_bundle.js">` (relative path to project root), then mount with `const { <Name> } = window.<Namespace>` in a `<script type="text/babel">` block — call `check_design_system` to get the exact `<Namespace>`. Do NOT `<script src>` the `.jsx` directly (its `export` is unreachable from inline script). Show key states/variants (primary/secondary/ghost; sizes; disabled; with icon; etc.). Make it dense and scannable, not a single default render.
  每个目录一张卡片 HTML（文件名随意——如 `buttons.card.html`）：首行为 `<!-- @dsCard group="Components" viewport="700x<height>" name="<Directory label>" -->`。用正确的相对路径链接 `styles.css`，通过 `<script src="…/_ds_bundle.js">`（相对项目根的路径）加载打包产物，然后在 `<script type="text/babel">` 块中用 `const { <Name> } = window.<Namespace>` 挂载——调用 `check_design_system` 获取确切的 `<Namespace>`。不要直接 `<script src>` 引用 `.jsx`（其 `export` 对内联脚本不可达）。展示关键状态/变体（primary/secondary/ghost；尺寸；禁用；带图标；等）。让它密而可扫读，而不是只渲染一个默认状态。
- Do NOT write `_ds_bundle.js`, `_ds_manifest.json`, `_adherence.oxlintrc.json`, or a barrel `index.js` — those are generated automatically.
  不要编写 `_ds_bundle.js`、`_ds_manifest.json`、`_adherence.oxlintrc.json` 或桶文件 `index.js`——它们会自动生成。

Starting points / 起始点

- Consuming projects show a "Starting Points" picker that lets users seed a new design with a component or screen from this system. Entries are opt-in via a tag — separate from `@dsCard` (which populates the Design System tab).
  使用方项目会显示一个 "Starting Points"（起始点）选择器，让用户用本系统中的某个组件或界面来初始化新设计。条目通过一个标记选择性加入——该标记独立于 `@dsCard`（后者填充 Design System 标签页）。
- To mark a component: add `@startingPoint section="<group>" subtitle="<one line>" viewport="<WxH>"` to the JSDoc on its `<Name>.d.ts` props interface. The picker thumbnail is that directory's `@dsCard`-tagged HTML, so make sure it renders sensibly at the declared viewport.
  标记组件：在其 `<Name>.d.ts` props 接口的 JSDoc 中加入 `@startingPoint section="<group>" subtitle="<one line>" viewport="<WxH>"`。选择器缩略图就是该目录带 `@dsCard` 标记的 HTML，因此要确保它在声明的视口下渲染得体。
- To mark a screen: add `<!-- @startingPoint section="<group>" subtitle="<one line>" viewport="<WxH>" -->` as the first line of the HTML file. The screen itself is the thumbnail.
  标记界面：把 `<!-- @startingPoint section="<group>" subtitle="<one line>" viewport="<WxH>" -->` 作为 HTML 文件的首行。界面本身就是缩略图。
- When the user says "create a starting point `<X>`" (or "add `<X>` as a starting point"), write an HTML file with the `<!-- @startingPoint section="…" -->` comment as its first line — any `.html` in the project with that tag is indexed. `ui_kits/<x>/index.html` is the conventional home but not required.
  当用户说 "create a starting point `<X>`"（或 "add `<X>` as a starting point"）时，写一个首行带 `<!-- @startingPoint section="…" -->` 注释的 HTML 文件——项目中任何带该标记的 `.html` 都会被索引。`ui_kits/<x>/index.html` 是惯例位置，但并非必需。
- When the user asks to remove or retitle a starting point, edit the tag. When they ask to change a thumbnail, edit the `@dsCard`-tagged HTML in that component's directory (component) or the screen HTML itself.
  当用户要求移除或重命名某个起始点时，编辑该标记。当他们要求更换缩略图时，编辑该组件目录下带 `@dsCard` 标记的 HTML（组件）或界面 HTML 本身。

UI kit details: / UI kit 细节：

- UI kits are high-fidelity visual + interaction recreations of full interfaces — screens, not primitives. They cut corners on functionality (not 'real production code') but are pixel-perfect, created by reading the original UI code if possible, or using figma's get-design-context. UI kits compose the component primitives you authored above; don't re-implement Button inside a kit. A UI kit's `index.html` must look like a typical view of the product. These are recreations, not storybooks.
  UI kit 是对完整界面的高保真视觉与交互复刻——是屏幕，不是基元。它们在功能上偷工减料（并非"真正的生产代码"），但要像素级精确，尽可能通过阅读原始 UI 代码创建，或使用 figma 的 get-design-context。UI kit 由你前面编写的组件基元组合而成；不要在 kit 内重新实现 Button。UI kit 的 `index.html` 必须看起来像产品的典型视图。这些是复刻，不是故事书。
- To start, update the todo list to contain these steps for each product: (1) Explore codebase + components in Figma (design context) and code, (2) Create 3-5 core screens for each product (e.g. homepage or app) with interactive click-thru components, (3) Iterate visually on the designs 1-2x, cross-referencing with design context.
  开始时，把待办清单更新为每个产品包含这些步骤：(1) 在 Figma（设计上下文）和代码中探索代码库与组件，(2) 为每个产品创建 3-5 个核心屏幕（如主页或 app），带可点击交互的组件，(3) 对照设计上下文对设计做 1-2 轮视觉迭代。
- Figure out the core products from this company/codebase. There may be one, or a few. (e.g. mobile app, marketing website, docs website).
  从该公司/代码库中找出核心产品。可能只有一个，也可能有几个。（如移动 app、营销网站、文档网站。）
- Each UI kit contains JSX (well-factored; small, neat) for that product's surfaces — sidebars, composers, file panels, hero units, headers, footers, blog posts, video players, settings screens, login, etc.
  每个 UI kit 包含该产品各表面的 JSX（拆分良好；小而整洁）——侧边栏、编写器、文件面板、主视觉区、页眉、页脚、博客文章、视频播放器、设置界面、登录等。
- The index.html file should demonstrate an interactive version of the UI (e.g a chat app would show you a login screen, let you create a chat, send a message, etc, as fake)
  index.html 应展示一个可交互版本的 UI（例如聊天应用会显示登录界面、让你创建聊天、发送消息等，均为模拟）
- You should get the visuals exactly right, using design context or codebase import. Don't copy component implementations exactly; make simple mainly-cosmetic versions. It's important to copy.
  视觉必须完全正确，使用设计上下文或代码库导入。不要完全照搬组件实现；做简单的、以外观为主的版本。复制很重要。
- Cover every component family the source defines — coverage means the full enumerated inventory, not a hand-picked subset. Within a UI kit screen you may abbreviate repeated content (e.g. 3 rows standing in for 30 identical ones), but never skip a component family.
  覆盖来源定义的每个组件族——覆盖意味着完整枚举出的清单，而非手工挑选的子集。在 UI kit 屏幕内可以缩略重复内容（如用 3 行代表 30 行相同内容），但绝不跳过任何组件族。
- Do not invent new designs for UI kits. The job of the UI kit is to replicate the existing design, not create a new one. Copy the design, don't reinvent it. If you do not see it in the project, omit, or leave purposely blank with a disclaimer.
  不要为 UI kit 发明新设计。UI kit 的职责是复刻既有设计，而不是创造新设计。复制设计，不要重新发明。如果项目中没有看到，就省略，或特意留空并附免责说明。

Guidance / 指导原则

- Run independently without stopping unless there's a crucial blocker (E.g. lack of Figma access to a pasted link; lack of codebase access).
  独立运行、不要停顿，除非出现关键阻塞（如无法通过粘贴的链接访问 Figma；无法访问代码库）。
- When creating slides and UI kits, avoid cutting corners on iconography; instead, copy icon assets in! Do not create halfway representations of iconography using hand-rolled SVG, emoji, etc.
  制作幻灯片和 UI kit 时，图标不要偷工减料；要把图标素材复制进来。不要用手写 SVG、emoji 等制作半吊子的图标替身。
- CRITICAL: Do not recreate UIs from screenshots alone unless you have no other choice! Use the codebase, or Figma's get-design-context, as a source of truth. Screenshots are much lossier than code; use screenshots as a high-level guide but always find components in the codebase if you can!
  关键：绝不仅凭截图复刻 UI，除非别无选择！以代码库或 Figma 的 get-design-context 为事实来源。截图的信息损失远大于代码；把截图当作高层级指引，但只要可能就在代码库中找到组件！
- The attached kit is the ground truth. When its values differ from the published conventions of a component library it resembles (shadcn, MUI, etc.), the kit wins. Copy exact numeric values — paddings, radii, font sizes, line-heights — from the source; never round or snap them to a 4/8-px grid or a framework default. If the kit says 5px, write 5px, not 4px.
  所附的 kit 是事实来源。当它的取值与某个相似的组件库（shadcn、MUI 等）的公开约定不同时，以 kit 为准。从来源复制精确数值——内边距、圆角、字号、行高；绝不把它们四舍五入或对齐到 4/8 像素网格或框架默认值。kit 写 5px 就写 5px，不是 4px。
  【评论】"kit 取值优先于知名组件库的公开约定、数值不做网格对齐"，体现了以用户所附素材为唯一事实来源的原则，防止模型用训练记忆中的默认值覆盖真实设计。
- Avoid these visual motifs unless you are sure you see them in the codebase or Figma: bluish-purple gradients, emoji cards, cards with rounded corners and colored left-border only
  除非你确信在代码库或 Figma 中看到了这些视觉母题，否则避免使用：蓝紫渐变、emoji 卡片、仅以圆角加彩色左边框为特征的卡片
- Avoid reading SVGs -- this is a waste of context! If you know their usage, just copy them and then reference them.
  避免读取 SVG——这是浪费上下文！如果你知道它们的用途，直接复制然后引用即可。
- When using Figma, use get-design-context to understand the design system and components being used. Screenshots are ONLY useful for high-level guidance. Make sure to expand variables and child components to get their content, too. (get_variable_defs)
  使用 Figma 时，用 get-design-context 来理解所用的设计系统与组件。截图只对高层级指引有用。确保同时展开变量与子组件以获取其内容。（get_variable_defs）
- Stop if key resources are unnecessible: iff a codebase was attached or mentioned, but you are unable to access it via local_ls, etc, you MUST stop and ask the user to re-attach it using the Import menu. These get reattached often; do not complete a design system if you get a disconnect! Similarly, if a Figma url is inaccessible, stop and ask the user to rectify. NEVER go ahead spending tons of time making a design system if you cannot access all the resources the user gave you. This applies mid-run too: if reads start failing or rate-limiting partway through, stop and report exactly what you did and did not read — never infer or invent component names, structures, or values for content you could not read.
  关键资源不可访问时停止：如果附上或提到了代码库，但你无法通过 local_ls 等访问它，必须停止并请用户用 Import 菜单重新附加。这类附件经常掉线；断连时不要继续完成设计系统！同样，如果 Figma 链接无法访问，停止并请用户修正。在无法访问用户给的全部资源时，绝不要硬着头皮花大量时间做设计系统。这一点在运行中途同样适用：如果读取在中途开始失败或被限流，停止并准确报告你读了什么、没读什么——绝不对未能读取的内容臆测或编造组件名、结构或取值。
  【评论】反复强调"资源不可达即停止并如实报告"，是防止模型在输入缺失时编造内容的防幻觉设计。

SKILL.md

- When you are done, we should make this file cross-compatible with Agent SKills in case the user wants to download it and use it in Claude Code.
  完成后，应让该文件与 Agent SKills 交叉兼容，以便用户想下载它并在 Claude Code 中使用。
- Create a SKILL.md file like this:
  按如下方式创建 SKILL.md 文件：

```
<skill-md>
---
name: {brand}-design
description: Use this skill to generate well-branded interfaces and assets for {brand}, either for production or throwaway prototypes/mocks/etc. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for protoyping.
user-invocable: true
---

Read the README.md file within this skill, and explore the other available files.
If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out and create static HTML files for the user to view. If working on production code, you can copy assets and read the rules here to become an expert in designing with this brand.
If the user invokes this skill without any other guidance, ask them what they want to build or design, ask some questions, and act as an expert designer who outputs HTML artifacts _or_ production code, depending on the need.
</skill-md>
```

Additionally, remind the user they need to set the File type to Design System in the Share menu so that others in their org can view this design system.

另外，提醒用户需要在 Share 菜单中把 File 类型设为 Design System，这样其组织内的其他人才能查看该设计系统。
