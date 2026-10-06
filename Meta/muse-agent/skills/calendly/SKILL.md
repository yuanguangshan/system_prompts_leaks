---
name: "calendly"
description: "View Calendly events and event types, and manage scheduling data using the Calendly CLI."
icon: "calendly"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Calendly / Calendly

## Purpose / 用途
Use the `calendly` CLI for Calendly connector status and scheduling-data reads.

使用 `calendly` CLI 查看 Calendly 连接器状态并读取排期数据。

## Tooling / 工具
Use the installed CLI directly from `PATH`.

直接使用 `PATH` 中已安装的 CLI。

Core commands:
核心命令：

- `calendly status`
  （`calendly status`：查看连接状态）
- `calendly disconnect`
  （`calendly disconnect`：断开连接）
- `calendly authorize-url`
  （`calendly authorize-url`：获取授权链接）
- `calendly me`
  （`calendly me`：查看当前用户）
- `calendly list-events --user-uri <uri> [--status active] [--min-start-time <iso>] [--max-start-time <iso>] [--count 20] [--page-token <token>]`
  （列出指定用户的日程事件，可按状态、起止时间过滤，支持分页）
- `calendly event-types --user-uri <uri> [--active true] [--count 20] [--page-token <token>]`
  （列出指定用户的事件类型，可按是否启用过滤，支持分页）
- `calendly request --method GET --path '/scheduled_events' --query 'user=<uri>' [--query 'status=active'] [--json-body '<json>']`
  （发送原始 API 请求：指定方法、路径、查询参数与可选 JSON 请求体）

Output contract:
输出约定：

- Every command returns JSON with a top-level `ok`.
  - 每个命令都返回带有顶层 `ok` 字段的 JSON。
- For `disconnect`, parse `action` and `removed`.
  - 对于 `disconnect`，解析 `action` 和 `removed`。
- For `status`, parse the reported connection state and any returned auth URL or recovery details.
  - 对于 `status`，解析报告的连接状态以及返回的认证 URL 或恢复详情（如有）。
- For `me`, parse the current user URI and organization URI before listing user-scoped resources.
  - 对于 `me`，在列出用户范围的资源之前，先解析当前用户 URI 和组织 URI。
- For `list-events` and `event-types`, parse the returned collection plus any pagination token for follow-up pages. Timed scheduled events add `event_starts_at` / `event_ends_at` with UTC and user-local forms, and read outputs add runtime-generated `retrieved_at`.
  - 对于 `list-events` 和 `event-types`，解析返回的集合以及用于后续分页的分页令牌（如有）。带时间的日程事件会附加 `event_starts_at` / `event_ends_at`（含 UTC 和用户本地两种形式），读取类输出会附加运行时生成的 `retrieved_at`。

## Auth / 认证
`calendly` owns the Calendly connection workflow.

`calendly` 负责 Calendly 的连接流程。

Auth contract:
认证约定：

- Run `calendly status` first.
  - 先运行 `calendly status`。
- If the user wants to disconnect, run `calendly disconnect`.
  - 如果用户想断开连接，运行 `calendly disconnect`。
- If not connected, run `calendly authorize-url`. When `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Calendly](<connect_url>)`; do not paste the raw URL separately.
  - 如果尚未连接，运行 `calendly authorize-url`。当返回 `connect_url` 时，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Calendly](<connect_url>)`；不要单独粘贴原始 URL。
- After auth, run `calendly me` to verify.
  - 认证完成后，运行 `calendly me` 进行验证。

Credential storage:
凭据存储：

- Client application credentials and user token state are never read directly by this skill.
  - 本技能从不直接读取客户端应用凭据和用户令牌状态。
- The CLI accesses connection state exclusively through shared `hatch-tool-sdk` connector helpers; there is no agent-visible config file path.
  - CLI 完全通过共享的 `hatch-tool-sdk` 连接器辅助模块访问连接状态；没有任何对代理可见的配置文件路径。
- The CLI resolves and refreshes service tokens automatically; never print access or refresh tokens.
  - CLI 会自动解析并刷新服务令牌；绝不打印访问令牌或刷新令牌。

## Operating Rules / 操作规则
1. Before any Calendly API call, require `calendly status` to show a connected state.
   1. 在任何 Calendly API 调用之前，必须确认 `calendly status` 显示已连接状态。
2. Use `calendly me` first when you need the current user URI or organization URI.
   2. 当需要当前用户 URI 或组织 URI 时，先使用 `calendly me`。
3. Token refresh on expired-token errors is automatic. If API calls still fail after auto-refresh, re-check `calendly status` and re-link if needed.
   3. 遇到令牌过期错误时自动刷新令牌。如果自动刷新后 API 调用仍然失败，重新检查 `calendly status` 并在需要时重新建立连接。
4. Confirm before any destructive or user-visible mutation, including raw `request` calls that could cancel events or delete subscriptions.
   4. 任何破坏性或用户可见的变更操作之前都要先确认，包括可能取消事件或删除订阅的原始 `request` 调用。
5. Prefer `event_starts_at.user_local` and `event_ends_at.user_local` in summaries.
   5. 在摘要中优先使用 `event_starts_at.user_local` 和 `event_ends_at.user_local`。

【评论】该技能通过"只分享 Markdown 链接、不单独粘贴原始 URL"的条款约束输出形态，属于对认证流程呈现方式的规范化设计；同时明确禁止打印令牌，属于凭据泄露防护。
