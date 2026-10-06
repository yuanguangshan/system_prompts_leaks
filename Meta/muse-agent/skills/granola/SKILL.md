---
name: "granola"
description: "Search and read Granola meeting notes and transcripts through Granola's OAuth-backed MCP server."
icon: "granola"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Granola / Granola

## Purpose / 用途
Search Granola meeting notes, summaries, and transcripts through the hosted  
Granola MCP server (`https://mcp.granola.ai/mcp`).

通过托管的 Granola MCP 服务器（`https://mcp.granola.ai/mcp`）搜索 Granola 会议笔记、摘要和转录文本。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
granola-cli <subcommand> [options]
```

### Connection management / 连接管理

```sh
granola-cli status
granola-cli authorize-url
granola-cli exchange-code --code <code> [--redirect-uri <url>]
granola-cli refresh
granola-cli disconnect
```

### MCP operations / MCP 操作

```sh
granola-cli list-tools
granola-cli call-tool --name <tool> --arguments-json '<json-object>'
```

`--arguments-json` must be a JSON object; arrays or scalars are rejected. Use
`list-tools` first to discover the tool catalogue and each tool's  
`input_schema`.

`--arguments-json` 必须是 JSON 对象；数组或标量会被拒绝。先用 `list-tools` 发现工具目录及每个工具的 `input_schema`。

## Auth / 认证
OAuth is handled by authd via Dynamic Client Registration + PKCE (S256). The
credential stays behind authd; do not read or edit connector auth files.

OAuth 由 authd 通过动态客户端注册 + PKCE（S256）处理。凭据保存在 authd 之后；不要读取或编辑连接器的认证文件。

## First-use setup flow / 首次使用设置流程

1. Run `granola-cli status`.
   1. 运行 `granola-cli status`。
2. If status is `not_connected`, run `granola-cli authorize-url`. When
   `connect_url` is present, replace `<connect_url>` with the returned URL
   and share exactly this Markdown link: `[Connect Granola](<connect_url>)`; do not paste the raw
   URL separately. Wait for the user to complete authorization.
   2. 如果状态为 `not_connected`，运行 `granola-cli authorize-url`。当返回 `connect_url` 时，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Granola](<connect_url>)`；不要单独粘贴原始 URL。等待用户完成授权。
3. After the user authorizes, the browser is redirected via the Muse relay
   back to this VM, and authd completes the code exchange automatically. The
   agent resumes once the token lands.
   3. 用户授权后，浏览器经 Muse 中继重定向回本 VM，authd 自动完成授权码交换。令牌落地后代理即恢复工作。
4. Re-run `granola-cli status`. When status flips to `connected` the output also
   includes the discovered MCP tool catalogue.
   4. 重新运行 `granola-cli status`。当状态变为 `connected` 时，输出还会包含发现到的 MCP 工具目录。

For manual environments where the relay is not wired up, call
`granola-cli exchange-code --code <auth-code>` after receiving the code.

在中继未接通的手动环境中，收到授权码后调用 `granola-cli exchange-code --code <auth-code>`。

## Operating Rules / 操作规则
1. Run `granola-cli status` before any MCP work. If not `connected`, complete
   the setup flow first.
   1. 任何 MCP 工作之前先运行 `granola-cli status`。如果不是 `connected`，先完成设置流程。
2. Call `list-tools` before `call-tool` unless you already know the tool name
   and its argument shape. Never guess tool names.
   2. 除非已知工具名称及其参数形态，否则在 `call-tool` 之前先调用 `list-tools`。绝不猜测工具名称。
3. `arguments-json` must be a JSON object; wrap every argument appropriately.
   3. `arguments-json` 必须是 JSON 对象；对每个参数进行恰当的包装。
4. Prefer `query_granola_meetings` for natural-language questions,
   `list_meetings` for metadata, `get_meetings` for known meeting IDs, and
   `get_meeting_transcript` only when the user needs verbatim detail. Use
   `get_account_info` to answer which account/workspace is connected and,
   when results come back empty, to check `mcp_note_access.scopes` — a
   workspace whose MCP access is `public`-only excludes personal notes.
   `list_meetings` defaults to the last 30 days; when it returns zero
   meetings, retry with `time_range: "custom"` plus `custom_start` and
   `custom_end` before concluding the account has no meetings, and use
   `list_meeting_folders` to see what the workspace actually holds.
   4. 自然语言问题优先用 `query_granola_meetings`，元数据用 `list_meetings`，已知会议 ID 用 `get_meetings`，只有当用户需要逐字细节时才用 `get_meeting_transcript`。用 `get_account_info` 回答当前连接的是哪个账户/工作区；当结果为空时，用它检查 `mcp_note_access.scopes`——MCP 访问仅为 `public` 的工作区不包含个人笔记。`list_meetings` 默认只查最近 30 天；当它返回零条会议时，先用 `time_range: "custom"` 加 `custom_start` 和 `custom_end` 重试，再下结论说账户没有会议，并用 `list_meeting_folders` 查看工作区实际包含的内容。
5. Preserve Granola citation links in user-facing answers.
   5. 在面向用户的回答中保留 Granola 的引用链接。
6. Do not use Granola for calendar scheduling or upcoming-event planning.
   6. 不要把 Granola 用于日历排期或日程规划。
7. Token refresh happens automatically on 401. If `call-tool` keeps failing
   with `unauthorized`, run `granola-cli refresh` explicitly or ask the user to
   re-authorize.
   7. 遇到 401 时自动刷新令牌。如果 `call-tool` 持续以 `unauthorized` 失败，显式运行 `granola-cli refresh`，或请用户重新授权。

【评论】"results come back empty → 先查 scopes 再下结论"的排障顺序设计，把权限范围问题（`public`-only 排除个人笔记）列为空结果的常见根因，避免代理把权限缺陷误报为数据不存在。
