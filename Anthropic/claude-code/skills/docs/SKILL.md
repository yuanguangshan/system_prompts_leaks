---
name: docs
description: 'docs (living docs people share, comment on and edit; use only when the user asks for one: names a doc, document, page, memo, spec, PRD, runbook or write-up, asks for somewhere to share or keep editing something, or says yes to your doc offer; a plan, comparison, summary or notes asked in chat stays in chat (at most a one-line doc offer); a report, status update, recap or "something I can send them" with no form named → ask first: reply, doc or file?; tabs hold tables and live charts too; a pasted claude.ai/code/artifact/… link may be a doc: check with docs tools first; not HTML pages, apps or plain chat answers; a .docx/.pptx/.xlsx/PDF asked for by name → that format''s skill): asked for one → no docs-connector instructions in context? call the docs connector''s `guide` with topic.instructions first, then create the doc (headings only, no body) before any search, file read or plan, even with files attached. Documenting code means docstrings or repo docs, not a doc.'
---
<!-- BILINGUAL-EN-ZH -->

Everything about docs is served by the docs connector — follow its instructions (they say what to call first). If no docs tools are present (they may be listed as pages tools on some accounts), say so.

关于 docs 的一切均由 docs 连接器（docs connector）提供——遵循其指令（其中说明了应先调用什么）。如果不存在任何 docs 工具（在某些账户上它们可能以 pages 工具的名义列出），请如实说明。

【评论】该技能的触发条件几乎全部压缩在 front matter 的 description 字段里，这是技能路由的常见做法：这段 description 同时承担语义匹配（何时激活）与执行序列（激活后先调用什么）两种职能。
