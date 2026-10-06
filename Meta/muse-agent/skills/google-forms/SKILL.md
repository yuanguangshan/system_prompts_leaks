---
name: "google_forms"
description: "Read, create, and update the user's Google Forms, and read responses."
icon: "google_forms"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Google Forms

## Purpose / 目的
Manage Google Forms through `hatch_gws_cli`; the vendored Google Workspace CLI is the only implementation path.

通过 `hatch_gws_cli` 管理 Google Forms；内置（vendored）的 Google Workspace CLI 是唯一的实现路径。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
hatch_gws_cli forms <resource> <method> [flags]
```

### Connection management / 连接管理

```sh
hatch_gws_cli forms status
hatch_gws_cli forms disconnect
```

### Forms operations / Forms 操作

Core patterns / 核心模式：
- `hatch_gws_cli forms status`
- `hatch_gws_cli forms disconnect`
- `hatch_gws_cli schema forms.forms.get`
- `hatch_gws_cli schema forms.forms.create`
- `hatch_gws_cli schema forms.forms.batchUpdate`
- `hatch_gws_cli schema forms.forms.responses.list`
- `hatch_gws_cli schema forms.forms.responses.get`

Common raw API calls / 常见原始 API 调用：
- `hatch_gws_cli forms forms get --params '{"formId":"<form_id>"}'`
- `hatch_gws_cli forms forms create --json '{"info":{"title":"Feedback Survey","documentTitle":"Feedback Survey"}}'`
- `hatch_gws_cli forms forms batchUpdate --params '{"formId":"<form_id>"}' --json '{"requests":[{"updateFormInfo":{"info":{"description":"Quarterly survey"},"updateMask":"description"}}]}'`
- `hatch_gws_cli forms forms responses list --params '{"formId":"<form_id>","pageSize":20}'`
- `hatch_gws_cli forms forms responses get --params '{"formId":"<form_id>","responseId":"<response_id>"}'`

JSON output contract / JSON 输出契约：
- `status`: parse `status`, `connect_url`, and `disconnect_url`
  `status`：解析 `status`、`connect_url` 和 `disconnect_url`
- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
  `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`

## Auth / 认证
Authentication is handled by the wrapper's `status` and `disconnect` subcommands. Do not hand-write credential files or run raw `gws auth ...`.

身份验证由包装器的 `status` 和 `disconnect` 子命令处理。不要手工编写凭据文件或直接运行 `gws auth ...`。

## First-use setup flow / 首次使用设置流程
1. Run `hatch_gws_cli forms status`.
   运行 `hatch_gws_cli forms status`。
2. If `status` is `unavailable`, tell the user that Google Forms is not available on this device. Do not offer alternative integration approaches or ask the user for credentials.
   如果 `status` 为 `unavailable`，告知用户此设备上无法使用 Google Forms。不要提供替代的集成方案，也不要向用户索要凭据。
3. If `status` is `not_connected` and `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Google Forms](<connect_url>)`; do not paste the raw URL separately. Wait for the user to reconnect.
   如果 `status` 为 `not_connected` 且存在 `connect_url`，用返回的 URL 替换 `<connect_url>`，并原样分享这个 Markdown 链接：`[Connect Google Forms](<connect_url>)`；不要单独粘贴原始 URL。等待用户重新连接。
4. Once `status` is `connected`, proceed with Forms operations.
   一旦 `status` 为 `connected`，即可开始 Forms 操作。

## Operating Rules / 操作规则
1. Use `schema` before unfamiliar Forms methods so `--params` and `--json` match the current vendored CLI contract.
   在使用不熟悉的 Forms 方法之前先用 `schema` 查询，确保 `--params` 和 `--json` 符合当前内置 CLI 的契约。
2. Creating a form and editing an unpublished form owned only by the user may proceed from a clear request. Confirm before editing a shared or published form, or publishing one, because its contents can reach other people.
   创建表单，以及编辑仅由用户拥有的未发布表单，可依据明确请求直接进行。编辑共享或已发布的表单、或发布表单之前须先确认，因为其内容可能被其他人看到。

   【评论】该规则按表单的可见范围划分审批级别：仅用户私有的表单写入门槛较低，而涉及他人的共享/发布操作需要确认，属于按数据外泄风险分级的权限设计。
3. Run `forms.get` before presenting response data so you can map question IDs to human-readable form structure.
   在呈现响应数据之前先运行 `forms.get`，以便将问题 ID 映射到人类可读的表单结构。
4. Treat form IDs and response IDs as opaque strings and only use IDs returned by prior commands or explicit user input.
   将表单 ID 和响应 ID 视为不透明字符串，只使用先前命令返回的 ID 或用户显式输入的 ID。
5. Run `hatch_gws_cli forms disconnect`. After running it, when `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Google Forms](<disconnect_url>)`; do not paste the raw URL separately.
   运行 `hatch_gws_cli forms disconnect`。运行后若存在 `disconnect_url`，用返回的 URL 替换 `<disconnect_url>`，并原样分享这个 Markdown 链接：`[Disconnect Google Forms](<disconnect_url>)`；不要单独粘贴原始 URL。
6. Paginate through responses when the user asks for all responses; stop when `nextPageToken` is absent.
   当用户要求获取全部响应时进行分页遍历；当 `nextPageToken` 不存在时停止。
7. After a read or write action, confirm the user-visible result only. Do not surface raw API identifiers (form, response, and question IDs) or other internal response fields (page tokens/cursors, raw JSON) in text shown to the user unless the user asks for them or you need them to troubleshoot a failure; keep using them internally to chain follow-up commands.
   在读取或写入操作之后，只确认用户可见的结果。除非用户要求或你需要排查故障，否则不要在展示给用户的文本中出现原始 API 标识符（表单、响应和问题 ID）或其他内部响应字段（分页令牌/游标、原始 JSON）；在内部继续使用它们串联后续命令。
8. Response reads retain Google's raw timestamps and add semantic UTC and
   user-local forms for response creation/submission times.
   响应读取保留 Google 的原始时间戳，并为响应的创建/提交时间附加语义化的 UTC 与用户本地时间格式。
