---
name: restaurant-recommendations
description: "Find restaurants that fit the people, occasion, location and budget. Use for choosing where to eat or order from, not for placing an order or making a reservation."
---
<!-- BILINGUAL-EN-ZH -->

# Restaurant Recommendations / 餐厅推荐

Find the place you'd actually send these people to, then check that it works for the occasion.

找到你真正愿意推荐这些人去的地方，然后核实它是否适合该场合。

## Find the right place / 找到合适的地方

- Start with who is going, why, where, when, and how much they want to spend. Check the conversation, plans, and past choices before asking. Look for what they liked or disliked, such as a loud room, long trip, formal service, or small portions.
  先弄清谁去、为什么去、去哪里、什么时候去，以及想花多少钱。在提问之前，先查看对话、计划和以往的选择。寻找他们喜欢或不喜欢的东西，例如吵闹的环境、路途遥远、正式的服务或分量太小。
- Keep each person's needs separate. A companion's order does not establish the user's preference, and a reservation says little about whether anyone enjoyed it. Treat stated allergies and dietary restrictions as firm requirements; don't infer them from orders. If a venue needs identifiable allergy or other health information, follow `<confirmation_policy>` for that specific information and destination. You can often research or ask a general question without identifying anyone.
  把每个人的需求分开对待。同伴点的菜不能确立用户的偏好，订过位也说明不了是否有人吃得满意。把明确说出的过敏和饮食限制当作硬性要求；不要从点单记录中推断。如果场所需要可识别个人身份的过敏或其他健康信息，就按 `<confirmation_policy>` 处理该具体信息与目的地。通常你无需识别任何人的身份就可以做调研或提出一般性问题。
- Look at the day around the meal: where people are coming from, the next event, how long they have, and whether they need a quiet room, quick service, an accessible entrance, or a place that works for children. Balance the group's needs rather than optimizing only for the user.
  审视这顿饭前后的当天安排：人们从哪里来、下一个日程是什么、时间有多充裕，以及是否需要安静的包间、快捷的服务、无障碍入口，或适合儿童的地方。要平衡全组的需要，而不是只针对用户做优化。
- Check current menus, hours, location, prices and booking options. Use the restaurant itself for facts and recent independent local coverage for what the experience is like. For a special or expensive meal, look for more than one credible opinion; sponsored or repeated coverage does not count as separate evidence. Check details that could rule it out, such as a fixed menu, accessibility or when the kitchen closes.
  核实当前的菜单、营业时间、位置、价格和预订选项。事实类信息以餐厅自身为准，用餐体验则参考近期独立的本地报道。对于特殊或昂贵的一餐，要寻找不止一个可信的意见；软文或重复的报道不能算作独立证据。检查那些可能一票否决的细节，例如固定菜单、无障碍设施或厨房打烊时间。
- If the date and party size are known, look for a real table. Say when availability is unconfirmed; one empty booking search doesn't mean the restaurant is full. A menu label doesn't confirm allergy safety or cross-contact practices. Note what the venue still needs to answer.
  如果已知日期和人数，就去查找真实的空位。当空位情况未确认时要如实说明；一次订位搜索没有结果不代表餐厅已订满。菜单上的标注并不能证实过敏安全性或防交叉接触的做法。记录下该场所仍需答复的问题。
- For delivery or pickup, consider the actual distance, current menu, and whether the dishes travel well. A great sit-down restaurant may be a weak delivery choice. Use `$orbit:food-ordering` if the user wants the order prepared or placed.
  对于外卖或自取，要考虑实际距离、当前菜单，以及菜品是否经得起配送。一家很棒的堂食餐厅可能是糟糕的外卖选择。如果用户想要准备或下达订单，使用 `$orbit:food-ordering`。

## Give a useful answer / 给出有用的回答

- Lead with the best choice and why it fits. Add alternatives when they offer a useful difference. Include expected spend, a dish or two worth ordering when supported, the caveat most likely to change the choice, and links to the menu or booking. Add travel time when useful. Use `$orbit:restaurant-booking` if asked to reserve, and follow `<confirmation_policy>` if a message to the venue would help.
  以最佳选择及其契合理由开头。当备选项具有有意义的差异时再补充。内容包括预计花费、在支持的情况下值得一两道推荐菜、最可能改变选择的注意事项，以及菜单或预订链接。在有用时加上路上时间。如果被要求预订，使用 `$orbit:restaurant-booking`；如果向场所发消息会有帮助，则遵循 `<confirmation_policy>`。
- If the favorite is booked or badly timed, find a workable second option instead of sending the user back to search. If they later tell you how it went, use that feedback to improve future suggestions; their own reaction is better evidence than a reservation or order.
  如果首选已订满或时间不合适，就找一个可行的次选，而不是让用户自己重新去搜。如果他们之后告诉你结果如何，就把该反馈用于改进今后的推荐；用户本人的反馈比一次订位或点单更有证明力。

【评论】该技能把"同伴点单≠用户偏好"与"过敏信息须按确认政策处理"写成硬性规则，是对推荐场景中偏好误读与健康责任风险的针对性防御。
