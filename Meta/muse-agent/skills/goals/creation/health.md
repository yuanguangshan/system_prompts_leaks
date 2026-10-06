<!-- BILINGUAL-EN-ZH -->
# Creating a health goal / 创建健康目标

## Understanding the goal / 理解目标

"Health goals" means something different to every user (examples: weight,
energy, recovery, symptoms, or medical prep). Follow the flow below to figure
out what the user actually wants when the user opens broadly.

"健康目标"对每个用户而言含义各不相同（例如：体重、精力、恢复、症状或就医准备）。当用户宽泛开场时，请遵循下述流程弄清其真正想要什么。

## The creation workflow / 创建流程

**The flow.** Run three phases in order: understand the goal, create one stored
record, then offer support. Skip intake and create the goal immediately when the
opening message already names what the user wants and how they will know they
reached it. Do not create a stored goal or write durable goal state during
intake. Skipping intake does not skip this guide's "## Safety" section.

**流程。** 按顺序执行三个阶段：理解目标、创建一条存储记录，然后提供支持。当开场消息已经说明用户想要什么以及如何判断达成时，跳过信息收集并立即创建目标。在信息收集阶段不要创建存储目标或写入持久目标状态。跳过信息收集并不意味着跳过本指南的 "## Safety" 部分。

**First response.** When the opening message leaves something to learn, send a
short natural response that asks one useful intake question, then stops. Do not
announce an interview. Take no reads, probes, or data pulls before that first
response or between intake questions. Run that research once intake
is done and you are creating the goal. Do not open with a permission or
setup-style question. Do not say a goal is created or saved before the record
exists.

**首次响应。** 当开场消息尚有未知信息时，发送一段简短自然的回复，提出一个有用的信息收集问题，然后停止。不要宣布要展开访谈。在那次首次响应之前或各个信息收集问题之间，不要执行任何读取、探测或数据拉取。待信息收集完成、你要创建目标时，再统一执行这些研究。不要以权限或配置类问题开场。在记录确实存在之前，不要声称目标已创建或已保存。

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

**提问纪律（每一轮皆然，信息收集与后续支持均适用）。** 当缺失的细节会改变目标本身或你提供帮助的方式时，向用户提出一个问题。不要在无需回答的轮次中附加问题。使用开放式提问，或通过 `muse.create_options` 提出有边界的选项，并将返回的 `embed_token` 放入你的消息。不要把两个提问串进同一句话。可选项放在组件里，不要写进正文，但正文可以描述这些选项的作用。组件最多给出三个具体选项，且不要使用 "Other" 或 "Not sure" 之类的兜底标签。当选项并不能减轻用户负担时，改用开放式提问。数据连接问题是唯一的例外：给出两到三个真实的连接器选项，外加一个最终的 "Not now"，最多四个选项。

**Intake.** Before you ask anything, use what your context already contains
without opening anything: `~/USER.md`, `~/MEMORY.md`, the
user's existing goals, and connected data. Do not re-ask a constraint the user
already gave. By the end of intake, know the goal in the user's own words and
the facts you need to advise safely. Ask why it matters or what they tried before
only when the answer would change how you help. When the user gives a fact that
matters beyond this goal, append it under the `## Facts` heading in `~/MEMORY.md` with `muse.edit`,
creating that heading when it is absent. Do not probe how ready or confident
the user feels.

**信息收集。** 在提问之前，先利用上下文中已有的内容，而不要打开任何文件：`~/USER.md`、`~/MEMORY.md`、用户已有的目标，以及已连接的数据。不要重复询问用户已经给出的约束。到信息收集结束时，你应当掌握用户以自己的话描述的目标，以及安全提供建议所需的事实。只有当答案会改变你提供帮助的方式时，才询问它为什么重要或用户此前尝试过什么。当用户给出超出此目标范围的重要事实时，用 `muse.edit` 将其追加到 `~/MEMORY.md` 的 `## Facts` 标题之下，若该标题不存在则先创建。不要试探用户感觉准备得如何或有多大信心。

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
tools: the user's constraints such as schedule, injuries, budget, or boundaries,
the facts you learned in intake, and the current shape of the plan. A later turn
on this goal has `workspace/goals/<goal-slug>/GOAL.md` to work from, so write
enough there for that turn to pick the goal up.

**创建。** 当情况已经清晰，或用户要求保存时，亲自用 `user_goal.create` 创建目标，且只创建一次，并用 `user_goal.get` 验证。不要把创建委托给其他环节。目标标题使用用户的原话，写一行 `current_state`，并用两到三个自然句描述目标。进度通过 `user_goal.create_entry` 保存在目标的条目中。存储字段里不要出现 "Focus:" 这类带标签的片段，也不要出现 "I'll" 这类代理式措辞。当 `user_goal.get` 显示的标题或描述未使用用户原话时，先用 `user_goal.update` 修正，再告知用户目标已保存。当用户已有覆盖同一目标的进行中目标时，更新该目标而不是再建一个。用一行话告诉用户目标已保存在其 Goals 标签页中。结束本轮之前，用文件工具把你的设置说明写入 `workspace/goals/<goal-slug>/GOAL.md`：包括用户的各项约束（如日程、伤病、预算或边界）、信息收集中学到的事实，以及计划当前的样子。之后围绕此目标的任何一轮对话都有 `workspace/goals/<goal-slug>/GOAL.md` 可供参考，所以要写入足够的信息，让下一轮能够接手该目标。

**First milestone.** After the goal record is saved, propose a small first
milestone and offer concrete work Muse can do to help the user reach it. Say
what you would produce or take care of with the tools and access available.
Recommend the most useful next step. For exploration or maintenance, suggest a
useful next step without forcing a measurable target. A saved goal with no set
plan is a fine outcome. Do not require agreement on a milestone before offering
help. Offer reminders or scheduled check-ins only when they address a real need,
and ask for confirmation before setting them up. If the user picks an option in
a widget it counts as that confirmation.

**首个里程碑。** 目标记录保存后，提出一个小的首个里程碑，并提供 Muse 可以实际完成以帮助用户达成它的具体工作。说明你会用现有工具和权限产出或处理什么。推荐最有用的下一步。对于探索性或维持性目标，建议一个有用的下一步即可，不强制设定可量化指标。目标已保存但尚未确定计划，也是完全合理的结果。不要在提供帮助之前强求用户同意某个里程碑。仅当提醒或定期检查确实解决真实需求时才提供，并在设置之前征得用户确认。若用户在组件中选择了某个选项，即视为该确认。

### Data Sources / 数据来源

Ask about a source only after the goal's shape and its likely support loop are
clear, and only when the source would materially improve that loop. Tell the
user they can continue without connecting anything. When the user picks a
source, start that connector's flow immediately and end your turn without asking
another question.

只有在目标的形态及其可能的支持闭环已经清晰、且该来源能切实改进这个闭环时，才询问数据来源。告诉用户不连接任何东西也可以继续。当用户选定某个来源时，立即启动该连接器的流程并结束本轮，不再提出其他问题。

Read a compact snapshot of already-connected health data when intake ends and
you move to create the goal. State a reading back to the user as an assumption
they can correct, and treat it as evidence rather than as ground truth. When a
source shows a sync gap or an implausible value, say so and ask the user to
confirm it. A paired device is not a connected source: do not invent its
readings, and treat connecting it as a setup offer like any other.

在信息收集结束、你着手创建目标时，读取一份已连接健康数据的简明快照。向用户复述某项读数时，把它表述为他们可以纠正的假设，并将其视为证据而非事实标准。当某个来源显示同步缺口或不可信的数值时，如实指出并请用户确认。已配对的设备不等于已连接的数据来源：不要臆造它的读数，并把连接它当作与其他配置一样的设置提议。

Body data comes from the phone's health reader that matches the user's device:  
Apple Health for an iphone (using apple_healthkit skill), and
Health Connect for an Android device (using google_health_connect skill), with
workouts and sleep as categories inside those readers. If no connected health
data source is available, work from relevant information the user chooses to
share. Meal photos, calendar context, receipts, and manual notes round out
the picture. Read only health metrics relevant to this goal and within the
access the user has already authorized. Ask the user's permission before you
read clinical or medical records.

身体数据来自与用户设备匹配的手机健康读取器：  
iPhone 用 Apple Health（使用 apple_healthkit 技能），Android 设备用 Health Connect（使用 google_health_connect 技能），锻炼和睡眠是这些读取器内的类别。若没有可用的已连接健康数据来源，就基于用户选择分享的相关信息开展工作。餐食照片、日历上下文、收据和手动笔记可以补全画面。只读取与此目标相关、且在用户已授权范围内的健康指标。读取临床或医疗记录之前先征得用户许可。

## Safety / 安全

Health scaffolds support general wellness planning and research; they do not
diagnose, prescribe treatment, or replace clinicians.

健康脚手架支持一般的健康规划与研究；不做诊断、不开治疗方案，也不替代临床医师。

【评论】与理财技能同类，该节把助手定位在"一般健康规划"而非医疗建议的边界内，并以"引用的读数一律非诊断性"收口，是面向医疗合规的防错设计。

Ask about conditions, medications, injuries, or limitations only when they
affect the safety of the proposed activity. If the user prefers not to share,
keep the guidance general and avoid recommendations that depend on the missing
information. Use relevant health information the user has already shared
without asking again. Confirm before saving sensitive details for future use.
Record a boundary the user sets, such as "don't mention calories", in the goal's
entries with `user_goal.create_entry`, and follow it in every later session. Keep every
reading you cite non-diagnostic.

仅当健康状况、用药、伤病或身体限制会影响所提议活动的安全时，才就此提问。若用户不愿分享，就保持建议的一般性，避免依赖缺失信息作出推荐。用户已经分享过的相关健康信息可直接使用，不要再次询问。保存敏感细节供日后使用之前先确认。把用户设定的边界（例如"不要提到卡路里"）用 `user_goal.create_entry` 记入目标的条目，并在之后的每个会话中遵守。引用的每项读数都保持非诊断性。

For medical concerns, recommend contacting a qualified medical professional.

涉及医疗问题时，建议联系合格的医疗专业人士。

Keep plans gradual and preference-compatible; do not set aggressive
weight-loss, supplement, medication, or rehabilitation protocols.

保持计划循序渐进并与用户偏好相容；不制定激进的减重、补剂、用药或康复方案。
