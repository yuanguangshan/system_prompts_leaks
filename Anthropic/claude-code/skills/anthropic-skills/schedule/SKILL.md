<!-- BILINGUAL-EN-ZH -->
---
name: "schedule"
description: "Create or update a scheduled task that runs automatically. Use when the user says things like \"every day\", \"each morning\", \"remind me in an hour\", \"run this at noon\", or wants to reschedule an existing task."
---

First, decide whether the user wants to **create a new** scheduled task or **change an existing** one.

首先，判断用户是想**新建**一个定时任务，还是想**修改**一个已有的定时任务。

## Updating an existing task / 更新已有任务

If the user wants to reschedule, edit the prompt, or pause/resume a task that already exists, call the `update_scheduled_task` tool with its `taskId` — do **not** call `create_scheduled_task`. Use `list_scheduled_tasks` if you need to look up the ID. When this session is itself a scheduled run, the current task's ID is the `name` attribute in the `<scheduled-task name="…">` tag at the top of the conversation.

如果用户想重新安排时间、编辑提示词，或暂停/恢复一个已存在的任务，应携带其 `taskId` 调用 `update_scheduled_task` 工具——**不要**调用 `create_scheduled_task`。如果需要查找 ID，可使用 `list_scheduled_tasks`。当本会话本身就是一次定时运行时，当前任务的 ID 就是会话顶部 `<scheduled-task name="…">` 标签中的 `name` 属性。

## Creating a new task / 新建任务

You are distilling the current session into a reusable shortcut. Follow these steps:

你要把当前会话提炼为一个可复用的快捷方式。请按以下步骤执行：

### 1. Analyze the session / 1. 分析会话

Review the session history to identify the core task the user performed or requested. Distill it into a single, repeatable objective.

回顾会话历史，找出用户执行或请求的核心任务，并将其提炼为一个单一的、可重复的目标。

### 2. Draft a prompt / 2. 起草提示词

The prompt will be used for future autonomous runs — it must be entirely self-contained. Future runs will NOT have access to this session, so never reference "the current conversation," "the above," or any ephemeral context.

该提示词将用于未来的自主运行——它必须完全自包含。未来的运行将无法访问本会话，因此绝不要引用"当前对话"、"上文"或任何临时性上下文。

Include in the description:
- A clear objective statement (what to accomplish)
- Specific steps to execute
- Any relevant file paths, URLs, repositories, or tool names
- Expected output or success criteria
- Any constraints or preferences the user expressed

描述中应包含：
- A clear objective statement (what to accomplish)
  - 清晰的目标陈述（要完成什么）
- Specific steps to execute
  - 要执行的具体步骤
- Any relevant file paths, URLs, repositories, or tool names
  - 一切相关的文件路径、URL、代码仓库或工具名称
- Expected output or success criteria
  - 预期输出或成功标准
- Any constraints or preferences the user expressed
  - 用户表达过的任何约束或偏好

Write the description in second-person imperative ("Check the inbox…", "Run the test suite…"). Keep it concise but complete enough that another Claude session could execute it cold.

描述以第二人称祈使句书写（"Check the inbox…"、"Run the test suite…"）。保持简洁，但要足够完整，使另一个 Claude 会话能够在毫无上下文的情况下直接执行。

### 3. Choose a taskName / 3. 选择 taskName

Pick a short, descriptive name in kebab-case (e.g. "daily-inbox-summary", "weekly-dep-audit", "format-pr-description").

选择一个简短、具有描述性的 kebab-case 名称（例如 "daily-inbox-summary"、"weekly-dep-audit"、"format-pr-description"）。

### 4. Determine scheduling / 4. 确定调度方式

The `create_scheduled_task` tool description explains the options (`cronExpression` for recurring, `fireAt` for one-time, omit both for ad-hoc) and their formats. If the user didn't give a clear schedule, propose one and ask them to confirm before proceeding — don't rely on an approval prompt to catch a wrong guess, since task creation may be approved automatically in some permission modes.

`create_scheduled_task` 工具的描述说明了各可选项（周期任务用 `cronExpression`、一次性任务用 `fireAt`、临时任务则两者都省略）及其格式。如果用户没有给出明确的时间安排，应先提出一个方案并请其确认后再继续——不要指望靠审批提示来纠正错误的猜测，因为在某些权限模式下，任务创建可能会被自动批准。

Finally, call the `create_scheduled_task` tool.

最后，调用 `create_scheduled_task` 工具。

【评论】"不要指望靠审批提示来纠正错误的猜测"这一条款针对的是权限模式下审批环节可能被跳过的情况，属于对自动化操作风险的一种防御性设计。
