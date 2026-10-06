---
description: What the words say in a document artifact. Scope check, outline before writing, headings that carry a claim, tone that fits the format, and the read-back that catches drift.
---
<!-- BILINGUAL-EN-ZH -->
# Prose in a document artifact / 文档工件中的正文

## Scope check / 适用范围检查

Read this file when the artifact's value is its text: a PDF or a Word
document, whatever it is called, including a report, a brief, or a letter.
Skip it for slide decks, whose copy rules and plan file live in the
presentation skill, and for spreadsheets, which have no body copy to govern.

当工件的价值在于其文字内容时阅读本文件：PDF 或 Word 文档，无论其名称如何，包括报告、简报或信函。幻灯片请跳过本文件，其文案规则与计划文件位于演示文稿技能中；电子表格也请跳过，它没有需要规范的正文文案。

Two exceptions override every rule below:

以下两个例外优先于下述所有规则：

- **The user asked for it.** Give them the conclusion, the note-form bullets,
  the voice, or anything else a rule here discourages.
  **用户明确要求。** 那就把结论、笔记式要点、那种语气，或本文任何规则不鼓励的其他东西交给他们。
- **This is an edit.** Keep the voice already in the document and change only
  what the user raised. Run the read-back over what you changed; where it
  flags something you were not asked to touch, say so in your final response
  instead of rewriting it.
  **这是一次编辑。** 保持文档中已有的语气，只修改用户提出的内容。对你修改的部分执行回读检查；若检查标记出你未被要求改动的地方，在最终回复中说明即可，不要自行改写。

## Write the outline first / 先写提纲

Write `project_dir/.src/OUTLINE.md` before the first line of body copy, one
entry per section, in this shape:

在写正文第一行之前，先写 `project_dir/.src/OUTLINE.md`，每个章节一条，格式如下：

```text
Section: Why the pilot stalled
Claim:   adoption flattened because onboarding took three weeks, not price.
Source:  data_payload rows 4-11 (weekly signups), user's message on pricing.
```

The outline is the document's argument, so give every section a claim. Cut a
section you cannot find a claim for, and merge two sections that make the same
one. You write the body in pieces, and without the outline those pieces drift
apart.

提纲就是文档的论证，因此每个章节都要给出一个主张。找不到主张的章节就删掉，提出相同主张的两个章节就合并。正文是分片写成的，没有提纲，这些片段就会彼此走样。

`Source` is a working note for you. It never appears in the document.

`Source` 是给你自己的工作备注。它绝不会出现在文档中。

Keep the outline after writing. The read-back checks the document against it,
and the next revision starts from it.

写完后保留提纲。回读检查要以它为基准核对文档，下一次修订也从它开始。

## Put the claim in the heading / 把主张写进标题

State the finding, not the subject. Write "Onboarding is where the funnel
leaks", not "Onboarding analysis". Write "Three suppliers quoted under
budget", not "Supplier quotes". Someone who reads only your headings should
come away with the argument.

陈述发现，而不是主题。写 "Onboarding is where the funnel leaks"，而不是 "Onboarding analysis"。写 "Three suppliers quoted under budget"，而不是 "Supplier quotes"。只读标题的人也应当能把握整篇论证。

Use plain descriptive headings in reference documents instead. A manual,
catalog, invoice, form, glossary, or CV gets looked up rather than read
through, and its reader already knows what they want.

参考类文档则应改用朴素的描述性标题。手册、目录、发票、表单、术语表或简历是被查阅而非通读的，其读者早已知道自己要什么。

- Support every heading in its body. A heading the body does not prove is
  worse than a dull one.
  正文要支撑每个标题。正文证明不了的标题，比一个平淡的标题更糟。
- Organize the information in the title, heading, or caption. Do not echo the
  request back: "Home battery storage brief for a non-technical reader" is the
  prompt, not a title.
  在标题、小标题或图注中组织信息。不要复述请求："Home battery storage brief for a non-technical reader" 是提示词，不是标题。
- Do not close with a section that restates the document. A summary at the
  front that states findings is welcome.
  不要以复述全文的章节收尾。放在开头、陈述发现的摘要是受欢迎的。

## Match the tone to the format / 使语气匹配文体

- Write in a professional register by default, since a document usually gets
  filed, forwarded, or handed to someone else. Follow the request into a
  creative, playful, or personal register when it asks for one.
  默认使用专业语域，因为文档通常会被归档、转发或交给他人。当请求要求时，也可以采用创意、俏皮或个人化的语域。
- Use the established voice of the format the request names. A travel brochure
  sounds like a travel brochure, and an incident review sounds like an
  incident review. Match a reference document or a named style when the user
  supplies one.
  使用请求所指文体的惯有笔调。旅游宣传册就该像旅游宣传册，事故复盘就该像事故复盘。用户提供了参考文档或指定风格时，与其保持一致。
- Address the reader the request names, who is often not the person asking. A
  brief someone forwards to their leadership is written for that leadership.
  面向请求所指明的读者写作，此人往往不是提问者本人。一份会被转发给某人领导层的简报，就是为那个领导层写的。
- Write the whole document in one language, the language of the request,
  unless the user asked for a multilingual document.
  全文使用一种语言写作，即请求所用的语言，除非用户要求多语言文档。
- Write body copy in complete sentences. Keep fragments to labels, table
  cells, and list items with their own grammar.
  正文要用完整的句子。残缺句式只限于标签、表格单元格和具有自身语法的列表项。
- Expand an abbreviation the first time it appears, in every language the
  document uses, then use the short form freely.
  缩写首次出现时展开写全，文档使用的每种语言都如此，之后即可自由使用缩写。
- Write running prose, not stacked keywords. "Two-bedroom flat, Alfama, 1,400
  a month, no lift" is a table row; in a paragraph, make it a sentence.
  写连贯的行文，不要堆砌关键词。"Two-bedroom flat, Alfama, 1,400 a month, no lift" 是表格行；在段落里，把它写成一句话。
- Cut a section that carries nothing, but keep the articles, connectives, and
  explanation a reader needs to follow you. Note-form shorthand, dropped
  articles, and unexplained abbreviations are not concision, and they hurt
  most in a document written for a non-expert.
  删掉没有任何内容的章节，但要保留读者跟上你的思路所需的冠词、连接词和解释。笔记式速记、省略冠词和不作解释的缩写并非简洁，它们对面向非专业读者的文档伤害最大。

## What does not ship / 不应出现在成品中的内容

Cite only what the reader can look up: a page, a publication, a document.
Never cite a tool, an endpoint, a query you ran, or a harness field.
Hyperlink the source where the format allows a link. Grounding itself is
governed by the content-integrity rules in your system prompt; follow those.

只引用读者可以查证的东西：一页纸、一份出版物、一份文档。绝不引用某个工具、某个端点、你运行过的查询或运行环境字段。格式允许链接时，用超链接给出来源。事实依据本身由你系统提示词中的内容完整性规则管辖；请遵循那些规则。

Keep all of this off the page:

以下内容一律不要写进页面：

- How the document was assembled, which instructions you followed, what an
  earlier draft contained, what you excluded, or how confident you are in your
  own checks.
  文档是如何组装的、你遵循了哪些指令、早先的草稿包含什么、你排除了什么，或者你对自己的检查有多大把握。
- Tool names, file paths, the user's prompt quoted back, and images or
  material belonging to another task.
  工具名称、文件路径、被复述的用户提示词，以及属于其他任务的图片或素材。
- Throat-clearing that delays the content: "This section will explore", "It is
  worth noting that", "In conclusion".
  拖延进入正题的清场式套话："This section will explore"、"It is worth noting that"、"In conclusion"。
- Decorative labels carrying no information: unclickable tags, filler chips,
  status pills, an eyebrow line above every section.
  不携带信息的装饰性标签：不可点击的标签、填充性小徽章、状态胶囊、每个章节上方的一行眉题。
- Plausible prose over a gap. This is the most expensive defect here, because
  it reads as the most finished. Say the fact is missing, or leave the slot
  out.
  用貌似合理的行文掩盖事实空缺。这是本文中最昂贵的缺陷，因为它读起来最像成品。要么明说该事实缺失，要么干脆省去这个位置。

【评论】"貌似合理却无依据的行文是最昂贵的缺陷"这一条，把幻觉文本视为比格式错误更严重的失败，体现了对事实真实性的排序。

## Read the prose back / 回读正文

Do this after the content is complete and before you return any link, in the
same pass as the format's visual validation. For a PDF, read the rendered
pages you already rasterized. For a Word document, read the emitted text back.

在内容完成之后、返回任何链接之前执行本步骤，与该格式的视觉校验在同一轮进行。对 PDF，读取你已栅格化的渲染页面；对 Word 文档，回读生成的文本。

Check the document against `.src/OUTLINE.md`:

以 `.src/OUTLINE.md` 为基准核对文档：

- Every outlined section is present and delivers the claim it promised.
  提纲中的每个章节都在场，并兑现了它承诺的主张。
- No section makes a claim the outline does not carry, and nothing rests on a
  fact with nothing behind it.
  没有章节提出提纲中不存在的主张，也没有任何内容建立在无依据的事实之上。
- No two sections repeat the same point in different words.
  没有两个章节用不同措辞重复同一个观点。
- The headings, read alone in order, tell the argument.
  只按顺序读标题，就能读出整篇论证。
- The register holds from first page to last and is still the one the request
  asked for. A document written in pieces drifts formal, drifts chatty, or
  compresses into notes partway through.
  语域从第一页到最后一页保持一致，且仍是请求所要求的语域。分片写成的文档会中途变得过于正式、变得过于闲聊，或压缩成笔记体。
- Nothing about your process, checks, drafts, or tooling reached the page.
  任何关于你的过程、检查、草稿或工具的内容都没有出现在页面上。

Fix what fails, re-render, and read it again. If a fix needs source material
you do not have, say so in your final response rather than writing around it.

修复未通过之处，重新渲染，再读一遍。若某处修复需要你没有的源材料，在最终回复中如实说明，而不是绕过去写。
