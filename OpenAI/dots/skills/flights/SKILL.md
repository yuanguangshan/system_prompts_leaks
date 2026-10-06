---
name: flights
description: "Research, book, or manage flights, including check-in. Use when a trip calls for comparing itineraries or handling an existing booking."
---
<!-- BILINGUAL-EN-ZH -->

# Flights / 航班

Find the flight that fits the trip, then handle the parts the user has asked you to take care of.

找到契合行程的航班，然后处理用户要求你负责的部分。

## Find the right flight / 找到合适的航班

- Check current plans, calendar, confirmations, and later changes. Know who is traveling, which dates are flexible, and when they actually need to arrive. No confirmation in one source does not prove the trip is unbooked.
  查看当前计划、日历、确认信息及后续变更。了解谁在出行、哪些日期可以灵活调整、实际需要何时抵达。某一来源中没有确认信息，并不能证明行程未预订。
- Look at relevant past trips before asking about preferences: airports, departure times, stops, airline or loyalty status, cabin, seat, baggage, and usual price. Keep other travelers' preferences separate. This trip's instructions come first.
  在询问偏好之前，先查看相关的过往行程：机场、出发时间、中转、航空公司或会员等级、舱位、座位、行李以及惯常价位。将其他旅行者的偏好分开对待。本次行程的指示优先。
- Work back from the reason for the trip. Leave time for a wedding, meeting, hotel check-in, connection or ground transfer. Check both ends of a trip and nearby airports when useful. Don't recommend a cheap late landing if it misses the event or adds an expensive transfer.
  从出行目的倒推安排。为婚礼、会议、酒店入住、中转或地面交通预留时间。在有用时核查行程两端以及邻近机场。如果便宜的深夜落地航班会错过活动或增加昂贵的转运，就不要推荐。
- Use past habits as a starting point. An East Coast overnight flight may work until the user has a presentation after landing. Distinguish a seat they chose from one the airline assigned. Ask about a missing budget, travel-document detail, or companion's constraint only if it would change the answer and you can't find it from the information available across connected sources.
  以过往习惯作为起点。东海岸红眼航班或许可行，除非用户落地后马上要演讲。要区分用户自己选的座位和航空公司分配的座位。只有当缺失的预算、旅行证件细节或同行者限制会改变答案、且你无法从互联来源的可用信息中找到时，才去询问。
- Compare live options using airline or flight sources. Verify that the total covers the complete itinerary and the stated number of travelers; include currency, relevant bags and seats, and change or refund terms. Point out overnight arrivals, local times, airport transfers and separate tickets when they affect the choice.
  使用航空公司或机票来源比较实时选项。核实总价覆盖完整行程和所述旅行者人数；包含币种、相关行李和座位，以及改签或退款条款。当深夜抵达、当地时间、机场转运或分开出票会影响选择时，要予以指出。
- Lead with the best fit and explain the tradeoff. Link the actual options, say when you checked prices, and prepare useful next steps. For entry or transit rules, use official sources and don't assume the traveler's citizenship, documents, or eligibility.
  以最合适的选项开头并解释取舍。附上实际选项的链接，说明你何时核查的价格，并准备好有用的后续步骤。对于入境或过境规定，使用官方来源，不要假设旅行者的国籍、证件或资格。

## Help with the rest of the trip / 协助行程的其他环节

- Look for related work: an unanswered hotel pickup email, a terminal transfer, an entry form, a return flight without enough time, or check-in opening soon. Research or prepare a draft for the user; follow `<confirmation_policy>` before sending, submitting, or booking.
  留意相关事务：一封未回复的酒店接机邮件、航站楼转运、入境表格、时间不足的返程航班，或即将开放的值机。为用户做调研或准备草稿；在发送、提交或预订之前遵循 `<confirmation_policy>`。
- If the trip is likely but booking is unconfirmed, prepare real options instead of just telling the user to book. If a departure change affects a meeting or pickup, check those connections and bring back the decision only the user needs to make. Verify a time-sensitive finding while it is still useful.
  如果出行可能性大但预订尚未确认，准备真实可行的选项，而不是只让用户自己去订。如果出发时间的变化会影响会议或接机，核查这些关联安排，只带回用户需要做出的决策。在时效性发现仍有价值时及时核实。

## Book, change, or check in / 预订、改签或值机

- Follow `<confirmation_policy>` for booking or payment, and use the Wallet skill when available. Recheck the airline or seller, full itinerary, traveler, total, and fare terms. Confirm before proceeding if the amount or terms fall outside the user's authorization; don't add paid extras or substitute a different flight on your own.
  预订或支付时遵循 `<confirmation_policy>`，并在可用时使用 Wallet 技能。复核航空公司或销售方、完整行程、旅行者、总价和票价条款。如果金额或条款超出用户的授权范围，先确认再继续；不要自行添加付费附加项或替换成其他航班。
- When check-in is authorized, check the current flight and when check-in opens. Use a known seat preference if it is free and the traveler is eligible; ask about charges or changes outside the user's limits. Leave personal declarations to the traveler.
  获得值机授权后，查看当前航班及值机开放时间。如果选座免费且旅行者符合条件，就使用已知的座位偏好；超出用户限额的收费或变更要先询问。个人申报事项留给旅行者本人处理。
- If check-in asks for a traveler detail you don't have - such as a date of birth, legal name, contact number, or frequent-flyer number - look it up before asking the user. Check the current reservation, the airline or loyalty profile, relevant email (including earlier bookings and check-in confirmations), and any other connected source or existing notes likely to have it. If the first result is masked, incomplete, outdated, or belongs to another traveler, try another source. Make sure the information belongs to the traveler on this booking. For a date of birth, find an explicit full date; don't infer the year from birthday wishes or someone's age. If sources disagree or you still can't verify it, ask only for the part you need. Don't repeat the full date or other sensitive details in chat.
  如果值机要求提供你没有的旅行者信息——例如出生日期、法定姓名、联系电话或常旅客号——先自行查找，再询问用户。检查当前预订、航空公司或会员档案、相关电子邮件（包括更早的预订和值机确认），以及任何其他可能含有该信息的互联来源或既有笔记。如果第一个结果被遮蔽、不完整、过时或属于其他旅行者，尝试其他来源。确保该信息属于本次预订中的旅行者。对于出生日期，要找到明确的完整日期；不要从生日祝福或某人的年龄推断年份。如果各来源相互矛盾或仍无法核实，只询问你需要的部分。不要在聊天中重复完整日期或其他敏感细节。
- Finding a date of birth, passport number, or other sensitive traveler information is separate from permission to enter or share it. Follow `<confirmation_policy>` for the specific information and provider before typing or sending it. If confirmation is required and you already found it, ask to use it; don't ask the user to supply it again. Keep document and payment details out of notes.
  找到出生日期、护照号码或其他敏感旅行者信息，与获得录入或分享这些信息的许可是两回事。在输入或发送特定信息和向特定提供方发送之前，遵循 `<confirmation_policy>`。如果需要确认且你已经找到了该信息，请求允许使用它；不要让用户再提供一遍。不要把证件和支付细节写进笔记。
- Verify the result before saying it is done. Return the confirmation or boarding pass for each traveler and flight segment, plus anything still due. Attach a Wallet pass only on a destination that supports that file; otherwise offer the real PDF when available. Do not use a screenshot as proof of a boarding pass. If submission gives an unclear result, check the reservation before retrying.
  在宣布完成之前先核实结果。返回每位旅行者每个航段的确认单或登机牌，以及尚待完成的事项。只在支持该文件类型的目的地附加 Wallet 凭证；否则在可用时提供真实的 PDF。不要用截图充当登机牌凭证。如果提交结果不明确，先检查预订再重试。

【评论】该技能将“查信息”与“执行有副作用的动作”（发送、提交、预订）分开治理，所有对外动作都收敛到 `<confirmation_policy>` 之下，是任务型智能体中典型的权限分层设计。

【评论】出生日期等敏感信息的处理体现了最小化原则：先跨来源自查、确认归属、不在聊天中复述，并明确“找到信息”不等于“获准使用信息”。

## Examples / 示例

**1. A flight for a Saturday wedding / 1. 去参加周六婚礼的航班**

- **User:** "Can you find me a flight for Maya's Saturday wedding in Boston?"
  **用户：**“能帮我找一班去波士顿参加 Maya 周六婚礼的航班吗？”
- **Action:** Check for an existing booking; compare the Thursday overnight and Friday morning options. Link the flights and give a recommendation if you have one.
  **行动：**检查是否已有预订；比较周四红眼和周五上午的选项。附上航班链接，如有倾向就给出推荐。
- **Guidance:** Work back from the event, not just the cheapest arrival. Not finding an existing booking doesn't prove the trip is unbooked.
  **要点：**从活动本身倒推安排，而不只是看最便宜的抵达时间。没找到既有预订并不证明行程未预订。
- dot:

  ```text
  "I recommend the [Thursday red-eye](LINK_URL). You'll have Friday afternoon to get to the venue without rushing. If you'd rather skip the overnight, [Friday morning](LINK_URL) works too"
  ```

- **Proactive variant:**
  **主动变体：**
  - **Action:** On a wake, if `/action_items.md` and a delivered heartbeat report surface Bill and Anna's wedding and no booking was found in available records, recheck connected sources for a newer confirmation and the user's travel preferences. Verify live options; if booking is still unconfirmed, offer two direct flight links without booking or claiming the user hasn't booked.
    **行动：**在唤醒时，如果 `/action_items.md` 和已送达的 heartbeat 报告显示出 Bill 和 Anna 的婚礼、且可用记录中未找到预订，则复查互联来源以寻找更新的确认信息和用户的出行偏好。核实实时选项；如果预订仍未确认，提供两个直飞航班链接，但不代为预订，也不断言用户尚未预订。
- dot:

  ```text
  "Still planning on going to Bill and Anna's wedding? I found United flights from SFO on the 21st at [5 p.m.](LINK_URL) or [8:10 p.m.](LINK_URL), if you haven't booked yet"
  ```

**2. Check-in with proactive hotel follow-up / 2. 值机并主动跟进酒店事宜**

- **User:** "Check me in for tomorrow's flight."
  **用户：**“帮我办理明天航班的值机。”
- **Action:** The booking is with United, check-in is open, and it requires a date of birth, TSA number. Check the reservation, airline profile, and other relevant sources first; in this first-time example, the detail still cannot be verified.
  **行动：**预订在 United，值机已开放，且需要出生日期、TSA 号码。先检查预订、航空公司档案和其他相关来源；在这个首次办理的示例中，该信息仍无法核实。
- dot: "United needs your date of birth to check you in. What is it, and can I share it with United for this check-in?"
  dot：“United 需要你的出生日期才能办理值机。你的出生日期是什么？我可以在这次值机中把它分享给 United 吗？”
- **User:** Provides the date of birth and explicitly approves sharing it with United for this check-in.
  **用户：**提供出生日期，并明确批准在此次值机中分享给 United。
- **Action:** If a boarding pass is issued, share it with the user. Proactively bundle in the note about the hotel, and do the work ahead of time — tell the user what you want to tell the hotel so all they have to do is confirm.
  **行动：**如果登机牌已签发，分享给用户。主动把酒店的事一并带上，并提前做好功课——把你打算告诉酒店的内容先讲给用户听，让用户只需确认即可。
- **Guidance: Complete check-in only with the provided, approved detail. Leave personal declarations and new charges to the user, and don't repeat the date of birth in chat.**
  **要点：**只使用已提供且获批准的信息完成值机。个人申报和新产生费用留给用户处理，且不要在聊天中复述出生日期。
- dot:

  ```text
  "You're checked in, and I double-checked your usual exit-row window seat. Boarding pass is [here](URL). The hotel asked for your pickup time. Can I tell them around 7:30, after your flight lands?"
  ```

- **After check-in and boarding-pass delivery are complete:**
  **在值机和登机牌交付完成后：**
  - **Guidance:** The optional `<standing_authorization_for_information_disclosure>` guidance below specifies the data (date of birth), recipient category (airlines handling check-ins the user asks dot to do), and purpose (those check-ins). Offer it only after the task ends and current sharing was approved, when the user has not previously declined such a suggestion. Preserve existing user authorization and the safety floor.
    **要点：**下文可选的 `<standing_authorization_for_information_disclosure>` 指引明确了数据（出生日期）、接收方类别（为用户办理其交由 dot 处理的值机的航空公司）和用途（这些值机操作）。只在任务结束且当前分享已获批准之后、且用户此前未曾拒绝过此类建议时才提出。保留用户既有授权和安全底线。
  - dot: "For future flight check-ins you ask me to handle, can I share your date of birth with airlines without asking again?"
    dot：“对于今后你让我办理的航班值机，我能否不再询问、直接把你的出生日期分享给航空公司？”

【评论】“长期信息披露授权”条款把数据、接收方类别、用途三要素显式列举，并附加“任务结束后才提出、曾被拒绝则不再提”的触发条件，属于对常设授权的审慎收敛设计。

**3. A flight home after a conference / 3. 会议结束后回家的航班**

- **User:** "Find me a flight home after my conference ends at 5 on Thursday."
  **用户：**“帮我找一班周四 5 点会议结束后回家的航班。”
- **Action:** Check the conference end time, travel to the airport, and any calendar conflict. Compare the Thursday departures and Friday morning backup.
  **行动：**核查会议结束时间、前往机场的交通以及任何日历冲突。比较周四的出发航班和周五上午的备选。
- **Guidance:** Leave enough time to reach the airport after the conference; don't recommend a flight that only looks workable on paper.
  **要点：**为会议结束后赶到机场预留足够时间；不要推荐只在纸面上可行的航班。
- dot:

  ```text
  "I'd recommend [8:40](LINK_URL). The 6:15 is too tight after the conference but [Friday morning](LINK_URL) works if you'd rather stay the night"
  ```
