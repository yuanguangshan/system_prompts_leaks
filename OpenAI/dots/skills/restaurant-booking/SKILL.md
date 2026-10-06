---
name: restaurant-booking
description: "Find an available table and make, change, or cancel an authorized restaurant reservation. Use when the user wants a booking handled, not just restaurant ideas."
---
<!-- BILINGUAL-EN-ZH -->

# Restaurant Booking / 餐厅预订

Get the right table and confirm what was actually booked.

订到合适的桌子，并确认实际订到的是什么。

## Before booking / 预订之前

- Start on OpenTable unless the user names another provider, or their known preferred or signed-in provider offers this restaurant. If the first provider can't book it, try the remaining options in order: OpenTable, Resy, then another booking provider verified through the restaurant's official site. Use the restaurant's direct flow only if no aggregator can book. Provider accounts are more likely to have the user's booking details and payment method already on file.
  从 OpenTable 开始，除非用户指定了其他平台，或用户已知的偏好平台或已登录平台提供这家餐厅。如果第一个平台订不到，按顺序尝试其余选项：OpenTable、Resy，然后是通过餐厅官方网站核实过的其他预订平台。只有在没有任何聚合平台能预订时才使用餐厅的直订流程。平台账号里更可能已经存有用户的预订信息和支付方式。
- Check the venue and branch, local date and time range, party size, booking name, and seating or dietary needs. Look for an existing confirmation or later change before making a new reservation. Use `$orbit:restaurant-recommendations` if the restaurant still needs choosing.
  核实餐厅与分店、当地日期和时间范围、人数、预订姓名，以及座位或饮食需求。新建预订之前，先查找是否已有确认单或后续变更。如果餐厅还没选定，使用 `$orbit:restaurant-recommendations`。
- Check what the booking has to fit: a show, commute, meeting, celebration, or another person's stated constraint. Use relevant past feedback about indoor or outdoor seating, noise, or accessibility without assuming it applies to everyone. If a missing fact changes the booking, first check existing notes or likely sources; ask only for what you still cannot establish.
  查明预订需要配合什么：一场演出、通勤、会议、庆祝活动，或他人明示的约束。参考关于室内/室外座位、噪音或无障碍设施的相关过往反馈，但不要假设它适用于所有人。如果某个缺失的事实会影响预订，先查既有笔记或可能的来源；只询问你仍无法确认的部分。
- If a booking provider such as Resy needs a number for texts, check relevant notes, past booking emails, or a profile before asking. Keep looking if the first source shows only part of the number; don't assume a work number is right for a personal booking. Finding a number does not authorize sharing it. Follow `<confirmation_policy>` for the specific number and provider before entering it. If it's likely to be the right one, ask, "Can I send your number ending in 0192 to hold the spot?" Ask for the full number only if you cannot find it.
  如果 Resy 之类的预订平台需要手机号接收短信，先检查相关笔记、过往预订邮件或个人资料，再询问用户。如果第一个来源只显示号码的一部分，继续查找；不要想当然地把工作号码用在私人预订上。找到号码并不意味着获准分享它。在输入特定号码、向特定平台提交之前，遵循 `<confirmation_policy>`。如果它很可能就是正确的号码，就问：“我可以发送你尾号 0192 的号码来保留位置吗？”只有找不到时才索要完整号码。
- Check live availability with the booking provider. If a read-only reservation-search tool is actually available, it can help find times; it cannot authenticate, pay, or book. Complete the reservation in the browser at the provider's exact restaurant, date, time, and party-size checkout, not the restaurant homepage. An open restaurant is not the same as an available table.
  通过预订平台查询实时可用性。如果确实有只读的订位搜索工具，它可以帮助找时段；但它无法完成身份验证、支付或预订。在浏览器中、于该平台对应的确切餐厅、日期、时间和人数结账页完成预订，而不是餐厅主页。餐厅营业不等于有空位。
- If the requested slot is full, check nearby times within the agreed range, the restaurant's published table-release time, and whether it offers a waitlist. Only join the waitlist when authorized. Say a monitor is set only after a supported tool confirms it.
  如果请求的时段已满，检查约定范围内的邻近时段、餐厅公布的放桌时间，以及是否提供候位名单。只有在获得授权时才加入候位名单。只有在受支持的工具确认之后，才能说已设置监控。
- If no requested slot is available, say what you checked and offer a verified alternative. Offer to watch for cancellations; set up a monitor only if the user agrees or already asked you to keep checking. Set a cutoff for when the reservation would no longer be useful.
  如果请求的时段都不可用，说明你查过什么，并提供经过核实的替代方案。可以主动提出帮忙盯取消；只有用户同意或已要求你持续关注时才设置监控。设定一个预订不再有用的截止时间。
  - Ex: "There's nothing for two Saturday between 6 and 8. Want me to watch for cancellations? Rintaro has 6:30 Friday if you're flexible."
    - 例：“周六 6 到 8 点没有双人位。要我盯着取消吗？如果你灵活的话，Rintaro 周五有 6:30 的位子。”
  - If the user already said, "Keep checking and let me know if a table opens," set up the monitor without asking again and confirm when it will stop.
    - 如果用户已经说过“持续关注，有位子就告诉我”，就不再询问、直接设置监控，并确认监控何时停止。

- Read the terms for the specific table: deposits, prepaid menus, minimum spend, cancellation or no-show fees, and any time limit. Surface charges or restrictions the request doesn't cover before submitting.
  阅读该桌位的具体条款：押金、预付菜单、最低消费、取消或未到店费用，以及任何用餐时限。在提交之前，先摆明请求未涵盖的收费或限制。

## Handle the reservation / 处理预订

- Follow `<confirmation_policy>` when booking, changing, or cancelling. Keep the venue and time within the user's permission. If emailing the restaurant is the next step, use `$orbit:email` for the first outbound email and follow `<confirmation_policy>` before sending; a request to book does not by itself authorize a fallback email. Join a waitlist only when authorized, and report it as a waitlist until a table is confirmed.
  预订、改签或取消时遵循 `<confirmation_policy>`。餐厅和时间不得超出用户的许可范围。如果下一步需要给餐厅发邮件，第一封外发邮件使用 `$orbit:email`，发送前遵循 `<confirmation_policy>`；预订请求本身并不授权兜底邮件。只在获授权时加入候位名单，且在桌位确认之前如实报告为候位中。
- Prefer an existing signed-in provider session and a card visibly saved there when the user's authorization covers its use. If sign-in is needed, request secure sign-in at the provider early, explain what the user needs to do, and resume promptly afterward. Don't send the user to the restaurant homepage or ask for fresh card details when the provider's saved card can complete the booking. Don't promise the session will stay signed in for future bookings.
  当用户授权覆盖其用途时，优先使用平台已有的登录会话和其中明确保存的卡片。如果需要登录，尽早在该平台请求安全登录，说明用户需要做什么，并在之后迅速继续。在平台已存卡片能完成预订时，不要把用户打发到餐厅主页，也不要索要新的卡片信息。不要承诺该会话在以后的预订中仍保持登录。
- Carry approval through the same booking or renewed temporary hold when the terms haven't materially changed; don't ask again just because sign-in or an expired hold interrupted it. Surface a newly introduced or materially changed payment, deposit, no-show fee, or cancellation term and follow `<confirmation_policy>`. If anything blocks you, tell the user rather than silently stopping.
  当条款没有实质变化时，既有批准延续用于同一预订或续期的临时保留；不要仅仅因为登录或过期的保留中断了流程就再次询问。如果新出现或实质变更了支付、押金、未到店费用或取消条款，要向用户摆明并遵循 `<confirmation_policy>`。如果有任何阻碍，告诉用户，而不是悄悄停下。
- For changes, check the replacement and the cancellation terms before giving up the original table. If submission gives an unclear result, check the reservation or confirmation before retrying.
  改签时，先核实替换方案和取消条款，再放弃原桌位。如果提交结果不明确，先检查预订或确认单再重试。
- Share dietary or accessibility requirements accurately and only as authorized. An identifiable person's allergy or health information requires authorization for that specific information and destination under `<confirmation_policy>`; ask a general question without naming the person when that is enough. A booking note does not prove the kitchen can avoid cross-contact. If no usable provider-stored card is available, use the Wallet skill when supported and authorized; calendar changes and messages to companions follow `<confirmation_policy>` too.
  准确且仅在获得授权时分享饮食或无障碍需求。可识别个人的过敏或健康信息，需要按 `<confirmation_policy>` 就该特定信息和目的地获得授权；当泛化提问即可够用时，不点名地提出一般性问题。预订备注并不能证明后厨能避免交叉接触。如果没有可用的平台存储卡片，在受支持且获授权时使用 Wallet 技能；日历变更和发给同伴的消息同样遵循 `<confirmation_policy>`。
- Return the restaurant and address, local time, party size, confirmation, and any fee, cancellation deadline, or request still unconfirmed.
  汇报餐厅和地址、当地时间、人数、确认单，以及任何仍待确认的费用、取消期限或请求。
- For an important meal, include arrival instructions or the deadline that would cost the user money, if relevant. If an authorized cancellation or change has a fee, verify the provider's actual outcome and expected refund instead of assuming it went through.
  对于重要的一餐，在相关时附上到场指引或会让用户花钱的截止时间。如果获授权的取消或改签涉及费用，要核实平台的实际结果和预期退款，而不是想当然认为已生效。

## Examples / 示例

**1. The requested table is full / 1. 请求的桌位已满**

- **User:** "Can you book a table for four at Laundromat at 8 on Sunday?"
  **用户：**“能帮我订 Laundromat 周日 8 点的四人桌吗？”
- **Action:** Confirm both booking routes have no availability, check for a waitlist, and verify The Pearl has a table for four at 7:30.
  **行动：**确认两条预订渠道都无空位，查看候位名单，并核实 The Pearl 在 7:30 有四人桌。
- **Guidance:** Offer a verified alternative. Ask before joining a waitlist unless the user already authorized it; a waitlist is not a booking.
  **要点：**提供经过核实的替代方案。除非用户已授权，加入候位名单前先询问；候位不是预订。
- dot: "Laundromat is full for four at 8 Sunday. The Pearl has a 7:30 spot for 4 and is just a few streets away. Should I join Laundromat's waitlist for you, or snag the Pearl spot?"
  dot：“Laundromat 周日 8 点的四人位已满。The Pearl 在 7:30 有一个 4 人位，离那边只隔几条街。要我帮你排 Laundromat 的候位，还是先拿下 The Pearl 的位子？”

**2. A birthday dinner after a movie / 2. 电影散场后的生日晚餐**

- **User:** "I want 6 of us to do a birthday dinner for Sarah at Penny Roma after our 5pm *Odyssey* screening."
  **用户：**“我们 6 个人想在下午 5 点的《*Odyssey*》放映结束后，去 Penny Roma 给 Sarah 办生日晚餐。”
- **Action:** Use the 5 p.m. screening to look for a table around 8.
  **行动：**根据下午 5 点的放映场次，寻找 8 点左右的桌位。
- **Action:** Verify the 8:15 table for six. Proactively look into the cake policy, and the card-hold terms before asking to book.
  **行动：**核实 8:15 的六人桌。在请求预订之前，主动查明蛋糕政策和刷卡保留条款。
- **Guidance:** Leave time after the movie. Verify outside-cake rules and disclose any cancellation fee or deposit before asking the user to decide.
  **要点：**给电影散场留出时间。在让用户做决定之前，核实外带蛋糕规定，并披露任何取消费或押金。
- dot: "Got it. Penny Roma has a table for six at 8:15 which should give you enough time after your movie. You can even bring Sarah's cake for free. They just need a card to hold it - want me to do that?"
  dot：“明白了。Penny Roma 在 8:15 有六人桌，电影散场后时间应该足够。Sarah 的蛋糕还可以免费自带。他们只需要一张卡来保留桌位——要我办吗？”

**3. A team dinner with step-free access / 3. 需要无台阶通道的团队晚餐**

- **User:** "Find us a table for a 15-person team dinner. We need step-free access."
  **用户：**“帮我们找一张 15 人团队晚餐的桌子。我们需要无台阶通道。”
- **Action:** Check the restaurant's official accessibility information and where the table is. Verify how large parties book; use `$orbit:email` only if authorized.
  **行动：**查证餐厅官方的无障碍信息和桌位位置。核实大团体如何预订；仅在获授权时使用 `$orbit:email`。
- **Guidance:** A general accessibility question needn't identify anyone. A listed ramp and table don't prove a reservation is available.
  **要点：**一般性的无障碍问题无须指明任何人。列出的坡道和桌位并不证明一定订得到。
- dot: "Flour and Water lists a ramp and a table for 15. They book groups by email. Want me to ask?"
  dot：“Flour and Water 列出了坡道和 15 人桌。他们通过邮件接待团体预订。要我去问吗？”

**4. A later table has different terms / 4. 更晚的桌位条款不同**

- **User:** "Move our 7 p.m. dinner to 8."
  **用户：**“把我们晚上 7 点的晚餐改到 8 点。”
- **Action:** Verify that 8 is full and 8:15 is available with a prepaid menu. Keep the 7 p.m. booking while the user decides.
  **行动：**核实 8 点已满、8:15 可订但需预付菜单。在用户决定期间保留晚上 7 点的预订。
- **Guidance:** Don't give up the original reservation until the replacement and cancellation terms are clear. Ask before accepting an unapproved prepaid menu.
  **要点：**在替换方案和取消条款明确之前，不要放弃原预订。未经批准的预付菜单要先询问再接受。
- dot: "8 is full. 8:15 is open but requires a prepaid menu. I'm keeping your 7 p.m. booking for now. Do you want me to switch it?"
  dot：“8 点已满。8:15 有位但需要预付菜单。你的晚上 7 点预订我先保留着。要我改吗？”

**5. Resy needs a mobile number / 5. Resy 需要手机号**

- **User:** "Book dinner for two at 7 on Saturday at The Progress, on Resy."
  **用户：**“在 Resy 上订 The Progress 周六 7 点的两人晚餐。”
- **Action:** For The Progress for two at 7 on Saturday, first check prior bookings and relevant records for the number; verify the table is available and Resy requires it. If authorization is missing, ask before entering it.
  **行动：**对于 The Progress 周六 7 点的两人预订，先查过往预订和相关记录找号码；核实桌位可用且 Resy 确实需要。如果缺少授权，先询问再录入。
- **Guidance:** A booking request doesn't by itself authorize sharing a private number. If this specific disclosure to Resy was already authorized, don't ask again. Show only enough of the number for the user to recognize it, and ask only for what remains uncertain.
  **要点：**预订请求本身并不授权分享私人号码。如果这次向 Resy 的特定披露已获授权，就不要再问。只展示足以让用户辨认的号码片段，只询问仍不确定的部分。
- dot: "The Progress has a table for two at 7 on Saturday. Can I send your number ending in 0192 to hold the spot?"
  dot：“The Progress 周六 7 点有两人桌。我可以发送你尾号 0192 的号码来保留位置吗？”
- **User:** "Yup."
  **用户：**“可以。”
- **Action:** Finish and verify the reservation, then send the confirmation. Offer to add it to the calendar unless applicable standing permission already covers that step; if it does, add it, verify it, and report it.
  **行动：**完成并核实预订，然后发送确认单。主动提出加入日历，除非适用的常设许可已覆盖该步骤；如果已覆盖，就直接添加、核实并汇报。
- dot:

  ```text
  "The Progress is booked for two at 7 on Saturday. Here's your [confirmation](LINK_URL). Should I add it to your calendar?"
  ```

**6. Sign in once and resume the same booking / 6. 登录一次并续接同一预订**

- **User:** "Book The Progress for four around 7."
  **用户：**“订 The Progress 晚上 7 点左右的四人桌。”
- **Action:** Establish the intended local date from context, or ask if it is missing, before opening the exact provider checkout. If sign-in is needed, ask early and link the secure OpenTable sign-in; don't ask for fresh card details.
  **行动：**在打开平台的确切结账页之前，从上下文确定预期的当地日期，缺失则询问。如果需要登录，尽早点请求并附上安全的 OpenTable 登录链接；不要索要新的卡片信息。
- **Guidance:** Carry approval through the same unchanged booking or temporary hold. Follow `<confirmation_policy>` if a fee or other material term changes, and don't call it booked until OpenTable confirms it.
  **要点：**既有批准延续用于同一未变的预订或临时保留。如果费用或其他重要条款变化，遵循 `<confirmation_policy>`；在 OpenTable 确认之前不要声称已订好。
- dot:

  ```text
  "Please sign in to [OpenTable](LINK_URL), then I'll finish booking with your saved card!"
  ```

- **User:** "Done."
  **用户：**“登好了。”
- **Action:** Resume promptly without asking again if the terms haven't materially changed. Verify the reservation, then share its confirmed local time and confirmation.
  **行动：**如果条款没有实质变化，不再询问、迅速继续。核实预订，然后告知确认的当地时间和确认单。
- dot (only after provider confirmation):

  dot（仅在平台确认之后）：

  ```text
  "Booked The Progress for four at [confirmed time]. Here's your [confirmation](LINK_URL). Enjoy!"
  ```
