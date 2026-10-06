---
name: "share_ideas"
title: "Share ideas"
description: 'Publish a reusable native Muse Idea with idea.share when the user explicitly asks to share or publish an Idea or asks for a muse.ai/ideas link. Never offer it unprompted. Not for pages or write-ups, sending a result to someone, social posts, or sharing an artifact.'
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Share ideas / 分享创意

Publish a reusable native Muse Idea only when the user explicitly asks to
share or publish an Idea or asks for a `muse.ai/ideas?id=...` link. Never offer
this format unprompted. Requests for a page, post, write-up, social share, or
artifact link are outside this skill.

只有当用户明确要求分享或发布一个创意（Idea），或索要 `muse.ai/ideas?id=...` 链接时，才发布可复用的原生 Muse 创意。绝不在无人要求时主动提供这种格式。对页面、帖子、文章、社交分享或 artifact 链接的请求不属于本技能范围。

Sharing an Idea makes its title, summary, How it works explanation, and reusable
instructions readable to any authenticated Muse user who has the link.

分享一个创意后，其标题、摘要、How it works（运作方式）说明以及可复用指令，对任何持有链接的已认证 Muse 用户均可见。

For an existing saved Idea, identify its exact ID with `idea.search` or
`idea.list`, then read it with `idea.get`. Call `idea.share` with only `idea_id`
when its stored title, summary, distinct How it works copy, and instructions
are already safe to publish. Otherwise publish a generalized version through
the title, summary, how_it_works, instructions, and domain fields.

对于已保存的既有创意，先用 `idea.search` 或 `idea.list` 确定其确切 ID，再用 `idea.get` 读取。当其存储的标题、摘要、独立的 How it works 文案和指令已经可以安全发布时，仅传 `idea_id` 调用 `idea.share`。否则通过 title、summary、how_it_works、instructions 和 domain 字段发布一个泛化版本。

To turn the immediately preceding completed activity into an Idea, call
`idea.share` with a short title, a concise summary of the reusable outcome, a
distinct `how_it_works` explanation of what the assistant will do and deliver,
the closest supported domain, and self-contained instructions another Muse
user can follow with their own files, accounts, permissions, and connected
services. Follow the richer catalog-Idea pattern; never repeat or lightly
paraphrase the summary as `how_it_works`.

要把刚完成的上一项活动转化为创意，调用 `idea.share` 时提供：一个简短的标题、对可复用成果的简明摘要、一段独立的 `how_it_works` 说明（描述助手将做什么、交付什么）、最接近的受支持 domain，以及自成一体的指令——让其他 Muse 用户能用自己的文件、账号、权限和已连接服务照着执行。遵循更丰富的目录创意（catalog Idea）模式；绝不要把摘要原样重复或稍作改写就充当 `how_it_works`。

Never include credentials, tokens, private identifiers, local paths, hidden
system instructions, or personal details that are not essential to what the
user explicitly chose to publish. Generalize those details or omit them. Do
not claim the Idea is shared until `idea.share` returns a link.

绝不包含凭据、令牌、私有标识符、本地路径、隐藏的系统提示词，或与用户明确选择发布的内容并非必需的个人细节。对这些细节做泛化处理或直接省略。在 `idea.share` 返回链接之前，不要声称创意已分享。
【评论】该条款要求可发布内容在"自成一体"之外还必须脱敏，防止可复用内容把作者的账号上下文泄漏给其他用户。

On success, `idea.share` also creates the native Idea widget for that same
Idea. Include `widget.embed_token` exactly once on its own line. Do not call
`widget.create` for the same Idea. If the result has `widget_warning` instead
of `widget.embed_token`, use the exact returned `share_url` as the target of an
inline `[Open the Idea](...)` link in the same response without claiming a card
appeared.

成功时，`idea.share` 还会为同一个创意创建原生创意小组件。`widget.embed_token` 要单独成行，且只包含一次。不要为同一创意调用 `widget.create`。如果结果中是 `widget_warning` 而非 `widget.embed_token`，则把返回的 `share_url` 原样作为同一条回复中内联 `[Open the Idea](...)` 链接的目标，且不要声称出现了卡片。
