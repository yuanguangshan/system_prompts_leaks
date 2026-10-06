---
name: google-workspace
description: "Read this before the first Google Drive, Docs, Sheets or Slides connector call whenever the task creates or changes a Google file. Use this skill whenever the user wants to create or change a Google Doc, Sheet or Slides file in their Google Drive. Triggers include: a request that names Google Docs, Sheets, Slides or Drive and asks to make, edit, format, copy or rename a file; a docs.google.com link with a request to change that file, even a one-line fix or suggested edits; and any follow-up change to a Google file from earlier in the chat, even \"change it\" or \"add a tab\". Includes helper scripts for document positions, cell ranges and slide layout. However, if the user asks for a doc, deck or spreadsheet without naming Google, or gives a Google file only as source material for something new, use Claude's own output type instead. Do NOT use for read-only questions about a Google file, or for Word, Excel, PowerPoint or PDF files."
---
<!-- BILINGUAL-EN-ZH -->

# Google Docs, Sheets, and Slides / Google 文档、表格与幻灯片

The Google connectors are thin wrappers over Google's raw APIs, with almost no guidance of their own. This skill supplies that guidance: which connector does what, the rules that keep edits in the user's file, and, in one reference file per app, how each API really behaves. Most of the app-specific rules in the references were tested against Google's Docs, Sheets and Slides APIs; the rest were seen through the connectors, and a few have not been checked yet.

Google 连接器是对 Google 原始 API 的轻量封装，自身几乎没有提供任何指导。本技能补充这些指导：哪个连接器做什么、保证编辑落在用户文件内的规则，以及每个应用一份参考文件中各 API 的真实行为。参考文档中大多数应用专属规则都针对 Google 的 Docs、Sheets 和 Slides API 做过测试；其余规则通过连接器观察得到，还有少数尚未验证。

## Read the reference before you touch the file / 动手改文件前先读参考文档

| Working on | Read first | Why it matters |
|---|---|---|
| A Google Doc | `references/docs.md` | Docs edits address UTF-16 positions that shift after every insert and go stale after every write. Tabs and pending suggestions change the positions too. |
| A Google Sheet | `references/sheets.md` | Both write tools parse input like the Sheets UI, so text can silently become numbers or dates. Formatting needs numeric sheet IDs, 0-based ranges, and field masks without parentheses. |
| A Google Slides deck | `references/slides.md` | Positions are in EMU, an element's real size is its size times its scale, and an unmasked read can exceed 150 KB for three slides. Slides never shrinks text to fit. |

| 处理对象 | 先读文档 | 为何重要 |
|---|---|---|
| Google 文档 | `references/docs.md` | Docs 编辑使用 UTF-16 位置，每次插入后位置都会移动，每次写入后位置都会失效。标签页和待处理的建议也会改变位置。 |
| Google 表格 | `references/sheets.md` | 两个写入工具都按 Sheets 界面的方式解析输入，文本可能被静默转换成数字或日期。格式化需要数字形式的表格 ID、从 0 开始的范围以及不带括号的字段掩码。 |
| Google 幻灯片 | `references/slides.md` | 位置以 EMU 表示，元素的真实尺寸是其尺寸乘以缩放比例，且一次不带掩码的读取对三张幻灯片就可能超过 150 KB。幻灯片从不会缩小文字以适应形状。 |

Read the reference for every app the task touches before the first edit. Embedding a Sheets chart in a deck means reading both. The references are short, and skipping one is how edits land in the wrong place.

在首次编辑前，阅读任务涉及的每个应用的参考文档。在幻灯片中嵌入 Sheets 图表意味着两份都要读。参考文档都很简短，跳过其中一份正是编辑落错位置的原因。

## 1. Check the connectors before you start / 开始前先检查连接器

| Connector | What it can do |
|---|---|
| Google Drive | Create files, upload and convert content, rename, read a file as text, export (PDF and other formats), search, trash |
| Google Docs | Read a doc's full structure and edit it in place, directly or as suggestions |
| Google Sheets | Read values and structure, write values and formulas, format, add tabs and charts |
| Google Slides | Read a deck, add and edit slides, shapes, text, tables, and linked charts |

| 连接器 | 能做的事 |
|---|---|
| Google Drive | 创建文件、上传并转换内容、重命名、将文件读取为文本、导出（PDF 及其他格式）、搜索、移入回收站 |
| Google Docs | 读取文档完整结构并就地编辑，可直接编辑或以建议形式编辑 |
| Google Sheets | 读取值与结构、写入值与公式、设置格式、添加标签页和图表 |
| Google Slides | 读取演示文稿、添加和编辑幻灯片、形状、文本、表格以及关联图表 |

Drive can create all three file types. It can't edit a file after that. Without the matching editor connector, every change means a new file and a new link, and the user loses the link they already have.

Drive 可以创建全部三种文件类型，但创建之后无法编辑文件。没有对应的编辑器连接器时，每次修改都意味着一个新文件和一个新链接，用户会失去他们已有的链接。

1. Check which Google tools are available in this conversation. On surfaces where tools are deferred, search for and load the tools you need first, such as "google sheets update". Tool names differ by surface, so use the names your surface lists. If a search returns nothing, list all available tools before deciding the connector is missing, because the tool may exist under a different name.
   检查本次对话中可用的 Google 工具。在工具延迟加载的界面上，先搜索并加载所需工具，例如 "google sheets update"。工具名称因界面而异，请使用你的界面所列出的名称。如果搜索无结果，先列出所有可用工具再断定连接器缺失，因为该工具可能以其他名称存在。
2. If the editor tools are missing, tell the user. When you can list the conversation's connectors, say which case it is: a connector that is set up but turned off in this chat (ask them to turn it on in the chat's connector settings, then continue), or one that isn't connected at all (tell them which connector to add and what it enables).
   如果缺少编辑器工具，请告知用户。当你能列出对话的连接器时，说明属于哪种情况：连接器已配置但在本对话中被关闭（请用户在对话的连接器设置中开启，然后继续），或完全未连接（告知用户应添加哪个连接器以及它能实现什么）。
3. With an editor connector missing, a request to create a new file continues with Drive. A change to an existing file stops and asks. See the missing-connector rule in section 2.
   在缺少编辑器连接器的情况下，创建新文件的请求继续用 Drive 完成；对既有文件的修改则停下来询问。参见第 2 节的缺失连接器规则。
4. If the user asks only for a new file, create it with Drive. If the editor connector is off, add one line saying edits will need it.
   如果用户只要求创建新文件，就用 Drive 创建。如果编辑器连接器被关闭，加一句说明后续编辑需要它。

Example: "I can create the sheet now. If you want changes later, turn on the Google Sheets connector in this chat first. Then I can edit this file, and the link will stay the same."

示例："我现在就可以创建这个表格。如果之后需要修改，请先在本对话中开启 Google Sheets 连接器。届时我就能编辑这个文件，链接也将保持不变。"

## 2. Rules for every file / 适用于所有文件的规则

- **A change goes in the same file.** "Change", "update", "fix", "add", and "switch it to" all mean the user wants the same file and link. Do not recreate the file to skip an edit.
  **修改落在同一个文件里。**"Change"、"update"、"fix"、"add"和"switch it to"都表示用户想要同一个文件和同一个链接。不要为了绕开某次编辑而重建文件。
- **A missing editor connector is a choice for the user, not a workaround for you.** When the user asks for a change to an existing file and the editor connector is missing, your whole reply is a short question, not a deliverable. Building the next-best thing feels helpful, but the user's file is still untouched and now there are two artifacts, which is the exact failure this skill exists to prevent. Lead with the fix: name the connector and say that with it on, the edit lands in their existing file and the link stays the same. You may offer an alternative, such as drafted text to paste or a file to import by hand, but only as a named option, and build it only after the user picks it. Example reply, in full: "To add the slide to your deck directly, turn on the Google Slides connector in this chat and I'll do it. Same deck, same link. Or I can draft the slide as a file you'd import by hand. Which do you prefer?"
  **缺失编辑器连接器是留给用户的选择，而不是你绕过的理由。**当用户要求修改既有文件而编辑器连接器缺失时，你的整个回复应当是一个简短的问题，而不是一份替代成品。自作主张造一个次优方案看似有帮助，但用户的文件仍未被改动，而且现在多了两个产物，这正是本技能要防止的失败。以解决方案开头：指出连接器名称，并说明开启后编辑会落入他们已有的文件且链接保持不变。你可以提供替代方案，例如起草好待粘贴的文本或供手动导入的文件，但只能作为明确标注的选项，且在用户选择后才去构建。完整回复示例："要直接把这张幻灯片加进你的演示文稿，请在本对话中开启 Google Slides 连接器，我来操作。同一个文稿，同一个链接。或者我可以把幻灯片起草成一个供你手动导入的文件。你更倾向哪种？"
  【评论】这是一条防"自作主张"条款：禁止模型在工具缺失时用替代产出冒充完成，把方案选择权显式交还给用户。
- **Never trash a file the user didn't ask you to delete.** Drive can trash a file, but it can't restore one. The Docs, Sheets, and Slides connectors can't open a trashed file.
  **绝不删除用户没有要求删除的文件。**Drive 可以把文件移入回收站，但无法恢复它。Docs、Sheets 和 Slides 连接器无法打开已进回收站的文件。
  【评论】这是一条不可逆操作防护条款：连接器只提供单向的移入回收站能力而无恢复能力，因此规则要求删除决定完全由用户做出。
- **Read before you edit, and guard the write.** Every Docs and Slides read returns a `revisionId`. Pass it as `writeControl.requiredRevisionId` on the next batch update. If the file changed in between, the whole batch is rejected with a 400 ("does not match the latest revision") instead of landing on stale positions. Write replies don't return the new revision, so read again before the next guarded write. On a rejection, read again and rebuild the requests; never retry without the guard. Sheets is different: the Sheets reads in `references/sheets.md` return no `revisionId`, so read again right before a Sheets write and send it without `writeControl`.
  **先读后改，并为写入加保护。**每次 Docs 和 Slides 读取都会返回一个 `revisionId`。在下一次批量更新中把它作为 `writeControl.requiredRevisionId` 传入。如果文件在此期间发生了变化，整批请求会被以 400（"does not match the latest revision"）拒绝，而不是落在过期的位置上。写入响应不返回新版本号，因此在下一次受保护的写入前要重新读取。被拒绝时，重新读取并重建请求；绝不要在没有保护的情况下重试。Sheets 不同：`references/sheets.md` 中的 Sheets 读取不返回 `revisionId`，因此要在 Sheets 写入前立即重新读取，并不带 `writeControl` 地发送。
  【评论】这是典型的乐观并发控制：用读取时获得的 revisionId 检测读取与写入之间发生的外部变更，避免编辑基于过期的位置信息。
- **Put everything for one step in one batch.** Batch requests run in order and atomically: if one is invalid, none apply. One call per logical step is faster and leaves no half-finished state.
  **一个步骤的所有内容放进一个批次。**批次请求按顺序且原子性地执行：只要有一个无效，全部都不生效。每个逻辑步骤一次调用更快，也不会留下完成一半的状态。
- **Verify the result.** Read the file again after each edit and check the result before you report it. A tool accepting the call does not mean the content is correct. Each reference has a Verify section.
  **核实结果。**每次编辑后重新读取文件，在汇报前检查结果。工具接受调用并不代表内容正确。每份参考文档都有一个 Verify 小节。
- **Rename with Drive `update_file`.** A rename keeps the same link.
  **用 Drive 的 `update_file` 重命名。**重命名会保持链接不变。
- **Link every file you name.** When you name a Google file you created, copied, found, or edited, make its title a link, inside the sentence that says what you did: "I added the row to [Team roster](its link)." Don't set the file apart after a colon or on a line of its own ("I've created a file for you: Team roster"). If a Drive result says the user can already see or open a file from the chat, leave that file's link out unless they ask. Always give the link when the user asks for it, wants to send it to someone, or says they can't see or open the file, even if a result said they could. Don't tell the user to use a card or preview unless a result said one is shown. After an edit, say that the link stays the same.
  **提到的每个文件都加上链接。**当你提到你创建、复制、找到或编辑过的 Google 文件时，把它的标题做成链接，并放在说明你做了什么的句子内："I added the row to [Team roster](its link)."。不要在冒号之后或单独一行把文件隔开（"I've created a file for you: Team roster"）。如果 Drive 结果表明用户已经能在对话中看到或打开某个文件，除非用户要求，否则省略该文件的链接。当用户主动要链接、想把它发给别人，或说自己看不到或打不开文件时，即使有结果说他们能看到，也始终给出链接。不要让用户去使用卡片或预览，除非结果表明有卡片展示。编辑之后，说明链接保持不变。

## 3. Drive, for all three apps / Drive：三个应用通用

- **Find a file:** use Drive search when the user names a file without a link. Confirm the match with the user if more than one file fits.
  **查找文件：**当用户说出文件名但没有给链接时，使用 Drive 搜索。如果匹配的文件不止一个，与用户确认。
- **Create:** `create_file` with `contentMimeType` set to `application/vnd.google-apps.document`, `.spreadsheet`, or `.presentation` creates an empty file. Uploading content with a source type (HTML, CSV, .xlsx, .pptx) converts it into the matching Google type. Each reference says which route fits that app.
  **创建：**`create_file` 将 `contentMimeType` 设为 `application/vnd.google-apps.document`、`.spreadsheet` 或 `.presentation` 时会创建空文件。上传带有源类型（HTML、CSV、.xlsx、.pptx）的内容会将其转换为对应的 Google 类型。每份参考文档会说明哪个路径适合该应用。
- **Read as text:** `read_file_content` returns a compact text rendering: Markdown for a Doc. It is a few KB where the editor connector's full read can be over 100 KB, so use it to understand content, then use the editor read when you need positions.
  **以文本读取：**`read_file_content` 返回紧凑的文本呈现：对文档是 Markdown。它只有几 KB，而编辑器连接器的完整读取可能超过 100 KB，因此可先用它理解内容，需要位置信息时再用编辑器读取。
- **Export:** `download_file_content` with `exportMimeType: "application/pdf"` returns the rendered file as base64. It is large (about 65 KB of text for a four-slide deck), so export only for a visual check, and only where you can run code.
  **导出：**`download_file_content` 设 `exportMimeType: "application/pdf"` 会以 base64 返回渲染后的文件。它很大（四页幻灯片约 65 KB 文本），因此仅在需要视觉检查且能运行代码的场景使用。

## 4. Helper scripts / 辅助脚本

The `scripts/` folder next to this file holds tested helpers. They turn the APIs' raw JSON into short readable summaries, and turn simple specs into correct request batches, so the error-prone arithmetic never happens by hand. Use them wherever you can run Python. Run them by their full path inside this skill's folder; the references write that folder as `<skill>`.

本文件旁边的 `scripts/` 文件夹存放经过测试的辅助脚本。它们把 API 的原始 JSON 转换为简短易读的摘要，并把简单规格转换为正确的请求批次，从而避免手工进行容易出错的算术计算。凡能运行 Python 的地方都应使用它们。以本技能文件夹内的完整路径运行；参考文档中把该文件夹写作 `<skill>`。

| Script | Commands | Used for |
|---|---|---|
| `docs_index.py` | `outline`, `find`, `new-table`, `fill-table` | Docs positions, text search with bold and suggestion state, adding a filled table in one call, filling an existing table |
| `sheets_helper.py` | `range`, `format`, `cells` | A1 ranges to grid ranges, formatting batches from a short spec, reading formulas and error cells |
| `slides_helper.py` | `outline`, `build` | Real slide geometry with overflow and overlap warnings, building slides from an inch-based spec |
| `render_export.py` | one command | Decoding a PDF export into page images to look at |

| 脚本 | 命令 | 用途 |
|---|---|---|
| `docs_index.py` | `outline`, `find`, `new-table`, `fill-table` | Docs 位置、带加粗与建议状态的文本搜索、一次调用添加已填充的表格、填充既有表格 |
| `sheets_helper.py` | `range`, `format`, `cells` | A1 范围转网格范围、由简短规格生成格式化批次、读取公式与错误单元格 |
| `slides_helper.py` | `outline`, `build` | 带溢出与重叠警告的真实幻灯片几何信息、按英寸规格构建幻灯片 |
| `render_export.py` | one command | 把 PDF 导出解码为页面图片以供查看 |

**Getting JSON to a script.** Large tool results are often saved to a file by the app, and the result tells you the path. Point the script at that path; the scripts read the file as the app saved it. When a result comes back in context and is small (a few KB, such as a masked read or sheet metadata), write it to a file and run the script on it. Don't re-type a large result into a file: that doubles the cost and invites copying errors. If a large result was cut off and not saved, read a smaller slice instead (a field mask, a range, or one tab) rather than guessing.

**把 JSON 交给脚本。**大型工具结果通常由应用保存到文件，结果中会告知路径。让脚本指向该路径；脚本会按应用保存的原样读取文件。当结果直接返回在上下文中且较小时（几 KB，例如带掩码的读取或表格元数据），把它写入文件再对文件运行脚本。不要把大结果重新敲进文件：那会使成本翻倍并容易引入抄写错误。如果大结果被截断且未保存，改为读取更小的一片（字段掩码、一个范围或单个标签页），而不是靠猜测。

Every script prints `--help` with its full usage. Each reference shows the commands in context.

每个脚本都会输出 `--help` 及其完整用法。每份参考文档都会在上下文中展示这些命令。

## 5. Common failures across apps / 各应用常见故障

| Symptom | Cause | Fix |
|---|---|---|
| "No such tool available" | The tool is deferred, or the name is different on this surface | Search for and load the tool. Use the exact name your surface lists. |
| Permission denied on a file you created | The file is in the trash | Ask the user to restore it from Drive's trash. You can't restore it with the connectors. |
| 400: required revision ID does not match | The file changed after your read, often because of your own previous write | Read again, rebuild the requests from the new read, and send them with the new revision. |
| A whole batch failed on one bad request | Batches are atomic | Fix the named request and resend the full batch. Nothing from the failed call was applied. |
| A read is too large or cut off | Unmasked editor reads include everything | Use Drive `read_file_content`, a field mask, a range, or a single tab. Run the helper on a saved result. |
| Two files where the user expected one | An edit was done by creating a new file | Make changes in place with the editor connector. Tell the user about the extra file; don't trash it without asking. |

| 症状 | 原因 | 解决方法 |
|---|---|---|
| "No such tool available" | 工具被延迟加载，或在该界面上名称不同 | 搜索并加载该工具。使用你的界面列出的确切名称。 |
| 对你创建的文件权限被拒 | 文件在回收站里 | 请用户从 Drive 回收站恢复。你无法用连接器恢复。 |
| 400: required revision ID does not match | 文件在你的读取之后发生了变化，往往是你自己上一次写入所致 | 重新读取，基于新读取重建请求，并携带新版本号发送。 |
| 整批因一个错误请求而失败 | 批次是原子的 | 修复所指出的请求并重发完整批次。失败的调用没有应用任何内容。 |
| 读取结果过大或被截断 | 不带掩码的编辑器读取会包含全部内容 | 改用 Drive 的 `read_file_content`、字段掩码、范围或单个标签页。对已保存的结果运行辅助脚本。 |
| 用户期望一个文件却出现两个 | 修改是通过创建新文件完成的 | 用编辑器连接器就地修改。告知用户多出的文件；未经询问不要删除。 |

Each reference ends with the failures specific to that app.

每份参考文档的结尾是该应用特有的故障。

## 6. What the connectors can't do / 连接器做不到的事

When a request runs into one of these, say so plainly and do the alternative. *Reported*: seen through the connectors or in their tool schemas. *Untested*: not yet checked.

当请求遇到下列情况之一时，如实说明并采用替代做法。*Reported*：通过连接器或其工具 schema 观察到。*Untested*：尚未验证。

- **Claude can't see the user's screen.** No connector shows what the user has selected, or which file, tab or slide they have open. If "this" or "here" isn't clear from the chat, ask which file, heading, slide number or cell range they mean. *Reported.*
  **Claude 看不到用户的屏幕。**没有任何连接器能显示用户选中了什么，或打开的是哪个文件、标签页或幻灯片。如果"this"或"here"无法从对话中判断，询问用户指的是哪个文件、哪个标题、第几页幻灯片或哪个单元格范围。*Reported.*
- **Comments:** `read_doc` has returned no comments even with `commentsIncluded: true`. To read comments, use Drive `read_file_content` with `includeComments: true`, or ask the user to paste them. `update_doc` has accepted `insertComment` in one report, although its description is cut before listing it; if that request fails, say you can't add comments and offer to list your notes in the reply. *Reported.*
  **评论：**即使设置 `commentsIncluded: true`，`read_doc` 也未返回过评论。要读取评论，使用 Drive 的 `read_file_content` 并设 `includeComments: true`，或请用户粘贴评论。有一例报告显示 `update_doc` 接受了 `insertComment`，尽管其描述在列出该功能之前就被截断；如果该请求失败，说明你无法添加评论，并提出在回复中列出你的笔记。*Reported.*
- **Suggestion mode can be refused** with "Unsupported WriteControl mode". Never make direct edits in its place: offer to list the proposed changes or to edit directly (see `references/docs.md`). *Reported.*
  **建议模式可能被拒绝**，报 "Unsupported WriteControl mode"。绝不要因此改为直接编辑：应提出列出拟议修改，或直接编辑（见 `references/docs.md`）。*Reported.*
- **Request lists are cut short.** The `update_doc`, `update_spreadsheet` and `update_presentation` descriptions stop partway through their request lists. Requests missing from the description, such as `addDocumentTab` and `addChart`, have still worked, so try one before deciding it isn't supported. *Reported.*
  **请求列表被截断。**`update_doc`、`update_spreadsheet` 和 `update_presentation` 的描述都在请求列表中途停止。描述中缺失的请求（例如 `addDocumentTab` 和 `addChart`）仍然可能成功，因此在断定不支持之前先试一次。*Reported.*
- **Uploads travel inside the call.** `create_file` takes the whole file as text or base64, so an .xlsx or .pptx upload is slow and often fails. Build in the file with the editor connectors where you can, and keep uploads small. *Reported.*
  **上传随调用本身传输。**`create_file` 把整个文件作为文本或 base64 放进调用，因此 .xlsx 或 .pptx 上传很慢且经常失败。尽量用编辑器连接器在文件内构建内容，并保持上传文件较小。*Reported.*
- **Images need a public URL.** Slides `createImage` and Docs `insertInlineImage` fetch the image from a URL Google can reach; a file from the chat can't be inserted this way. For a deck, offer the .pptx route; otherwise ask the user to insert the image. *Untested.*
  **图片需要公开 URL。**Slides 的 `createImage` 和 Docs 的 `insertInlineImage` 从 Google 能访问的 URL 获取图片；对话中的文件无法以这种方式插入。对演示文稿，提供 .pptx 路径；否则请用户自行插入图片。*Untested.*
- **A Slides chart must already exist in a Sheet.** Create it there with `addChart`, then embed it with `createSheetsChart`. *Reported.*
  **Slides 图表必须已存在于某个 Sheet 中。**先用 `addChart` 在其中创建，再用 `createSheetsChart` 嵌入。*Reported.*
- **Drive search uses `title`, not `name`.** `name contains '…'` fails with "Unsupported query field: name"; write `title contains '…'`, and put the file type in a `mimeType` clause. *Reported.*
  **Drive 搜索使用 `title`，而不是 `name`。**`name contains '…'` 会报 "Unsupported query field: name"；应写 `title contains '…'`，并把文件类型放进 `mimeType` 子句。*Reported.*
- **Slides edits may not show in an open deck right away.** If the user doesn't see a change, ask them to reload the deck before you change anything again. *Reported; may be fixed.*
  **幻灯片编辑可能不会立即显示在已打开的文稿中。**如果用户看不到更改，请他们重新加载文稿，然后你再做下一次修改。*Reported; may be fixed.*
