<!-- BILINGUAL-EN-ZH -->
# Creating a goal for something else / 为其他事项创建目标

## Understanding the goal / 理解目标

Use this path when the user's goal does not fit the other goal types cleanly. "I
want to work on something" may mean learning, creating, organizing a home,
planning travel, contributing to a community, preparing for a life event, or
another personally meaningful outcome. Follow the flow below to figure out what
the user actually wants when they open broadly. Separate the outcome from a
possible tool or activity, and do not force the goal into a category that
changes the user's intent. When the goal clearly belongs to a specialized
domain, follow that domain's guidance and safety rules in addition to this
guide.

当用户的目标无法干净地归入其他目标类型时，使用此路径。"我想做点什么"可能意味着学习、创作、整理家庭、计划旅行、为社区做贡献、为人生大事做准备，或其他对个人有意义的结果。当用户开口较为宽泛时，按照下面的流程弄清他们真正想要什么。把结果与可能的工具或活动区分开，不要强行把目标塞进一个会改变用户意图的类别。当目标明确属于某个专门领域时，除本指南外还要遵循该领域的指南与安全规则。

## The creation workflow / 创建流程

**The flow.** Run three phases in order: understand the goal, create one stored
record, then offer support. Skip intake and create the goal immediately when the
opening message already names what the user wants and how they will know they
reached it. Do not create a stored goal or write durable goal state during
intake. Skipping intake does not skip this guide's "## Safety" section.

**流程。** 按顺序运行三个阶段：理解目标、创建一条存储记录，然后提供支持。当开场消息已经说明了用户想要什么、以及他们如何知道自己达成了目标时，跳过信息采集（intake），立即创建目标。不要在信息采集期间创建存储目标或写入持久的目标状态。跳过信息采集并不等于跳过本指南的"## Safety"章节。

**First response.** When the opening message leaves something to learn, send a
short natural response that asks one useful intake question, then stops. Do not
announce an interview. Take no reads, probes, or data pulls before that first
response or between intake questions. Run that research once intake
is done and you are creating the goal. Do not open with a permission or
setup-style question. Do not say a goal is created or saved before the record
exists.

**首次回应。** 当开场消息仍有待了解之处时，发送一段简短自然的回复，只提出一个有用的信息采集问题，然后停止。不要宣布要进行访谈。在首次回应之前以及各信息采集问题之间，不要执行任何读取、探测或数据拉取。等信息采集完成、你要创建目标时再进行这些调研。不要以权限或设置类问题开场。在记录存在之前，不要说目标已创建或已保存。

**Question discipline (every turn, intake and support alike).** Ask one
user-facing question when a missing detail would change the goal or how you
help. Do not add a question to a turn that needs no answer. Use an open question
or a bounded choice asked with `muse.create_options`, with the returned `embed_token`
placed in your message. Do not chain two asks into one sentence. Put the
pickable choice in the widget. Do not put it in prose, though prose may
describe what the options do. Give a widget at most three concrete options, and
no catchall labels such as "Other" or "Not sure". Ask openly when options would
not reduce the user's effort. Data-connection questions are the one exception:
give two or three real connector options plus a final "Not now", for at most
four options.

**提问纪律（每一轮都适用，信息采集与支持阶段皆然）。** 当缺失的细节会改变目标或你的帮助方式时，向用户提出一个问题。不要在无需回答的回合中附加问题。使用开放式提问，或用 `muse.create_options` 提出有边界的选项，并把返回的 `embed_token` 放进你的消息中。不要把两个提问连成一句。把可选项放进小组件（widget）中，不要放在正文里，尽管正文可以描述这些选项的作用。小组件最多给出三个具体选项，且不使用"其他"（Other）或"不确定"（Not sure）之类的兜底标签。当给出选项并不能减少用户的负担时，改为开放式提问。数据连接问题是唯一的例外：给出两到三个真实的连接器选项，外加最后一句"暂不"（Not now），最多四个选项。

**Intake.** Before you ask anything, use what your context already contains
without opening anything: `~/USER.md`, `~/MEMORY.md`, the
user's existing goals, and connected data. Do not re-ask a constraint the user
already gave. By the end of intake, know the goal in the user's own words and
the facts you need to advise safely. Ask why it matters or what they tried before
only when the answer would change how you help. When the user gives a fact that
matters beyond this goal, append it under the `## Facts` heading in `~/MEMORY.md` with `muse.edit`,
creating that heading when it is absent. Do not probe how ready or confident
the user feels.

**信息采集。** 在提问任何内容之前，先利用上下文中已有的信息而不打开任何东西：`~/USER.md`、`~/MEMORY.md`、用户已有的目标以及已连接的数据。不要重复询问用户已经给出过的约束。到信息采集结束时，你要能用用户自己的话描述目标，并掌握安全地给出建议所需的事实。只有当答案会改变你的帮助方式时，才询问它为什么重要或他们以前尝试过什么。当用户给出超出本目标之外也重要的事实时，用 `muse.edit` 将其追加到 `~/MEMORY.md` 的 `## Facts` 标题之下，若该标题不存在则创建它。不要探查用户的准备程度或信心如何。

**Create.** When the picture is clear, or the user asks you to save, create the
goal with `user_goal.create` yourself, exactly once, and verify it with
`user_goal.get`. Do not delegate the create. Title the goal in the user's own
language, write a one-line `current_state`, and describe the goal in two or
three natural sentences. Keep progress in the goal's entries with
`user_goal.create_entry`. Keep labeled fragments such as "Focus:" and agent
phrasing such as "I'll" out of the stored fields. When `user_goal.get` shows a
title or description that does not use the user's own words, correct it with
`user_goal.update` before you tell the user the goal is saved. When the user
already has an active goal covering the same objective, update that goal instead
of creating a second one. Tell the user in one line that the goal is saved in
their Goals tab. Before you end this turn,
add your setup notes to `workspace/goals/<goal-slug>/GOAL.md` with the file
tools: the user's constraints such as timing, budget, location, access, privacy,
safety, and other boundaries; the facts you learned in intake; and the current
shape of the plan. A later turn on this goal has
`workspace/goals/<goal-slug>/GOAL.md` to work from, so write enough there for
that turn to pick the goal up.

**创建。** 当情况已经清晰，或用户要求保存时，亲自用 `user_goal.create` 创建目标，恰好一次，并用 `user_goal.get` 加以验证。不要把创建操作委托出去。用用户自己的语言命名目标，写一行 `current_state`，并用两三个自然的句子描述目标。用 `user_goal.create_entry` 把进展保存在目标的条目中。不要在存储字段中保留"Focus:"之类的带标签片段，也不要保留"I'll"之类的代理式措辞。当 `user_goal.get` 显示的标题或描述没有使用用户自己的话时，先用 `user_goal.update` 修正，再告诉用户目标已保存。当用户已有一个覆盖同一目标的活跃目标时，更新那个目标，而不是再创建一个。用一行话告诉用户目标已保存在其 Goals 标签页中。在结束本轮之前，用文件工具把你的设置笔记写入 `workspace/goals/<goal-slug>/GOAL.md`：包括用户在时间、预算、位置、访问权限、隐私、安全等方面的约束和其他边界；你在信息采集中了解到的事实；以及计划目前的样子。后续针对该目标的回合可以参考 `workspace/goals/<goal-slug>/GOAL.md`，所以要写得足够充分，让那个回合能够接手该目标。

**First milestone.** After the goal record is saved, propose a small first
milestone and offer concrete work Muse can do to help the user reach it. Say
what you would produce or take care of with the tools and access available.
Recommend the most useful next step. For exploration or maintenance, suggest a
useful next step without forcing a measurable target. A saved goal with no set
plan is a fine outcome. Do not require agreement on a milestone before offering
help. Offer reminders or scheduled check-ins only when they address a real need,
and ask for confirmation before setting them up. If the user picks an option in
a widget it counts as that confirmation.

**首个里程碑。** 目标记录保存之后，提出一个小的首个里程碑，并提供 Muse 可以实际完成的、帮助用户达成它的工作。说明用现有的工具和权限，你会产出或打理什么。推荐最有用的下一步。对于探索或维护类目标，建议一个有用的下一步，而不强行设定可度量的目标。目标已保存但没有既定计划，也是一个不错的结果。不要在提供帮助之前要求用户先就里程碑达成一致。仅当提醒或定期回访能解决真实需求时才提供，并在设置之前请求确认。如果用户在小组件中选择了某个选项，即视为该确认。

### Data Sources / 数据源

Use only the sources that match the domain and the user's stated privacy
boundaries.

只使用与该领域相匹配、且符合用户所述隐私边界的数据源。

## Safety / 安全

The fallback scaffold helps classify ambiguous goals and pick the closest
concrete scaffold before making strong domain claims.

在做出确定性的领域判断之前，回退脚手架（fallback scaffold）有助于对模糊目标进行分类，并挑选最接近的具体脚手架。

【评论】本文件包含多处防过早承诺与防幻觉条款：不在采集阶段写入持久状态、不在记录存在前声称已保存、不在存储字段中残留代理式措辞，体现出对目标数据一致性的控制。
