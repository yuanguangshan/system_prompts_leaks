---
name: "google_sheets"
description: "Read, write, and manage the user's Google Sheets."
icon: "google_sheets"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Google Sheets / Google 表格

## Purpose / 用途
Manage Google Sheets through `hatch_gws_cli`; the vendored Google Workspace CLI is the only implementation path.

通过 `hatch_gws_cli` 管理 Google Sheets；随附（vendored）的 Google Workspace CLI 是唯一的实现路径。

## Tooling / 工具
Use `exec` to run:

使用 `exec` 运行：

```sh
hatch_gws_cli sheets <resource> <method> [flags]
```

### Connection management / 连接管理

```sh
hatch_gws_cli sheets status
hatch_gws_cli sheets disconnect
```

### Sheets operations / Sheets 操作

Core patterns:

核心模式：

- `hatch_gws_cli sheets status`
- `hatch_gws_cli sheets disconnect`
- `hatch_gws_cli schema sheets.spreadsheets.get`
- `hatch_gws_cli schema sheets.spreadsheets.create`
- `hatch_gws_cli schema sheets.spreadsheets.values.get`
- `hatch_gws_cli schema sheets.spreadsheets.values.append`
- `hatch_gws_cli schema sheets.spreadsheets.batchUpdate`

Common raw API calls:

常用的原始 API 调用：

- `hatch_gws_cli sheets spreadsheets get --params '{"spreadsheetId":"<spreadsheet_id>"}'`
- `hatch_gws_cli sheets spreadsheets create --json '{"properties":{"title":"My Spreadsheet"}}'`
- `hatch_gws_cli sheets spreadsheets values get --params '{"spreadsheetId":"<spreadsheet_id>","range":"Sheet1!A1:D10"}'`
- `hatch_gws_cli sheets spreadsheets values append --params '{"spreadsheetId":"<spreadsheet_id>","range":"Sheet1!A:D","valueInputOption":"USER_ENTERED"}' --json '{"values":[["a","b"],["c","d"]]}'`
- `hatch_gws_cli sheets spreadsheets batchUpdate --params '{"spreadsheetId":"<spreadsheet_id>"}' --json '{"requests":[{"addSheet":{"properties":{"title":"Q2"}}}]}'`

Vendored helpers are also available:

还可以使用随附的辅助命令：

- `hatch_gws_cli sheets +read ...`
- `hatch_gws_cli sheets +append ...`

JSON output contract:

JSON 输出契约：

- `status`: parse `status`, `connect_url`, and `disconnect_url`
  `status`：解析 `status`、`connect_url` 和 `disconnect_url`
- `disconnect`: parse `ok`, `action`, `status`, and `disconnect_url`
  `disconnect`：解析 `ok`、`action`、`status` 和 `disconnect_url`

## Formatting spreadsheet content / 电子表格内容的格式化

Write and format the spreadsheet through the Sheets API. Do not upload a file over it.

通过 Sheets API 写入电子表格并设置格式。不要改用上传文件的方式。

1. Write the values with `spreadsheets.create`, `values.update`, or `values.append`. Set `valueInputOption` to `USER_ENTERED` so Sheets parses dates, currency, and formulas.
   用 `spreadsheets.create`、`values.update` 或 `values.append` 写入数值。将 `valueInputOption` 设为 `USER_ENTERED`，让 Sheets 解析日期、货币和公式。
2. Set the styling in one `spreadsheets.batchUpdate` call. `repeatCell` with `userEnteredFormat` sets the header style and the number formats. `updateSheetProperties` freezes the header row. `updateDimensionProperties` sets column widths. A new spreadsheet can carry these formats in the `spreadsheets.create` body instead.
   在一次 `spreadsheets.batchUpdate` 调用中设置样式。`repeatCell` 配合 `userEnteredFormat` 设置表头样式和数字格式。`updateSheetProperties` 冻结表头行。`updateDimensionProperties` 设置列宽。新建电子表格时，也可以改为在 `spreadsheets.create` 请求体中直接携带这些格式。
3. Read the result back before you call it ready. A successful write proves the API accepted the request, not that the sheet reads correctly. `spreadsheets.get` returns no cell data by default, so pass a field mask: `hatch_gws_cli sheets spreadsheets get --params '{"spreadsheetId":"<spreadsheet_id>","ranges":"<tab>!A1:F10","fields":"sheets(properties(title,gridProperties(frozenRowCount)),data(rowData(values(formattedValue,effectiveFormat(backgroundColor,numberFormat,textFormat)))))"}'`
   在宣布完成之前先把结果读回来验证。写入成功只证明 API 接受了请求，并不代表表格显示正确。`spreadsheets.get` 默认不返回单元格数据，所以要传入字段掩码（field mask）：`hatch_gws_cli sheets spreadsheets get --params '{"spreadsheetId":"<spreadsheet_id>","ranges":"<tab>!A1:F10","fields":"sheets(properties(title,gridProperties(frozenRowCount)),data(rowData(values(formattedValue,effectiveFormat(backgroundColor,numberFormat,textFormat)))))"}'`
4. Never publish a `.xlsx` file over a spreadsheet with `drive files update`. Google replaces the full contents of the file, so that upload discards every other tab and every edit the user made. To change one part of a spreadsheet, use `values.update` or `spreadsheets.batchUpdate`. To give the user a workbook to download, build a spreadsheet artifact and attach the file.
   绝不要用 `drive files update` 把 `.xlsx` 文件覆盖发布到电子表格上。Google 会替换文件的完整内容，这种上传会丢弃所有其他工作表标签页以及用户做过的所有编辑。要修改电子表格的一部分，使用 `values.update` 或 `spreadsheets.batchUpdate`。要给用户一个可下载的工作簿，构建一个 spreadsheet artifact 并附加该文件。

## Auth / 认证
Authentication is handled by the wrapper's `status` and `disconnect` subcommands. Do not hand-write credential files or run raw `gws auth ...`.

认证由包装器的 `status` 和 `disconnect` 子命令处理。不要手写凭据文件，也不要直接运行原始的 `gws auth ...`。

## First-use setup flow / 首次使用设置流程
1. Run `hatch_gws_cli sheets status`.
   运行 `hatch_gws_cli sheets status`。
2. If `status` is `unavailable`, tell the user that Google Sheets is not available on this device. Do not offer alternative integration approaches or ask the user for credentials.
   如果 `status` 为 `unavailable`，告知用户此设备上 Google Sheets 不可用。不要提供替代集成方案，也不要向用户索要凭据。
3. If `status` is `not_connected` and `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Google Sheets](<connect_url>)`; do not paste the raw URL separately. Wait for the user to reconnect.
   如果 `status` 为 `not_connected` 且存在 `connect_url`，将 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Google Sheets](<connect_url>)`；不要单独粘贴原始 URL。等待用户完成重新连接。
4. Once `status` is `connected`, proceed with Sheets operations.
   一旦 `status` 为 `connected`，即可继续进行 Sheets 操作。

## Operating Rules / 操作规则
1. Use `schema` before unfamiliar Sheets methods so `--params` and `--json` match the current vendored CLI contract.
   在使用不熟悉的 Sheets 方法之前先查看 `schema`，确保 `--params` 和 `--json` 与当前随附 CLI 的契约一致。
2. Use A1 notation for ranges unless the user explicitly wants a different API path.
   范围使用 A1 表示法，除非用户明确要求其他 API 路径。
3. Creating a spreadsheet, clearing cells, and editing a spreadsheet owned only by the user may proceed from a clear request. Confirm before editing a shared spreadsheet because its contents can be exposed to or changed for other people.
   创建电子表格、清除单元格、编辑仅归用户所有的电子表格，可以在明确请求下直接进行。编辑共享电子表格前要先确认，因为其内容可能被暴露给其他人或因更改而影响他人。
4. Treat spreadsheet IDs and sheet IDs as opaque strings and only use IDs returned by prior commands or explicit user input.
   将电子表格 ID 和工作表 ID 视为不透明字符串，只使用先前命令返回的 ID 或用户明确输入的 ID。
5. Run `hatch_gws_cli sheets disconnect`. After running it, when `disconnect_url` is present, replace `<disconnect_url>` with the returned URL and share exactly this Markdown link: `[Disconnect Google Sheets](<disconnect_url>)`; do not paste the raw URL separately.
   运行 `hatch_gws_cli sheets disconnect`。运行之后，若存在 `disconnect_url`，将 `<disconnect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Disconnect Google Sheets](<disconnect_url>)`；不要单独粘贴原始 URL。
6. Prefer `spreadsheets.get` or `sheets +read` before writing when you need to inspect current sheet structure or cell contents. To write or style a sheet, follow "Formatting spreadsheet content" above.
   写入之前，如果需要查看当前工作表结构或单元格内容，优先使用 `spreadsheets.get` 或 `sheets +read`。要写入或设置工作表样式，遵循上文的"Formatting spreadsheet content"（电子表格内容的格式化）。
7. After a read or write action, confirm the user-visible result only. Do not surface raw API identifiers (spreadsheet and sheet IDs) or other internal response fields (page tokens/cursors, raw JSON) in text shown to the user unless the user asks for them or you need them to troubleshoot a failure; keep using them internally to chain follow-up commands.
   读写操作之后，只确认用户可见的结果。除非用户索要、或你需要它们排查故障，否则不要在展示给用户的文本中露出原始 API 标识符（电子表格和工作表 ID）或其他内部响应字段（分页 token/游标、原始 JSON）；在内部继续使用它们串联后续命令。

【评论】规则 4 禁止用 .xlsx 上传覆盖电子表格，属于防数据丢失条款——整文件替换会清掉其余标签页和用户编辑历史；对共享表格要求先确认，则针对多人协作场景。
