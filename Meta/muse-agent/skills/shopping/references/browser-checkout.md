<!-- BILINGUAL-EN-ZH -->

# Browser Checkout / 浏览器结账

Use browser checkout when the Purchase workflow sends a purchase through a
product or checkout page. Use this reference to start and continue the
BrowserTask.

当购买工作流需要通过商品页或结账页完成购买时，使用浏览器结账。用本参考文档来启动和续接 BrowserTask。

## Start the browser task / 启动浏览器任务

Call `browser.spawn_task` with the exact product or checkout URLs and every
choice already made for this purchase:

调用 `browser.spawn_task`，传入确切的商品或结账 URL，以及此次购买已确定的所有选择：

```json
{
  "task": "Purchase <items> from <exact product or checkout URLs>. Use these choices: <variants, quantities, delivery details, payment choice, and other requirements>. Ask only for missing required item or checkout choices. Continue through checkout and hand off the exact final terms before submission."
}
```

When a wallet route is selected, include its exact provider ID. When a saved
method is also selected, include its masked label. Do not include the opaque
payment-method ID in `task`. Include a payment refusal or checkout failure when
one already occurred. When no route is selected, omit one. The runtime adds
checkout-supported providers to the BrowserTask handoff.
Do not ask the BrowserTask to infer providers from checkout buttons. When the
user selects Link with a condition such as "if it is there" or "if available",
pass `stripe-link` as the selected provider. Do not turn that condition into a
requirement for a merchant Link button. Browser checkout permits `Use another
method` through browser takeover. Present that choice with the other eligible
routes. When the user selected it,
state that they will enter payment during browser takeover. Do not include card
details.

当选择了钱包路径时，附上其确切的提供商 ID。当同时选择了已保存的支付方式时，附上其掩码标签。不要在 `task` 中包含不透明的支付方式 ID。如果已发生过支付被拒或结账失败，要一并写明。未选择任何路径时则不写。运行时会自动把支持结账的提供商加入 BrowserTask 交接。
不要让 BrowserTask 从结账按钮自行推断提供商。当用户带条件地选择 Link（例如"如果有就用"或"如果可用就用"）时，把 `stripe-link` 作为所选提供商传入。不要把该条件转变成对商家 Link 按钮的硬性要求。浏览器结账允许通过浏览器接管使用 `Use another
method`。将这一选项与其他合格路径一起呈现。当用户选择了它时，说明他们将在浏览器接管过程中输入支付信息。不要包含卡片细节。

## Add catalog route information / 添加商品目录路径信息

Some products returned by `shopping product-details` include a
`hatch_telemetry_context`. For those products, add `shopping_checkout` to the
browser task. Copy each product's complete `hatch_telemetry_context` into
`products` without changing it. This information records why browser checkout
was used. It does not change the checkout.

`shopping product-details` 返回的部分商品带有 `hatch_telemetry_context`。对这类商品，要在浏览器任务中加入 `shopping_checkout`。把每个商品完整的 `hatch_telemetry_context` 原封不动地复制进 `products`。该信息用于记录使用浏览器结账的原因，不会改变结账本身。

Set `stage` and `reason` from the situation that started the browser task:

根据触发浏览器任务的情形设置 `stage` 和 `reason`：

| Situation | `stage` | `reason` |
|---|---|---|
| Agentic checkout creation was unavailable, and `checkout create` was not called | `checkout_start` | `agentic_creation_ineligible` |
| The user chose browser checkout before `checkout create` was called | `checkout_start` | `user_selected_browser` |
| Shop Pay must finish in the browser | `payment_lane` | `shop_pay_selected` |
| `checkout create` failed | `agentic_fallback` | `agentic_create_failed` |
| The user chose browser checkout after `checkout create` | `agentic_fallback` | `user_selected_browser` |
| The selected provider requires browser checkout | `agentic_fallback` | `provider_requires_browser` |
| Agentic checkout completion was unavailable | `agentic_fallback` | `agentic_completion_ineligible` |
| The browser must collect required buyer details | `agentic_fallback` | `buyer_details_required` |
| Stripe Link was unavailable for agentic completion | `agentic_fallback` | `stripe_link_unavailable` |

| 情形 | `stage` | `reason` |
|---|---|---|
| 智能结账创建不可用，且未调用 `checkout create` | `checkout_start` | `agentic_creation_ineligible` |
| 用户在调用 `checkout create` 之前就选择了浏览器结账 | `checkout_start` | `user_selected_browser` |
| Shop Pay 必须在浏览器中完成 | `payment_lane` | `shop_pay_selected` |
| `checkout create` 失败 | `agentic_fallback` | `agentic_create_failed` |
| 用户在 `checkout create` 之后选择浏览器结账 | `agentic_fallback` | `user_selected_browser` |
| 所选提供商要求浏览器结账 | `agentic_fallback` | `provider_requires_browser` |
| 智能结账完成环节不可用 | `agentic_fallback` | `agentic_completion_ineligible` |
| 浏览器必须收集必需的买家信息 | `agentic_fallback` | `buyer_details_required` |
| Stripe Link 无法用于智能结账完成 | `agentic_fallback` | `stripe_link_unavailable` |

Do not add `shopping_checkout` for a product found only by the browser.

对仅由浏览器发现的商品，不要添加 `shopping_checkout`。

Example:

示例：

```js
{
  "task": "<self-contained browser checkout task>",
  "shopping_checkout": {
    "products": [<complete hatch_telemetry_context for each catalog product>],
    "stage": "checkout_start",
    "reason": "agentic_creation_ineligible"
  }
}
```

## Continue the purchase / 续接购买流程

Follow the acknowledgment returned by `browser.spawn_task`. Continue the same
task with `browser.steer_task`. Do not replace it with a new task during wallet
setup or confirmation.

遵循 `browser.spawn_task` 返回的确认信息。用 `browser.steer_task` 续接同一个任务。在钱包设置或确认过程中，不要用新任务替换它。
