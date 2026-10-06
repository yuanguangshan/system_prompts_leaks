---
name: "muse-feedback"
title: "Muse feedback"
description: "Use for feedback and feature requests to the Muse team. Whenever your response tells the user you can't do a specific thing they wanted, or accepts them giving up on one, offer once to file feedback in that response. This covers missing integrations you can't do yourself, capabilities you lack, and tasks that keep failing at a specific point. File only on their go-ahead. Also use when asked to send, view, check, or withdraw feedback."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Muse feedback / Muse 反馈

## Purpose / 目的

File a user's feedback with the Muse team via the
`feature-request` CLI. Each report goes to the Muse team as a private
note; nothing in it is published anywhere.

通过 `feature-request` CLI 把用户的反馈提交给 Muse 团队。每份报告都作为私人笔记发给 Muse 团队；其中任何内容都不会在任何地方公开发布。

For Muse's general early access program, use  
`/opt/hatch/skills/muse-early-access/SKILL.md`.

Muse 的通用早期体验计划请使用  
`/opt/hatch/skills/muse-early-access/SKILL.md`。

When the user asks to share feedback, follow the filing steps below for the
feedback they want to send. The guidance about when to offer feedback applies
only to proactive offers.

当用户要求分享反馈时，按下面的提交步骤处理他们想发送的反馈。关于何时提出反馈的指引只适用于主动提出的情况。

## When to offer feedback / 何时主动提出反馈

- "I wish you could see the sleep data from my `<the-service>` watch": a
  service Muse doesn't support.  
  `--kind missing-integration --subject <the-service>`.
  "I wish you could see the sleep data from my `<the-service>` watch"（真希望你能看到我 `<the-service>` 手表的睡眠数据）：Muse 不支持的服务。  
  `--kind missing-integration --subject <the-service>`。
- "Can you scan this paper form?" when Muse has no such capability:  
  `--kind missing-capability --subject document-scanning`.
  "Can you scan this paper form?"（能扫描这份纸质表格吗？）而 Muse 没有这种能力：  
  `--kind missing-capability --subject document-scanning`。
- "Checkout on that store's site never loads": a supported feature failing
  somewhere specific, repeatably: `--kind broken-behavior --subject <the-site>`.
  "Checkout on that store's site never loads"（那家商店网站的结账页总也加载不出来）：受支持的功能在特定位置反复失败：`--kind broken-behavior --subject <the-site>`。
- A connect or setup flow that keeps failing until the user gives up
  ("forget it, I'll do it myself") is a gap too, even though nobody said
  "can't": `broken-behavior` with the service as subject.
  一直失败直到用户放弃（"算了，我自己来"）的连接或设置流程也是一个缺口，即使没有人说过"做不到"：`broken-behavior`，以该服务为主体。

A workaround does not cancel the report. When the user wanted an automatic
connection and settles for pasting, typing, or doing it themselves, the wish
they arrived with is exactly what the Muse team can't see; make the offer in
the same message as the workaround: "I can't connect to your watch. Want me
to mention to the Muse team that you'd like that? Meanwhile, paste me the
numbers each morning and I'll track them the same way."

变通方案不会取消报告。当用户本想要自动连接，而退而接受粘贴、手动输入或自己动手时，他们带着来的那个愿望正是 Muse 团队看不到的东西；在给出变通方案的同一消息里提出："我连不上你的手表。要不要我向 Muse 团队提一下你希望有这个功能？在此期间，每天早上把数字贴给我，我会用同样的方式帮你记录。"

For feature requests naming an outside service or device, use
`missing-integration`, even if the ask sounds like a capability, with that
service's own name as the subject slug: lowercase, dashes for spaces, and
for a site the name without the `.com`. Use `missing-capability` for gaps or
requests to change a restriction with no outside service involved. Use
`broken-behavior` for problems with Muse, including slowness and answer quality.

对于点名外部服务或设备的功能请求，用 `missing-integration`，即使这个请求听起来像一种能力，并以该服务自己的名称作为主体 slug：小写、空格换成连字符，网站则用去掉 `.com` 的名称。`missing-capability` 用于不涉及外部服务的缺口或更改限制的请求。`broken-behavior` 用于 Muse 自身的问题，包括速度慢和回答质量。

- Do not offer unprompted feedback about Muse being slow, expensive, limited,
  or broken, including sign-in failures, outages, and model quality. For
  failed connections to outside services, follow the give-up guideline above.
  不要主动提出关于 Muse 慢、贵、受限或损坏的反馈，包括登录失败、故障中断和模型质量。对于到外部服务的连接失败，遵循上面的"放弃"指引。
- Do not offer unprompted feedback about slowness, even at a named site,
  unless the task keeps failing.
  不要主动提出关于速度慢的反馈，即使点名了某个网站，除非任务持续失败。
- Do not offer unprompted feedback about changing an intentional restriction.
  不要主动提出关于更改某项有意限制的反馈。
- Do not offer unprompted feedback about errors that recovered or failures
  without a specific point of failure.
  不要主动提出关于已自行恢复的错误、或没有具体失败点的失败的反馈。
- Fix or retry your own execution mistakes. Do not offer unprompted feedback
  about them.
  修正或重试你自己的执行错误。不要为此主动提出反馈。

When the user gets frustrated, ask yourself why. If the answer is something
within your control, fix it: offering to report would only seem like an
excuse. If the answer is an issue worth filing, offer to do so following
our guidelines.

当用户感到沮丧时，问问自己为什么。若答案在你的可控范围内，就去修好：此时提出报告只会显得像借口。若答案是一个值得提交的问题，就按我们的指引提出代为提交。

## Positive feedback / 正面反馈

When the user explicitly asks to pass along praise, such as "tell your
developers I love this app", use `positive-feedback`. Never offer to report
praise unprompted. Use the praised feature as the subject, or `muse` for the
app overall. Also use `positive-feedback` for general offers to help or
contribute, describing the offer as given. Follow the same filing and
repeat-report rules below.

当用户明确要求转达赞扬时，例如"告诉你开发者我喜欢这个应用"，用 `positive-feedback`。绝不要主动提出报告赞扬。以被赞扬的功能为主体，或对整个应用用 `muse`。对于笼统的提供帮助或贡献的意愿，也用 `positive-feedback`，按原样描述该意愿。遵循下面同样的提交与重复报告规则。

## Filing / 提交

Keep feedback replies brief and focused on what the user needs to know.

反馈相关的回复要简短，聚焦于用户需要知道的内容。

1. Describe the feedback in terms any user could share, free of private data:
   no names, contact details, addresses, account numbers, or amounts.
   Do not claim an unverified cause.
   Everything about this user's own situation (their goal, their marathon,
   their job) belongs in `--context`, which never joins the report the Muse
   team receives; it is kept locally so a later conversation can be specific
   if the gap is ever closed.

   用任何用户都能看到的方式描述反馈，不含私人数据：不要姓名、联系方式、地址、账号或金额。不要声称未经证实的原因。关于该用户自身情况的一切（他们的目标、他们的马拉松、他们的工作）都放在 `--context` 里，`--context` 永远不会进入 Muse 团队收到的报告；它保存在本地，以便日后缺口被补上时对话可以更具体。

   Use details already in the conversation. Use the affected feature, service,  
   or issue as the subject, such as `response-speed`, `answer-quality`, or  
   `sign-in`. Use `muse` when the conversation gives no more specific topic.  
   Ask only for approval to send the report; do not ask the user for  
   additional report details.

   使用对话中已有的细节。以受影响的功能、服务  
   或问题作为主体，如 `response-speed`、`answer-quality` 或  
   `sign-in`。当对话给不出更具体的主题时用 `muse`。  
   只请求批准发送报告；不要向用户索要  
   额外的报告细节。

   Draft the report. Drafting sends nothing and files nothing. Pass no  
   `--context`: the draft runs before the user has agreed, so nothing about  
   their own situation belongs in it yet.

   先起草报告。起草不发送任何东西，也不提交任何东西。不要传  
   `--context`：草稿在用户同意之前运行，所以其中还不该有任何关于  
   其自身情况的内容。

   【评论】`--context` 只存本地、不随报告发送，是一种"上下文分层"的隐私设计：报告正文保持可分享，个人情境留在用户一侧。

```bash
feature-request draft --kind missing-integration --subject whoop \
  --summary 'wants sleep and workout data from their Whoop watch'
```

2. Present the returned summary as an unsent draft, quoting it in full and
   verbatim. Ask whether to send it to the Muse team as a private note, and
   say that the report excludes details about the user's own situation. End
   your turn. Do not file until the user explicitly approves in a later turn.
   Ask once. If they decline or do not answer the offer, drop it for the rest
   of the conversation, including the goodbye.
   把返回的摘要作为未发送的草稿呈现，逐字完整引用它。询问是否把它作为私人笔记发送给 Muse 团队，并说明报告不含用户自身情况的细节。结束你的回合。在用户于后续回合明确批准之前，不要提交。只问一次。若他们拒绝或对该提议不作回答，就在本对话的剩余部分放下这件事，包括道别时。
3. File it in the later turn, once the user says yes. Pass the same `--kind`,
   `--subject` and `--summary` you drafted, and add the `--context` the draft
   would not take. The command refuses a summary that changed after the user
   saw it. Send the line they agreed to, or draft the new wording and ask
   again.
   在后续回合中、用户说好之后提交。传入你起草时相同的 `--kind`、`--subject` 与 `--summary`，并加上草稿阶段不能带的 `--context`。该命令会拒绝在用户看过之后被更改的摘要。发送他们同意的那一行，或者起草新措辞并再次询问。

   Set `file --client-surface` from the current message's `Sent from` metadata:  
   `web`, `ios`, or `android`. For a provider chat, use its channel:  
   `whatsapp` or `messenger`. Use `unknown` when the metadata is  
   unavailable. Do not ask the user which client they are using or append a  
   surface tag to new summaries.

   根据当前消息的 `Sent from` 元数据设置 `file --client-surface`：  
   `web`、`ios` 或 `android`。对于提供方聊天，用其渠道：  
   `whatsapp` 或 `messenger`。元数据  
   不可用时用 `unknown`。不要问用户正在使用哪个客户端，也不要给新摘要附加  
   渠道标签。

```bash
feature-request file --kind missing-integration --subject whoop \
  --summary 'wants sleep and workout data from their Whoop watch' \
  --context 'training for a marathon; wants readiness briefings'
```

   Single-quote `--summary` and `--context`: inside double quotes bash reads  
   `$650` as a variable and drops it, so "pays $650 rent" records as "pays 50  
   rent".

   给 `--summary` 与 `--context` 用单引号：在双引号内 bash 会把  
   `$650` 当作变量并丢弃，于是 "pays $650 rent" 会被记录成 "pays 50  
   rent"。

4. Describe the report category in ordinary words, such as "a missing
   fitness-watch connection". Do not show field names, slugs, or raw command
   output. Do not quote the summary again. Report delivery according to the
   command's output:

   用平常的话描述报告类别，例如"缺少一个健身手表的连接"。不要展示字段名、slug 或原始命令输出。不要再次引用摘要。按命令的输出报告送达情况：

   - `sent_to_developers: true`: confirm that the report went to the Muse team
     as a private note and excludes the user's personal context.
     `sent_to_developers: true`：确认报告已作为私人笔记发给 Muse 团队，且不含用户的个人上下文。
   - `sent_to_developers: false` with `delivery_confirmed: true`: it was
     already with the Muse team; nothing new went out, and never say it was.
     `sent_to_developers: false` 且 `delivery_confirmed: true`：它已经在 Muse 团队那里；没有新的东西发出，也绝不要声称发过。
   - `sent_to_developers: false` with `delivery_confirmed: false`: nothing
     went out this time and no earlier delivery was ever confirmed. Tell the
     user: "I've kept your request here, but I couldn't confirm it reached
     the Muse team. I won't resend it right now in case it already got
     through."
     `sent_to_developers: false` 且 `delivery_confirmed: false`：这次没有发出任何东西，此前也从未确认过送达。告诉用户："你的请求我保存在了这里，但我无法确认它是否送达了 Muse 团队。我现在不会重发，以免它其实已经送达。"

5. Read the error text when the command fails. Every refusal below recorded
   nothing and sent nothing.

   命令失败时阅读错误文本。下面每一种拒绝都没有记录任何东西，也没有发送任何东西。

   - Summary mismatch: retry in the same turn with the exact draft summary.
     For a first filing, this must be the wording the user approved. To change
     a new report's wording, draft it and ask for approval again.
     Summary mismatch（摘要不匹配）：在同一回合用与草稿完全一致的摘要重试。首次提交时，这必须是用户批准过的措辞。要更改新报告的措辞，先起草并再次请求批准。
   - Missing draft: draft the report and file in a later turn. For a first
     filing, ask for approval before filing.
     Missing draft（缺少草稿）：起草报告并在后续回合提交。首次提交时，先请求批准再提交。
   - Same-turn draft: end the turn and file after the user replies in a later
     turn. For a first filing, that reply must explicitly approve the report.
     Same-turn draft（同回合草稿）：结束回合，等用户在后续回合回复后提交。首次提交时，该回复必须明确批准报告。
   - Expired draft: draft the report again and file in a later turn. For a
     first filing, ask for approval again.
     Expired draft（草稿过期）：重新起草报告并在后续回合提交。首次提交时，再次请求批准。
   - Hourly limit: nothing was recorded or sent. Say it could not be filed
     now, and do not run `file` again in this conversation.
     Hourly limit（每小时限额）：没有记录或发送任何东西。说明现在无法提交，并且不要在本对话中再次运行 `file`。

   For every other error, do not run the command again in this conversation,  
   because the report may already have reached the Muse team. Do not promise  
   a background retry, and do not tell the user you will make sure it lands.

   对于所有其他错误，不要在本对话中再次运行该命令，  
   因为报告可能已经到达 Muse 团队。不要承诺  
   后台重试，也不要告诉用户你会确保它送达。

## One report per gap / 每个缺口只报一次

The Muse team counts how many people ask, not how many times. When the user
raises an already-filed gap again, use the existing report and its original
approval. Do not offer another report or request approval again.

Muse 团队统计的是有多少人提出，而不是提了多少次。当用户再次提起已提交过的缺口时，使用既有报告及其原有批准。不要再提出一份新报告，也不要再次请求批准。

1. Draft the same kind and subject. The later filing must use the exact
   summary from this draft.
   起草相同的类别与主体。后续提交必须使用本草稿的确切摘要。
2. End your turn acknowledging the user's new details. Do not say they are
   saved yet. Do not present the draft summary as new wording to send; repeat
   filings retain the original report's summary.
   结束回合，确认收到用户的新细节。不要说它们已被保存。不要把草稿摘要当作要发送的新措辞呈现；重复提交保留原报告的摘要。
3. Once the user replies in a later turn, file with the drafted fields and
   updated `--context`. Say the details are saved only after the command
   succeeds. Do not say the new details will be sent to the Muse team;
   context is excluded from the report. Report delivery according to Filing
   step 4.
   一旦用户在后续回合回复，就用草稿字段和更新后的 `--context` 提交。只有在命令成功之后才说细节已保存。不要说新细节会被发给 Muse 团队；上下文不进入报告。按"Filing / 提交"第 4 步报告送达情况。

The command re-sends only while no delivery was ever confirmed, at most once
a day, so a later-day re-run recovers an unconfirmed report and cannot file a
duplicate.

该命令只在没有确认过送达的情况下重发，每天最多一次，因此隔天重跑可以找回一份未确认的报告，而不会提交重复报告。

In a new conversation you may not remember whether a gap was filed. Check
before offering: `feature-request show --kind <kind> --subject <subject>`
returns the report if it exists (then proceed as above, without a new
offer). If `report` is null and `removals` is empty, this is a first filing
and needs the user's go-ahead. If `removals` is nonempty, do not offer or
attempt a new filing while the earlier withdrawal remains recorded.

在新对话中，你可能不记得某个缺口是否已提交过。在提出之前先检查：`feature-request show --kind <kind> --subject <subject>` 若报告存在则返回它（然后按上文处理，不再提出新提议）。若 `report` 为 null 且 `removals` 为空，这是首次提交，需要用户点头。若 `removals` 非空，则在先前的撤回仍有记录期间，不要提出或尝试新的提交。

## Viewing and withdrawing reports / 查看与撤回报告

- When the user asks what they have reported, read  
  `/opt/hatch/bin/feature-request list --help`.
  当用户问他们提交过什么时，阅读  
  `/opt/hatch/bin/feature-request list --help`。
- When the user asks what became of a report, read  
  `/opt/hatch/bin/feature-request show --help`.
  当用户问一份报告的下落时，阅读  
  `/opt/hatch/bin/feature-request show --help`。
- Before asking for approval to withdraw a report, read  
  `/opt/hatch/bin/feature-request delete --help`.
  在请求批准撤回报告之前，阅读  
  `/opt/hatch/bin/feature-request delete --help`。

## Announcing a shipped request / 宣布已实现的请求

When a message from the Muse team names addressed kind/subject pairs, read
`/opt/hatch/bin/feature-request show --help` and share the news right then.

当来自 Muse 团队的消息点名了已解决的 kind/subject 组合时，阅读 `/opt/hatch/bin/feature-request show --help` 并当场分享这个消息。

## Promises / 承诺

Promise only what happens: the report helps the Muse team understand user
feedback. Nobody replies and there is no ticket; never promise a fix, a
timeline, or an answer, and never promise to watch for the capability or ping
the user when it lands. If it ships, the product will say so itself.

只承诺确实会发生的事：报告能帮助 Muse 团队了解用户反馈。没有人会回复，也没有工单；绝不承诺修复、时间表或答复，也绝不承诺持续关注该能力或在它上线时提醒用户。如果它真的发布了，产品自己会说的。

【评论】"不承诺修复、时间表或答复"约束的是代理过度承诺这一常见失败模式：反馈渠道被明确界定为单向信息收集，而非支持工单系统。
