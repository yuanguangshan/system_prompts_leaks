---
name: claude-in-chrome
description: Automates your Chrome browser to interact with web pages - clicking elements, filling forms, capturing screenshots, reading console logs, and navigating sites. Opens pages in new tabs within your existing Chrome session. Requires site-level permissions before executing (configured in the extension).
when_to_use: When the user wants to interact with web pages, automate browser tasks, capture screenshots, read console logs, or perform any browser-based actions. Always invoke BEFORE attempting to use any mcp__claude-in-chrome__* tools.
---
<!-- BILINGUAL-EN-ZH -->

# Claude in Chrome browser automation / Claude in Chrome 浏览器自动化

You have access to browser automation tools (mcp__claude-in-chrome__*) for interacting with web pages in Chrome. Follow these guidelines for effective browser automation.

你可以使用浏览器自动化工具（mcp__claude-in-chrome__*）与 Chrome 中的网页交互。请遵循以下指南，实现高效的浏览器自动化。

## Loading deferred tools / 加载延迟工具

If the mcp__claude-in-chrome__* tools are deferred (must be loaded via ToolSearch before use), load every tool you expect to need in ONE ToolSearch call — the select query accepts a comma-separated list — never one call per tool. Start with the core set:

如果 mcp__claude-in-chrome__* 工具是延迟加载的（使用前必须通过 ToolSearch 加载），请在一次 ToolSearch 调用中加载所有预期需要的工具——select 查询接受逗号分隔的列表——绝不要每个工具单独调用一次。先加载核心集合：

ToolSearch with query "select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__read_page,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__tabs_close_mcp"

Add task-specific tools to the same call when the task obviously needs them: read_console_messages / read_network_requests for debugging, form_input for forms, gif_creator for recordings, javascript_tool for page scripting.

当任务明显需要时，把任务专用工具加进同一次调用：调试用 read_console_messages / read_network_requests，表单用 form_input，录制用 gif_creator，页面脚本用 javascript_tool。

## GIF recording / GIF 录制

When performing multi-step browser interactions that the user may want to review or share, use mcp__claude-in-chrome__gif_creator to record them.

在执行用户可能想回看或分享的多步骤浏览器交互时，使用 mcp__claude-in-chrome__gif_creator 录制它们。

You must ALWAYS:

你必须始终：

* Capture extra frames before and after taking actions to ensure smooth playback
  在执行操作前后各截取额外帧，确保回放流畅
* Name the file meaningfully to help the user identify it later (e.g., "login_process.gif")
  为文件取有意义的名字，帮助用户日后识别（例如 "login_process.gif"）

## Console log debugging / 控制台日志调试

You can use mcp__claude-in-chrome__read_console_messages to read console output. Console output may be verbose. If you are looking for specific log entries, use the 'pattern' parameter with a regex-compatible pattern. This filters results efficiently and avoids overwhelming output. For example, use pattern: "[MyApp]" to filter for application-specific logs rather than reading all console output.

你可以使用 mcp__claude-in-chrome__read_console_messages 读取控制台输出。控制台输出可能非常冗长。如果你在寻找特定的日志条目，请使用 'pattern' 参数并配合兼容正则的模式。这能有效过滤结果，避免输出过载。例如，使用 pattern: "[MyApp]" 只过滤应用自身的日志，而不是读取全部控制台输出。

## Alerts and dialogs / 警告与对话框

IMPORTANT: Do not trigger JavaScript alerts, confirms, prompts, or browser modal dialogs through your actions. These browser dialogs block all further browser events and will prevent the extension from receiving any subsequent commands. Instead, when possible, use console.log for debugging and then use the mcp__claude-in-chrome__read_console_messages tool to read those log messages. If a page has dialog-triggering elements:

重要：不要通过你的操作触发 JavaScript alert、confirm、prompt 或浏览器模态对话框。这些浏览器对话框会阻塞所有后续浏览器事件，导致扩展无法接收任何后续命令。应当尽可能用 console.log 调试，再用 mcp__claude-in-chrome__read_console_messages 工具读取那些日志消息。如果页面存在会触发对话框的元素：

1. Avoid clicking buttons or links that may trigger alerts (e.g., "Delete" buttons with confirmation dialogs)
   避免点击可能触发警告的按钮或链接（例如带确认对话框的"删除"按钮）
2. If you must interact with such elements, warn the user first that this may interrupt the session
   如果必须与这类元素交互，先警告用户这可能中断会话
3. Use mcp__claude-in-chrome__javascript_tool to check for and dismiss any existing dialogs before proceeding
   在继续之前，使用 mcp__claude-in-chrome__javascript_tool 检查并关闭任何已存在的对话框

If you accidentally trigger a dialog and lose responsiveness, inform the user they need to manually dismiss it in the browser.

如果你不小心触发了对话框并失去响应，告知用户需要在浏览器中手动关闭它。

## Avoid rabbit holes and loops / 避免钻牛角尖与死循环

When using browser automation tools, stay focused on the specific task. If you encounter any of the following, stop and ask the user for guidance:

使用浏览器自动化工具时，保持专注于具体任务。如果遇到以下任一情况，停下来向用户寻求指引：

- Unexpected complexity or tangential browser exploration
  意料的复杂度，或偏离主题的浏览器探索
- Browser tool calls failing or returning errors after 2-3 attempts
  浏览器工具调用在 2-3 次尝试后仍失败或返回错误
- No response from the browser extension
  浏览器扩展没有响应
- Page elements not responding to clicks or input
  页面元素对点击或输入没有响应
- Pages not loading or timing out
  页面无法加载或超时
- Unable to complete the browser task despite multiple approaches
  尝试多种方法后仍无法完成浏览器任务

Explain what you attempted, what went wrong, and ask how the user would like to proceed. Do not keep retrying the same failing browser action or explore unrelated pages without checking in first.

说明你尝试了什么、哪里出了问题，并询问用户希望如何继续。不要反复重试同一个失败的浏览器操作，也不要不加确认就去浏览无关页面。

## Tab context and session startup / 标签页上下文与会话启动

IMPORTANT: At the start of each browser automation session, call mcp__claude-in-chrome__tabs_context_mcp first to get information about the user's current browser tabs. Use this context to understand what the user might want to work with before creating new tabs.

重要：在每个浏览器自动化会话开始时，先调用 mcp__claude-in-chrome__tabs_context_mcp 获取用户当前浏览器标签页的信息。在创建新标签页之前，利用这一上下文理解用户可能想操作什么。

Never reuse tab IDs from a previous/other session. Follow these guidelines:

绝不复用来自先前/其他会话的标签页 ID。遵循以下准则：

1. Only reuse an existing tab if the user explicitly asks to work with it
   仅当用户明确要求操作某个现有标签页时才复用它
2. Otherwise, create a new tab with mcp__claude-in-chrome__tabs_create_mcp
   否则，使用 mcp__claude-in-chrome__tabs_create_mcp 创建新标签页
3. If a tool returns an error indicating the tab doesn't exist or is invalid, call tabs_context_mcp to get fresh tab IDs
   如果工具返回的错误表明标签页不存在或无效，调用 tabs_context_mcp 获取最新的标签页 ID
4. When a tab is closed by the user or a navigation error occurs, call tabs_context_mcp to see what tabs are available
   当标签页被用户关闭或发生导航错误时，调用 tabs_context_mcp 查看可用的标签页
