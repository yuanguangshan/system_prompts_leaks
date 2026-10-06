---
description: Resolving user-provided visual direction into the style plan for PDF artifacts.
---

<!-- BILINGUAL-EN-ZH -->

# PDF Visual Guidance / PDF 视觉指南

User-provided visual direction in `verbatim_request` or the parent conversation
is authoritative. Before styling, take one pass against generic defaults:
make each visual choice because it fits this deliverable, not because it
is the easiest template (one accent everywhere, a single font doing all
the work, no hierarchy between title and body, emoji as icons).

`verbatim_request` 或父对话中用户提供的视觉方向具有权威性。在设置样式之前，先对照通用默认做法过一遍：每个视觉选择都应该因为它适合这个交付物而做出，而不是因为它是最省事的模板（到处用同一个强调色、让单一字体承担一切、标题与正文没有层级、把 emoji 当图标）。

- Resolve the document theme before authoring. Read `~/workspace/themes/` and
  its optional `_default.json`, whose `default` field names the preferred theme
  file without `.json`. If the request supplies a complete theme, use it. If it
  carries visual direction, match a saved theme or derive a fitting system
  from that direction. With no explicit direction, use the saved default
  when one exists; otherwise choose a distinctive system fitting the document.
  Do not save a newly derived system as a reusable theme unless the user asked.
  在编写之前先确定文档主题。读取 `~/workspace/themes/` 及其可选的 `_default.json`，其 `default` 字段指定首选主题文件名（不含 `.json`）。如果请求提供了完整主题，就使用它。如果请求带有视觉方向，则匹配一个已保存的主题，或从该方向推导出合适的系统。没有明确方向时，若存在已保存的默认值则使用它；否则选择一个适合该文档且有辨识度的系统。除非用户要求，不要把新推导的系统保存为可复用主题。
- The resolved theme's `ui` block controls typography, color, spacing, shape,
  and hierarchy; its `visual` block controls generated or hero imagery.
  所确定主题的 `ui` 块控制排版、颜色、间距、形状和层级；其 `visual` 块控制生成或主视觉图像。
- Map every requested font to an installed local family verified with
  `fc-list`. PDF rendering must not depend on Google Fonts or Microsoft fonts.
  把每个请求的字体映射到用 `fc-list` 验证过的本地已安装字族。PDF 渲染不得依赖 Google Fonts 或 Microsoft 字体。
- Keep one consistent design system across every page: the same fonts, the
  same spacing scale, the same treatment for the same kind of element. A
  reader flicking through should not be able to tell where you stopped and
  started. Use readable contrast, deliberate whitespace, and a restrained
  hierarchy instead of generic cards or decorative filler. Whitespace is part
  of the system: under fit pressure, cut content, never the gaps.
  在每一页上保持同一个一致的设计系统：相同的字体、相同的间距标尺、同类元素相同的处理方式。快速翻阅的读者不应该能看出你在哪里停笔、从哪里开始。使用可读的对比度、有意图的留白和克制的层级，而不是通用卡片或装饰性填充物。留白是系统的一部分：在排版空间吃紧时，删减内容，绝不删减间隙。
- Design to the document's TYPE, not to a generic report template. A form, an
  invoice, a CV, a manual, an academic paper, a brochure, and an image gallery
  each have established conventions a reader expects; follow the one that fits
  the ask.
  按文档的类型设计，而不是按通用报告模板设计。表单、发票、简历、手册、学术论文、宣传册和图片集各自都有读者预期的成规；遵循符合要求的那一种。
- Page background is one of those conventions: settle what this type of
  document usually sits on before you set one. Invoices, forms, and academic
  papers are normally plain white. Take that as the default and deviate only
  with a reason.
  页面背景就是这些成规之一：在设置背景之前，先弄清这类文档通常以什么为底。发票、表单和学术论文通常是纯白底。以此为默认，只在有理由时才偏离。
- Decorative accents answer to the same test: work out whether the genre
  usually carries decorative color before adding any. Invoices, forms, data
  tables, and academic papers normally have no colored top band or side
  stripe; their structure comes from typography and alignment, with thin
  neutral rules where needed. Reserve accent bands and stripes for genres
  where branding is the point.
  装饰性点缀同样要经受这个检验：在添加任何装饰色之前，先弄清该体裁通常是否带有装饰色。发票、表单、数据表和学术论文通常没有彩色顶栏或侧条；它们的结构来自排版和对齐，必要时辅以纤细的中性分隔线。把强调色带和色条留给以品牌展示为核心的体裁。
- Use a standard page size in portrait with print-safe margins, unless the
  user asks otherwise. Keep anything load-bearing out of the outer margin.
  除非用户另有要求，使用纵向的标准页面尺寸和适合打印的页边距。不要把任何承重内容放进外边距。
- Use headings and bullets on dense pages, styled so the same level always
  looks the same. Leave them out where the page carries little information.
  在信息密集的页面上使用标题和项目符号，并让同级元素的样式始终一致。在信息量很少的页面上则省去它们。
- Give every element on the page a reason to be there. A chart or diagram
  earns its place by carrying information the text cannot. Photos follow the
  image guidance in your build contract: documents about everyday life
  (travel, food, lifestyle) read as unfinished without real photos of the
  places and things they name, while dense data and utility documents read
  better image-free.
  让页面上的每个元素都有存在的理由。图表或示意图靠承载文字无法表达的信息赢得位置。照片遵循构建契约中的图像指引：关于日常生活（旅行、美食、生活方式）的文档若没有所写地点和事物的真实照片会显得未完成，而高密度数据和工具类文档则不用图片更好。
- Treat an empty-feeling area as a layout problem to solve, not a cue to add a
  card, a chip, or a stock illustration.
  把感觉空旷的区域当作需要解决的排版问题，而不是添加卡片、小标签或库存插图的提示。
- Use full-bleed imagery only for an intentional cover. Practical reports,
  guides, manuals, and data-heavy documents use normal in-page images that do
  not compete with the content.
  只在有意设计的封面上使用满版图像。实用报告、指南、手册和数据密集型文档使用不与内容争夺注意力的普通页内图片。
- Read `workflow.md` (beside this file) for HTML authoring, pagination,
  rendering, and visual validation mechanics. Saved-theme resolution for a
  document follows the `~/workspace/themes/` rule above; the presentation
  skill's StylePlan system is deck-only and does not apply here.
  阅读 `workflow.md`（与本文件同目录）了解 HTML 编写、分页、渲染和视觉校验机制。文档的已存主题解析遵循上述 `~/workspace/themes/` 规则；演示技能的 StylePlan 系统仅适用于幻灯片，在此不适用。

If a delivered PDF has a reported visual problem, re-render the affected pages
to PNG and read them before changing the source. Do not guess at layout or
styling defects without visual evidence.

如果交付的 PDF 被报告存在视觉问题，先把受影响的页面重新渲染为 PNG 并查看，然后再修改源文件。没有视觉证据时不要猜测排版或样式缺陷。
