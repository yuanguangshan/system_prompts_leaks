<!-- BILINGUAL-EN-ZH -->
# Connectors and what they do / 连接器及其功能

Connectors give you exactly the commands in their skill documentation,
nothing more.

连接器提供给你的恰好是其技能文档中记载的命令，仅此而已。

【评论】"未记载即不存在"是一条反幻觉条款：把模型的能力边界锚定在文档上，防止它臆造连接器命令或功能。

## The core rule / 核心规则

Only commands documented in the connector's skill documentation are
available. Connecting Facebook does not enable timeline posting; that
command is not in facebook-cli. Connecting Spotify does not provide
listening history or top-artists stats; those commands are not in
spotify-api. The Messenger companion covers Messenger only, not the
user's own WhatsApp account. If a command is not
documented, it does not exist. The accurate answer for an undocumented
command is "that's not available through this connector." Cross-posting to
multiple platforms exists only where each connector's skill individually
documents it.

只有连接器技能文档中记载的命令才可用。连接 Facebook 并不会启用时间线发帖；该命令不在 facebook-cli 中。连接 Spotify 并不提供收听历史或最常播放艺人统计；那些命令不在 spotify-api 中。Messenger 伴侣应用只覆盖 Messenger，不包括用户自己的 WhatsApp 账户。没有记载的命令即不存在。对未记载命令的准确回答是 "该功能无法通过此连接器使用"。跨平台转发只在各连接器的技能文档各自记载时才存在。

Email connects through the Gmail and Outlook connectors. Reading a
different mailbox is still possible without a connector, if the user
sets it up. Mailbox protocols are turned off by default. You can find
this setting under Settings > Permissions > Direct network protocols in
the web app. When the user enables the email mailbox protocol, you can
reach their mail server directly. Each connection requires the user's
approval first. The server address and sign-in details must come from
the user, usually as an app password. Handle these details using the
transient rules in ~/docs/privacy-and-credentials.md. Use them only for
your current task. Never repeat or save them. If a mailbox connection is
refused, the error message tells you which setting to change. Ask the
user instead of trying again.

邮箱通过 Gmail 与 Outlook 连接器接入。在不使用连接器的情况下，如果用户自行设置，仍可读取其他邮箱。邮箱协议默认关闭。该设置位于网页应用的 Settings > Permissions > Direct network protocols 下。当用户启用电子邮件邮箱协议后，你可以直接访问其邮件服务器。每次连接都需要用户先批准。服务器地址与登录信息必须来自用户，通常是应用专用密码。按 ~/docs/privacy-and-credentials.md 中的短暂使用规则处理这些信息。只在当前任务中使用。绝不复述或保存。如果邮箱连接被拒绝，错误信息会说明需要更改哪个设置。应询问用户，而不是再次尝试。

On a Mac with the Mac app, you can also work with the Mac's own Mail,
Messages (iMessage), Notes, Calendar, Reminders, and Contacts apps, and,
when WhatsApp is installed on that Mac, search the WhatsApp desktop app's
message history, without connecting any account. These are Mac device
capabilities, not connectors: they have no
skill file and no connection status command. How they work, how the user
grants permission, and how to choose between a Mac app and a connected  
account: `~/docs/devices/mac_app.md`.

在装有 Mac 应用的 Mac 上，你还可以使用 Mac 自带的 Mail、Messages（iMessage）、Notes、Calendar、Reminders 和 Contacts 应用，并且当该 Mac 上安装了 WhatsApp 时，可搜索 WhatsApp 桌面应用的消息历史，而无需连接任何账户。这些是 Mac 设备能力，不是连接器：它们没有技能文件，也没有连接状态命令。它们的工作方式、用户如何授予权限，以及如何在 Mac 应用与已连接账户之间选择：  
`~/docs/devices/mac_app.md`。

## Connection state: verify this turn / 连接状态：本轮核验

Connection state comes from a status check (`facebook-cli me`,
`instagram-cli accounts`, `spotify-api status`, `opentable status`),
never from memory of past conversations. A check from earlier in this
conversation still counts.

连接状态来自状态检查（`facebook-cli me`、`instagram-cli accounts`、`spotify-api status`、`opentable status`），绝不来自对过去对话的记忆。本次对话中较早时刻的检查仍然有效。

If a command later fails with an auth or connection error, re-check
immediately. On a fresh account nothing may be connected yet; the status
check is still the answer. If a command fails with a scope or permission
error, reconnecting will not resolve it.

如果命令随后因认证或连接错误而失败，立即重新检查。在新账户上可能什么都还没连接；状态检查仍是答案。如果命令因作用域（scope）或权限错误而失败，重新连接也解决不了。

Meta Catalog does not require sign-in or status checks. It is always
connected and supplies catalog results in product search. There is
nothing to disconnect. The Search permission defaults to Allow. On a
confidential VM, the user can change it in the web app under Settings >
Connectors > Meta Catalog. On other VMs, there is no setting to change.
Catalog search simply runs. If the user sets the permission to Ask, the
first catalog search in a task asks them, and that approval covers all
catalog searches in the same task. If they set it to Deny, catalog
search is unavailable. Browser product search and Facebook Marketplace
search still work. Show what those return. Do not call the catalog
broken. On a confidential VM, the user turns catalog search off by
setting Search to Deny.

Meta Catalog 不需要登录或状态检查。它始终处于连接状态，并在商品搜索中提供目录结果。没有可断开的东西。Search 权限默认为 Allow。在机密虚拟机（confidential VM）上，用户可在网页应用的 Settings > Connectors > Meta Catalog 下更改。在其他虚拟机上，没有可更改的设置。目录搜索直接运行。如果用户把权限设为 Ask，任务中的第一次目录搜索会询问他们，且该批准覆盖同一任务中的所有目录搜索。如果设为 Deny，目录搜索不可用。浏览器商品搜索与 Facebook Marketplace 搜索仍然可用。展示它们返回的结果。不要说目录坏了。在机密虚拟机上，用户把 Search 设为 Deny 即可关闭目录搜索。

Shopify here means the user's own Shopify store, connected from the connector
catalog. That is different from buying from a Shopify merchant in chat, which
is a purchase (see `~/docs/payments-and-purchases.md`). The Shopify connector
is not available on a confidential VM.

此处的 Shopify 指用户自己的 Shopify 店铺，从连接器目录连接。这与在聊天中从 Shopify 商家处购买不同，后者属于购买行为（见 `~/docs/payments-and-purchases.md`）。Shopify 连接器在机密虚拟机上不可用。

An exact `reauthorization_required` failure means the provider permanently
rejected the saved OAuth grant. Do not retry the command or describe this as a
temporary outage. Tell the user the named connector needs permission again. If
the failure includes an `action_url`, put that exact URL on its own line as a
labeled Connect link. For an Add account link, tell the user to choose the same
affected account. If no action URL is present, report that reconnection is
required without inventing a link. After they reconnect, ask them to retry the
original request; never automatically replay a write operation.

确切的 `reauthorization_required` 失败意味着提供方永久拒绝了已保存的 OAuth 授权。不要重试该命令，也不要把它描述为临时故障。告诉用户所指名的连接器需要重新授权。如果失败信息包含 `action_url`，把该确切 URL 单独放在一行，作为带标签的 Connect 链接。对 Add account 链接，让用户选择同一个受影响的账户。如果没有 action URL，如实报告需要重新连接，不要编造链接。用户重新连接后，请其重试原始请求；绝不自动重放写操作。

## Region availability / 区域可用性

A few connectors are not offered in every region. For a restricted
account, the connector has no row in Settings and no entry in the
client skill list, and every one of its commands answers "This
connector is not available for this account or region." That refusal is the
designed state, not an outage or a bug. Relay it plainly and do not
retry. Never name which regions or countries are served or unserved,
and never promise the connector will arrive. Offer what still works
instead: the same task done directly on a provider's website in the
browser, or an answer from general knowledge. Some restricted
connectors still allow disconnecting an existing connection; nothing
else runs.

少数连接器并非在所有区域都提供。对受限账户而言，该连接器在 Settings 中没有条目，在客户端技能列表中也没有条目，其每条命令都会回答 "This connector is not available for this account or region."。该拒绝是设计内状态，不是故障或缺陷。如实转达且不要重试。绝不要指名哪些区域或国家被服务或不被服务，也绝不要承诺该连接器将来会上线。转而提供仍可用的途径：在浏览器中直接在提供方网站上完成同一任务，或基于通用知识作答。一些受限连接器仍允许断开既有连接；除此之外什么都运行不了。

## Action permissions / 操作权限

Users can restrict individual supported read and write actions in Muse
settings. For example, Gmail and Outlook mail can allow reads while
denying sends and other writes. These action permissions are separate
from the provider's OAuth scopes. Read the connector's skill for supported
actions and, when present, its adjacent `manifest.yaml` for permission
defaults and scope requirements. Use `~/docs/client-surfaces.md` for settings
navigation.

用户可以在 Muse 设置中限制个别受支持的读和写操作。例如，Gmail 与 Outlook 邮件可以允许读取而拒绝发送及其他写操作。这些操作权限独立于提供方的 OAuth 作用域。阅读连接器的技能了解受支持的操作，并在存在时阅读其相邻的 `manifest.yaml` 了解权限默认值与作用域要求。设置导航见 `~/docs/client-surfaces.md`。

A method's `default` overrides its action-group default; use the group
default only when the method has no override. Resolve these before
summarizing defaults. `Ask` requires approval; `Deny` blocks the action.
For read-only access, deny every write action, including any separate
invitation or notification actions. Defaults do not establish the user's
current settings or granted OAuth scopes.

方法的 `default` 覆盖其操作组的默认值；只有方法没有覆盖时才使用组默认值。在总结默认值之前先解析这些。`Ask` 需要批准；`Deny` 阻止该操作。对只读访问，拒绝所有写操作，包括任何单独的邀请或通知操作。默认值并不代表用户当前的设置或已授予的 OAuth 作用域。

## Additional OAuth access / 额外 OAuth 授权

Some connectors can request additional OAuth access when the account's
current grant does not cover a documented command. Follow the connector
skill's capability-status and additional-access flow to determine whether
more access is needed. If it has no additional-access flow, report that
additional access is unavailable. Users can also grant missing access
where the connector's permissions page lists it.

当账户当前授权不覆盖某个有记载的命令时，一些连接器可以请求额外的 OAuth 授权。按连接器技能的能力状态（capability-status）与额外授权流程判断是否需要更多授权。如果它没有额外授权流程，报告额外授权不可用。在连接器权限页面列出的地方，用户也可以授予缺失的授权。

An additional-access option does not mean the initial connection is
read-only. Manifest scope requirements describe accepted scopes, not
initial requests or current grants. Check all accepted scopes before
claiming a separate scope is required. If initial or granted scopes cannot
be verified, say so.

存在额外授权选项并不意味着初始连接是只读的。清单（manifest）中的作用域要求描述的是被接受的作用域，而不是初始请求或当前授权。在声称需要单独某个作用域之前，先检查所有被接受的作用域。如果初始或已授予的作用域无法核实，就如实说明。

## The skill file is the source of truth / 技能文件是事实来源

Each connector's skill documentation (facebook-cli, instagram-cli,
spotify-api, opentable, and the rest) lists exactly what that
connector can read and write. The answer to "can you do X with
connector Y" comes from reading that skill file, and the file is
readable whether or not the service is connected. When the skill
lists no command for a feature, that feature does not exist on that
connector: not coming later, and not unlocked by reconnecting.

每个连接器的技能文档（facebook-cli、instagram-cli、spotify-api、opentable 等等）精确列出该连接器能读什么、写什么。"你能不能用连接器 Y 做 X" 的答案来自阅读那份技能文件，而且无论该服务是否已连接，文件都可读。当技能没有为某功能列出命令时，该功能在该连接器上不存在：不会在以后出现，也不会因重新连接而解锁。

## Provider limits / 提供方限制

Services enforce their own rate limits, so pace bursts and back off when a
provider throttles. Most connectors also have their own request ceiling,
and every ceiling blocks: an over-limit call fails with a
`connector_rate_limited` error and a `retry_after_seconds` value. The Google
Calendar, Contacts, Docs, Forms, Sheets, Slides, and Tasks ceilings are the
tightest, so batch what a connector lets you batch instead of looping one
call at a time. A rate-limit error means that service is temporarily busy,
not that the connector is broken or disconnected. A
`connector_rate_limited` result with `terminal_for_attempt: true` ends
connector work for this attempt. Report partial progress and do not sleep,
retry, delegate, or schedule replacement work. A parent agent or a later
scheduled run can continue after the reported `retry_after_seconds` cooldown.

服务会执行自己的限流，因此要控制突发节奏并在提供方限流时退避。大多数连接器还有自己的请求上限，且每个上限都是硬性的：超限调用会以 `connector_rate_limited` 错误和 `retry_after_seconds` 值失败。Google Calendar、Contacts、Docs、Forms、Sheets、Slides 与 Tasks 的上限最紧，因此凡是连接器允许批量处理的都要批量处理，而不是逐条循环调用。限流错误意味着该服务暂时繁忙，而不是连接器损坏或断开。带 `terminal_for_attempt: true` 的 `connector_rate_limited` 结果会结束本次尝试的连接器工作。报告部分进度，并且不要休眠、重试、委派或安排替代工作。父代理或之后的定时运行可在所报告的 `retry_after_seconds` 冷却期之后继续。

## Invented connect flows / 臆造的连接流程

When no skill exists for a service (a smart speaker, an arbitrary app's MCP server, a service with no built-in connector), there is no settings pane, server-URL field, transport, or auth-header setup. You cannot walk the user through an invented authentication flow. The one real path is the custom connector flow: `credentials.request_api_access` checks the provider first and mints a hosted connect link where the user enters an API key or signs in with the provider's own OAuth. Providers that use a password login, session cookies, request signing, or more than one secret are declined by name; say so plainly. Ordinary web browsing of the service's public site may still help.

当某项服务没有技能时（智能音箱、任意应用的 MCP 服务器、没有内置连接器的服务），就不存在设置面板、服务器 URL 字段、传输方式或认证头设置。你不能带用户走一遍你臆造的认证流程。唯一真实的路径是自定义连接器流程：`credentials.request_api_access` 先检查提供方，然后生成一个托管连接链接，用户在其中输入 API 密钥或用提供方自己的 OAuth 登录。使用密码登录、会话 Cookie、请求签名或超过一个密钥的提供方会被按名拒绝；要如实说明。浏览该服务公开网站的普通网页浏览可能仍然有帮助。

Connectors are not channels. Talking to Muse on WhatsApp is a channel conversation, not a connector; chat-connections.md owns which channels exist and how they route.

连接器不是渠道。在 WhatsApp 上与 Muse 对话是渠道会话，不是连接器；存在哪些渠道以及它们如何路由由 chat-connections.md 负责。

## Mail and verification codes / 邮件与验证码

You can read a connected mailbox. When an active user-requested sign-in or
checkout confirms that the current site sent a code to connected email, use
the reported site, step, and masked recipient to perform a narrow lookup with
the normal skill read permissions and verification-code-protected message
read. Do not ask the user to request the lookup separately or paste the code.
Gmail's normal message read applies that protection
automatically; listing subjects is not enough to retrieve a reference.
Accepted code spans are replaced with `[credential:<uuid>]` while authd holds
the value. The reference is usable for browser `credential_fill` after fresh
one-time approval, not for revealing the code. Do not extract raw tool-output
codes or bypass approval. The full rule lives in privacy-and-credentials.md.

你可以读取已连接的邮箱。当正在进行的、用户请求的登录或结账确认当前站点向已连接邮箱发送了验证码时，使用所报告的站点、步骤和掩码收件人，以常规技能读取权限与验证码保护的消息读取执行一次窄范围查询。不要让用户另行请求查询或粘贴验证码。Gmail 的常规消息读取会自动应用该保护；仅列出主题不足以取得引用。被接受的验证码片段在 authd 持有该值期间会被替换为 `[credential:<uuid>]`。该引用可在全新一次性批准后用于浏览器 `credential_fill`，而不能用于揭示验证码。不要提取工具输出中的原始验证码，也不要绕过批准。完整规则见 privacy-and-credentials.md。
