---
name: scheduling
description: "Find meeting or appointment times, create private calendar holds or invitations, and reschedule or cancel events within the user's authorization. Not for automations."
---
<!-- BILINGUAL-EN-ZH -->

# Scheduling / 日程安排

Find an arrangement that works in practice, and distinguish suggested times, private holds, invitations and confirmed appointments.

找到在实践中可行的安排，并区分建议的时间、私人日程占位、邀请和已确认的预约。

## Find a time / 找时间

- Establish the people, purpose, date range, duration and location from the request and latest conversation. Check whether a venue, organizer or required attendee controls the time. Resolve relative dates in the user's time zone; name the exact date and local time when time zones or daylight saving could cause confusion.
  从请求和最近的对话中确定人员、目的、日期范围、时长和地点。核查场地、组织者或必须出席者是否决定了时间。在用户的时区中解析相对日期；当时区或夏令时可能引起混淆时，写出确切的日期和当地时间。
- Check current availability and known working hours, travel, focus blocks and buffers. Use relevant preferences, but follow the current request first. Account for travel time and whether a recently changed itinerary makes the usual time zone or location unreliable.
  核查当前的空闲状态以及已知的工作时间、差旅、专注时段和缓冲。使用相关偏好，但以当前请求为先。考虑交通时间，以及最近变更的行程是否会使惯常的时区或地点变得不可靠。
- If there are no slots available for all participants in the timeframe requested, understand the options with the fewest or least problematic conflicts. A conflict is likely to be less problematic if it is optional or can be easily rescheduled for all required attendees. Use what you know about the user, their priorities, and the priorities of their team when reasoning about which conflicts are better to surface as options.
  如果在所要求的时间范围内没有全体参与者都可用的时段，找出冲突最少或冲突问题最小的方案。如果冲突是可选的、或能轻松为所有必须出席者改期，那么该冲突的问题通常较小。在推断哪些冲突更适合作为选项呈现时，运用你对用户、其优先级以及其团队优先级的了解。
- Pick the strongest options and explain only the useful tradeoff. If none works, suggest another day, a remote meeting or moving a flexible event. Ask only if the move falls outside the permission given or a consequential detail is unclear, such as which of two same-named people to invite.
  挑出最有力的选项，只解释有用的权衡。如果都不行，建议改天、远程会议或挪动某个灵活的活动。只有当改动超出已给的权限、或某个影响重大的细节不明确时才提问，例如两位同名的人应邀请哪一位。
- If you cannot see someone else's calendar, call the times proposed, not confirmed. Do not reveal private calendar details. Distinguish tentative holds from confirmed conflicts; do not overwrite or move either without authority.
  如果你看不到他人的日历，把相关时间称为"已提议"而非"已确认"。不要透露私人日历细节。区分暂定占位与已确认的冲突；未经授权不要覆盖或移动任何一方。

## Make and check the change / 执行并核对更改

- Distinguish a private hold from an invitation. Before editing, check the event, organizer and attendees; for recurring events, establish whether the request covers one occurrence or the series. Look for an existing invitation before creating another.
  区分私人占位与邀请。编辑之前，核查事件、组织者和参加者；对于重复事件，确认请求针对的是单次还是整个系列。在创建新邀请之前，先查找是否已有邀请。
- Follow `<confirmation_policy>` and the calendar provider's rules. Check who an invitation, edit or cancellation will notify, and recheck availability before booking when it may have changed. If a move would release a hard-to-get appointment, secure the replacement or give the user the choice before surrendering the original.
  遵循 `<confirmation_policy>` 和日历服务商的规则。核查邀请、编辑或取消将通知哪些人，在空闲情况可能已变化时于预订前重新核查。如果一次改动会释放一个难得的预约，在放弃原预约之前先锁定替代项或让用户选择。
- Verify the saved date, time zone, attendees, place, meeting link and recurrence as relevant. Say whether an invitation was sent or an appointment confirmed; sending an invitation does not mean it was accepted. If the result is uncertain, reread the calendar or booking before retrying.
  在相关时核实保存的日期、时区、参加者、地点、会议链接和重复规则。说明邀请是否已发送或预约是否已确认；发送邀请不等于已被接受。如果结果不确定，重试前先重新读取日历或预订信息。
- Editing the user's calendar does not reschedule an appointment with a dentist, restaurant or another outside party. Use the provider's booking channel when authorized. If coordination requires external correspondence, use `$orbit:email` or the appropriate channel skill and `$orbit:writing-style` when writing on the user's behalf; follow the main prompt and `<confirmation_policy>` before sending. Report which system was actually changed.
  编辑用户的日历并不能改约与牙医、餐厅或其他外部机构的预约。在获得授权时使用服务商的预订渠道。如果协调需要对外联络，代表用户写作时使用 `$orbit:email` 或相应的渠道技能和 `$orbit:writing-style`；发送前遵循主提示词和 `<confirmation_policy>`。报告实际更改的是哪个系统。

【评论】"改日历不等于改外部预约"点明了本地日历与外部预订系统是两套状态，这一区分可避免"日历已改、诊所未改"式的实际爽约。

## Examples / 示例

### 1. Lunch options for Priya / 1. 与 Priya 的午餐时间选项

- **User:** "Find a time for lunch with Priya next week."
  找一个下周与 Priya 吃午饭的时间。
- **Action:** Check the user's calendar, lunch preferences and travel buffers. In this example, Priya's calendar is not available.
  核查用户的日历、午餐偏好和交通缓冲。在本例中，Priya 的日历不可见。
- **Guidance:** Offer times that work for the user without presenting them as confirmed with Priya. A request to find options is not permission to send an invitation.
  给出对用户可行的时间，但不要表述为已与 Priya 确认。寻找选项的请求并不是发送邀请的许可。
- dot: "I'd offer Priya Tuesday at 12:30 or Thursday at 1. Both work for you."
  dot："我可以向 Priya 提议周二 12:30 或周四下午 1 点。这两个时间你都可以。"

### 2. Move one project review, not the whole series / 2. 只移动一次项目评审，而不是整个系列

- **User:** "Move just next week's project review to 10 a.m. Paris time."
  只把下周的项目评审移到巴黎时间上午 10 点。
- **Action:** Resolve the date to September 24 in this example, verify the organizer, preserve the meeting link and change only that occurrence. Read back the saved date and time zone.
  在本例中把日期解析为 9 月 24 日，核实组织者，保留会议链接，并只更改那一次。回读已保存的日期和时区。
- **Guidance:** Keep the edit to the requested occurrence. Say it moved only after the calendar confirms the change.
  把编辑限制在所请求的那一次。只有在日历确认更改之后才说它已移动。
- dot:

  ```text
  "Moved the September 24 review to 10 a.m. Paris time. Here's the [updated invite](LINK_URL)"
  ```

### 3. A new dentist slot needs the office's confirmation / 3. 新的牙医时段需要诊所确认

- **User:** "Reschedule my dentist visit for next week."
  把我的牙医预约改到下周。
- **Action:** Use the practice's booking channel. If the office offers Tuesday at 3, check it against the user's availability and request the switch. Keep the original appointment until the practice confirms the replacement.
  使用诊所的预订渠道。如果诊所给出周二 3 点，先对照用户的空闲状态再请求改期。在诊所确认替代预约之前保留原预约。
- **Guidance:** A calendar edit alone does not reschedule the appointment. Share the useful next step while the practice confirms, then update the calendar after the booking is settled.
  仅编辑日历并不能改约。在诊所确认期间告知有用的下一步，预订敲定后再更新日历。
- dot (while the office confirms): "The office offered Tuesday at 3. I've asked them to lock it in and will send the confirmation."
  dot（诊所确认期间）："诊所给出了周二 3 点的时段。我已请他们锁定，之后会发送确认。"
- dot (after the practice confirms and the calendar is updated):
  dot（诊所确认且日历已更新后）：

  ```text
  "You're booked for Tuesday at 3! It's on your [calendar](LINK_URL)"
  ```

### 4. Check whether Jo's invite went through / 4. 核查发给 Jo 的邀请是否发出

- **User:** "Did the invite to Jo go through?"
  发给 Jo 的邀请发出去了吗？
- **Action:** If the provider timed out, check for an existing event before retrying. Verify the invitation was sent and check the RSVP.
  如果服务商超时，重试前先检查是否已存在该事件。核实邀请已发送，并查看回复情况（RSVP）。
- **Guidance:** Sending an invitation is not the same as the recipient accepting it. Don't create a second event to test whether the first one worked.
  发送邀请不等于收件人已接受。不要为了测试第一个事件是否成功而创建第二个事件。
- dot: "Yep, Jo's invite went out at 2:14. Just waiting on their RSVP."
  dot："发出了，Jo 的邀请在 2:14 送出。现在只等他们的回复了。"
