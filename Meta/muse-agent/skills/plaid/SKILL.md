---
name: "plaid"
title: "Finances (Plaid)"
description: "Use to connect Plaid and read linked financial accounts: metadata, balances, transactions, recurring transactions, liabilities, and investments."
icon: "plaid"
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Plaid / Plaid

Everything runs as `plaid <command>` and returns JSON for you to read, not to show the user. Add `--help` to any command to see its options.

一切都以 `plaid <command>` 形式运行，返回的 JSON 供你阅读，而不是展示给用户。任何命令加 `--help` 可查看其选项。

Reads cover every linked institution at once. To narrow to one bank, pass `--credential-id <id>`, taking the id from `body.institutions[]` in an earlier read. Some reads also take `--account-id <id>` (repeatable) to narrow to specific accounts; when more than one institution is linked, pair it with `--credential-id` so the account ids resolve to the right bank.

读取操作一次覆盖所有已关联的机构。要缩小到某一家银行，传入 `--credential-id <id>`，该 id 取自先前读取结果中的 `body.institutions[]`。部分读取还接受 `--account-id <id>`（可重复）以缩小到特定账户；当关联了多个机构时，须与 `--credential-id` 搭配使用，使账户 id 解析到正确的银行。

## Connecting / 连接

Plaid needs a one-time connect before any read returns data. Run `plaid status`. If it comes back not connected, post the exact `connect_url` it returns as `[Connect Plaid](<connect_url>)` and wait for the user to finish linking, then run `plaid status` again before reading. Don't invent a URL, send the user to Settings, or ask for bank credentials.

Plaid 需要先完成一次性连接，任何读取才会返回数据。运行 `plaid status`。若返回未连接，将其返回的 `connect_url` 原样以 `[Connect Plaid](<connect_url>)` 形式发布，等待用户完成关联，读取前再次运行 `plaid status`。不要编造 URL、不要让用户去设置页、也不要索要银行凭据。

If the user asks to add or link another bank or institution, run `plaid status` and post the exact `add_account_url` it returns as `[Add financial account](<add_account_url>)`. Never reuse `connect_url` for an additional institution. If `add_account_url` is absent, say that adding another institution is unavailable.

若用户要求添加或关联另一家银行或机构，运行 `plaid status` 并将其返回的 `add_account_url` 原样以 `[Add financial account](<add_account_url>)` 形式发布。绝不为额外机构复用 `connect_url`。若 `add_account_url` 不存在，说明添加其他机构的功能不可用。

To disconnect every institution linked through Plaid, run `plaid disconnect`. If it returns a `disconnect_url`, post it as `[Disconnect Plaid](<disconnect_url>)` and wait for the user to confirm through that link; never claim disconnection before they confirm. If it has no URL, say Plaid is already disconnected. A bank-specific unlink is not available from chat: if the user asks to remove only one institution, direct them to that institution's account row in Settings instead of running the whole-Plaid disconnect. Don't set up a background check, goal, or reminder to monitor disconnection.

要断开通过 Plaid 关联的所有机构，运行 `plaid disconnect`。若其返回 `disconnect_url`，以 `[Disconnect Plaid](<disconnect_url>)` 形式发布，并等待用户通过该链接确认；用户确认前绝不声称已断开。若没有 URL，说明 Plaid 已处于断开状态。聊天中无法针对单一银行解除关联：若用户要求只移除一个机构，引导其前往设置中该机构的账户行，而不是执行整个 Plaid 的断开操作。不要设置后台检查、目标或提醒来监视断开状态。

## Common flows / 常用流程

### Accounts and balances / 账户与余额

List the linked accounts with `plaid accounts` — names, types, masked numbers, and each account's balances (`balances` carries `available`/`current`/`limit`). Start here to see what's linked and to answer "how much is in my checking," "what's my total across accounts," or net-worth questions. There is no separate balances command; `plaid accounts` is the balance read. Balances are Plaid's last reported figures, not live.

用 `plaid accounts` 列出已关联账户——名称、类型、掩码号码以及每个账户的余额（`balances` 含 `available`/`current`/`limit`）。从这里开始查看已关联的内容，并回答"我的支票账户有多少钱""我所有账户总共多少"或净资产类问题。没有单独的余额命令；`plaid accounts` 就是余额读取。余额是 Plaid 最后一次上报的数字，并非实时。

### Spending and transactions / 支出与交易

Use `plaid transactions-get --start-date YYYY-MM-DD --end-date YYYY-MM-DD` as the default for
reading transaction history.
Dates are inclusive; posted transactions use posting dates. A successful read returns all available
transactions in the requested date range; no manual pagination is needed.

读取交易历史的默认方式是 `plaid transactions-get --start-date YYYY-MM-DD --end-date YYYY-MM-DD`。
日期为闭区间；已入账交易使用入账日期。成功的读取会返回请求日期范围内所有可用交易；无需手动分页。

Results are returned directly as JSON. If the JSON response is too large to return inline,
it is written to the file named by `output_file`. Transactions are in `body.transactions`;
match `body.accounts` using `_plaid_source.credential_id` and `account_id`. Failed reads can
contain partial results: when `ok` is false, use `body.institutions` to explain missing coverage.

结果直接以 JSON 返回。若 JSON 响应过大而无法内联返回，则写入 `output_file` 指定的文件。交易位于 `body.transactions`；
用 `_plaid_source.credential_id` 与 `account_id` 匹配 `body.accounts`。失败的读取可能包含部分结果：当 `ok` 为 false 时，用 `body.institutions` 解释缺失的覆盖范围。

Positive amounts are outflows; negative amounts are inflows, including refunds.

正数为流出；负数为流入，包括退款。

Merchant names vary. Before saying a charge isn't there, try alternate merchant names, also search by amount and date, and include pending rows.

商户名称可能不同。在断言某笔扣款不存在之前，先尝试其他商户名称，也按金额和日期搜索，并包含待入账（pending）记录。

### Incremental transaction state / 增量交易状态

Use `plaid transactions-sync` only for workflows that maintain a persistent transaction store
and cursor, such as a user-built spending dashboard with a database that updates daily. Apply
additions, modifications, and removals to the store and save the returned cursor. For scheduled
time-range reports without a persistent store, use `transactions-get`.

`plaid transactions-sync` 仅用于维护持久化交易存储和游标的工作流，例如用户自建、数据库每日更新的支出看板。将新增、修改和移除应用到存储并保存返回的游标。对无持久化存储的定期时间范围报告，使用 `transactions-get`。

Continue pages with `--cursor-map-json <body.next_cursors>`, or `--cursor <next_cursor>` for one
institution. Do not use `--days-requested` to select a date range; it does not filter returned
transactions. Use `transactions-get` with explicit dates to select a time range.

用 `--cursor-map-json <body.next_cursors>` 续翻分页，单一机构则用 `--cursor <next_cursor>`。不要用 `--days-requested` 来选择日期范围；它不会过滤返回的交易。选择时间范围应使用带明确日期的 `transactions-get`。

### Recurring bills and subscriptions / 周期性账单与订阅

`plaid transactions-recurring` returns recurring money in (`body.inflow_streams`, e.g. paychecks) and out (`body.outflow_streams`, e.g. subscriptions and regular bills). Use it for "what am I subscribed to" or "what are my monthly bills." `is_active` indicates whether Plaid considers the recurring payment pattern ongoing. Keep it and `last_date` when parsing streams, and exclude inactive streams from current-bill totals. `average_amount` can include one-off payments, so check recent charges with `transactions-get` before quoting a monthly cost.

`plaid transactions-recurring` 返回周期性流入（`body.inflow_streams`，如工资）与流出（`body.outflow_streams`，如订阅和固定账单）。用于回答"我订了哪些服务"或"我每月有哪些账单"。`is_active` 表示 Plaid 是否认为该周期性付款模式仍在持续。解析流时保留它和 `last_date`，并把非活跃流排除在当前账单合计之外。`average_amount` 可能包含一次性付款，因此在引用月度成本前，先用 `transactions-get` 核对近期扣款。

### Loans and credit / 贷款与信贷

`plaid liabilities` returns credit-card, student-loan, and mortgage details — balances, rates, minimum payments, and due dates — under `body.liabilities`. Use it for what's owed or when a payment is due. A card's `last_statement_balance` is the amount billed on `last_statement_issue_date`. Payments made before that date are already accounted for in the bill. Payments posted afterward reduce the amount still unpaid. If the statement balance is null, say the statement amount is unavailable. Only say a card costs interest if its transactions show interest charges or the user says so.

`plaid liabilities` 在 `body.liabilities` 下返回信用卡、学生贷款和房贷的详细信息——余额、利率、最低还款额和到期日。用于回答欠款多少或何时到期。信用卡的 `last_statement_balance` 是 `last_statement_issue_date` 当期账单的金额。在该日期之前完成的还款已计入账单。此后入账的还款会减少未付金额。若账单余额为 null，说明账单金额不可用。只有当交易记录显示有利息扣款或用户自己说明时，才可称该卡产生利息。

### Investments / 投资

Before reading holdings, glance at `plaid accounts` for an account of `type` `investment` (a brokerage/retirement account). If none is linked, skip the read — it would only return empty while still prompting the user to approve it — and tell the user their linked accounts don't include a brokerage. Otherwise `plaid investments-holdings` returns current positions, holdings, and securities. Holdings are priced as of `institution_price_as_of`. For buy/sell/dividend activity, use `plaid investments-transactions --start-date YYYY-MM-DD --end-date YYYY-MM-DD`; page it with `--offset`/`--count` (default 100, max 500) until the fetched count reaches `body.total_investment_transactions`.

读取持仓之前，先查看 `plaid accounts` 中是否存在 `type` 为 `investment` 的账户（券商/退休账户）。若没有关联，跳过该读取——它只会返回空结果，却仍会让用户批准授权——并告知用户其关联账户中不含券商账户。否则 `plaid investments-holdings` 返回当前仓位、持仓和证券。持仓价格截至 `institution_price_as_of`。查询买入/卖出/分红活动，使用 `plaid investments-transactions --start-date YYYY-MM-DD --end-date YYYY-MM-DD`；用 `--offset`/`--count`（默认 100，最大 500）翻页，直到获取数量达到 `body.total_investment_transactions`。

## Rules / 规则

- Everything you say to the user is plain English. The commands, flags, cursors, and JSON output are for you, not the user. Keep them out of your replies: no command or flag (`plaid`, `transactions-sync`, `--credential-id`), no field name (`next_cursors`, `inflow_streams`, `total_investment_transactions`), no raw JSON, and no `credential_id`, account id, or cursor. Name the account in words ("your Chase checking"); you may add the masked last digits ("…4471") only to tell two similar accounts apart, but never an internal id. When you give a figure, say which accounts and what time range it covers so it matches what the user sees at their bank.
  你对用户说的一切都用平实语言。命令、标志、游标和 JSON 输出是给你用的，不是给用户的。不要让它们出现在回复中：不出现命令或标志（`plaid`、`transactions-sync`、`--credential-id`），不出现字段名（`next_cursors`、`inflow_streams`、`total_investment_transactions`），不出现原始 JSON，也不出现 `credential_id`、账户 id 或游标。用文字指称账户（"你的 Chase 支票账户"）；仅在需要区分两个相似账户时可以补充掩码尾号（"…4471"），但绝不给出内部 id。给出数字时，说明它涵盖哪些账户和哪个时间范围，使其与用户在银行看到的一致。
- Never print or repeat secrets, tokens, or full account or routing numbers. Plaid only exposes masked numbers (last few digits); share at most the mask, and never a reconstructed full number.
  绝不打印或复述机密、令牌或完整的账号/路由号码。Plaid 只暴露掩码号码（最后几位）；最多分享掩码，绝不复现完整号码。
- This is read-only reference, not financial advice. Report what the data shows; don't tell the user to buy, sell, refinance, or move money, and don't guarantee outcomes.
  本文档是只读参考，不是理财建议。报告数据所显示的内容；不要建议用户买入、卖出、再融资或转移资金，也不要保证结果。
- Plaid data may lag behind what the bank currently shows. If a balance or a recent charge looks stale or missing, say the data may not be fully up to date rather than asserting it's wrong or complete.
  Plaid 数据可能滞后于银行当前显示的内容。若某笔余额或近期扣款看起来过时或缺失，应说明数据可能未完全更新，而不是断言它有误或完整。
- If a read returns nothing for an account or institution, say so plainly instead of inventing balances, transactions, or holdings.
  若某账户或机构的读取没有返回任何内容，如实说明，而不是编造余额、交易或持仓。
- Transaction reads preserve Plaid's raw dates. Sync, recurring, and investment reads also add `transaction_posted_at` / `transaction_authorized_at` for true instants in UTC and the user's timezone. Keep date-only fields as dates.
  交易读取保留 Plaid 的原始日期。Sync、recurring 和 investment 读取还会添加 `transaction_posted_at` / `transaction_authorized_at`，以提供 UTC 和用户时区的真实时刻。仅含日期的字段保持为日期。

## Limits / 限制

- Read-only. You can't move money, pay a bill, transfer funds, open or close accounts, or change anything at the bank. For those, tell the user to use their bank directly.
  只读。你不能转账、付账单、划转资金、开户或销户，也不能在银行端更改任何内容。此类需求请用户直接使用其银行。
- Muse never sees full account or routing numbers, only masked digits, so you can't supply them for wire transfers or direct-deposit setup.
  Muse 从不接触完整的账号或路由号码，只看到掩码数字，因此你无法为电汇或直接存款设置提供它们。
- Coverage depends on what the user linked and what each bank shares through Plaid. A missing account type (say, a loan the bank doesn't expose through Plaid) isn't an error — tell the user it isn't available rather than treating it as zero.
  覆盖范围取决于用户关联了什么以及各家银行通过 Plaid 共享了什么。缺失的账户类型（例如银行未通过 Plaid 暴露的贷款）不是错误——告知用户该项不可用，而不是把它当作零。
