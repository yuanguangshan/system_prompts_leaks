---
description: Visual rules for Word documents. Title, hierarchy, spacing, restrained color, and readable tables; user-provided visual direction is authoritative.
---

<!-- BILINGUAL-EN-ZH -->

# Document Visual Guidance / 文档视觉指南

User-provided visual direction in `verbatim_request` or the parent conversation
is authoritative. Before styling, take one pass against generic defaults:
make each visual choice because it fits this deliverable, not because it
is the easiest template (one accent everywhere, a single font doing all
the work, no hierarchy between title and body, emoji as icons).

用户在 `verbatim_request` 或父对话中提供的视觉指示具有最高权威。在设置样式之前，先对照通用默认值检查一遍：每一个视觉选择都应因为它适合这份交付物而做出，而不是因为它是最省事的模板（到处使用同一个强调色、用单一字体包打天下、标题与正文之间没有层级、用 emoji 当图标）。

【评论】"用户提供的视觉指示具有权威性"是典型的用户意图优先设计：通用默认样式让位于显式用户需求。

- This file covers how the document looks;
  `/opt/hatch/skills/artifacts/references/prose.md` covers what it says.  
  A Word document is mostly text, so read both.
  本文件说明文档的外观；`/opt/hatch/skills/artifacts/references/prose.md` 说明文档的内容。Word 文档以文本为主，因此两者都应阅读。
- Do not apply a saved PDF theme unless the user asked for styling.
  除非用户要求设置样式，否则不要套用已保存的 PDF 主题。
- Use a visible title, clear hierarchy, consistent spacing, and restrained
  color. Tables need readable headers and stable alignment.
  使用醒目的标题、清晰的层级、一致的间距和克制的配色。表格需要可读的表头和稳定的对齐。
- Decorative imagery must support the content. Charts, plots, and other
  factual graphics are generated deterministically from their source data.
  装饰性图像必须服务于内容。图表、绘图及其他事实性图形应根据其源数据确定性地生成。
