<!-- BILINGUAL-EN-ZH -->

# Browser booking / 浏览器预订

Use the browser when no connected native path can complete the booking, or
when the provider's authenticated site is needed for loyalty, credits, saved
profile data, or inventory unavailable elsewhere.

当没有已连接的原生路径能完成预订时，或当需要提供方经过身份验证的站点来使用会员权益、积分、已保存的资料数据或其他渠道没有的库存时，使用浏览器。

Before starting checkout on a website, ask whether the user has an account
there and wants to sign in; explain briefly that signing in can reuse profile
details and expose member pricing, loyalty benefits, credits, points, or booking
history. If they do, start with the site's supported sign-in flow. If they do
not, continue as a guest when the site permits it. Do not ask when an authorized
signed-in session is already available, and do not require account creation.

在网站上开始结账之前，询问用户是否在该网站有账户并想要登录；简要说明登录可以复用资料信息，并可用到会员价、会员权益、储值、积分或预订历史。如果用户愿意，从该站点支持的登录流程开始。如果不愿意，在站点允许时以访客身份继续。当已有经过授权的登录会话时不要询问，也不要强制要求创建账户。

## Keep one checkout alive / 保持唯一一个结账会话

Start one browser task for the selected option and keep its task identifier
through review, changes, submission, and verification. Do not create competing
checkouts on the same site or start over after the user approves. Carts,
holds, and authentication state belong to the original session.

为选定的选项启动一个浏览器任务，并在审查、修改、提交和验证的整个过程中保留其任务标识符。不要在同一站点上创建相互竞争的结账，也不要在用户批准后推倒重来。购物车、占位和身份验证状态都属于原始会话。

Give the browser task the exact provider URL and all known non-sensitive
choices. Tell it to use an existing authenticated session when available,
prepare the booking through final review, and stop before the action that
creates a reservation, charge, deposit, points transfer, or cancellation
penalty. Do not place passport, traveler-security, loyalty, or payment numbers
in the task text. Let the site, wallet, or user provide them at the appropriate
field.

向浏览器任务提供确切的提供方 URL 和所有已知的非敏感选择。告知它：在可用时使用已有的已验证会话，把预订准备到最终审查环节，然后在将产生预订、扣费、押金、积分转移或取消费用的动作之前停下。不要把护照号、旅客安保号码、会员号或支付号码放入任务文本。让站点、钱包或用户在相应字段中提供这些信息。

【评论】"准备到最终审查即停止"的硬边界把不可逆的支付/预订动作与自动化执行分离，是高风险操作人工确认的典型设计。

Do not ask the user to paste a card number, expiry, security code, passport
number, traveler-security number, loyalty number, or account password into
chat. Use an approved secure wallet, credential flow, provider form, or browser
handoff. If no secure input path is available, state that checkout is blocked
and leave the exact option prepared; do not downgrade to collecting the secret
in conversation.

不要要求用户把卡号、有效期、安全码、护照号、旅客安保号码、会员号或账户密码粘贴到聊天中。使用已批准的安全钱包、凭据流程、提供方表单或浏览器交接。如果没有可用的安全输入路径，说明结账已受阻，并把确切的选项保留在准备就绪状态；不要退而求其次在对话中收集机密信息。

This remains true when a connector card or browser handoff fails to render for
the user. Do not offer chat entry as a fallback. Do not type a credential
received in chat into the website. Give the exact non-secret itinerary or cart
handoff and explain what secure surface must become available.

即使连接器卡片或浏览器交接未能为用户正常渲染，上述规则仍然成立。不要把聊天输入作为后备方案。不要把在聊天中收到的凭据输入到网站。给出确切的非机密行程或购物车交接，并说明需要何种安全界面变为可用。

## Final review and approval / 最终审查与批准

At final review, obtain the exact item, date and time, traveler or party count,
selected seats/room/rate/variant, itemized charges, full total, amount due
later, credits or points used, and cancellation/refund/no-show terms.

在最终审查时，获取确切的条目、日期与时间、旅客或人数、选定的座位/房型/价格/变体、逐项费用、总额、后续应付金额、使用的储值或积分，以及取消/退款/未到场条款。

Show those terms to the user and ask for confirmation without repeating full
birth dates, contact details, identity documents, loyalty numbers, or payment
data. Continue the same task only after approval of the exact terms. If
anything material changes, stop and re-confirm the changed term.

把这些条款展示给用户并请求确认，但不要复述完整的出生日期、联系方式、身份证件、会员号或支付数据。只有在用户批准了确切条款之后才继续同一任务。如果任何实质性内容发生变化，停下并重新确认变更的条款。

## Verification and duplicate protection / 验证与重复预订防护

Treat the booking as complete only when the task sees a confirmation number,
ticket, reservation record, or unmistakable booked state. If the result is  
ambiguous:

只有当任务看到确认号、票券、预订记录或明确无误的已预订状态时，才把预订视为完成。如果结果不明确：

1. Do not submit again.
   不要再次提交。
2. Check the signed-in provider account for the booking.
   检查已登录的提供方账户中是否存在该预订。
3. Search connected email narrowly for a matching confirmation.
   在已连接的电子邮箱中做窄范围搜索，查找匹配的确认信息。
4. Retry only after establishing that no booking was created.
   只有在确认没有创建任何预订之后才重试。

## Common browser failures / 常见浏览器故障

- Custom date and party selectors: provide the exact date and count in words.
  自定义日期和人数选择器：用文字提供确切的日期和人数。
- Seat maps: report the exact section, row, seat numbers, and any obstructed
  view notation.
  座位图：报告确切的区域、排、座位号以及任何视线受阻标注。
- Late fees: totals shown before final review are provisional.
  滞期费：最终审查之前显示的总额只是暂估值。
- Hold timers: if approval arrives after expiry, refresh inventory and terms.
  占位计时器：如果批准在过期之后才到达，刷新库存与条款。
- Login, CAPTCHA, or one-time-code walls: use the supported authentication or
  consent path. Do not bypass controls.
  登录、验证码或一次性验证码屏障：使用受支持的身份验证或同意流程。不要绕过这些控制。
- Blocked automation: try another authorized provider, then
  `phone.place_call` when available, then a precise handoff. Do not report a
  blocked checkout as complete.
  自动化受阻：尝试另一个已授权的提供方，然后在可用时使用 `phone.place_call`，再不然做精确的交接。不要把受阻的结账报告为已完成。

【评论】"不要绕过 CAPTCHA/登录控制"与"受阻结账不得报为完成"共同约束代理不得伪造成功结果，是结果真实性方面的防护条款。

Do not quietly downgrade the booking to a different date, venue, property,
seat, room, rate, or price because the first checkout was difficult.

不要因为第一次结账困难，就悄悄把预订降级为不同的日期、场馆、物业、座位、房型、价格或费率。
