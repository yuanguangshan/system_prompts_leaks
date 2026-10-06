<!-- BILINGUAL-EN-ZH -->
# Google Docs reference / Google Docs 参考指南

As a first step, before you create a Google Doc or change one, you must read the design rules below in full: how a document should look and read, and how to edit one. After them, "Applying the design rules" says how to carry out the rules with these tools, and the rest of the file covers the Docs connector's `read_doc` and `update_doc` and the Drive tools that help with Docs.

作为第一步，在创建或修改 Google 文档之前，你必须完整阅读下面的设计规则：文档应呈现何种外观与行文，以及如何编辑文档。读完规则后，"Applying the design rules"（应用设计规则）一节说明如何借助这些工具落实规则，本文件其余部分介绍 Docs 连接器的 `read_doc` 与 `update_doc`，以及配合 Docs 使用的 Drive 工具。

# Document design / 文档设计

When the user asks for something specific, such as a font, a color, a length or a structure, do that; these rules decide what the user left open. The rules for the text itself are under Writing, and the rules for changing a document that already exists are under Editing an existing document.

当用户提出具体要求，例如字体、颜色、篇幅或结构时，照做即可；这些规则只决定用户未明确说明的部分。针对文本本身的规则见 Writing 一节，修改既有文档的规则见 Editing an existing document 一节。

## Structure / 结构

- **Plan the sections before you write, and merge any two that answer the same question in different words.** If the user asked for a set number of sections, replace the merged one with a section that covers something new.
  动笔之前先规划章节，并把用不同措辞回答同一问题的两个章节合并。如果用户指定了章节数量，就用一个涵盖新内容的章节补上被合并的那个。
- **Each section appears once.** Before you finish, read the headings in order and check that none is repeated.
  每个章节只出现一次。完成之前，按顺序通读各标题，确认没有重复。
- **An executive summary opens with the conclusion.** Its first paragraph states what the reader should believe or do. Numbers belong in the body sections, where they support the conclusion; they are not the conclusion. "$1.2B total debt across 7 instruments, weighted-average coupon 4.8%" is data. "Refinancing risk is concentrated in 2027. We recommend an opportunistic tender for the 2027 notes given the current cash position" is a conclusion. A summary that is only a list of numbers has not stated a conclusion.
  执行摘要以结论开篇。第一段就要说明读者应当相信什么或做什么。数字属于正文章节，用来支撑结论；数字本身不是结论。"$1.2B total debt across 7 instruments, weighted-average coupon 4.8%" 是数据。"Refinancing risk is concentrated in 2027. We recommend an opportunistic tender for the 2027 notes given the current cash position" 才是结论。只罗列数字的摘要等于没有给出结论。
- **A length the user gave is a limit.** "Three pages", "one page" or "500 words" is a hard constraint: check it before you finish and cut if you are over. Five pages for a three-page request is a defect, not thoroughness. Count the pages in the rendered document; without a render, estimate about 3,000 characters per page.
  用户给出的篇幅是一个上限。"Three pages"、"one page" 或 "500 words" 都是硬性约束：完成之前核对篇幅，超出就删。要求三页却写出五页是缺陷，不是详尽。应在渲染后的文档中数页数；无法渲染时按每页约 3,000 字符估算。
- **Limits still apply after the first request.** A length or a set of sources the user gave at the start still holds when a later request doesn't repeat it.
  首次请求之后限制依然有效。用户最初给出的篇幅或资料来源范围，在后续请求未重复提及时仍然适用。
- **Numbers and quotes taken from a source match the source.** Take every figure, quote and page reference from a part of the source you have actually read, not from memory. If you can't find a value, say so instead of estimating it.
  取自来源的数字与引文必须与来源一致。每个数字、引文和页码引用都要来自你实际读过的来源部分，而不是凭记忆。找不到某个数值时如实说明，不要估算。

## Styles / 样式

The look of a document comes from its styles: the Normal text style for body text and the built-in heading styles for headings. Formatting set directly on individual paragraphs carries over into the text added after them and spreads across a long document.

文档的外观来自其样式：正文使用 Normal text 样式，标题使用内置标题样式。直接设置在单个段落上的格式会延续到其后新增的文本，并在长文档中蔓延。

- **Set the body font once, for the whole body**, not on each paragraph or run. A font set paragraph by paragraph misses the paragraphs added later, and the document ends up in two fonts.
  **正文字体只设置一次，作用于整个正文**，而不是逐段落或逐文本段（run）设置。逐段设置的字体照顾不到后来新增的段落，最终文档会出现两种字体。
- **Headings use the heading styles.** Never make a heading out of bold, larger body text: it is missing from the table of contents and the outline, and its formatting carries over into the next paragraph.
  标题必须使用标题样式。绝不要用加粗、放大后的正文冒充标题：它不会出现在目录和文档大纲中，其格式还会延续到下一段。
- **Don't resize a single heading.** Each heading level already has its own size, and an override on one paragraph can make a subsection look like a section. If a heading is at the wrong level, change its level. If every heading at one level should be smaller, change that heading style. Don't move a correctly placed heading to a lower level only to make it smaller.
  不要单独调整某一个标题的字号。每个标题层级都有自己的字号，对单个段落强行覆盖会让小节看起来像大节。标题层级放错了，就改它的层级；某一层的所有标题都应更小时，改那一层的标题样式。不要仅仅为了让标题变小，就把位置正确的标题挪到更低的层级。
- **New content of the same kind takes the style around it; new content of a different kind does not.** A clause added next to other clauses gets their style. A table or a body paragraph added right after a list item or a heading gets the Normal text style, without the heading's bold. Otherwise the table cells get list markers and the body text comes out bold or as a heading.
  同类的新内容沿用周边样式；不同类的新内容则不然。插在其他条款旁边的条款沿用它们的样式。紧跟在列表项或标题之后插入的表格或正文段落应使用 Normal text 样式，不带标题的加粗。否则表格单元格会带上列表标记，正文会变成加粗或标题样式。
- **Color goes on a phrase, not on a section.** One sentence in red is emphasis. Three paragraphs in red is too much, and the reader stops reading the color as a signal. If you want to color more than one paragraph, you need a heading or a callout box instead. Color the words themselves, not the whole paragraph, so the color doesn't carry over into the next paragraph.
  颜色只用于短语，不用于整节。一句话标红是强调；连续三段标红就过了，读者会不再把颜色当作信号。想给多段着色时，应改用标题或标注框。只给词语本身着色，不要给整段着色，以免颜色延续到下一段。

## Fonts / 字体

- **New legal documents use Times New Roman.** When you draft a contract, brief, motion, legal memo or legal letter from scratch, with no template, set the body in Times New Roman. It is the professional default in legal practice, and other fonts read as informal. This doesn't apply when the document already has content (use its body font), when a template sets the font, or when the user names a font.
  新建法律文件使用 Times New Roman。在没有模板的情况下从零起草合同、诉状、动议、法律备忘录或法律函件时，正文一律使用 Times New Roman。它是法律行业的专业默认字体，其他字体显得不正式。文档已有内容（沿用其正文字体）、模板指定了字体、或用户点名了字体时，不适用此规则。

## Lists and numbering / 列表与编号

- **Never type list markers or section numbers** in a new document, or in a list or heading the document numbers automatically. In those, don't write "•", "-", "*", "1.", "3.2" or "(a)" as text; use the document's list and heading numbering. Typed markers look right until someone inserts an item or a section above them, and then the numbers are wrong.
  在新文档中，或在文档自动编号的列表和标题里，**绝不要手动键入列表标记或章节编号**。这些场合不要把 "•"、"-"、"*"、"1."、"3.2" 或 "(a)" 当作文字写出，而要使用文档的列表和标题编号功能。手动键入的标记起初看起来没问题，一旦有人在上方插入条目或章节，编号就全错了。
- **List items directly next to each other with the same list style join one list.** Numbering can also continue across a paragraph between two lists, so when a second numbered list should start again at 1, check that it does.
  列表样式相同且紧密相邻的列表项会合并为同一个列表。编号还可能跨越两个列表之间的段落继续递增，因此当第二个编号列表应当从 1 重新开始时，务必核实它确实如此。

## Tables / 表格

- **Match the existing tables.** When the document already has tables, give a new one the same borders, fills and header row. One table with a colored header among three plain ones looks like a mistake.
  与既有表格保持一致。文档中已有表格时，新表格应使用相同的边框、底纹和表头行。三张素表中间夹着一张彩色表头的表格，看起来就像失误。

## Page layout / 页面布局

- **Leave the page setup as it is unless the user asks**, and in a pageless document skip the rules below about pages and columns. "Applying the design rules" says how to recognize one.
  除非用户要求，否则保持页面设置不变；在无页面（pageless）文档中，跳过下文关于页和分栏的规则。"Applying the design rules" 一节说明了如何识别无页面文档。
- **Keep each heading on the same page as the paragraph after it.** A heading alone at the bottom of a page reads as broken even when the content is right. In a document you create, set "keep with next" on the heading styles, so it applies to every heading; in an existing document, do it when you find a heading separated from its paragraph.
  让每个标题与其后的段落保持在同一页。孤立在页面底部的标题即使内容无误，看起来也像排版出了问题。自己创建的文档，应在标题样式上设置 "keep with next"，使其作用于每个标题；修改既有文档时，发现标题与后段被分开就补上该设置。
- **To keep a section on one page, such as a financial statement, start it on a new page with one page break before its heading.** Check for an existing break first: a second one only adds a blank page. If the section still splits with a break in front of it, it is longer than a page; say so instead of adding another break.
  要让某一节（如财务报表）保持在一页内，应在其标题前加一个分页符，让该节从新的一页开始。先检查是否已有分页符：再加一个只会多出一页空白。加了分页符后该节仍然跨页，说明它超过一页；应如实说明，而不是再加分页符。
- **When a table is longer than a page, stop its rows from breaking across pages**, so that each row is whole on one page.
  表格超过一页时，禁止行跨页断开，使每一行都完整地留在单页内。
- **Page numbers are fields, not typed text.** "Page 1" typed into a footer reads 1 on every page.
  页码是域，不是手动键入的文字。在页脚里键入 "Page 1"，每一页都会显示 1。
- **Headers and footers can have variants.** If the document has a different first-page header, or different headers on odd and even pages, each one is separate: change every one that applies.
  页眉和页脚可以有多个变体。文档若设有不同的首页页眉，或奇偶页不同的页眉，每个变体相互独立：凡适用的都要逐一修改。
- **Columns belong to a section, not to a paragraph style.** To set part of a document in two columns, give that part its own section.
  分栏属于节（section），不属于段落样式。要把文档的一部分排成两栏，应让该部分自成一节。
- **Use real footnotes, not [1] markers typed into the text.**
  使用真正的脚注，不要在正文中键入 [1] 之类的标记。

## Check the result / 检查成果

Look at the rendered pages, not only the text. Assume there are formatting problems and look for them:

要查看渲染后的页面，不能只看文本。先假定存在格式问题，再逐项排查：

- **Fonts:** a paragraph in a different font from the rest, or a size change in the middle of a section.
  **字体：**某段与其余部分字体不同，或一节中间出现字号变化。
- **Styles:** added text that looks like plain body text where the paragraphs around it are headings or styled body text.
  **样式：**周边段落是标题或带样式的正文，新增文本看起来却是普通正文。
- **Numbering:** a list item that lost its marker, or (a) (b) that restarted as (1) (2).
  **编号：**列表项丢失标记，或 (a) (b) 变成从 (1) (2) 重新开始。
- **Tables:** columns that changed width, or cells that wrap where they didn't before.
  **表格：**列宽发生变化，或单元格在原本不换行处换行。
- **Formatting that runs on:** bold or italic that continues past where it should stop.
  **格式蔓延：**加粗或斜体越过应停止的位置继续延伸。
- **Spacing:** double blank lines, a paragraph pressed against the one above it, uneven indents.
  **间距：**连续两行空行、段落与上段紧贴、缩进不齐。
- **Suggestions:** insertions and deletions that render garbled or overlap the normal text.
  **修订建议：**插入与删除内容渲染乱码，或与正文重叠。
- **Page breaks:** a heading alone at the bottom of a page, a blank page, a section that used to fit on one page and now splits.
  **分页：**标题孤立在页面底部、出现空白页、原本一页容纳的章节现在跨页。

After you fix one problem, check the paragraphs and pages around it. A fix to one paragraph often changes the next one, and a change in length moves the page breaks.

修复一个问题后，检查其周边的段落和页面。对某段的修改常常影响下一段，篇幅变化也会移动分页位置。

## Writing / 行文

Text you write in a document should read as though a person wrote it. When readers think something was written by AI, they judge it as sloppy and stop trusting it, whatever the content. They make that judgment from a set of common indicators, listed below, so take extra care to keep them out of your writing. These rules are for text you write, and a style the user or their style guide asks for takes priority. Do not rewrite the user's existing text to follow them unless the user asks you to.

文档中写出的文字应读起来像出自真人之手。读者一旦认定某段文字是 AI 写的，无论内容如何都会视之为草率并不再信任它。他们依据的是一组常见特征，列举如下，因此要格外注意不让这些特征出现在你的文字里。这些规则约束的是你写出的文本；用户或其风格指南指定的风格优先。除非用户要求，不要为套用这些规则而改写用户已有的文字。

【评论】该节把读者的 AI 文本识别特征显式列为禁用项，是对模型输出风格质量的工程化约束；同时明确用户或风格指南的优先级更高，避免规则与用户要求冲突。

- Say what is true without first denying something else. "Revenue grew 12%, three times the US rate," not "This isn't a growth story, it's a market-share story." Do not open a paragraph or bullet with "Here's the thing" or "The real story is".
  直接陈述事实，不要先否定别的说法。应写 "Revenue grew 12%, three times the US rate"，而不是 "This isn't a growth story, it's a market-share story"。段落或要点不要以 "Here's the thing" 或 "The real story is" 开头。
- Match the number of bullets, examples, and adjectives to the content, not to a default of three. Two drivers get two bullets; five get five. A list of exactly three ("fast, reliable, and scalable") usually means the third item was added for rhythm, not because there were three things to say, and readers read it as filler.
  要点、示例和形容词的数量应与内容匹配，而不是默认凑三个。有两个驱动因素就列两条；有五条就列五条。恰好三条的清单（"fast, reliable, and scalable"）通常意味着第三条只是为了节奏感而凑数，并非真有第三件事可说，读者会把它读作填充内容。
- Do not use a metaphor where a literal word will do. If a plain description exists ("the same construction," "the same pattern," "slowed," "fell"), use it. Metaphors are for when the literal version would be longer or less precise, which is rare in analytical writing. Test: if the metaphor can be replaced by a plain word without losing meaning, replace it. A metaphor makes the reader translate it back into the plain claim and carries meaning you did not choose. The ones that appear most are "north star", "move the needle", "double-click" (meaning look closer), "unpack", "journey", and "landscape" (meaning a market). Common ones in business writing: "moat", "headwind", "drag" (meaning a cost on results), "safety net", "clears the bar", and "land" meaning finish or total ("lands $4.4k under budget"). Examples:
  字面词够用时不要用比喻。如果存在直白的说法（"the same construction"、"the same pattern"、"slowed"、"fell"），就用直白说法。比喻只在字面表述会更冗长或更不精确时使用，这在分析性写作中很少见。检验方法：把比喻换成平实词语而不损失含义，就换掉它。比喻迫使读者把它翻译回平实的论断，还夹带了你未曾选择的意义。最常出现的比喻有 "north star"、"move the needle"、"double-click"（意为细看）、"unpack"、"journey" 和 "landscape"（意为市场）。商业写作中的常见比喻还有："moat"、"headwind"、"drag"（意指对业绩的成本拖累）、"safety net"、"clears the bar"，以及表示完成或合计的 "land"（如 "lands $4.4k under budget"）。示例：
  - Bad: "The ones it has are the same species." Good: "The ones it has follow the same pattern."
    差："The ones it has are the same species." 好："The ones it has follow the same pattern."
  - Bad: "A coordinated digestion pause would be visible immediately." Good: "If several large customers cut capex in the same quarter, it would show up in the next guide."
    差："A coordinated digestion pause would be visible immediately." 好："If several large customers cut capex in the same quarter, it would show up in the next guide."
  - Bad: "Architecture transitions compressed margin on the way in and expanded it on the way out." Good: "Gross margin fell during the Hopper-to-Blackwell ramp and recovered once Blackwell shipped at volume."
    差："Architecture transitions compressed margin on the way in and expanded it on the way out." 好："Gross margin fell during the Hopper-to-Blackwell ramp and recovered once Blackwell shipped at volume."
  - Bad: "The lever that unlocks growth." Good: "The pricing change is what makes the target reachable."
    差："The lever that unlocks growth." 好："The pricing change is what makes the target reachable."
- Cut words that claim importance without giving evidence: "genuinely", "truly", "actually", "clearly", "significantly", "robust", "leverage", "delve", "actionable insights", "learnings". Where one of them stood in for a fact, put the fact there ("margins fell 4 points", not "margins fell significantly"); otherwise delete it. When "leverage" or "significant" carries its financial or statistical meaning ("net leverage", "statistically significant"), it is a literal term; keep it.
  删掉那些宣称重要却不给证据的词："genuinely"、"truly"、"actually"、"clearly"、"significantly"、"robust"、"leverage"、"delve"、"actionable insights"、"learnings"。如果某个词原本替代了一个事实，就写出事实（"margins fell 4 points"，而不是 "margins fell significantly"）；否则直接删除。当 "leverage" 或 "significant" 取其金融或统计含义（"net leverage"、"statistically significant"）时，属于字面术语，保留。
- Use full stops and commas, and a colon before a list. No emoji in the document.
  使用句号和逗号，列表前用冒号。文档中不要用表情符号。
- Use em dashes sparingly: at most one in a paragraph, and none in a heading, a title, or between a bold label and the text after it. Several em dashes in one paragraph is one of the first things readers use to spot machine writing. In place of one, use a comma, a colon, parentheses, or a new sentence, not an en dash or a spaced hyphen.
  破折号（em dash）要节制使用：每段至多一个，标题、小标题以及粗体标签与其后文字之间不用。一段里出现多个破折号，是读者识别机器写作的首要线索之一。需要替代时，用逗号、冒号、括号或另起一句，不要用半字线（en dash）或带空格的连字符。
- Headings in the document are about the content, not about your process: "Europe missed plan by $1.9M" or "Pricing", not "What I changed" or "The hard part". Standard labels such as "Executive Summary" or "Next steps" are fine. When the document already has headings, phrase new ones the same way, for example as short labels or as full sentences.
  文档中的标题应描述内容，而不是你的操作过程："Europe missed plan by $1.9M" 或 "Pricing"，而不是 "What I changed" 或 "The hard part"。"Executive Summary"、"Next steps" 这类标准标签没有问题。文档已有标题时，新标题与其保持同样的措辞方式，例如都作短标签或都作完整句。
- Keep the conversation out of the document. When the user asks for a shorter or more formal version, make it shorter or more formal; do not title it "Executive Summary (Condensed)" or open it with "Updated per your feedback". The document's readers did not see the request, so those lines mean nothing to them. Say what changed in your reply, not in the document. This covers the document's text, not replies in comment threads.
  不要把对话痕迹带进文档。用户要求更简短或更正式的版本时，就把它改得更简短或更正式；不要起 "Executive Summary (Condensed)" 这样的标题，也不要以 "Updated per your feedback" 开头。文档的读者看不到请求过程，这些字句对他们毫无意义。改动说明写在你的回复里，不要写进文档。此规则约束文档正文，不约束评论串里的回复。

## Editing an existing document / 编辑既有文档

These rules apply whenever you change a document that already exists. Everything you add also follows the rules above.

只要你修改的是已存在的文档，这些规则就适用。你新增的一切内容同样遵循上文所有规则。

### Change only what was asked / 只改要求之处

- **Match the scope of the edit to the request.** "Fill in this section" means add text. It doesn't mean also changing the alignment, adding underlining, reformatting tables or restyling nearby paragraphs. If a check shows a formatting change you didn't intend, undo it.
  修改范围与请求相符。"填写这一节"意味着添加文字，不包括顺带改对齐、加下划线、重排表格或改变周边段落的样式。检查时发现并非本意的格式改动，就撤销它。
- **Change the smallest range that covers the change.** Replace the words that change, not the paragraph or section around them. Never delete and rebuild a paragraph, a section or the document to change part of it: that loses comments, bookmarks, images, charts and other embedded objects.
  修改覆盖改动的最小范围。只替换变化的词语，而不是其周边的段落或节。绝不要为了改动一部分就删除并重建某个段落、节乃至整篇文档：那会丢失评论、书签、图片、图表及其他嵌入对象。
- **Keep the original wording wherever you aren't deliberately changing it.** Rewording the user didn't ask for is one more change they have to find and review. In a contract it is a substantive change: "aggregate" becoming "total", or "shall not" becoming "will not", changes what the text says.
  非有意修改之处保持原措辞。用户没有要求的改写，是他们必须另行查找和审阅的改动。在合同里这属于实质变更："aggregate" 改成 "total"，或 "shall not" 改成 "will not"，都会改变文本含义。
- **Reformat by role.** "Make the body text 11pt" means the body text, not the headings; "indent the section headers" means the headings. Pick out the paragraphs by their style before you change them.
  按角色重排格式。"正文改为 11pt"指的是正文，不包括标题；"缩进节标题"指的是标题。动手之前先按样式把目标段落挑选出来。
- **Take target values from the document, not from a guess.** If one table's header row is wrong and three others are right, copy the values from a correct one. When the user points at a reference ("make it look like section 3"), read that section's exact style, font, size, color, alignment and line spacing, and apply those. A guessed 18pt in a document built at 11pt is worse than the original problem.
  目标取值来自文档本身，不要凭猜测。某张表格的表头行不对而另外三张都对时，从正确的表格复制取值。用户给出参照物时（"改得像第 3 节那样"），读取该节确切的样式、字体、字号、颜色、对齐和行距，然后照搬。在通篇 11pt 的文档里猜一个 18pt，比原来的问题更糟。
- **Keep similar tables consistent, but only when the user asked you to reformat tables.** When the user asks you to reformat one of several similar tables, make the same change to the others or tell the user you changed only the one.
  相似表格保持一致，但仅当用户要求重排表格时。用户要求重排几张相似表格中的一张时，要么对其余表格做同样修改，要么告知用户你只改了那一张。
- **A table that changed as a side effect of another edit is a mistake to undo**, not a change to copy to the other tables.
  因其他编辑而连带变化的表格属于应撤销的错误，而不是可以照搬到其他表格的样板。
- **Never replace a bare number across the whole document.** Closing a gap in section numbers by replacing every "7" with "6" also changes a "7.1x" interest coverage, a "7.5%" coupon and "FY2027". Renumber the headings one at a time.
  绝不在全文档范围内替换裸数字。为修正章节编号断档而把每个 "7" 替换成 "6"，也会改掉 "7.1x" 的利息保障倍数、"7.5%" 的票息和 "FY2027"。要重编号，就逐个标题处理。

### Match what is there / 与既有内容保持一致

- **Text you add uses the document's body font and style**, not the editor's default. After you insert text, check its font and size against the paragraph before it.
  新增文本使用文档的正文字体和样式，而不是编辑器默认值。插入文本后，对照前一段检查其字体和字号。
- **Use the document's spelling variety**, for example British or American English. Keep new text in the variety the document already uses, never mix varieties in one document, and don't change existing text to another variety unless the user asks. If the document has no text yet, follow the variety of the user's messages.
  使用文档既有的拼写变体，例如英式或美式英语。新增文本沿用文档已有的变体，绝不在同一文档中混用两种变体；除非用户要求，不要把既有文本改成另一变体。文档尚无文本时，跟随用户消息的变体。
- **Set explicit colors from the document.** If you must set a text color, copy it from an existing body paragraph instead of assuming black: many document styles use a dark gray or a theme color.
  显式颜色取自文档。确需设置文字颜色时，从既有正文段落复制，而不要想当然用黑色：许多文档样式使用深灰或主题色。
- **New numbered headings get their number from the document.** When the document numbers its headings automatically, write the heading text without a number and give the heading the same style or list level as the headings next to it, so it takes the next number. If the document numbers its headings by hand, follow that convention instead.
  新增的编号标题从文档获取编号。文档自动编号标题时，写标题文字不带编号，并让新标题使用与相邻标题相同的样式或列表级别，从而顺次取得下一个编号。文档手动编号标题时，遵循该惯例。
- **New list items continue the list.** Add an item as part of the list it belongs to, and check that it took the next marker: (b) after (a), not a second (a) or a skipped letter.
  新增列表项延续原列表。把新条目作为所属列表的一部分加入，并核实它取得了下一个标记：(a) 之后是 (b)，而不是重复的 (a) 或跳号。
- **Making headings consistent.** When the user asks for all the headings to match, first give any hand-formatted heading (bold, larger body text) the right heading style. Then set each level to the font of its own heading style, one level at a time; applying the first level's font to every level removes the difference between levels. Applying a heading style doesn't remove a size or color set directly on the text, so clear those as well.
  统一标题样式。用户要求所有标题一致时，先把任何手动格式化的标题（加粗、放大的正文）改为正确的标题样式；再逐一级别把每层设置为该层标题样式自带的字体。把第一级的字体套到所有层级会抹平层级差异。应用标题样式并不会移除直接设置在文字上的字号或颜色，这些也要一并清除。

### Things inside the text / 文本内部的元素

Some elements sit inside a paragraph's text without looking like separate objects: footnote marks, chips (such as a person, date or file chip), bookmark boundaries, comment anchors, and inline images and charts. Replacing or deleting a range that contains one removes it: the footnote is gone, the chip is deleted, the bookmark moves, the chart disappears.

有些元素嵌在段落文本内部，看起来不像独立对象：脚注标记、智能片段（chip，如联系人、日期或文件 chip）、书签边界、评论锚点，以及行内图片和图表。替换或删除包含它们的范围会把它们一并删掉：脚注消失、chip 被删除、书签移位、图表不见。

- **Before you edit a sentence, check what is inside it**, and change the text on either side of such an element rather than through it.
  编辑句子之前先检查句内有什么，围绕这类元素两侧修改文字，不要穿过元素本身修改。
- **A paragraph with no visible text can still hold an image or a chart**, and a floating shape is anchored to a paragraph. Deleting the paragraph deletes them, so check before you delete an empty-looking paragraph.
  没有可见文字的段落仍可能承载图片或图表，浮动形状也锚定在某个段落上。删段落就会删掉它们，所以删除看似空白的段落之前先检查。
- **In a section set in columns, edit the text inside the section.** Don't remove the section break that holds the column setting.
  在分栏的节内编辑文字时，就在节内编辑。不要删除承载分栏设置的分节符。

### Templates / 模板

- **Fill the placeholders and keep the structure.** Never delete an image placeholder or a signature line while filling a template; removing one breaks the template for the next person who uses it.
  填充占位符，保留结构。填模板时绝不要删除图片占位符或签名行；删掉一个，模板对下一个使用者就坏了。
- **Find every placeholder before you fill any.** Templates often use typed markers such as [CLIENT NAME]. If a template has none, find each section by its heading and write after it.
  填任何一个占位符之前先找齐全部占位符。模板常使用 [CLIENT NAME] 之类手动键入的标记。模板没有这类标记时，按标题逐节定位，在其后书写。

### Comments / 评论

- **Reply in the thread.** When you respond to a comment, reply to it; don't start a new comment. If your tool can't reply to comments, give the user the reply text to paste into each thread.
  在原讨论串中回复。回应某条评论时，直接回复该评论，不要另开新评论。工具无法回复评论时，把回复文字交给用户，由其粘贴到各讨论串。
- **One thread per topic.** Before you add a comment, look for an existing thread on the same text or topic, including one you left earlier, and reply there. Several comments stacked on one paragraph make it unclear which note is current.
  一个话题一个讨论串。添加评论之前，先查找针对同一文本或话题的既有讨论串（包括你自己早先留下的），在原串中回复。多条评论堆在同一段落上，会让人分不清哪条是当前有效的意见。
- **Leave comments where they are.** Don't delete or resolve a comment unless the user asks. Reply once per comment; don't add a second reply that says nothing new.
  评论留在原处。除非用户要求，不要删除或标记解决任何评论。每条评论只回复一次；不要追加没有新内容的第二条回复。
- **When you edit commented text, change words inside the commented range, not all of it.** A comment is attached to its text, and replacing all of that text deletes the comment and its replies. Make the edit first and reply after it, so that no reply describes an edit that didn't happen.
  编辑被评论的文字时，只改该评论范围内的词语，不要整体替换。评论附着于其文本，整体替换文本会连同评论及其回复一并删除。先完成修改再回复，确保没有哪条回复描述了一场并未发生的修改。
- **"Address the comments" means every comment.** Handle each one: make the edit it asks for, if any, reply with a one-line note, and leave the thread open unless the user asks you to resolve it.
  "处理评论"意味着每一条。逐条处理：有修改要求就执行修改，回复一行说明；除非用户要求解决，否则保持讨论串开放。

### Suggestions / 修订建议

- **Use real suggestions**, never strikethrough and colored text made to look like them. The reviewer needs to accept or reject each change.
  使用真正的修订建议，绝不要用删除线和彩色文字模拟。审阅者需要对每处改动逐一接受或拒绝。
- **Don't accept or reject changes, or delete comments, to clean up.** In a review, the suggestions and comment threads are the work product: accepting them erases the record of what changed, and deleting comments erases the reviewers' notes. "Clean up the document" means fix the formatting. Accept, reject or delete only the ones the user names.
  不要为了"清理"而接受或拒绝修订，或删除评论。在审阅场景中，修订建议和评论串就是工作成果：接受它们会抹掉改动记录，删除评论会抹掉审阅者的批注。"清理文档"指的是修整格式。只接受、拒绝或删除用户点名的那些。
- **Mark only the words that change.** A paragraph replaced as a whole shows as the whole paragraph deleted and inserted again, and the reviewer can't see what changed. Replace only the words that change, and make several small changes in one paragraph as separate edits. To change a cap from twelve months to six, the change is "twelve (12)" to "six (6)", not the sentence it sits in.
  只标记真正变化的词语。整段替换会把整段显示为先删后插，审阅者看不出改了什么。只替换变化的词语，同一段里的多处小改动拆成多次独立编辑。把上限从十二个月改成六个月，修订的是 "twelve (12)" 到 "six (6)"，而不是它所在的整个句子。

### After structural changes / 结构性修改之后

- **Keep the table of contents current.** After you add, remove or rename a heading in a document that has a table of contents, update it in the same step: a table of contents that lists "Section 4: Risk Factors" when section 4 is now "Liquidity" is a visible defect. Update an existing one; don't add one the document doesn't have unless the user asks for it. If your tool can't update it, tell the user it needs updating.
  保持目录为最新。在带目录的文档中新增、删除或重命名标题后，同步更新目录：第 4 节已改为 "Liquidity" 而目录仍列着 "Section 4: Risk Factors"，是肉眼可见的缺陷。只更新既有目录；除非用户要求，不要给没有目录的文档添加目录。工具无法更新时，告知用户目录需要更新。
- **Remove everything in a section you delete.** Deleting a section's paragraphs can leave its table or images behind. Count the tables before and after, and check the count dropped by the number you removed.
  删除某一节时清空其全部内容。只删段落可能把该节的表格或图片遗留在文档里。删除前后各统计一次表格数量，核实减少量与删除数一致。
- **Before removing a duplicate section, read both copies.** Keep the one whose formatting matches the rest of the document, and tell the user which one you kept and why.
  删除重复的章节之前，先通读两份副本。保留格式与文档其余部分一致的那份，并告知用户你保留了哪份以及为什么。
- **Clear what deletions leave behind:** empty tables, two or more blank paragraphs in a row (collapse them to one), list items cut off from their list, and blank paragraphs at the end that push an empty last page.
  清理删除留下的残余：空表格、连续两个及以上的空段落（压缩为一个）、脱离所属列表的列表项，以及把空白末页顶出来的尾部空段落。

### Check your edits / 检查你的修改

After each edit, read back what you changed: the text, its style, its font and any list marker. After an edit that can move the layout, such as added or removed content, a table or formatting change, or a fix to something that looked wrong, also check the rendered pages, including the pages around the change, against the list under Check the result above.

每次编辑之后回读改动：文字、样式、字体以及列表标记。编辑可能影响版式时——例如增删内容、表格或格式变更，或修复了看起来不对的地方——还要按上文"检查成果"清单核查渲染后的页面，包括改动周边的页面。

# Docs and Drive tools / Docs 与 Drive 工具

## Applying the design rules / 应用设计规则

Some of the design rules above act on a named style (Normal text, Heading 1 to 6) or on a page-number field, and the Docs API can apply a named style but not change one, and can't insert a page number. Others depend on the page setup. Do this instead:

上文部分设计规则作用于命名样式（Normal text、Heading 1 至 6）或页码域，而 Docs API 只能应用命名样式、不能修改样式，也无法插入页码。另一些规则依赖页面设置。请改用以下做法：

- **Body font, such as Times New Roman for a new legal draft:** set it once over the whole body with `updateTextStyle` (`weightedFontFamily`, `fields: "weightedFontFamily"`) after the text is in, and give text you add later the same font.
  **正文字体，例如新法律文稿的 Times New Roman：**在文本就位后，用 `updateTextStyle`（`weightedFontFamily`，`fields: "weightedFontFamily"`）对整个正文一次性设置，之后再为新增文本指定同一字体。
- **Keep each heading with the next paragraph:** set `keepWithNext: true` on each heading paragraph with `updateParagraphStyle` (`fields: "keepWithNext"`).
  **让每个标题与后段同页：**用 `updateParagraphStyle`（`fields: "keepWithNext"`）在每个标题段落上设置 `keepWithNext: true`。
- **Make every heading at one level smaller or larger:** tell the user they can resize one heading at that level and then choose Format > Paragraph styles > Heading N > Update 'Heading N' to match, which changes the style itself. If they want you to do it instead, set the same size on every heading at that level in one batch.
  **统一调整某一层级全部标题的字号：**告知用户可以手动改一个该层级标题的字号，然后选择 Format > Paragraph styles > Heading N > Update 'Heading N' 使样式匹配，这会改动样式本身。若用户希望由你代劳，就对该层级每个标题批量设置同一字号。
- **Page numbers:** tell the user to add them with Insert > Page numbers.
  **页码：**告知用户通过 Insert > Page numbers 添加。
- **Whether a tab is pageless:** in the `read_doc` result, check the tab you are editing. It is pageless when `documentFormat.documentMode` is `PAGELESS` in its `documentTab.documentStyle`, or in the top-level `documentStyle` when there is no `tabs` key. In a pageless tab, don't add page numbers, headers, footers or columns; tell the user to switch it to pages first, with Format > Switch to Pages format.
  **判断标签页是否为无页面模式：**在 `read_doc` 结果中检查正在编辑的标签页。当其 `documentTab.documentStyle` 中的 `documentFormat.documentMode` 为 `PAGELESS`，或在无 `tabs` 键时顶层 `documentStyle` 中如此，即为无页面模式。无页面标签页中不要添加页码、页眉、页脚或分栏；告知用户先通过 Format > Switch to Pages format 切换为页面模式。
- **Page setup, when the user asks for a change:** send `updateDocumentStyle` with `tabId` on the request itself (it has no location or range), the new values in `documentStyle`, and only those field names in `fields`, without the `documentStyle.` prefix: `{"updateDocumentStyle": {"tabId": "t.0", "documentStyle": {"marginTop": {"magnitude": 54, "unit": "PT"}}, "fields": "marginTop"}}`. A field named in `fields` but missing from `documentStyle` is cleared, so never use `*`, which names every field.
  **页面设置，当用户要求修改时：**发送 `updateDocumentStyle`，把 `tabId` 放在请求本身上（它没有位置或范围参数），新值放在 `documentStyle`，`fields` 只列这些字段名且不带 `documentStyle.` 前缀：`{"updateDocumentStyle": {"tabId": "t.0", "documentStyle": {"marginTop": {"magnitude": 54, "unit": "PT"}}, "fields": "marginTop"}}`。在 `fields` 中列出但 `documentStyle` 中缺失的字段会被清除，因此绝不要用 `*`，它代表全部字段。

These are based on the Docs API reference and have not been tested through the connector.

以上做法依据 Docs API 参考文档整理，尚未通过连接器实测。

【评论】该节披露了连接器能力与 Docs API 能力之间的落差：样式只能套用不能定义、页码无法程序化插入，因此部分设计规则只能转达给用户手动完成；文档也如实注明这些建议未经实测。

## Create / 创建

Call Drive `create_file` with `contentMimeType: "text/html"` and the document as `textContent`. Drive converts `<h1>`, `<h2>`, `<p>`, `<ul>`, `<ol>`, `<b>`, `<i>`, and `<a>` into native Docs formatting. This is much faster and less error-prone than building a new doc with Docs edit requests. Use the Docs connector for changes after that. For a Word file the user will download rather than edit in Google Docs, use the `docx` skill instead.

调用 Drive 的 `create_file`，使用 `contentMimeType: "text/html"`，文档内容作为 `textContent` 传入。Drive 会把 `<h1>`、`<h2>`、`<p>`、`<ul>`、`<ol>`、`<b>`、`<i>` 和 `<a>` 转换为 Docs 原生格式。这比用 Docs 编辑请求从零搭建新文档快得多，也更不容易出错。此后的修改再使用 Docs 连接器。若用户要下载 Word 文件而非在 Google Docs 中编辑，改用 `docx` 技能。

## How a doc is addressed / 文档如何寻址

Every Docs edit points at a position, so the model of positions matters more than anything else in this section.

每次 Docs 编辑都指向一个位置，因此位置模型比本节其他任何内容都重要。

【评论】Google Docs 以 UTF-16 码元为寻址单位，且每次写入都会使后续索引整体位移，这是 API 编辑错位的常见根源；本节因此要求请求按索引逆序排列、写入携带修订号守卫，并在每次写入后重新读取。

- **Indexes are UTF-16 code units** counted from the start of a tab's body, which starts at index 1. A character outside the Basic Multilingual Plane, such as 🙂, counts as 2, and an emoji built from several characters counts as more: 👍🏽 (👍 plus a skin tone) is 4. Docs refuses an insert inside an emoji with "The insertion index cannot be within a grapheme cluster", and a delete through the middle of 🙂 with "Invalid deletion range". Every paragraph ends with a newline that occupies one index.
  **索引是 UTF-16 码元**，从标签页正文起点起计数，起点为索引 1。基本多文种平面之外的字符（如 🙂）计为 2，由多个字符组成的表情符号计得更多：👍🏽（👍 加肤色修饰）为 4。在表情符号内部插入会被拒绝，报 "The insertion index cannot be within a grapheme cluster"；删除范围穿过 🙂 中部则报 "Invalid deletion range"。每个段落以一个换行结尾，该换行占一个索引。
- **Every insert or delete shifts everything after it.** Requests in one `update_doc` call run in order, so an index computed from the read is only valid for the first request that touches that region. Order the requests from the highest index to the lowest, and no request moves a position a later request relies on.
  每次插入或删除都会移动其后的所有内容。同一次 `update_doc` 调用中的请求按顺序执行，因此依据读取结果算出的索引只对触碰该区域的第一个请求有效。把请求按索引从高到低排列，任何请求都不会移动后续请求所依赖的位置。
- **Indexes go stale after any write.** Never reuse indexes from a read taken before your last write. Read again, then compute.
  任何写入之后索引即告失效。绝不要复用上一次写入之前读取到的索引。先重新读取，再计算。
- **Guard every index-based write with the revision.** Pass the `revisionId` from your read as `writeControl.requiredRevisionId`. If someone edited the doc in between, the whole batch is rejected with a 400 instead of landing in the wrong place. On that error, read again and recompute. Do not retry without the guard.
  每次基于索引的写入都要用修订号防护。把你读取时得到的 `revisionId` 作为 `writeControl.requiredRevisionId` 传入。若期间有人编辑过文档，整个批次会被 400 拒绝，而不是落到错误位置。遇到该错误就重新读取并重算。绝不要不带防护直接重试。
- **Tabs are separate index spaces.** A doc can have several tabs, and child tabs under them. Every `location` and `range` takes a `tabId`; without it, the request applies to the first tab, which is a silent failure when the user meant another one. A pasted URL like `.../edit?tab=t.abc123` names the tab: `t.abc123` is the `tabId`. `replaceAllText` is the opposite: without `tabsCriteria` it changes every tab, child tabs included, and naming a parent tab in `tabsCriteria` does not include its child tabs.
  各标签页是相互独立的索引空间。一个文档可以有多个标签页，标签页下还可以有子标签页。每个 `location` 和 `range` 都要带 `tabId`；不带时请求作用于第一个标签页，用户本意是另一个标签页时这就是一次静默失败。形如 `.../edit?tab=t.abc123` 的粘贴 URL 中标明了标签页：`t.abc123` 就是 `tabId`。`replaceAllText` 正相反：不带 `tabsCriteria` 时它会改动所有标签页（含子标签页），而在 `tabsCriteria` 中写了父标签页并不涵盖其子标签页。
- **Pending suggestions are in the index space.** `read_doc` returns suggested insertions and deletions inline, as text runs that carry `suggestedInsertionIds` or `suggestedDeletionIds`, and they occupy real indexes. Don't treat a suggested deletion as live text, and don't insert inside one.
  待处理的修订建议占据索引空间。`read_doc` 把建议的插入和删除内联返回，表现为携带 `suggestedInsertionIds` 或 `suggestedDeletionIds` 的文本段，它们占用真实的索引。不要把建议删除的内容当作现存文本，也不要在其内部插入。
- **The body's final newline can't be deleted.** To append, insert at the last element's `endIndex - 1`.
  正文末尾的换行无法删除。要在末尾追加，就在最后一个元素的 `endIndex - 1` 处插入。

## Read / 读取

There are two reads, and they serve different purposes.

读取方式有两种，用途各不相同。

- **Drive `read_file_content`** returns the doc as Markdown: headings, tables, and bold, typically a few KB. Use it first to understand what the doc says and to find the text you'll anchor on. It is not the index space: it escapes Markdown characters, and it glues pending suggestions to the text around them. Never compute an index from it.
  **Drive 的 `read_file_content`** 以 Markdown 返回文档：标题、表格和加粗，通常只有几 KB。先用它了解文档内容、找到将要锚定的文本。它不是索引空间：它会转义 Markdown 字符，还会把待处理的修订建议与周边文字粘在一起。绝不要据它计算索引。
- **Docs `read_doc`** returns the full `documents.get` JSON: every element with its `startIndex` and `endIndex`, every style, the `revisionId`, and the tabs. Use it when an edit needs indexes. It is large: a two-page doc with one table can exceed 100 KB. Pass `commentsIncluded: true` only when you need comments. The connector's read has come back without comments even with it set, showing `COMMENTS_VIEW_MODE_OMITTED`; if so, read them with Drive `read_file_content` and `includeComments: true`, or ask the user to paste the ones they want handled.
  **Docs 的 `read_doc`** 返回完整的 `documents.get` JSON：每个元素及其 `startIndex` 和 `endIndex`、每种样式、`revisionId` 以及各标签页。编辑需要索引时使用它。它体积很大：一份带一个表格的两页文档可能超过 100 KB。仅在需要评论时传 `commentsIncluded: true`。连接器的读取即使设置了该参数也可能不带评论返回，并显示 `COMMENTS_VIEW_MODE_OMITTED`；此时可改用 Drive 的 `read_file_content` 加 `includeComments: true` 读取评论，或请用户粘贴需要处理的评论。

Where the content lives in the `read_doc` JSON:

内容在 `read_doc` JSON 中的位置：

```
revisionId
tabs[].tabProperties.tabId
tabs[].documentTab.body.content[]          body elements, in order
  .paragraph.elements[].textRun.content    text, with startIndex / endIndex on the element
  .paragraph.paragraphStyle.namedStyleType HEADING_1, NORMAL_TEXT, ...
  .table.tableRows[].tableCells[].content[]  each cell holds its own paragraphs
tabs[].childTabs[]                          nested tabs, same shape
```

If the doc has no `tabs` key, the body is at the top level: `body.content[]`.

文档没有 `tabs` 键时，正文位于顶层：`body.content[]`。

### Use the helper script for positions / 用辅助脚本处理位置

`scripts/docs_index.py` (in this skill's folder) reads the `read_doc` JSON from a file and prints only what an edit needs. Where you can run code, use it rather than walking the JSON by eye. That walk is where index errors come from.

`scripts/docs_index.py`（位于本技能文件夹内）从文件读取 `read_doc` JSON，只打印编辑所需的信息。能运行代码的场合就用它，不要用肉眼遍历 JSON。索引错误正是出在肉眼遍历上。

- A large `read_doc` result may be saved to a file, and the result then gives the path. Run the script on that path.
  较大的 `read_doc` 结果可能被保存为文件，结果中会给出路径。对该路径运行脚本。
- If the JSON came back in context and the doc is small, read the positions from it directly.
  若 JSON 直接返回在上下文中且文档较小，直接从中读取位置。
- If the read was cut off and not saved anywhere, don't guess positions. Use `replaceAllText`, which needs none, when the find text matches only the text you mean to change (add neighboring words until it does), or tell the user the doc is too large to edit by position here.
  若读取被截断且未保存到任何地方，不要猜测位置。在查找文本恰好只匹配目标文字时，使用无需位置的 `replaceAllText`（不断加入相邻词语直到唯一匹配），或告知用户文档过大、无法在此按位置编辑。

Commands:

命令：

```
python <skill>/scripts/docs_index.py outline DOC.json
    one line per element: index range, style, table cell position, text.
    Pending suggestions show as [+inserted] / [-deleted].
    Ends with the body end index for appends.
python <skill>/scripts/docs_index.py find DOC.json "exact text"
    index range of every match within one paragraph, whether it is bold, and whether it sits
    in a pending suggestion. A match can't cross a non-text element, such as a chip, image or footnote mark.
python <skill>/scripts/docs_index.py fill-table DOC.json --table N --data rows.json [--bold-header]
    the full update_doc arguments (requests + writeControl) to fill empty table N, the
    TABLE #N that outline prints (every tab and nested table is counted), from a JSON list of rows.
python <skill>/scripts/docs_index.py new-table --at I --data rows.json --revision REV [--tab T] [--bold-header]
    the full update_doc arguments to insert a table at I, the endIndex - 1 of the paragraph
    it goes after, and fill it in the same call. Needs no DOC.json.
```

outline and find print the `revisionId` and the tab ID; fill-table puts the `revisionId` in its `writeControl`.

outline 和 find 会打印 `revisionId` 和标签页 ID；fill-table 把 `revisionId` 写入其 `writeControl`。

## Edit recipes / 编辑配方

Each recipe is one `update_doc` call. Add `tabId` to every `location` and `range` when the doc has more than one tab, and add `writeControl: {"requiredRevisionId": ...}` to any call that uses indexes. Put real newlines in inserted text, not an escaped `\n`.

每个配方对应一次 `update_doc` 调用。文档有多个标签页时，给每个 `location` 和 `range` 加上 `tabId`；任何使用索引的调用都加上 `writeControl: {"requiredRevisionId": ...}`。插入文本中使用真实换行，而不是转义的 `\n`。

**Change a word or phrase.** `replaceAllText` needs no read and no indexes:

**替换单词或短语。**`replaceAllText` 无需读取、无需索引：

```json
{"replaceAllText": {"containsText": {"text": "Q3 launch", "matchCase": true},
  "replaceText": "Q4 launch", "tabsCriteria": {"tabIds": ["t.0"]}}}
```

The reply reports `occurrencesChanged`. Zero means the text didn't match exactly: check it against `read_file_content`, but remember that file escapes characters like `&` and `*`. The replacement takes the style of the first character it replaces, so replacing a span that starts in bold makes the whole replacement bold. If that matters, start the find on an unstyled character, or check the result with `find` afterward. It changes every match in the tabs it covers (every tab unless `tabsCriteria` names some), so use it when every match should change, such as a date or a name the user wants changed throughout. When only one occurrence should change, or the find text is short enough to occur inside unrelated text (a bare number such as "7", a common word), delete and insert at the range `find` gives for that occurrence. Never put a paragraph's trailing newline in the find text: the replaced paragraph can take the next paragraph's style (a body paragraph becomes a heading), or the next paragraph can lose its heading style.

响应会报告 `occurrencesChanged`。为零说明文本未精确匹配：对照 `read_file_content` 检查，但注意该文件会转义 `&`、`*` 之类字符。替换文本继承被替换首字符的样式，因此替换以加粗开头的片段会让整个替换变粗。这有影响时，把查找起点放在无样式的字符上，或事后用 `find` 核查结果。它会改动其覆盖标签页中的每一处匹配（除非 `tabsCriteria` 另有指定，否则覆盖所有标签页），因此只应在所有匹配都应修改时使用，例如用户要求全文修改的日期或名称。当只应改一处，或查找文本短到可能出现在无关文字中（如裸数字 "7" 或常用词）时，改为在 `find` 给出的该处范围上先删后插。绝不要把段落的结尾换行放进查找文本：被替换段落可能继承下一段的样式（正文段落变成标题），或下一段丢失标题样式。

**Rewrite a paragraph.** Take its `startIndex` and `endIndex` from `outline`. Delete `[start, end - 1]`, which keeps the paragraph's newline and style, then insert at `start`:

**重写一个段落。**从 `outline` 取其 `startIndex` 和 `endIndex`。删除 `[start, end - 1]` 以保留该段落的换行和样式，然后在 `start` 处插入：

```json
[{"deleteContentRange": {"range": {"startIndex": 120, "endIndex": 184}}},
 {"insertText": {"location": {"index": 120}, "text": "New paragraph text."}}]
```

For several paragraphs, do the highest one first. When only some words in the paragraph change, delete and insert only those words at the range `find` gives, not the whole paragraph. A footnote mark, chip or image elsewhere in the paragraph then stays, and in suggestion mode the suggestion shows only the words that changed. To delete a whole paragraph, delete `[start, end]`. In three cases that range is refused, with "Invalid deletion range" or "The range cannot include the newline character at the end of the segment"; use these ranges instead. For the doc's last paragraph, delete from the previous paragraph's `endIndex - 1` to the last paragraph's `endIndex - 1`. For the paragraph just before a table, and for a table cell's only paragraph, delete the text only, `[start, end - 1]`; an empty paragraph stays, because Google keeps one before every table and in every cell.

重写多段时先处理索引最高的一段。段落中只有部分词语变化时，只在 `find` 给出的范围内删除并插入那些词语，而不是整段。段落其他位置的脚注标记、chip 或图片得以保留；在建议模式下，修订也只显示变化的词语。要删除整段，就删除 `[start, end]`。有三种情况该范围会被拒绝，报 "Invalid deletion range" 或 "The range cannot include the newline character at the end of the segment"；此时改用以下范围。对文档最后一段，从上一段的 `endIndex - 1` 删到最末段的 `endIndex - 1`。对紧邻表格之前的段落，以及表格单元格内唯一的段落，只删文字本身 `[start, end - 1]`；空段落保留，因为 Google 会在每张表格前和每个单元格内保留一个段落。

**Append to the end.** Insert at the body end index minus 1 that `outline` prints. Start the text with a newline to begin a new paragraph. The new paragraph takes the named style of the one it splits from, so after a heading, set it to `NORMAL_TEXT` with `updateParagraphStyle`.

**追加到文末。**在 `outline` 打印的正文结束索引减 1 处插入。让文本以换行开头即可开启新段落。新段落继承被拆开段落的命名样式，因此在标题之后追加时，要用 `updateParagraphStyle` 把它设为 `NORMAL_TEXT`。

**Insert new paragraphs with styles.** Insert the text, then style ranges you compute from the insert point and the text length (in UTF-16 units). Headings use `updateParagraphStyle` with `namedStyleType` `HEADING_1` to `HEADING_6`, `TITLE`, or `NORMAL_TEXT` and `fields: "namedStyleType"`. Lists use `createParagraphBullets` over the range with a `bulletPreset` such as `BULLET_DISC_CIRCLE_SQUARE` or `NUMBERED_DECIMAL_ALPHA_ROMAN`. Typing "- " or "1. " makes text, not a list, and `createParagraphBullets` over typed markers keeps them as text, so delete them first. For nesting, put one leading tab per nesting level at the start of each nested line, before the first `createParagraphBullets`. It turns the tabs into levels and removes them, which shifts every later index by the number of tabs. Bulleting a paragraph directly after a list, with the same preset, adds it to that list. Inserted text takes the style of the text before it, or at the start of a paragraph the style of that paragraph's first character, so reset bold and italic on new body text with `updateTextStyle` (`fields: "bold,italic"`, empty `textStyle`). Don't reset headings, because that removes their bold.

**插入带样式的新段落。**先插入文本，再对依据插入点和文本长度（以 UTF-16 单位计）算出的范围设置样式。标题用 `updateParagraphStyle`，`namedStyleType` 取 `HEADING_1` 至 `HEADING_6`、`TITLE` 或 `NORMAL_TEXT`，并带 `fields: "namedStyleType"`。列表对相应范围用 `createParagraphBullets`，`bulletPreset` 取 `BULLET_DISC_CIRCLE_SQUARE` 或 `NUMBERED_DECIMAL_ALPHA_ROMAN` 等。手动键入 "- " 或 "1. " 生成的是文本而不是列表，对已键入的标记运行 `createParagraphBullets` 也会把它们保留为文本，因此要先删除这些标记。需要嵌套时，在第一次 `createParagraphBullets` 之前，给每个嵌套行的行首按嵌套层级加相应数量的制表符。该调用把制表符转换为层级并移除它们，这会使之后的每个索引发生制表符数量的偏移。对一个紧跟在列表之后的段落用相同 preset 加项目符号，会把它并入该列表。插入的文本继承其前方文本的样式，或（在段落开头时）继承该段落首字符的样式，因此要用 `updateTextStyle`（`fields: "bold,italic"`，空的 `textStyle`）重置新正文的加粗和斜体。不要对标题重置，那会去掉它们的加粗。

**Add a table with content.** One call, with no second read. Place the table after a paragraph, at that paragraph's `endIndex - 1`: for a table under a heading, that is the last paragraph of the section, or the heading itself if the section is empty. Never use the start of a heading: that leaves an empty heading above the table, the heading's font in its cells, and, with `new-table`, the heading itself turned into body text. Write the rows to a JSON file and run:

**添加带内容的表格。**一次调用完成，无需二次读取。把表格放在某个段落之后、该段落 `endIndex - 1` 的位置：标题下的表格，该位置就是本节最后一个段落；节为空时就是标题本身。绝不要用标题的起始位置：那会在表格上方留下一个空标题，让表格单元格带上标题字体，而且用 `new-table` 时标题本身会变成正文。把行数据写入 JSON 文件，然后运行：

```
python <skill>/scripts/docs_index.py new-table --at <endIndex - 1> --data rows.json --revision <revisionId> [--tab <tabId>] [--bold-header]
```

Pass its output as the `requests` and `writeControl` of one `update_doc` call. It inserts the table, fills the cells, bolds the header row with `--bold-header`, and sets the empty paragraph Google adds after the table to `NORMAL_TEXT` (otherwise, after a heading, it becomes an empty heading). Pass `--tab` whenever the doc has more than one tab; without it every request goes to the first tab. The command needs only the index, the revisionId and the tabId, which you can read from `read_doc` even when the result came back in the chat rather than as a file.

把它的输出作为一次 `update_doc` 调用的 `requests` 和 `writeControl`。它会插入表格、填充单元格、按 `--bold-header` 加粗表头行，并把 Google 在表格后自动添加的空段落设为 `NORMAL_TEXT`（否则，在标题之后它会变成一个空标题）。文档有多个标签页时必须传 `--tab`；不传则所有请求都落到第一个标签页。该命令只需要索引、revisionId 和 tabId；即使 `read_doc` 结果是返回在对话里而非保存为文件，你也能读到这三项。

Without the script, put the same requests in one batch: `insertTable` at index `i` makes a table that starts at `i + 1`; in an R x C table, cell (r, c)'s text goes at `i + 4 + r × (2C + 1) + 2c`, and the empty paragraph after the table is at `i + 3 + R × (2C + 1)` until you insert text, so put its `NORMAL_TEXT` reset right after `insertTable`. Fill from the last cell to the first, so each insert only shifts cells already filled.

不用脚本时，把同样的请求放进一个批次：在索引 `i` 处 `insertTable` 生成的表格从 `i + 1` 开始；R 行 C 列的表格中，单元格 (r, c) 的文字位于 `i + 4 + r × (2C + 1) + 2c`，表格后的空段落在你插入文字之前位于 `i + 3 + R × (2C + 1)`，因此把它的 `NORMAL_TEXT` 重置紧跟在 `insertTable` 之后。从最后一个单元格向前逐个填充，这样每次插入只会移动已填充过的单元格。

To fill a table that already exists, a cell's text goes at its first paragraph's `startIndex`, which is the cell's own `startIndex + 1`; Google rejects an insert at the cell's `startIndex` ("The insertion index must be inside the bounds of an existing paragraph"). With a saved read, `fill-table` builds these requests; it assumes the cells are empty.

填充既有表格时，单元格文字位于其首个段落的 `startIndex`，即该单元格自身 `startIndex + 1` 的位置；在单元格的 `startIndex` 处插入会被 Google 拒绝（"The insertion index must be inside the bounds of an existing paragraph"）。有已保存的读取结果时，`fill-table` 会构建这些请求；它假定单元格为空。

**Change table structure.** `insertTableRow`, `deleteTableRow`, `insertTableColumn`, and `deleteTableColumn` take `tableCellLocation: {"tableStartLocation": {"index": <table startIndex>}, "rowIndex": r, "columnIndex": c}`. `updateTableCellStyle` sets cell backgrounds over a `tableRange` built from the same location plus `rowSpan` and `columnSpan`. Colors use `rgbColor` values from 0 to 1. A row inserted below a bold header row comes out bold, and a new column copies its neighbor's width and background. To delete a whole table, delete exactly `[table startIndex, table endIndex]`.

**修改表格结构。**`insertTableRow`、`deleteTableRow`、`insertTableColumn` 和 `deleteTableColumn` 接受 `tableCellLocation: {"tableStartLocation": {"index": <table startIndex>}, "rowIndex": r, "columnIndex": c}`。`updateTableCellStyle` 通过由同一位置加上 `rowSpan` 和 `columnSpan` 构成的 `tableRange` 设置单元格背景。颜色使用 0 到 1 的 `rgbColor` 值。在加粗表头行下方插入的行会带加粗，新列会复制相邻列的宽度和背景色。要删除整张表格，精确删除 `[table startIndex, table endIndex]`。

**Suggest instead of edit.** Add `"writeMode": "SUGGEST"` to `writeControl`, and the same requests land as tracked suggestions that the doc's owner can accept or reject. Use it when the user asks for suggestions, redlines, tracked changes, or a review, or when the doc belongs to someone else and the user wants to propose rather than change. You can't accept or reject suggestions through the API, so tell the user they're waiting in the doc. Before you tell them, read the doc again with `read_doc` and run `outline`: inserted and deleted text should show as `[+...]` or `[-...]` (outline doesn't show suggested formatting changes). If new text shows as plain text, or deleted text is gone instead of showing as `[-...]`, the change was made directly; say so, and don't call it a suggestion. Suggested deletions stay in the index space until someone resolves them; see "How a doc is addressed". The connector has answered this mode with "Unsupported WriteControl mode". If it does, make no direct edits in its place: tell the user suggestions aren't available, and offer to list the proposed changes in your reply or to edit directly.

**以建议模式代替直接编辑。**在 `writeControl` 中加上 `"writeMode": "SUGGEST"`，同样的请求会以跟踪修订的形式落地，由文档所有者接受或拒绝。用户要求建议、红线批注、修订跟踪或审阅时，或文档属于他人而用户想提案而非直接修改时使用。API 无法接受或拒绝建议，因此要告知用户建议正在文档中等候处理。告知之前，先用 `read_doc` 重读文档并运行 `outline`：插入和删除的文本应显示为 `[+...]` 或 `[-...]`（outline 不显示建议的格式变更）。若新文本显示为普通文本，或被删文本直接消失而没有显示为 `[-...]`，说明改动是直接做出的；要如实说明，不要称之为建议。建议的删除在有人处理之前一直占据索引空间，见"文档如何寻址"一节。连接器曾对该模式返回 "Unsupported WriteControl mode"。若遇到，不要改用直接编辑：告知用户建议功能不可用，并提出可以在回复中列出拟议改动，或直接编辑。

```json
{"documentId": "...", "requests": [...],
 "writeControl": {"writeMode": "SUGGEST", "requiredRevisionId": "..."}}
```

**Tabs.** `addDocumentTab` with `tabProperties.title` creates a tab, and the reply holds its `tabId`. `updateDocumentTabProperties` renames one (`fields: "title"`), and `deleteTab` removes one along with its child tabs, but not the only tab. Set `tabProperties.parentTabId` to create a child tab; tabs nest at most three levels deep. A new tab starts with one empty paragraph, so its first insert index is 1.

**标签页。**`addDocumentTab` 带 `tabProperties.title` 创建标签页，响应中含其 `tabId`。`updateDocumentTabProperties` 重命名标签页（`fields: "title"`）；`deleteTab` 删除标签页及其子标签页，但唯一标签页不可删。设置 `tabProperties.parentTabId` 可创建子标签页；标签页最多嵌套三层。新标签页以一个空段落开始，因此其首个插入索引为 1。

## Verify / 验证

- For content, read with Drive `read_file_content` and check the text you changed.
  内容方面，用 Drive 的 `read_file_content` 读取，核对你改过的文字。
- For positions and styles, read with `read_doc` and run `outline` or `find` on the result. `find` shows whether replaced text picked up bold, and whether a match sits in a pending suggestion.
  位置和样式方面，用 `read_doc` 读取并对结果运行 `outline` 或 `find`。`find` 能显示被替换文本是否带上了加粗，以及某处匹配是否位于待处理的建议内。
- For layout, where you can run code, export with Drive `download_file_content` and `exportMimeType: "application/pdf"`, then run `python <skill>/scripts/render_export.py <saved export> <out dir>` and check the page images against the list under "Check the result" above. If the export comes back in the chat instead of being saved to a file, skip the render rather than copying it into a file. If you can't run code, skipped the render, or it failed, rely on the two checks above and tell the user you couldn't check the layout visually.
  版式方面，在能运行代码的场合，用 Drive 的 `download_file_content` 加 `exportMimeType: "application/pdf"` 导出，然后运行 `python <skill>/scripts/render_export.py <saved export> <out dir>`，按上文"检查成果"清单核查页面图像。导出结果是返回在对话中而非保存为文件时，跳过渲染，不要把它复制进文件。无法运行代码、跳过了渲染或渲染失败时，依靠前两项检查，并告知用户无法进行版式的视觉核查。

## Docs failures / Docs 常见故障

| Symptom | Cause | Fix |
|---|---|---|
| `read_doc` result too large or cut off | Full document JSON for a long or table-heavy doc | Orient with Drive `read_file_content`. Use `replaceAllText` where the find text matches only the text you mean to change (add neighboring words until it does). If the result was saved to a file, run `docs_index.py` on it. |
| An edit landed in the wrong tab | No `tabId` in the location or range | Add the `tabId` from the read or from the URL's `?tab=` value. |
| `replaceAllText` changed other tabs | No `tabsCriteria` | Scope it with `tabsCriteria.tabIds`. |
| `replaceAllText` reports 0 occurrences | Find text copied from `read_file_content`, which escapes `&`, `*`, and similar characters, or glues suggestions | Use the literal text, or check it with `docs_index.py find`. |
| Text shows as "RBRunning back" | A pending suggestion read as plain text | Read with `read_doc`. `outline` marks suggested text as `[+...]` and `[-...]`. |
| A heading loses its bold after an edit | A style reset was applied to the heading | Reset styles only on body text. |
| Replaced text turned bold | The replacement inherited the style of the first replaced character | Clear it with `updateTextStyle` over the new range. |
| Edits land in the wrong place | Requests ran from the lowest index to the highest, or used indexes from before an earlier write | Order requests from the highest index to the lowest, and read again after every write. |
| Typed "- " shows as text, not a bullet | Lists are paragraph properties | Delete the typed markers, then use `createParagraphBullets` over the range. Bulleting keeps typed markers as text. |

| 症状 | 原因 | 解决方法 |
|---|---|---|
| `read_doc` 结果过大或被截断 | 长文档或表格繁多的文档的完整 JSON | 先用 Drive 的 `read_file_content` 了解全貌。在查找文本只匹配目标文字处使用 `replaceAllText`（不断加入相邻词语直到唯一匹配）。结果已保存为文件时，对其运行 `docs_index.py`。 |
| 编辑落到了错误的标签页 | `location` 或 `range` 缺少 `tabId` | 加上读取结果或 URL `?tab=` 参数中的 `tabId`。 |
| `replaceAllText` 改动了其他标签页 | 缺少 `tabsCriteria` | 用 `tabsCriteria.tabIds` 限定范围。 |
| `replaceAllText` 报告 0 处匹配 | 查找文本复制自 `read_file_content`，其中 `&`、`*` 等字符已被转义，或建议内容与正文粘连 | 使用字面文本，或用 `docs_index.py find` 核查。 |
| 文本显示为 "RBRunning back" | 待处理的建议被当作普通文本读取 | 用 `read_doc` 读取。`outline` 会把建议文本标为 `[+...]` 和 `[-...]`。 |
| 标题在编辑后失去加粗 | 对标题执行了样式重置 | 只对正文文本重置样式。 |
| 被替换文本变成加粗 | 替换继承了被替换首字符的样式 | 对新范围用 `updateTextStyle` 清除。 |
| 编辑落在错误位置 | 请求按索引从低到高执行，或使用了早前写入之前读取的索引 | 请求按索引从高到低排序，且每次写入后重新读取。 |
| 手动键入的 "- " 显示为文本而非项目符号 | 列表是段落属性 | 删除键入的标记，再对该范围运行 `createParagraphBullets`。加项目符号会把键入的标记保留为文本。 |
