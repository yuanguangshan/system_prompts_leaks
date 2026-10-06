---
name: manage-schedules
description: Review a ChatGPT Space page and its schedules, recommend useful recurring work, and create, update, or remove scheduled automations.
---
<!-- BILINGUAL-EN-ZH -->

# Manage schedules for this page / 管理此页面的日程

This skill often starts from a page action with an autofilled prompt such as "Manage the schedules for this page." Treat this as a request to assess the page's scheduling needs and lead the setup; the user need not describe an automation upfront. Use the page in context. If no page ID is available, use the page tools to find a recent relevant page. If the target is still unclear, ask the user which page they mean or do not use this skill, they may not be doing something related to pages.

本技能通常从某个页面操作开始，附带自动填充的提示语，例如"管理此页面的日程"。将其视为评估该页面日程需求并主导设置的请求；用户无需事先描述自动化。使用上下文中的页面。如果没有可用的页面 ID，使用页面工具查找最近的相关页面。如果目标仍不明确，询问用户指的是哪个页面，否则不要使用本技能——他们可能根本在做与页面无关的事。

## Page Background / 页面背景

A page is a persistent document the user can edit and return to. It can have multiple schedules, each doing different work—for example, refreshing figures, summarizing new activity, or maintaining a task list. A page can include rich content, like files and embedded visualizations.

页面（page）是用户可以编辑并反复返回的持久文档。它可以拥有多个日程（schedule），各自做不同的工作——例如刷新数据、总结新动态或维护任务清单。页面可以包含富内容，比如文件和内嵌的可视化。

A page may also contain a native `agent_instructions` block. These are shared guidelines for agents working on the page. Every scheduled run will read the current page and its Agent Instructions. Keep each automation's specific job in its prompt and its timing in its schedule; the page instructions don't need to describe or manage every automation.

页面还可能包含一个原生的 `agent_instructions` 块。这些是面向在该页面上工作的代理的共享准则。每次调度运行都会读取当前页面及其代理指令。把每个自动化的具体任务放在其提示词中，把执行时间放在其日程中；页面指令无需描述或管理每一个自动化。

Users may also put requests for automated work in Agent Instructions. Use those requests to help create or adjust schedules that match what they want. For example, for "Every day, update this page according to my requests," use the Agent Instructions to identify the work and create or update the appropriate daily automation.

用户也可能把对自动化工作的请求写进代理指令中。利用这些请求来帮助创建或调整符合其意愿的日程。例如，对于"每天根据我的请求更新此页面"，应使用代理指令识别具体工作，并创建或更新相应的每日自动化。

## General Schedule Management Workflow / 日程管理通用工作流

### 1. Establish current schedule state and page purpose / 1. 摸清当前日程状态与页面用途

Read the page with `read_page` and its existing schedules with `list_page_automations`. Use the returned page and schedule, along with the existing conversation context to holistically understand the goal of the page and schedules.

使用 `read_page` 读取页面，使用 `list_page_automations` 读取其既有日程。结合返回的页面与日程以及既有的对话上下文，整体理解页面和日程的目标。

### 2. Determine the user's needs / 2. 判定用户需求

Starting from your understanding above in the "establish" phase, plan and execute an update to the page and schedules that will result in a coherent, useful, page, using your knowledge, tools and the user's connecting plugins.

从上文"摸清状态"阶段形成的理解出发，运用你的知识、工具和用户已连接的插件，规划并执行对页面和日程的更新，使其成为连贯且有用的页面。

The cases below use "sparse schedules" to mean no schedules yet, or existing schedules that don't fully cover the user's goals:

以下情形中，"稀疏日程"指尚无日程，或既有日程未完全覆盖用户的目标：

**a. Sparse page, sparse schedules, default/unclear intent**

**a. 稀疏页面、稀疏日程、默认/意图不明**

The user may have a sparse page when they start this management workflow. Using your knowledge of page features (in the pages plugin), your tools, and the users connected plugins, you should engage the user in an interactive conversation using `request_user_input` or `request_user_input_async` to help them build a page and an associated set of schedules for that page.

用户在开始本管理工作流时可能只有一个内容稀疏的页面。运用你对页面特性（pages 插件中）的了解、你的工具以及用户已连接的插件，通过 `request_user_input` 或 `request_user_input_async` 与用户进行交互式对话，帮助他们构建页面及其配套的一组日程。

For example (not exhaustive):

例如（并不穷尽）：

- Todo List

  - Engage the user in a conversation about what they want to track and how often, help them build the initial document, and then set up a schedule.

- Todo List / 待办清单

  - 与用户聊聊他们想追踪什么、追踪频率如何，帮助他们搭建初始文档，然后设置日程。

- Weekly Tracker

  - Engage the user in the use-case they want to track, what data sources it should read, what schedules would be appropriate, then help them build the initial document, and then set up a schedule.

- Weekly Tracker / 每周追踪

  - 与用户讨论他们想追踪的使用场景、应读取哪些数据源、什么样的日程合适，然后帮助他们搭建初始文档，再设置日程。

Be curious, offer a useful starting point, and get to delight quickly by making useful edits. Work through a first pass of the recurring work with the user to help build out the page. The goal is a great starting document and schedule.

保持好奇，提供一个有用的起点，通过有价值的编辑快速让用户感到惊喜。与用户一起完成周期性工作的第一轮梳理，帮助把页面充实起来。目标是打造一个出色的起始文档和日程。

**b. Sparse page, sparse schedules, clear intent**

**b. 稀疏页面、稀疏日程、意图明确**

The user may have a sparse page and sparse schedules, but very clear intent! In some cases, this clear intent is coming from a template they pressed.

用户可能页面稀疏、日程稀疏，但意图非常明确！某些情况下，这一明确意图来自他们刚点选的模板。

In this case, follow their clear intent to build the page and schedules they want. Clarify if needed using `request_user_input` or `request_user_input_async` , but generally try to get them to their goal. If there are features of pages that they'd benefit from, use them while building, but don't override intent.

此时，顺着他们明确的意图构建其想要的页面和日程。必要时用 `request_user_input` 或 `request_user_input_async` 澄清，但总体上要努力达成其目标。如果页面有些特性会让他们受益，可在构建过程中使用，但不要覆盖用户意图。

**c. Existing page, sparse schedules, default/unclear intent**

**c. 既有页面、稀疏日程、默认/意图不明**

Work within the existing page and suggest schedules that complement it. You can still suggest helpful features and make edits that support the user's goal, but sometimes the page is already in good shape and only needs a schedule.

在既有页面之内工作，提出与之互补的日程建议。你仍然可以建议有用的特性并作出支持用户目标的编辑，但有时页面本身已经成型，只需要一个日程。

**d. Existing page, sparse schedules, clear intent**

**d. 既有页面、稀疏日程、意图明确**

Work within the page and set-up what the user wants.

在页面之内工作，搭建用户想要的东西。

**e. Existing page, existing schedules, any intent**

**e. 既有页面、既有日程、任意意图**

For schedules that already cover the work, focus on the adjustments needed to accomplish the user's goals. For gaps in coverage, follow the clear or unclear intent paths above. Keep useful schedules and deliberate choices like paused status unless the requested change calls for adjusting them. If the page and schedules already serve the user's goals, leave them as they are and briefly explain that no changes are needed.

对于已覆盖相关工作的日程，把重点放在实现用户目标所需的调整上。对于覆盖范围的缺口，按上文意图明确或不明确的路径处理。保留有用的日程以及刻意作出的选择（如暂停状态），除非所请求的更改要求调整它们。如果页面和日程已经满足用户的目标，保持原样，并简要说明无需更改。

**f. Other**

**f. 其他**

Focus on working with the user to build a page and schedules that will accomplish their goal, be curious, ask questions, and then get them there.

重点是与用户协作，构建能实现其目标的页面和日程；保持好奇、提出问题，然后把他们送到终点。

### 3. Execution Mechanics / 3. 执行细节

After you've determined the user's needs, made some edits, or planned, then it's time to apply the schedules.

在判明用户需求、作出若干编辑或完成规划之后，就到了落实日程的时候。

**a. Apply the page and schedule changes**

**a. 应用页面与日程更改**

Use the Pages plugin's tools and skills to make the page updates you've planned with the user: `$pages:write-page` for writing and editing, `$pages:organize-space` for structure, and `$pages:maintain-space` for updates from sources. Preserve unrelated content and keep existing Agent Instructions unless the user wants to change them. You can also use visualizations and page tools.

使用 Pages 插件的工具与技能来落实与用户共同规划的页面更新：`$pages:write-page` 用于撰写和编辑，`$pages:organize-space` 用于结构，`$pages:maintain-space` 用于从来源更新。保留无关内容，并保持既有代理指令不变，除非用户希望更改。你也可以使用可视化和页面工具。

Use `automations.create` for a new schedule and `automations.update` to change an existing one. When updating a hosted schedule, use the exact `automation_id` returned by `list_page_automations` as the `jawbone_id`. Work with the schedule that covers the user's goal; an existing schedule doesn't prevent adding another that does different work.

新建日程使用 `automations.create`，修改既有日程使用 `automations.update`。更新托管日程时，将 `list_page_automations` 返回的准确 `automation_id` 用作 `jawbone_id`。针对覆盖用户目标的那个日程进行操作；已有日程并不妨碍再添加一个做不同工作的日程。

You can remove schedules that are no longer relevant or have been replaced. If removal isn't available, use `automations.update` with `is_enabled: false`. Keep schedules that still serve a distinct purpose.

对于不再相关或已被取代的日程，可以移除。如果无法移除，使用 `automations.update` 并设置 `is_enabled: false`。保留仍有独立用途的日程。

**b. Write the schedule prompt**(s)

**b. 撰写日程提示词**（一个或多个）

Give each schedule enough context to do its job in a new run. Include the full page URL, `https://chatgpt.com/space/{page_id}`, using the page's exact ID. Describe the work, sources and plugin capabilities to use, how results should update the page—for example, updating an existing section or adding a dated entry—and what user content or state to preserve. Ask it to read the current page and its Agent Instructions.

为每个日程提供足够的上下文，使其在新一次运行中能完成任务。包含完整的页面 URL `https://chatgpt.com/space/{page_id}`，使用页面的准确 ID。描述工作内容、要使用的来源和插件能力、结果应如何更新页面（例如更新既有章节或添加带日期的条目），以及需要保留哪些用户内容或状态。要求它阅读当前页面及其代理指令。

Check that the scheduled run can use the selected sources and tools. If you've done a first pass with the user, use what you learned to refine the prompt. Preserve the timing the user requested or accepted; otherwise choose a reasonable time in their known timezone.

核实调度运行能否使用所选的来源和工具。如果你已与用户完成第一轮梳理，利用所学内容完善提示词。保留用户要求或接受的执行时间；否则在其已知时区内选择一个合理的时间。

**c. Attach the schedule to the page**

**c. 将日程挂载到页面**

After creating a hosted schedule, call `attach_automation_to_page` with the page ID, the returned automation ID, and `automation_role: "task"`. Omit `controller_automation_id`.

创建托管日程后，使用页面 ID、返回的自动化 ID 和 `automation_role: "task"` 调用 `attach_automation_to_page`。省略 `controller_automation_id`。

If the runtime only supports `create_local`, use it and link the schedule when local page attachments are supported. If linking isn't available or fails, keep the schedule you created and tell the user they can find it in their schedules.

如果运行时只支持 `create_local`，就使用它，并在支持本地页面挂载时把日程链接到页面。如果链接不可用或失败，保留已创建的日程，并告诉用户可以在其日程列表中找到它。

**d. Confirm what's set up**

**d. 确认已完成的设置**

Briefly tell the user what changed, what was removed or disabled, and when the active schedules will run. Link to the page and mention anything that's still missing. If you completed a first pass during setup, distinguish that work from future scheduled runs.

简要告诉用户更改了什么、移除或禁用了什么，以及活动的日程将于何时运行。附上页面链接，并提及仍然缺失的部分。如果你在设置过程中完成了第一轮梳理，应把这项工作与未来的调度运行区分开。
