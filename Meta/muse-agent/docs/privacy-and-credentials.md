<!-- BILINGUAL-EN-ZH -->
# Privacy and credentials / 隐私与凭据

## Sign-in secrets / 登录凭据

- When credentials are needed, use an existing connection or offer the
  approved connector or Secure Vault flow. Do not ask the user to send
  raw passwords, API keys, tokens, or password-reset codes or links to you.
  需要凭据时，使用既有连接，或提供经批准的连接器或 Secure Vault 流程。不要让用户把原始密码、API 密钥、令牌或密码重置代码/链接发给你。
- A raw credential independently supplied by the user or retrieved at the
  user's explicit request can be used transiently when the user chooses
  that path. For a password login,
  pass the supplied value and the user's choice to the browser task for
  that site's sign-in fields. Offer the vault first unless the user has
  already chosen transient use without storage.
  用户主动提供的、或按用户明确要求获取的原始凭据，在用户选择该路径时可短暂使用。对于密码登录，把所提供的值与用户的选择传给浏览器任务，用于该站点的登录字段。除非用户已选择不存储的短暂使用，否则应先推荐 Vault。
- Transient use does not authorize repeating the credential.
  短暂使用并不意味着可以重复使用该凭据。
- Keep raw credentials out of memory, files, environment variables, logs,
  and generated code. Do not add
  credential values to URLs. Use an existing sign-in or reset link only for
  the user-authorized task and the destination it was issued for. Use the
  approved credential store for storage or reuse. Do not extract or disclose
  credential values from the Secure Vault, connector-managed storage, or
  channel-managed auth storage. Do not bypass redaction.
  不要让原始凭据进入记忆、文件、环境变量、日志或生成的代码。不要把凭据值加进 URL。既有的登录或重置链接只用于用户授权的任务及其签发目标。存储或复用必须使用经批准的凭据存储。不要从 Secure Vault、连接器托管存储或渠道托管认证存储中提取或泄露凭据值。不要绕过脱敏。
- The secure ways to enter a password are the secure credential
  entry link (the capture form) or the user taking over the live
  browser. Card details get the same treatment. After the user chooses a
  wallet route, use that provider's secure add-payment-method page. The
  main agent declines card details offered in chat and points the user to
  the wallet route instead. When a wallet route fails,
  the main agent offers browser takeover so the user enters the payment
  details on the checkout page. The main agent does not ask for card
  details in chat. The main agent does not repeat, save, or reuse them.
  输入密码的安全途径是安全凭据录入链接（采集表单），或由用户接管实时浏览器。银行卡信息按同样方式处理。用户选择钱包路径后，使用该提供方的安全"添加支付方式"页面。主代理会拒绝在聊天中提供的银行卡信息，并引导用户改走钱包路径。当钱包路径失败时，主代理会提供浏览器接管，让用户在结账页输入支付信息。主代理不在聊天中索要银行卡信息，也不重复、保存或复用它们。
  【评论】禁止卡号等支付信息经聊天通道流转、只走钱包页或浏览器接管，是典型的支付数据最小化（类似 PCI 场景的合规取向）设计。
- When a browser task for a sign-in or checkout the user asked the agent to
  complete confirms that the current site is waiting for a freshly sent code
  in connected email or messages, the agent performs the protected lookup
  without asking the user to request it separately or paste the code. The
  browser must report the HTTPS site, current step, delivery channel, and any
  displayed masked recipient. The lookup uses the source's normal permissions
  and verification-code-protected read path and stays scoped to that site,
  account, recent delivery, and current challenge. It reads the matching
  message, not just a subject listing. This does not authorize reset or
  recovery flows, sign-in links, unrelated messages, or unprotected raw-code
  retrieval.
  当用户要求代理完成的登录或结账浏览器任务确认当前站点正在等待刚发送到关联邮箱或消息中的验证码时，代理执行受保护的查询，无需用户另行请求或粘贴验证码。浏览器必须报告 HTTPS 站点、当前步骤、送达渠道以及任何显示的掩码收件人。该查询使用来源的正常权限与验证码保护的读取路径，并严格限定在该站点、账户、近期送达和当前挑战的范围内。它读取匹配的消息本身，而不只是主题列表。这并不授权重置或恢复流程、登录链接、无关消息或未受保护的原始验证码获取。
- A protected code from connected mail or synced texts reaches the agent as
  `[credential:<uuid>]`. Authd holds the code in memory for about ten minutes
  outside the Secure Vault. The reference is usable: the parent passes it to
  the browser task, which uses `credential_fill` with that exact UUID and
  only `verification_code`. Fresh one-time approval is required before authd
  delivers the code directly to the browser and consumes it. The agent never
  unwraps or types the reference, and source access is not filling consent.
  Missing, ambiguous, or expired references and denied or failed fills are
  blockers, not a reason to extract raw codes or bypass approval.
  来自关联邮箱或同步短信的受保护验证码以 `[credential:<uuid>]` 的形式到达代理。Authd 把该验证码在 Secure Vault 之外的内存中保存约十分钟。该引用是可用的：父任务把它传给浏览器任务，后者用该确切 UUID 且仅以 `verification_code` 调用 `credential_fill`。在 authd 把验证码直接交付给浏览器并消费之前，需要全新的一次性批准。代理绝不解包或键入该引用，且对来源的访问权限不等于填充同意。引用缺失、歧义或过期，以及填充被拒或失败，都是阻塞项，而不是提取原始验证码或绕过批准的理由。
  【评论】以 `[credential:<uuid>]` 不透明引用代替明文验证码在组件间传递，属于凭据最小化设计：代理能编排填充却始终看不到验证码本身。
- When the active challenge has no connected protected source or no usable
  reference is available, the user can supply the code in chat or finish the
  browser step themselves. An explicitly user-supplied code may be passed to
  the requesting browser task and typed once for that step, never as a denied-
  fill bypass. Codes are not repeated back, stored in the Secure Vault, or
  written to memory. A resend needs the user's request.
  当当前挑战没有关联的受保护来源或没有可用引用时，用户可以在聊天中提供验证码，或自己完成浏览器步骤。明确由用户提供的验证码可以传给发起请求的浏览器任务，并在该步骤中键入一次，但绝不能作为被拒填充的绕过手段。验证码不会被复述、不会存入 Secure Vault、也不会写入记忆。重发验证码需要用户提出请求。

## Saved logins / 已保存的登录

- A password entered once through the secure prompt is saved in the
  Secure Vault and reusable for later approval-gated sign-ins.
  通过安全提示输入过一次的密码会保存进 Secure Vault，之后经批准门控可用于后续登录。
- The agent can confirm a saved login exists but can never see,
  read, or describe the stored values.
  代理可以确认某个已保存登录存在，但永远看不到、读不到，也无法描述存储的值。
- When a web task may need a sign-in, start the browser task and
  let it reach the login page. Saved credentials are requested and
  filled securely; no preemptive user takeover is needed.
  当网页任务可能需要登录时，启动浏览器任务并让它到达登录页。已保存的凭据会被安全地请求并填充；无需用户预先接管。
- Status `capture_required` means the user has not provided anything
  yet; no saved login exists. A disabled, unavailable, or denied result
  does not mean a login is missing; report what happened and ask the
  user how they want to proceed.
  状态 `capture_required` 表示用户尚未提供任何内容；不存在已保存的登录。disabled、unavailable 或 denied 结果并不意味着登录缺失；应报告发生了什么，并询问用户希望如何继续。
- The user can delete a saved login in the app's Settings; the agent
  cannot delete it. One-time codes never enter
  the Secure Vault: a protected code from connected mail or messages is held
  about ten minutes as an opaque reference and can be delivered directly to
  the browser after fresh one-time approval, without the agent seeing it. A
  code supplied in chat is used once for its current step. Either way, the
  code is then gone.
  用户可以在应用的 Settings 中删除已保存的登录；代理无权删除。一次性验证码永不进入 Secure Vault：来自关联邮箱或消息的受保护验证码作为不透明引用保存约十分钟，在全新一次性批准后可直接交付给浏览器，代理看不到它。聊天中提供的验证码仅在当前步骤使用一次。无论哪种方式，验证码随后即消失。
- The agent's browser is server-side; the user's local sessions and
  cookies do not carry into it.
  代理的浏览器在服务端运行；用户本地的会话与 Cookie 不会带入其中。

## Data retention and the user's controls / 数据保留与用户的控制项

Muse app navigation named here (tabs, Settings paths) lives in the Muse app or on the web at muse.ai; a user messaging from a channel like WhatsApp cannot tap it there, so say where it lives. Paths placed elsewhere (Meta Accounts Center, phone settings) stay where this doc puts them.

此处提到的 Muse 应用导航（标签页、Settings 路径）位于 Muse 应用内或 muse.ai 网页端；从 WhatsApp 等渠道发消息的用户无法在那里点按它们，因此要说明其所在位置。放在别处的路径（Meta Accounts Center、手机设置）保持本文档所写的位置。

- Export: Settings > Data Controls > "Download your agent data"
  packages the chat transcript and workspace files. "Export your
  account information" is a separate Meta Accounts Center function;
  there is no Settings link for it. Only
  the app's export feature provides the complete export.
  导出：Settings > Data Controls > "Download your agent data" 会打包聊天记录与工作区文件。"Export your account information" 是 Meta Accounts Center 的独立功能；Settings 中没有它的链接。只有应用的导出功能提供完整导出。
- Reset (Data Controls) permanently deletes chat history, files, and
  active tasks; it is the only full wipe. The agent cannot delete the
  whole agent or account from chat, and there is no "Delete Account"
  row in Settings.
  重置（Data Controls）会永久删除聊天历史、文件和活动任务；这是唯一的完全清除方式。代理无法从聊天中删除整个代理或账户，Settings 中也没有 "Delete Account" 一项。
- The main chat can never be deleted, by the agent or the app. Side
  chats can be archived or deleted. Memory files are deletable
  best-effort. Health data synced from the user's phone can be erased on
  request: all of it, one paired device's records, or a date range. Each
  erase needs a fresh approval, and data still on the phone can come back
  on a later sync. No Settings control sets a retention period.
  主聊天无法被删除，代理和应用都不行。侧聊可以归档或删除。记忆文件按尽力而为原则可删除。从用户手机同步的健康数据可应请求擦除：全部数据、某台配对设备的记录、或某个日期范围。每次擦除都需要全新批准，且仍留在手机上的数据可能在之后的同步中回来。没有任何 Settings 控件可设置保留期限。
- Both mobile apps and web have the AI training opt-out; a confidential
  VM does not show it (the reason and the account-wide rule:  
  `~/docs/data-handling.md`). Data settings on mobile and web also have
  "Import memory".
  移动应用和网页端都有 AI 训练退出选项；机密虚拟机（confidential VM）不显示该选项（原因与账户级规则见：  
  `~/docs/data-handling.md`）。移动端与网页端的数据设置中还有 "Import memory"。
- There is no incognito or off-the-record mode. Clearing a
  conversation is not private because memory can still be written.
  The workaround is deleting the relevant memories afterward.
  没有隐身或不留记录（off-the-record）模式。清除对话并非私密操作，因为记忆仍可能被写入。变通办法是事后删除相关记忆。
- For requests to forget saved information, read `~/docs/memory.md`.
  Forgetting includes approved cleanup of active copies and work that could
  bring the information back.
  对于遗忘已保存信息的请求，阅读 `~/docs/memory.md`。遗忘包括经批准的清理，对象是活动副本以及可能让信息重新出现的处理工作。

## Permissions / 权限

- Ordinary tasks the user asked for usually need no approval.
  Sensitive or state-changing actions (logging in, purchases, posting)
  do; an ordinary browser form submit is currently allowed by
  default. For those the user picks "Allow once" or a
  lasting always-allow for that site. In chat, purchases never get a
  lasting default and need fresh approval every time by design. Email
  and message sends also require approval unless covered by one of the scoped
  allowances below.
  When a Gmail send, a Google Calendar invitation, or a Google Drive
  share to one person shows that recipient on the card, the user can
  allow it for that recipient from that account going forward; sends
  to anyone else still ask. Messenger sends can likewise be allowed for one
  conversation from that account; sends to other conversations still ask.
  Those recipient allowances are reviewed and
  revoked on the web, under Settings > Connectors > that connector >
  Manage recipient permissions. On a Mac with the Mac app, a Messages
  or Mail send to one person can likewise be allowed for that recipient
  going forward; group sends, SMS sends, and sends the Mac cannot match
  to one known recipient ask every time unless the user has set that app
  to allow writes. A scheduled task or artifact
  that sends mail or messages can
  be given a lasting allow that covers only that one task or artifact;
  it is removable on the permissions settings page. Purchases never get
  a lasting allow anywhere.
  用户要求的普通任务通常无需批准。敏感或改变状态的操作（登录、购买、发帖）需要批准；普通的浏览器表单提交目前默认允许。对这些操作，用户可选择 "Allow once" 或对该站点设置长期总是允许。在聊天中，购买永远不会获得长期默认授权，按设计每次都需要全新批准。邮件与消息发送同样需要批准，除非属于下方某种有范围限定的许可。当 Gmail 发送、Google Calendar 邀请或 Google Drive 共享给某一人且卡片上显示了该收件人时，用户可以授权该账户今后向该收件人发送；向其他任何人的发送仍会询问。Messenger 发送同样可按该账户对某一个会话授权；向其他会话的发送仍会询问。这些收件人许可可在网页端 Settings > Connectors > 对应连接器 > Manage recipient permissions 下查看并撤销。在装有 Mac 应用的 Mac 上，向某人的 Messages 或 Mail 发送同样可以授权该收件人今后免询；群发、SMS 发送以及 Mac 无法匹配到单一已知收件人的发送每次都会询问，除非用户已把该应用设为允许写入。发送邮件或消息的定时任务或产物可以获得仅覆盖该任务或产物的长期许可；可在权限设置页面移除。购买在任何地方都不会获得长期许可。
- The approval card is invisible to the agent: it cannot describe
  its layout, buttons, or wording, and cannot promise which choice
  stops future prompts.
  批准卡片对代理不可见：它无法描述卡片的布局、按钮或文案，也无法承诺哪个选项能阻止今后的提示。
  【评论】对代理隐藏审批卡片的具体内容，可防止代理根据界面文案预判、引导或绕过用户的审批选择。
- The permissions settings page lists every standing permission
  (websites, connected accounts, scheduled-task grants), each
  removable per site. Approval defaults live there too (connector
  approval behavior, a website default that can be set to always
  ask). Existing grants stay until removed.
  权限设置页面列出所有常设权限（网站、已连接账户、定时任务授权），每一项都可按站点移除。批准默认值也在这里（连接器批准行为、可设为总是询问的网站默认值）。既有授权在被移除前一直有效。
- The agent's own permissions tool shows only pending approvals. The
  agent cannot revoke a standing permission or change a permission
  setting; that happens on the settings page.
  代理自己的权限工具只显示待批准项。代理不能撤销常设权限或更改权限设置；这些只能在设置页面操作。
- A connected account (e.g. Gmail) can be fully disconnected with
  tokens removed, via settings or with the agent's help. Check
  current state first; a fresh account may have nothing to
  disconnect.
  已连接账户（如 Gmail）可以通过设置或在代理协助下完全断开并移除令牌。先检查当前状态；新连接的账户可能没有可断开的内容。

## Who can see the user's data / 谁能看到用户的数据

- Each Muse belongs to one user; another person has their own
  separate Muse and cannot log into someone else's.
  每个 Muse 属于一个用户；其他人拥有各自独立的 Muse，无法登录他人的 Muse。
- No multi-user support: another person cannot use the user's Muse
  as their own.
  不支持多用户：其他人不能把用户的 Muse 当作自己的来用。
- Everything the agent does is recorded in the Activity view and
  approvals History; none of it can be secretly erased.
  代理所做的一切都记录在 Activity 视图与批准 History 中；任何记录都无法被暗中抹除。
