<!-- BILINGUAL-EN-ZH -->

# Referrals and invite codes / 推荐与邀请码

Muse invitations can include a share link and an invite code. Sharing an
invitation, joining Muse, and redeeming a code are separate steps. Opening a
link is not confirmation that a code was redeemed or a reward was credited.
An invitation does not give anyone access to the sender's chats or agent.

Muse 邀请可以包含分享链接和邀请码。分享邀请、加入 Muse、兑换邀请码是彼此独立的步骤。打开链接并不代表邀请码已被兑换或奖励已入账。邀请不会让任何人访问发送者的聊天或智能体。

## Sharing an invitation / 分享邀请

The mobile Invite button is at the top right of the chat header (a gift icon
on iOS) and opens the invite share sheet. The sender can share the invitation
provided there. An invitation supplied in chat carries the sender's exact
code; use the Invite button's share sheet for the link.

移动端的邀请按钮位于聊天标题栏右上角（iOS 上是礼物图标），点开会打开邀请分享面板。发送者可以从那里分享邀请。聊天中提供的邀请携带发送者的精确邀请码；链接请使用邀请按钮的分享面板获取。

A generic Muse homepage or a guessed join URL is not a replacement for the
sender's supplied invite link. A code can be entered in the redemption form;
it is not necessary to turn a bare code into a URL to redeem it.

通用的 Muse 主页或猜测出的加入 URL 不能替代发送者提供的邀请链接。邀请码可以直接输入兑换表单；把裸邀请码拼成 URL 并不是兑换所必需的。

## Redeeming a code / 兑换邀请码

On iPhone and Android, open the Muse app and go to **Settings > Redeem
token**. Enter the friend's invite code there. This option is available for
the first 48 hours after the user joins Muse. A confidential VM has no
redemption entry on any platform; see the note below.

在 iPhone 和 Android 上，打开 Muse 应用，进入 **Settings > Redeem
token**。在那里输入朋友的邀请码。此选项仅在用户加入 Muse 后的前 48 小时内可用。机密虚拟机（confidential VM）在任何平台上都没有兑换入口；参见下文说明。

On the Muse website at muse.ai, the entry is **Settings > General > Usage >
Redeem invite code** for accounts with redemption available. The recipient
enters the friend's code in that form and confirms. The entry disappears
after the account has confirmed redemption, including on another device.
Invitation sharing and code redemption have separate availability; seeing
an Invite button does not establish whether that account can redeem a code.

在 muse.ai 上的 Muse 网站中，对于可兑换的账户，入口是 **Settings > General > Usage >
Redeem invite code**。接收者在该表单中输入朋友的邀请码并确认。账户确认兑换后，该入口即消失，包括在另一台设备上确认的情况。邀请分享与邀请码兑换的可用性相互独立；看到邀请按钮并不能说明该账户能否兑换邀请码。

Settings in these directions belongs to Muse, not the phone's system
Settings or a messaging app such as WhatsApp.

这里所说的设置指的是 Muse 应用内的设置，而不是手机的系统设置，也不是 WhatsApp 之类的消息应用。

## Offer terms and outcomes / 活动条款与结果

The current invitation and redemption screen supply the applicable offer
terms, including reward amounts, remaining invitations, and other
eligibility conditions. A remembered offer for one account does not
establish another account's terms. A reward amount missing from the
invitation is unknown, not a promise of a particular number of tokens.

当前的邀请与兑换界面会提供适用的活动条款，包括奖励金额、剩余邀请次数及其他资格条件。为某个账户记住的活动条款不适用于其他账户。邀请中缺失的奖励金额属于未知信息，而不是对某个具体代币数量的承诺。

The redemption result establishes whether the code was accepted. A pending
result is not confirmed redemption or credited usage. The web form can report
an invalid, used-up, or revoked code, an already-redeemed account, an expired
redemption window, too many attempts, or a temporary failure. The displayed
result is the basis for explaining a refusal; a code's appearance alone does
not establish validity. Current subscription and usage questions use the
`subscription_status` skill; this document does not expose account balances
or provide a way for the agent to validate or redeem a code.

兑换结果才能确定邀请码是否被接受。待定（pending）的结果不代表兑换已确认或用量已入账。网页表单可能报告：邀请码无效、已用尽或已被吊销、账户已兑换过、兑换窗口已过期、尝试次数过多，或临时性失败。应以显示的结果为依据来解释拒答原因；仅凭邀请码的外观不能确定其有效性。当前的订阅与用量问题应使用 `subscription_status` 技能；本文档不暴露账户余额，也不为智能体提供验证或兑换邀请码的途径。

【评论】此节刻意划清文档能力边界：智能体既无余额查询途径也无兑换手段，只能依据界面显示的结果转述，避免凭空承诺奖励。

## Missing options or failed lookups / 选项缺失或查询失败

An unavailable page does not establish that a code is invalid, that redemption
requires a link, or that no code-entry field exists. On mobile, Redeem token
is only available during the first 48 hours after joining. On a confidential
VM the redemption entry is absent by design, on every platform; that missing
row is the expected state, not an account or rollout problem. A missing Settings
row alone does not establish the account's age or prove a rollout delay.
The user's platform, visible screen, and any displayed error help narrow down
what happened. Where those facts do not establish the answer, the cause is
unknown. Muse's Help & Support options are in `~/docs/client-surfaces.md`.

页面不可用并不能证明邀请码无效、兑换必须通过链接、或不存在邀请码输入框。在移动端，Redeem token 仅在加入后的前 48 小时内可用。在机密虚拟机上，兑换入口在所有平台上都是有意不提供的；该条目缺失是预期状态，不是账户或灰度发布的问题。仅凭设置中缺少某个条目，不能确定账户的注册时长，也不能证明存在灰度发布延迟。用户的平台、可见屏幕及任何显示的错误信息有助于缩小问题范围。若这些事实仍无法确定答案，则原因属于未知。Muse 的帮助与支持选项见 `~/docs/client-surfaces.md`。
