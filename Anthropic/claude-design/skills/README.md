<!-- BILINGUAL-EN-ZH -->
# Skills / 技能

One folder per skill, each holding a `SKILL.md` with YAML frontmatter and the verbatim prompt text. Extracted from the live environment (reconciled August 19, 2026).

每个技能一个文件夹，各自包含一份带 YAML front matter 的 `SKILL.md` 与逐字记录的提示词文本。提取自线上环境（2026 年 8 月 19 日核对）。

Frontmatter follows the schema this project's own `SKILL.md` uses — `name`, `description`, `user-invocable`, nothing else. The environment serves skills as plain prompt text with no frontmatter of their own, so `name` and `description` are reconstructed from the skill list in the system prompt.

front matter 遵循本项目自身 `SKILL.md` 所用的模式——`name`、`description`、`user-invocable`，别无其他。环境以纯提示词文本的形式提供技能，技能本身不带 front matter，因此 `name` 与 `description` 是根据系统提示词中的技能清单重建的。

The system prompt and the raw tool schemas live in `claude-design.md` at the project root. Starter component sources live in `starter-components/`.

系统提示词与原始工具 schema 位于项目根目录的 `claude-design.md`。起始组件源码位于 `starter-components/`。

## Built-in skills / 内置技能

All 19 are user-invocable from the slash menu, in this order:

全部 19 个技能都可由用户从斜杠菜单调用，顺序如下：

**Create / 创建**

| Skill | Folder |
|---|---|
| Make a deck | `skills/make-a-deck/` |
| Make a doc | `skills/make-a-doc/` |
| Interactive prototype | `skills/interactive-prototype/` |
| Wireframe | `skills/wireframe/` |
| Animated video | `skills/animated-video/` |
| Create design system | `skills/create-design-system/` |
| Frontend design | `skills/frontend-design/` |
| Maps & geography | `skills/maps-geography/` |
| 3D object | `skills/3d-object/` |
| HTML email | `skills/html-email/` |
| Flier | `skills/flier/` |

| 技能 | 文件夹 |
|---|---|
| 制作演示文稿 | `skills/make-a-deck/` |
| 制作文档 | `skills/make-a-doc/` |
| 交互式原型 | `skills/interactive-prototype/` |
| 线框图 | `skills/wireframe/` |
| 动画视频 | `skills/animated-video/` |
| 创建设计系统 | `skills/create-design-system/` |
| 前端设计 | `skills/frontend-design/` |
| 地图与地理 | `skills/maps-geography/` |
| 3D 对象 | `skills/3d-object/` |
| HTML 邮件 | `skills/html-email/` |
| 传单 | `skills/flier/` |

**Enhance / 增强**

| Skill | Folder |
|---|---|
| Make tweakable | `skills/make-tweakable/` |
| Claude API in prototypes | `skills/claude-api-in-prototypes/` |

| 技能 | 文件夹 |
|---|---|
| 使可调整 | `skills/make-tweakable/` |
| 原型中的 Claude API | `skills/claude-api-in-prototypes/` |

**Research & data / 研究与数据**

| Skill | Folder |
|---|---|
| Web research | `skills/web-research/` |

| 技能 | 文件夹 |
|---|---|
| 网络研究 | `skills/web-research/` |

**Export & handoff / 导出与交接**

| Skill | Folder |
|---|---|
| Save as PDF | `skills/save-as-pdf/` |
| Export as PPTX (editable) | `skills/export-as-pptx-editable/` |
| Export as PPTX (screenshots) | `skills/export-as-pptx-screenshots/` |
| Save as standalone HTML | `skills/save-as-standalone-html/` |
| Handoff to Claude Code | `skills/handoff-to-claude-code/` |

| 技能 | 文件夹 |
|---|---|
| 另存为 PDF | `skills/save-as-pdf/` |
| 导出为 PPTX（可编辑） | `skills/export-as-pptx-editable/` |
| 导出为 PPTX（截图） | `skills/export-as-pptx-screenshots/` |
| 另存为独立 HTML | `skills/save-as-standalone-html/` |
| 交接给 Claude Code | `skills/handoff-to-claude-code/` |

## Internal skills / 内部技能

Built in and fetchable, but absent from the system prompt's skill list and the slash menu — Claude invokes these itself, the user never picks them.

已内置且可获取，但不出现在系统提示词的技能清单和斜杠菜单中——这些由 Claude 自行调用，用户永远不会主动选择。

| Skill | Folder |
|---|---|
| Hi-fi design | `skills/hi-fi-design/` |
| Options | `skills/options/` |

| 技能 | 文件夹 |
|---|---|
| 高保真设计 | `skills/hi-fi-design/` |
| 选项 | `skills/options/` |

## Changed in the August 19, 2026 reconciliation / 2026 年 8 月 19 日核对中的变更

- **Animated video** rewritten for the `animations_v3.jsx` continuous-composition engine (one element tree keyed to an authored clock, `OM_SCENES` / `OM_PLAYBACK` write-back contract, `<Shot>` / `<Captions>`). `animations_v2.jsx` is no longer offered by `copy_starter_component`.
  **动画视频**针对 `animations_v3.jsx` 连续合成引擎重写（单一元素树绑定一个作者设定的时钟、`OM_SCENES` / `OM_PLAYBACK` 回写契约、`<Shot>` / `<Captions>`）。`copy_starter_component` 不再提供 `animations_v2.jsx`。
- **Flier** now builds on the `doc_page.js` starter (explicitly paginated single `<section class="page">`) instead of hand-rolled print CSS.
  **传单**现在基于 `doc_page.js` 起始组件构建（显式分页的单个 `<section class="page">`），不再手写打印 CSS。
- **Make a doc** rewritten around `doc_page.js` — flowing pages vs fixed sheet, CSS-columns print rules; the old `<main class="doc">` layout/typography guidance is gone.
  **制作文档**围绕 `doc_page.js` 重写——流式页面与固定页面之分、CSS 多列打印规则；旧版 `<main class="doc">` 的布局/排版指导已移除。
- **Make a deck** now says to ask with `ask_user` (including a design-system question); the duplicated slide-writing/planning blocks were deduplicated.
  **制作演示文稿**现在要求通过 `ask_user` 提问（包含设计系统问题）；重复的幻灯片撰写/规划代码块已被去重。
- **Save as PDF** adds the `omelette-print-source` provenance stamp, the doc_page rebuild path, and the fixed-canvas page decision.
  **另存为 PDF** 新增了 `omelette-print-source` 来源戳、doc_page 重建路径，以及固定画布页面的决策。
- **questions_v2** replaced by **ask_user**: 13 question kinds (adds chips, segmented, select, color, user-questions, file-options, design-system, code-source), `prompt` subhead, `follow_up` rounds, decide-for-me and skip semantics.
  **questions_v2** 被 **ask_user** 取代：13 种问题类型（新增 chips、segmented、select、color、user-questions、file-options、design-system、code-source）、`prompt` 小标题、`follow_up` 轮次、"替我决定"与"跳过"语义。
- System prompt gained **Tool search**, an expanded **GitHub** section (`github.md` receipt file, screen map, one-turn sync), **Additional design guidance**, and the injected **default aesthetic** / **system-info** blocks.
  系统提示词新增了**工具搜索**、扩充的 **GitHub** 章节（`github.md` 回执文件、屏幕映射、单轮同步）、**额外设计指导**，以及注入的**默认审美** / **系统信息**块。
- New tools documented: `github_compare`, `tool_search_tool_bm25`. `connect_github` is now a no-op banner superseded by the code-source question.
  新增了工具文档：`github_compare`、`tool_search_tool_bm25`。`connect_github` 现在只是一个无操作横幅，已被代码来源问题取代。
- Verified unchanged: Interactive prototype, 3D object, Web research, HTML email, Make tweakable, Claude API in prototypes, Frontend design, Wireframe, both PPTX exports, Create design system, Save as standalone HTML, Handoff to Claude Code, Maps & geography, Hi-fi design, Options.
  经核实未变化：交互式原型、3D 对象、网络研究、HTML 邮件、使可调整、原型中的 Claude API、前端设计、线框图、两个 PPTX 导出、创建设计系统、另存为独立 HTML、交接给 Claude Code、地图与地理、高保真设计、选项。

Removed in earlier reconciliations: **Send to Canva** (dropped from the built-in list), **Canvas** (replaced by Options — `design_canvas.jsx` no longer exists), **Read PDF** (`read_skill_prompt` no longer serves it, though the system prompt's workflow section still mentions invoking it).

在更早的核对中移除：**发送到 Canva**（从内置清单中删除）、**画布**（被"选项"取代——`design_canvas.jsx` 已不存在）、**读取 PDF**（`read_skill_prompt` 不再提供它，尽管系统提示词的工作流章节仍提及调用它）。
