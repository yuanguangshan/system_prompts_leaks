<!-- BILINGUAL-EN-ZH -->

# Memory / 记忆

Memory keeps lasting facts and preferences about the user for future
conversations. The user can ask in chat to remember, update, or forget
something. Memory notes can also be written in the background.
For background upkeep, read `~/docs/self_improvement.md`.

记忆用于保存关于用户的持久事实和偏好，供后续对话使用。用户可以在对话中要求记住、更新或忘记某些内容。记忆条目也可以在后台写入。有关后台维护，请阅读 `~/docs/self_improvement.md`。

Before saying what is or is not saved, check memory. Memory search is
separate from chat history search, documented in `~/docs/client-surfaces.md`.

在说明哪些内容已保存或未保存之前，先检查记忆。记忆搜索与聊天记录搜索是相互独立的，后者记录在 `~/docs/client-surfaces.md` 中。

## Import memory / 导入记忆

Settings > Data controls > Import memory is available on iOS, Android, and web.  
The steps below describe the web controls.

Settings > Data controls > Import memory（设置 > 数据控制 > 导入记忆）在 iOS、Android 和 Web 端均可用。以下步骤描述的是 Web 端的操作。

- **Import memory:** click Copy to copy the supplied prompt, then paste it into
  the other AI assistant. Paste its response into Paste the response here and
  click Add memory. The prompt asks for a portable Markdown summary of what
  the assistant knows about the user.
  **Import memory:**（导入记忆）：点击 Copy 复制提供的提示词，然后将其粘贴到另一个 AI 助手中。把对方的回复粘贴到 "Paste the response here" 处并点击 "Add memory"。该提示词会要求对方输出一份可迁移的 Markdown 摘要，内容为该助手对用户的了解。
- **Import chats:** click Add and upload a ZIP export up to 100 MB, then click  
  Summarize into memory. Web lists ChatGPT, Claude, Gemini, and Meta AI exports.
  **Import chats:**（导入聊天记录）：点击 Add 并上传最大 100 MB 的 ZIP 导出文件，然后点击 "Summarize into memory"。Web 端列出 ChatGPT、Claude、Gemini 和 Meta AI 的导出。

Both paths ask the main agent to extract lasting facts and preferences about
the user and confirm what it saved. Uploading the ZIP alone does not save
memory. Importing does not copy app settings or restore the original
conversations as chats.

两种途径都是让主智能体提取关于用户的持久事实和偏好，并确认其保存了什么。仅上传 ZIP 并不会保存记忆。导入不会复制应用设置，也不会把原始对话恢复为聊天记录。

【评论】导入流程通过让用户把提示词复制到外部 AI 助手、再把其输出粘贴回来完成迁移，意味着记忆内容的来源可以是另一个模型生成的摘要。

## Forget saved information / 忘记已保存的信息

For questions about forgetting saved information or requests to remove it,
read `/opt/hatch/skills/forget/SKILL.md`.

关于忘记已保存信息的疑问或删除请求，请阅读 `/opt/hatch/skills/forget/SKILL.md`。

For cleanup requests during a live voice conversation, direct the user to
main chat.

在实时语音通话中遇到清理请求时，引导用户前往主对话。

## Related controls / 相关控制项

For data use, training choices, export, and Reset, read  
`~/docs/data-handling.md`. For retention and credentials, read  
`~/docs/privacy-and-credentials.md`.

关于数据使用、训练选择、导出和 Reset，请阅读 `~/docs/data-handling.md`。关于保留策略和凭据，请阅读 `~/docs/privacy-and-credentials.md`。
