---
name: "tessie"
description: "Monitor a Tesla vehicle, inspect live state, and run explicit Tessie command endpoints."
icon: "tessie"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Tessie (Tesla Vehicle Control) / Tessie（Tesla 车辆控制）

## Purpose / 用途
Use `tessie-api` to list vehicles, inspect status/state, and run explicit Tessie vehicle commands.

使用 `tessie-api` 列出车辆、查看状态/实时状态，并执行明确的 Tessie 车辆命令。

## Tooling / 工具
Use:

使用：

```sh
tessie-api <subcommand> [options]
```

Core subcommands:
核心子命令：

- `authorize-url`
  （获取授权链接）
- `set-token --token '<TOKEN>'`
  （设置令牌）
- `disconnect`
  （断开连接）
- `verify`
  （验证连接）
- `vehicles`
  （列出车辆）
- `status --vin '<VIN>'`
  （查询车辆状态）
- `state --vin '<VIN>'`
  （查询车辆实时状态）
- `command --vin '<VIN>' --command <command_name> [--wait-for-completion] [--extra-query k=v]`
  （执行车辆命令，可选等待完成、附加查询参数）
- `request --method GET --path '/<VIN>/battery' [--query k=v] [--json-body '{"...":"..."}']`
  （发送原始 API 请求：指定方法、路径、查询参数与可选 JSON 请求体）

JSON output contract:
JSON 输出约定：

- `authorize-url`: parse `ok`, `authorize_url`, and `connect_url`
  - `authorize-url`：解析 `ok`、`authorize_url` 和 `connect_url`
- `set-token`: parse `ok` and `action`
  - `set-token`：解析 `ok` 和 `action`
- `disconnect`: parse `ok`, `action`, `path`, and `removed`
  - `disconnect`：解析 `ok`、`action`、`path` 和 `removed`
- `verify`: parse `ok`, `action`, and `error`
  - `verify`：解析 `ok`、`action` 和 `error`
- all read/command calls: parse top-level `ok`, `status`, and `body`
  - 所有读取/命令调用：解析顶层的 `ok`、`status` 和 `body`
- common `body` fields include `results[]`, `battery_level`, `battery_range`, `status`, and `result`
  - 常见的 `body` 字段包括 `results[]`、`battery_level`、`battery_range`、`status` 和 `result`

## Auth / 认证
Manage Tessie auth through the CLI helpers only.

只通过 CLI 辅助命令管理 Tessie 认证。

Auth contract:
认证约定：

- Run `tessie-api verify` before Tessie API use when connection state is unknown.
  - 在连接状态未知时，使用 Tessie API 之前先运行 `tessie-api verify`。
- If no API key is configured, run `tessie-api authorize-url`. When `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Tessie](<connect_url>)`; do not paste the raw URL separately.
  - 如果未配置 API 密钥，运行 `tessie-api authorize-url`。当返回 `connect_url` 时，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Tessie](<connect_url>)`；不要单独粘贴原始 URL。
- The user can generate a token at `https://dash.tessie.com/settings/api`.
  - 用户可以在 `https://dash.tessie.com/settings/api` 生成令牌。
- If the user provides a key through the CLI flow, store it only through `tessie-api set-token --token '<TESSIE_API_TOKEN>'`; do not write auth files directly.
  - 如果用户通过 CLI 流程提供密钥，只能通过 `tessie-api set-token --token '<TESSIE_API_TOKEN>'` 存储；不要直接写认证文件。
- Remove stored auth only through `tessie-api disconnect`.
  - 只能通过 `tessie-api disconnect` 删除已存储的认证。
- Never print token values.
  - 绝不打印令牌值。

## Operating Rules / 操作规则
1. If VIN is not given, call `vehicles` and ask user to pick one when multiple exist.
   1. 如果未提供 VIN，调用 `vehicles`；存在多辆车时请用户选择。
2. Use `status` or `state` before impactful commands when the current vehicle condition matters.
   2. 当车辆当前状况影响判断时，在执行有影响的命令之前先用 `status` 或 `state` 查询。
3. `honk`, `flash`, `remote_boombox`, software-update scheduling or cancellation, and fleet telemetry configuration may proceed from a clear, unambiguous request without an additional confirmation. Other vehicle commands and raw API writes use the connector approval gate; invoke the resolved command directly and do not add a duplicate chat confirmation. Never invent a command or infer a physical action the user did not request.
   3. `honk`、`flash`、`remote_boombox`、软件更新的排期或取消，以及车队遥测配置，在有清晰、无歧义的请求时可以直接执行，无需额外确认。其他车辆命令和原始 API 写操作走连接器审批门；直接调用已解析的命令，不要在聊天中重复追加确认。绝不编造命令，也绝不推断用户没有请求的物理动作。
4. Use `command --command <name>` only for explicit Tessie command names. Do not invent a mandatory pre-wake flow or undocumented helper behavior.
   4. `command --command <name>` 只用于明确的 Tessie 命令名称。不要编造强制的预唤醒流程或未公开的辅助行为。
5. On HTTP or API errors, report `status` and `body` clearly. Common cases are `401`, `408`, and `503`.
   5. 遇到 HTTP 或 API 错误时，清楚地报告 `status` 和 `body`。常见情况是 `401`、`408` 和 `503`。
6. Never print token values.
   6. 绝不打印令牌值。
7. Read results preserve Tessie's raw timestamps and add semantic UTC and
   user-local forms for state observations, last-seen values, and record
   creation/update times.
   7. 读取结果保留 Tessie 的原始时间戳，并针对状态观测值、最后可见值以及记录的创建/更新时间，补充语义化的 UTC 和用户本地时间形式。

【评论】规则 3 划出了一条"低风险动作免确认、其他一律走审批门"的分级授权线，并明确禁止推断用户未请求的物理动作——车辆控制属于高物理影响场景，这类"禁止推断"条款是典型的安全护栏。
