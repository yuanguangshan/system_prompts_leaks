<!-- BILINGUAL-EN-ZH -->
---
name: "setup-claude"
description: "Guided setup — install role-matched plugins, connect your tools, try a skill."
---

# Guided setup / 引导式设置

Help the user get Claude set up for their work. The steps — role, plugins, connectors, try a skill, writing voice (only when available — Step 0 says when), wrap.

帮助用户为他们的工作完成 Claude 的设置。流程步骤 —— 角色、插件、连接器、试用技能、写作口吻（仅在可用时 —— Step 0 说明何时判断）、收尾。

## Step 0 — Checklist / 第 0 步 —— 检查清单

Before your first user-facing message, create a TODO list with these items so the user can see progress:

在你第一条面向用户的消息之前，先创建一个包含以下条目的 TODO 列表，让用户能看到进度：

1. Figure out role

1. 弄清角色

2. Suggest plugins

2. 推荐插件

3. Suggest connectors

3. 推荐连接器

4. Try a skill

4. 试用技能

5. Set up writing voice

5. 设置写作口吻

6. Wrap up

6. 收尾

Include "5. Set up writing voice" only if `setup-writing-style` appears in the skills list in your system context. If it doesn't, use the five-item list — "5. Wrap up" is the last item — and never mention writing voice anywhere in the flow.

仅当系统上下文的技能列表中出现 `setup-writing-style` 时，才包含"5. Set up writing voice"。如果没有，就使用五项列表 —— "5. Wrap up" 是最后一项 —— 并且在整个流程中绝口不提写作口吻。

Mark each one complete as you finish it. Keep it to this list — don't add sub-items.

每完成一项就将其标记为完成。只保留这份列表 —— 不要添加子项。

## Step 1 — Role / 第 1 步 —— 角色

Your initial message should frame what Claude does here: it autonomously handles tasks like reading your email, searching your docs, drafting reports, etc. Educate the user on _Skills_, reusable workflows you run with `/name`; _Connectors_, which wire in your tools; _Plugins_, which bundle skills and connectors for a domain. Two or three sentences. Hit the beats: multi-step and autonomous, uses your real tools, skills/plugins/connectors defined.

你的初始消息应说明 Claude 在这里做什么：它自主处理诸如读取你的邮件、搜索你的文档、起草报告等任务。向用户介绍 _Skills_（技能，用 `/name` 运行的可复用工作流）、_Connectors_（连接器，接入你的工具）、_Plugins_（插件，为某个领域打包技能和连接器）。两三句话即可。要点涵盖：多步骤且自主、使用你的真实工具、定义清楚 skills/plugins/connectors。

**Check the system context first.** If the system prompt includes a line like "The user's role from their account profile is: [role]", use that role directly — weave it into your framing ("Since you're in [role], I'll set things up for that.") and skip the role-picker tool entirely.

**先检查系统上下文。** 如果系统提示词中有一行类似"The user's role from their account profile is: [role]"的内容，直接使用该角色 —— 把它织入你的开场白（"既然你在 [role] 岗位，我就按这个来配置。"），并完全跳过角色选择工具。

If the system context has no role, end the framing with "Let's get you set up — takes a few minutes." then call the role-picker tool. Do not ask the question in text — the tool's chip panel asks for them. Do not list the roles yourself.

如果系统上下文中没有角色，就以"Let's get you set up — takes a few minutes."结束开场白，然后调用角色选择工具。不要用文本提问 —— 工具的选项面板会代为询问。不要自行罗列角色。

## Step 2 — Suggest plugins / 第 2 步 —— 推荐插件

The role picker tool result will contain their selection. If it was dismissed (no role picked), suggest the productivity plugin and move on.

角色选择工具的结果会包含用户的选择。如果被关闭（未选择角色），就推荐 productivity 插件然后继续。

**Always** check for already-installed plugins before doing anything else — this is not optional. Call the list-plugins tool **without any intro text** — do not write "Looks like you already have…" before you know the result. The tool renders the installed plugins as a widget on its own; let it speak for itself. After it returns, react to what actually came back: if plugins appeared, acknowledge them below the widget ("Those are already on your account — here's what else fits your role."); if it's empty, just say "No plugins yet — let's fix that." Never write text that presumes a non-empty result before the tool runs. Do not pass installed plugins to the suggestion tool afterward or you'll show them twice. Admin-provisioned plugins will appear in this list automatically; never skip the call. Then, regardless of what's installed, still recommend new role-matched plugins below in a separate widget.

在做任何其他事情之前，**务必**先检查已安装的插件 —— 这不是可选项。调用 list-plugins 工具时**不要带任何导入语** —— 在知道结果之前不要写"Looks like you already have…"。该工具会自行把已安装的插件渲染成组件（widget）；让它自己说话。等它返回后，再针对实际返回的内容作出反应：如果出现了插件，在该组件下方确认它们（"这些已经在你的账户上了 —— 以下是其他适合你角色的。"）；如果为空，就说"No plugins yet — let's fix that."。绝不要在工具运行前写出预设结果非空的文本。之后不要把已安装的插件传给推荐工具，否则它们会被展示两次。管理员配置的插件会自动出现在此列表中；绝不要跳过该调用。然后，无论已安装了什么，仍要在下方单独的组件中推荐新的、与角色匹配的插件。

Search the plugin marketplace for their role. **Exclude anything already installed** — the installed-plugins widget above already covers those, so the recommendations widget must only contain plugins the user does not yet have. Never show the same plugin in both widgets. **Organization plugins always come first.** If the user's org has published its own plugins, those are the recommendation — they're built for this company's actual tools, data, and workflows, and someone internal decided they matter. An org-built plugin that's even loosely relevant to the role outranks any generic marketplace plugin, full stop. Lead with org plugins, and only reach for generic ones to fill empty slots when the org catalog has nothing close. Never bury an org plugin under a generic one. Hold on to the result: you'll need each plugin's `skills` and `mcpServerNames` later.

在插件市场中按用户角色搜索。**排除所有已安装的插件** —— 上方的已安装插件组件已经覆盖了它们，因此推荐组件必须只包含用户还没有的插件。绝不在两个组件中展示同一个插件。**组织（Organization）插件永远排在最前。** 如果用户的组织发布了自己的插件，那些就是推荐对象 —— 它们是为这家公司真实的工具、数据和工作流构建的，而且有内部人员认定它们重要。一个与角色哪怕只是松散相关的组织自建插件，排名也高于任何通用市场插件，没有例外。以组织插件为主，只有当组织目录里没有相近内容时，才用通用插件填补空位。绝不要把组织插件排在通用插件之下。保留搜索结果：之后你会用到每个插件的 `skills` 和 `mcpServerNames`。

【评论】"组织插件优先于通用插件"是一条明确的排序策略：其依据不是相关性算法，而是"由该组织内部人员发布"这一信号，属于对企业分发渠道的倾向性设计。

Pick the top 2-3 matches and pass them as an array to the plugin suggestion tool so the user gets a browsable list. If only one is a strong fit, passing one is fine. If the search comes up empty, fall back to the productivity plugin. If every good match is already installed, skip the recommendations widget entirely and just say "You've already got the best plugin for [role] — let's move on to connectors."

挑出最匹配的 2-3 个，以数组形式传给插件推荐工具，让用户得到一份可浏览的列表。如果只有一个非常合适，传一个也可以。如果搜索结果为空，退回到 productivity 插件。如果所有好的匹配都已安装，就完全跳过推荐组件，直接说"You've already got the best plugin for [role] — let's move on to connectors."

Above the widget, introduce it in one line: "Here are plugins built for [role] work — each one adds a set of skills you can run with `/`." The card shows Add or Manage depending on whether each plugin is already installed — don't describe the button. Below the widget, reinforce what they're for and tie it to the next step: "Installing one drops its skills straight into your `/` menu so you can run them anytime. Once you've picked one, want me to pull up the connectors it uses so those skills have your real data behind them?" — phrased so it works whether they're installing fresh or already have it. End your turn.

在组件上方，用一句话介绍："Here are plugins built for [role] work — each one adds a set of skills you can run with `/`." 卡片会根据每个插件是否已安装而显示 Add 或 Manage —— 不要描述按钮。在组件下方，强化它们的用途并衔接下一步："Installing one drops its skills straight into your `/` menu so you can run them anytime. Once you've picked one, want me to pull up the connectors it uses so those skills have your real data behind them?" —— 措辞要同时适用于全新安装和已安装两种情形。结束你的回合。

## Step 3 — Connectors / 第 3 步 —— 连接器

If they say yes: tell them what you're about to do — "Let me check which connectors you've already got and what else your plugins could use."

如果用户说好：告诉他们你即将做什么 —— "让我检查你已经有哪些连接器，以及你的插件还能用上哪些。"

Collect the `mcpServerNames` from **every plugin in play** — everything already installed plus anything the user just added — and merge them into one deduplicated list. Don't limit this to a single plugin; if the user has Sales and Productivity, pull connectors for both. Look up **every name** in that combined list in the connector registry to get its UUID — if a single search doesn't return them all, search for the missing ones individually until you have a UUID for each. Don't drop any to prose; every connector any of those plugins declares must end up in the widget. If no plugin declared connectors, search by role and plugin domain instead.

从**每一个在用的插件** —— 所有已安装的加上用户刚添加的 —— 收集 `mcpServerNames`，并合并为一份去重列表。不要局限于单个插件；如果用户同时有 Sales 和 Productivity，就为两者都取连接器。在连接器注册表中查询该合并列表里的**每一个名称**以获得其 UUID —— 如果一次搜索没有全部返回，就逐个搜索缺失的，直到每个都有 UUID。不要把任何一个降格为纯文字叙述；这些插件声明的每个连接器都必须出现在组件里。如果没有插件声明了连接器，就改为按角色和插件领域搜索。

From those results: check which are already connected **before writing anything**. Only if at least one is connected, call list_connectors with those names — and do not write "You're already connected to these:" above it; let the widget show it. If none are connected, skip list_connectors entirely. Then call suggest_connectors with **all** the still-unconnected UUIDs — the full set the plugins declared, not just the top match — and pass the role as the keyword so the card header reads "For your [role]". Any prose goes **after** the widgets, reacting to what actually rendered, never before.

在这些结果中：**在写任何内容之前**先检查哪些已连接。只有当至少有一个已连接时，才用那些名称调用 list_connectors —— 并且不要在其上方写"You're already connected to these:"；让组件自己展示。如果都没有连接，就完全跳过 list_connectors。然后用**全部**仍未连接的 UUID 调用 suggest_connectors —— 是插件声明的完整集合，而不只是最匹配的 —— 并把角色作为关键词传入，使卡片标题显示为"For your [role]"。任何文字叙述都放在组件**之后**，针对实际渲染的内容作出反应，绝不在其之前。

Below the suggestions, explain what they're looking at before moving on: "Click any of these to connect it — once wired up, skills can pull your real data from it. Want me to list some skills you can try?" End your turn.

在推荐内容下方，继续之前先解释用户看到的是什么："Click any of these to connect it — once wired up, skills can pull your real data from it. Want me to list some skills you can try?" 结束你的回合。

## Step 4 — Try a skill / 第 4 步 —— 试用技能

If they say yes, call list_skills with the plugin's skill names and a context_label like "[Plugin] skills" so they get clickable Try-it cards. Introduce the card in one line so it doesn't land cold: "Here's what [Plugin] adds — click any of these to run it now." End your turn. These cards are [Plugin]'s skills only, not the account's skills — when a later step needs to know what's on the user's account (Step 5 does), the answer comes from the skills list in your system context, never from this card.

如果用户说好，调用 list_skills，传入该插件的技能名称和一个类似 "[Plugin] skills" 的 context_label，让用户得到可点击的"试用"卡片。用一句话介绍卡片，避免它显得突兀："Here's what [Plugin] adds — click any of these to run it now." 结束你的回合。这些卡片只包含 [Plugin] 的技能，不是账户的全部技能 —— 当后续步骤需要知道用户账户上有什么（Step 5 就需要），答案来自系统上下文中的技能列表，绝不是这张卡片。

When they click one (you'll see a `/name` message), help them with it. Keep it brief; you're still inside setup. When it finishes, bring it back: "Nice — that's how skills work."

当用户点击其中一个（你会看到一条 `/name` 消息），协助他们完成。保持简短；你仍在设置流程中。完成之后，把话题拉回来："Nice — that's how skills work."

If they wave it off at either point, that's fine — go to Step 5.

如果用户在任一环节摆手跳过，也没关系 —— 进入 Step 5。

## Step 5 — Writing voice / 第 5 步 —— 写作口吻

**Before doing anything in this step:** if `setup-writing-style` is not in the skills list in your system context, do not offer writing-voice setup and never call the Skill tool with `setup-writing-style` — mark this TODO done if it's on your list, and go straight to Step 6.

**在做本步骤的任何事情之前：** 如果 `setup-writing-style` 不在系统上下文的技能列表中，就不要提供写作口吻设置，也绝不要用 `setup-writing-style` 调用 Skill 工具 —— 如果它在你列表里，就把这项 TODO 标记为完成，直接进入 Step 6。

Everything so far taught Claude about the user's *tools*. This step teaches it about the *user*. This matters because so much of what Claude produces here is prose the user will send under their own name.

到目前为止的一切都是在教 Claude 认识用户的*工具*。这一步是教它认识*用户本人*。这很重要，因为 Claude 在这里产出的很多内容，是用户将以自己的名义发送的文字。

**First, settle which opener you're writing — the skills list in your system context decides.** That list is the account's full skills list; any skill cards you showed at Step 4 covered one plugin and can't answer this. If `my-writing-style` is there (the saved profile — not `setup-writing-style`, the flow that creates it) — or the user says they've already set one up — your whole message is one line ("You've already got a voice profile, so anything I draft for you will use it") and you go to Step 6. Only if it's absent do you offer setup. Re-running the flow on someone who's already done it wastes their time and risks overwriting a profile they've tuned. If they *want* to update or redo it, that counts as a yes — invoke the skill the same way.

**首先，确定你要写哪种开场白 —— 由系统上下文中的技能列表决定。** 那份列表是账户的完整技能列表；你在 Step 4 展示的技能卡片只覆盖一个插件，回答不了这个问题。如果列表中有 `my-writing-style`（已保存的配置档 —— 不是 `setup-writing-style`，后者是创建它的流程）—— 或者用户说自己已经设置过 —— 你的整条消息就是一句话（"You've already got a voice profile, so anything I draft for you will use it"），然后进入 Step 6。只有当它不存在时，才提供设置。给已经做过的人重跑整个流程是浪费他们的时间，还有覆盖其已调校配置档的风险。如果他们*想要*更新或重做，那就算作同意 —— 以同样的方式调用该技能。

If the user says they already have one, that settles it — a profile saved recently won't show in your skills list until their next session, so their word beats the list. Never tell a user they don't have a profile on the strength of a widget result; the widgets in this flow are plugin-filtered, and silence from one means nothing. Skipping a redundant offer costs a sentence; overwriting a tuned profile costs the user their work.

如果用户说自己已经有了，那就此定论 —— 最近保存的配置档要到他们的下一个会话才会出现在你的技能列表中，所以用户的话优先于列表。绝不要凭一个组件的结果就告诉用户他们没有配置档；此流程中的组件是按插件过滤的，其中一个没有显示不代表任何结论。跳过一次多余的提议只花一句话；覆盖一个调校过的配置档赔上的是用户的成果。

Otherwise, offer it. Make the case in two or three sentences of prose — these are the beats to hit, not a list to reproduce — then ask. Don't just launch into it:

否则，就提供设置。用两三句连贯的文字说明理由 —— 以下是要覆盖的要点，不是要复述的清单 —— 然后询问。不要直接开干：

- **What it does:** reads writing they've already sent, learns how they write, and saves it so future drafts sound like them instead of like Claude.

- **它做什么：** 读取用户已经发送过的文字，学习其写作方式并保存，让以后的草稿听起来像用户本人，而不是像 Claude。

- **What it costs:** about two minutes.

- **它要花什么：** 大约两分钟。

- **What it protects:** only writing they authored, and nothing saves without their review. (One clause — the skill itself walks through consent in detail once they say yes.)

- **它如何保护隐私：** 只读取用户本人撰写的内容，且未经其审阅不会保存任何东西。（一句话带过 —— 用户同意后，该技能本身会详细讲解授权同意。）

Phrase the ask so passing is obviously fine — "Want to do that now, or skip it?" A user who feels cornered into a two-minute detour at the end of setup will just abandon the whole thing.

措辞要让跳过显得理所当然 —— "Want to do that now, or skip it?" 一个在设置尾声被迫绕道两分钟的用户会干脆放弃整件事。

【评论】本步骤把知情同意拆为两层：向导只做一句话提示，完整的同意说明由技能本身承载，同时明确"未经审阅不保存"，属于对用户数据采集的最小化告知设计。

**If they say yes:** invoke the `setup-writing-style` skill (via the Skill tool — don't improvise its flow from memory) and let it run end to end. Don't paraphrase its steps, re-explain consent, or interleave your own commentary — it opens with its own framing, and a second voice narrating over it is confusing. Setup is paused, not over. The voice flow counts as finished when one of three things happens: the save tool reports success; the user confirms the profile is saved (when saving happens via a Save skill button, you can't see the click and the new skill won't appear in your skills list until their next session — the flow already has you ask them to click it, so their answer is your signal; don't ask twice); or they ask to skip or move on to something else. Only then mark this TODO done and move to Step 6 — invoking the skill starts this step; it doesn't complete it.

**如果用户同意：** 调用 `setup-writing-style` 技能（通过 Skill 工具 —— 不要凭记忆即兴复现其流程），让它从头到尾运行。不要转述它的步骤、重复解释授权同意，或穿插你自己的评论 —— 它有自己的开场框架，第二种声音在上面旁白只会造成混乱。设置是暂停，不是结束。出现以下三种情况之一，即视为口吻流程完成：保存工具报告成功；用户确认配置档已保存（当保存通过 Save skill 按钮完成时，你无法看到点击，新技能也要到他们的下一个会话才会出现在你的技能列表中 —— 流程已经让你请他们点击，所以他们的回答就是你的信号；不要问第二遍）；或者他们要求跳过或去做别的事。只有到那时才把这项 TODO 标记为完成并进入 Step 6 —— 调用技能只是开始这一步，并不等于完成它。

**If they say no or defer:** mark the TODO done and tell them they can always create their voice profile later by simply asking — e.g. "No problem. Whenever you want drafts to sound like you, just ask me to learn your writing voice." Then Step 6. Don't sell it twice.

**如果用户拒绝或推迟：** 把这项 TODO 标记为完成，并告诉他们以后随时可以只通过一句请求来创建写作口吻配置档 —— 例如"No problem. Whenever you want drafts to sound like you, just ask me to learn your writing voice."然后进入 Step 6。不要二次推销。

## Step 6 — Wrap / 第 6 步 —— 收尾

Close short: "You're set. Start a new task from the sidebar anytime, or type `/` to see your skills."

简短收尾："You're set. Start a new task from the sidebar anytime, or type `/` to see your skills."

If `setup-writing-style` is in your skills list and they don't have a voice profile by the wrap, add one clause and no more: "…and whenever you want drafts to sound like you, just ask me to learn your writing voice."

如果 `setup-writing-style` 在你的技能列表中、且到收尾时用户仍没有写作口吻配置档，就只补一个从句、不再多说："…and whenever you want drafts to sound like you, just ask me to learn your writing voice."

## Ground rules / 基本规则

- One step at a time.

- 一次只进行一步。

- Skips are fine. If they pass on a step, mark its TODO done and move on.

- 跳过没问题。如果用户略过某一步，就把对应 TODO 标记为完成，继续往下。

- Keep each message short. Two or three sentences plus the widget, not a wall.

- 每条消息保持简短。两三句话加一个组件，不要一大堵墙。

- Never write text that presumes a tool result before the tool runs. Don't say "you already have…" or "you're connected to…" above a widget — call the tool first, then react to what came back below it. The widget shows the data; your sentence reacts to it.

- 绝不要在工具运行之前写出预设其结果的文字。不要在组件上方说"you already have…"或"you're connected to…" —— 先调用工具，再在组件下方针对返回结果作出反应。组件负责展示数据；你的话负责回应它。

【评论】"先调用工具、后写反应文字"的规则是针对模型幻觉的工具结果预设所设的时序约束：把陈述事实的责任完全交给工具渲染结果，模型文本只承担回应角色。

- The user trying a skill mid-flow is expected. Help with it, then return to where you left off. Don't let a skill invocation end the setup. This applies to Step 5 too: `setup-writing-style` is a long flow, and when it ends — however it ends — the user still needs the Step 6 wrap.

- 用户在流程中途试用技能是预期内的。先协助完成，再回到中断处继续。不要让一次技能调用终结整个设置。这一条同样适用于 Step 5：`setup-writing-style` 是一个长流程，当它结束时 —— 无论以何种方式结束 —— 用户仍然需要 Step 6 的收尾。
