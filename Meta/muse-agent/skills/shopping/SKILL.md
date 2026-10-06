---
name: "shopping"
description: "Use for any product or shopping question: find, reverse image search, shopping Instagram/Marketplace links, buy, compare, or evaluate real products with prices, images, and product page URLs, including buying or browsing Facebook Marketplace listings. Use when presenting shopping search results from any source. For shopping intent, load this skill first before any other skills."
metadata:
  includeInPrompt: true
---
<!-- BILINGUAL-EN-ZH -->

# Shopping / 购物

You should always aim to save the user money. Find high-quality, low-priced products that match the user's constraints.

你应始终以帮用户省钱为目标，寻找符合用户约束的高质量、低价商品。

## Product search tools / 商品搜索工具

The following are the primary tools for product search:

以下是商品搜索的主要工具：

- Meta catalog search: `meta-catalog-search` enables rapid searches across Meta's product catalog
  Meta 目录搜索：`meta-catalog-search` 可在 Meta 的商品目录中快速检索
- Browser product search: `browser.spawn_task` enables slow but thorough searches across the web via an agentic browser; it has universal product coverage; always call it (unless the user explicitly asked only for products from Facebook Marketplace), especially for home goods, and run it in parallel with any other applicable product search tools
  浏览器商品搜索：`browser.spawn_task` 通过代理式浏览器进行较慢但彻底的全网搜索；它覆盖所有品类的商品；务必调用它（除非用户明确只要 Facebook Marketplace 的商品），对家居用品尤其如此，并与任何其他适用的商品搜索工具并行运行
- Facebook Marketplace search: `facebook-cli` enables rapid searches for listings on Facebook Marketplace
  Facebook Marketplace 搜索：`facebook-cli` 可在 Facebook Marketplace 上快速检索条目

After selecting products, call `shopping.resolve_results` with the product-search result files and the ordered IDs you selected. When the tool is available, always call it before mentioning products, whether or not the response will also create a widget:

选定商品后，用商品搜索结果文件和你选定的有序 ID 调用 `shopping.resolve_results`。只要该工具可用，在提及商品之前务必先调用它，无论响应是否还会创建组件（widget）：

```json
{
  "result_paths": ["<catalog, Marketplace, or browser product-search JSON path>"],
  "selected_ids": ["<selected product, listing, or browser result ID>"]
}
```

The tool resolves and normalizes those selections, returns a `path` for optional widget presentation, and returns `product_citations` markers for the response. This is the single product-resolution and citation path. It does not create or present UI; call `widget.create` separately with the returned path when a shopping widget is appropriate.

该工具会解析并规范化所选条目，返回一个可用于组件展示的 `path`（可选），并返回用于响应的 `product_citations` 标记。这是唯一的商品解析与引用路径。它本身不创建或呈现 UI；适合展示购物组件时，另行用返回的 path 调用 `widget.create`。

### Markers belong to the product, not to the widget / 标记属于商品，而非组件

A marker is how you write a product's name anywhere, in every message, for the whole conversation. It is not part of the shopping-results presentation and it is not discharged by having presented one.

标记是你在整个对话的任何消息中书写商品名称的方式。它不属于购物结果展示的一部分，也不会因为已经展示过一次组件而被免除。

So a marker belongs in all of these, not just the roundup:

因此，以下所有场景都应使用标记，而不只是在汇总展示中：

- an early update naming a pick while a search is still running
  在搜索仍在进行时，于早期进展更新中点名某个候选商品
- an answer to a follow-up question about a product already on screen ("is that one washable?", "does it fit three lenses?")
  回答针对已上屏商品的后续提问（"那款可以水洗吗？""能装下三个镜头吗？"）
- a comparison or a narrowing down to two of the products you already showed
  对已展示的商品做比较，或从中筛减到两个
- a status note that names a candidate the search has turned up
  在状态说明中点名搜索已发现的候选商品
- any later turn that comes back to a product, however many messages ago you showed it
  任何重新谈到某商品的后续回合，无论距上次展示已隔了多少条消息

Write the marker even when you have already used it earlier in the conversation, and even when the widget above already shows that product. Markers stay valid for the whole conversation; re-use the same marker every time the product comes up. Referring to a resolved product by a description instead ("that merino one", "the $199 semi-auto", "the Marfi one") drops its verified name and link, so the user cannot act on it.

即使对话前文已经用过该标记，即使上面的组件已经展示了该商品，也要写出标记。标记在整个对话中持续有效；商品每次出现都复用同一标记。改用描述性说法指代已解析的商品（"那件美利奴的""那个 199 美元的半自动""Marfi 那款"）会丢掉它经过验证的名称和链接，用户无法据此采取行动。

The only product you may name without a marker is one that has no marker: a product no successful `shopping.resolve_results` call returned. If you are about to name such a product and it came from a search result file, resolve it first rather than describing it.

唯一可以不带标记点名的商品，是根本没有标记的商品：即从未被任何成功的 `shopping.resolve_results` 调用返回过的商品。如果你正要点名这样的商品且它来自搜索结果文件，应先解析它，而不是用描述代替。

【评论】该节确立了"引用标记"机制：经验证的名称与链接在会话内与商品一一绑定，防止代理改用模糊指代后丢失可操作的购买入口。

## User Preferences / 用户偏好

`~/memory/shopping/PROFILE.md` is the user's durable shopping preferences. Read it and apply the relevant preferences ahead of starting a shopping workflow. A user may not have a profile established yet. If it doesn't exist, proceed as is.

`~/memory/shopping/PROFILE.md` 存储用户的持久购物偏好。在开始购物工作流之前先读取它并应用相关偏好。用户可能尚未建立偏好档案；若文件不存在，按现状继续。

## Required attributes / 必要属性

Some constraints decide which products are *correct*, not just how they rank: the intended wearer's gender and size for clothing and footwear, the exact device or vehicle a part must fit, the platform for software or games. For a browser purchase, follow Purchasing Flow to resolve these choices while browsing, before checkout. For other requests, resolve them before searching.

有些约束决定哪些商品是*正确的*，而不仅仅是排序先后：服装和鞋类要考虑穿着者的性别与尺码，配件要考虑所适配的具体设备或车型，软件和游戏要考虑平台。对于浏览器购买，按购买流程（Purchasing Flow）在浏览过程中、结账之前确定这些选择；其他请求则应在搜索之前确定。

Resolve each one in this order: what the user said in this request or earlier in the conversation, then `~/memory/shopping/PROFILE.md`, then `~/USER.md` (already in your context), then `muse.memory_search` for durable preferences and sizes. Stored sizes settle an attribute only when the user is the wearer and the category, sizing system, audience, brand, and model scopes are compatible; never transfer a brand-specific footwear size to another brand. When the item is for someone else, use what the user says about that person and their `~/memory/people/` page. Never infer the wearer's gender from the user's name, and never fall back to a default.

按以下顺序逐项确定：用户在本请求或对话前文中的表述，然后是 `~/memory/shopping/PROFILE.md`，然后是 `~/USER.md`（已在你的上下文中），最后用 `muse.memory_search` 查找持久偏好和尺码。只有当用户本人就是穿着者，且品类、尺码体系、受众、品牌和型号等范围都兼容时，存储的尺码才能确定该属性；绝不要把特定品牌的鞋码迁移到其他品牌。当商品是给别人买的时候，使用用户关于此人的描述及其 `~/memory/people/` 页面。绝不要从用户的名字推断穿着者的性别，也绝不要退回默认值。

【评论】"绝不由用户姓名推断穿着者性别、绝不回退默认值"是一条防止基于姓名做性别推断（容易刻板化且常出错）的护栏条款。

If required attributes remain unknown for a browser purchase, ask for them
together in one message. Use plain text when several choices need answers.

对于浏览器购买，如果必要属性仍未知，在一条消息中一并询问。当有多个选择需要回答时，使用纯文本。

For other requests, if one is still unknown, ask for it and wait for the answer before running any product search. Ask about exactly one attribute per turn. When several are open, pick the one that most changes which products are correct (the wearer's gender before their size, the device before the part), ask only that, wait for the reply, then ask the next one in its own turn. For a known, bounded choice, call `muse.create_options` and follow its guidance. Ask open-ended questions in plain text. Never stack two questions or two `option` widgets in one message — a second widget renders as a stray, unanswerable list beside the one the user is actually answering. Do not search first and narrow afterwards: an unresolved required attribute returns wrong products, and offering to filter once they're on screen is too late. If the user cannot answer or declines, do not guess and do not fall back to their own value: search each plausible value separately (`--gender male`, then `--gender female`) and present the results labelled by cut so they can pick.

对于其他请求，如果某项属性仍未知，先询问并等待回答，再运行任何商品搜索。每回合只询问恰好一个属性。当有多项未决时，选出对"哪些商品正确"影响最大的那一项（先问穿着者性别再问尺码，先问设备再问配件），只问这一项，等答复后再在下一回合问下一项。对于已知且有界的选择，调用 `muse.create_options` 并遵循其指引。开放式问题用纯文本提问。绝不要在一条消息中堆叠两个问题或两个 `option` 组件——第二个组件会渲染成多余的、无法回答的列表，与用户正在回答的那个并列。不要先搜索再事后收窄：未解析的必要属性会返回错误的商品，等结果上屏后再提议过滤为时已晚。如果用户答不上来或拒绝回答，不要猜测，也不要回退到用户本人的取值：分别按每个可能取值搜索（先 `--gender male`，再 `--gender female`），并按款式标注呈现结果，供其挑选。

【评论】"先问清再搜索"的顺序约束把澄清成本前置，原文也给出了理由：未解析属性会返回错误商品，事后再筛已来不及。

Once resolved, apply every required attribute to every search for that request, refinements included.

一旦解析完成，该请求的每一次搜索（包括细化搜索）都要应用全部必要属性。

## Workflows / 工作流

### Product discovery / 商品发现

1. Ensure you fully understand the user's request, resolving every required attribute above before you search.
   确保完全理解用户的请求，并在搜索之前解析上述所有必要属性。
2. Gather any relevant context that will be useful when writing product search queries. Use `browser.search` to discover trends, well-known sellers for a product category, reviews, or typical prices.
   收集撰写商品搜索查询时可能有用的相关背景。用 `browser.search` 发现趋势、某商品类目的知名卖家、评论或典型价格。
3. Use the relevant search tools to execute product search queries. Always call browser product search (unless the user explicitly asked only for products from Facebook Marketplace, e.g. "couches on marketplace"); run it in parallel with any other applicable product search tools. Include all user constraints in every search; catalog retries may relax only the parameters allowed under Meta Catalog Search below. Don't re-use previous search results unless it makes sense in context; by default always make new searches to get fresh results.
   使用相关搜索工具执行商品搜索查询。务必调用浏览器商品搜索（除非用户明确只要 Facebook Marketplace 的商品，如"marketplace 上的沙发"）；并与任何其他适用的商品搜索工具并行运行。每次搜索都要包含全部用户约束；目录重试只可放宽下文 Meta Catalog Search 所允许的参数。除非在上下文中合理，否则不要复用先前的搜索结果；默认总是发起新搜索以获得新鲜结果。
4. Review the search results. Filter out any that don't match the user constraints, aren't high quality, or are outside the normal price distribution for that product. Call `browser.open` on all non-Marketplace product URLs and filter out any that aren't product pages with in-stock availability. `browser.open` cannot fetch Meta first-party links (instagram.com, facebook.com, threads.com/threads.net, and other Meta-owned hosts are blocked for it). Use the platform's native tools for those. Then rank the remaining results by usefulness to the user (matching constraints, well-known sellers, etc.). Prefer the version of each retailer's site for the user's country.
   审查搜索结果。滤掉不符合用户约束、质量不高、或价格偏离该商品正常分布的结果。对所有非 Marketplace 商品 URL 调用 `browser.open`，滤掉不是有货商品页面的结果。`browser.open` 无法抓取 Meta 第一方链接（instagram.com、facebook.com、threads.com/threads.net 及其他 Meta 所属主机对其禁用）。这类链接请改用平台原生工具。然后按对用户的有用程度（匹配约束、卖家知名度等）为其余结果排序。优先选择各零售商网站中面向用户所在国家/地区的版本。
5. If you don't find enough relevant search results, adjust your queries (and potentially your product search tools) and repeat steps 3-4.
   如果没有找到足够多的相关结果，调整查询（必要时也调整商品搜索工具）并重复步骤 3-4。
6. A request gets one shopping-results presentation, and it comes after every product search for that request has finished. Until then, do not create a `shopping_results` widget. Always mention products in your text responses while you wait for all product searches to finish if there are quality interim results (e.g. catalog search results from a well-known retailer), but before you name any of them, call `shopping.resolve_results` for the exact products you are about to name and write each one as its marker. A marker you already have stays good for the rest of the conversation. Once those products are resolved, mention 1-2 products using product markers as long as the update says it is early and says what is still running. Once the last product search finishes, make that presentation cover everything gathered for the request: pass every result file for it (catalog, Marketplace, and browser) in a single `result_paths` array, select and rank the best products across that whole pool, then call `widget.create` with the returned `path`. That is one widget, unless the request spans distinct product groups: those get one widget each in the same response, each resolving its own selection from that same pool (see Response Formatting). A group split is still one presentation, never a sequence of them over time. A later search adds candidates to the pool; it never replaces the searches that came before it, and the presentation must not be only about the search task that finished last. A search that fails or returns nothing usable has finished: present what the other sources returned rather than withholding the widget. Follow the Response Formatting section below.
   每个请求只有一次购物结果展示，且在该请求的所有商品搜索都结束之后进行。在此之前，不要创建 `shopping_results` 组件。在等待所有商品搜索完成期间，如果有质量的阶段性结果（如来自知名零售商的目录搜索结果），应始终在文本回复中提及商品；但在点名任何商品之前，先对即将点名的确切商品调用 `shopping.resolve_results`，并把每个商品写成其标记。已有的标记在对话其余部分持续有效。这些商品解析完成后，只要更新说明尚处早期、并说明还有哪些搜索在运行，就可以用商品标记提及 1-2 个商品。当最后一个商品搜索结束后，让展示覆盖为该请求收集的全部内容：把该请求的每个结果文件（目录、Marketplace 和浏览器）放进同一个 `result_paths` 数组传入，在整池中挑选并排序最佳商品，然后用返回的 `path` 调用 `widget.create`。这算一个组件，除非请求跨越多个不同的商品组：各组在同一响应中各得一个组件，每个组件都从同一池中解析各自的选择（见 Response Formatting）。按组拆分仍是一次展示，绝不是随时间推移的一系列展示。后续搜索只是向池中补充候选，绝不会取代先前的搜索，展示也不得只围绕最后完成的那个搜索任务。失败或没有可用结果的搜索同样算已结束：呈现其他来源返回的内容，而不是扣住组件不发。遵循下文 Response Formatting 一节。
7. If the user continues the original query in follow-up turns, maintain the constraints of the original request. Drop constraints only when the user explicitly instructs you to do so or pivots to a new search query, in which case drop all previous constraints that aren't generally applicable or based on high-level user preference. On a pivot, close every active browser search for the old request with `browser.close_task` before starting the new search. Use `browser.list_tasks` if you need the task IDs. Ignore late results from the old request.
   如果用户在后续回合继续原始查询，保持原请求的约束。只有当用户明确指示放弃，或转向新的搜索查询时才放弃约束；转向时应放弃所有既不普遍适用、也非基于高层用户偏好的先前约束。转向时，先用 `browser.close_task` 关闭旧请求的每个仍在运行的浏览器搜索，再开始新搜索；需要任务 ID 时用 `browser.list_tasks`。忽略旧请求迟到的结果。

Note: Always resolve and then mention products in your text response using markers in the turn that you receive the search results. Don't wait for all the search results to be returned before doing this.

注意：收到搜索结果的当回合，就要解析商品并在文本回复中用标记提及它们。不要等所有搜索结果都返回后才做。

Example: "help me shop for a black sweater"

示例："help me shop for a black sweater"（帮我买一件黑色毛衣）

### Finding deals / 寻找优惠

1. Identify the typical/baseline price for the product.
   确定该商品的典型/基准价格。
2. Research where deals might be found for this specific product or product category e.g. newegg.com often has sales for electronics
   调研该商品或该品类可能出现优惠的渠道，如 newegg.com 经常有电子产品促销
3. Use `browser.spawn_task` to perform a thorough and detailed search for any product listings that have a lower price than the baseline.
   用 `browser.spawn_task` 彻底细致地搜索价格低于基准的商品条目。
4. Use `browser.spawn_task` to look for any coupon or promo codes for relevant products.
   用 `browser.spawn_task` 查找相关商品的优惠券或促销码。
5. Present the relevant search results to the user. Be succinct and aim to provide a recommendation or takeaway. If you didn't find any current deals, say so and share any knowledge you gained about past and/or future deals.
   向用户呈现相关搜索结果。要简洁，并给出推荐或结论。如果没有找到当前优惠，如实说明，并分享你获得的有关过去和/或将来优惠的信息。

Example: "find me a deal on a new microwave"

示例："find me a deal on a new microwave"（帮我找新微波炉的优惠）

### Reverse image search / 反向图片搜索

1. If the image contains multiple plausible products and the user does not specify which one to shop, ask which item to search for before running a product search.
   如果图片中有多个可能的商品且用户未指定要买哪一个，先询问要搜索哪一件，再运行商品搜索。
2. If the current user message attaches an image and asks to shop a clear target, and every required attribute is already resolved, run direct catalog image search before any other step.
   如果当前用户消息附有图片且要求购买明确目标，且所有必要属性均已解析，则在任何其他步骤之前先运行直接目录图片搜索。
3. If direct image search is unavailable or no uploaded image path is available, derive a detailed visual description.
   如果无法进行直接图片搜索，或没有可用的已上传图片路径，则推导一段详细的视觉描述。
4. Execute searches with the relevant product search tools, using image search inputs when available and descriptive queries otherwise.
   用相关商品搜索工具执行搜索：可用时使用图片搜索输入，否则使用描述性查询。
5. Verify product pages / images against the original image; rank exact matches ahead of visually similar results.
   对照原始图片核验商品页面/图片；把完全匹配排在视觉相似结果之前。
6. Continue searching and adjusting your queries as necessary in order to identify the product. Stop after reasonable query refinements and return similar matches if necessary.
   为识别商品持续搜索并按需调整查询。经过合理的查询细化后即停止；必要时返回相似匹配。
7. Present the relevant search results to the user. Be honest with the user if you're unable to find an exact match.
   向用户呈现相关搜索结果。如果找不到完全匹配，如实相告。

### Purchase / 购买

Run the purchase workflow on the main agent. Do not hand any part of it to `subagent.spawn`: a subagent has neither wallet nor browser access, so it cannot complete a purchase.

购买工作流在主代理上运行。不要把其中任何部分交给 `subagent.spawn`：子代理既没有钱包权限也没有浏览器权限，无法完成购买。

1. Identify the products and any variants or quantities already specified.
   确认商品及已指定的变体或数量。
2. Close active browser discovery tasks that are no longer relevant with `browser.close_task`. Use `browser.list_tasks` to find their IDs. Ignore late results for those products after purchase begins.
   用 `browser.close_task` 关闭不再相关的进行中浏览器发现任务；需要 ID 时用 `browser.list_tasks`。购买开始后，忽略这些商品迟到的结果。
3. Route on the eligibility flags `meta-catalog-search` already returned; do not call `shopping product-details` just to decide the route.
   依据 `meta-catalog-search` 已返回的资格标志选择路径；不要只为了决定路径而调用 `shopping product-details`。
4. For Meta catalog products with `is_agentic_checkout_creation_enabled: true`, call `shopping product-details` once per selected product to pin the exact variant. Then load `/opt/hatch/skills/shopping/references/shopify-ucp.md` and follow its purchase flow.
   对于 `is_agentic_checkout_creation_enabled: true` 的 Meta 目录商品，对每个选定商品调用一次 `shopping product-details` 以锁定确切变体。然后加载 `/opt/hatch/skills/shopping/references/shopify-ucp.md` 并遵循其购买流程。
5. Otherwise, load `/opt/hatch/skills/shopping/references/browser-checkout.md` and follow its handoff flow.
   否则，加载 `/opt/hatch/skills/shopping/references/browser-checkout.md` 并遵循其交接流程。

### Cart-building / 购物车构建

The user is gathering items to buy later, not buying now. A cart is the
merchant's own basket, not a list you keep: the store holds it, prices it, and
applies its discounts and availability, so what the user sees is what they would
pay. Remembering products yourself gets you none of that.

用户是在收集以后要买的商品，而不是现在购买。购物车是商家自己的篮子，不是你自己维护的清单：它由商店保存、计价，并应用商店的折扣与库存状态，因此用户看到的就是他们将要支付的。自己记住商品清单得不到这些。

1. For Meta catalog products with `is_agentic_checkout_creation_enabled: true`, load `/opt/hatch/skills/shopping/references/shopify-ucp.md` and follow its Cart section. Products without that capability have no cart, and neither does browser checkout; say so rather than improvising one.
   对于 `is_agentic_checkout_creation_enabled: true` 的 Meta 目录商品，加载 `/opt/hatch/skills/shopping/references/shopify-ucp.md` 并遵循其 Cart（购物车）一节。不具备该能力的商品没有购物车，浏览器结账也没有；应如实说明，而不是即兴造一个。
2. Keep the `cart_id` for the rest of the conversation. It is internal state. Do not show it to the user.
   在对话其余部分保留 `cart_id`。它属于内部状态，不要展示给用户。
3. Follow the Purchase workflow once the user is ready to check out their cart.
   用户准备好为购物车结账时，执行 Purchase（购买）工作流。

### Shopping Instagram links / 处理 Instagram 购物链接

1. Use `instagram-cli post` and `instagram-cli media-understanding` to fetch the shopping context for the provided Instagram link.
   用 `instagram-cli post` 和 `instagram-cli media-understanding` 获取所给 Instagram 链接的购物上下文。
2. If the shopping context contains multiple products, ask the user which product they want to focus on.
   如果购物上下文包含多个商品，询问用户想聚焦哪一个。
3. If the shopping context contains product IDs, use `shopping product-details --product-id <product id>` to fetch the corresponding product details. Write a temporary `{"products": [...]}` JSON file whose entries copy `product.id`, `product.name`, `product.price`, `product.url`, and `product.images[0].url` verbatim from the product-details response into `product_id`, `price`, `name`, `url`, and `image_url`. Copy no field the response does not contain, and never take a name, price, or URL from the Instagram post. Pass that file and the copied product IDs to `shopping.resolve_results`. Do not execute a product search if you already have the relevant product ID, that wastes the user's time.
   如果购物上下文包含商品 ID，用 `shopping product-details --product-id <product id>` 获取对应的商品详情。写一个临时的 `{"products": [...]}` JSON 文件，其条目把 product-details 响应中的 `product.id`、`product.name`、`product.price`、`product.url` 和 `product.images[0].url` 原样复制到 `product_id`、`price`、`name`、`url` 和 `image_url`。响应中没有的字段不要复制，也绝不要从 Instagram 帖子中取名称、价格或 URL。把该文件和复制的商品 ID 传给 `shopping.resolve_results`。如果已有相关商品 ID，就不要再执行商品搜索，那会浪费用户时间。
4. If the shopping context contains no product IDs, execute the product discovery workflow using the data provided in the shopping context.
   如果购物上下文不包含商品 ID，则使用购物上下文提供的数据执行商品发现工作流。
5. Surface the found products in the shopping results widget and mention them via product markers in the text response.
   在购物结果组件中呈现找到的商品，并在文本回复中用商品标记提及它们。

## Meta Catalog Search / Meta 目录搜索

### Product Shape / 商品结构

```json
{
  "rank": 1,
  "product_id": "Meta catalog product id",
  "url": "product page URL",
  "name": "product title",
  "brand": "brand or merchant name",
  "price": "$49.00",
  "sale_price": "$39.00",
  "description": "product description",
  "image_url": "direct image URL",
  "color": "available or selected color",
  "material": "material when present",
  "pattern": "pattern when present",
  "size": "available or selected size",
  "gender": "gender/audience when present",
  "category": "category when present",
  "rating": "rating and review count when present",
  "is_agentic_checkout_creation_enabled": true,
  "is_agentic_checkout_completion_enabled": true
}
```

### Search / 搜索

`--query` performs semantic text matching. Query terms influence relevance but
do not filter the result set, so they are not a substitute for corresponding
structured flags. For example, a query for a boy's product can return products
for other genders unless `--gender male` is also passed.

`--query` 执行语义文本匹配。查询词影响相关性，但不过滤结果集，因此不能替代对应的结构化标志。例如，查询男孩商品时，如果不同时传 `--gender male`，仍可能返回其他性别的商品。

A text call accepts up to eight distinct queries; use only as many as are
useful.

一次文本调用最多接受八个不同查询；只使用有用的数量。

```sh
CATALOG_RESULTS_JSON=$(mktemp "${TMPDIR:-/tmp}/meta-catalog-search.XXXXXX")
meta-catalog-search \
  --query "<product query>" \
  --retries 2 --out "$CATALOG_RESULTS_JSON"

# Preview the first 20 products
jq '.products[0:20] | map(del(.image_url))' "$CATALOG_RESULTS_JSON"

# Stream products
jq '.products[] | select((.sale_price // .price // "") | test("\\$[0-9]")) | del(.image_url)' "$CATALOG_RESULTS_JSON"

# Pick products under a budget
jq '
  [
    .products[]
    | select(.product_id != null and .url != null and .image_url != null)
    | select((.sale_price // .price // "") | test("^\\$[0-9]"))
    | select(((.sale_price // .price) | gsub("[^0-9.]"; "") | tonumber) <= 100)
    | del(.image_url)
  ][0:50]
' "$CATALOG_RESULTS_JSON"
```

When the current user message attaches an image and asks to shop a clear target, start with a direct image search using the uploaded image path from the prompt context:

当当前用户消息附有图片且要求购买明确目标时，先用提示上下文中已上传的图片路径做直接图片搜索：

```sh
CATALOG_RESULTS_JSON=$(mktemp "${TMPDIR:-/tmp}/meta-catalog-search.XXXXXX")
meta-catalog-search --image-path <uploaded_file_path> --retries 2 \
  --out "$CATALOG_RESULTS_JSON"
```

Do not use `--visual-query` instead of `--image-path` on the attaching turn. Use text or `--visual-query` only to complement direct image search, or when no uploaded image path is available.

在附图的当回合，不要用 `--visual-query` 代替 `--image-path`。文本或 `--visual-query` 只用于补充直接图片搜索，或在没有已上传图片路径时使用。

`product_id` is what `shopping.resolve_results` takes in `selected_ids`, so keep it in every projection you make of these results. A narrowed preview that selects only display fields (`{name, brand, price, url}`) leaves you unable to cite anything you then talk about, and re-reading the file later costs an extra turn. When you narrow, keep `product_id` alongside whatever else you need:

`product_id` 是 `shopping.resolve_results` 在 `selected_ids` 中接受的值，因此对结果做任何投影都要保留它。只选展示字段（`{name, brand, price, url}`）的窄化预览会让你无法引用之后谈到的任何商品，而稍后重读文件又要多花一个回合。做窄化时，把 `product_id` 与所需的其他字段一起保留：

```sh
jq -r '.products[] | [.product_id, .brand, .name, (.sale_price // .price), .size] | @tsv' "$CATALOG_RESULTS_JSON"
```

Constraint flags for the `meta-catalog-search` CLI:

`meta-catalog-search` CLI 的约束标志：

- `--category` for a hard category constraint
  `--category`：硬性类目约束
- `--gender` for a hard gender/audience constraint resolved under Required attributes. Use exactly `male`, `female`, or `unisex`, preserve it on refinements, and do not infer it from product type or styling. Use `--prefer-gender` instead for a soft preference.
  `--gender`：硬性性别/受众约束，按"必要属性"一节解析。必须精确使用 `male`、`female` 或 `unisex`，在细化搜索中保持不变，且不得从商品类型或风格推断。软性偏好请改用 `--prefer-gender`。
- `--brand` ensures available products from the specified brand are returned and ranked first, with other brands available as backfill. Always specify `--brand` when the user requests results from a specific brand.
  `--brand`：确保返回指定品牌的在售商品并排名靠前，其他品牌作为补充。用户要求特定品牌的结果时，务必指定 `--brand`。
- `--domain` ensures available products from the specified seller domain are returned and ranked first, with products from other sellers available as backfill. Always specify `--domain` when the user requests results from a specific seller; pass its canonical hostname without a scheme or path.
  `--domain`：确保返回指定卖家域名的在售商品并排名靠前，其他卖家的商品作为补充。用户要求特定卖家的结果时，务必指定 `--domain`；传入其规范主机名，不带协议或路径。
- `--boost-brand-seller-website true|false` defaults to `true` and boosts the official seller website associated with each value passed to `--brand`
  `--boost-brand-seller-website true|false`：默认为 `true`，会提升传给 `--brand` 的各值对应的官方卖家网站的排名
- `--seller-type direct|secondhand` adds a seller-type ranking preference; the default is `direct`
  `--seller-type direct|secondhand`：添加卖家类型排序偏好；默认为 `direct`
- `--currency` with `--min-price` / `--max-price` (values in cents) for budget limits
  `--currency` 配合 `--min-price` / `--max-price`（数值单位为分）设定预算上下限
- `--color`, `--material`, `--style`, `--prefer-brand`, `--prefer-gender` for soft preferences
  `--color`、`--material`、`--style`、`--prefer-brand`、`--prefer-gender`：软性偏好

For requests for used, pre-owned, secondhand, thrifted, vintage, or
refurbished inventory, set `--seller-type secondhand` and  
`--boost-brand-seller-website false`.

对二手、旧物、secondhand、古着、vintage 或翻新库存的请求，设置 `--seller-type secondhand` 和 `--boost-brand-seller-website false`。

Apply every requirement in the request and context to every Meta Catalog Search
call, in each query and every applicable structured parameter. Required
attributes, stated price limits, and parameters that express a user requirement
are never relaxed. If results are insufficient, a later call may relax only
preference parameters that do not express a user requirement. Keep deliberately
relaxed results separate and do not present them as exact matches. If the CLI
rejects an argument you passed, correct that argument and rerun once with the
same constraints. If it fails for any other reason, or the results file cannot
be parsed, use the other search results rather than issuing diagnostic catalog
calls or silently dropping constraints.

请求与上下文中的每一项要求都要应用到每一次 Meta Catalog Search 调用中，体现在每个查询和每个适用的结构化参数上。必要属性、明确的价格上限以及表达用户要求的参数绝不可放宽。如果结果不足，后续调用只可放宽不表达用户要求的偏好参数。有意放宽后得到的结果要单独保存，不得当作精确匹配呈现。如果 CLI 拒绝了你传入的某个参数，改正该参数并以相同约束重跑一次。如果因其他原因失败，或结果文件无法解析，就使用其他搜索结果，而不是发起诊断性目录调用或悄悄丢弃约束。

### Product details / 商品详情

During product selection, before purchase, or when the user asks, it can be beneficial to determine the variants available for a given product, such as different clothing sizes.

在商品选择过程中、购买之前，或用户询问时，确定某商品的可用变体（如不同的服装尺码）可能有帮助。

For a specific Meta catalog product returned by `meta-catalog-search`, retrieve its corresponding product variant data from the catalog with the following command:

对于 `meta-catalog-search` 返回的特定 Meta 目录商品，用以下命令从目录获取其对应的商品变体数据：

```sh
shopping product-details --product-id "<product_id>"
```

Keep the response's runtime-authored `hatch_telemetry_context` unchanged for
the selected product and pass it whole to the checkout route as described by
the route reference. Never invent, edit, or reuse it for another product.

对选定商品，保持响应中由运行时生成的 `hatch_telemetry_context` 原样不变，并按路径参考文档的说明把它整体传给结账路径。绝不要编造、编辑它，也不要把它复用到其他商品上。

To confirm the size of a selected product, find that size in the size variant group and inspect its `product_ids`. Keep every non-size selected attribute (such as color) constant. Call `shopping product-details --product-id` for candidate IDs as needed and choose only a response whose `product.selected_variant_info` confirms both the requested size and the original non-size attributes. Treat an option with `is_available: false` as unavailable; never choose an arbitrary candidate when the attributes cannot be confirmed. Use the confirmed response's `product.id` for checkout, not the product ID from the `variant_groups`. If you select a product ID from the `variant_groups`, call `shopping product-details` for that variant product ID to confirm availability and checkout eligibility. Do not expose product IDs or this lookup process to the user.

要确认选定商品的尺码，在尺码变体组中找到该尺码并检查其 `product_ids`。保持所有非尺码的已选属性（如颜色）不变。按需对候选 ID 调用 `shopping product-details --product-id`，只选择其 `product.selected_variant_info` 同时确认请求尺码和原有非尺码属性的响应。`is_available: false` 的选项视为无货；无法确认属性时，绝不随意挑选候选。结账使用确认响应中的 `product.id`，而不是 `variant_groups` 里的商品 ID。如果你从 `variant_groups` 中选择了某个商品 ID，对该变体商品 ID 调用 `shopping product-details` 以确认可用性与结账资格。不要向用户暴露商品 ID 或这一查找过程。

### Widget / 组件

The `shopping_results` widget can be used to surface Meta catalog products in a detailed list-view component. Create it only once every product search for the request has finished, never as an early look at whichever source returned first. It's imperative to rank/order the products such that the top 5 are the highest quality (matching user constraints, from well-known merchants/websites).

`shopping_results` 组件可用于在详细的列表视图组件中呈现 Meta 目录商品。只有该请求的每个商品搜索都结束后才创建它，绝不要作为"哪个来源先返回就抢先看哪个"的展示。必须对商品进行排序，使前 5 个质量最高（匹配用户约束、来自知名商家/网站）。

```json
{
  "result_paths": ["<CATALOG_RESULTS_JSON>"],
  "selected_ids": ["product-id-1", "product-id-2"]
}
```

Call `widget.create` with the `path` returned by `shopping.resolve_results`:

用 `shopping.resolve_results` 返回的 `path` 调用 `widget.create`：

```json
{
  "kind": "shopping_results",
  "present_now": true,
  "data": {
    "path": "<path returned by shopping.resolve_results>"
  }
}
```

## Browser Product Search / 浏览器商品搜索

### Search / 搜索

Call `browser.spawn_task` with a complete search brief:

用完整的搜索简报调用 `browser.spawn_task`：

```json
{
  "task": "<what the user asked to find, in their words>. Find real purchasable products online for: <user request>. Preserve these requirements: <constraints>. Unless instructed otherwise, default to searching a maximum of 3 merchant sites and a maximum of 10 products in total - prefer to limit searching to the minimum amount necessary to provide a high-quality and seller-diverse response (e.g. if searching 2 merchant sites gives you enough high quality products, stop there). Return a concise shortlist with product name, merchant, price, availability, product URL, and why each product matches. Also populate browser_hand_off.product_results with every shortlisted product. Ensure that each URL is a dedicated product page, not a search page. For image_url, identify the primary rendered product image; inspect its src, srcset, lazy-load attributes, or element HTML, resolve relative URLs against the product-page URL, and include only a direct absolute HTTPS image URL. Omit products whose primary image cannot be verified or resolves to a blob/data URL, placeholder, logo, or tracking pixel. Use price for the regular/list price, or the current price when there is no sale. Include sale_price only when the page explicitly shows an actual sale with a distinct discounted price. Set availability to in_stock only when the product page indicates it can be added to cart or purchased. This is product discovery only: do not purchase, enter checkout, add items to cart, or request payment/shipping details. If the user asked to buy, order, purchase, or check out, ask them to choose or confirm one product before any separate checkout handoff."
}
```

Ensure the search brief is complete and self-contained: the browser task cannot see this conversation and receives no information about the user or their request beyond what you put in `task`.

确保搜索简报完整且自包含：浏览器任务看不到这段对话，除了你写入 `task` 的内容之外，它不会收到关于用户及其请求的任何信息。

Parallel browser tasks are for distinct discovery angles that improve coverage; follow-up refinements stay within the browser task already pursuing that angle.

并行浏览器任务用于改善覆盖面的不同发现角度；后续细化应留在已在追踪该角度的浏览器任务内部完成。

Never surface product details (price, availability, product page URLs) from `browser.search` results to the user. They are unreliable. Always use `browser.spawn_task` for product search and fetching product details.

绝不要把 `browser.search` 结果中的商品详情（价格、库存、商品页 URL）呈现给用户。它们不可靠。商品搜索和获取商品详情一律使用 `browser.spawn_task`。

【评论】普通 `browser.search` 摘要结果被视为不可靠，须经 `browser.spawn_task` 的代理式浏览器实际打开商品页核验后才能呈现，形成两级信息可靠性分层。

Call `browser.open` on all product URLs returned from the browser product search and filter out any products where the URL isn't a dedicated product page with in-stock availability.

对浏览器商品搜索返回的所有商品 URL 调用 `browser.open`，滤掉 URL 不是有货的专用商品页面的商品。

### Product shape / 商品结构

Browser product searches finish asynchronously. When the completion arrives, use the structured
`completion_result.product_results` object rather than reconstructing products from the prose
report. It has this shape:

浏览器商品搜索异步完成。收到完成通知时，使用结构化的 `completion_result.product_results` 对象，而不是从文字报告中重建商品。其结构如下：

```json
{
  "version": 1,
  "kind": "browser_product_search_results",
  "count": 2,
  "products": [
    {
      "result_id": "browser:<stable URL hash>",
      "name": "Product name",
      "url": "https://merchant.example/product",
      "image_url": "https://merchant.example/product.jpg",
      "availability": "in_stock",
      "brand": "Merchant or brand",
      "price": "$49.00"
    }
  ]
}
```

Do not copy fields out of the prose report or invent a missing image URL.

不要从文字报告中复制字段，也不要编造缺失的图片 URL。

### Widget / 组件

The `shopping_results` widget can surface verified browser products using the generic catalog card
layout. Browser cards deliberately use `type: "browser"`, omit `product_id`, and disable agentic
checkout.

`shopping_results` 组件可用通用目录卡片布局呈现经验证的浏览器商品。浏览器卡片有意使用 `type: "browser"`、省略 `product_id` 并禁用代理结账。

Write `completion_result.product_results` verbatim to a temporary JSON file. Select and rank
products by `result_id`, then resolve them into the presentation payload:

把 `completion_result.product_results` 原样写入一个临时 JSON 文件。按 `result_id` 挑选并排序商品，然后把它们解析为展示载荷：

```json
{
  "result_paths": ["<browser completion_result.product_results JSON path>"],
  "selected_ids": [
    "browser:stable-url-hash-1",
    "browser:stable-url-hash-2"
  ]
}
```

Pass catalog or Marketplace result files in the same `result_paths` array when combining sources. Then call `widget.create` with the following payload:

合并来源时，把目录或 Marketplace 结果文件放进同一个 `result_paths` 数组传入。然后用以下载荷调用 `widget.create`：

```json
{
  "kind": "shopping_results",
  "present_now": true,
  "data": {
    "path": "<path returned by shopping.resolve_results>"
  }
}
```

If several browser tasks belong to one shopping request, aggregate their completed result files
by passing each file in one `shopping.resolve_results` call. The request's presentation comes
once every one of those tasks has finished, and carries the catalog and Marketplace files for the
same request alongside them. If not every task has completed, wait for automatic completion
delivery; do not poll. A single browser task may cover several retailers when aggregation would
otherwise be unnecessary overhead.

如果多个浏览器任务同属一个购物请求，在一次 `shopping.resolve_results` 调用中传入每个文件，以聚合它们已完成的结果。该请求的展示要等所有这些任务都结束后进行，并同时携带同一请求的目录与 Marketplace 文件。如果并非所有任务都已完成，等待自动送达的完成通知，不要轮询。当聚合只会带来不必要的开销时，单个浏览器任务可以覆盖多个零售商。

## Marketplace Search / Marketplace 搜索

### Printed listing shape / 打印输出的条目结构

```json
{
  "listing_id": "Marketplace listing id",
  "title": "listing title",
  "location": "city, state location of the listing",
  "price": "$49.00",
  "condition": "condition of the item, e.g., Used (good)",
  "description": "listing description",
  "product_url": "listing URL",
  "image_url": {"withheld": "signed URL; ..."}
}
```

### Search / 搜索

```sh
MARKETPLACE_RESULTS_JSON=$(mktemp "${TMPDIR:-/tmp}/facebook-marketplace-search.XXXXXX")

facebook-cli marketplace search \
  --query "<natural language item query>" \
  --limit <N> \
  --out "$MARKETPLACE_RESULTS_JSON"
```

With location or local pickup filters:

带位置或线下自提过滤条件：

```sh
facebook-cli marketplace search \
  --query "<item>" \
  --max-price <dollars> \
  --latitude <lat> \
  --longitude="<lng>" \
  --radius-in-miles <miles> \
  --delivery-method local_pickup_only \
  --limit <N> \
  --out "$MARKETPLACE_RESULTS_JSON"
```

The command prints each listing with `image_url` replaced by a `withheld` object. The full payload, including signed photo URLs, goes only to the `--out` file. Pick listings and their `listing_id` values from the printed output. Do not read, print, or `jq` the `--out` file; pass its path unchanged in `result_paths`. To show a listing's photo, select it in `shopping.resolve_results`: its card carries the photo. Select only listings whose printed output has an `image_url`: `shopping.resolve_results` fails the whole call when a selected listing has no photo.

该命令打印每条条目时，`image_url` 被替换为 `withheld` 对象。包含签名图片 URL 的完整载荷只写入 `--out` 文件。从打印输出中挑选条目及其 `listing_id`。不要读取、打印 `--out` 文件或对它运行 `jq`；在 `result_paths` 中原样传入其路径。要展示某条目的照片，在 `shopping.resolve_results` 中选中它即可：其卡片会携带照片。只选择打印输出中带 `image_url` 的条目：选中的条目没有照片时，`shopping.resolve_results` 会使整个调用失败。

### Listing details / 条目详情

Fetch full details and seller trust signals with `facebook-cli marketplace listing details --listing-id <listing_id>` and `facebook-cli marketplace seller-info --listing-id <listing_id>`. Search results carry only `seller_id` (no seller name), so `seller-info` is how you surface the seller's name, rating, and review count. See `references/backends.md` for the full flag set and pagination.

用 `facebook-cli marketplace listing details --listing-id <listing_id>` 和 `facebook-cli marketplace seller-info --listing-id <listing_id>` 获取完整详情和卖家信任信号。搜索结果只携带 `seller_id`（无卖家名称），因此要呈现卖家名称、评分和评价数，需使用 `seller-info`。完整的标志集合与分页说明见 `references/backends.md`。

### Pasted listing links / 粘贴的条目链接

When the message contains a Marketplace item or share link, do not open it in the browser: those pages sit behind a login wall. Use the facebook-cli to fetch the listing instead: `facebook-cli marketplace listing details --url '<pasted link>'` for item links (needs no linked account), or `facebook-cli link-sharing decode-url --url '<pasted link>'` first for share links (needs a linked account), then pass the decoded item link to `details --url`. Then search similars from the title and description and resolve them with `shopping.resolve_results`.

当消息中包含 Marketplace 商品或分享链接时，不要在浏览器中打开：这些页面位于登录墙之后。改用 facebook-cli 抓取条目：商品链接用 `facebook-cli marketplace listing details --url '<pasted link>'`（无需关联账号）；分享链接先用 `facebook-cli link-sharing decode-url --url '<pasted link>'` 解码（需要关联账号），再把解码后的商品链接传给 `details --url`。然后根据标题和描述搜索相似商品，并用 `shopping.resolve_results` 解析。

### Widget / 组件

The `shopping_results` widget can be used to surface Marketplace listings in a detailed list-view component. Create it only once every product search for the request has finished, never as an early look at whichever source returned first.

`shopping_results` 组件可用于在详细的列表视图组件中呈现 Marketplace 条目。只有该请求的每个商品搜索都结束后才创建它，绝不要作为"哪个来源先返回就抢先看哪个"的展示。

```json
{
  "result_paths": ["<MARKETPLACE_RESULTS_JSON>"],
  "selected_ids": ["listing-id-1", "listing-id-2"]
}
```

Then call `widget.create` with the `path` returned by `shopping.resolve_results`:

然后用 `shopping.resolve_results` 返回的 `path` 调用 `widget.create`：

```json
{
  "kind": "shopping_results",
  "present_now": true,
  "data": {
    "path": "<path returned by shopping.resolve_results>"
  }
}
```

## Constraints / 约束

Before surfacing any products to the user (either via widget or text), double-check that the products match all constraints for the user request and/or general preferences. E.g. for clothing requests, are all the products of the right gender & size for the user request

在向用户呈现任何商品之前（无论通过组件还是文本），再次核对商品是否匹配用户请求和/或通用偏好的全部约束。例如服装请求：所有商品是否符合用户请求的性别与尺码

## Response Formatting / 响应格式

- Prefer presenting products in widgets rather than text when the product search tool supports them, in the one presentation that comes after every search for the request has finished. Any response before that, or any request whose searches support no widget, keeps to the takeaway and at most one or two recommendations.
  当商品搜索工具支持组件时，优先用组件而非文本呈现商品，且只在该请求所有搜索结束后的那一次展示中呈现。在此之前的一切响应，以及搜索不支持组件的请求，都只保留结论和至多一两条推荐。
- Lead with the recommendation or takeaway. Keep the rest of the response succinct and easily scannable. Briefly compare the decisive tradeoff between the top choices when useful.
  以推荐或结论开头。回复其余部分保持简洁、易于扫读。有用时，简要比较首选之间的决定性权衡。
- Say plainly when an update is early. Any response that names a product before every search is in opens by saying it is early and what is still running ("here are a few initial picks, still running a more thorough search"), and stays provisional throughout: no superlatives, no ranking language, no category verdict, nothing shaped like "the best X is Y". You have not seen the whole pool yet, so you cannot know. A recommendation is earned only once every search has returned and you have reviewed all of it together.
  更新尚处早期时要如实说明。在所有搜索结束之前点名商品的任何响应，都要在开头说明尚处早期、还有哪些搜索在运行（"这里有几个初步候选，仍在进行更彻底的搜索"），并且全程保持暂定性：不用最高级、不用排序性语言、不对品类下结论、不说任何形如"最好的 X 是 Y"的话。你还没有看到全部候选池，所以无从判断。只有当所有搜索都已返回且你整体审阅过之后，推荐才算成立。
- When presenting products in widgets, pay attention to their order. Rank them from most relevant to least relevant, placing products that match the constraints from well-known sellers first.
  在组件中呈现商品时注意顺序。按相关度从高到低排序，把匹配约束且来自知名卖家的商品排在前面。
- If there is a natural grouping of products in the response (e.g. user asked for shoes & pants), surface those different groups in separate widgets, side by side in that one presentation rather than spread across turns.
  如果响应中的商品存在自然的分组（如用户同时要鞋和裤子），把这些不同组放进不同组件，在同一展示中并排呈现，而不是分散到多个回合。
- For every product included in `product_citations`, use only the exact complete `marker` value returned by the successful `shopping.resolve_results` call. Put the marker where the product name belongs. Do not construct it from `citation_id`, copy a product name next to it, or invent names, URLs, links, or citation IDs. If a product has no returned marker, only reference it by name.
  对包含在 `product_citations` 中的每个商品，只使用成功的 `shopping.resolve_results` 调用返回的确切完整 `marker` 值。把标记放在商品名称应在的位置。不要用 `citation_id` 拼出标记，不要在其旁复制商品名，也不要编造名称、URL、链接或引用 ID。没有返回标记的商品，只能按名称提及。
- If you know that variant (size, color, etc.) data exists for a product named in the text response, offer to surface it if the user is interested, e.g. "Would you like to know what other colors that sweater comes in?" Do not name additional products while making the offer.
  如果你知道文本回复中提到的商品存在变体（尺码、颜色等）数据，可在用户感兴趣时提议展示，如"想知道那件毛衣还有哪些颜色吗？"提议时不要再点名其他商品。
- Never surface results in a Markdown file (no link, no preview, no attachment) to the user unless explicitly asked to do so.
  除非用户明确要求，绝不要以 Markdown 文件（无链接、无预览、无附件）的形式向用户呈现结果。

### When this turn mentions a product without presenting a widget / 当本回合提及商品但未呈现组件时

This is most turns in a shopping conversation: the early update while a search runs, every follow-up question about something already on screen, every comparison, every later turn that circles back to a product. The marker rule is the same here as it is in the presentation, and this is where it is easiest to lose.

购物对话中的大多数回合都属于这种情况：搜索运行中的早期更新、针对已上屏商品的每个后续提问、每次比较、以及每个重新谈到某商品的后续回合。标记规则在这里与展示时相同，而这里最容易失守。

- Write each product as its marker, exactly as `shopping.resolve_results` returned it. This holds no matter how far back the product was resolved.
  每个商品都写成其标记，与 `shopping.resolve_results` 返回的完全一致。无论该商品是很早之前解析的，这一点都成立。
- Answering a question about one product still names that product, so put its marker where the name would have gone: confirming that a bag fits three lenses reads as "yes, `<marker>` fits three lenses", not "yes, the Lowepro fits three lenses".
  回答关于某商品的问题仍然是点名该商品，因此把标记放在名称应在的位置：确认某包能装三个镜头，应写成"是的，`<marker>` 能装三个镜头"，而不是"是的，Lowepro 那款能装三个镜头"。
- Narrowing the products already on screen down to one or two names each of the ones you keep, so each of those takes its marker. Do not substitute a distinguishing attribute ("the merino one", "the $199 one", "the Brooks pair") because it is shorter.
  把已上屏的商品收窄到一两个时，要逐一点名保留的商品，因此每个都要用其标记。不要因为更短就改用区分性属性（"美利奴那件""199 美元那双""Brooks 那双"）。
- A product from a search result file that no successful resolve returned has no marker yet. Resolve it before naming it. If it genuinely cannot be resolved, name it plainly and do not invent a marker, a URL, or a link for it.
  来自搜索结果文件、但从未被成功解析返回的商品还没有标记。点名之前先解析。如果确实无法解析，就平实地按名称提及，不要为它编造标记、URL 或链接。

<!-- shopping-results-response-contract:start -->  
### When this turn presents a shopping-results widget / 当本回合呈现购物结果组件时

These rules apply only to user-facing prose accompanying a shopping-results presentation. Begin directly with the takeaway. Do not mention or acknowledge the skill, instructions, tools, widgets, or formatting rules.

这些规则只适用于伴随购物结果展示的用户可见文字。直接以结论开头。不要提及或承认技能、指令、工具、组件或格式规则的存在。

- Do not mention the shopping-results presentation or narrate its contents.
  不要提及购物结果展示，也不要复述其内容。
- Never surface the results in a Markdown file unless explicitly requested.
  除非明确要求，绝不要以 Markdown 文件形式呈现结果。
- Do not recap, enumerate, or describe the products already presented.
  不要复述、列举或描述已呈现的商品。
- Give one short takeaway. Name at most two products, and only when they are needed for the decisive recommendation or tradeoff. Treat those names as the complete allowlist and refer to all other presented products collectively.
  给出一条简短结论。最多点名两个商品，且只在决定性推荐或权衡需要时点名。把这几个名称视为完整的白名单，其余已呈现商品一律概括提及。
- Base the takeaway and any recommendations on all available search results, not only the search task that finished last.
  结论与推荐要基于所有可用的搜索结果，而不只是最后完成的那个搜索任务。
- When multiple products are named, preserve their relative order from the corresponding widget.
  点名多个商品时，保持它们在对应组件中的相对顺序。
- For every named product included in `product_citations`, use only the exact complete `marker` value returned by the successful `shopping.resolve_results` call. Put the marker where the product name belongs. Do not construct it from `citation_id`, copy a product name next to it, or invent names, URLs, links, or citation IDs. If a named product has no returned marker, reference it by name only.
  对点名的、包含在 `product_citations` 中的每个商品，只使用成功的 `shopping.resolve_results` 调用返回的确切完整 `marker` 值。把标记放在商品名称应在的位置。不要用 `citation_id` 拼出标记，不要在其旁复制商品名，也不要编造名称、URL、链接或引用 ID。没有返回标记的点名的商品，只按名称提及。

<!-- shopping-results-response-contract:end -->

## Auth / 认证
- No user-provided token is required.
  不需要用户提供令牌。
- Meta catalog search relies on installed runtime tools and environment-backed access.
  Meta 目录搜索依赖已安装的运行时工具和由环境提供的访问。
- Meta catalog requests use the Meta Catalog connector's read permission.
  Meta 目录请求使用 Meta Catalog 连接器的读取权限。
