---
name: "google_contacts"
description: "Search, view, create, update, and delete the user's Google Contacts."
icon: "google_contacts"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->
# Google Contacts

Everything runs through `hatch_gws_cli people <resource> <method>` — resource and method as separate words, with parameters in a `--params` JSON object and, for creates and edits, a `--json` request body. The flows below give the exact command for each task, so start there; for a command's flags run `hatch_gws_cli people <resource> <method> --help`, and for a method's `--params`/`--json` shape run `hatch_gws_cli schema people.people.searchContacts` (and so on).

一切操作都通过 `hatch_gws_cli people <resource> <method>` 运行——resource 和 method 作为分开的词，参数放在 `--params` JSON 对象中，创建和编辑操作还需一个 `--json` 请求体。下面的流程给出了每项任务的确切命令，请从这里开始；查看某个命令的标志可运行 `hatch_gws_cli people <resource> <method> --help`，查看某个方法的 `--params`/`--json` 结构可运行 `hatch_gws_cli schema people.people.searchContacts`（依此类推）。

A contact's id is an opaque `resourceName` like `people/c1234567890`, and `people/me` is the connected account itself. Reads must name the fields they want: searches take a `readMask`, and `get`/`connections list` take a `personFields` mask (both a comma list like `names,emailAddresses,phoneNumbers`). Never invent a `resourceName` — carry the one a read returned straight through to the edit or delete.

联系人的 id 是形如 `people/c1234567890` 的不透明 `resourceName`，而 `people/me` 代表已连接账户本身。读取操作必须指明所需字段：搜索使用 `readMask`，`get`/`connections list` 使用 `personFields` 掩码（两者都是逗号分隔列表，如 `names,emailAddresses,phoneNumbers`）。绝不要凭空编造 `resourceName`——把读取返回的那个 `resourceName` 原样传递给后续的编辑或删除。

## Connecting / 建立连接
Contacts needs a one-time connect before commands return data. Run `hatch_gws_cli people status`. If it is not connected, post the exact `connect_url` it returns as `[Connect Google Contacts](<connect_url>)` and wait for the user to tap it. Don't invent a URL, send the user to Settings, or ask for credentials.

Contacts 需要先进行一次性连接，命令才会返回数据。运行 `hatch_gws_cli people status`。如果未连接，把它返回的 `connect_url` 原样以 `[Connect Google Contacts](<connect_url>)` 形式发出，并等待用户点击。不要编造 URL，不要让用户去设置页面，也不要索要凭据。

To disconnect, run `hatch_gws_cli people disconnect` and post its `disconnect_url` as `[Disconnect Google Contacts](<disconnect_url>)`.

要断开连接，运行 `hatch_gws_cli people disconnect`，并把它的 `disconnect_url` 以 `[Disconnect Google Contacts](<disconnect_url>)` 形式发出。

If a command reports an auth failure or not-connected, rerun `status` and follow the link it returns. If status is unavailable, say Google Contacts isn't available on this device and stop. Auth flows only through `status` and `disconnect`, with no hand-authored credential files or raw `gws auth`.

如果某个命令报告认证失败或未连接，重新运行 `status` 并遵循它返回的链接。如果 status 不可用，告知用户此设备上 Google Contacts 不可用并停止。认证只经由 `status` 和 `disconnect` 进行，不要手写凭据文件，也不要直接运行 `gws auth`。

## Common flows / 常用流程

### Find a contact / 查找联系人
- Look someone up by name or email: `people people searchContacts --params '{"query":"alice","readMask":"names,emailAddresses,phoneNumbers","pageSize":10}'`.
- 按姓名或邮箱查找某人：`people people searchContacts --params '{"query":"alice","readMask":"names,emailAddresses,phoneNumbers","pageSize":10}'`。
- Browse the whole address book: `people people connections list --params '{"resourceName":"people/me","personFields":"names,emailAddresses,phoneNumbers","pageSize":50}'`. Use this for "who's in my contacts" or to page through everyone.
- 浏览整个通讯录：`people people connections list --params '{"resourceName":"people/me","personFields":"names,emailAddresses,phoneNumbers","pageSize":50}'`。用于"我的联系人里有哪些人"，或分页遍历所有人。
- One person's full detail, once you have their `resourceName`: `people people get --params '{"resourceName":"people/<id>","personFields":"names,emailAddresses,phoneNumbers"}'`.
- 拿到某人的 `resourceName` 后，查看其完整详情：`people people get --params '{"resourceName":"people/<id>","personFields":"names,emailAddresses,phoneNumbers"}'`。

### Add a contact / 添加联系人
`people people createContact --json '{"names":[{"givenName":"Alice","familyName":"Smith"}],"emailAddresses":[{"value":"alice@example.com"}],"phoneNumbers":[{"value":"+15551234567"}]}'`. A name is enough; add `emailAddresses` and `phoneNumbers` when the user gives them.

`people people createContact --json '{"names":[{"givenName":"Alice","familyName":"Smith"}],"emailAddresses":[{"value":"alice@example.com"}],"phoneNumbers":[{"value":"+15551234567"}]}'`。有姓名即可；当用户提供邮箱和电话时，再加上 `emailAddresses` 和 `phoneNumbers`。

### Edit a contact / 编辑联系人
`people people updateContact --params '{"resourceName":"people/<id>","updatePersonFields":"emailAddresses"}' --json '{"etag":"<etag>","emailAddresses":[<the full list with your change>]}'`.

`people people updateContact --params '{"resourceName":"people/<id>","updatePersonFields":"emailAddresses"}' --json '{"etag":"<etag>","emailAddresses":[<the full list with your change>]}'`。

### Delete a contact / 删除联系人
`people people deleteContact --params '{"resourceName":"people/<id>"}'`.

`people people deleteContact --params '{"resourceName":"people/<id>"}'`。

## Rules / 规则
- Contact creation, editing, and deletion may proceed from a clear, unambiguous user request without an additional confirmation. Resolve the exact person before editing or deleting.
- 只要用户请求清晰、无歧义，联系人的创建、编辑和删除即可直接进行，无需额外确认。编辑或删除之前，先精确定位目标联系人。
- Read before you edit or delete: resolve the person with a search or browse first, use the exact `resourceName` (and `etag`) that read returned, and never act on a contact you did not find. For an edit, include the fields you are keeping so an update does not drop them.
- 编辑或删除之前先读取：先用搜索或浏览定位该联系人，使用读取返回的确切 `resourceName`（和 `etag`），绝不对未找到的联系人执行操作。编辑时要包含希望保留的字段，以免更新时将其丢失。
- Talk to the user in plain language only. The commands and their JSON output are for you, not the user: keep them out of your replies — no command or flag (`hatch_gws_cli`, `--params`), no status word (`not_connected`, `unavailable`), no `resourceName`, and no API field (`etag`, page tokens) or raw JSON. A contact's own name, email, and phone number are what the user asked for, so keep those in your reply.
- 只用平实的语言与用户交流。命令及其 JSON 输出是给你用的，不是给用户看的：不要让它们出现在回复中——不出现命令或标志（`hatch_gws_cli`、`--params`），不出现状态词（`not_connected`、`unavailable`），不出现 `resourceName`，也不出现 API 字段（`etag`、分页令牌）或原始 JSON。联系人本人的姓名、邮箱和电话号码才是用户要的东西，这些应保留在回复中。
- Never print tokens, secrets, or credential material. Redact them if they appear in tool output.
- 绝不打印令牌、机密或凭据材料。如果它们出现在工具输出中，应予脱敏。
- Read results add `contact_source_updated_at` with UTC and user-local forms
  when Google supplies a source update time. Birthdays remain calendar dates.
- 当 Google 提供来源更新时间时，读取结果会附带 `contact_source_updated_at`，含 UTC 与用户本地时间两种形式。生日保持为日历日期。

## Limits / 限制
- Contact groups and labels, "other contacts" (addresses auto-saved from mail, not full contacts), and directory or domain people are not first-class flows here. If a task needs one, check its shape with `hatch_gws_cli schema people.<resource>.<method>` and confirm before any change, but there is no seeded, tested recipe for it.
- 联系人分组和标签、"其他联系人"（从邮件自动保存的地址，不是完整联系人）以及目录或网域人员不属于这里的一等流程。如果任务需要用到它们，先用 `hatch_gws_cli schema people.<resource>.<method>` 检查其结构并在任何变更前进行确认，但不存在现成的、经过测试的配方。
