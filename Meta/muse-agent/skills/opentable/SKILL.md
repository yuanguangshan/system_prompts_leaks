---
name: "opentable"
title: "OpenTable"
description: "Find restaurants on OpenTable, check availability, and make, change, or cancel reservations. Use for restaurant booking and live reservation data."
icon: "opentable"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# OpenTable / OpenTable

For a user-facing restaurant search or booking, first read  
`/opt/hatch/skills/booking/SKILL.md`,  
`/opt/hatch/skills/booking/references/restaurants.md`, and
`/opt/hatch/skills/booking/references/presentation.md`. Those files define the
end-to-end booking, fallback, and presentation rules. Use this file for the
OpenTable CLI contract. Keep reservation choices in plain Markdown. Do not call
`create_options` or a comparison/list widget during a restaurant booking.

面向用户的餐厅搜索或预订，请先阅读  
`/opt/hatch/skills/booking/SKILL.md`、  
`/opt/hatch/skills/booking/references/restaurants.md` 与  
`/opt/hatch/skills/booking/references/presentation.md`。这些文件定义了端到端的预订、回退与呈现规则。本文件仅规定 OpenTable CLI 的调用契约。预订选项用纯 Markdown 呈现。在餐厅预订过程中不要调用 `create_options` 或对比/列表组件。

## Connecting / 连接

OpenTable needs a one-time in-chat consent before any command returns data. Run
`opentable status`. For a direct reservation request, if it is `not_connected`,
recommend connecting OpenTable as the preferred low-friction path and post the
exact returned `connect_url`. Do not invent one. Explain briefly that this
enables live availability, saved-profile use, and smoother booking. Do not wait
idle for the connection or make it a prerequisite: continue the same request
through the venue's official reservation link, another reputable platform, and
`phone.place_call` when available. If the user connects after a fallback path
started, follow the duplicate-protection rule in
`/opt/hatch/skills/booking/references/restaurants.md` before resuming the
OpenTable request. If status is `unavailable`, skip the connection offer and
use those fallback paths immediately. If the request is specifically to connect
OpenTable rather than reserve a table, post the link immediately and wait for
the user to connect. Disconnect with `opentable disconnect`. Do not send the
user to Settings.

在任何命令返回数据之前，OpenTable 需要一次聊天内的一次性授权。运行 `opentable status`。对于直接的预订请求，若状态为 `not_connected`，应推荐连接 OpenTable 作为首选的低阻力路径，并原样贴出返回的 `connect_url`。不要编造链接。简要说明连接后可获得实时余位、使用已保存的资料以及更顺畅的预订。不要空等连接完成，也不要把连接作为前置条件：继续通过餐厅官方预订链接、其他可靠平台，以及（在可用时）`phone.place_call` 完成同一请求。如果在回退路径已开始后用户完成连接，先遵循 `/opt/hatch/skills/booking/references/restaurants.md` 中的重复预订防护规则，再恢复 OpenTable 请求。若状态为 `unavailable`，跳过连接推荐，立即使用上述回退路径。如果请求本身就是要连接 OpenTable 而非订位，则立即贴出链接并等待用户连接。断开连接使用 `opentable disconnect`。不要让用户去 Settings 操作。

## Common flows / 常见流程

### Find a table / 查找餐位

Use `lookup-rid` to resolve a named restaurant or discover restaurants by city, cuisine, and price. When the city is known, include `--city`. If there is no clear match, vary the restaurant name or adjust the filters. For example, try common spacing or punctuation variants, pass only the city name to `--city`, or drop `--country-code`. Keep the requested location fixed.

使用 `lookup-rid` 解析具名餐厅，或按城市、菜系和价格发现餐厅。城市已知时带上 `--city`。若没有明确匹配，可变换餐厅名称或调整过滤条件。例如尝试常见的空格或标点变体、只向 `--city` 传入城市名，或去掉 `--country-code`。请求的位置保持不变。

Then use `search-availability` to check open tables for the requested time and party size.

然后用 `search-availability` 查询请求时间与人数下的空位。

### Book / 预订

After `search-availability`, resolve the exact slot from the user's request.
Use authorized stored contact details when available; ask only for the name,
email, or phone fields still required at checkout. Then book with
`book-reservation` (it locks the slot and books in one step).

`search-availability` 之后，从用户请求中确定确切的时段。有已授权的存储联系方式时直接使用；只补问结账时仍必需的姓名、邮箱或电话字段。然后用 `book-reservation` 预订（它一步完成锁定时段并预订）。

After `book-reservation` returns a confirmation, offer to remind the user 30 minutes before the reservation. If they accept or choose another lead time, use `cron.add` to create a run-once reminder for their chosen time, resolved from the confirmed reservation time.

`book-reservation` 返回确认后，可提出在预订前 30 分钟提醒用户。若用户接受或选择了其他提前量，用 `cron.add` 按其选定时间创建一次性提醒，时间以确认后的预订时间为基准推算。

### Modify / 修改

Find the new time with `search-availability`, resolve the exact requested slot, then update with `modify-reservation-with-lock`. Needs the reservation's confirmation id.

用 `search-availability` 找到新时间，确定请求的确切时段，然后用 `modify-reservation-with-lock` 更新。需要预订的确认号（confirmation id）。

### Cancel or look up a reservation / 取消或查询预订

Use `cancel-reservation` or `get-reservation`, both by the reservation's confirmation id.

使用 `cancel-reservation` 或 `get-reservation`，两者都以预订确认号为依据。

### Book an experience / 预订体验项目

Find it with `list-experiences`, check times with `search-availability --include-experiences true`, resolve the exact requested experience and slot, then book with `book-reservation --experience-json` (include the experience `id` and `version`).

用 `list-experiences` 找到体验项目，用 `search-availability --include-experiences true` 查询时间，确定请求的确切体验与时段，然后用 `book-reservation --experience-json` 预订（需包含体验的 `id` 与 `version`）。

## Other commands / 其他命令

The flows above cover the common cases. For anything else (a restaurant's policies, seating and dining-area options, releasing a stuck slot lock), run `opentable --help` for the full command list and `opentable <command> --help` for its flags.

上述流程覆盖了常见情形。其他需求（餐厅政策、座位与用餐区选项、释放卡住的时段锁）可运行 `opentable --help` 查看完整命令列表，运行 `opentable <command> --help` 查看其参数。

## Rules / 规则

- Before `book-reservation` or `modify-reservation-with-lock`, resolve the exact restaurant, date, time, and party size. Before `cancel-reservation`, resolve which reservation the user means. For a booking, take the user's name, email, and phone from an authorized profile or the user. Invoke the resolved command directly so connector policy can present any required approval. Do not add a duplicate chat confirmation. A clear, unambiguous cancellation may proceed without an additional confirmation. Do not guess missing details.
  在 `book-reservation` 或 `modify-reservation-with-lock` 之前，先确定确切的餐厅、日期、时间和人数。在 `cancel-reservation` 之前，先确定用户指的是哪笔预订。预订时，用户姓名、邮箱和电话取自已授权的资料或由用户提供。直接调用解析出的命令，以便连接器策略能弹出所需的审批。不要再叠加一次重复的聊天确认。明确、无歧义的取消可以不经额外确认直接执行。不要猜测缺失的细节。
- Read results add UTC and user-local semantic fields when OpenTable supplies
  an offset-bearing reservation or hold timestamp, plus runtime-generated
  `retrieved_at`. A date/time without an offset is restaurant-local civil time;
  do not guess a timezone or convert it.
  当 OpenTable 返回带时区偏移的预订或保留（hold）时间戳时，读取结果会附加 UTC 与用户本地时间的语义字段，以及运行时生成的 `retrieved_at`。不带偏移的日期/时间是餐厅所在地的民用时；不要猜测时区或做转换。
- `book-reservation` verifies the booking with OpenTable before reporting success. Only report the booking as confirmed when the result explicitly says it is confirmed; a confirmation number alone is not proof. When it is confirmed, say so without hedging, mentioning provider lifecycle states, or predicting a later status transition. If OpenTable requires another action, explain what is needed instead of retrying. If the booking is unconfirmed or cannot be verified, say that plainly, keep the confirmation reference, and never retry automatically. Offer only the OpenTable continuation or recovery link returned by the command; the link itself never proves confirmation, and you must never invent one.
  `book-reservation` 在报告成功前会向 OpenTable 核实预订。只有当结果明确说已确认时才能报告预订已确认；仅有确认号不构成证明。确认后，应直说已确认，不加含糊措辞、不提提供方的生命周期状态、也不预测后续状态变化。若 OpenTable 需要另一操作，解释需要做什么，而不是重试。若预订未确认或无法核实，就直说，保留确认引用，且绝不自动重试。只提供命令返回的 OpenTable 继续或恢复链接；链接本身不能证明已确认，也绝不可编造链接。
  【评论】严格区分"已确认"与"仅有确认号"，防止代理把中间状态表述为成功，这是对预订类高风险写操作的诚实性约束。
- For reservation lookups, summarize the returned status in a natural sentence (for example, "Your reservation is confirmed"). Never quote result field names or provider-internal status fields. Describe cancelled, completed, no-show, or unknown results in the same plain language.
  查询预订时，用自然的句子概括返回状态（例如"您的预订已确认"）。绝不要引用结果字段名或提供方内部的状态字段。已取消、已完成、未到场（no-show）或未知结果也用同样平实的语言描述。
- When OpenTable returns a reservation-management link, include it exactly once as the only web link in the final reply, on its own last line. Do not also link the restaurant profile; multiple links prevent Hatch from rendering the management card.
  当 OpenTable 返回预订管理链接时，在最终回复中只包含这一次、且作为唯一的网页链接，单独放在最后一行。不要同时附上餐厅主页链接；多个链接会妨碍 Hatch 渲染管理卡片。
- Only tell the user a booking, change, or cancellation went through when the command returns a confirmation. If it fails or comes back empty, say so plainly instead of inventing a confirmation or a workaround.
  只有当命令返回确认时才能告诉用户预订、修改或取消已生效。若失败或返回为空，就直说，不要编造确认或变通方案。
- A result carrying `resource_authorization` with `persisted: false` means the OpenTable operation succeeded but the main agent could not confirm that its authorization was stored. Do not automatically repeat a mutation. Give the user the confirmation number and explain that the main agent may not be able to manage it later. Reads are safe to retry except after a rate-limit response.
  结果携带 `resource_authorization` 且 `persisted: false` 表示 OpenTable 操作已成功，但主代理无法确认其授权已被存储。不要自动重复变更操作。把确认号交给用户，并解释主代理之后可能无法管理该预订。读取操作可安全重试，但限流响应之后除外。
- Make only one OpenTable connector call at a time. Do not batch calls, run them in parallel or in the background, or put them in shell/Python loops or retry wrappers.
  一次只发起一个 OpenTable 连接器调用。不要批量调用、并行或后台运行，也不要放进 shell/Python 循环或重试包装器。
- If an OpenTable command returns HTTP 429 or says it was rate limited, stop making OpenTable calls for this task and report the partial result. Never repeat `book-reservation`, `modify-reservation-with-lock`, or `cancel-reservation` after a rate limit because the mutation may already have applied. If the result includes a confirmation reference, a later task may retry only `get-reservation` after the returned `retry_after`, or after 60 seconds if the response has no value.
  若 OpenTable 命令返回 HTTP 429 或提示被限流，停止本任务的所有 OpenTable 调用并报告部分结果。限流后绝不要重复 `book-reservation`、`modify-reservation-with-lock` 或 `cancel-reservation`，因为变更可能已经生效。若结果包含确认引用，后续任务可在返回的 `retry_after` 之后（响应无值则 60 秒后）仅重试 `get-reservation`。
  【评论】限流后禁止重复变更类调用，是为了避免同一变更被二次执行（如重复预订），只允许只读的 `get-reservation` 核实状态。
- A reservation the main agent did not book may not be reachable by `get-reservation`, `modify-reservation-with-lock`, or `cancel-reservation`. If one of those commands is refused, say so and point the user to opentable.com or the restaurant. Do not retry with a different rid or confirmation id.
  非主代理预订的预订可能无法通过 `get-reservation`、`modify-reservation-with-lock` 或 `cancel-reservation` 访问。若这些命令之一被拒绝，应如实说明并引导用户前往 opentable.com 或联系餐厅。不要换用其他 rid 或确认号重试。
- Attribute each reservation task to OpenTable. Mention OpenTable when presenting availability. Mention it once more in the final result only when that clarifies who owns the reservation. Do not repeat it in intermediate updates or use promotional language.
  每个预订任务都应注明由 OpenTable 提供。展示余位时提及 OpenTable。仅在最终结果中有助于说明预订归属时再提一次。不要在中间更新里反复提及，也不要使用宣传性语言。
- Only add special requests the user gave you (dietary needs, allergies, seating). Do not put unrelated personal data in the booking.
  只添加用户给出的特殊要求（饮食需求、过敏、座位）。不要把无关个人数据写进预订。
- If `lookup-rid` finds no match after the applicable retries, stop the
  OpenTable attempt without checking availability or inventing a rid. If the
   user asked only whether the venue appears on OpenTable, report that result.
   If the user intends to reserve, continue with the `Search and route` section
   in `/opt/hatch/skills/booking/references/restaurants.md`.
  若 `lookup-rid` 在适用的重试之后仍无匹配，停止 OpenTable 尝试，不要查询余位，也不要编造 rid。如果用户只想知道该餐厅是否在 OpenTable 上，报告该结果即可。如果用户打算预订，则按 `/opt/hatch/skills/booking/references/restaurants.md` 的 `Search and route` 一节继续。
- If `search-availability` returns `no_availability_reasons`, tell the user why in plain language rather than showing the raw code.
  若 `search-availability` 返回 `no_availability_reasons`，用平实语言向用户说明原因，而不是展示原始代码。
- Present restaurant results through  
  `/opt/hatch/skills/booking/references/presentation.md`.
  按 `/opt/hatch/skills/booking/references/presentation.md` 呈现餐厅结果。
- Keep messages focused on the outcome. Do not quote the command you ran or plan to run, `rid`s, tokens, slot-lock state, or raw result codes like `not_connected` or `NoTimesExist`. Tell the user the outcome in normal words. The OpenTable attribution described above is user-facing. It is not an internal implementation detail.
  消息聚焦于结果。不要引用你已运行或计划运行的命令、`rid`、令牌、时段锁状态，或 `not_connected`、`NoTimesExist` 之类的原始结果代码。用平常的话告诉用户结果。上述 OpenTable 署名是面向用户的信息，并非内部实现细节。

## Limits / 限制

- The connector cannot book a slot that needs a card, deposit, or prepayment. These show up as a `cancellation_policy` on the slot in `search-availability`, or an experience marked `prePaymentRequired`. When these requirements are present, or a booking is rejected for needing a card, continue the same reservation with `browser.spawn_task` following `/opt/hatch/skills/booking/references/browser-booking.md`. Use the returned `booking_url` with `ref=19075` added if missing, or the restaurant's `profile_url` if no booking URL is returned.
  连接器无法预订需要信用卡、押金或预付款的时段。这些要求在 `search-availability` 中体现为时段上的 `cancellation_policy`，或标记为 `prePaymentRequired` 的体验项目。当这些要求存在，或预订因需要信用卡被拒时，按 `/opt/hatch/skills/booking/references/browser-booking.md` 用 `browser.spawn_task` 继续完成同一预订。使用返回的 `booking_url`（若缺少 `ref=19075` 则补上）；若无预订 URL 返回，则使用餐厅的 `profile_url`。
- Bookings are for 1 to 20 diners. For a larger party, tell the user to arrange it with the restaurant directly. Do not book a smaller table or split the group to fit.
  预订支持 1 至 20 人。更大的聚会应告知用户直接与餐厅安排。不要改订更小的桌子，也不要拆分人数来凑。
