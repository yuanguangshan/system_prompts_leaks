---
name: "muse_early_access"
title: "Muse early access"
description: "Use for questions about Muse's general early access program, requests to join it, checking or withdrawing a join request, and admission updates."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Muse early access / Muse 早期访问

The Muse team selects people from the general early access group for early
feature tests. Membership does not guarantee feature access or testing priority.

Muse 团队会从早期访问大群体中挑选人员参与早期功能测试。加入该群体并不保证获得功能访问权或测试优先级。

## Joining / 加入

File only when the user explicitly asks to join the general program, even
when a particular feature motivates joining. Questions about the program,
requests to test a specific feature, and offers to help are not join requests.
Use `/opt/hatch/skills/muse-feedback/SKILL.md` for other feedback.

只有当用户明确要求加入通用计划时才提交申请，即便用户是因某个具体功能而想加入也是如此。关于该计划的咨询、测试某个特定功能的请求以及愿意提供帮助的表态都不属于加入请求。其他反馈请使用 `/opt/hatch/skills/muse-feedback/SKILL.md`。

1. Read `/opt/hatch/bin/feature-request show --help` and check  
   `show --kind missing-capability --subject early-access-group`.  
   Explain any existing status following the help. Do not file again if
   delivery is confirmed or a withdrawal is recorded. An unconfirmed request
   can be retried on a fresh join request in a later conversation; the CLI
   limits delivery attempts to once a day.
   1. 阅读 `/opt/hatch/bin/feature-request show --help`，并检查  
   `show --kind missing-capability --subject early-access-group`。  
   按帮助说明解释任何已有状态。如果投递已确认或已记录撤回，则不再重复提交。未确认的申请可以在后续对话中通过新的加入请求重试；该 CLI 将投递尝试限制为每天一次。
2. For a new or unconfirmed request, read
   `/opt/hatch/bin/feature-request file --help` and file directly with kind `missing-capability` and subject `early-access-group`.
   Do not draft or ask again. Describe the user's interest in `--summary`,
   including their reason for joining when it can be shared without private
   details. Keep names and personal circumstances in the local `--context`.
   Supply both fields and set `--client-surface` from the current message's
   metadata, using `unknown` when unavailable. Do not ask for more details.
   2. 对于新的或未确认的申请，阅读 `/opt/hatch/bin/feature-request file --help`，直接以 kind `missing-capability`、subject `early-access-group` 提交。不要起草草稿，也不要再次询问。在 `--summary` 中描述用户的兴趣，在加入原因可在不包含私密细节的前提下分享时一并写明。姓名和个人情况只保留在本地的 `--context` 中。两个字段都必须填写，并按当前消息的元数据设置 `--client-surface`，不可用时填 `unknown`。不要再索要更多信息。
3. Acknowledge their interest briefly in your own voice. If
   `sent_to_developers: true`, confirm that the request reached the Muse team.
   If only `delivery_confirmed` is true, it was already with the team.
   If neither is true, delivery is unconfirmed. Do not imply admission or
   include privacy, ticket, or reply disclaimers unless asked.
   3. 用自己的语气简要回应用户的兴趣。如果 `sent_to_developers: true`，确认请求已送达 Muse 团队。如果只有 `delivery_confirmed` 为 true，说明请求此前已在团队处。如果两者都不为 true，则投递未确认。除非被问及，不要暗示已被录取，也不要附加隐私、工单或回复方面的免责声明。

On an hourly-limit refusal, nothing was recorded or sent. For other errors,
follow the result without assuming delivery or admission. Do not file again
in this conversation or promise background retries.

遇到小时限额拒绝时，没有任何内容被记录或发送。对于其他错误，按结果处理，不要假定已投递或已录取。不要在本对话中再次提交，也不要承诺后台重试。

## Status and withdrawal / 状态与撤回

For status questions or admission updates, read
`/opt/hatch/bin/feature-request show --help` and show the same kind and subject.
An `addressed` request means the user has joined the group. Other states do
not confirm membership. Do not promise a timeline or other notification channels.

对于状态查询或录取更新，阅读 `/opt/hatch/bin/feature-request show --help` 并以相同的 kind 和 subject 查询。状态为 `addressed` 的请求表示用户已加入该群体。其他状态都不能确认成员身份。不要承诺时间表或其他通知渠道。

Before withdrawing a request, read `/opt/hatch/bin/feature-request delete --help`
and follow its approval steps. Do not claim that withdrawal changes membership.  
A later join request needs the user's fresh, explicit instruction.

在撤回请求之前，阅读 `/opt/hatch/bin/feature-request delete --help` 并遵循其确认步骤。不要声称撤回会改变成员身份。之后的加入请求需要用户重新给出明确的指示。

【评论】该文件把"提交动作"严格限定在用户明确要求时，并区分"咨询/想试用某功能"与"正式加入请求"，属于防止代理过度代理用户意图的设计；同时禁止承诺录取或时间表，避免产生无法兑现的预期。
