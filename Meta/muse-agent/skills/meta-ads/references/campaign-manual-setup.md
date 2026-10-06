<!-- BILINGUAL-EN-ZH -->
# Campaign setup the advertiser completes in Ads Manager / 由广告主在 Ads Manager 中自行完成的广告系列设置

Read this when a write to the account was rejected as not available for this ad
account — whether a tool returned it (`writes.md`) or the advertiser quotes it
from an earlier attempt — or when the advertiser asks to set a campaign up
themselves in Ads Manager. It replaces creation, not planning: the plan is
still yours to get right, and they do the clicking.

当对账户的写入被以"该广告账户不可用"为由拒绝时——无论是工具返回了该信息（`writes.md`），还是广告主引用了此前某次尝试中的信息——或者当广告主要求自己在 Ads Manager 中搭建广告系列时，阅读本文档。它取代的是创建环节，而不是规划环节：把计划做对仍是你的责任，点击操作则由他们完成。

## Explaining the rejection / 解释拒绝信息

It means making changes to that account from here was not available when it
was returned. The cause is not visible from here, so do not guess one:
business verification, payment methods, admin roles and reconnecting do not
change it, and there is no onboarding checklist or step they can complete,
whatever an older error message says. Do not search for one. A missing payment
method still stops delivery once they publish, so name it as that, never as
the cause. Say it once, then move straight to what you can do: plan the
campaign in full. Never offer to hold the plan until access returns.

它的含义是：在该信息返回时，从这里对该账户进行变更不可用。原因从这里不可见，因此不要猜测：企业验证、支付方式、管理员角色和重新授权都不会改变这一状态，也不存在他们可以完成的入门清单或步骤，无论旧版错误信息怎么说。不要去搜索原因。支付方式缺失仍会在他们发布后中断投放，因此要把它作为这一独立问题指出，绝不说成是拒绝的原因。说明一次，然后直接转向你能做的事：完整规划该广告系列。绝不要提出"等权限恢复后再保留该计划"。

【评论】此段明确禁止代理猜测账户被拒原因并推荐"修复清单"，属于防止幻觉归因与无效用户操作的防护性设计。

## Plan in full, create nothing / 完整规划，不做任何创建

Which way you arrived decides what happens next:

你以何种方式走到这一步，决定了接下来会发生什么：

- **After a tool returned the account-level rejection for this account in this
  conversation**, that account is plan-only for the rest of the conversation.
  Send it no write of any kind again, and ask for no creation approval —
  nothing here can be approved into existence.
  **如果在本对话中工具已对该账户返回过账户级拒绝**，则该账户在对话剩余部分中处于仅规划状态。不再向它发送任何形式的写入，也不请求创建批准——这里没有任何东西可以通过批准而存在。
- **When the advertiser quotes the rejection from an earlier attempt**, it is
  not proof about this account now: eligibility can change, and it may have
  been another account. Plan in full and offer both the setup guide and
  creating it here. If they choose creation, follow `campaign-execution.md`;
  the first write either succeeds or returns the rejection, which then applies
  the lock above.
  **当广告主引用的是此前某次尝试中的拒绝信息时**，它不能证明该账户现在仍被拒：资格可能变化，而且当时可能是另一个账户。完整规划，并同时提供设置指南和在此直接创建两种选项。如果他们选择创建，遵循 `campaign-execution.md`；第一次写入要么成功，要么返回拒绝信息，一旦返回即套用上述锁定。
- **When the advertiser only chose to build it themselves**, the account can
  still be written. Make no write for this campaign unless they ask, and if
  they change their mind, go back to `campaign-execution.md`. Every other write
  they ask for follows `writes.md` as usual.
  **当广告主只是选择自行搭建时**，该账户仍然可写。除非他们提出要求，否则不要为该广告系列执行任何写入；如果他们改变主意，回到 `campaign-execution.md`。他们请求的其他写入照常遵循 `writes.md`。

Either way, every read still works, so planning keeps its full weight: run
`campaign-creation.md` and the stage references it routes to exactly as for a
created campaign — identity, research, objective, `campaign-targeting.md`,
`campaign-budget.md`, `campaign-creative.md` — and stop where
`campaign-execution.md` would begin. Never run `render-campaign-success` or
describe anything as created, staged or paused; nothing exists until they
publish it.

无论哪种情况，所有读取操作仍然可用，因此规划的完整分量得以保留：完全按照创建广告系列的流程运行 `campaign-creation.md` 及其路由到的各阶段参考文档——身份、调研、目标、`campaign-targeting.md`、`campaign-budget.md`、`campaign-creative.md`——并在本应开始执行 `campaign-execution.md` 的位置停下。绝不运行 `render-campaign-success`，也绝不把任何东西描述为已创建、已暂存或已暂停；在他们发布之前，什么都不存在。

If the rejection arrived at final creation, the reviewed plan is already
complete. Turn it into the setup guide in the same response; do not replan it.

如果拒绝发生在最终创建时，已审查的计划已经完成。在同一条回复中把它转化为设置指南；不要重新规划。

## Creative they can upload / 他们可自行上传的创意

Write the finished copy — primary text, headline, description, call to action —
per ad. An image generated under `campaign-creative.md` is an asset they upload
themselves, so say which ad it belongs to. Existing Page posts or account media
are referenced by name and where they will find them, never by id.

为每条广告写好成品文案——主文本、标题、描述、行动号召。在 `campaign-creative.md` 下生成的图片是需他们自行上传的素材，因此要说明它属于哪条广告。既有主页帖子或账户素材以名称及其所在位置来引用，绝不以 id 引用。

## The setup guide / 设置指南

One section per level, in the order Ads Manager builds a campaign: **Campaign**,
then each **Ad set**, then each **Ad**. Under each, list only the settings the
plan decided, as the advertiser will see them, with the value to choose:

每个层级一节，按 Ads Manager 搭建广告系列的顺序排列：先是 **Campaign**（广告系列），然后每个 **Ad set**（广告组），再是每个 **Ad**（广告）。在每一节下，只列出计划已决定的设置，按广告主将看到的样式呈现，并给出应选择的值：

- Campaign: objective, name, Special Ad Category when one applies, and whether
  the budget sits on the campaign.
  Campaign：目标、名称、适用时的特殊广告类别（Special Ad Category），以及预算是否设在广告系列层级。
- Ad set: conversion location and performance goal, budget and schedule,
  locations, age, and the resolved interests and languages by name, placements.
  Ad set：转化位置与效果目标、预算与排期、地区、年龄、按名称解析出的兴趣和语言，以及版位。
- Ad: Facebook Page and Instagram account, format, media, copy, destination URL,
  and the dataset or pixel if the plan tracks conversions.
  Ad：Facebook 主页和 Instagram 账户、格式、素材、文案、落地页 URL，以及计划追踪转化时所需的数据集或像素（pixel）。

Use the labels from `response-style.md` — `Sales`, `Highest volume` — never API
values or ids. Do not describe screens, menus or button positions you have not
verified; the setting name and its value are what they need. Say that spend
starts when they publish, and that choosing Advantage+ defaults Ads Manager
offers can change what the plan chose.

使用 `response-style.md` 中的标签——如 `Sales`、`Highest volume`——绝不使用 API 值或 id。不要描述你未验证过的界面、菜单或按钮位置；设置名称及其取值才是他们需要的东西。说明花费从他们发布时开始，并说明选择 Ads Manager 提供的 Advantage+ 默认项可能改变计划所选的内容。

Finish with one `muse.create_options` menu of what you can still do: adjust the
plan, rework a creative, or check the campaign against the plan once they have
published it — that check is a read, and it works. Offer to create it for them
unless a tool rejected a write to this account in this conversation.

最后给出一个 `muse.create_options` 菜单，列出你仍可做的事：调整计划、重做某个创意，或在他们发布后对照计划检查该广告系列——该检查是读取操作，并且可用。主动提出可以代他们创建，除非在本对话中已有工具对该账户的写入作出过拒绝。
