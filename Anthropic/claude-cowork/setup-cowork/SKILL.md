<!-- BILINGUAL-EN-ZH -->
---
name: setup-cowork
description: Guided Cowork setup — install a matching plugin, try a skill, connect tools. Use when: set up cowork, setup cowork, get started with cowork, cowork onboarding, configure cowork, personalize cowork.
---

# Setup Cowork / 设置 Cowork

Help the user get Cowork configured for their work. Six steps — role, plugins, connectors, try a skill, writing voice, wrap.

帮助用户为他们的工作配置好 Cowork。共六步——角色、插件、连接器、试用技能、写作风格、收尾。

## Step 0 — Checklist / 第 0 步——清单

Before your first user-facing message, create a TODO list with these items so the user can see progress:

在发出第一条面向用户的消息之前，先创建一个包含以下条目的 TODO 列表，让用户能看到进度：

1. Figure out role
   确定角色
2. Suggest plugins
   推荐插件
3. Suggest connectors
   推荐连接器
4. Try a skill
   试用技能
5. Set up writing voice
   设置写作风格
6. Wrap up
   收尾

Mark each one complete as you finish it. Keep it to these six — don't add sub-items.

每完成一项就标记为完成。保持只有这六项——不要添加子项。

## Step 1 — Role / 第 1 步——角色

Your initial message should frame what Cowork is: it autonomously handles tasks like reading your email, searching your docs, drafting reports, etc. Educate the user on _Skills_, reusable workflows you run with `/name`; _Connectors_, which wire in your tools; _Plugins_, which bundle skills and connectors for a domain. Two or three sentences. Hit the beats: multi-step and autonomous, uses your real tools, skills/plugins/connectors defined.

你的初始消息应说明 Cowork 是什么：它能自主处理诸如阅读邮件、搜索文档、起草报告等任务。向用户介绍 _Skills_（技能，用 `/name` 运行的可复用工作流）、_Connectors_（连接器，接入你的工具）、_Plugins_（插件，把某个领域的技能与连接器打包在一起）。两三句话即可。要覆盖的要点：多步骤且自主、使用你的真实工具、定义清楚技能/插件/连接器。

Next, ask the user for their role. Something like: "Let's get you set up — takes a few minutes. What kind of work do you do?" Then call the ShowOnboardingRolePicker tool, which renders a clickable role-picker chip row: do not list the roles yourself. The tool result is their answer — {"role": ...} is their role for the rest of setup; {"dismissed": true} or {} means they didn't pick one.

接下来，询问用户的角色。类似："我们来完成设置——只需几分钟。你是做什么工作的？"然后调用 ShowOnboardingRolePicker 工具，它会渲染一排可点击的角色选择标签：不要自己罗列角色。工具结果就是他们的回答——{"role": ...} 是他们在后续设置中使用的角色；{"dismissed": true} 或 {} 表示他们没有选择。

If the ShowOnboardingRolePicker tool is not available in this session, ask in plain text instead and offer these options as a short list they can reply to (they can also answer in their own words):

如果本会话中 ShowOnboardingRolePicker 工具不可用，则改为用纯文本询问，并把以下选项作为简短列表提供给他们回复（他们也可以用自己的话回答）：

- Product management
  产品管理
- Engineering
  工程
- Human resources
  人力资源
- Finance
  财务
- Marketing
  市场营销
- Sales
  销售
- Operations
  运营
- Data science
  数据科学
- Design
  设计
- Scientist
  科研
- Legal
  法务
- Student
  学生
- Founder
  创业者
- Healthcare
  医疗健康

In the plain-text case, end your turn after asking. Their reply — one of the options or a free-form answer — is their role for the rest of setup.

在纯文本情况下，问完即结束回合。他们的回复——选项之一或自由作答——就是他们在后续设置中使用的角色。

## Step 2 — Suggest plugins / 第 2 步——推荐插件

The role picker tool result will contain their selection. If it was dismissed or came back empty — or they skipped the plain-text question — they didn't pick a role: just suggest the productivity plugin and move on.

角色选择工具的结果会包含他们的选择。如果被关闭或返回为空——或者他们跳过了纯文本提问——说明他们没有选择角色：直接推荐 productivity 插件然后继续。

**Always** check for already-installed plugins before doing anything else — this is not optional. Call ListPlugins **without any intro text** — do not write "Looks like you already have…" before you know the result. The tool renders the installed plugins as a widget on its own; let it speak for itself. After it returns, react to what actually came back: if plugins appeared, acknowledge them below the widget ("Those are already on your account — here's what else fits your role."); if it's empty, just say "No plugins yet — let's fix that." Never write text that presumes a non-empty result before the tool runs. Do not pass installed plugins to SuggestPluginInstall afterward or you'll show them twice. Admin-provisioned plugins will appear in this list automatically; never skip the call. Then, regardless of what's installed, still recommend new role-matched plugins below in a separate widget.

**务必**在做任何其他事情之前先检查已安装的插件——这不是可选项。调用 ListPlugins 时**不要带任何引导文字**——在知道结果之前不要写"看起来你已经有了…"。该工具会自行把已安装插件渲染为一个组件；让它自己说话。返回之后，根据实际返回的内容作出反应：如果出现了插件，在组件下方予以确认（"这些已经装在你的账号上了——再看看还有哪些适合你的角色。"）；如果为空，就说"还没有插件——我们来装一些。"绝不要在工具运行前写出假定结果非空的文字。之后不要把已安装插件传给 SuggestPluginInstall，否则会重复展示。管理员配置的插件会自动出现在该列表中；绝不要跳过这次调用。然后，无论已安装了什么，仍要在下方单独的组件中推荐与角色匹配的新插件。

Search the plugin marketplace for their role with SearchPlugins. **Exclude anything already installed** — the installed-plugins widget above already covers those, so the recommendations widget must only contain plugins the user does not yet have. Never show the same plugin in both widgets. **Organization plugins always come first.** If the user's org has published its own plugins, those are the recommendation — they're built for this company's actual tools, data, and workflows, and someone internal decided they matter. An org-built plugin that's even loosely relevant to the role outranks any generic marketplace plugin, full stop. Lead with org plugins, and only reach for generic ones to fill empty slots when the org catalog has nothing close. Never bury an org plugin under a generic one.

用 SearchPlugins 在插件市场中搜索与其角色相关的内容。**排除所有已安装的插件**——上方的已安装插件组件已经覆盖了它们，因此推荐组件必须只包含用户还没有的插件。绝不要在两个组件中展示同一个插件。**组织插件永远排在最前。**如果用户的组织发布了自己的插件，它们就是推荐对象——它们是为这家公司实际的工具、数据和工作流构建的，而且有内部人员认定它们重要。哪怕只是与角色松散相关的组织自建插件，其优先级也高于任何通用市场插件，没有例外。以组织插件打头，只有当组织目录中没有相近之物时，才用通用插件填补空位。绝不要把组织插件排在通用插件之下。

Pick the top 2-3 matches and pass them as an array to SuggestPluginInstall so the user gets a browsable list. If only one is a strong fit, passing one is fine. If the search comes up empty, fall back to the productivity plugin. If every good match is already installed, skip the recommendations widget entirely and just say "You've already got the best plugin for [role] — let's move on to connectors."

选出最匹配的 2-3 个，作为数组传给 SuggestPluginInstall，让用户得到一份可浏览的列表。如果只有一个非常合适，只传一个也可以。如果搜索结果为空，回退到 productivity 插件。如果所有合适的匹配都已安装，就完全跳过推荐组件，直接说"你已经拥有了最适合 [role] 的插件——我们进入连接器环节吧。"

Above the widget, introduce it in one line: "Here are plugins built for [role] work — each one adds a set of skills you can run with `/`." The card shows Add or Manage depending on whether each plugin is already installed — don't describe the button. Below the widget, reinforce what they're for and tie it to the next step: "Installing one drops its skills straight into your `/` menu so you can run them anytime. Once you've picked one, want me to pull up the connectors it uses so those skills have your real data behind them?" — phrased so it works whether they're installing fresh or already have it. End your turn.

在组件上方，用一行话介绍它："这些是为 [role] 工作构建的插件——每个插件都会添加一组可用 `/` 运行的技能。"卡片会根据各插件是否已安装显示 Add 或 Manage——不要描述这个按钮。在组件下方，强调它们的用途并衔接下一步："安装一个插件，它的技能就会直接进入你的 `/` 菜单，随时可运行。选好之后，要我把它用到的连接器调出来吗？这样那些技能就能接上你的真实数据。"——措辞要同时适用于全新安装和已经安装两种情况。结束回合。

## Step 3 — Connectors / 第 3 步——连接器

If they say yes: tell them what you're about to do — "Let me check which connectors you've already got and what else your plugins could use."

如果他们同意：告诉他们你即将做什么——"我来查一下你已经有哪些连接器，以及你的插件还能用上哪些。"

Cover **every plugin in play** — everything already installed plus anything the user just added. Don't limit this to a single plugin; if the user has Sales and Productivity, pull connectors for both. Search SearchMcpRegistry per plugin domain, using the plugin's name and the user's role as queries, until every plugin in play has connector results — the results carry each connector's directoryUuid and whether it's already installed. Don't drop any relevant hit to prose; every connector those searches surface for their plugins should end up in the widget.

覆盖**每一个在用的插件**——所有已安装的加上用户刚添加的。不要只限于单个插件；如果用户同时有 Sales 和 Productivity，就为两者都拉取连接器。按插件领域逐一搜索 SearchMcpRegistry，以插件名和用户角色作为查询词，直到每个在用插件都有连接器结果——结果中带有每个连接器的 directoryUuid 以及是否已安装。不要把任何相关命中降级为纯文本叙述；这些搜索为其插件找出的每个连接器都应进入组件。

From those results: check which are already connected **before writing anything**. Only if at least one is connected, call ListConnectors with those names as keywords — and do not write "You're already connected to these:" above it; let the widget show it. If none are connected, skip ListConnectors entirely. Then call SuggestConnectors with **all** the still-unconnected UUIDs — the full set the searches surfaced, not just the top match. Any prose goes **after** the widgets, reacting to what actually rendered, never before.

从这些结果中：**在写任何文字之前**先检查哪些已经连接。只有当至少有一个已连接时，才以这些名称为关键词调用 ListConnectors——并且不要在其上方写"你已经连接了这些："；让组件自行展示。如果一个都没有连接，就完全跳过 ListConnectors。然后用**全部**仍未连接的 UUID 调用 SuggestConnectors——是搜索找出的完整集合，而不仅仅是排最前的匹配。任何文字都放在组件**之后**，针对实际渲染出的内容作出反应，绝不放在之前。

Below the suggestions, explain what they're looking at before moving on: "Click any of these to connect it — once wired up, skills can pull your real data from it. Want me to list some skills you can try?" End your turn.

在推荐内容下方，先解释他们看到的是什么再继续："点击任意一项即可连接——接好之后，技能就能从中拉取你的真实数据。要我列出一些可以试用的技能吗？"结束回合。

## Step 4 — Try a skill / 第 4 步——试用技能

If they say yes, call ListSkills with the plugin's name and their role as keywords so they get clickable skill cards; if the filter comes back empty, call it again with no keywords. Introduce the card in one line so it doesn't land cold: "Here's what [Plugin] adds — click any of these to run it now." End your turn. That card is keyword-filtered — when a later step needs to know everything on the user's account (Step 5 does), the answer comes from a keywordless ListSkills call or your system context's skills list, never from this filtered card.

如果他们同意，以插件名和他们的角色为关键词调用 ListSkills，让他们得到可点击的技能卡片；如果过滤结果为空，就不带关键词再调用一次。用一行话介绍这张卡片，避免显得突兀："这些是 [Plugin] 带来的内容——点击任意一项即可立即运行。"结束回合。该卡片是按关键词过滤的——当后续步骤需要知道用户账号上的全部技能时（第 5 步就需要），答案来自不带关键词的 ListSkills 调用或你系统上下文中的技能列表，绝不是这张过滤后的卡片。

When they click one (you'll see a `/name` message), help them with it. Keep it brief; you're still inside setup. When it finishes, bring it back: "Nice — that's how skills work."

当他们点击某个技能时（你会看到一条 `/name` 消息），协助他们完成。保持简短；你仍在设置流程中。完成后收束一下："不错——技能就是这样用的。"

If they wave it off at either point, that's fine — go to Step 5.

如果他们在任一节点表示不需要，也没关系——进入第 5 步。

## Step 5 — Writing voice / 第 5 步——写作风格

Everything so far taught Cowork about the user's *tools*. This step teaches it about the *user*. This matters because so much of what Cowork produces is prose the user will send under their own name.

到目前为止的一切都是在让 Cowork 了解用户的*工具*。这一步是让它了解*用户本人*。这很重要，因为 Cowork 产出的许多内容都是用户将以自己名义发出的文字。

**First, settle which opener you're writing — the account's full skills list decides.** Check the skills in your system context, or call ListSkills with no keywords; the plugin-filtered card from Step 4 covered one plugin and can't answer this. If `my-writing-style` is there (the saved profile — not `setup-writing-style`, the flow that creates it) — or the user says they've already set one up — your whole message is one line ("You've already got a voice profile, so anything I draft for you will use it") and you go to Step 6. Only if it's absent do you offer setup. Re-running the flow on someone who's already done it wastes their time and risks overwriting a profile they've tuned. If they *want* to update or redo it, that counts as a yes — invoke the skill the same way.

**首先，确定你要写的是哪种开场白——账号的完整技能列表说了算。**检查系统上下文中的技能，或不带关键词调用 ListSkills；第 4 步那张按插件过滤的卡片只覆盖一个插件，回答不了这个问题。如果 `my-writing-style` 在列表中（已保存的风格档案——注意不是 `setup-writing-style`，后者是创建该档案的流程）——或者用户说他们已经设置过——你的整条消息就是一句话（"你已经有一个文风档案了，我以后为你起草的内容都会使用它"），然后进入第 6 步。只有当它不存在时才提出设置。对已经做过的人重新运行该流程是浪费他们的时间，还有覆盖其已调校档案的风险。如果他们*想*更新或重做，那也算作同意——以同样的方式调用该技能。

If the user says they already have one, that settles it — a recently saved profile may not show in your skills list yet, so their word beats the list. Never tell a user they don't have a profile on the strength of a widget result; the widgets in this flow are plugin-filtered, and silence from one means nothing. Skipping a redundant offer costs a sentence; overwriting a tuned profile costs the user their work.

如果用户说自己已经有了，那就以此为准——最近保存的档案可能尚未出现在你的技能列表中，所以他们的话优先于列表。绝不要仅凭一个组件的结果就告诉用户他们没有档案；本流程中的组件都按插件过滤，其中一个没有显示不代表任何结论。跳过一次多余的提议只损失一句话；覆盖一个已调校的档案损失的是用户的作品。

【评论】以用户的口头确认覆盖工具列表的"沉默"，是在处理数据同步延迟：组件结果可能滞后于真实状态，文档选择把误报"没有档案"的代价（覆盖用户调校过的数据）看得比多问一句更重。

If `setup-writing-style` itself isn't available in this session, skip the offer entirely: mark this TODO done and go to Step 6 — the wrap's closing clause covers it.

如果本会话中 `setup-writing-style` 本身不可用，就完全跳过该提议：把这条 TODO 标记为完成并进入第 6 步——收尾部分的补充从句会覆盖这一点。

Otherwise, offer it. Make the case in two or three sentences of prose — these are the beats to hit, not a list to reproduce — then ask. Don't just launch into it:

否则，就提出设置。用两三句正文陈述理由——以下是要覆盖的要点，不是要照抄的列表——然后询问。不要不加分说就径直开始：

- **What it does:** reads writing they've already sent, learns how they write, and saves it so future drafts sound like them instead of like Claude.
  **它的作用：** 阅读他们已经发过的文字，学习其写作方式并保存下来，让以后的草稿听起来像他们本人，而不是像 Claude。
- **What it costs:** about two minutes.
  **它的代价：** 大约两分钟。
- **What it protects:** only writing they authored, and nothing saves without their review. (One clause — the skill itself walks through consent in detail once they say yes.)
  **它的保障：** 只学习他们本人撰写的文字，且任何内容不经他们审阅都不会保存。（一句话带过即可——一旦他们同意，该技能会详细讲解同意流程。）

Phrase the ask so passing is obviously fine — "Want to do that now, or skip it?" A user who feels cornered into a two-minute detour at the end of setup will just abandon the whole thing.

提问的措辞要让"跳过"显得完全没问题——"想现在做，还是跳过？"在设置接近尾声时感到被逼着绕两分钟弯路的用户，会直接放弃整个流程。

**If they say yes:** invoke the `setup-writing-style` skill (via the Skill tool — don't improvise its flow from memory) and let it run end to end. Don't paraphrase its steps, re-explain consent, or interleave your own commentary — it opens with its own framing, and a second voice narrating over it is confusing. Cowork setup is paused, not over. The voice flow counts as finished when one of three things happens: the save tool reports success; the user confirms the profile is saved (when saving happens via a Save skill button, you can't see the click and the new skill won't appear in your skills list until their next session — the flow already has you ask them to click it, so their answer is your signal; don't ask twice); or they ask to skip or move on to something else. Only then mark this TODO done and move to Step 6 — invoking the skill starts this step; it doesn't complete it.

**如果他们同意：** 调用 `setup-writing-style` 技能（通过 Skill 工具——不要凭记忆即兴复现其流程），让它从头到尾运行。不要转述它的步骤、重新解释同意事项，或穿插你自己的评论——它有自己的开场框架，第二个声音在上面解说会令人困惑。Cowork 设置是暂停，不是结束。以下三种情况之一发生时，文风流程即视为完成：保存工具报告成功；用户确认档案已保存（当保存通过"Save skill"按钮完成时，你看不到点击动作，且新技能要到他们下次会话才会出现在你的技能列表中——流程已经让你请他们点击，所以他们的回答就是你的信号；不要问第二次）；或者他们要求跳过或转向其他事情。只有到那时才把这条 TODO 标记为完成并进入第 6 步——调用技能只是开始这一步，并不等于完成它。

**If they say no or defer:** mark the TODO done and tell them they can always create their voice profile later by simply asking — e.g. "No problem. Whenever you want drafts to sound like you, just ask me to learn your writing voice." Then Step 6. Don't sell it twice.

**如果他们拒绝或推迟：** 把这条 TODO 标记为完成，并告诉他们以后随时可以通过一句话创建文风档案——例如"没问题。以后任何时候你想让草稿听起来像你，只要让我学习你的写作风格就行。"然后进入第 6 步。不要推销第二次。

## Step 6 — Wrap / 第 6 步——收尾

Close short: "You're set. Start a new task from the sidebar anytime, or type `/` to see your skills."

简短收尾："设置完成。随时可以从侧边栏发起新任务，或输入 `/` 查看你的技能。"

If they don't have a voice profile by the wrap, add one clause and no more: "…and whenever you want drafts to sound like you, just ask me to learn your writing voice."

如果到收尾时他们还没有文风档案，只加一个从句、不多不少："……以后任何时候你想让草稿听起来像你，只要让我学习你的写作风格就行。"

## Ground rules / 基本规则

- One step at a time.
  一次只进行一步。
- Skips are fine. If they pass on a step, mark its TODO done and move on.
  跳过没有问题。如果他们放弃某一步，把对应的 TODO 标记为完成然后继续。
- Keep each message short. Two or three sentences plus the widget, not a wall.
  每条消息保持简短。两三句话加一个组件，不要写成一大篇。
- Never write text that presumes a tool result before the tool runs. Don't say "you already have…" or "you're connected to…" above a widget — call the tool first, then react to what came back below it. The widget shows the data; your sentence reacts to it.
  绝不要在工具运行之前写出假定其结果的文字。不要在组件上方说"你已经有了…"或"你已经连接了…"——先调用工具，再在组件下方针对返回内容作出反应。组件负责展示数据；你的话负责对数据作出反应。

  【评论】"先调用工具、后写文字"的反复强调，是对大模型常见失败模式（在证据出现前预写结论）的防幻觉约束，同时兼顾 UI 组件与文本的分工。
- The user trying a skill mid-flow is expected. Help with it, then return to where you left off. Don't let a skill invocation end the setup. This applies to Step 5 too: `setup-writing-style` is a long flow, and when it ends — however it ends — the user still needs the Step 6 wrap.
  用户在流程中途试用技能是预期之中的。协助完成后，回到你之前进行到的位置。不要让一次技能调用终结整个设置。这一点也适用于第 5 步：`setup-writing-style` 是一个较长的流程，当它结束时——无论以何种方式结束——用户仍然需要第 6 步的收尾。
- If a tool named above isn't available in this session, skip that step's card and keep going in plain text.
  如果上文提到的某个工具在本会话中不可用，就跳过该步骤的卡片，继续用纯文本进行。
