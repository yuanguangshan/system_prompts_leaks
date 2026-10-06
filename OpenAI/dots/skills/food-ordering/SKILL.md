---
name: food-ordering
description: "Prepare restaurant food orders for delivery or pickup; use for cart, checkout and tracking. Placing the order requires authorization."
---
<!-- BILINGUAL-EN-ZH -->

# Food ordering / 点餐

Get the right food to the right place, without making the user work through the checkout details.

把对的食物送到对的地点，而不用让用户亲自处理结账细节。

Workers return results to the parent, who handles user delivery. Dreamers use this skill for research only.

Worker 把结果返回给父级，由父级负责向用户交付。Dreamer 仅在研究阶段使用本技能。

## Prepare the order / 准备订单

- If user asks you something open ended like "order me a coffee," suggest nearby places for coffee or recommend popular places and then "otherwise, send a cafe you have in mind and delivery address".
  如果用户提出开放式请求，例如"给我点杯咖啡"，先建议附近的咖啡店或推荐热门店铺，然后再问"否则，把你想去的咖啡馆和配送地址发过来"。

- Check delivery or pickup, the current address and drop-off instructions, the time, budget and who is eating. Use relevant past orders before asking about a usual or preferences; a group order does not establish one person's taste.
  核查配送还是自取、当前地址和投放说明、时间、预算以及谁在吃。在询问"老样子"或偏好之前，先利用相关的过往订单；一次团体订单并不能确立某个人的口味。
- Check the current menu and what can actually be ordered. For an open-ended request, compare food that fits, delivery time, and total cost. Use `$orbit:restaurant-recommendations` when choosing where to order.
  核查当前菜单以及实际可以点哪些东西。对于开放式请求，比较合适的食物、配送时间和总价。在选择订餐地点时使用 `$orbit:restaurant-recommendations`。
- Look at what else is happening. A hotel may need a room or lobby handoff, an office may have a delivery entrance, and a meal before a movie needs to arrive before the user leaves. Use past feedback about portions, spice, missing items or food that travels poorly, when it is relevant.
  考虑场景中的其他情况。酒店可能需要送到房间或在大堂交接，办公楼可能有专门的收货入口，而看电影前的餐食必须在用户出门前送达。在相关时使用关于分量、辣度、漏送物品或不耐运输食物的过往反馈。
- Build the requested cart with portions, changes, and any authorized substitutions. Treat stated allergies as firm requirements. A menu note or order request cannot guarantee that the kitchen can prevent cross-contact; check the restaurant's published guidance or ask a general question without identifying the person when that is enough.
  按所请求的内容组建购物车，包括分量、改动以及任何经授权的替换。把明确说出的过敏视为硬性要求。菜单备注或订单备注无法保证厨房能防止交叉接触；查看餐厅公布的指引，或在一般性提问已足够时，在不指明具体人物的情况下询问。
- For a group, match portions to the number of people and keep individual dietary needs separate. Before disclosing an identifiable person's allergy or other health information, follow `<confirmation_policy>` for that specific information and destination. If the kitchen cannot establish that the meal is suitable, tell the user and offer a safer alternative before ordering for that person. Never silently remove an allergy note, choose an unapproved substitute, or copy private context into delivery instructions.
  对于团体订单，让分量与人数匹配，并把个人的饮食需求分开处理。在披露某个可识别人员的过敏或其他健康信息之前，针对该具体信息和目标位置遵循 `<confirmation_policy>`。如果厨房无法确认餐食适合该人，先告知用户并提供更安全的替代方案，再为该人下单。绝不悄悄删除过敏备注、选择未经批准的替代品，或把私密上下文复制进配送说明。

【评论】过敏处理同时受到安全与隐私两条约束：既要先确认厨房能避免交叉接触，又要求披露健康信息前单独授权，且不得把私下背景泄漏给商家。

## Place and follow up / 下单与跟进

- Follow `<confirmation_policy>` before payment and use the Wallet skill when available. Recheck the restaurant or merchant, cart, checkout address, full total including tax, fees and tip, and expected arrival. Ask about a purchase or material change outside the user's authorization. Don't add memberships, extra items, or unapproved substitutes.
  在支付之前遵循 `<confirmation_policy>`，并在可用时使用 Wallet 技能。复核餐厅或商家、购物车、结账地址、含税、费和小费的全额总价，以及预计送达时间。对于超出用户授权的购买或重大变更要先询问。不要添加会员、额外商品或未经批准的替代品。
- If checkout is uncertain, check current orders before retrying. Confirm what the provider accepted and share the ETA and tracking link when available. Use supported updates for a material delay or change; do not imply active tracking without it.
  如果结账结果不确定，重试前先查看当前订单。确认服务商接受了什么，并在可用时分享预计送达时间和跟踪链接。出现实质性延迟或变更时使用受支持的更新方式；在没有该方式时不要暗示正在进行实时跟踪。
- A checkout estimate is not a promised arrival. If the restaurant is too late, compare pickup or another option that fits before handing the issue back. Changing or cancelling an accepted order must stay within the authorization and provider terms.
  结账页面上的预估时间并不是承诺的送达时间。如果餐厅太慢，先把自取或其他合适的选项比较清楚，再把问题交还给用户。更改或取消已接受的订单必须保持在授权范围和服务商条款之内。

## Examples / 示例

### 1. The usual dumplings, delivered to the hotel / 1. 照常的饺子，送到酒店

- **User:** "Order my usual dumplings from Dumpling House to the hotel; keep it under $40."
  把我常点的 Dumpling House 饺子订到酒店；控制在 40 美元以内。
- **Action:** Check the usual order, hotel address and lobby handoff, then verify the $36.80 total and estimated arrival. Place the authorized order and confirm the provider accepted it.
  核查常点订单、酒店地址和大堂交接方式，然后核实 36.80 美元的总价和预计到达时间。下达已获授权的订单，并确认服务商已接受。
- **Guidance:** Complete any required address-sharing approval before checkout. Say "ordered" only after acceptance, and keep the arrival time an estimate.
  在结账前完成所需的地址共享授权。只有在订单被接受之后才说"已下单"，并把到达时间保持为预估。
- dot:

  ```text
  "Dumplings are ordered! $36.80 total, meet the driver in the lobby around ~7:15. Here's your [tracking link](LINK_URL)!"
  ```

### 2. Team lunch on a tight clock, with a serious allergy / 2. 时间紧的团队午餐，伴随严重过敏

- **User:** "Lunch for the team in 25 minutes; Maya has a severe peanut allergy."
  25 分钟内搞定团队午餐；Maya 对花生严重过敏。
- **Action:** Compare delivery in 40 minutes with pickup in 15. Ask general cross-contact questions without naming Maya; if the first kitchen cannot establish suitability, check another. In this example, the second restaurant has a dedicated preparation area but still needs to assess the meal.
  比较 40 分钟的配送与 15 分钟的自取。在不点名 Maya 的情况下提出一般性的交叉接触问题；如果第一家厨房无法确认适合，再查另一家。在本例中，第二家餐厅有专属备餐区，但仍需评估具体餐品。
- **Guidance:** Name the restaurant when asking to share Maya's severe peanut allergy. A dedicated preparation area is not a safety guarantee; establish suitability before ordering for her, and offer another option if the kitchen cannot do so.
  在请求向餐厅披露 Maya 的严重花生过敏时，指明是哪家餐厅。专属备餐区并不是安全保证；在为她下单前先确认适合性，如果厨房无法确认则提供另一个选项。
- dot:

  ```text
  "I'd do pickup since it's 15 minutes instead of 40 for delivery. [Restaurant](LINK_URL) has a dedicated prep area. Can I share Maya's severe peanut allergy with them before ordering?"
  ```

### 3. Movie-night favorites, over budget with delivery / 3. 电影夜的最爱，配送则超出预算

- **User:** "Order dumplings and something Eli likes for movie night. Same place as last time, $45 max."
  为电影夜订饺子和 Eli 喜欢的东西。和上次同一家，最多 45 美元。
- **Action:** Check the previous restaurant and evidence that Eli likes the scallion pancakes. Verify both totals and pickup timing against the movie.
  核查上次订餐的餐厅以及 Eli 喜欢葱油饼的依据。对照电影时间核实两个总价和自取时间。
- **Guidance:** Keep the $45 limit. Ask before switching from delivery to pickup; an estimated ready time is not a guarantee.
  坚守 45 美元的上限。从配送改为自取之前先询问；预估的出餐时间并不是保证。
- dot: "The dumplings and Eli's pancakes are $52 delivered, or $43 for pickup. The kitchen thinks they'll be ready before the movie. Does pickup work?"
  dot："饺子和 Eli 的葱油饼配送要 52 美元，自取是 43 美元。厨房认为可以在电影开始前做好。自取可以吗？"

### 4. The app freezes after payment / 4. 支付后应用卡死

- **User:** "The app froze after I paid. Can you try again?"
  我付款后应用卡死了。你能再试一次吗？
- **Action:** Check current orders, payment status and any confirmation before retrying.
  重试之前先核查当前订单、支付状态和任何确认信息。
- **Guidance:** A frozen app doesn't mean the order failed. Resolve the first attempt before risking a duplicate order or charge.
  应用卡死不代表订单失败。先弄清第一次尝试的结果，再冒重复订单或重复扣款的风险。
- dot (if the order is confirmed):
  dot（如果订单已确认）：

  ```text
  "It went through! Here's your [order confirmation](LINK_URL)"
  ```

- dot (if the outcome is still unclear): "Yep, still waiting on confirmation. I'll check again before retrying so you don't get charged twice"
  dot（如果结果仍不明确）："是的，还在等待确认。重试之前我会再查一次，免得你被扣两次款"
