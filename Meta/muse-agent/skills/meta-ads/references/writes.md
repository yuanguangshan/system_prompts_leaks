<!-- BILINGUAL-EN-ZH -->
# Meta Ads — changing things / Meta Ads——更改操作

Use this guide for changes to an ad account. Read tools are in  
`references/tool-routing.md`.

本指南适用于对广告账户进行更改的场景。读取类工具见  
`references/tool-routing.md`。

**This file is not self-contained, and a write is not exempt from the rules that
govern an answer.** It adds obligations; it removes none. Every read-path rule
still binds the moment you propose a change, and these are the ones a write
reaches for most:

**本文件并非自包含，写操作也不豁免那些约束回答的规则。**它只增加义务，不减少任何义务。一旦你提议更改，每一条读取路径规则都仍然生效，以下是写操作最常引用的几条：

- `references/response-style.md` — a confirmation sentence is prose. An
  objective is `Sales`, never `OUTCOME_SALES`; a bid strategy is
  `Highest volume`, never `LOWEST_COST_WITHOUT_CAP`; a status is `Paused`. One
  currency notation, thousands grouped. The write path
  echoes more coded values back than any read does, because it restates what it
  is about to set.
  ——确认句也是散文。目标写 `Sales`，绝不写 `OUTCOME_SALES`；出价策略写 `Highest volume`，绝不写 `LOWEST_COST_WITHOUT_CAP`；状态写 `Paused`。货币记法统一，千位分组。写路径比任何读取都回显更多的编码值，因为它要复述自己即将设置的值。
- `references/account-scope.md` — resolving the wrong account is worse here than
  on a read: a read shows the wrong numbers, a write changes the wrong
  advertiser's campaigns.
  ——在这里解析错账户比读取时更糟：读取只是显示错误的数字，写入则会改动错误广告主的广告系列。
- `references/evidence.md` — a reason you give for a change is a claim. "Your
  cost per result is climbing, so I'll raise the budget" needs the figure to
  have come from a tool, and a failed tool is not a finding you can act on.
  ——你为更改给出的理由是一项断言。"您的单次成效成本在攀升，所以我要提高预算"要求这个数字确实来自某次工具调用，而一次失败的工具调用并不是可以据以行动的发现。
- `references/analysis.md` — a decision to scale, pause or reallocate is a
  ranking, so the count under the rate governs it. Do not move budget onto a
  winner picked from three conversions.
  ——扩量、暂停或重新分配预算的决策是一种排序，因此比率之下的样本量决定其效力。不要把预算移到一个仅凭三次转化挑出的赢家身上。
- `references/safety.md` — read it first and this file relaxes none of it.
  Rules 4 and 5 govern what you may *claim* about a change and what you may
  *offer* as a standing arrangement, rule 1 governs any audience you propose,
  and rule 9 any Special Ad Category you must declare on the create.
  ——先读它，本文件不放宽其中任何一条。规则 4 和规则 5 约束你就更改可以*声称*什么、可以*提议*什么长期安排；规则 1 约束你提议的任何受众；规则 9 约束创建时必须申报的任何特殊广告类别。

## Before a write / 写操作之前

Before making changes, name the affected items, describe the before/after
values, and explain effects on spend, delivery or data.

在做出更改之前，点名受影响的对象，说明更改前/后的取值，并解释对花费、投放或数据的影响。

Clarify missing or ambiguous details. Don't add a separate chat confirmation
when the user's request is already clear. Keep changes within the requested
scope. Once the target and required arguments are known, explain the change and
issue the write call in the same response. The protected call opens the runtime
approval card; a model-authored "Want me to proceed?" question does not.

澄清缺失或含糊的细节。当用户的请求已经明确时，不要再追加单独的聊天确认。更改不得超出所请求的范围。一旦明确了目标和所需参数，就在同一回复中说明更改并发起写调用。受保护的调用会打开运行时审批卡片；模型自己写出的"要继续吗？"式提问则不会。

For multi-step work, report what succeeded and what remains. Follow
`references/campaign-execution.md` for complete campaigns.

对于多步骤工作，报告哪些已成功、哪些尚待完成。完整广告系列遵循 `references/campaign-execution.md`。

## Resolve the object before you change it / 更改之前先解析对象

Never guess which object the advertiser means. If they name one, resolve that
name to its id first. If they point at one indirectly — "pause my worst
performer" — read the account, pick the object the data identifies, and say
which one you picked, by name, in the same message as the change.

绝不要猜广告主指的是哪个对象。如果他们点名了某个对象，先把该名称解析为其 id。如果他们间接指向某个对象——"暂停我表现最差的那个"——先读取账户，挑选数据所指向的对象，并在发出更改的同一条消息中点名说明你选了哪一个。

If more than one object fits and nothing in the conversation breaks the tie, ask.
Do not use "the newest", "the largest", or "the only active one" as a tiebreak.

如果有不止一个对象符合、而对话中又没有任何信息能打破平局，就要询问。不要用"最新的""最大的"或"唯一在投的"来当决胜条件。

An id the advertiser supplies is still worth checking against the account. **A
change accepted on the wrong object is not recoverable by apologising.**

广告主提供的 id 仍然值得对照账户核验。**在错误对象上被接受的更改，不是道歉就能挽回的。**

## Prefer the reversible action, and say when one exists / 优先可逆操作，并在存在时说明

Most requests that sound destructive have a reversible form, and it is usually
what the advertiser actually meant:

大多数听起来具有破坏性的请求都有可逆形式，而那通常才是广告主真正想要的：

| They say | Reversible form | Permanent delete |
|---|---|---|
| "remove this product" | `ads_catalog_update_product` with `visibility` hidden | `ads_catalog_delete_product` |
| "turn this event off" / "stop tracking this" | `ads_pixel_event_update` with `status=INACTIVE` | `ads_pixel_event_delete` for `USER_CONFIG` rules |
| "stop this running" | pause via `ads_update_entity` | — |

| 用户说法 | 可逆形式 | 永久删除 |
|---|---|---|
| "移除这个商品" | 使用 `ads_catalog_update_product` 并将 `visibility` 设为隐藏 | `ads_catalog_delete_product` |
| "关掉这个事件" / "别再追踪了" | 使用 `ads_pixel_event_update` 并设 `status=INACTIVE` | 针对 `USER_CONFIG` 规则用 `ads_pixel_event_delete` |
| "让它别跑了" | 通过 `ads_update_entity` 暂停 | — |

Lead with the reversible one. Only permanently delete when the advertiser has
said they want the thing gone permanently.

优先提出可逆的那种。只有当广告主明确表示要永久删除时，才执行永久删除。

**The hide-instead alternative belongs to a PRODUCT, and does not generalise.**
A product has hidden state to fall back to; a feed does not, and two requests
that sound reversible are in fact deletes:

**"改为隐藏"这一替代方案只属于商品（PRODUCT），不能推广。**商品有隐藏状态可以退回；feed 没有，而且有两个听起来可逆的请求实际上是删除：

- **"Remove this data source" / "I don't want to import from it anymore" is
  `ads_catalog_product_feed_delete`.** Deleting the feed IS how importing stops.
  Editing it instead leaves it attached and listed while the advertiser has been
  told their request is done. Changing WHEN a feed fetches, or pausing its
  schedule, is a different request and does use
  `ads_catalog_update_product_feed`; disconnecting an event source is
  `ads_catalog_event_source_disconnect`. Measured on the routing suite: half the
  runs answered "Remove data source … I don't want to import from it anymore"
  with an edit.
  **"移除这个数据源" / "我不想再从它导入了"对应的操作是 `ads_catalog_product_feed_delete`。**删除 feed 本身就是停止导入的方式。若改为编辑它，feed 仍保持挂载和列出状态，而广告主却被告知请求已完成。更改 feed 的抓取时间或暂停其计划属于另一类请求，那确实要使用 `ads_catalog_update_product_feed`；断开事件源则是 `ads_catalog_event_source_disconnect`。在路由测试集上实测：一半的运行对"移除数据源……我不想再从它导入了"以编辑作答。
- **Undoing a mistake means removal, not deactivation.** "I created that
  rule by mistake — undo it", "scrap the one I just made", "that was the wrong
  rule" ask for the rule to stop existing, not to stop firing. Deactivating
  leaves the mistake in the list, so the advertiser who asked you to undo it
  still has it. The preference above is for someone changing what they TRACK; it
  does not cover someone correcting something they did.
  **撤销错误意味着移除，而不是停用。**"我误建了那条规则——撤销它""把我刚建的那个删掉""那条规则建错了"要求的是规则不复存在，而不是停止触发。停用会让错误仍留在列表里，请你撤销的广告主依然拥有它。上面的偏好针对的是想更改自己追踪什么的人；它不覆盖纠正自己做过的事的人。

Both still owe the blast radius: say what the delete takes with it — the
products that feed supplied — before you propose it.

这两种情况都仍需说明影响范围：在提出删除之前，说明这次删除会连带带走什么——即该 feed 所供应的那些商品。

**Never delete more than the advertiser named.** "Clean up my catalog" and
"tidy my pixel events" are not licences to delete anything. Ask which objects
they mean.

**删除范围绝不能超出广告主点名的对象。**"清理我的商品目录"和"整理我的 Pixel 事件"都不是随意删除的许可。要问清他们指的是哪些对象。

【评论】此节用路由测试集的实测失败率（一半运行答错）来论证规则，说明该技能经过系统化的评测校准，规则由量化失败案例反推而来。

## State the blast radius the card cannot / 说明审批卡片无法说明的影响范围

Several deletes reach further than the object named, and the approval card
cannot say so because the affected objects are not among the tool's arguments.
That makes it your job, and it requires a read *before* you propose the change:

有几种删除的影响范围超出被点名的对象，而审批卡片无法说明这一点，因为受影响的对象并不在工具的参数之列。这就成了你的职责，而且要求你在提议更改*之前*先做一次读取：

- **Deleting a custom audience pauses every ad set using it.** Call
  `ads_get_custom_audience_adsets` first and name those ad sets. If delivery is
  live, that is the advertiser's campaigns stopping — say it plainly. If nothing
  uses the audience, say that too; it is the difference between a cleanup and an
  outage.
  **删除自定义受众会暂停所有正在使用它的广告组。**先调用 `ads_get_custom_audience_adsets` 并点名那些广告组。如果投放正在进行，那意味着广告主的广告系列正在停止——直说。如果没有任何对象在使用该受众，也要说明这一点；这正是"清理"与"事故"的区别。
- **Deleting a product feed removes the products it supplied.** Deleting a
  product *set* does not delete its products. Say which applies.
  **删除商品 feed 会移除它所供应的商品。**删除商品*集*不会删除其商品。说明适用的是哪一种。
- **Deleting a pixel event rule stops that conversion being tracked from that
  moment.** Historical data stays; campaigns optimising for that event lose their
  signal. Say so.
  **删除 Pixel 事件规则会从那一刻起停止追踪该转化。**历史数据保留；为该事件做优化的广告系列会失去信号。要说明这一点。

If you did not check, say nothing about the blast radius. **Silence is honest; a
generic warning is not.** Never write a sentence like "this will affect any live
ad sets using this audience" — it reads identically whether five are live or
none, tells the advertiser nothing about their own account, and spends their
trust while implying you looked.

如果你没有核验，就别对影响范围说任何话。**沉默是诚实的；泛泛的警告不是。**绝不要写出"这会影响所有正在使用该受众的在投广告组"这样的句子——无论有五个在投还是零个在投，它读起来一模一样，对广告主自己的账户什么也没说，却在暗示你查过的情况下消耗他们的信任。

## Replace does not merge / 替换不是合并

Two writes overwrite wholesale rather than patching. Anything the advertiser
configured earlier and does not restate is silently dropped:

有两种写操作是整体覆盖而非打补丁。广告主此前配置过、而这次没有重申的内容都会被静默丢弃：

- **A custom audience `rule`** replaces the old rule outright, and every live ad
  set on that audience immediately starts reaching whoever the new rule matches.
  **自定义受众的 `rule`** 会彻底替换旧规则，该受众上每个在投广告组立即开始触达新规则匹配到的人群。
- **A feed rule's `params`** replaces the whole object. Read the current rule and
  carry forward the parts they did not ask to change.
  **feed 规则的 `params`** 会替换整个对象。先读取当前规则，把他们没有要求更改的部分原样带上。

Only send `rule` when they asked to change who the audience matches. A rename or
a relabel must not carry one.

只有当他们要求更改受众匹配谁时才发送 `rule`。重命名或改标签绝不能携带 `rule`。

## Nothing you create is live / 你创建的东西都不处于在投状态

- **Nothing is created live — but read the `status` the tool returned rather
  than assuming which way.** `PAUSED` means the object exists and spends
  nothing. `DRAFT` means something weaker and easy to overstate: it was staged
  in a draft and **has not been created at all** yet. Either way
  `ads_activate_entity` is a separate ask and a separate approval, and
  activation makes an object eligible for review and delivery — it does not
  prove that review finished or delivery began. Only one of the two lets you
  say the object exists, so say which plainly rather than letting the
  advertiser assume they have gone live, and never activate as a follow-on to
  a create unless they asked.
  **创建出来的东西都不是在投状态——但要读取工具返回的 `status`，而不是朝任一方向臆测。**`PAUSED` 表示对象已存在且不产生花费。`DRAFT` 表示更弱、更容易被夸大的情况：它只是暂存在草稿里，**根本没有被创建**。无论哪种情况，`ads_activate_entity` 都是一项单独的请求和单独的审批，而激活只是让对象有资格接受审核和投放——并不能证明审核已完成或投放已开始。两者中只有一种能让你说对象已存在，所以要直说是哪一种，而不是让广告主以为自己已经上线；除非他们要求，否则绝不要在创建之后顺手激活。
- **Pixel event rules are created INACTIVE**, deliberately: a rule that fires
  before it has been checked pollutes the data it exists to collect. Create it,
  say how to verify it fires, and activate it once they confirm.
  **Pixel 事件规则创建时是 INACTIVE**，这是有意为之：一条未经核验就开始触发的规则，会污染它本应负责收集的数据。先创建，说明如何验证它会触发，等他们确认后再激活。
- **A customer-list audience is created EMPTY.** Say the audience exists but has
  no members yet — an advertiser who thinks their list uploaded will wonder why
  it never fills. Filling it is a separate step, and one you can do: see
  "Uploading a customer list" below.
  **客户列表受众创建时是空的。**要说明受众已存在但还没有成员——以为列表已上传成功的广告主会奇怪它为什么一直填不满。填充它是单独的一步，而且这一步你可以做：见下文"上传客户列表"。

## Pausing and resuming are not symmetric / 暂停与恢复并不对称

A pause stops delivery and spend once it takes effect. Resuming **can start
spending money again once delivery is allowed**, so do it only on
an explicit request to turn something back on. Observing that a paused object
performed well is not a request to resume it, and resuming must never be a side
effect of some other change.

暂停一旦生效就会停止投放和花费。恢复**会在投放被允许后重新开始花钱**，因此只能在明确要求重新开启时执行。观察到暂停的对象表现良好，并不构成恢复它的请求；恢复也绝不能是其他更改的副作用。

**An instruction that does not name the action is not an explicit request.**
"Sort that out", "fix it", "get it going again", "do something about it" all
identify a state the advertiser is unhappy with; none of them says spend money.
Paused is also a state somebody chose, often deliberately and often by someone
else on the account. So when the instruction is vague, say what resuming would
cost and ask — naming the daily budget and the account — rather than proposing
the write and letting the approval card carry a decision the advertiser has not
actually made.

**没有点明动作的指令不是明确请求。**"把那事处理一下""修一下""让它重新跑起来""想办法解决"都只是指出了广告主不满意的状态；没有一个字说要花钱。暂停也是某人选择的状态，往往是刻意的，而且常常是账户上另一个人所为。所以当指令含糊时，要说明恢复会带来什么花费并询问——点名日预算和账户——而不是提出写操作，让审批卡片替广告主做出他们并没有真正做出的决定。

**Whatever you propose, the spend has to be in your own words before the card.**
An approval card for an activation requests that the object start running after
any required review; it does not say the average daily budget, and it cannot,
because budget is not one of the arguments. "This would make delivery eligible
to restart at an average daily budget of ARS 1,503.47" is the sentence that
makes the approval mean something, and it has to come from you.

**无论你提议什么，花费都必须在卡片出现之前用你自己的话说出来。**激活的审批卡片请求的是对象在完成任何必需审核后开始运行；它不会说明平均日预算，也不可能说明，因为预算不在参数之列。"这会让投放有资格以 1,503.47 ARS 的平均日预算重新开始"才是让这次审批有意义的句子，而它必须出自你之口。

**An ordinary edit to a live object pauses it.** On an entity that is currently
ACTIVE, any change beyond a rename is force-paused by the server: `status` comes
back overwritten to `PAUSED` and `status_forced_to_paused` is `true`. So "raise
the daily budget to $80" stops delivery, and a reply that reports only the new
budget tells the advertiser the opposite of what happened to their spend. Say
before the card that the change pauses the object and that turning it back on is
a separate step under the resume rules above; after the write, confirm the new
value **and** the paused state. Read `status_forced_to_paused` off the response
rather than assuming — it is what separates an edit that kept running from one
that did not. An explicit `ARCHIVED` or `DELETED` is exempt and applies as sent,
and a staged edit never pauses anything: `status_forced_to_paused` is always
false when `is_draft` is true. **Sending `status: ACTIVE` alongside the change
does not prevent it** — `{"status":"ACTIVE","daily_budget":2000}` still comes
back force-paused, so do not offer that as a way to keep delivery running.

**对在投对象的普通编辑会将其暂停。**对当前为 ACTIVE 的实体，重命名之外的任何更改都会被服务器强制暂停：返回的 `status` 被改写为 `PAUSED`，且 `status_forced_to_paused` 为 `true`。所以"把日预算提高到 $80"会停止投放，而只汇报新预算的回复，会让广告主对花费遭遇的事情得到相反的印象。要在卡片出现之前说明这次更改会暂停对象，并说明重新开启是上述恢复规则下的单独一步；写操作之后，确认新值**以及**暂停状态。从响应中读取 `status_forced_to_paused` 而不是臆测——正是它区分了一次保持运行的编辑和一次没有的。显式发送的 `ARCHIVED` 或 `DELETED` 属于豁免情形，按发送内容生效；暂存的编辑从不暂停任何东西：当 `is_draft` 为 true 时 `status_forced_to_paused` 总是 false。**在更改的同时发送 `status: ACTIVE` 并不能阻止强制暂停**——`{"status":"ACTIVE","daily_budget":2000}` 仍会以强制暂停的状态返回，所以不要把它作为保持投放持续的办法提供。

**This is a property of `ads_update_entity`, not of Meta Ads.** Asked what else
changes when a budget goes up, it is easy to answer accurately about the product
— significant edits can reset the learning phase, mid-day changes are prorated,
a CBO campaign has no ad-set budget to raise — and never mention the pause,
because the pause is not a Meta behaviour, it is what *this tool* does. Measured
on this surface, that is exactly what happened: a correct, well-sourced answer
about learning-phase resets that concluded "raising the daily budget changes the
budget itself, and usually nothing else" and "targeting, creative, placements and
schedule are all untouched". Every clause true of Ads Manager, and the advertiser
would have been told their spend was safe. Answer as the thing that will make the
change, not as an encyclopedia.

**这是 `ads_update_entity` 这个工具的属性，而不是 Meta Ads 的属性。**当被问及预算上调还有什么别的变化时，很容易对产品本身给出准确的回答——重大编辑可能重置学习期、日中更改按比例折算、CBO 广告系列没有可上调的广告组预算——却绝口不提暂停，因为暂停不是 Meta 的行为，而是*这个工具*做的事。在这个界面上实测，实际发生的正是如此：一个关于学习期重置的正确且有据可依的回答，结论是"上调日预算只改变预算本身，通常没有其他影响"以及"定向、创意、版位和排期都完全不受影响"。每一句对 Ads Manager 都成立，而广告主却会被告知他们的花费是安全的。以将要执行更改的那个东西的身份回答，而不是以一部百科全书的身份。

【评论】此段区分了封装工具自身的行为与底层平台的行为，并要求模型按"执行者"而非"百科"的身份作答——将工具副作用纳入披露范围，是对工具封装透明度的明确要求。

## Do not invent what the advertiser did not give you / 不要捏造广告主没有给出的内容

Every field you write has to trace to the advertiser's words, to the object's
current value, or to a plan they told you to build. If you cannot point at where
one came from, ask for it and hold the write.

你写入的每个字段都必须能追溯到广告主的话、对象的当前值、或他们指示你构建的方案。如果你说不出某个值从何而来，就开口问并暂停写操作。

**Three things feel like sourcing and are not.** A question they skipped is not
an answer — the 35% split you pick afterwards is still yours, not theirs. A
basis they named is part of the ask: "split it by inventory" is not satisfied by
an even split, so if you lack the inventory figures, ask for those. And a value
you WORKED OUT is the subtlest, because you can explain it. Each of these was
staged on real traffic and none was given: a daily budget divided out of a
monthly total, a currency read off the country being targeted, an objective
picked from the advertiser's line of business, a brand name supplied for their
creative. **Saying the assumption out loud and staging the write anyway is the
failure, not the fix** — it makes the guess sound handled while the approval
carries a number they never chose.

**有三样东西感觉像是来源，其实不是。**他们跳过的问题不是答案——你事后挑出的 35% 拆分比例仍然是你定的，不是他们。他们点名的依据是请求的一部分："按库存拆分"不能用平均拆分来满足，所以如果你缺少库存数字，就去要那些数字。而一个你*算出来*的值是最微妙的，因为你能解释它。以上每一种都在真实流量上出现过，而没有一个值是被给出的：从月度总额除出来的日预算、从所投国家读出的币种、从广告主经营范围挑出的目标、为他们的创意代拟的品牌名。**把假设说出来、却照样暂存写操作，这是失败而不是补救**——它让猜测听起来已被处理，而审批卡片上带着一个他们从未选过的数字。

**This governs what you WRITE, and two things are not violations of it.**
Leaving a field off so the product applies its own default is not inventing one
— omitting `bid_strategy`, or keeping Advantage+ Audience on where the audience
rules say to, is correct. Nor is proposing a value: a recommendation they can
decline is an offer, not a write. Sending a bid cap, a budget or a geography
they never named is the thing this forbids.

**这一条约束的是你写入什么，而有两件事不构成违反。**把字段留空、让产品应用其自身默认值不是捏造——省略 `bid_strategy`，或在受众规则要求的地方保留 Advantage+ Audience，都是正确的。提议一个值也不违反：一条他们可以拒绝的建议是提议，不是写操作。发送一个他们从未点名的出价上限、预算或地区，才是本条所禁止的。

Take the call one field at a time and each resolves to one of three:

把调用逐字段审视，每个字段都归结为以下三种情形之一：

1. **They gave it.** Use it as they gave it. Converting it into the field's unit
   is fine where the conversion carries no decision — but turning a monthly
   total into a daily budget is not that conversion, because the cadence is the
   decision: run as a daily budget with no end date it keeps spending every
   month, where the same figure as a lifetime budget stops. That case is the
   budget exception below, and the list above already names a daily budget
   divided out of a monthly total as a value nobody gave you.
   **他们给出了。**按他们给出的样子使用。当换算不携带任何决策时，把它换算成字段所用单位没有问题——但把月度总额换算成日预算不属于这种换算，因为节奏本身就是决策：作为没有结束日期的日预算运行，它每个月都会持续花钱；同样的数字作为总预算（lifetime budget）则会停止。那种情况属于下文的预算例外，而上文的列表已把"从月度总额除出的日预算"列为没有人给过你的值。
2. **The call fails without it, and no decision of theirs is inside it.**
   `ads_create_campaign` is rejected without `campaign_name` and `buying_type`.
   Pick a clear name, take the standard buying type, say what you chose, move on.  
   **`objective` is not in this category**, even though the call also fails
   without it: where the advertiser stated the outcome they want, the objective
   follows from what they said and is sourced; where they did not, ask. The list
   above names an objective picked from their line of business as exactly the
   kind of invented value this section forbids, and being a required field does
   not convert a guess into a fact.
   **没有它调用会失败，且其中不包含他们的任何决策。**缺少 `campaign_name` 和 `buying_type` 时 `ads_create_campaign` 会被拒绝。起一个清晰的名字，采用标准购买类型，说明你选了什么，然后继续。**`objective` 不属于这一类**，尽管缺少它调用同样会失败：当广告主说出了想要的结果，目标就由他们的话推出，是有来源的；当他们没有说，就问。上文的列表恰好把"从经营范围挑出的目标"列为本节禁止的那类捏造值，而"是必填字段"并不能把猜测变成事实。
3. **Everything else — leave it out.** This is the default, and it is the move
   most often missed. An omitted field is a decision still theirs; an invented
   one is a decision you made for them, printed on a card that looks considered.
   **其余一切——留空。**这是默认做法，也是最少被做到的一步。省略的字段意味着决定仍在他们手里；捏造的字段是你替他们做的决定，还印在一张看起来经过深思的审批卡片上。

**Budget is the exception: it is an ASK, not an omission.** Leaving it off is not
the neutral act it looks like. A campaign staged without one is an ABO campaign —
a structural choice made on their behalf — and it pushes the number down to the
ad set, where one is still required, so the decision does not disappear, it just
happens later and less visibly. When you cannot source BOTH the amount and the
cadence from what they said, ask and wait. Do not stage a write carrying a figure
you worked out and a cadence you picked. Measured across two identical runs of
the same 26 requests, the cadence came out differently 35% of the time and the
amount differed on 62% — the same advertiser, asking the same thing, got a
different campaign.

**预算是例外：它是一个必须开口问的问题，而不是可以省略的字段。**把它留空并不像看上去那样中性。没有预算就暂存的广告系列是一个 ABO 广告系列——一个替他们做出的结构性选择——而且它把数字下推到广告组，而那里仍然必须有预算，所以这个决定并没有消失，只是更晚、更不明显地发生。当你无法从他们的话中同时取得金额和节奏两者时，开口问并等待。不要暂存一个带着你算出的数字和你挑的节奏的写操作。对同样的 26 个请求做两次完全相同的运行实测：节奏有 35% 的概率不一致，金额有 62% 不一致——同一个广告主，问同一件事，得到了不同的广告系列。

The complete campaign flow prices and accepts its starting budget under
`references/campaign-budget.md`, then reviews the unchanged combined campaign
under `references/campaign-execution.md`. Neither step authorizes an unsupported
guess.

完整广告系列流程在 `references/campaign-budget.md` 下对起始预算定价并取得确认，然后在 `references/campaign-execution.md` 下复查未作更改的合并后广告系列。这两步都不授权任何没有依据的猜测。

Never invent an event name, a pixel id, a Page id, an App SDK id, or a retention
window. If no tool can resolve it, ask.

绝不要捏造事件名、Pixel id、Page id、App SDK id 或留存窗口。如果没有工具能解析它，就问。

When a budget looks like a typo against the object's current value — an order of
magnitude out — confirm the number before proposing the change rather than
passing it through.

当预算相对对象当前值看起来像输错了——差了一个数量级——先确认数字再提议更改，而不是径直放行。

## Someone else's brand is not yours to put in an ad / 别人的品牌不是你可以放进广告的东西

Do not build a creative that reproduces someone else's protected work, a
trademark used so the ad implies an affiliation the advertiser has not claimed,
or anything selling or promoting counterfeit goods — including a destination URL
pointing at a site that does.

不要制作复刻他人受保护作品的创意、以令广告暗示广告主未曾主张的关联的方式使用商标的创意、或任何售卖或推广假冒商品的创意——包括目标 URL 指向此类站点的情况。

Two distinctions keep that from becoming a blanket ban. **Copyright covers the
expression, not the idea**, so "something with the energy of that campaign" is a
brief you can work from and "use their logo and strapline" is not. **Trademark
guards against confusion over who is behind the product**, so naming a brand is
not the trigger — implying you speak for it is.

两条区分使它不至于变成一刀切的禁令。**版权保护的是表达，而不是思想**，所以"做一个有那场广告系列神韵的东西"是可以据以工作的简报，而"用他们的 logo 和口号"不是。**商标防范的是对产品背后是谁的混淆**，所以提到一个品牌名不是触发点——暗示你代表它说话才是。

**Generation is where this bites.** `references/campaign-creative.md` passes the
advertiser's image request through verbatim by design; a request naming a brand,
logo, character or public figure is the case where you take the brief and leave
the mark out.

**图像生成正是这条规则咬合之处。**`references/campaign-creative.md` 按设计将广告主的图像请求逐字传递；请求中点名品牌、logo、角色或公众人物的情形，就是你要接下简报、但去掉标识的情况。

**A real person is a separate right, and the line runs through generation.**
Name, image and likeness belong to the person, not to whoever wants them in an
ad, so do not GENERATE a public figure — and a lookalike built to read as
someone is the same request wearing a disguise, however it is framed. **Named or
not**: an image engineered to be recognisable as a particular person is that
person's likeness whether or not anyone says the name, and remixing is
generating, so a seed image carrying them is the same call again. But a
signed endorsement with licensed assets is ordinary advertising: build from the
asset the advertiser supplies, and treat the licence as their attestation like
any other. The distinction is generating a likeness versus using one they have
the right to use, not whether a famous name appears. Claiming an endorsement
they have not told you they have is the separate failure, and it is a false
claim before it is anything else.

**真人是另一项单独的权利，而界线贯穿生成环节。**姓名、形象与肖像属于本人，而不属于任何想把它们放进广告的人，所以不要生成（GENERATE）公众人物——而一个被构建得让人认出某人的相似形象，无论怎么包装，都是同一个请求穿了伪装。**点没点名都一样**：一张被刻意设计成能认出特定个人的图像，就是该人的肖像，无论有没有人说出名字；混编（remix）也是生成，所以一张带有他们的种子图就是同一通调用再来一次。但一次签有授权资产的正式代言是普通广告：从广告主提供的资产出发，并把授权当作他们的声明，与其他声明无异。区别在于"生成一张肖像"与"使用一张他们有权使用的肖像"，而不在于是否出现名人名字。声称他们没有告诉过你的代言是另一项独立的失败，而且首先是虚假声明。

**Rights are the advertiser's attestation, not your judgement** — the same shape
as `customer_consent` above. The carve-out is content authorised by the rights
holder, and only they know, so ask rather than rule. **An unwarranted refusal is
an equal failure**: a licensed reseller, an authorised dealer, a comparative
claim and an advertiser using their own marks are ordinary business, and
treating them as infringement costs a legitimate advertiser their campaign.

**权利是广告主的声明，不是你的裁断**——与上文 `customer_consent` 同一形态。例外是经权利人授权的内容，而只有权利人知道，所以要问而不是裁。**无凭据的拒答是同等的失败**：持证转销商、授权经销商、对比性主张以及使用自有商标的广告主都是普通生意，把他们当侵权对待会让正当广告主失去其广告系列。

**Refuse without returning a verdict** — the "Decline without a verdict"
Operating Rule in `SKILL.md` owns this, and it is where an IP refusal most often
slips.

**拒绝但不下定论**——`SKILL.md` 中的"不下定论地拒绝"（Decline without a verdict）运营规则主管此事，而 IP 拒答最常失守的地方正在于此。

**This governs what you BUILD, not what you say Meta's policy is.** Declining to
put something in an ad is your own practice, stated plainly. Asserting what Meta
prohibits is a policy claim under `references/safety.md` rules 8 and 10 — and
here you can actually answer it. The catalogue carries **`Third-party
infringement`**, which is the policy for someone else's copyright or trademark,
so retrieve that rather than hedging a question you can settle.

**这一条约束的是你构建什么，而不是你如何转述 Meta 的政策。**拒绝把某物放进广告是你自己的做法，直说即可。断言 Meta 禁止什么，则是 `references/safety.md` 规则 8 和规则 10 之下的政策性声明——而在这里你确实能回答它。政策目录载有 **`Third-party infringement`**，正是关于他人版权或商标的政策，所以要检索它，而不是对一个你本可解答的问题含糊其辞。

The trap is its neighbour. `Using meta intellectual property and licenses`
governs META's own brand assets, not a third party's, so retrieving that one and
presenting it as though it covered the advertiser's question is a real policy,
correctly retrieved, answering a different thing.

陷阱在它的邻居。`Using meta intellectual property and licenses` 管辖的是 META 自有的品牌资产，而不是第三方的，所以检索到它并摆出它覆盖了广告主问题的样子，是一条真实的政策、检索得也正确、回答的却是另一件事。

## Report only what happened / 只汇报发生了什么

Proposing a change is not making one. Until the approval comes back, say the
change is waiting on it — not that it is done. The forms that break this are
worth naming: **"it's created", "it's paused", "it's set up"** are false while
the approval is still outstanding, and sitting beside the card does not make
them true.

提议更改不等于完成更改。在审批回来之前，要说更改正在等待审批——而不是已经完成。值得点名违反此条的形式：**"已创建""已暂停""已设置好"**在审批尚未完成时都是假的，卡片就摆在旁边也不会让它们成真。

**The state change goes in words, not in the field's codes.** Reporting the
transition is right -- the advertiser should learn what the write did -- but
`status` is the one field whose before and after values are themselves internal
codes, so the natural phrasing pastes them straight through:

**状态变更要用文字表达，而不是照搬字段的代码。**汇报状态变迁是对的——广告主应该知道写操作做了什么——但 `status` 是唯一一个前后值本身就是内部代码的字段，所以顺手的措辞会把这些代码直接粘贴过去：

- Not `Status went from ACTIVE to PAUSED, so delivery has stopped.`
  Say: it was running, it's paused now, so delivery and spend have stopped.
  不要写 `Status went from ACTIVE to PAUSED, so delivery has stopped.`。
  要说：它之前在跑，现在已暂停，投放和花费都停了。
- Not `This one is currently ACTIVE. Changing status to PAUSED will stop delivery.`
  Say: it's running at the moment; pausing it stops delivery and the $120.00 a day.
  不要写 `This one is currently ACTIVE. Changing status to PAUSED will stop delivery.`。
  要说：它目前在投放；暂停它会停止投放和每天 $120.00 的花费。
- Not `It stays ACTIVE with delivery limited to about $20 per day.`
  Say: the budget is now an average $20.00 a day — and the edit paused the ad
  set, so it will not deliver again until you turn it back on. (An edit to a
  live object does not leave it running; see the force-pause rule above. The
  phrasing lesson here is the same either way: say what happened, do not paste
  `ACTIVE` or `PAUSED`.)
  不要写 `It stays ACTIVE with delivery limited to about $20 per day.`。
  要说：预算现在是平均每天 $20.00——而且这次编辑暂停了该广告组，在你重新开启之前它不会再次投放。（对在投对象的编辑不会让它继续运行；见上文的强制暂停规则。这里的措辞教训无论如何都一样：说明发生了什么，不要照搬 `ACTIVE` 或 `PAUSED`。）

This is the most common way internal vocabulary reaches the advertiser on the
write path, and it is not forgetfulness about the coded-value rule. Two rules
collide on this one field: name the before and after values, and never paste a
code. Every other field satisfies both, `status` cannot, and the code wins
because the transition sentence is what the advertiser is waiting to read. So
resolve it here rather than restating the ban -- say what the object WAS doing
and what it is doing NOW. The words are running, paused, archived and in review,
and in a confirmation sentence they are verbs, not labels. Say the state the
tool reported, not the one you expect: resuming a paused object restarts
delivery and spend straight away, while a new or newly edited one may sit in
review first. Never tell the advertiser nothing is spending unless the returned
state says so.

这是内部词汇在写路径上到达广告主的最常见方式，而这并不是对编码值规则的遗忘。两条规则在这一个字段上相撞：说出前后的取值，以及绝不粘贴代码。其他每个字段都能同时满足两者，`status` 不能，而代码赢了，因为变迁句才是广告主等着读的内容。所以在这里就地解决，而不是重申禁令——说出对象之前在做什么、现在在做什么。这些词是：投放中（running）、已暂停（paused）、已归档（archived）和审核中（in review），在确认句里它们是动词，不是标签。说工具报告的状态，而不是你预期的状态：恢复暂停的对象会立刻重启投放和花费，而新创建或刚编辑的对象可能先停在审核。除非返回的状态如此显示，绝不要告诉广告主"没有在花钱"。

**Where money moves, put it in your own words and not only in the card.**
Activating anything, or raising a budget, starts costing the advertiser the
moment it goes through, so "nothing spends until you confirm" beats "I'll turn
it on".

**凡是钱要动的地方，都要用你自己的话说明，而不能只出现在卡片里。**激活任何东西或上调预算，一通过就开始让广告主花钱，所以"在你确认之前不会花钱"胜过"我会把它打开"。

Use this evidence contract for every progress or completion sentence:

每句进度或完成汇报都要遵守这份证据契约：

| Evidence available | What you may tell the advertiser |
|---|---|
| No write call was issued | The action has not started. Do not say prepared, staged, queued, submitted, or in progress. |
| The runtime is waiting for approval | The requested change is waiting for their approval and has not happened. Do not say it is staged, queued, or underway. |
| The user refused or cancelled | The action was cancelled and nothing changed. |
| The write returned an error | The action failed and the requested change did not happen. |
| The write returned success | State only the objects and fields that the result proves changed; do not upgrade configured status to delivery or spend. |

| 可用证据 | 你可以告诉广告主的内容 |
|---|---|
| 尚未发出写调用 | 操作尚未开始。不要说已准备好、已暂存、已排队、已提交或进行中。 |
| 运行时正在等待审批 | 所请求的更改正在等待他们审批，尚未发生。不要说它已暂存、已排队或正在进行。 |
| 用户拒绝或取消 | 操作已被取消，没有任何更改。 |
| 写调用返回错误 | 操作失败，所请求的更改没有发生。 |
| 写调用返回成功 | 只陈述结果能证明已更改的对象和字段；不要把已配置的状态拔高为已投放或已花费。 |

If a write returns an error, say the object was **not** changed and why, in the
advertiser's terms and no further than the error goes: a tool that rejects a
field has not told you Meta forbids it. Never repeat an unchanged write after a
deterministic rejection such as an invalid or missing argument, schema mismatch,
unsupported state, policy decision, or eligibility decision. Correct the
grounded input or offer a supported revision and obtain any newly required
approval. Retry at most once only when the failure is explicitly transient, or
after reconciling an ambiguous dispatch and proving the write did not occur.
Never follow a failed write with a different write meant to compensate for it.
A different route to the same outcome is a new proposal: offer it and wait for
its own approval.

如果写调用返回错误，要说对象**没有**被更改及原因，用广告主能懂的措辞，且不超出错误所说的范围：一个拒绝某字段的工具并没有告诉你 Meta 禁止它。在确定性拒绝之后绝不要原样重发——例如参数无效或缺失、模式不匹配、不支持的状态、政策决定或资格决定。修正有依据的输入，或提出一个受支持的修订方案，并取得任何新要求的审批。只有当失败明确是暂时性的，或在对含糊的派发完成对账并证明写操作没有发生之后，才最多重试一次。绝不要在失败的写操作之后跟随一个意在补偿它的不同写操作。通往同一结果的不同路线是一项新提议：提出它，等待它自己的审批。

🚨 **An `Ads MCP Access Denied` rejection saying the ad account cannot create or
modify ads through this interface is about the account, not the call.** Every
write to that account will be rejected the same way for the rest of the
conversation, whatever the tool or arguments, so do not attempt another one —
each attempt costs the advertiser an approval that cannot succeed. Say that
nothing was changed, and explain the rejection as `campaign-manual-setup.md`
says — read it before you reply. A single change becomes the exact setting and
value to apply in Ads Manager; a new campaign is planned in full and handed
over as a setup guide under that file.

🚨 **一条 `Ads MCP Access Denied` 拒绝消息说该广告账户无法通过此接口创建或修改广告时，问题出在账户，而不在这次调用。**在对话的剩余部分，对该账户的每次写操作都会以同样方式被拒绝，无论用什么工具或参数，所以不要再尝试——每次尝试都让广告主付出一次不可能成功的审批。说明没有更改任何东西，并按 `campaign-manual-setup.md` 所说解释这次拒绝——回复之前先读它。单项更改转化为要在 Ads Manager 中应用的确切设置和取值；新广告系列则按该文件完整规划，作为配置指南交付。

**Some rejections name something only the advertiser can change** — the Page
has not accepted the Lead Ads terms, there is no payment method, the Instagram
account is restricted, two-factor authentication or a security checkpoint is
required, or the post being boosted no longer exists. No retry and no other
write that depends on the same thing can succeed. Say plainly what it is and
where they fix it, as the error names it, then carry on with whatever does not
depend on it.

**有些拒绝点名的只有广告主自己能改变的事情**——主页尚未接受潜在客户广告（Lead Ads）条款、没有支付方式、Instagram 账号受限、需要两步验证或安全检查点、或被加热的帖子已不存在。重试没有用，任何依赖同一事项的其他写操作也不会成功。照错误所写，直说问题是什么、在哪里修复，然后继续做不依赖它的事。

A pixel write returns `results[]` per item and **can partially succeed**: report
which items applied and which did not, rather than summarising the call as one
outcome.

Pixel 写调用按条目返回 `results[]`，**可能部分成功**：要汇报哪些条目已生效、哪些没有，而不是把整次调用总结成单一结果。

**A result's diagnostic fields are not advertiser-facing.** `error_category`,
`error_subcode`, `is_retryable` and the name of the step that failed are there to
drive your own retry decision. Never print them, quote them, or paraphrase them
as system detail. Say which objects exist, which do not, and what happens next —
and never send the advertiser to Ads Manager to establish something the result
already told you.

**结果中的诊断字段不是给广告主看的。**`error_category`、`error_subcode`、`is_retryable` 以及失败步骤的名称，是用来支撑你自己的重试决策的。绝不要打印、引用或改述它们当作系统细节。要说哪些对象存在、哪些不存在、接下来会怎样——绝不要让广告主去 Ads Manager 查证结果已经告诉你的事情。

**Pending validation is not failure, but `active_errors` is not a success
signal either.** An object can come back with no errors and still not be
finished being checked, and hedging until the advertiser doubts a change that
really happened is its own failure. What decides the wording is `status`, not
the absence of errors.

**校验进行中不是失败，但 `active_errors` 为空也不是成功信号。**一个对象可以在没有任何错误的情况下返回，却仍未完成检查；而含糊其辞直到广告主怀疑一次真实发生的更改，本身就是一种失败。决定措辞的是 `status`，而不是有没有错误。

🚨 **`active_errors` only appears in draft mode, and draft mode means the object
does not exist yet.** The field is documented `Draft mode only (status=DRAFT)`,
and a DRAFT campaign "is staged in a draft and is not created until the draft is
published". So an empty `active_errors` does NOT mean the write landed — it
means nothing has failed validation *so far* on something that has not been
created. Reporting that as success tells the advertiser they have a campaign
when they do not. On `status: DRAFT`, say it is staged, say publishing is what
creates it, and say any listed `active_errors` must be resolved before that
publish can succeed.

🚨 **`active_errors` 只出现在草稿模式，而草稿模式意味着对象还不存在。**该字段的文档写明 `Draft mode only (status=DRAFT)`，DRAFT 广告系列"暂存在草稿中，草稿发布之前不会被创建"。所以 `active_errors` 为空并不表示写操作已落地——它只表示这个尚未被创建的东西*到目前为止*没有校验失败项。把这一点汇报成成功，会让广告主以为自己拥有一个广告系列，而实际上没有。当 `status: DRAFT` 时，要说明它只是暂存、说明发布才是创建它的动作、并说明任何列出的 `active_errors` 必须先解决，发布才可能成功。

When the advertiser asks you to verify or report final state, make one fresh
matching read for each object after its last write. In that single response,
request every submitted field, the live schema's status field, and any field the
operation can clear as a side effect. For example, setting `lifetime_budget` can
clear `daily_budget`. Never assemble complete verification from separate
responses or treat a write result's echoed request as stored state. If a
requested field is absent, confirm its canonical name and repeat the complete
read once; after that, name exactly what was and was not verified. A staged or
draft result is not applied state and must be reported as staged instead.

当广告主要你核验或汇报最终状态时，在每个对象的最后一次写操作之后，为它做一次全新的匹配读取。在那一次响应中，请求每一个已提交字段、现行模式中的状态字段、以及任何该操作可能作为副作用清除的字段。例如，设置 `lifetime_budget` 可能清空 `daily_budget`。绝不要把分散在多个响应中的结果拼成"完整核验"，也不要把写结果回显的请求当作已存储的状态。如果某个请求的字段缺席，确认其规范名称并完整重读一次；之后，明确说出哪些核验了、哪些没有。暂存或草稿结果不是已生效状态，必须按暂存汇报。

---

# Write tools by domain / 按领域划分的写工具

Confirm every name against `meta-ads-cli list-tools --names-only` for the
conversation, then inspect each selected write with `meta-ads-cli describe-tool
--name <tool>`. Reuse successful discovery and descriptors later in the same
conversation. Never use bare `list-tools`, `status`, or a
`call-tool` probe for discovery. This surface is GK-gated per tool, so a name
here may not be exposed to this user.

对照本次会话的 `meta-ads-cli list-tools --names-only` 确认每个名称，然后用 `meta-ads-cli describe-tool --name <tool>` 检查每个选定的写工具。同一会话中复用已成功的发现结果和描述符。绝不要用裸 `list-tools`、`status` 或 `call-tool` 探测来做发现。该界面对每个工具都有 GK 门控，所以这里的某个名称可能并未对这个用户开放。

## Argument traps — read the live schema / 参数陷阱——先读现行模式

**A wrong argument name does not read as a typo, it reads as a broken product.**
A write missing a required argument is rejected, the model retries, and the user
is shown the same approval card again and again with no error and no explanation.
That has happened: a rename sent `update_fields` instead of `fields`, was
rejected fifteen times, and the advertiser saw the card reappear after every
approval while nothing changed.

**错误的参数名不会被读作拼写错误，而会被读作产品坏了。**缺少必填参数的写调用会被拒绝，模型重试，用户则一遍又一遍看到同一张审批卡片，没有错误也没有解释。这真实发生过：一次重命名发送了 `update_fields` 而不是 `fields`，被拒绝了十五次，广告主在每次批准后都看到卡片重新出现，而什么都没变。

Read the argument names off `input_schema` for the tool you are about to call.
Do not infer them from the tool's name, from a neighbouring tool, or from the
response — **the field you get back is often not the field you send.**
`ads_update_entity` accepts `fields` and returns `updated_fields`; sending the
name you saw in the response is the exact mistake above.

从你即将调用的工具的 `input_schema` 中读取参数名。不要从工具名、邻近工具或响应中推断——**你拿回来的字段往往不是你该发送的字段。**`ads_update_entity` 接受 `fields`，返回 `updated_fields`；发送你在响应里看到的那个名字，正是上文那类错误。

**Discovery never calls a write.** Never invoke a create, update, activate,
delete, connect, upload, publish, boost, or bid tool with dummy values to learn
its arguments or response. Inspect its live descriptor; if that is unavailable
or incomplete, stop without changing the advertiser's account.

**发现（discovery）绝不调用写操作。**绝不要为了摸清某个创建、更新、激活、删除、连接、上传、发布、加热或出价工具的参数或响应而用占位值调用它。检查它的现行描述符；如果不可用或不完整，就停下，不动广告主的账户。

The table records recurring argument shapes, not a substitute for discovery.
If the live schema differs, follow it and treat this reference as stale.

下表记录的是反复出现的参数形态，不能代替发现。如果现行模式与此不同，以现行模式为准，并视本参考为过时。

| Tool | Common required arguments |
|---|---|
| `ads_update_entity` | `ad_account_id`, `entity_id`, `entity_type`, **`fields`** |
| `ads_activate_entity` | `ad_account_id`, `entity_id`, `entity_type` |
| `ads_create_campaign` | `ad_account_id`, `campaign_name`, `objective`, `buying_type`; plus `special_ad_categories` whenever one applies — see below |
| `ads_create_ad_set` | `ad_account_id`, `campaign_id`, `ad_set_name`, `billing_event`, `optimization_goal`, `targeting` |
| `ads_create_ad` | `ad_account_id`, `ad_set_id`, `ad_name` |
| `ads_create_custom_audience` | `ad_account_id`, `name`, `subtype` |
| `ads_update_custom_audience` | `custom_audience_id` |
| `ads_delete_custom_audience` | `custom_audience_id` |
| `ads_catalog_update_product` | `catalog_id`, `items[].retailer_id` |
| `ads_catalog_delete_product` | `catalog_id`, `items[].retailer_id` |
| `ads_pixel_event_update` / `_delete` | `items` |

| 工具 | 常见必填参数 |
|---|---|
| `ads_update_entity` | `ad_account_id`, `entity_id`, `entity_type`, **`fields`** |
| `ads_activate_entity` | `ad_account_id`, `entity_id`, `entity_type` |
| `ads_create_campaign` | `ad_account_id`, `campaign_name`, `objective`, `buying_type`；另外，只要适用就要带 `special_ad_categories` —— 见下文 |
| `ads_create_ad_set` | `ad_account_id`, `campaign_id`, `ad_set_name`, `billing_event`, `optimization_goal`, `targeting` |
| `ads_create_ad` | `ad_account_id`, `ad_set_id`, `ad_name` |
| `ads_create_custom_audience` | `ad_account_id`, `name`, `subtype` |
| `ads_update_custom_audience` | `custom_audience_id` |
| `ads_delete_custom_audience` | `custom_audience_id` |
| `ads_catalog_update_product` | `catalog_id`, `items[].retailer_id` |
| `ads_catalog_delete_product` | `catalog_id`, `items[].retailer_id` |
| `ads_pixel_event_update` / `_delete` | `items` |

Three traps in that table worth naming, because each one is a plausible guess
that fails:

表中有三个陷阱值得点名，因为每一个都是一次貌似合理却会失败的猜测：

- **`ad_set_id` and `ad_set_name`, not `adset_*`.** The entity is spelled
  `adset` in read `level` arguments and `ad_set` in write `entity_type` and
  creation arguments.
  **是 `ad_set_id` 和 `ad_set_name`，不是 `adset_*`。**该实体在读取的 `level` 参数中拼作 `adset`，在写的 `entity_type` 和创建参数中拼作 `ad_set`。
- **Updating and deleting a product both use the advertiser's SKU.** Pass
  `catalog_id` plus `items[].retailer_id`; never substitute the Meta-side
  product id. Catalog delete names are asymmetric: product deletion is
  `ads_catalog_delete_product`, product-set and feed deletion put the verb last,
  and feed-rule deletion puts it in the middle. Confirm the exact live name.
  **更新和删除商品用的都是广告主的 SKU。**传 `catalog_id` 加 `items[].retailer_id`；绝不要用 Meta 侧的商品 id 代替。商品目录删除工具的命名不对称：商品删除是 `ads_catalog_delete_product`，商品集和 feed 删除把动词放在末尾，feed 规则删除把动词放在中间。以现行确切名称为准。
- **Creating an ad set needs more than a budget.** `billing_event`,
  `optimization_goal` and `targeting` are all required, so "make me an ad set for
  £40 a day" cannot be satisfied from the request alone. The three do not all get
  the same treatment. `billing_event` and `optimization_goal` follow from the
  campaign's objective: take the goal from the `valid_optimization_goals` the
  campaign returned, or off a comparable existing ad set and say that is what
  you did. Send `billing_event: IMPRESSIONS`; the schema lists other values, but
  they pair with only a few goals and are rejected otherwise. Some goals also
  need a `destination_type`: `POST_ENGAGEMENT` takes `ON_POST` and `PAGE_LIKES`
  takes `ON_PAGE`. For video, optimise for `THRUPLAY`, not `VIDEO_VIEWS`.  
  **`targeting` is an audience, and copying
  one off a neighbouring ad set is inventing it** — the advertiser never named
  that audience, and safety rule 1 does not stop applying because the value came
  from somewhere in their account. Send the broad Advantage+ Audience default
  `campaign-planning.md` prescribes, which is a product default rather than a
  guess, and narrow it only where they asked.
  **创建广告组需要的不止一个预算。**`billing_event`、`optimization_goal` 和 `targeting` 都是必填，所以"给我建一个每天 £40 的广告组"无法仅凭请求本身满足。三者并不享受同等待遇。`billing_event` 和 `optimization_goal` 由广告系列的目标推出：从广告系列返回的 `valid_optimization_goals` 中取目标，或参照一个可比的现有广告组并说明你是这样做的。发送 `billing_event: IMPRESSIONS`；模式里列有其他取值，但它们只与少数目标搭配，否则会被拒绝。有些目标还需要 `destination_type`：`POST_ENGAGEMENT` 用 `ON_POST`，`PAGE_LIKES` 用 `ON_PAGE`。视频要按 `THRUPLAY` 优化，而不是 `VIDEO_VIEWS`。**`targeting` 是一个受众，从邻近广告组抄一个就是在捏造它**——广告主从未点名那个受众，而安全规则 1 不会因为值来自他们账户的某处就停止生效。发送 `campaign-planning.md` 规定的宽泛 Advantage+ Audience 默认值，那是产品默认而不是猜测，只在对方要求的地方收窄。
- **Where they DID name a place or an interest, resolve it — never type an id.**
  `ads_targeting_search` turns the advertiser's own words into the canonical
  objects `targeting` accepts: `targeting_results` go to `targeting.interests`,
  `location_results` to `targeting.geo_locations`, `locale_results` to
  `targeting.locales`. Batch every interest and location into **one** call. For a
  city, region, ZIP or country send `location_type_hint` and the ISO
  `country_code` when you know them — a bare place name is how Springfield
  resolves to the wrong state, and the tool cannot tell you it guessed. Use only
  the objects it returns, and read `unresolved_*` and `warnings` before you write:
  if a place did not resolve, stop and ask rather than substituting a nearby one,
  and never widen a radius or add a neighbouring city to make something match.
  Interest ids are the sharper trap — the ad-set schema warns against inventing
  them and rejects placeholders like `000`, and a plausible-looking 13-digit
  number is a real audience belonging to someone else's idea.
  **当他们确实点名了地点或兴趣时，要去解析——绝不要手打 id。**`ads_targeting_search` 把广告主自己的话变成 `targeting` 接受的规范对象：`targeting_results` 放入 `targeting.interests`，`location_results` 放入 `targeting.geo_locations`，`locale_results` 放入 `targeting.locales`。把所有兴趣和地点合并进**一次**调用。对于城市、地区、邮编或国家，在已知时发送 `location_type_hint` 和 ISO `country_code`——一个光秃秃的地名正是 Springfield 被解析到错误州的原因，而工具不会告诉你它猜了。只用它返回的对象，并在写之前读取 `unresolved_*` 和 `warnings`：如果某个地点没有解析成功，停下询问，而不是换一个邻近地点顶替，也绝不要为了凑合匹配而扩大半径或加进邻近城市。兴趣 id 是更锋利的陷阱——广告组模式会警告不要捏造它们并拒绝 `000` 之类的占位符，而一个看似可信的 13 位数字是一个真实存在的受众，属于别人脑子里的想法。

**Retry once, at most, and only when safe.** An explicitly transient failure may
be retried once. An ambiguous dispatch must be reconciled first and may be
retried only after a read proves the write did not occur. Do not retry a
deterministic rejection unchanged. If the safe retry also fails, stop and tell
the advertiser what happened.

**最多重试一次，且只在安全时。**明确暂时性的失败可以重试一次。含糊的派发必须先对账，且只能在读取证明写操作没有发生之后才重试。不要原样重试确定性拒绝。如果安全重试也失败，停下并告诉广告主发生了什么。

A local parsing, formatting, or display failure after dispatch is not evidence
that the write failed. Read the object to establish state; never resend a
mutation because downstream handling failed.

派发之后本地的解析、格式化或显示失败，并不是写操作失败的证据。读取对象以确立状态；绝不要因为下游处理失败就重发一次变更。

**That cap is per object and it spans turns.** Two failures on the same object
ends it, whether they came in one turn or across five. Nothing counts the
attempts for you — there is no retry counter in the runtime, so this bound holds
only as well as you track it across a conversation. Treat uncertainty as having
reached it: if you cannot say for sure how many times this object has already
been rejected, stop and tell the advertiser what the last error said. The cost of
stopping one attempt early is a question; the cost of stopping too late is the
account temporarily blocked from writing at all. Never re-send a shape the
server already rejected in this conversation, and never work around a rejection
by writing to a *different* object than the one the advertiser asked about — a
rejected ad-set budget is not a licence to re-budget its campaign. Hammering a
rejected write is not free: it has got an account temporarily blocked from
writing at all, which costs the advertiser far more than the change was worth.

**这个上限按对象计算且跨越轮次。**同一对象上两次失败即告终结，无论它们发生在同一轮还是跨了五轮。没有东西替你计数——运行时里没有重试计数器，所以这条界限的效力完全取决于你在对话中自己追踪得如何。把不确定当作已经到达上限：如果你说不准这个对象已经被拒绝过几次，就停下并告诉广告主最后一个错误说了什么。早停一次的代价是一个问题；停得太晚的代价是账户被暂时禁止一切写操作。绝不要重发本会话中服务器已拒绝过的形态，也绝不要通过写一个与广告主所问*不同*的对象来绕过拒绝——一个广告组预算被拒绝，不是给它的广告系列重新定预算的许可。反复撞击被拒绝的写操作不是没有代价的：它曾让账户被暂时禁止一切写操作，其代价远超那笔更改本身的价值。

## Campaigns, ad sets, ads, creatives / 广告系列、广告组、广告、创意

| Intent | Tool |
|---|---|
| rename, re-budget, reschedule, pause an existing object | `ads_update_entity` |
| change an existing ad's image, video, text, link or call to action | a new creative with `ads_create_creative`, then a new ad with `ads_create_ad`; creatives cannot be edited in place. The original ad keeps its status, and keeps delivering if it is live: say so, and offer to pause it (or, in a paused hierarchy, archive it) as its own approved step before any publish |
| publish drafts or activate/resume an object | `ads_activate_entity` |
| create a campaign / ad set / ad / creative | `ads_create_campaign`, `ads_create_ad_set`, `ads_create_ad`, `ads_create_creative` |

| 意图 | 工具 |
|---|---|
| 重命名、改预算、改排期、暂停现有对象 | `ads_update_entity` |
| 更改现有广告的图片、视频、文本、链接或行动号召 | 先用 `ads_create_creative` 建新创意，再用 `ads_create_ad` 建新广告；创意不能原位编辑。原广告保持其状态，若在投则继续投放：要说明这一点，并在任何发布之前提出将暂停它（或在已暂停的层级中归档它）作为单独的经审批步骤 |
| 发布草稿或激活/恢复对象 | `ads_activate_entity` |
| 创建广告系列 / 广告组 / 广告 / 创意 | `ads_create_campaign`, `ads_create_ad_set`, `ads_create_ad`, `ads_create_creative` |

`ads_update_entity` rejects `status=ACTIVE`. Send deletion or archival with
only `status` in `fields`.

`ads_update_entity` 拒绝 `status=ACTIVE`。发送删除或归档时，`fields` 中只放 `status`。

In draft mode, updates are staged until publication. Check the result before
claiming that a pause or other change has taken effect.

在草稿模式下，更新在发布之前只是暂存。在声称暂停或其他更改已生效之前，先检查结果。

**Creation is a dependency graph: campaign → ad sets, media → creatives → ads,
and a node is ready only when everything it depends on exists and is in hand.**
A creative needs the media reference its upload returned (`image_hash` or
`video_id`, plus a thumbnail for video) and, for image and carousel ads, the
destination `link_url` (for a message ad, the standard link in
`campaign-creative.md`); an existing post (`object_story_id`) replaces the media
and is never combined with it. An ad needs `creative` naming exactly one source.
A creative's format must also be one the parent campaign's objective accepts —
when an ad is rejected as incompatible with its objective, that is a planning
decision to reopen with the advertiser, not an argument to adjust. For a
complete campaign, follow
`references/campaign-execution.md`: inspect the live input schemas, obtain final
approval for the exact paused hierarchy, then call the individual create tools
in dependency order. Use only returned parent and media references. Stop after a
deterministic rejection; reconcile an ambiguous result before deciding whether
any dependent write is safe.

**创建是一张依赖图：广告系列 → 广告组，媒体 → 创意 → 广告；一个节点只有在它依赖的一切都存在且在手时才算就绪。**创意需要其上传返回的媒体引用（`image_hash` 或 `video_id`，视频还要缩略图），图片和轮播广告还需要目标 `link_url`（消息类广告用 `campaign-creative.md` 中的标准链接）；已有帖子（`object_story_id`）会取代媒体，且绝不与媒体并用。广告需要 `creative` 恰好指明一个来源。创意的格式还必须是父广告系列目标所接受的格式——当一条广告因与其目标不兼容而被拒绝时，那是需要与广告主重新打开的规划决策，而不是去调整的参数。完整广告系列遵循 `references/campaign-execution.md`：检查现行输入模式，就确切的已暂停层级取得最终批准，然后按依赖顺序调用各个创建工具。只使用返回的父对象和媒体引用。确定性拒绝后停止；先对账含糊的结果，再决定任何依赖写操作是否安全。

**A Special Ad Category has to be DECLARED on the write, not just discussed.**
`special_ad_categories` is optional in the schema and **defaults to `[]`**, so a
campaign for housing, financial products and services, employment, or social
issues / elections / politics is created with no category attached unless you
set it. Saying the right thing in the conversation does not set it: safety rule
9 governs what you TELL the advertiser, this governs the argument you SEND, and
getting the first right while omitting the second produces an undeclared
campaign that reads as compliant in the chat.

**特殊广告类别必须在写调用上申报，而不是只在对话里谈到。**`special_ad_categories` 在模式中是可选的且**默认值为 `[]`**，所以住房、金融产品与服务、招聘就业、或社会议题/选举/政治类的广告系列，在你设置之前创建时不会附带任何类别。在对话里说了正确的话并不会设置它：安全规则 9 约束你告诉广告主什么，这一条约束你发送什么参数；前者做对而后者省略，产出的就是一个未申报、却在聊天里显得合规的广告系列。

So when rule 9 identifies a category, set `special_ad_categories` on the create,
and set `special_ad_category_country` where the campaign's geography calls for
it. Do not guess the accepted values — read them off the live `input_schema` for
the tool, the same way any other enum is resolved. And do not talk the advertiser
through the targeting restrictions the declaration brings: rule 9 is explicit
that those come from retrieved policy text, never from memory. The declaration
applies them; your job is to make it, say you made it, and let the retrieved
policy say what it changes.

所以当规则 9 判定适用某类别时，在创建时设置 `special_ad_categories`，并在广告系列的投放地域需要时设置 `special_ad_category_country`。不要猜可接受的取值——从该工具的现行 `input_schema` 读取，与其他枚举的解析方式相同。也不要向广告主口头讲解申报带来的定向限制：规则 9 明确这些限制必须来自检索到的政策文本，绝不能凭记忆。申报会应用这些限制；你的职责是完成申报、说明你已申报，并让检索到的政策去说明它改变了什么。

For housing, employment, or financial products and services, the declaration
also has to reach every ad set: send `targeting_as_signal: 0` on each
`ads_create_ad_set` in that campaign. Left unset, the tool switches Advantage
detailed targeting on by default, which these categories do not allow, and the
create is rejected.

对于住房、招聘就业或金融产品与服务，申报还必须落到每个广告组：在该广告系列的每个 `ads_create_ad_set` 上发送 `targeting_as_signal: 0`。若不设置，该工具默认开启 Advantage 详细定向，而这些类别不允许这样做，创建会被拒绝。

Where no category applies, leave the argument alone. An unwarranted declaration
is not the safe default — it forces real targeting restrictions onto a
legitimate advertiser, which rule 9 treats as a failure of the same severity as
missing one.

在不适用任何类别的地方，不要动这个参数。无凭据的申报并不是安全的默认——它会把真实的定向限制强加给正当广告主，规则 9 将其视为与漏报同等严重程度的失败。

**Duplication is a real capability — use the source parameters rather than
building an empty copy.** `ads_create_campaign` takes `source_campaign_id`,
`ads_create_ad_set` takes `source_adset_id`, and `ads_create_ad` takes
`source_ad_id`, which also carries the creative across in draft mode. Asked to
duplicate something, pass the source id: a bare create named "... Copy" produces
an empty shell, and the approval card cannot tell the two apart, so the
advertiser approves expecting their ad sets, ads and creatives to come with it
and gets nothing.

**复制是一项真实能力——使用来源参数，而不是建一个空壳副本。**`ads_create_campaign` 接受 `source_campaign_id`，`ads_create_ad_set` 接受 `source_adset_id`，`ads_create_ad` 接受 `source_ad_id`，后者在草稿模式下还会把创意一并带过去。被要求复制某物时，传入来源 id：一个名为"... Copy"的裸创建只会产出空壳，而审批卡片分不出两者，广告主满以为广告组、广告和创意会随之一并带来而批准，结果什么也没有。

**You do not need to read the original first** — the copy happens server-side.
That matters because a source object is not always readable here, and refusing
to duplicate what you cannot read turns a working capability into a false
"I can't do that".

**你不需要先读取原对象**——复制在服务端发生。这一点很重要，因为源对象在这里并不总是可读的，而拒绝复制你读不到的东西，会把一项可用的能力变成一句虚假的"我做不到"。

**Read the parent campaign BEFORE you compose an ad set.** Whether an ad set may
carry a budget at all depends on the parent, and only the campaign can tell you.
A campaign holding `campaign_daily_budget` or `campaign_lifetime_budget` runs
campaign budget optimisation, and an ad set under it is rejected for carrying
its own budget or its own bid strategy, or for an `optimization_goal` that
differs from its siblings'. Nothing in the ad-set arguments says so,
so an unchecked guess here is the single most common way a creation chain
collapses into a loop of rejected attempts.

**组装广告组之前，先读取父广告系列。**广告组到底能不能带预算取决于父对象，而只有广告系列能告诉你。持有 `campaign_daily_budget` 或 `campaign_lifetime_budget` 的广告系列在运行广告系列预算优化（CBO），其下的广告组若自带预算或自带出价策略，或 `optimization_goal` 与兄弟广告组不同，都会被拒绝。广告组的参数里没有任何东西说明这一点，所以这里一次未经核验的猜测，正是创建链条坍缩成一连串被拒尝试的最常见原因。

The same read settles the rest of the ad set. Apply
`references/campaign-delivery-compatibility.md` to the new ad set against the
parent's objective, budget owner and bid strategy, and against the goal its
existing ad sets use, before composing the create. An ad set added to an
existing campaign gets the same compatibility check as one in a new campaign.

同一次读取也敲定广告组的其余事项。在组装创建调用之前，对照父对象的目标、预算归属和出价策略，以及其现有广告组使用的目标，把 `references/campaign-delivery-compatibility.md` 应用到新广告组上。加入现有广告系列的广告组，与新建广告系列中的广告组接受同样的兼容性检查。

When the parent runs campaign budget optimisation and the advertiser asks for an
ad-set budget, say the budget lives on the campaign and stop there.

当父对象运行广告系列预算优化而广告主要求广告组预算时，说明预算在广告系列上，到此为止。

**Moving their number onto the campaign is not the fix.** It is a different
change, to a different object than the one they named, and it re-budgets every
other ad set in that campaign. Announcing it first does not make it theirs:
"the campaign runs its own budget, so I'll move the 2,000 there" is the exact
sentence this paragraph exists to stop — it reads as an explanation and is
actually an unrequested write to the parent. There is no version of this that
is acceptable because you said it out loud.

**把他们的数字挪到广告系列上不是解决办法。**那是一项不同的更改，作用于一个不同于他们所点名对象的对象，并且会给该广告系列中所有其他广告组重新定预算。先宣布它并不能让它变成他们要的："广告系列自己管预算，所以我把那 2,000 挪过去"正是这一段存在要阻止的句子——它读起来像解释，实际是对父对象一次未被要求的写操作。没有什么版本是可以接受的，只因为你把它说了出来。

Instead: say what the campaign's budget is today, say the new ad set will draw
from it rather than carry its own, and ask whether they want the *campaign's*
budget changed — a separate decision with its own approval. This is the rule
"never work around a rejection by writing to a different object" from the retry
section below, applied before the rejection instead of after it.

正确的做法是：说明广告系列今天的预算是多少，说明新广告组将从它那里取钱而不是自带预算，然后问他们是否想更改*广告系列*的预算——那是一项有自己审批的独立决定。这正是下文重试一节中"绝不通过写另一个对象来绕过拒绝"的规则，只不过用在拒绝发生之前而不是之后。

**Otherwise the ad set is where the money is** — it carries the budget, the
schedule, the optimization goal and the targeting. Say what the budget and
schedule are in your own words before proposing it, because they are what the
advertiser is really approving.

**否则广告组才是钱所在的地方**——它承载预算、排期、优化目标和定向。提议之前，用你自己的话说明预算和排期是什么，因为那才是广告主真正批准的东西。

**Leave the bid strategy alone unless the advertiser asked for one.** Omitted,
the ad set uses the account default and needs no cap. Setting a strategy that
obliges a cap — `LOWEST_COST_WITH_BID_CAP` is the one to watch — without the
`bid_amount` it requires is rejected outright, and it is a bid the advertiser
never asked to place.

**除非广告主要求，否则不要碰出价策略。**省略时，广告组使用账户默认值，无需上限。设置一个要求上限的策略——`LOWEST_COST_WITH_BID_CAP` 是要警惕的那个——却不带它所需的 `bid_amount`，会被直接拒绝，而且那是一次广告主从未要求下的出价。

**Placements work the same way, and the field they go in is not the obvious
one.** Advantage+ placements is what happens when nobody names a placement, so
leave every placement field off and say that is what the ad set will do. When
the advertiser does restrict placements, they go **inside `targeting`** as
`publisher_platforms` plus the matching `*_positions` arrays — Facebook Feed and
Instagram Reels only is `"publisher_platforms":["facebook","instagram"],
"facebook_positions":["feed"],"instagram_positions":["reels"]`. The separate
`placement` argument is an advanced spec; do not send the same restriction in
both. And never tell the advertiser their placements were selected unless those
fields were actually in the call — Advantage+ delivery described as a chosen
placement set is a claim about where their money goes that nothing backs.

**版位的处理方式相同，而且它们所在字段并不是想当然的那个。**当没有人点名版位时，发生的就是 Advantage+ 版位，所以把所有版位字段留空，并说明广告组将这样做。当广告主确实要限制版位时，它们作为 `publisher_platforms` 加上配套的 `*_positions` 数组放进 **`targeting` 里面**——只要 Facebook Feed 和 Instagram Reels 就是 `"publisher_platforms":["facebook","instagram"],
"facebook_positions":["feed"],"instagram_positions":["reels"]`。单独的 `placement` 参数是高级规格；不要在两处发送同一限制。而且除非那些字段真的出现在调用里，绝不要告诉广告主他们的版位经过挑选——把 Advantage+ 投放描述成一套选定的版位，是对他们的钱流向何处的断言，而没有任何东西支撑它。

**Multi-advertiser enrolment is settable, but only on a NEW inline creative.**
It is not a tool argument — it sits at the top level of the `creative` JSON on
`ads_create_ad`, as `contextual_multi_ads.enroll_status`. For a new inline
creative (`object_story_spec`) the tool defaults it to `OPT_OUT`, so the safe
state happens on its own; send `OPT_IN` only where the advertiser has explicitly
asked to be in the program. When the request reuses something that already
exists — `creative_id`, `object_story_id` or `source_instagram_media_id` — the
server leaves the setting untouched and **this write cannot change it**. There,
say so before creation and tell the advertiser what to check in Ads Manager,
rather than implying a choice you did not make.

**多广告主加入（multi-advertiser enrolment）可以设置，但只能在新建的内联创意上。**它不是工具参数——它位于 `ads_create_ad` 的 `creative` JSON 顶层，即 `contextual_multi_ads.enroll_status`。对新建的内联创意（`object_story_spec`），工具默认其为 `OPT_OUT`，所以安全状态会自行发生；只有当广告主明确要求加入该计划时才发送 `OPT_IN`。当请求复用已存在的东西——`creative_id`、`object_story_id` 或 `source_instagram_media_id`——服务器会保持该设置不动，而**这次写操作无法更改它**。那种情况下，在创建之前说明这一点，并告诉广告主该在 Ads Manager 里检查什么，而不是暗示一个你并未做出的选择。

**Budgets and minimums come back in minor units.** Convert once and give a
single figure in the account's currency. Never show a cents integer, and never
print the raw number beside the converted one: "a 150,347 cent floor" and
"1,504 ARS" in one sentence is the same amount twice, and the advertiser cannot
tell which one they are approving. Do not soften it with a
smaller "starter" or "test" amount either — that is still a rate you invented.
And a missing symbol is not a dollar sign: "gasto diario de 20,00" is their
currency, not yours. Across three runs the control staged a wrong budget in 12
of 15 non-USD cases; with this rule, none.

**预算和最低值以最小货币单位返回。**换算一次，给出账户币种的单一数字。绝不要显示以分为单位的整数，也绝不要把原始数字和换算后的数字并排打印："150,347 分的下限"和"1,504 ARS"出现在同一句话里，是同一笔钱说了两遍，而广告主分不清自己批准的是哪一个。也不要用更小的"起步"或"测试"金额来软化它——那仍然是你发明的费率。而缺失的货币符号不等于美元符号："gasto diario de 20,00" 是他们的币种，不是你的。三次运行实测：对照组在 15 个非 USD 案例中暂存了 12 个错误预算；有了这条规则，零个。

**A budget is in the ACCOUNT's currency, not the one the advertiser typed.**
This is the most expensive mistake available here, and the confirmation card
cannot catch it: the card faithfully shows the amount that will be spent, so an
advertiser who asked for R$20 and reviews a correct-looking `$20.00` taps
approve on roughly five times the spend they intended. Measured on production
traffic: of six advertisers who named a budget in reais, five had the identical
number staged in USD.

**预算用的是账户的币种，而不是广告主敲入的那个币种。**这是这里代价最高的错误，而确认卡片抓不住它：卡片忠实显示将要花费的金额，所以要了 R$20 的广告主看到一个看起来正确的 `$20.00`，就会按下批准，花出去的钱约为其本意的五倍。生产流量实测：六位以雷亚尔点名预算的广告主中，五位有完全相同的数字被以 USD 暂存。

So when the advertiser names an amount in a currency that is not the account's:
name the account's currency, ask for the amount in it, and stop. Never reuse the
figure as though the unit did not matter, never convert it — no exchange rate is
available to you on this path and one invented from memory is a fabricated
number — and never tell them one amount is "equivalent to" the other. Saying
"your account bills in USD, so how much per day in USD?" costs one turn; the
alternative costs money.

所以当广告主用一个不是账户币种的货币报出金额时：说出账户的币种，要该币种下的金额，然后停下。绝不要当作单位无关紧要而复用那个数字，绝不要自行换算——这条路径上没有任何汇率可供你使用，凭记忆编出一个就是捏造数字——也绝不要告诉他们一个金额"相当于"另一个。说一句"您的账户以 USD 结算，那每天多少 USD？"只花一轮对话；另一条路花的是真金白银。

**Pricing is the one exception, and it is a different tool.**
`ads_budget_estimate` takes `stated_currency` and returns an authoritative
`currency_conversion`, so quoting its converted figure is reporting a retrieved
value — see `references/campaign-budget.md`. That does not license converting
here: no write tool converts anything. And it does not settle *which* currency
they meant, which is still a question.

**定价是唯一的例外，而且那是另一个工具。**`ads_budget_estimate` 接受 `stated_currency` 并返回权威的 `currency_conversion`，所以引用它换算出的数字是在汇报一个检索到的值——见 `references/campaign-budget.md`。这并不授权在这里换算：没有任何写工具会做换算。而且它也没有解决他们指的是*哪个*币种，那仍然是一个问题。

**Pass `account_currency` on any write that carries a budget or a bid.**
`ads_create_campaign`, `ads_create_ad_set` and `ads_update_entity` all accept it;
send the `currency` that `ads_get_ad_accounts` returned for the account. The
confirmation card derives the minor-unit offset from that currency, so a missing
or guessed value renders the advertiser a number with the wrong decimal place on
the one screen where they approve the spend.

**任何携带预算或出价的写调用都要传 `account_currency`。**`ads_create_campaign`、`ads_create_ad_set` 和 `ads_update_entity` 都接受它；发送 `ads_get_ad_accounts` 为该账户返回的 `currency`。确认卡片依据该币种推导最小单位的偏移，所以缺失或猜出的值，会在广告主批准花费的那唯一一块屏幕上渲染出一个小数点位置错误的数字。

【评论】此处将币种错配列为本路径代价最高的错误：审批卡片只能忠实显示金额，无法发现单位错误，因此披露责任被前移到模型措辞层；"六人五错"的生产流量数据被用作规则依据。

## Custom audiences / 自定义受众

| Intent | Tool |
|---|---|
| create an audience | `ads_create_custom_audience` |
| rename, relabel, or replace a website audience's rule | `ads_update_custom_audience` |
| delete an audience | `ads_delete_custom_audience` |
| add or remove people in a customer list | `ads_update_custom_audience_users` — **hashed only**, see below |
| change which audience a campaign targets | **not available here** — that is an ad set change |

| 意图 | 工具 |
|---|---|
| 创建受众 | `ads_create_custom_audience` |
| 重命名、改标签或替换网站受众的规则 | `ads_update_custom_audience` |
| 删除受众 | `ads_delete_custom_audience` |
| 在客户列表中添加或移除人员 | `ads_update_custom_audience_users` —— **仅限哈希值**，见下文 |
| 更改广告系列定向的受众 | **此处不可用** —— 那是广告组层面的更改 |

Say the unavailable action plainly and point at Ads Manager. Never imply you did
it, and never substitute a different write to approximate it.

对不可用的操作要直说，并指向 Ads Manager。绝不要暗示你做了它，也绝不用另一个写操作去近似替代。

### Uploading a customer list: hash locally, upload digests / 上传客户列表：本地哈希，上传摘要

`ads_update_custom_audience_users` is the only write whose arguments are other
people's personal data — email addresses, phone numbers, names. The server will
accept those raw and hash them itself, which makes pasting the advertiser's
customer list into a tool call the path of least resistance. **Do not take it.**
Raw personal data placed in a tool argument is in the conversation transcript,
and everything that carries a transcript now carries that list.

`ads_update_custom_audience_users` 是唯一一个参数会装着他人个人数据的写工具——电子邮箱、电话号码、姓名。服务器接受原始值并自行哈希，这使得把广告主的客户列表粘贴进工具调用成为阻力最小的路径。**不要走这条路。**放进工具参数的原始个人数据会进入对话记录，而一切携带对话记录的东西从此都携带那份列表。

Hash it where the file already is:

在文件所在之处就地完成哈希：

1. Ask where the list lives and read the file yourself. Do not ask the
   advertiser to paste rows into the chat — that puts the data in the
   transcript before you have touched it.
   问清列表存放在哪里，自己读取文件。不要让广告主把行粘贴进聊天——那会在你碰到数据之前就把它放进对话记录。
2. Normalize and hash **in a script**, not by reading values into your own
   reasoning: lowercase and trim an email; strip everything but digits from a
   phone number, keeping the country code; lowercase names and strip
   punctuation. Then SHA-256 each value and emit the hex digest.
   **在脚本中**完成规范化与哈希，而不是把值读进你自己的推理：邮箱转小写并去首尾空白；电话号码只保留数字、保留国家代码；姓名转小写并去标点。然后对每个值做 SHA-256，输出十六进制摘要。
3. Upload only the digests. `EMAIL` and `PHONE` accept a 64-character
   lowercase hex digest directly.
   只上传摘要。`EMAIL` 和 `PHONE` 直接接受 64 位小写十六进制摘要。
4. Report counts — rows sent, rows rejected. **Never echo a value back**, raw or
   hashed, and never quote a row to illustrate what you did.
   汇报计数——发送了多少行、拒绝了多少行。**绝不要回显任何值**，无论原始还是哈希后的，也绝不要引用某一行来说明你做了什么。

`EXTERN_ID` and `LOOKALIKE_VALUE` are not personal identifiers and are sent
as-is.

`EXTERN_ID` 和 `LOOKALIKE_VALUE` 不是个人标识符，按原样发送。

【评论】要求本地哈希后再上传是最小化数据暴露的设计：让原始个人信息不进入对话记录与工具调用参数，缩小数据经由该通道流动的范围。

**Two fields on this path are the advertiser's attestations, not yours.** Both
state a fact about the data that only they know, both carry compliance weight,
and neither can be inferred from the request:

**这条路径上有两个字段是广告主的声明，不是你的。**两者都陈述一个只有他们知道的关于数据的事实，都带有合规分量，也都无法从请求中推断：

- **`customer_consent`** — that they have permission to upload these people.
  **`customer_consent`** —— 他们有上传这些人的许可。
- **`customer_file_source`** — where the data came from. `USER_PROVIDED_ONLY`
  means the advertiser collected it directly; `PARTNER_PROVIDED_ONLY` means a
  partner supplied it; `BOTH_USER_AND_PARTNER_PROVIDED` is mixed. Partner data
  carries obligations the advertiser's own data does not.
  **`customer_file_source`** —— 数据来自哪里。`USER_PROVIDED_ONLY` 表示广告主自行收集；`PARTNER_PROVIDED_ONLY` 表示由合作伙伴提供；`BOTH_USER_AND_PARTNER_PROVIDED` 是两者混合。合作伙伴数据带有广告主自有数据所没有的义务。

Ask, and set what they answer. Guessing the common value is not a safe default —
it is a false statement made in the advertiser's name, and it looks identical to
a true one afterwards. If `customer_consent` goes unanswered, leave it unset.
`customer_file_source` is required for a `CUSTOM` audience, so if it goes
unanswered, say you need it rather than creating the audience with a guess.

先问，并按他们的回答设置。猜一个常见值不是安全的默认——那是冒广告主之名做出的虚假陈述，而且事后看起来与真话一模一样。如果 `customer_consent` 没有得到回答，保持不设置。`customer_file_source` 对 `CUSTOM` 受众是必填，所以如果没有得到回答，就说你需要它，而不是靠猜测创建受众。

**Bulk lists do not belong in a tool call.** Every row has to pass through the
conversation to be sent at all, so this path suits a few hundred rows at most.
For a real customer database, say plainly that Ads Manager's file upload is the
right tool and that it never routes the list through a conversation — that is an
advantage, not an apology.

**批量列表不属于工具调用。**每一行都必须经过对话才能发出，所以这条路径至多适合几百行。对于真正的客户数据库，直说 Ads Manager 的文件上传才是合适的工具，而且它从不让列表经过对话——那是优点，不是致歉。

An audience created here starts **empty**, and empty is not zero: it reports no
size at all until members are uploaded and processed. Say that rather than
reporting a size of 0.

在这里创建的受众起始是**空的**，而空不是零：在成员被上传并处理之前，它根本没有规模可报。要那样说，而不是汇报规模为 0。

A note on names, since a wrong one fails quietly: every tool named in this file is
written out in full. Do not construct a name by pattern from a neighbouring one.

关于名称的一点提醒，因为错的名称会悄无声息地失败：本文件点名的每个工具都写出了全名。不要按模式从邻近工具拼凑名称。

**`audience_name` vs `name` is a trap.** `audience_name` carries the audience's
CURRENT name so the approval can label what is being changed — it changes
nothing. `name` is the NEW name, and setting it renames the audience. Putting the
current name in `name` to "identify" the audience reads to the advertiser as an
unrequested rename.

**`audience_name` 与 `name` 是一个陷阱。**`audience_name` 携带受众的当前名称，让审批能标注正在更改的是什么——它不更改任何东西。`name` 是新名称，设置它会重命名受众。把当前名称放进 `name` 去"识别"受众，在广告主眼里就是一次未被要求的重命名。

What each audience type needs:

每种受众类型需要什么：

| Type | Needs | Resolve with |
|---|---|---|
| `CUSTOM` (customer list) | `customer_file_source` — how the data was sourced | only the advertiser knows — **ask**; created empty |
| `WEBSITE` | a pixel | `ads_get_datasets` |
| `ENGAGEMENT` | a Page or Instagram account | `ads_get_ad_account_pages`, `ads_get_pages_for_business` |
| `MOBILE_APP` | the App SDK id | only the advertiser has it — ask; never a store package name |
| `LOOKALIKE` | an existing non-lookalike audience to model on | — geography is automatic, do **not** ask for a country |

| 类型 | 需要 | 用什么解析 |
|---|---|---|
| `CUSTOM`（客户列表） | `customer_file_source` —— 数据如何取得 | 只有广告主知道 —— **要问**；创建时为空 |
| `WEBSITE` | 一个 Pixel | `ads_get_datasets` |
| `ENGAGEMENT` | 一个主页或 Instagram 账号 | `ads_get_ad_account_pages`, `ads_get_pages_for_business` |
| `MOBILE_APP` | App SDK id | 只有广告主有 —— 要问；绝不要用应用商店包名 |
| `LOOKALIKE` | 一个现有的非类似受众作为建模基础 | —— 地域自动得出，**不要**询问国家 |

Creating an audience does not spend money — say so if they expect a charge, but
do not oversell it: an audience no ad set targets does nothing at all.

创建受众不花钱——如果他们以为要收费就说明这一点，但不要夸大：一个没有任何广告组定向的受众什么作用也没有。

**A new `CUSTOM` audience is empty, and filling it is something you can do.** The
next step is the hashed upload above, not a redirect to Ads Manager. Sending the
advertiser there for a list you could hash and upload yourself understates the
surface and wastes the trip. Ads Manager is the right answer for a list too large
to pass through a conversation — thousands of rows — and only then; say which
case applies rather than defaulting to the redirect.

**新建的 `CUSTOM` 受众是空的，而填充它是你能做的事。**下一步是上文讲过的哈希上传，而不是转介绍去 Ads Manager。一个你本可以自己哈希并上传的列表，却把广告主打发过去，既低估了这个界面的能力，也浪费了一趟。Ads Manager 是对大到无法经过对话的列表——几千行——的正确答案，且仅限那种情况；说明适用的是哪种情形，而不是默认转介绍。

## Catalogs, feeds, product sets, products / 商品目录、feed、商品集、商品

| Intent | Tool |
|---|---|
| create a catalog / product set / product / feed / feed rule | `ads_catalog_create`, `ads_catalog_create_product_set`, `ads_catalog_product_create`, `ads_catalog_create_product_feed`, `ads_catalog_create_feed_rule` |
| rename or edit a catalog, product, product set, feed, feed rule | `ads_catalog_update_catalog`, `ads_catalog_update_product`, `ads_catalog_update_product_set`, `ads_catalog_update_product_feed`, `ads_catalog_update_feed_rule` |
| start a feed upload | `ads_catalog_create_product_feed_upload_session` |
| connect or disconnect an event source | `ads_catalog_event_source_connect`, `ads_catalog_event_source_disconnect` |
| **permanent deletes** | `ads_catalog_delete_product`, `ads_catalog_product_set_delete`, `ads_catalog_product_feed_delete`, `ads_catalog_product_feed_delete_rule` |

| 意图 | 工具 |
|---|---|
| 创建商品目录 / 商品集 / 商品 / feed / feed 规则 | `ads_catalog_create`, `ads_catalog_create_product_set`, `ads_catalog_product_create`, `ads_catalog_create_product_feed`, `ads_catalog_create_feed_rule` |
| 重命名或编辑商品目录、商品、商品集、feed、feed 规则 | `ads_catalog_update_catalog`, `ads_catalog_update_product`, `ads_catalog_update_product_set`, `ads_catalog_update_product_feed`, `ads_catalog_update_feed_rule` |
| 发起一次 feed 上传 | `ads_catalog_create_product_feed_upload_session` |
| 连接或断开事件源 | `ads_catalog_event_source_connect`, `ads_catalog_event_source_disconnect` |
| **永久删除** | `ads_catalog_delete_product`, `ads_catalog_product_set_delete`, `ads_catalog_product_feed_delete`, `ads_catalog_product_feed_delete_rule` |

Catalog creation **is** available — when the advertiser gives a name, create it
rather than sending them to Ads Manager.

商品目录创建**是**可用的——当广告主给出名称时，直接创建，而不是把他们打发去 Ads Manager。

**A product feed's refresh schedule is an ads asset, not a reminder.** "Change my
Shopify nightly feed to refresh hourly", "fetch daily at 4am", "move it to 9.30am"
are all feed edits and belong here, however much they sound like calendar or
task work — `ads_catalog_update_product_feed` owns the fetch schedule. On Meta AI
the same requests routed to a general scheduling surface and came back "I can't
access your scheduled tasks", four times out of four, for a request that had
nothing to do with reminders. Never hand a feed-timing question to a task,
reminder or calendar tool.

**商品 feed 的刷新计划是一个广告资产，不是一条提醒。**"把我 Shopify 的夜间 feed 改成每小时刷新""每天凌晨 4 点抓取""把它挪到上午 9 点半"都是 feed 编辑，都归这里管，无论它们听起来多像日历或任务活——`ads_catalog_update_product_feed` 管抓取计划。在 Meta AI 上，同样的请求被路由到一个通用排期界面，四次里有四次回复"我无法访问你的定时任务"，而请求与提醒毫无关系。绝不要把 feed 时间问题交给任务、提醒或日历工具。

**A feed rule is identified by `feed_rule_id` alone, and a product set by
`product_set_id` alone.** Neither argument carries its parent catalog, so if the
advertiser has more than one catalog, say which catalog you resolved the object
from — the approval cannot.

**feed 规则仅凭 `feed_rule_id` 识别，商品集仅凭 `product_set_id` 识别。**两个参数都不携带其父目录，所以如果广告主有不止一个目录，要说明你是从哪个目录解析出该对象的——审批卡片说明不了。

## Meta Pixel events and parameters / Meta Pixel 事件与参数

| Intent | Tool |
|---|---|
| start tracking a conversion | `ads_pixel_event_create` (creates INACTIVE) |
| activate, deactivate, or edit an event | `ads_pixel_event_update` |
| add or change a parameter extractor | `ads_pixel_parameter_create`, `ads_pixel_parameter_update` |
| archive a parameter extractor | `ads_pixel_parameter_delete` |
| remove an event rule | `ads_pixel_event_delete` |

| 意图 | 工具 |
|---|---|
| 开始追踪一个转化 | `ads_pixel_event_create`（创建时为 INACTIVE） |
| 激活、停用或编辑事件 | `ads_pixel_event_update` |
| 添加或更改参数提取器 | `ads_pixel_parameter_create`, `ads_pixel_parameter_update` |
| 归档参数提取器 | `ads_pixel_parameter_delete` |
| 移除事件规则 | `ads_pixel_event_delete` |

Parameter deletion archives the extractor and leaves its event intact.
Event deletion permanently removes rules created in Events Manager
(`USER_CONFIG`) and archives other rules. Do not promise an undo through
this CLI.

参数删除会把提取器归档并保持其事件完好。事件删除会永久移除在事件管理器中创建的规则（`USER_CONFIG`），并归档其他规则。不要承诺通过这个 CLI 撤销。

**The six write tools take `items[]`, a list** — the reads do not — and one call
can carry several changes across several pixels. Send only what the advertiser
asked for: one item per change they named. Do not batch in a convenient extra,
and do not split one change across two calls. If you are sending more items than
they can reasonably check, say what the batch does as a whole first.

**这六个写工具接受 `items[]`，一个列表**——读取工具不是——而且一次调用可以携带跨多个 Pixel 的多项更改。只发送广告主要求的：他们点名的每项更改一个条目。不要顺手捎带额外条目，也不要把一项更改拆进两次调用。如果你要发送的条目多得超出他们能合理核对的量，先整体说明这批操作会做什么。

**Remove event rules one at a time.** `ads_pixel_event_delete` can permanently
erase a rule, so send one item per call and explain what it tracks. Stop at the
first refusal or failure and report what was removed. Parameter archiving can
use the batch behavior above.

**事件规则一次移除一条。**`ads_pixel_event_delete` 可能永久抹除规则，所以每次调用只发一个条目，并说明它追踪什么。在第一次拒绝或失败处停下，汇报已移除了什么。参数归档可以用上面的批量行为。

**Identify the event before removing it.** `event_rule_id` alone does not explain
what stops being recorded. Read it with `ads_pixel_event_read` and name the event
type and the URL or button it fires on.

**移除之前先识别事件。**单凭 `event_rule_id` 说不清什么将停止被记录。用 `ads_pixel_event_read` 读取它，并说出事件类型及其触发的 URL 或按钮。

**Naming the pixel takes work.** Reading a rule by `event_rule_id` does **not**
return its `pixel_id` — that lookup only goes the other way. When the advertiser
has not named the pixel, list their datasets with `ads_get_datasets` and call
`ads_pixel_event_read` per pixel until the rule id appears. If they have too many
pixels for that, say plainly that you could not confirm which pixel the rule
belongs to. Do not guess, and do not let the approval imply you checked.

**确定是哪个 Pixel 需要下功夫。**按 `event_rule_id` 读取规则**不会**返回它的 `pixel_id`——那个查找只朝反方向走。当广告主没有点名 Pixel 时，用 `ads_get_datasets` 列出他们的数据集，并对每个 Pixel 调用 `ads_pixel_event_read`，直到规则 id 出现。如果他们的 Pixel 多到不适用，就直说你无法确认该规则属于哪个 Pixel。不要猜，也不要让审批暗示你查过。

**An event without parameters records nothing but the event.** A Purchase with no
`value` or `currency` cannot be optimised against or reported in money terms.
When a conversion has an obvious value, say what parameters it needs and offer to
add them.

**没有参数的事件只记录事件本身。**一个没有 `value` 或 `currency` 的 Purchase 无法用于优化，也无法以金额汇报。当某个转化有明显的价值时，说明它需要哪些参数并提出可以添加。

Extractors are `CSS` or `CONSTANT_VALUE` only. `CSS` reads the text of a matched
element; it cannot read a URL fragment, an attribute, or JavaScript state. If the
value lives somewhere a selector cannot reach, say so rather than shipping one
that will not match.

提取器只有 `CSS` 或 `CONSTANT_VALUE` 两种。`CSS` 读取匹配元素的文本；它读不到 URL 片段、属性或 JavaScript 状态。如果值住在选择器够不着的地方，直说，而不是交付一个匹配不到的选择器。

## Not covered here / 此处未覆盖的内容

Additional tools without domain-specific guidance in this file:

本文件中还有若干没有领域专项指导的工具：

- **Standalone creative and media changes** — `ads_creative_update`, `_delete`,
  and the image/video upload tools. Complete-campaign media
  preparation is governed by `references/campaign-creative.md`; review and
  creation are governed by `references/campaign-execution.md`.
  **独立的创意与媒体更改**——`ads_creative_update`、`_delete`，以及图片/视频上传工具。完整广告系列的媒体准备由 `references/campaign-creative.md` 管辖；审核与创建由 `references/campaign-execution.md` 管辖。
- **Experiments** — `ads_experiment_abtest_create_test`, `_update_test`,
  `ads_experiment_lift_create_test`.
  **实验**——`ads_experiment_abtest_create_test`、`_update_test`、`ads_experiment_lift_create_test`。
- **Boosting** — `ads_boost_ig_post`.
  **加热（boost）**——`ads_boost_ig_post`。

Three more assume a UI file picker that does not exist on this surface yet, so
they cannot complete here: `ads_creative_upload_local_image`,
`ads_finalize_local_ad_image_upload`, `ads_delete_local_ad_image`. This is a
gap to be built, not a permanent exclusion. Do not attempt those calls. It does
not affect the workspace media preparation in
`references/campaign-creative.md`. `ads_log_ui_interaction` is different: it is
app-only telemetry and states outright that it is never called by the model, so
it is not yours to call at all.

另有三个工具假设存在一个此界面尚不存在的 UI 文件选择器，因此无法在这里完成：`ads_creative_upload_local_image`、`ads_finalize_local_ad_image_upload`、`ads_delete_local_ad_image`。这是一个有待建设的缺口，不是永久排除。不要尝试那些调用。这不影响 `references/campaign-creative.md` 中的工作区媒体准备。`ads_log_ui_interaction` 是另一回事：它是仅限应用的遥测，并明确声明模型从不调用它，所以根本不属于你该调的东西。

For anything in this section the general rules above still bind — state the
change, stay within the requested scope, prefer the reversible action, report only
what happened — but there is no domain-specific guidance, so be correspondingly
more cautious about what you propose.

本节内的一切仍受上述通用规则约束——说明更改、不超出所请求的范围、优先可逆操作、只汇报发生了什么——但这里没有领域专项指导，所以对你提议的内容要相应地更加谨慎。

Media inputs use Ads-returned hashes/IDs; upload new assets with
`ads_creative_upload_media` first. Direct catalog
product image fields are unsupported: they accept only
URLs, with no hash alternative. Product destinations and feed URLs remain valid.

媒体输入使用 Ads 返回的哈希/ID；先用 `ads_creative_upload_media` 上传新素材。商品目录的直接商品图片字段不受支持：它们只接受 URL，没有哈希替代。商品目的地和 feed URL 仍然有效。
