---
name: remote-environments
description: "Discover available task environments and create or continue tasks when the user names an environment or computer, or refers to files or apps specifically on their computer. Excludes software engineering; use orbit:software-engineering for that."
---
<!-- BILINGUAL-EN-ZH -->

# Remote environments / 远程环境

Do not use this skill for software engineering. For codebase investigation, coding, debugging, tests, product QA, or PR/CI work, use `$orbit:software-engineering` instead, including its environment-selection guidance.

不要将本技能用于软件工程。代码库调查、编码、调试、测试、产品 QA 或 PR/CI 工作请改用 `$orbit:software-engineering`，包括其中的环境选择指导。

Use this skill in the parent dot for other tasks. If you are already the assigned task, work in your selected environment.

在父 dot 中对其他任务使用本技能。如果你本身就是被指派的任务，则在你被选定的环境中工作。

For follow-up work in a task whose environment the user already selected, go straight to **Follow through**. Keep its `threadId` and selected environment.

对于环境已由用户选定的任务的后续工作，直接前往**跟进**（Follow through）部分。保留其 `threadId` 和选定的环境。

This skill covers both collections returned by `cloud_threads.list_environments`: existing user computers and registered remotes in `environments`, and personal and workspace-shared saved coding configurations in `codingEnvironments`. The dot cloud computer is separate and is not included in that list.

本技能覆盖 `cloud_threads.list_environments` 返回的两组集合：`environments` 中的用户现有电脑和已注册的远程环境，以及 `codingEnvironments` 中的个人和工作区共享的已保存编码配置。dot 云电脑是独立的，不包含在该列表中。

## Decide where to work / 决定在哪里工作

Use the dot cloud computer for all computer work unless:

所有电脑工作都使用 dot 云电脑，除非：

- The user asks to use a specific environment or computer. Use that environment or computer.
  用户要求使用特定的环境或电脑。使用该环境或电脑。
- The user refers to files or apps specifically on their computer, such as "files on my computer" or "the app open on my desktop." Use the user's computer.
  用户提到专门位于其电脑上的文件或应用，例如"我电脑上的文件"或"我桌面上打开的应用"。使用用户的电脑。
- The browser tool's documentation calls for a local browser fallback. Use the user's connected computer, preserving their explicit browser and environment choices.
  浏览器工具的文档要求回退到本地浏览器。使用用户已连接的电脑，并保留其明确的浏览器和环境选择。

A request for a cloud task alone does not select another computer. The dot cloud computer remains the default.

仅请求一个云任务并不会选中另一台电脑。dot 云电脑仍是默认选择。

## Discover and choose / 发现与选择

- Call `cloud_threads.list_environments`. Use the returned names, IDs, connection status, and repository information; do not invent IDs or keep a fixed list of environments.
  调用 `cloud_threads.list_environments`。使用返回的名称、ID、连接状态和仓库信息；不要虚构 ID，也不要维护一份固定的环境列表。
- Read both collections. Follow `nextCursor` by passing it unchanged as `cursor` when needed to find the requested configuration or resolve ambiguity. To list all choices, continue until it is null, including through empty pages. Computer entries repeat across pages; list each computer once.
  读取两组集合。当需要找到所请求的配置或消解歧义时，把 `nextCursor` 原样作为 `cursor` 传递以继续翻页。要列出全部选项，持续翻页直到它为 null，包括经过空页。电脑条目会在各页之间重复；每台电脑只列出一次。
- Use the target selected under **Decide where to work**, preserving the user's instructions. "Your computer" means the dot cloud computer; "my computer," "my Mac," and "my laptop" mean the user's computer. A named remote refers to that remote. If multiple entries match and context does not distinguish them, ask which one.
  使用**决定在哪里工作**部分选定的目标，并保留用户的指令。"Your computer"指 dot 云电脑；"my computer""my Mac"和"my laptop"指用户自己的电脑。被点名的远程环境指那个远程环境。如果多个条目匹配而上下文无法区分，询问是哪一个。
- When the user selects a saved coding environment, its configuration creates a fresh workspace using its repositories and setup. References to files or apps specifically on the user's computer require that computer.
  当用户选择一个已保存的编码环境时，其配置会使用其中的仓库和设置创建一个全新的工作区。对用户电脑上文件或应用的引用则需要那台电脑本身。
- If the requested environment is missing, offline, or unavailable, explain the specific blocker and continue work that does not depend on it. Do not silently switch environments. An empty coding catalog means no configurations were returned; it does not establish that none exist.
  如果所请求的环境缺失、离线或不可用，说明具体的阻塞原因，并继续不依赖它的工作。不要静默切换环境。空的编码目录意味着没有返回任何配置；并不能证明不存在任何配置。

## Create the task / 创建任务

Use a short title and a self-contained prompt with the requested outcome, relevant context, repository or directory, constraints, and what to verify. The child does not inherit this conversation.

使用简短的标题和自包含的提示词，包含所请求的结果、相关上下文、仓库或目录、约束条件以及要验证的内容。子任务不会继承本对话。

【评论】"子任务不继承本对话"是一种上下文隔离设计：子任务只拿到明列的提示词内容，可防止父对话中的私密上下文被带入新的执行环境。

| Target | Arguments to `cloud_threads.create` |
| --- | --- |
| Existing user computer or registered remote | Pass its non-null `environmentId`. It must have `status: "connected"`; `attached: false` is allowed. |
| Fresh workspace from a saved coding configuration | Pass the configuration's `id` as `environmentConfigId`, not its `version_id`. No attached or connected user computer is required. |
| User's computer via the attached-computer default | Use only after the rules above select the user's computer. Omit both selectors and `cwd`; exactly one user computer must be attached and it must be connected. |

| 目标 | 传给 `cloud_threads.create` 的参数 |
| --- | --- |
| 用户现有电脑或已注册的远程环境 | 传递其非空的 `environmentId`。其状态必须为 `status: "connected"`；允许 `attached: false`。 |
| 由已保存编码配置创建的全新工作区 | 把配置的 `id` 作为 `environmentConfigId` 传入，而不是其 `version_id`。不需要任何已附加或已连接的用户电脑。 |
| 通过已附加电脑默认方式使用用户的电脑 | 仅在上述规则选中用户电脑之后使用。省略两个选择器和 `cwd`；必须恰好附加了一台用户电脑且其处于已连接状态。 |

- Check only the selected environment's prerequisites. A disconnected or unattached user computer does not block a saved coding environment.
  只检查所选环境的前置条件。一台未连接或未附加的用户电脑不会阻塞已保存的编码环境。
- When creating an authorized task, call `cloud_threads.create` with the requested target's selector. If blocked, explain the applicable instruction or actual tool error for that path. Do not infer a launch restriction from another computer's status or invent a separate attachment or approval requirement.
  创建已获授权的任务时，用所请求目标的选择器调用 `cloud_threads.create`。如果被阻塞，解释适用于该路径的指令或实际的工具错误。不要从另一台电脑的状态推断启动限制，也不要虚构额外的附加或审批要求。
- Pass at most one environment selector.
  最多传递一个环境选择器。
- With `environmentId`, optionally pass `cwd` as an absolute path on that computer. Otherwise use its default directory. Changing `cwd` does not expand write access.
  使用 `environmentId` 时，可选地以该电脑上的绝对路径传递 `cwd`。否则使用其默认目录。更改 `cwd` 不会扩大写权限。
- With `environmentConfigId`, omit `cwd`. Repository refs and setup come from the saved configuration.
  使用 `environmentConfigId` 时，省略 `cwd`。仓库引用和设置来自已保存的配置。
- Discovery does not attach or connect a computer. Explicit selection does not require changing the parent's attachments. Respect any access or feature-availability error from creation.
  发现操作不会附加或连接电脑。明确选择并不要求更改父级的附加设备。对于创建过程中出现的任何访问权限或功能可用性错误都要如实对待。
- Connection status does not describe browser or app capabilities. Have the child check the tools needed for the task before promising it can operate a browser or application.
  连接状态并不描述浏览器或应用的能力。在承诺子任务能操作浏览器或应用之前，先让它检查任务所需的工具。

## Follow through / 跟进

- Successful creation already attaches the task to the conversation; do not call `cloud_threads.attach` again. Share a supported task link when available without referring to a task card.
  创建成功即已把任务附加到对话；不要再次调用 `cloud_threads.attach`。在可用时分享受支持的任务链接，而不要提及任务卡片。
- Creation confirms the task was accepted; setup or execution may still be running. Use `cloud_threads.read` when results are needed and the child's completion notification to follow progress. Do not poll continuously.
  创建成功只确认任务被接受；设置或执行可能仍在进行。需要结果时使用 `cloud_threads.read`，并通过子任务的完成通知来跟踪进度。不要持续轮询。
- Continue related work with `cloud_threads.send_message` on the same `threadId`; it keeps the selected environment.
  用 `cloud_threads.send_message` 在同一个 `threadId` 上继续相关工作；它会保留选定的环境。
- Do not automatically repeat an uncertain create. If the error includes a `threadId`, read that task before sending a follow-up.
  不要自动重复一次结果不确定的创建。如果错误信息中包含 `threadId`，在发送后续消息前先读取该任务。
