<!-- BILINGUAL-EN-ZH -->
# Shopify UCP Checkout / Shopify UCP 结账

Load this file only after the user has selected one or more Meta catalog
products from the same Shopify merchant whose
`is_agentic_checkout_creation_enabled` fields are all exactly `true`.

仅在用户已从同一 Shopify 商家选择了一个或多个 Meta 目录商品、且其 `is_agentic_checkout_creation_enabled` 字段全部精确为 `true` 之后，才加载本文件。

The catalog flags route the flow; the checkout endpoint remains authoritative.
Use every selected product's exact `product_id`, capability fields, and `url`
from its catalog data. Bundle products only when their catalog URLs clearly
identify the same merchant storefront; matching brand labels are not enough.
Never mix merchants in one checkout. Treat a missing or null capability as  
`false`.

目录标志负责路由流程；结账端点始终是权威。使用每个所选商品目录数据中精确的 `product_id`、能力字段与 `url`。仅当各商品的目录 URL 清楚指向同一商家店面时才打包到一起；品牌标签一致并不足够。绝不在一个结账中混用商家。缺失或为 null 的能力按 `false` 处理。

## Safety and input boundary / 安全与输入边界

- `checkout create` moves no money. `checkout complete` creates the selected
  wallet spend request and places the order.
  `checkout create` 不移动任何资金。`checkout complete` 创建所选钱包的花费请求并下达订单。
- `checkout complete` serves direct Stripe Link. It also serves direct Shop Pay
  when the checkout supports direct completion. Shop Pay uses the
  provider-approved credential and does not expose card details on this path.
  If direct completion is unavailable, use the browser route instead. On either
  browser route, the browser task places the order, so do not call `checkout
  complete`.
  `checkout complete` 服务于直连 Stripe Link。当结账支持直接完成时，它也服务于直连 Shop Pay。Shop Pay 使用提供商批准的凭据，在该路径上不暴露卡片细节。若直接完成不可用，改用浏览器路线。在任一浏览器路线上，都由浏览器任务下达订单，因此不要调用 `checkout complete`。
- Run `checkout complete` at most once for a checkout and only in the
  foreground. Never create a wallet spend request separately, background
  completion, poll it, or retry it automatically.
  每个结账最多运行一次 `checkout complete`，且仅在前台运行。绝不单独创建钱包花费请求、把完成放到后台、轮询它或自动重试。
- A denial is the user's decision. Stop without placing an order. A later retry
  requires a new explicit user message.
  拒绝即用户的决定。停止而不下单。此后重试需要一条新的、明确的用户消息。
- CLI inputs are JSON files with snake_case keys. Omit unknown optional fields
  and empty strings. Never carry endpoint-derived merchant, item, total, buyer,
  fulfillment, or legal-link data into completion input.
  CLI 输入是使用 snake_case 键的 JSON 文件。省略未知的可选字段与空字符串。绝不把端点派生的商家、商品、总额、买家、履约或法律链接数据带进完成输入。
  【评论】把资金动作限定为前台、单次、不可自动重试，并将"拒绝"定性为用户决定而非错误，是代理支付场景中典型的资金安全约束设计。

## Cart (optional) / 购物车（可选）

A pre-purchase draft, never required — `checkout create` takes `items[]`
directly. Reach for one only when the basket must survive the turn: the user is
still adding or removing, or wants to come back to it later.
`shopify-ucp-cli cart --help` covers the subcommands; carry the `cart_id` from
`agent_state.cart_id` and treat it as opaque. Four things --help does not tell  
you:

一种购前草稿，永远不是必需——`checkout create` 直接接受 `items[]`。只有当购物篮必须跨越当前回合存活时才用它：用户仍在增删商品，或想稍后回来。`shopify-ucp-cli cart --help` 覆盖各子命令；从 `agent_state.cart_id` 携带 `cart_id` 并将其视为不透明。有四件事 --help 不会告诉你：

- `cart update` replaces the whole basket — that is how an item is removed, and
  why a partial list deletes the rest. Unsure you have every item? `cart get`
  first and rebuild from `cart.line_items[]`.
  `cart update` 替换整个购物篮——这正是移除商品的方式，也是部分列表会删掉其余商品的原因。不确定自己掌握了每一件商品？先 `cart get`，再从 `cart.line_items[]` 重建。
- One cart, one merchant. A cart spanning two is refused outright:
  `multiple_merchants_not_supported`, "All cart line items must belong to the
  same merchant." Start a separate cart rather than retrying.
  一个购物车，一个商家。横跨两个商家的购物车会被直接拒绝：`multiple_merchants_not_supported`，"All cart line items must belong to the same merchant."。另起一个购物车，而不是重试。
- `cart` accepts a catalog `product_id` or the variant GID it echoes back;
  `checkout create` accepts catalog ids only. A resumed cart can be revised but
  not checked out until you search again for its catalog ids.
  `cart` 接受目录 `product_id` 或它回显的变体 GID；`checkout create` 只接受目录 id。恢复的购物车可以修订，但在你重新搜索到其目录 id 之前不能结账。
- Nothing links a cart to a checkout: it supplies items only. Cancel it once
  `checkout complete` returns `ok: true`, or once a browser task reports the
  order placed; nothing else closes it, and there is no way to enumerate the
  ones you left open.
  没有任何东西把购物车关联到结账：它只提供商品。在 `checkout complete` 返回 `ok: true` 之后、或浏览器任务报告订单已下达之后取消它；其他任何东西都不会关闭它，也无法枚举你遗留在打开状态的购物车。
- Cart, checkout, and order reads preserve provider timestamps and add semantic
  UTC and user-local forms such as `checkout_expires_at`, `order_placed_at`,
  and `order_event_occurred_at`. Do not treat an order-status message time as a
  delivery time.
  购物车、结账与订单读取保留提供商时间戳，并补充语义化的 UTC 与用户本地时间形式，如 `checkout_expires_at`、`order_placed_at` 与 `order_event_occurred_at`。不要把订单状态消息的时间当作配送时间。

## Create the checkout / 创建结账

Confirm every product, exact variant, and quantity. Gather the buyer email
required to create the checkout. Include other buyer details only when they are
already known. Never guess or fabricate a value. Checkout creation moves no
money; save final purchase approval for the completed quote.

确认每个商品、精确变体与数量。收集创建结账所需的买家邮箱。仅在已知时包含其他买家细节。绝不猜测或捏造取值。结账创建不移动资金；把最终购买批准留给完成后的报价单。

Creation needs no payment method. Do not resolve Link, connect a wallet, or ask
about payment before this call. The user picks the payment route after the
checkout exists, under *Choose the payment route* below.

创建不需要支付方式。在此调用之前，不要解析 Link、不要连接钱包、也不要询问支付。用户在结账存在之后、按下方*选择支付路线*挑选支付路线。

Create one JSON file containing every selected product. Use each catalog
`product_id` as `items[].item_id`; use one entry per distinct variant and fold
repeated identical IDs into its quantity. Quantity defaults to `1`. Do not add
a merchant field: the endpoint resolves the merchant from the catalog IDs. The
endpoint requires buyer email and USD. Include phone and address fields only
when known. Native completion additionally requires a trusted cardholder first
or last name and billing/shipping address with street, city, state, postal code,
and ISO alpha-2 country.

创建一个包含每个所选商品的 JSON 文件。用每个目录 `product_id` 作为 `items[].item_id`；每个不同变体一条记录，并把重复的相同 ID 折叠进其数量。数量默认为 `1`。不要添加商家字段：端点会从目录 ID 解析商家。端点要求买家邮箱与 USD。仅在已知时包含电话与地址字段。原生完成还额外要求可信的持卡人名或姓，以及含街道、城市、州、邮政编码与 ISO 两位字母国家代码的账单/收货地址。

Before this call, use buyer details already known from the conversation and
`~/USER.md`. Ask only for a missing email, because the endpoint requires it.
Do not ask for a name, phone number, or delivery address before checkout
creation.

在此调用之前，使用对话与 `~/USER.md` 中已知的买家细节。只对缺失的邮箱提问，因为端点要求它。不要在结账创建之前询问姓名、电话号码或收货地址。

After the user selects a wallet route, follow the Wallet setup sequence in
Payments & Wallet before asking the user for missing checkout details.
If checkout creation omitted the required name or address, pass the retrieved
values to the browser route. `checkout update` cannot add them, so do not use
direct completion for that checkout.

在用户选择钱包路线之后，先遵循 Payments & Wallet 中的钱包设置序列，再向用户询问缺失的结账细节。如果结账创建遗漏了所需的姓名或地址，把检索到的值传给浏览器路线。`checkout update` 无法补加它们，因此不要对该结账使用直接完成。

```json
{
  "buyer": {
    "email": "<email>",
    "phone_number": "<phone-string>",
    "country_code": "<country-code>",
    "address": {
      "first_name": "<first-name>",
      "last_name": "<last-name>",
      "street1": "<street1>",
      "street2": "<street2>",
      "city": "<city>",
      "state": "<state>",
      "postal_code": "<postal-code>",
      "country": "<country-alpha-2>"
    }
  },
  "items": [
    {"item_id": "<product_id-1>", "quantity": 1},
    {"item_id": "<product_id-2>", "quantity": 2}
  ],
  "currency": "USD"
}
```

```sh
HATCH_SHOPPING_PRODUCT_CONTEXTS='[<each product hatch_telemetry_context, copied verbatim>]' shopify-ucp-cli checkout create --input-file "<checkout.json>" --format json
```

Copy each runtime-authored context whole, including its eligibility flags, in
the same order as `items`. Set that same environment value on every
`shopify-ucp-cli` checkout, cart, and order command for this purchase attempt.
It is read only by local telemetry and is not sent to Shopify.

按 `items` 的相同顺序，把每个运行时生成的上下文原样整体复制，包括其资格标志。在本次购买尝试的每个 `shopify-ucp-cli` checkout、cart 与 order 命令上设置相同的环境变量值。它只被本地遥测读取，不会发送给 Shopify。

Put the complete item set in this initial create call. `checkout update` cannot
add or remove products. Do not use the separate `cart` commands to assemble
this checkout.

把完整的商品集合放进这次初始的 create 调用。`checkout update` 无法增删商品。不要用独立的 `cart` 命令来拼装这个结账。

Inspect the authoritative create response before branching on the catalog
completion capability. A returned `continue_url` does not by itself require a
browser handoff because checkouts ready for direct completion may also include
one.

在按目录完成能力分支之前，先检查权威的 create 响应。返回的 `continue_url` 本身并不要求浏览器交接，因为就绪可直接完成的结账也可能带有它。

If create returns an error or rejects the item, explain the result and offer
browser checkout from the original catalog `url`; do not retry automatically.
When the user accepts, follow
`/opt/hatch/skills/shopping/references/browser-checkout.md`. Set `stage` to
`agentic_fallback` and `reason` to `agentic_create_failed`.

如果 create 返回错误或拒绝某商品，解释结果并提供从原始目录 `url` 发起的浏览器结账；不要自动重试。当用户接受时，遵循 `/opt/hatch/skills/shopping/references/browser-checkout.md`。将 `stage` 设为 `agentic_fallback`，`reason` 设为 `agentic_create_failed`。

Only a successful create response with a usable `.agent_state.checkout_id` may continue below. Save that checkout ID. The CLI stores the endpoint-derived checkout behind the trusted runtime boundary. Inspect `.result` without copying its trusted fields into later commands.

只有带可用 `.agent_state.checkout_id` 的成功 create 响应才能继续下文。保存该结账 ID。CLI 把端点派生的结账存储在受信任的运行时边界之后。检查 `.result` 时不要把其受信任字段复制进后续命令。

A response carrying `requires_escalation`, `status: "redirect"`, or a note that
buyer detail is still missing is a successful create when it returned a
checkout ID. Do not treat it as an error. It does not say which route's brief to
send, so ask the route question before handing off.

带有 `requires_escalation`、`status: "redirect"` 或"买家细节仍缺失"提示的响应，只要返回了结账 ID 就是一次成功的 create。不要把它当作错误。它并没有说明该发送哪条路线的简报，因此在交接之前先问路线问题。

Take the checkout URL now, from `.result.continue_url` or
`.result.checkout.continue_url`. Use only a value the endpoint returned. When
it is absent, fall back to one selected product's original catalog `url` from
that merchant rather than inventing one.

现在取结账 URL，来自 `.result.continue_url` 或 `.result.checkout.continue_url`。只使用端点返回的值。当它缺失时，回退到该商家某个所选商品的原始目录 `url`，而不是发明一个。

## Choose the payment route / 选择支付路线

The checkout exists and moves no money yet. Keep a route the user already
selected. Otherwise, ask with `muse.create_options` and wait. Offer Shop Pay
with provider `shop-pay`, Link with provider `stripe-link`, and `Use another
method` through browser takeover. A connected provider, saved default, or
available method does not select a route.

结账已存在，尚未移动任何资金。保留用户已选择的路线。否则，用 `muse.create_options` 询问并等待。提供提供商为 `shop-pay` 的 Shop Pay、提供商为 `stripe-link` 的 Link，以及通过浏览器接管实现的 `Use another
method`。已连接的提供商、保存的默认值或可用方式都不构成对路线的选择。

After the user chooses, follow *Route after creation* to decide whether
checkout continues directly or through a BrowserTask.

用户选择之后，遵循*创建后的路由*来决定结账是直接继续还是经由 BrowserTask 继续。

As soon as a wallet route is settled, record it once before calling any wallet
or browser tool:

钱包路线一经敲定，在调用任何钱包或浏览器工具之前先记录一次：

```sh
shopping payment-lane-selected --lane <shop-pay|stripe-link> --selection-source user --product-contexts-json '[<each product hatch_telemetry_context, copied verbatim>]'
```

Emit one lane once. Do not emit it again when browser or direct completion
starts. If this best-effort command fails, continue the checkout unchanged.

每条路线只发一次。浏览器或直接完成开始时不要再次发送。若这一尽力而为的命令失败，结账照常继续。

After recording the route, follow the Wallet setup sequence in Payments &
Wallet. Use the exact provider ID, payment-method ID, and masked label only for
this purchase. If the user declines setup or no usable method remains, return
to route selection. Connection and method selection do not approve the
purchase.

记录路线之后，遵循 Payments & Wallet 中的钱包设置序列。精确的提供商 ID、支付方式 ID 与掩码标签仅用于本次购买。如果用户拒绝设置、或没有可用方式剩余，回到路线选择。连接与方式选择不构成对购买的批准。

When the route question is needed, ask it before any other message that follows
creation. Ask it even when the create response reports `requires_escalation`,
`status: "redirect"`, a missing shipping address, no delivery options, or a
total that is not final. None of those says which route the user wants. Do not
offer to open the checkout in the browser before the answer arrives, because
that offer picks the route.

当路线问题需要提出时，在创建之后任何其他消息之前提出。即使 create 响应报告 `requires_escalation`、`status: "redirect"`、缺失收货地址、无配送选项或总额未定，也要提出。这些都不能说明用户想要哪条路线。在答案到达之前，不要提议在浏览器中打开结账，因为那个提议本身就是在替用户选路线。

On escalation, redirect, or a name or address missing at creation, say
alongside the available options that the browser will finish this checkout and
collect what is missing. Missing delivery options and an unsettled total are
ordinary direct-checkout work under *Refresh delivery and totals* below, so do not say
the browser will place the order for those.

在升级、重定向、或创建时缺失姓名或地址的情形下，在可用选项旁边说明浏览器将完成该结账并收集缺失内容。配送选项缺失与总额未定属于下方*刷新配送与总额*之下普通的直接结账工作，因此不要为这些情形说浏览器会代为下单。

When the user selects `Use another method`, load
`/opt/hatch/skills/shopping/references/browser-checkout.md`. Continue the
existing checkout in a BrowserTask from the exact checkout URL. Include the
user's payment choice in the brief without including card details. State that
the user will enter payment during browser takeover.
Set `stage` to `agentic_fallback` and `reason` to `user_selected_browser`. The
user chose this route; no provider limit forced it.

当用户选择 `Use another method` 时，加载 `/opt/hatch/skills/shopping/references/browser-checkout.md`。从精确的结账 URL 出发，在 BrowserTask 中继续既有结账。在简报中包含用户的支付选择，但不包含卡片细节。说明用户将在浏览器接管期间输入支付。将 `stage` 设为 `agentic_fallback`，`reason` 设为 `user_selected_browser`。这条路线是用户自己选的，不是任何提供商限制所迫。

If the user does not choose, stop and wait. Do not select a route for them.

如果用户不选择，停下等待。不要替他们选择路线。

## Route after creation / 创建后的路由

For a selected wallet route, take the first branch that matches:

对已选的钱包路线，取第一个匹配的分支：

1. The user selected Shop Pay with an exact saved method: use direct completion
   only when every selected product's
   `is_agentic_checkout_completion_enabled` is exactly `true`, create did not
   report `requires_escalation` or `status: "redirect"`, and the checkout carries
   the required name and address. Otherwise use the browser Shop Pay route below
   with the exact connected payment method. Finish connection or setup in the
   parent first.
   用户选择了 Shop Pay 且带有精确的已保存方式：仅当每个所选商品的 `is_agentic_checkout_completion_enabled` 精确为 `true`、create 未报告 `requires_escalation` 或 `status: "redirect"`、且结账携带所需的姓名与地址时，才使用直接完成。否则使用下方的浏览器 Shop Pay 路线并带上精确的已连接支付方式。先在父对话中完成连接或设置。
2. The Stripe Link route, and create reported `status: "redirect"`,
   `requires_escalation`, or messages that explicitly require buyer input or
   review: the browser, carrying Link. Take this branch regardless of  
   `is_agentic_checkout_completion_enabled`.
   Stripe Link 路线，且 create 报告 `status: "redirect"`、`requires_escalation`、或明确要求买家输入或审查的消息：走浏览器，携带 Link。无论 `is_agentic_checkout_completion_enabled` 为何都走此分支。
3. The Stripe Link route, and any selected product's
   `is_agentic_checkout_completion_enabled` is not exactly `true`: the browser,
   carrying Link.
   Stripe Link 路线，且任一所选商品的 `is_agentic_checkout_completion_enabled` 并非精确 `true`：走浏览器，携带 Link。
4. The Stripe Link route, and the checkout was created without the name and
   address direct completion requires: the browser, carrying Link.
   Stripe Link 路线，且结账创建时缺少直接完成所需的姓名与地址：走浏览器，携带 Link。

Otherwise every selected product's `is_agentic_checkout_completion_enabled` is
exactly `true`, and the direct Stripe Link flow below applies.

否则，每个所选商品的 `is_agentic_checkout_completion_enabled` 都精确为 `true`，适用下方的直接 Stripe Link 流程。

On any browser branch, briefly acknowledge the handoff and end the response
after delegating. Do not poll the browser task. Do not call `checkout complete`
for that checkout.

在任何浏览器分支上，简要确认交接并在委派后结束响应。不要轮询浏览器任务。不要为该结账调用 `checkout complete`。

### Shop Pay, in the browser / Shop Pay：浏览器中完成

Spawn the task with the Shop Pay route and the selected method's masked label.
Do not include the opaque `payment_method_id` in `task`. The trusted checkout
tool revalidates the exact selected ID against a fresh wallet read before
creating approval.

以 Shop Pay 路线和所选方式的掩码标签派生任务。不要在 `task` 中包含不透明的 `payment_method_id`。受信任的结账工具会在创建批准之前，用一次新鲜的钱包读取重新校验精确的所选 ID。

```js
{
  "task": "<what the user asked for, in their words>. Open <exact Shopify checkout URL> for <selected products>. The user selected Shop Pay for this purchase with saved method <masked card label>. Complete the purchase using these known choices: <color/size/quantity/other variants>. Ask only for missing required purchase choices. Shipping preference: <deadline/budget/speed, or none>.",
  "shopping_checkout": {
    "products": [<each product hatch_telemetry_context, copied verbatim>],
    "stage": "payment_lane",
    "reason": "shop_pay_selected"
  }
}
```

Resolve the Shop Pay connection and exact method before delegating. BrowserTask
does not call wallet tools or discuss another payment route. Follow
`/opt/hatch/skills/shopping/references/browser-checkout.md` for continuation.

在委派之前先解决 Shop Pay 连接与精确方式。BrowserTask 不调用钱包工具，也不讨论其他支付路线。续接遵循 `/opt/hatch/skills/shopping/references/browser-checkout.md`。

### Stripe Link, in the browser / Stripe Link：浏览器中完成

Use the exact Stripe Link method selected above, then spawn the task. Identify
the provider and saved method with the exact provider ID and masked label. Do
not include the opaque payment-method ID in `task`.

使用上文选定的精确 Stripe Link 方式，然后派生任务。用精确的提供商 ID 与掩码标签标识提供商与已保存方式。不要在 `task` 中包含不透明的支付方式 ID。

```js
{
  "task": "<what the user asked for, in their words>. Open <exact Shopify checkout URL> for <selected products>. Use provider stripe-link with saved method <masked label>. Use these known choices for every item: <color/size/quantity/other variants>. Ask only for missing required purchase choices. Continue through checkout and hand off the exact final terms before submission. Shipping preference: <deadline/budget/speed, or none>.",
  "shopping_checkout": {
    "products": [<each product hatch_telemetry_context, copied verbatim>],
    "stage": "agentic_fallback",
    "reason": "<provider_requires_browser | agentic_completion_ineligible | buyer_details_required | stripe_link_unavailable>"
  }
}
```

Choose the reason from the first matching *Route after creation* condition.
Do not use a post-create fallback reason for an initial browser route.

从第一个匹配的*创建后的路由*条件中选择 reason。不要对初始浏览器路线使用创建后的回退 reason。

Follow `/opt/hatch/skills/shopping/references/browser-checkout.md` for
continuation.

续接遵循 `/opt/hatch/skills/shopping/references/browser-checkout.md`。

### Stripe Link, completed directly / Stripe Link：直接完成

Use the exact Stripe Link method selected above, then continue with the direct
flow.

使用上文选定的精确 Stripe Link 方式，然后继续直接流程。

## Use the selected wallet / 使用所选钱包

Reuse the selected provider and exact payment method resolved above. Do not ask
the route question again. If the selected method is no longer available, stop
before completion or browser delegation and return to *Choose the payment
route*. Do not substitute another route. A browser route taken because Stripe
Link cannot complete this checkout uses `stage: "agentic_fallback"` and  
`reason: "stripe_link_unavailable"`.

复用上文解析的所选提供商与精确支付方式。不要再次询问路线问题。如果所选方式已不可用，在完成或浏览器委派之前停下，回到*选择支付路线*。不要替换为其他路线。因 Stripe Link 无法完成该结账而走的浏览器路线使用 `stage: "agentic_fallback"` 与 `reason: "stripe_link_unavailable"`。

## Refresh delivery, discounts, and totals / 刷新配送、折扣与总额

If the checkout offers delivery options, select one. With more than one, if the
user stated a shipping preference (a deadline, budget, or speed) or the options
are trivially close, pick the best fit and tell the user which you chose;
otherwise present the options with their price and delivery estimate and let the
user choose. Update the trusted quote before completion:

如果结账提供配送选项，选择一个。当多于一个时，若用户已表明配送偏好（期限、预算或速度）或各选项极为接近，就选最合适的并告诉用户你选了哪个；否则带价格与送达估计展示各选项，让用户选择。在完成之前更新受信任的报价单：

If the checkout requires independent delivery choices for different item
groups, do not attempt direct completion; continue in the browser from the
returned checkout URL with `stage: "agentic_fallback"` and  
`reason: "provider_requires_browser"`.

如果结账要求对不同商品组做独立的配送选择，不要尝试直接完成；从返回的结账 URL 出发在浏览器中继续，使用 `stage: "agentic_fallback"` 与 `reason: "provider_requires_browser"`。

```json
{
  "checkout_id": "<checkout-id>",
  "selected_delivery_option_id": "<delivery-option-id>"
}
```

```sh
HATCH_SHOPPING_PRODUCT_CONTEXTS='<same JSON array used for create>' shopify-ucp-cli checkout update --input-file "<update.json>" --format json
```

To apply promo or coupon codes, pass the complete desired set in
`discount_codes`. Use codes the user supplied, or codes found during a deal
search the user requested. Do not invent codes or interrupt every checkout to
ask for one. Omit `discount_codes` to preserve the checkout's existing codes;
use an empty array to clear all codes. Delivery selection and discount codes
may be changed in one call:

要应用促销或优惠码，把完整的目标集合传入 `discount_codes`。使用用户提供的码，或用户请求的优惠搜索中找到的码。不要发明码，也不要在每个结账上都打断去要码。省略 `discount_codes` 以保留结账已有的码；使用空数组清除全部码。配送选择与优惠码可以在一次调用中同时更改：

```json
{
  "checkout_id": "<checkout-id>",
  "selected_delivery_option_id": "<delivery-option-id>",
  "discount_codes": ["<promo-code>"]
}
```

A discount-only update needs only `checkout_id` and `discount_codes`. Report
applied discounts, `result.checkout.rejected_discount_codes`, and the refreshed
total; rejected codes can be present even when `ok` is `true`. A checkout
without delivery options skips delivery selection, not a requested discount
update.

仅折扣的更新只需要 `checkout_id` 与 `discount_codes`。报告已应用的折扣、`result.checkout.rejected_discount_codes` 与刷新后的总额；即使 `ok` 为 `true`，也可能存在被拒的码。没有配送选项的结账跳过的是配送选择，而不是用户请求的折扣更新。

## Review and complete / 评审并完成

Use the exact saved method selected above. If the user asks to switch methods,
return to exact saved-method selection in Payments & Wallet. Do not ask the
user to confirm a switch they just requested.

使用上文选定的精确已保存方式。如果用户要求切换方式，回到 Payments & Wallet 中的精确已保存方式选择。不要让用户确认他们刚刚自己请求的切换。

Show the completed quote with the masked method, items, final total, and
delivery choice. Present this quote as the purchase review under Purchasing
Flow. For Stripe Link, ask for explicit approval and wait. For Shop Pay, do not
ask for a separate chat confirmation. `checkout complete` requests the wallet
approval that serves as final purchase confirmation. A wallet connection and
an earlier request to buy are not approval for this quote. Then write
completion input containing only the trusted checkout ID, chosen wallet
provider, chosen payment-method ID, and selected delivery-option ID when one  
exists:

展示完成后的报价单，含掩码方式、商品、最终总额与配送选择。按 Purchasing Flow 之下将此报价单作为购买评审呈现。对 Stripe Link，请求明确批准并等待。对 Shop Pay，不要请求单独的聊天确认。`checkout complete` 会发起钱包批准，作为最终购买确认。钱包连接与先前的购买请求都不构成对本报价单的批准。然后撰写完成输入，只包含受信任的结账 ID、所选钱包提供商、所选支付方式 ID，以及存在时的所选配送选项 ID：

```json
{
  "checkout_id": "<checkout-id>",
  "wallet_provider": "<stripe_link-or-shop_pay>",
  "payment_method_id": "<selected-wallet-payment-method-id>",
  "selected_delivery_option_id": "<delivery-option-id>"
}
```

Include `wallet_provider` in every completion input: use `"stripe_link"` for
Stripe Link or `"shop_pay"` for Shop Pay. For Shop Pay use the exact
instrument ID returned by `wallet.list_payment_methods`. The Shop Pay branch
keeps credentials inside trusted payment workers. The provider CLI creates the
payment approval. When the merchant supports direct Shop Pay,
the runtime adds that approval ID to the selected credential and submits it
to the merchant. If direct Shop Pay completion is unavailable, use the browser
route instead of producing card details. The runtime does not receive
a separate buyer identity token.

每次完成输入都要包含 `wallet_provider`：Stripe Link 用 `"stripe_link"`，Shop Pay 用 `"shop_pay"`。对 Shop Pay，使用 `wallet.list_payment_methods` 返回的精确工具 ID。Shop Pay 分支把凭据保存在受信任的支付 worker 内。提供商 CLI 创建支付批准。当商家支持直连 Shop Pay 时，运行时把该批准 ID 附加到所选凭据并提交给商家。若直连 Shop Pay 完成不可用，改用浏览器路线而不是产出卡片细节。运行时不会收到单独的买家身份令牌。

Omit `selected_delivery_option_id` when the checkout has no delivery selection.

当结账没有配送选择时，省略 `selected_delivery_option_id`。

```sh
HATCH_SHOPPING_PRODUCT_CONTEXTS='<same JSON array used for create>' shopify-ucp-cli checkout complete --input-file "<complete.json>" --format json
```

Read the top-level `ok`: `true` means the order was placed; `false` means it was
not. Never paste raw `.result` JSON or expose internal IDs, API fields, buyer
contact information, or shipping-address details. Summarize only available
user-facing fields: order status, products, merchant, final amount, delivery
estimate, and confirmation link. If completion fails after card save or reports
an unknown outcome, do not claim no order was placed and do not retry or switch
to browser checkout automatically.

读取顶层的 `ok`：`true` 表示订单已下达；`false` 表示没有。绝不粘贴原始 `.result` JSON，也不暴露内部 ID、API 字段、买家联系方式或收货地址细节。只总结可用的面向用户字段：订单状态、商品、商家、最终金额、送达估计与确认链接。如果完成在保存卡片之后失败、或报告结果未知，不要声称没有下单，也不要自动重试或切换到浏览器结账。
