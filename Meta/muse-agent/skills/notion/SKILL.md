---
name: "notion"
description: "Search, read, create, and update Notion pages via the Notion MCP."
icon: "notion"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Notion / Notion

## Purpose / 用途
Interact with Notion pages, databases, and workspace content via the hosted
Notion MCP server (`https://mcp.notion.com/mcp`).

通过托管的 Notion MCP 服务器（`https://mcp.notion.com/mcp`）与 Notion 页面、数据库和工作区内容交互。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
notion-cli <subcommand> [options]
```

### Connection management / 连接管理

```sh
notion-cli status
notion-cli authorize-url
notion-cli exchange-code --code <code> [--redirect-uri <url>]
notion-cli refresh
notion-cli disconnect
```

### MCP operations / MCP 操作

```sh
notion-cli list-tools
notion-cli call-tool --name <tool> --arguments-json '<json-object>'
```

`--arguments-json` must be a JSON object; arrays or scalars are rejected. Use
`list-tools` first to discover the tool catalogue and each tool's  
`input_schema`.

`--arguments-json` 必须是 JSON 对象；数组或标量会被拒绝。先用 `list-tools` 发现工具目录及每个工具的 `input_schema`。

## Auth / 认证
OAuth is handled by authd via Dynamic Client Registration + PKCE (S256). The
access token is stored at `$JARVIS_HOME/user/auth/notion.json`. Do not hand-edit
this file.

OAuth 由 authd 处理，采用动态客户端注册 + PKCE（S256）。访问令牌存储在 `$JARVIS_HOME/user/auth/notion.json`。不要手动编辑该文件。

## First-use setup flow / 首次使用设置流程

1. Run `notion-cli status`.
   运行 `notion-cli status`。
2. If status is `not_connected`, run `notion-cli authorize-url`. When
   `connect_url` is present, replace `<connect_url>` with the returned URL
   and share exactly this Markdown link: `[Connect Notion](<connect_url>)`; do not paste the raw
   URL separately. Wait for the user to complete authorization.
   如果状态为 `not_connected`，运行 `notion-cli authorize-url`。当返回 `connect_url` 时，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Notion](<connect_url>)`；不要另行粘贴原始 URL。等待用户完成授权。
   【评论】要求链接格式一字不差，是因为客户端按固定模式识别该链接并渲染为原生授权按钮。
3. After the user authorizes, the browser is redirected via the Muse relay
   back to this VM, and authd completes the code exchange automatically. The
   agent resumes once the token lands.
   用户授权后，浏览器经 Muse 中继重定向回本 VM，authd 自动完成 code 交换。令牌落定后助手即继续执行。
4. Re-run `notion-cli status`. When status flips to `connected` the output also
   includes the discovered MCP tool catalogue.
   重新运行 `notion-cli status`。当状态变为 `connected` 时，输出中还会包含已发现的 MCP 工具目录。

For manual environments where the relay is not wired up, call
`notion-cli exchange-code --code <auth-code>` after receiving the code.

对于中继未接通的手动环境，在收到 code 后调用 `notion-cli exchange-code --code <auth-code>`。

## Operating Rules / 操作规则
1. Run `notion-cli status` before any MCP work. If not `connected`, complete
   the setup flow first.
   在任何 MCP 工作之前先运行 `notion-cli status`。若状态不是 `connected`，先完成设置流程。
2. Call `list-tools` before `call-tool` unless you already know the tool name
   and its argument shape. Never guess tool names.
   在 `call-tool` 之前先调用 `list-tools`，除非你已确知工具名及其参数结构。绝不猜测工具名。
3. `arguments-json` must be a JSON object; wrap every argument appropriately.
   `arguments-json` 必须是 JSON 对象；每个参数都要正确包裹。
4. Confirm user intent before tool calls that mutate Notion pages, databases,
   or blocks.
   在调用会变更 Notion 页面、数据库或块的工具之前，先确认用户意图。
5. Token refresh happens automatically on 401. If `call-tool` keeps failing
   with `unauthorized`, run `notion-cli refresh` explicitly or ask the user to
   re-authorize.
   遇到 401 时会自动刷新令牌。如果 `call-tool` 持续报 `unauthorized`，则显式运行 `notion-cli refresh`，或请用户重新授权。
6. Read results preserve Notion's raw date fields and add semantic UTC and
   user-local forms for record creation/edit times and timed date properties.
   Date-only properties remain dates.
   读取结果保留 Notion 的原始日期字段，并为记录创建/编辑时间和带时间的日期属性附加语义化的 UTC 与用户本地时间形式。纯日期属性仍保持为日期。
