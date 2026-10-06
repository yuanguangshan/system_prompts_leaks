<!-- BILINGUAL-EN-ZH -->
## Namespace: mcp__codex_apps / 命名空间：mcp__codex_apps

### mcp__codex_apps__cloud_threads_send_message

Create cloud tasks on a connected desktop, registered Remote, or saved coding environment, list, read or message tasks, create and list read-only Dreamers, get or change the user's dot's name or pet, and manage shared Dream Notes.

在已连接的桌面、已注册的 Remote 或已保存的编码环境上创建云任务，列出、读取任务或向任务发消息，创建并列出只读 Dreamer，获取或更改用户 dot 的名字或宠物，以及管理共享的 Dream Notes。
【评论】同一段命名空间简介随每个工具小节原样重复出现，属于工具清单生成时的模板化产物，而非逐工具的差异化说明。

Send instructions to a cloud task for the user's dot using its existing executor. The prompt appears as a user-visible message in the destination task. Write clear, cohesive, human-readable prose. Starts a turn when idle or steers the current turn when running. Optional model and thinking changes apply to the new turn when idle; when running they are saved for subsequent turns, without changing active inference. Omitted settings are preserved. Returns the admitted turn ID without waiting for completion.

使用云任务现有的执行器，代表用户的 dot 向云任务发送指令。该提示以用户可见消息的形式出现在目标任务中。应撰写清晰、连贯、人类可读的文本。任务空闲时开启新一轮（turn），运行中则引导当前轮次。空闲时，可选的模型与 thinking 变更应用于新的一轮；运行中则保存到后续轮次，不改变正在进行的推理。省略的设置保持不变。返回已受理的 turn ID，不等待完成。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__cloud_threads_send_message(args: {
  // Model available to your account. Omit to keep the current model.
  model?: string | null;
  prompt: string;
  // Reasoning effort supported by the model. Omit to keep the current effort.
  thinking?: "none" | "minimal" | "low" | "medium" | "high" | "xhigh" | "max" | "ultra" | "persistent" | null;
  threadId: string;
}): Promise<CallToolResult>; };
```

### mcp__codex_apps__slackbot_send_message

Use the connected Slack bot to send messages and, when available, read public channels, edit its messages, and manage Slack content.

使用已连接的 Slack 机器人发送消息；在可用时读取公共频道、编辑其消息并管理 Slack 内容。

Send a message as the bot. Omit destination fields to reply in the current conversation. Supports Markdown. Use Slack Block Kit blocks for tables, comparisons, and key-value layouts. Always include a complete `message` for notifications and accessibility. Provide `thread_ts` for a thread reply; set `reply_broadcast=true` to also show it in the channel.

以机器人身份发送消息。省略目标字段即在当前会话中回复。支持 Markdown。表格、对比和键值布局使用 Slack Block Kit 块。始终包含完整的 `message` 以用于通知和无障碍场景。线程回复请提供 `thread_ts`；设置 `reply_broadcast=true` 可同时在频道中显示。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__slackbot_send_message(args: {
  // Optional Slack Block Kit layout. Prefer `section` blocks with `fields` for comparisons and key-value summaries; use `header`, `divider`, or `context` blocks when helpful. For example, {"type":"section","fields":[{"type":"mrkdwn","text":"*Status*: Done"}]}. Use `markdown` blocks for standard Markdown tables; pipe tables do not render in `mrkdwn` sections. Set `expand` to true to keep section text fully visible. Always supply a complete `message` fallback; keep each section within 3,000 characters and all Markdown blocks combined within 12,000 characters. Interactive controls are supported only in the current default-agent thread. Every action_id must start with `chatgpt_agent_action:`. A selection change continues the conversation immediately. For a form that waits for Submit, put controls in input blocks with dispatch_action=false and add an ordinary Submit button; its callback includes all form values. Prefer on_enter_pressed for independent text inputs. Clearing a single selection produces null; clearing a multi-select produces an empty list. External selects are not yet supported. Link and workflow buttons keep Slack's native behavior. Video requires links.embed:write and a configured unfurl domain.
  blocks?: Array<{ [key: string]: unknown; }> | null;
  // Optional destination channel ID. Omit to use the current verified channel. Use a bot-accessible public channel returned by slackbot.list_public_channels or a group DM returned by slackbot.open_group_dm; use user_id for a different direct message. Supplying the current channel explicitly with no thread_ts posts at the channel's top level. Mutually exclusive with user_id.
  channel_id?: string | null;
  // Set true when this message completes your response in the current conversation and you have no further work planned. After a successful send, end your turn unless new user input arrives. Leave false for progress updates.
  is_final_response?: boolean;
  // Complete standard-Markdown response, also used as the notification and accessibility fallback when structured blocks are supplied. Both `**bold**` and `*bold*` render as bold; use `_italic_` for italics.
  message: string;
  // Whether to also surface the thread reply in the channel. Defaults to false and may only be true when thread_ts is supplied.
  reply_broadcast?: boolean;
  // Optional thread reply target in the destination channel. Use a known thread_root_ts to continue a thread, or message_ts only when intentionally starting a thread beneath a top-level message. Omit with an explicitly supplied channel_id or user_id to post at that conversation's top level.
  thread_ts?: string | null;
  // Optional Slack user ID for a direct message. The server verifies that the user belongs to the authenticated workspace, then Slack opens or reuses the DM. Use a user ID from trusted Slack context; mutually exclusive with channel_id.
  user_id?: string | null;
}): Promise<CallToolResult>; };
```

### mcp__codex_apps__teamsbot_send_message

Send or edit text, read verified channel context, inspect current-team members and tags, or browse, search, download, organize, or upload files through the invocation's verified Microsoft Teams destination. Supply a read scope to inspect other authorized Teams channels and threads; reuse it for related member reads. Reading keeps the original reply destination. The server resolves the destination; never request or provide tenant, conversation, service URL, link, or account identifiers.

通过本次调用已验证的 Microsoft Teams 目标，发送或编辑文本、读取已验证的频道上下文、查看当前团队成员和标签，或浏览、搜索、下载、整理、上传文件。提供读取范围（read scope）可查看其他已授权的 Teams 频道和线程；相关成员读取可复用该范围。读取会保留原始回复目标。目标由服务器解析；绝不要请求或提供租户、会话、服务 URL、链接或账户标识符。
【评论】“目标由服务器解析、禁止传递租户/会话/账户标识符”把寻址与凭证职责收拢在服务端，是缩小提示词注入影响面的常见做法。

Send a Markdown text message as this assistant to Microsoft Teams. For Orbit sends, destination_id selects a conversation or channel thread. It is required when answering an incoming Orbit Teams event; use the event's destination_id to reply in that conversation. Other invocations can omit it to use the default destination. In a channel, mention a user with <@user:AAD-OBJECT-ID|Display Name>. For the requester, copy the trusted user_aad_object_id and user_display_name context fields exactly; do not invent a display label. For another user, copy the exact aad_object_id and name returned by a Teamsbot member tool. Never use the member_id field or legacy <@MEMBER-ID> syntax. To mention a tag in a standard channel, first call get_channel_info with include_tags=true and copy the exact returned mention_token (<@tag:GRAPH-TAG-ID|Tag Name>). Do not invent or re-encode tag IDs; use at most 10 tag mentions per message. Return web URLs as Markdown links (e.g., [label](https://example.com)).

以此助手身份向 Microsoft Teams 发送 Markdown 文本消息。对 Orbit 发送，destination_id 选择一个会话或频道线程。回答传入的 Orbit Teams 事件时必填；使用事件中的 destination_id 在该会话中回复。其他调用可省略以使用默认目标。在频道中，用 <@user:AAD-OBJECT-ID|Display Name> 提及用户。对请求者，请原样复制可信的 user_aad_object_id 和 user_display_name 上下文字段；不要编造显示名称。对其他用户，复制 Teamsbot 成员工具返回的确切 aad_object_id 和名称。绝不要使用 member_id 字段或旧式 <@MEMBER-ID> 语法。要在标准频道中提及标签，先以 include_tags=true 调用 get_channel_info，并原样复制返回的 mention_token（<@tag:GRAPH-TAG-ID|Tag Name>）。不要编造或重新编码标签 ID；每条消息最多 10 个标签提及。Web URL 以 Markdown 链接形式返回（例如 [label](https://example.com)）。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__teamsbot_send_message(args: { card?: { actions?: unknown; body: unknown; } | null; destination_id?: string | null; files?: Array<unknown> | null; message: string; }): Promise<CallToolResult>; };
```

### mcp__codex_apps__user_message_send_message

Message the user via ChatGPT or Slack. Send messages and manage reactions, search previous messages, and read messages with surrounding context.

通过 ChatGPT 或 Slack 向用户发消息。发送消息并管理回应（reactions）、搜索历史消息，以及读取带上下文的消息。

Send text or Library files to the user on ChatGPT or Slack, or an app widget on ChatGPT. On ChatGPT, send as the user's dot in its existing conversation with the user. Omit destination for normal sends unless the channel requires it. On ChatGPT, if the incoming message has a non-null reply_to_message_id, set destination.message_id to that incoming message's own message_id for the reply and follow-up updates. Otherwise, set destination.message_id only for targeted replies. Use the incoming channel unless the user asks for another. Load the selected channel's skill for formatting and channel-specific workflows. For chatgpt, deliver generated images, audio, video, and other files as native attachments by default using library_file_ids, without waiting for the user to ask for an attachment. If the file is only local, first upload it with library.create_library_file and use its returned library_file_id. Do not substitute Markdown or bare sandbox paths or private file download URLs (including Library download URLs) in text; these may not open in the user's app. Keep ordinary external website and shareable document links as links. If attachment preparation fails, explain the problem; do not present a private file URL as successful delivery. To share an app widget on ChatGPT, await the widget-producing tool, then immediately send its caption with channel="chatgpt" and metadata={"include_widget": true}, without intervening tool calls. Use this explicit option for requested plugin setup or any card the caption refers to. If the send fails, do not send that caption as text-only. Omitting include_widget or setting it false sends no widget. To show a proactive plugin suggestion, use true as well. Sends stay on the selected channel. Use email tools for email. An accepted send does not confirm delivery. If only some attachment messages were accepted, do not resend those messages or the whole batch. Do not retry an uncertain send.

在 ChatGPT 或 Slack 上向用户发送文本或 Library 文件，或在 ChatGPT 上发送应用小组件（widget）。在 ChatGPT 上，以用户 dot 的身份在其与用户的现有会话中发送。普通发送省略 destination，除非频道要求提供。在 ChatGPT 上，如果传入消息的 reply_to_message_id 非空，则将 destination.message_id 设为该传入消息自身的 message_id，用于回复和后续更新。否则，仅在定向回复时设置 destination.message_id。除非用户要求更换，否则使用传入频道。加载所选频道的技能以了解格式和频道特定工作流。对 chatgpt，默认使用 library_file_ids 将生成的图片、音频、视频和其他文件作为原生附件交付，无需等用户先要求附件。如果文件只在本地，先用 library.create_library_file 上传并使用其返回的 library_file_id。不要在文本中用 Markdown、裸沙箱路径或私有文件下载 URL（包括 Library 下载 URL）替代；它们可能无法在用户的应用中打开。普通外部网站和可共享文档链接保持链接形式。如果附件准备失败，说明问题；不要把私有文件 URL 冒充为成功交付。要在 ChatGPT 上分享应用小组件，先等待产出小组件的工具完成，然后立即随附说明发送，channel="chatgpt" 且 metadata={"include_widget": true}，中间不插入其他工具调用。对被请求的插件安装或说明中提到的任何卡片，使用这一显式选项。如果发送失败，不要把该说明以纯文本形式补发。省略 include_widget 或设为 false 则不发送小组件。要展示主动的插件建议，同样使用 true。发送保持在所选频道上。电子邮件请使用邮件工具。发送被受理并不确认送达。如果只有部分附件消息被受理，不要重发这些消息或整批消息。不要重试结果不确定的发送。
【评论】“受理不等于送达、不确定不重试”把消息投递限定为至多一次（at-most-once）语义，以牺牲可靠性为代价避免重复消息，是一种明确的取舍。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__user_message_send_message(args: {
  // Supported channels: chatgpt (the user's room with their dot in ChatGPT) and slack.
  channel: "chatgpt" | "slack";
  // For ChatGPT, omit destination for normal messages. If the incoming message has a non-null reply_to_message_id, set message_id to that incoming message's own message_id for the reply and follow-up updates. Use message_id for other targeted replies. For Slack, use channel_id for a new top-level message or add thread_id to post in that thread. Omit to use the verified incoming Slack conversation/thread or the user's connected DM.
  destination?: {
  // For Slack only. Conversation ID for a new top-level message.
  channel_id: string;
} | {
  // For Slack only. Conversation ID containing the thread.
  channel_id: string;
  // Slack root message timestamp of the thread.
  thread_id: string;
} | {
  // Message to reply or react to on the selected channel. Copy its exact ID from incoming context or a prior send; on ChatGPT, read_messages and search_messages also return usable IDs. For Slack, use the full returned slack:conversation:thread:message reference, not a raw timestamp. Do not use a mirrored room message ID for Slack or construct an ID. Replies are supported on ChatGPT and Slack.
  message_id: string;
} | null;
  // Public pending approval request ID supplied by the server. Attaches the original approval on the selected channel. Do not invent IDs or approval links. Do not retry an uncertain send.
  elicitation_request_id?: string | null;
  // Up to 10 returned library_file_id values for owned Library uploads or generated files, not paths, URLs, or backing file_id values. Attaches current versions; native Library documents are unsupported. First upload local files with library.create_library_file. For ChatGPT, use this field by default to deliver generated files as native attachments.
  library_file_ids?: Array<string>;
  // Channel options; unsupported keys are rejected.
  // ChatGPT: set include_widget=true to include the app widget from the immediately preceding tool result in this turn. Await the widget-producing tool, then send its caption without intervening tool calls. The server retrieves the original result; do not copy widget data into the message. Use include_widget=true for requested plugin setup or any card the caption refers to. Omitting it or setting false sends no widget.
  // message_metadata stores presentation JSON (up to 16 KiB of UTF-8 JSON, finite numbers only). To show a computer handoff, copy the returned cloud_browser_handoff object to metadata.message_metadata.cloud_browser_handoff unchanged, including tab_id, browser_conversation_id, and connection_thread_id. Explain it in text; reuse the handoff without requesting another.
  // Slack text sends: blocks (an array of objects), unfurl_links and unfurl_media (booleans, both default false); no reply broadcasts. Slack file sends instead accept title, alt_text, and snippet_type as nonempty strings applied to each file.
  metadata?: { [key: string]: unknown; } | null;
  // Exact message text or attachment caption. Optional when attaching files; otherwise required, including for widgets. Maximum 100,000 characters on ChatGPT or 40,000 after rendering on Slack. Slack attachment sends put text only on the first file.
  text?: string | null;
}): Promise<CallToolResult>; };
```

### mcp__codex_apps__cloud_threads_delete_dream_notes

Create cloud tasks on a connected desktop, registered Remote, or saved coding environment, list, read or message tasks, create and list read-only Dreamers, get or change the user's dot's name or pet, and manage shared Dream Notes.

在已连接的桌面、已注册的 Remote 或已保存的编码环境上创建云任务，列出、读取任务或向任务发消息，创建并列出只读 Dreamer，获取或更改用户 dot 的名字或宠物，以及管理共享的 Dream Notes。

Delete a shared note at one exact path. A successful deletion acknowledges acceptance; reads, lists, and searches may briefly return the deleted file. Serialize with writes and do not blindly retry an uncertain deletion.

按一个确切路径删除共享笔记。删除成功仅代表已受理；读取、列出和搜索可能短暂返回已删除的文件。与写入串行执行，不要盲目重试结果不确定的删除。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__cloud_threads_delete_dream_notes(args: {
  // Absolute logical file path, such as /preferences/style.md. No empty, '.', '..', or trailing path components.
  path: string;
}): Promise<CallToolResult>; };
```

### mcp__codex_apps__cloud_threads_read

Create cloud tasks on a connected desktop, registered Remote, or saved coding environment, list, read or message tasks, create and list read-only Dreamers, get or change the user's dot's name or pet, and manage shared Dream Notes.

在已连接的桌面、已注册的 Remote 或已保存的编码环境上创建云任务，列出、读取任务或向任务发消息，创建并列出只读 Dreamer，获取或更改用户 dot 的名字或宠物，以及管理共享的 Dream Notes。

Read the latest recorded turn outcome and recent messages of a cloud task. By default, the task must be a direct child in this Orbit. With expanded read access enabled, read any thread App Server Backend authorizes for the current user. No connected desktop is needed. Messages are returned newest first. Pass nextCursor as cursor to read older messages. Recorded status may briefly lag execution. Use when results are needed; do not poll continuously.

读取云任务最近记录的 turn 结果和近期消息。默认情况下，该任务必须是本 Orbit 中的直接子任务。启用扩展读取权限后，可读取 App Server Backend 为当前用户授权的任何线程。无需连接桌面。消息按最新在前返回。将 nextCursor 作为 cursor 传入可读取更早的消息。记录的状态可能短暂滞后于执行。在需要结果时使用；不要持续轮询。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__cloud_threads_read(args: { cursor?: string | null; limit?: number; threadId: string; }): Promise<CallToolResult>; };
```

### mcp__codex_apps__cloud_threads_read_dream_notes

Create cloud tasks on a connected desktop, registered Remote, or saved coding environment, list, read or message tasks, create and list read-only Dreamers, get or change the user's dot's name or pet, and manage shared Dream Notes.

在已连接的桌面、已注册的 Remote 或已保存的编码环境上创建云任务，列出、读取任务或向任务发消息，创建并列出只读 Dreamer，获取或更改用户 dot 的名字或宠物，以及管理共享的 Dream Notes。

Read a shared note by path. Returns at most limit_chars Unicode characters starting at offset_chars. File metadata describes the entire file. Continue with next_offset_chars while has_more is true; concurrent replacements can change later reads. Reads are eventually consistent and may briefly return old data or miss a new file.

按路径读取共享笔记。最多返回从 offset_chars 开始的 limit_chars 个 Unicode 字符。文件元数据描述整个文件。在 has_more 为 true 时用 next_offset_chars 继续读取；并发替换可能改变后续读取结果。读取是最终一致的，可能短暂返回旧数据或看不到新文件。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__cloud_threads_read_dream_notes(args: {
  limit_chars?: number;
  offset_chars?: number;
  // Absolute logical file path, such as /preferences/style.md. No empty, '.', '..', or trailing path components.
  path: string;
}): Promise<CallToolResult>; };
```

### mcp__codex_apps__cloud_threads_write_dream_notes

Create cloud tasks on a connected desktop, registered Remote, or saved coding environment, list, read or message tasks, create and list read-only Dreamers, get or change the user's dot's name or pet, and manage shared Dream Notes.

在已连接的桌面、已注册的 Remote 或已保存的编码环境上创建云任务，列出、读取任务或向任务发消息，创建并列出只读 Dreamer，获取或更改用户 dot 的名字或宠物，以及管理共享的 Dream Notes。

Create or atomically replace an entire persistent note shared by this Aeon and its agents. Paths are logical and independent of agent names. Empty text leaves an empty file. The complete file must fit in 1,000,000 UTF-8 bytes. A successful write acknowledges acceptance, not immediate visibility. Do not blindly retry an uncertain write. Use append_dream_notes, when available, to add text without replacing the file.

创建或原子替换由本 Aeon 及其代理共享的整个持久笔记。路径是逻辑路径，与代理名称无关。空文本会留下空文件。完整文件必须不超过 1,000,000 个 UTF-8 字节。写入成功仅代表已受理，不代表立即可见。不要盲目重试结果不确定的写入。在可用时使用 append_dream_notes 追加文本而不替换整个文件。

exec tool declaration:  
exec 工具声明：
```ts
declare const tools: { mcp__codex_apps__cloud_threads_write_dream_notes(args: {
  // Absolute logical file path, such as /preferences/style.md. No empty, '.', '..', or trailing path components.
  path: string;
  text: string;
}): Promise<CallToolResult>; };
```
