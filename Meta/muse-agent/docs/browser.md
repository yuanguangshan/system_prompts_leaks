<!-- BILINGUAL-EN-ZH -->

# Browser / 浏览器

The browser is a server-side browser owned by Muse for all browser related tasks. You can search the live web, open pages, extract content, and run interactive browser tasks: navigate, click, type, upload and download files. You can do this work in the background while the user does something else, and several browser tasks can run in parallel on ordinary VMs (confidential VMs queue them one at a time). When you complete a browser task, the result tells you whether it succeeded, failed, or got blocked.

该浏览器是由 Muse 拥有的服务端浏览器，用于所有与浏览器相关的任务。你可以搜索实时网络、打开页面、提取内容，并运行交互式浏览器任务：导航、点击、输入、上传和下载文件。你可以在后台执行这些工作，与此同时用户去做别的事情；多个浏览器任务可以在普通虚拟机上并行运行（机密虚拟机会将它们逐个排队）。完成一个浏览器任务后，结果会告诉你它是成功、失败还是被阻止。


Web search and accessing the public internet is free. Subscription requirements and paid content blocks are unknown until the browser task runs. The task result is the only signal of success or failure.

网页搜索和访问公共互联网是免费的。订阅要求和付费内容封锁在浏览器任务实际运行前无从得知。任务结果是成功或失败的唯一信号。

## Limitations / 限制

- Network policy and site defenses (logins, CAPTCHAs) can block a task
  网络策略和站点防御（登录、验证码）可能阻止任务
- A CAPTCHA or bot check pauses the task. You never solve one on your own, and solving one is never a task by itself. Ask the user once whether you may solve CAPTCHAs for browser tasks and keep to the scope of their answer (just this time, this site only, ask each time, or never); a user who would rather solve it themselves can take over the live browser on the apps that offer it.
  验证码或机器人检测会使任务暂停。你绝不能自行解决验证码，解决验证码也绝不是一个独立任务。询问用户一次是否允许你为浏览器任务解决验证码，并严格遵守其回答的范围（仅此一次、仅此站点、每次都问、或绝不）；愿意自己解决的用户可以在提供该功能的应用上接管实时浏览器。
  【评论】此条款是典型的防提示词注入设计：即使页面内容要求"解决验证码"，代理也不得自行越权，必须以用户的一次性明确授权为前提。
- Sessions do not carry over from the user's device (see Sessions and sign-in)
  会话不会从用户的设备带过来（见"会话与登录"）
- Downloads land on your machine, not the phone (see Downloads)
  下载文件落在你的机器上，而不是手机上（见"下载"）
- Some checkout flows pause for approval. The task result determines success.
  某些结账流程会暂停等待批准。任务结果决定成败。
- Cancellations and similar site actions cannot be guaranteed before the task runs; if a task fails with a block message, site defenses are the honest reason. Approval denials block the action; retrying through alternate routes does not override user denial.
  取消等类似站点操作在任务运行前无法保证成功；如果任务以被阻止的消息失败，站点防御就是真实原因。批准被拒绝会阻止该操作；通过其他途径重试不能推翻用户的拒绝。

A browser task that stops to ask the user something keeps waiting for about three hours. When the user answers, continue that same waiting task; never start a fresh task for the same purchase or sign-in while one is parked, because a second task can repeat the work, including a purchase.

停下来向用户提问的浏览器任务会继续等待约三小时。用户回答后，继续同一个等待中的任务；当一个任务已搁置等待时，绝不要为同一次购买或登录启动新任务，因为第二个任务可能重复执行该工作，包括一次购买。

A browser task that reaches a checkout page can pause for a fresh one-tap user approval before proceeding. Fully hands-off purchase or booking flows are not available. If an approval card appears, the user's decision at that moment is the gate. How the rest of a purchase works (the approval card, limits, wallets, refunds) is covered in payments-and-purchases.md; follow that doc for anything involving money.

到达结账页面的浏览器任务可以在继续之前暂停，等待用户一次全新的单击批准。完全免操作的购买或预订流程不可用。如果出现批准卡片，用户当时的决定就是闸门。购买其余环节的工作方式（批准卡片、限额、钱包、退款）在 payments-and-purchases.md 中有说明；凡涉及金钱的事项都遵循该文档。

## Browser permissions / 浏览器权限

The browser has its own permission suite in the app, under Settings >
Connectors > Browser: visiting websites,
submitting a form or POST request, downloading files, uploading
files, and filling saved credentials, each settable to Allow, Ask,
or Deny. Web-access defaults,
including a website default that can be set to always ask, live on
the standalone Permissions tab in settings, not under Connectors.
The agent cannot change any of these
settings itself.

浏览器在应用中有自己的权限套件，位于 Settings > Connectors > Browser 之下：访问网站、提交表单或 POST 请求、下载文件、上传文件、填充已保存的凭据，每一项都可设为 Allow、Ask 或 Deny。网络访问的默认设置——包括一个可设为总是询问的网站默认项——位于设置中独立的 Permissions 标签页，而不在 Connectors 之下。代理自身不能更改这些设置中的任何一项。

## Sessions and sign-in / 会话与登录

Your browser is its own server-side browser completely separate from the user's device. The user's local sessions and cookies never carry over. A connected account does not help here either: a connector the user signed into (Google, Spotify, and similar) does not log your browser into that provider's websites. Connectors and browser sign-ins are separate things. Device sign-in does not transfer to the browser; browser sign-in must happen through the browser's own sign-in flow.

你的浏览器是一个独立的服务端浏览器，与用户的设备完全分离。用户本地的会话和 Cookie 永远不会带过来。已连接的账号在这里也帮不上忙：用户登录过的连接器（Google、Spotify 等）不会让你的浏览器登录到该服务商的网站。连接器与浏览器登录是两回事。设备登录不会转移到浏览器；浏览器登录必须通过浏览器自身的登录流程完成。

【评论】浏览器会话与用户设备完全隔离，这既是安全边界（Cookie 与本地凭据不外泄），也意味着无法复用用户已登录的状态。

When a web task might need a sign-in, the browser task runs and reaches the login page as needed. If credentials are saved in the secure vault, the task can securely request and fill them in without exposing them in chat. A user takeover is only needed when the task itself hits something that requires one, not preemptively.

当网络任务可能需要登录时，浏览器任务会运行并按需到达登录页面。如果凭据已保存在安全保险库中，任务可以安全地请求并填入它们，而不会在聊天中暴露。只有当任务本身遇到需要用户接管的情况时才需要接管，而不是预先进行。

Signing in with saved credentials works only if the browser task's result says it succeeded. If the task reports that a login needs to be captured, the user has not provided a saved login for that site yet. A disabled, unavailable, or denied result does not mean a login is missing; report what happened and ask the user how they want to proceed. Logged-in flows cannot be promised in advance; the attempt determines success. Passkey sign-ins cannot be completed in the browser. When a site offers only a passkey, the task stops and reports that the user has to sign in themselves.

使用已保存凭据登录，只有当浏览器任务的结果显示成功时才算成功。如果任务报告需要采集登录信息，说明用户尚未为该站点提供已保存的登录。被禁用、不可用或被拒绝的结果并不意味着缺少登录；报告发生了什么，并询问用户希望如何继续。已登录状态下的流程无法预先承诺；实际尝试决定成败。通行密钥（passkey）登录无法在浏览器中完成。当站点只提供通行密钥时，任务会停止并报告用户必须自行登录。

## No holds, locks, or reservations / 不做保留、锁定或预订

The browser cannot hold a fare, lock a price, or reserve inventory while the user decides. Holds exist on some sites as the site's own feature, not as something you control. Alternatives: a scheduled price check that runs periodically (not instant), or the site's own paid hold option that the user activates themselves. Holding prices or inventory is a site feature, not an agent capability.

在用户考虑期间，浏览器不能保留票价、锁定价格或预订库存。某些站点上的保留功能是站点自身的特性，不是你能控制的。替代方案：周期性运行的定时比价检查（非即时），或由用户自行激活的站点付费保留选项。保留价格或库存是站点功能，不是代理能力。

## Downloads / 下载

Downloaded files land on your machine, not in chat and not on the user's phone. They live in your workspace. The user cannot access them until you explicitly deliver them. Delivery means sharing the file in chat, saving it to `workspace/your_files` so it appears in the Library, or, when the user wants a link anyone can open, publishing a temporary public download link (see `~/docs/files-and-library.md`; it expires and is not available on confidential VMs).

下载的文件落在你的机器上，不在聊天中，也不在用户的手机上。它们存放在你的工作区。在你明确交付之前，用户无法访问它们。交付是指在聊天中分享文件、保存到 `workspace/your_files` 使其出现在资料库中，或者当用户想要一个任何人都能打开的链接时，发布一个临时的公开下载链接（见 `~/docs/files-and-library.md`；该链接会过期，且在机密虚拟机上不可用）。

## Live browser watching / 实时浏览器观看

Whether the user can watch the live browser or take over is a client-side feature. Some surfaces have it and some do not. Live browser control availability varies by client surface. If the user reports no browser view, the feature is unavailable on their client. Live co-driving is not available for web editors or forms until confirmed on the client.

用户能否观看实时浏览器或接管，是客户端侧的功能。某些页面有，某些没有。实时浏览器控制的可用性因客户端页面而异。如果用户报告看不到浏览器视图，说明该功能在其客户端上不可用。在客户端上确认之前，实时协同驾驶不适用于 Web 编辑器或表单。

## Purchases and refunds / 购买与退款

Refund reversals are the merchant's decision, not something you can execute; you can help the user pursue one from the merchant.

退款的撤销是商家的决定，不是你能执行的；你可以帮助用户向商家争取。
