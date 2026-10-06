---
name: "printify"
description: "Use Printify to browse catalog data, manage shops and products, and review or create orders."
icon: "printify"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Printify (Print on Demand) / Printify（按需印刷）

## Purpose / 用途
Use the `printify` CLI to manage shops, products, uploads, orders, and catalog lookups.

使用 `printify` CLI 管理店铺、商品、上传、订单以及目录查询。

## Tooling / 工具
Use the installed CLI directly from `PATH`:

直接使用 `PATH` 中已安装的 CLI：

```sh
printify <subcommand> [options]
```

Global flags:
全局标志：

- `--timeout-secs <seconds>` (optional, default 30): HTTP timeout.
  - `--timeout-secs <seconds>`（可选，默认 30）：HTTP 超时。

Core commands:
核心命令：

- `status`
  （查看状态）
- `authorize-url`
  （获取授权链接）
- `verify`
  （验证连接）
- `set-token` (reads token from stdin)
  （设置令牌，从 stdin 读取令牌）
- `disconnect`
  （断开连接）
- `shops`
  （列出店铺）
- `products --shop-id <id> [--product-id <id>]`
  （查询商品）
- `create-product --shop-id <id> --json '<json>'`
  （创建商品）
- `update-product --shop-id <id> --product-id <id> --json '<json>'`
  （更新商品）
- `delete-product --shop-id <id> --product-id <id>`
  （删除商品）
- `publish-product --shop-id <id> --product-id <id>`
  （发布商品）
- `orders --shop-id <id> [--order-id <id>]`
  （查询订单）
- `create-order --shop-id <id> --json '<json>'`
  （创建订单）
- `upload --file <path>`
  （上传文件）
- `blueprints [--blueprint-id <id>]`
  （查询蓝图目录）
- `print-providers --blueprint-id <id>`
  （查询印刷供应商）
- `variants --blueprint-id <id> --provider-id <id>`
  （查询变体）
- `shipping --blueprint-id <id> --provider-id <id>`
  （查询运费）

JSON output contract:
JSON 输出约定：

- `status`: parse `ok`, `status`, `connect_url`, `disconnect_url`, and `reason`
  - `status`：解析 `ok`、`status`、`connect_url`、`disconnect_url` 和 `reason`
- `authorize-url`: parse `ok`, `authorize_url`, and `connect_url`
  - `authorize-url`：解析 `ok`、`authorize_url` 和 `connect_url`
- `verify`: parse `ok`, `action`, and `error`
  - `verify`：解析 `ok`、`action` 和 `error`
- `set-token`: parse `ok`, `status`, and `reason`
  - `set-token`：解析 `ok`、`status` 和 `reason`
- `disconnect`: parse `ok`, `action`, `config_path`, `removed`
  - `disconnect`：解析 `ok`、`action`、`config_path`、`removed`
- parse `ok`, `status`, `body`, and `error`
  - 解析 `ok`、`status`、`body` 和 `error`
- Read results retain Printify's raw timestamps and add semantic UTC and
  user-local forms for record creation/update and order production,
  fulfillment, delivery, or cancellation times.
  - 读取结果保留 Printify 的原始时间戳，并针对记录的创建/更新以及订单的生产、履约、交付或取消时间，补充语义化的 UTC 和用户本地时间形式。

## Auth / 认证
First-use setup:
首次使用设置：

1. Run `printify status` to check connection state.
   1. 运行 `printify status` 检查连接状态。
2. If not connected and `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Printify](<connect_url>)`; do not paste the raw URL separately.
   2. 如果未连接且返回了 `connect_url`，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Printify](<connect_url>)`；不要单独粘贴原始 URL。
3. If `connect_url` is missing, run `printify authorize-url` and share the returned `connect_url` the same way.
   3. 如果缺少 `connect_url`，运行 `printify authorize-url`，并以同样方式分享返回的 `connect_url`。
4. The user can generate a token at `https://printify.com/app/account/api`.
   4. 用户可以在 `https://printify.com/app/account/api` 生成令牌。
5. If the user provides a token through the CLI flow, store it only through `printify set-token`; do not write auth files directly.
   5. 如果用户通过 CLI 流程提供令牌，只能通过 `printify set-token` 存储；不要直接写认证文件。
6. Run `printify verify` after setup when you need to confirm the token works.
   6. 设置完成后，如需确认令牌可用，运行 `printify verify`。
7. Never print the access token in chat output.
   7. 绝不在聊天输出中打印访问令牌。

## Operating Rules / 操作规则
1. Always check `printify status` before API calls. If not connected, guide the user through the auth setup above.
   1. API 调用之前必须先检查 `printify status`。如果未连接，引导用户完成上述认证设置。
2. List shops before performing shop-specific operations to get the correct `shop_id`.
   2. 在执行店铺相关操作之前先列出店铺，以获得正确的 `shop_id`。
3. Confirm with the user before creating, updating, or publishing products. Product deletion may proceed from a clear, unambiguous request without an additional confirmation.
   3. 创建、更新或发布商品之前与用户确认。商品删除在有清晰、无歧义的请求时可以直接执行，无需额外确认。
4. Confirm with the user before creating orders (orders may trigger charges).
   4. 创建订单之前与用户确认（订单可能触发扣款）。
5. When browsing the catalog, start with `blueprints` then drill into `print-providers` and `variants`.
   5. 浏览目录时，从 `blueprints` 开始，再深入 `print-providers` 和 `variants`。
6. On HTTP 401, tell the user their token may be expired and to generate a new one at `https://printify.com/app/account/api`.
   6. 遇到 HTTP 401 时，告知用户令牌可能已过期，可在 `https://printify.com/app/account/api` 生成新令牌。

【评论】值得注意的反差设计：创建/更新/发布商品需要确认，而删除商品反而只需"清晰、无歧义的请求"即可执行——与常见的"删除从严"直觉相反，属于把删除视为不可逆但低金钱影响的操作分级。
