---
name: "zoom"
description: >-
  Review Zoom meetings, calls, recordings, and transcripts. Use to
  catch up on missed calls, summarize decisions, and extract action items and
  owners; also work with Team Chat, Canvas, Tasks, Whiteboard, Hub, and Revenue
  Accelerator through Zoom's official MCP servers.
icon: "zoom"
metadata: { "includeInPrompt": false }
---

<!-- BILINGUAL-EN-ZH -->
# Zoom / Zoom

Use the installed `zoom` CLI. Start with `zoom status`. If it reports
`not_connected`, run `zoom authorize-url` and share only the returned
`connect_url` with the user.

使用已安装的 `zoom` CLI。先运行 `zoom status`。如果它报告 `not_connected`，则运行 `zoom authorize-url`，并只把返回的 `connect_url` 分享给用户。

OAuth uses the fixed Muse Zoom OAuth client and PKCE through authd. CAGI supplies
the client secret for token exchange and refresh; the secret must never enter
Muse or be requested in chat.

OAuth 使用固定的 Muse Zoom OAuth 客户端，并经由 authd 执行 PKCE。CAGI 为令牌交换与刷新提供客户端密钥；该密钥绝不能进入 Muse，也绝不能在聊天中被索要。
【评论】与无密钥的公共客户端方案不同，这里采用密钥由服务端持有的机密客户端，模型侧始终接触不到凭据。

Run `zoom list-tools` to inspect the live catalogues and schemas from every
official Zoom MCP server. To inspect only one server, use `zoom list-tools
--server <server>`, where `<server>` is one of `zoom`, `meeting`, `chat`,
`canvas`, `tasks`, `whiteboard`, or `revenue-accelerator`.

运行 `zoom list-tools` 查看每个官方 Zoom MCP 服务器的实时目录与模式。若只想查看某一个服务器，使用 `zoom list-tools --server <server>`，其中 `<server>` 是 `zoom`、`meeting`、`chat`、`canvas`、`tasks`、`whiteboard` 或 `revenue-accelerator` 之一。

Call an advertised tool on the server that returned it with:

用以下方式在返回该工具的服务器上调用已通告的工具：

```text
zoom call-tool --server <server> --name <tool> --arguments-json '<json-object>'
```

The `zoom` server is the default for backward compatibility. The dedicated
`meeting` server currently overlaps with the meeting and recording tools on the
all-in-one `zoom` server, while the other dedicated servers add broader product
capabilities.

`zoom` 服务器是为向后兼容而保留的默认值。专用的 `meeting` 服务器目前与一体化 `zoom` 服务器上的会议和录制工具存在重叠，而其他专用服务器则提供更广泛的产品能力。

Provider catalogues can contain both reads and mutations and may change over
time. The CLI validates the name against the selected server's live catalogue
and requires explicit approval before every tool call. Do not retry a failed or
timed-out tool call automatically because its side effect may have completed.

提供方目录可能同时包含读取与变更类工具，且会随时间变化。CLI 会对照所选服务器的实时目录校验工具名，并在每次工具调用前要求显式批准。不要自动重试失败或超时的工具调用，因为其副作用可能已经完成。

If a dedicated server reports a missing OAuth scope after an upgrade, ask the
user to reconnect Zoom so the expanded grant can be approved. Never disconnect
an existing connection without the user's confirmation.

如果某个专用服务器在升级后报告缺少 OAuth 作用域，请用户重新连接 Zoom，以便批准扩展后的授权。未经用户确认，绝不断开现有连接。

Some servers require separately licensed Zoom products. Treat a per-server
error as that server being unavailable; continue using catalogues that report
`ok: true` rather than claiming the whole Zoom connection failed.

某些服务器需要单独许可的 Zoom 产品。把单服务器错误视为该服务器不可用；继续使用报告 `ok: true` 的目录，而不要声称整个 Zoom 连接失败。

The `zoom` and `meeting` servers currently advertise `meeting_create`,
`meeting_update`, and `meeting_delete`, so use those live tools for meeting
scheduling and management. Do not infer availability merely from OAuth scopes:
confirm the tool and its current schema with `zoom list-tools --server meeting`
before calling it.

`zoom` 和 `meeting` 服务器当前通告 `meeting_create`、`meeting_update` 和 `meeting_delete`，因此会议的排期与管理要使用这些实时工具。不要仅凭 OAuth 作用域推断可用性：调用前先用 `zoom list-tools --server meeting` 确认该工具及其当前模式。
