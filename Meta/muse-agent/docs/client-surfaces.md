<!-- BILINGUAL-EN-ZH -->
# Where users access Muse / 用户访问 Muse 的渠道

Users reach Muse on iOS, Android, web, or the Mac app. This page lists shared
functionality on iOS, Android, and web, then platform-specific differences.
Muse app navigation named here (tabs, Settings paths) lives in the named Muse
app or on the web at muse.ai; a user messaging from a channel like WhatsApp
cannot tap it there, so say where it lives.

用户通过 iOS、Android、网页端或 Mac 应用访问 Muse。本页先列出 iOS、Android 和网页端的共通功能，再说明各平台的差异。此处提到的 Muse 应用导航（标签页、设置路径）位于所指的 Muse 应用内或 muse.ai 网页端；从 WhatsApp 这类渠道发消息来的用户在那里无法点按这些入口，因此要说明其所在位置。

Muse went live on September 8, 2026. It is available in the US and Canada.
The iOS app is on the App Store and the Android app is on Google Play Store.
Details in `~/docs/muse.md`.

Muse 于 2026 年 9 月 8 日上线，目前在美国和加拿大可用。iOS 应用上架 App Store，Android 应用上架 Google Play 商店。详情见 `~/docs/muse.md`。

## Mac app / Mac 应用

The Mac app is a chat surface. It also pairs the user's Mac as a device and
advertises the capabilities available on that Mac.

Mac 应用是一个聊天界面。它还会把用户的 Mac 配对为一台设备，并声明该 Mac 上可用的能力。

Before answering a question about the Mac app or operating the Mac, read  
`~/docs/devices/mac_app.md`.

在回答有关 Mac 应用的问题或操作 Mac 之前，先阅读  
`~/docs/devices/mac_app.md`。

The Mac app shows the web app inside a Mac window. Its tabs, chat, Library,
Settings, and approval cards are the web app's, with these Mac-only pieces on  
top:

Mac 应用在 Mac 窗口内呈现网页应用。其标签页、聊天、资料库、设置和审批卡片都是网页应用的，在此之上叠加了以下 Mac 独有的部分：

- A menu bar icon with Open Muse, Quick chat, Voice dictation, Check for
  updates… (Install update… once one is downloaded), and Quit. It has no
  Settings or Log out row. Closing the Muse window does not quit the app; a
  small floating button stays on screen and reopens it. Settings opens with
  Cmd-comma. Log out is the last item in the Settings list, and the Muse menu
  in the Mac menu bar also has a Log out item, just above Quit Muse.
  菜单栏图标，包含 Open Muse、Quick chat、Voice dictation、Check for updates…（一旦下载了更新则变为 Install update…）和 Quit。它没有 Settings 或 Log out 行。关闭 Muse 窗口不会退出应用；屏幕上会保留一个小浮动按钮，可用来重新打开。Settings 用 Cmd-comma 打开。Log out 是 Settings 列表的最后一项，Mac 菜单栏中的 Muse 菜单也有一个 Log out 项，位于 Quit Muse 之上。
- Quick chat: Option-Spacebar opens a small chat card over whatever app is in
  front; pressing it again closes the card. Files can be dropped into it or
  onto the floating button. The key is changed under Settings > General >
  Shortcuts > Quick Chat.
  Quick chat：Option-Spacebar 会在当前最前方的应用之上打开一个小聊天卡片；再按一次即关闭卡片。文件可以拖放进卡片，或拖放到浮动按钮上。快捷键可在 Settings > General > Shortcuts > Quick Chat 下更改。
- Dictation anywhere: holding the fn key dictates into the app in front, not
  only into Muse. Dictating into other apps needs the Microphone and
  Accessibility permissions the app asks for at setup. The keys are changed
  under Settings > Dictation (Push to talk to hold, Hands-free mode to tap).
  随处听写：按住 fn 键即可向当前最前方的应用（而不只是 Muse）听写。向其他应用听写需要应用在设置阶段请求的麦克风与辅助功能权限。快捷键可在 Settings > Dictation 下更改（Push to talk 为按住说话，Hands-free mode 为点按说话）。
- Settings tabs the other surfaces do not have: File system access (Full
  Disk Access; Read only, Read and interact, or Off for Mail.app,
  Messages.app, Notes.app, and WhatsApp.app; a Blocked folders list) and
  Dictation. General adds Run on startup, Show in menu bar, Show floating
  button, and an About row with Check for updates. Permissions lists the
  Mac's own apps under "On this Mac". When voice is on for the account,
  Voice controls is a row inside Dictation, not its own tab. When the Mac
  app offers computer use, a Computer use tab shows the Accessibility and
  Screen Recording status, a per-command policy choice, Keep screen awake,
  and a Blocked apps list.
  其他端没有的 Settings 标签页：File system access（完全磁盘访问；针对 Mail.app、Messages.app、Notes.app 和 WhatsApp.app 可设为 Read only、Read and interact 或 Off；含 Blocked folders 列表）和 Dictation。General 增加了 Run on startup、Show in menu bar、Show floating button，以及带 Check for updates 的 About 行。Permissions 在 "On this Mac" 下列出 Mac 自带应用。当账户开启语音时，Voice controls 是 Dictation 内的一行，而不是独立标签页。当 Mac 应用提供计算机使用能力时，会出现一个 Computer use 标签页，显示辅助功能与屏幕录制状态、按命令的策略选择、Keep screen awake，以及 Blocked apps 列表。
- The app updates itself: Check for updates… is in the menu bar icon, the
  Muse menu, and Settings > General. There is no app store listing.
  应用自行更新：Check for updates… 位于菜单栏图标、Muse 菜单和 Settings > General 中。没有应用商店上架条目。

The Mac app gets no push notifications. It can notify the user only while it
is running.

Mac 应用不接收推送通知。只有在运行时才能通知用户。

## iOS, Android, and web / iOS、Android 与网页端

### Tabs (variations per platform below) / 标签页（各平台差异见下文）

Every surface has Chat, Feed, Ideas, Goals, and Library. Chat search exists on every surface (see Chat search below). Web also answers a direct /artifacts URL.

每个端都有 Chat、Feed、Ideas、Goals 和 Library。每个端都有聊天搜索（见下文 Chat search）。网页端还能响应直接的 /artifacts URL。

- **Feed:** short posts you write for the user on a schedule,
  guided by the feed prompt. A brand-new reader's feed opens with a fixed
  set of intro posts shipped with the build instead (kicker and category
  `Getting started`), in your voice but drawing on no source of theirs.
  See `~/docs/feed.md`.
  **Feed:** 你按计划为用户撰写的短帖，由 feed 提示词引导。全新读者的 feed 会先展示随构建内置的一组固定介绍帖（kicker 与分类为 `Getting started`），以你的口吻撰写，但不引用该用户的任何素材。见 `~/docs/feed.md`。
- **Ideas:** suggested ideas curated by your agent for you.
  **Ideas:** 你的智能体为你精选的建议想法。
- **Goals:** list of active or completed goals.
  **Goals:** 进行中或已完成目标的列表。
- **Library:** where Artifacts, documents, media, and files live.
  **Library:** Artifacts、文档、媒体和文件所在之处。
- **Artifacts:** the apps and pages you build for the user. What they are and how publishing works: `~/docs/artifacts.md`.
  **Artifacts:** 你为用户构建的应用和页面。它们是什么以及发布机制如何：`~/docs/artifacts.md`。

On the mobile apps an Invite button sits at the top right of the chat
header (a gift icon on iOS). It opens the invite share sheet.
On iPhone and Android, **Settings > Redeem token** accepts a friend's invite
code during the first 48 hours after the user joins Muse (standard VMs only;
confidential VMs have no redemption entry).
Invite links, code redemption, and offer terms: `~/docs/referrals.md`.

移动应用的聊天页眉右上角有一个 Invite 按钮（iOS 上是礼物图标），点击会打开邀请分享面板。在 iPhone 和 Android 上，**Settings > Redeem token** 会在用户加入 Muse 后的最初 48 小时内接受朋友的邀请码（仅限标准 VM；机密 VM 没有兑换入口）。邀请链接、兑换码与优惠条款：`~/docs/referrals.md`。

### Side chats / 侧边聊天

Side chats organize conversations by topic; every conversation the user has
in the app, main or side, is between you and the user only. The Muse app has
no group chat, no way to add other people to a chat, and no "Add people"
control. The Invite button shares a Muse referral, not access to a chat.
Another person's account has its own separate agent and cannot see or join
this user's chats.

侧边聊天按主题组织对话；用户在应用内进行的每一个对话，无论主聊还是侧聊，都只发生在你和用户之间。Muse 应用没有群聊，无法把其他人加入聊天，也没有 "Add people" 控件。Invite 按钮分享的是 Muse 的推荐邀请，而不是某个聊天的访问权。他人的账户有各自独立的智能体，无法查看或加入该用户的聊天。

【评论】该段以绝对化表述（无群聊、无加人入口）约束模型对聊天隐私边界的回答，属于典型的隐私承诺条款；对 Invite 按钮的澄清是为了防止模型把推荐机制误述为拉人入聊。

### Gestures on mobile / 移动端手势

Both phone apps share the same gesture set, sometimes via different
mechanisms: long-press message menus, swipe a message to reply,
swipe feedback on idea
cards, pinch/double-tap/pan-to-dismiss in the media viewer,
shake the phone to report a bug, screenshot annotation, and touch
control when taking over the live VM browser.

两款手机应用共用同一套手势，实现机制有时不同：长按消息菜单、滑动消息进行回复、在想法卡片上滑动反馈、媒体查看器中的捏合/双击/拖动关闭、摇晃手机报告 bug、截图标注，以及接管实时 VM 浏览器时的触控操作。

The chat-bubble long-press menu is a reaction tray plus a set of
rows that each appear only when they apply: Reply, Copy, Select,
Share, View sources (when the message has citations), Save to Photos
(image messages, iOS), and Delete (only on the user's own messages). Go by what the user sees.

聊天气泡长按菜单是一个回应托盘，外加一组各自仅在适用时出现的行：Reply、Copy、Select、Share、View sources（消息带引用时）、Save to Photos（图片消息，iOS）和 Delete（仅限用户自己的消息）。以用户实际所见为准。

Platform exclusives: on iOS, pinned Library artifacts can be
reordered by long-press drag (Android has pinning but no reorder
control). Android-only: swiping a workspace file or folder row into
the composer to discuss it in chat; dragging goals in the Goals tab
to reorder them, nest one under another, or lift a subgoal back out
(the iPhone app has no goal drag gestures, and reordering or nesting
goals is not something you can do for the user either; on web, the
user's own goals can also be dragged into a manual order while the Goals
page's Sort automatically option is off, which is the default); and
double-tapping an agent message to toggle a heart reaction (on iOS,
reactions happen through the long-press tray).

平台独有功能：iOS 上，置顶的资料库 artifact 可长按拖动重新排序（Android 支持置顶但没有重排控件）。Android 独有：把工作区文件或文件夹行滑入输入框以便在聊天中讨论；在 Goals 标签页拖动目标以重新排序、把一个目标嵌套到另一个之下，或把子目标移出（iPhone 应用没有目标拖动手势，重排或嵌套目标也不是你能替用户完成的操作；在网页端，当 Goals 页面的 Sort automatically 选项关闭（默认即关闭）时，用户也可以把目标拖成手动顺序）；以及双击智能体消息切换心形回应（iOS 上回应通过长按托盘完成）。

Saving media: on web, right-click an image or video in chat for Download
(and Delete on eligible messages); on the mobile apps, the full-screen
viewer has a Save button and a menu with Reply, Copy, Share, and Delete.
Deleting from the viewer removes that media from the chat.

保存媒体：网页端在聊天中右键点击图片或视频可 Download（符合条件的消息还可 Delete）；移动应用的全屏查看器有 Save 按钮和一个含 Reply、Copy、Share、Delete 的菜单。从查看器中删除会把该媒体从聊天中移除。

### Deleting a sent message / 删除已发送的消息

Users can delete their own sent messages from the message long-press menu on
both mobile apps; the control is labeled Delete. On the
mobile apps a one-time notice explains how delete works and there is no
per-delete confirmation; on web, Delete shows a confirmation dialog on the
user's own messages. A deleted message means the user removed it on purpose.

在两款移动应用上，用户都可以从消息长按菜单删除自己发送的消息；该控件的标签是 Delete。移动应用会有一条一次性说明解释删除的工作方式，且每次删除无需确认；网页端的 Delete 在用户自己的消息上会弹出确认对话框。已删除的消息意味着用户是有意移除的。

### Chat search / 聊天搜索

Where it lives: the rail's Search entry on web (Cmd+K opens the same
quick-search palette), the header
and side-panel search on Android (also the `hatch://search` deep link),
and thread search on iOS.

入口位置：网页端是侧栏的 Search 入口（Cmd+K 打开同一个快速搜索面板）；Android 是页眉和侧面板搜索（还有 `hatch://search` 深链）；iOS 是会话搜索。

How it works: the server keeps a full-text index of every visible
message, yours and Muse's, across the main chat and all side chats,
including archived ones and channel-linked ones. Matching is by whole
word and case-insensitive, and the last word typed also matches as a
prefix, so results appear as you type. Results come newest first with
a short snippet and the chat each hit came from; hidden system and
background messages are never in the index, and deleting a message or
chat removes it from search at the same moment. The chat-list picker
search is separate and also matches chat titles.

工作方式：服务器对每条可见消息（你的和 Muse 的）维护全文索引，覆盖主聊天和所有侧边聊天，包括已归档和已关联渠道的聊天。匹配按整词且不区分大小写，最后输入的词还按前缀匹配，因此结果会随输入即时出现。结果按最新优先排列，附简短摘要及每次命中所在的聊天；隐藏的系统消息和后台消息绝不进入索引，删除消息或聊天会在同一时刻将其从搜索中移除。聊天列表选择器的搜索是独立的，也会匹配聊天标题。

You have no cross-chat search tool of your own: your path is
`chat.list`, then reading one chat's transcript at a time with
`chat.read_messages`. `memory_search` searches memory, not chat
history.

你没有自己的跨聊天搜索工具：你的路径是先用 `chat.list`，再用 `chat.read_messages` 逐个读取聊天记录。`memory_search` 搜索的是记忆，不是聊天历史。

### Status screen (tap your avatar) / 状态页（点按你的头像）

Has Activity, Approvals, Upcoming, and Identity on all surfaces. There is no Skills tab anywhere; connected skills live under Settings > Connectors. On Android the panel opens on Approvals when one is waiting, otherwise Activity. On iOS and web, tapping the avatar always opens Activity; Approvals opens when the user comes in through an approval prompt itself.

所有端都有 Activity、Approvals、Upcoming 和 Identity。任何地方都没有 Skills 标签页；已连接的技能位于 Settings > Connectors 下。在 Android 上，有待处理的审批时面板直接打开 Approvals，否则打开 Activity。在 iOS 和网页端，点按头像总是打开 Activity；用户通过审批提示本身进入时才打开 Approvals。

- **Approvals:** pending permission requests.
  **Approvals:** 待处理的权限请求。
- **Activity:** a log of your recent actions. On web, a running item shows a Stop control on hover that cancels that work.
  **Activity:** 你近期操作的日志。网页端上，进行中的条目在悬停时显示 Stop 控件，可取消该项工作。
- **Upcoming:** the user's view of your schedules. The user can remove a task from its detail view (system-owned tasks refuse deletion); iOS, Android, and web also show its run history. On web and iOS, an Edit action drafts a chat message and the user sends it to make the change; the Android app has no edit control, so changes there happen by asking you in chat.
  **Upcoming:** 用户视角下你的计划任务。用户可以在任务详情视图中移除任务（系统拥有的任务拒绝删除）；iOS、Android 和网页端还会显示其运行历史。在网页端和 iOS 上，Edit 操作会起草一条聊天消息，由用户发送来完成修改；Android 应用没有编辑控件，因此那边的修改要通过在聊天中向你提出。
- **Identity:** avatar and name. Editing the avatar's name renames you. A gated Share avatar action may also appear in the Edit menu. No separate second name, and no theme controls here (theme lives in Settings > Appearance).
  **Identity:** 头像和名字。编辑头像的名字即为给你改名。Edit 菜单中还可能出现一个受门控的 Share avatar 操作。没有单独的第二个名字，这里也没有主题控件（主题位于 Settings > Appearance）。

### Approval requests from the user's agent / 来自用户智能体的审批请求

When you ask the user for permission to do something, that approval request can reach them as a push notification. On Android and iOS the notification shows Allow and Deny buttons directly in the notification shade, so the user can answer your request without opening the app first. On iOS, acting on them requires unlocking and foregrounding the app; Android decides in the background, and on Android swiping the notification away counts as Deny, so a denial may just mean the notification was dismissed; it is fine to ask once whether they meant to decline. This is about your approval requests only, not system or app notifications in general.

当你请求用户许可执行某项操作时，该审批请求可以以推送通知的形式送达。在 Android 和 iOS 上，通知会在通知栏中直接显示 Allow 和 Deny 按钮，用户无需先打开应用即可回应你的请求。在 iOS 上，实际操作需要解锁并把应用切到前台；Android 可在后台决定，且在 Android 上划掉通知视为 Deny，因此一次拒绝可能只是通知被划掉了；可以问一次用户是否真的想拒绝。这只关乎你的审批请求，不涉及一般性的系统或应用通知。

【评论】"划掉通知视为拒绝"的说明是在提醒模型：通知手势属于模糊信号，未必是明确的拒绝意图，因此建议复核一次而非直接采信。

### Settings / 设置

How it opens: on iOS behind the gear icon, on Android from the gear in the
chat side panel's header (the panel opens from the tab bar or by
dragging from the edge); on web it is a dialog from the menu icon at
the bottom of the left rail (see the web section).

打开方式：iOS 在齿轮图标之后；Android 在聊天侧面板页眉的齿轮中（该面板从标签栏打开或从边缘拖出）；网页端是从左栏底部菜单图标打开的对话框（见网页端章节）。

Typically present: Messaging channels (which ones appear depends on the
account: `~/docs/chat-connections.md`; on confidential VMs this section
is absent on every platform), Connectors, Help/Legal, Log out, and a data
pane with the agent-data download (labeled "Download your agent data" on
every platform). "Export your information" is inside
Meta Accounts Center. iOS, Android, and web
all have a standalone Permissions pane in Settings. The settings rows listed for each platform in
the sections below are the complete set this doc can vouch for.

通常包含：Messaging channels（出现哪些取决于账户：`~/docs/chat-connections.md`；机密 VM 上所有平台都没有此板块）、Connectors、Help/Legal、Log out，以及带智能体数据下载的数据面板（各平台均标记为 "Download your agent data"）。"Export your information" 位于 Meta Accounts Center 内。iOS、Android 和网页端的 Settings 中都有独立的 Permissions 面板。下文各平台小节列出的设置行是本文档能够担保的完整集合。

AI training opt-out is available on every surface, in that data pane. Confidential VMs do not show it (the reason and the account-wide rule: `~/docs/data-handling.md`). On a confidential VM, Settings also has no subscription or usage area and no invite-code redemption entry, on any platform.

AI 训练退出选项在每个端的该数据面板中都可用。机密 VM 不显示它（原因及账户级规则：`~/docs/data-handling.md`）。在机密 VM 上，任何平台的 Settings 都没有订阅或用量区域，也没有邀请码兑换入口。

【评论】机密 VM 在功能上有系统性限制（训练退出、订阅、兑换等均不可用），文档反复强调这一点，是为了避免模型向使用不同 VM 类型的用户作出不成立的承诺。

Reset lives under Data Controls. Delete account is not a row anywhere; Reset wipes the agent's data, and deleting the account itself happens through Meta Accounts Center.

Reset 位于 Data Controls 下。任何地方都没有 Delete account 行；Reset 清除的是智能体的数据，删除账户本身则要通过 Meta Accounts Center 进行。

Publishing and sharing approval rules for artifacts live in `~/docs/artifacts.md`.

artifact 的发布与共享审批规则见 `~/docs/artifacts.md`。

Users can also share INTO Muse from other apps: the iOS share extension and
the Android share target accept links, images, text, and files, and stage
them in the composer for the user to send. Nothing is sent automatically.

用户还可以从其他应用分享进 Muse：iOS 分享扩展和 Android 分享目标接受链接、图片、文本和文件，并把它们放入输入框暂存，由用户手动发送。不会自动发送任何内容。

### Import memory / 导入记忆

For Settings > Data controls > Import memory, read `~/docs/memory.md`.
It covers importing context from another AI assistant and chat exports.

关于 Settings > Data controls > Import memory，请阅读 `~/docs/memory.md`。它涵盖从其他 AI 助手导入上下文以及聊天导出。

---

## iOS app only / 仅 iOS 应用

### Tabs / 标签页

Chat, Feed, Ideas, Goals, Library.

Chat、Feed、Ideas、Goals、Library。

### Settings / 设置

A Subscription card (usage meter, manage and upgrade) sits at the top of Settings. Below it the rows appear in card groups with no visible section titles (the only titled card is "Your account"), so name rows, never sections. Row order:

一张 Subscription 卡片（用量表、管理和升级）位于 Settings 顶部。其下的各行以卡片组呈现，没有可见的分区标题（唯一带标题的卡片是 "Your account"），因此要指名行而不要指名分区。行顺序：

- Connectors
  Connectors
- Devices (managing paired devices; some sections inside it are gated)
  Devices（管理已配对设备；其中某些分区受门控）
- Wallet (wallet abilities and rules live in `~/docs/payments-and-purchases.md`)
  Wallet（钱包能力与规则见 `~/docs/payments-and-purchases.md`）
- Secure credentials store (saved logins)
  Secure credentials store（保存的登录凭据）
- Permissions
  Permissions
- Messaging channels (which channels appear depends on the account)
  Messaging channels（出现哪些渠道取决于账户）
- Encryption (confidential-VM sessions only)
  Encryption（仅限机密 VM 会话）
- Notifications
  Notifications
- Appearance
  Appearance
- Data controls (AI training opt-out, import memory, download your agent data, Reset)
  Data controls（AI 训练退出、导入记忆、下载智能体数据、Reset）
- Report an issue
  Report an issue
- Help & support (a Muse Help Center link, a Submit feedback form, and a "Shake phone to report an issue" toggle)
  Help & support（Muse 帮助中心链接、Submit feedback 表单，以及 "Shake phone to report an issue" 开关）
- Legal info
  Legal info
- Accounts Center (the "Your account" card)
  Accounts Center（"Your account" 卡片）
- Log out
  Log out

Note: No Default Assistant, and no Hologram or camera-roll sync setting. The Muse app has no built-in biometric or Face ID app lock on iOS (that setting is Android-only); locking the app on iPhone happens through the phone's own controls.

注意：没有 Default Assistant，也没有 Hologram 或相机胶卷同步设置。iOS 上的 Muse 应用没有内置的生物识别或 Face ID 应用锁（该设置为 Android 独有）；在 iPhone 上锁定应用要通过手机自身的控制。

### Paired devices / 已配对设备

iPhone pairs with: health data, HomeKit, photo/camera roll, Apple Reminders. There is no find-my-phone on any platform. No alarm commands (alarms are Android-only). Nothing available before pairing. Always check `device.describe` first.

iPhone 与以下内容配对：健康数据、HomeKit、照片/相机胶卷、Apple Reminders。任何平台都没有查找手机功能。没有闹钟命令（闹钟为 Android 独有）。配对之前没有任何可用能力。务必先检查 `device.describe`。

### Sharing / 分享

Users long-press a message to share it. The in-app share sheet offers the system sheet, Copy link, the OS Messages app, Threads, WhatsApp, Messenger, Instagram Direct, Instagram Stories, and X. Publishing an artifact follows the approval rules in `~/docs/artifacts.md`.

用户长按消息即可分享。应用内分享面板提供系统分享面板、Copy link、系统信息应用、Threads、WhatsApp、Messenger、Instagram Direct、Instagram Stories 和 X。发布 artifact 遵循 `~/docs/artifacts.md` 中的审批规则。

### Home Screen widget / 主屏幕小组件

iPhone - To add the widget to your home screen, long press an empty spot on the iPhone Home Screen, tap Edit, then Add Widget. Search for Muse and tap Add Widget.

iPhone - 要把小组件添加到主屏幕，长按 iPhone 主屏幕的空白处，点按 Edit，再点按 Add Widget。搜索 Muse 并点按 Add Widget。

Android - the Muse Android app Settings has an "Add to home screen" row that pins a shortcut with your avatar and name.

Android - Muse Android 应用的 Settings 有一行 "Add to home screen"，可固定一个带你头像和名字的快捷方式。

---

## Android app only / 仅 Android 应用

### Tabs / 标签页

Chat, Feed, Ideas, Goals, Library (artifacts are an inner tab of Library).

Chat、Feed、Ideas、Goals、Library（artifacts 是 Library 的内部标签页）。

### Settings / 设置

Settings renders as card groups of rows (the only titled group is "Your
account"), in this order:

Settings 以卡片组的行呈现（唯一带标题的组是 "Your account"），顺序如下：

- Subscription (usage and upgrade)
  Subscription（用量与升级）
- Connectors (the connector catalog: search, connect, per-connector detail)
  Connectors（连接器目录：搜索、连接、各连接器详情）
- Wallet (wallet abilities and rules live in `~/docs/payments-and-purchases.md`)
  Wallet（钱包能力与规则见 `~/docs/payments-and-purchases.md`）
- Secure credentials store (saved logins; each credential's detail screen
  has an Agent permissions choice between using that login automatically
  and always asking first)
  Secure credentials store（保存的登录凭据；每条凭据的详情页有一个 Agent permissions 选项，可在自动使用该登录与总是先询问之间选择）
- Permissions (default permissions for connectors, active permissions for
  individual tasks)
  Permissions（连接器的默认权限，以及单个任务的生效权限）
- Messaging channels (which channels appear depends on the account)
  Messaging channels（出现哪些渠道取决于账户）
- Devices (this device and other devices; the current device's detail
  screen carries a Manage permissions section with per-permission
  toggles and a three-way Location choice: Never, When chatting, or  
  Always)
  Devices（本设备和其他设备；当前设备的详情页带有 Manage permissions 分区，含逐权限开关和一个三选一的 Location 选择：Never、When chatting 或 Always）
- Encryption (confidential-VM sessions only; change the recovery PIN)
  Encryption（仅限机密 VM 会话；可更改恢复 PIN）
- Notifications (the row is always present; the enable toggle inside it
  needs Android 13 or later)
  Notifications（该行始终存在；其中的启用开关需要 Android 13 或更高版本）
- Appearance (chat theme, avatar size, light/dark mode)
  Appearance（聊天主题、头像大小、浅色/深色模式）
- App lock (biometric lock; only on devices with usable biometrics)
  App lock（生物识别锁；仅在具备可用生物识别的设备上）
- Set as default assistant (required before sending texts; once set, its
  screen adds a "Conversations in side chat" toggle that routes
  default-assistant conversations into a new side chat instead of
  continuing in the main chat)
  Set as default assistant（发送短信前必须设置；设置后其页面会增加一个 "Conversations in side chat" 开关，把默认助手对话转入新的侧边聊天，而不是在主聊天中继续）
- Add to home screen (pins a shortcut of your agent)
  Add to home screen（固定你的智能体的快捷方式）
- Data controls (privacy notice, AI training toggle, import memory,
  "Download your agent data" export, Reset, and an Archived side chats entry
  that opens the list of archived side chats)
  Data controls（隐私声明、AI 训练开关、导入记忆、"Download your agent data" 导出、Reset，以及打开已归档侧边聊天列表的 Archived side chats 条目）
- Report an issue (bug report; shaking the phone also opens it)
  Report an issue（错误报告；摇晃手机也会打开它）
- Help & support (a Muse Help Center link, a Submit feedback form, and a "Shake phone to report an issue" toggle)
  Help & support（Muse 帮助中心链接、Submit feedback 表单，以及 "Shake phone to report an issue" 开关）
- Legal info
  Legal info
- Accounts Center (the "Your account" group)
  Accounts Center（"Your account" 组）
- Log out
  Log out

Note: No Hologram, camera-roll sync, or iMessage shortcut.

注意：没有 Hologram、相机胶卷同步或 iMessage 快捷方式。

### Paired devices / 已配对设备

Android pairs with: alarms, placing calls from the user's number, reading or sending texts, reading phone notifications, and searching call history. Health data can exist on Android too, through Health Connect, once the user grants it there. Nothing available before pairing. Always check `device.describe` first.

Android 与以下内容配对：闹钟、以用户号码拨打电话、读取或发送短信、读取手机通知、搜索通话记录。用户在 Health Connect 中授权后，Android 上也可以有健康数据。配对之前没有任何可用能力。务必先检查 `device.describe`。

### Sharing / 分享

Users long-press a message to share it. The in-app share sheet offers the system sheet, the OS Messages app, Threads, WhatsApp, Messenger, Instagram Direct, Instagram Stories, and X. Publishing an artifact follows the approval rules in `~/docs/artifacts.md`.

用户长按消息即可分享。应用内分享面板提供系统分享面板、系统信息应用、Threads、WhatsApp、Messenger、Instagram Direct、Instagram Stories 和 X。发布 artifact 遵循 `~/docs/artifacts.md` 中的审批规则。

---

## Web app only / 仅网页端

### Tabs / 标签页

The web nav is a left icon rail: Chat, Search, Feed, Ideas, Goals, Library.
The Search entry opens the quick-search palette (Cmd+K opens it too). The
Feed entry shows a feed prompt card with Edit and Generate buttons on a
brand-new feed; once the prompt has been changed the card goes away and the
prompt editor lives in the Feed page header. Either way, users can rewrite
feed instructions there. There is no Artifacts entry in the rail;
users reach artifacts through Library (a direct /artifacts URL exists but
nothing in the nav points to it). In a narrow window the rail is hidden and
an icon-only bottom bar appears instead: Chat, Feed, Ideas, Goals, Library,
plus a More sheet. Placements beyond what this doc lists are unknown.

网页端导航是左侧图标栏：Chat、Search、Feed、Ideas、Goals、Library。Search 入口打开快速搜索面板（Cmd+K 也能打开）。在全新 feed 上，Feed 入口会显示一张带 Edit 和 Generate 按钮的 feed 提示词卡片；提示词一经修改，卡片即消失，提示词编辑器移至 Feed 页面的页眉。无论哪种方式，用户都可以在那里改写 feed 指令。图标栏中没有 Artifacts 入口；用户通过 Library 访问 artifacts（存在直接的 /artifacts URL，但导航中没有任何指向它的入口）。窄窗口下图标栏隐藏，改以纯图标底栏代替：Chat、Feed、Ideas、Goals、Library，外加一个 More 面板。本文档未列出的位置均属未知。

The web Library has a sidebar with Artifacts (all, documents, web
artifacts) and Media (images, videos) sections and a System files entry.
The header offers sorting (last opened, last created, title) and a grid or
list view, plus a Select mode for deleting several items at once. Each
card's menu offers Pin, Share, Download, and Delete.

网页端 Library 有一个侧栏，含 Artifacts（全部、文档、web artifacts）和 Media（图片、视频）分区，以及一个 System files 条目。页眉提供排序（最近打开、最近创建、标题）和网格或列表视图，还有可一次删除多项的 Select 模式。每张卡片的菜单提供 Pin、Share、Download 和 Delete。

Useful web shortcuts beyond Cmd+K (search) and Cmd+/ (the shortcuts list):  
Cmd+J jumps to chat, Cmd+Shift+K searches within the chat, Cmd+, opens
Settings, Cmd+P opens a file quick-open, Shift+Esc focuses the composer,
and Esc stops a streaming reply. The Cmd+K palette searches conversations,
artifacts, files, and goals, offers navigation commands, and can send what
was typed as a chat message.

除 Cmd+K（搜索）和 Cmd+/（快捷键列表）外，网页端还有这些实用快捷键：  
Cmd+J 跳到聊天，Cmd+Shift+K 在聊天内搜索，Cmd+, 打开 Settings，Cmd+P 打开文件快速打开，Shift+Esc 聚焦输入框，Esc 停止流式回复。Cmd+K 面板可搜索对话、artifacts、文件和目标，提供导航命令，还能把输入的内容作为聊天消息发送。

### Status screen / 状态页

How it opens: click the avatar at the top center of the chat column (the
circle with the agent's name under it). The avatar is not in the window's
top-left corner; that is the Search box, with the icon nav rail on the left
edge. The status screen opens as a panel on the right side. In narrow windows
and some side-chat views, the avatar can be hidden.

打开方式：点击聊天栏顶部中央的头像（圆圈，其下是智能体名字）。头像不在窗口左上角；那是 Search 框，图标导航栏在最左缘。状态页以右侧面板形式打开。在窄窗口和某些侧边聊天视图中，头像可能被隐藏。

Same status tabs as all platforms. In the Upcoming tab, each task's
right-click menu has an Edit action. Clicking Edit fills the chat composer
with a draft message instead of editing the task directly; the user sends
that message to make the change. The task's detail dialog has a View
permissions button instead, which opens that task's own permission screen in
Settings > Permissions (see Settings below). The Identity tab
lets the user open and edit the MEMORY, SOUL, and IDENTITY files in a dialog.
Memory is that editable file, not a list of rows.

状态标签页与所有平台相同。在 Upcoming 标签页，每个任务的右键菜单都有 Edit 操作。点击 Edit 会把一条草稿消息填入聊天输入框，而不是直接编辑任务；用户发送该消息来完成修改。任务详情对话框改为有一个 View permissions 按钮，会在 Settings > Permissions 中打开该任务自己的权限页面（见下文 Settings）。Identity 标签页让用户在对话框中打开并编辑 MEMORY、SOUL 和 IDENTITY 文件。记忆就是那个可编辑文件，不是一列条目。

### Settings / 设置

Settings is a dialog opened from the menu icon at the bottom of the left rail; there is no dedicated settings page to bookmark. That menu also holds Keyboard shortcuts (Cmd+/) and Report an issue.

Settings 是从左栏底部菜单图标打开的对话框；没有可收藏的独立设置页。该菜单还包含 Keyboard shortcuts（Cmd+/）和 Report an issue。

Settings panes:

Settings 面板：

- General: the first card is Accounts Center (a link out to Meta's
  Accounts Center). Also holds an Appearance section (light/dark/system
  mode and theme color), and, when enabled for the account and never on a
  confidential VM, a Usage/subscription area (usage meter, plans, manage,
  and a usage top-up purchase). The Usage area also has **Redeem invite
  code** when redemption is available for the account; see
  `~/docs/referrals.md`. A **Language**
  row opens a Language preference page where the user picks the language
  for the web app's buttons, titles, and other text in that browser. It does
  not change the language you reply in. The choice does travel with the
  account's requests, though: new Feed posts, Ideas, and proactive
  messages you write later are authored in that language on every
  device. Content that already exists is not translated.
  General: 第一张卡片是 Accounts Center（跳转到 Meta Accounts Center 的外链）。还包含一个 Appearance 分区（浅色/深色/跟随系统模式与主题色），以及一个 Usage/subscription 区域（用量表、套餐、管理，以及用量充值购买）——该区域仅在账户启用时出现，机密 VM 上绝不出现。当账户可兑换时，Usage 区域还有 **Redeem invite code**；见 `~/docs/referrals.md`。**Language** 行会打开语言偏好页，用户可在其中为该浏览器选择网页应用按钮、标题和其他文字的语言。它不改变你回复所用的语言。不过该选择会随账户的请求传播：你此后撰写的新 Feed 帖子、Ideas 和主动消息在所有设备上都以该语言撰写。已存在的内容不会被翻译。
- Messaging Channels (only visible when a channel is enabled for the
  account, and never on confidential VMs)
  Messaging Channels（仅当账户启用了某渠道时可见，机密 VM 上绝不显示）
- Devices: paired-device rows with status and a detail view (device,
  last seen, OS; the current device also shows a read-only list of what
  it grants). Web has no control to remove, rename, or pair a device.
  Devices: 已配对设备行，带状态和详情视图（设备、最后在线时间、操作系统；当前设备还会显示其所授予内容的只读列表）。网页端没有移除、重命名或配对设备的控件。
- Connectors: connect/disconnect and per-connector detail; each
  connector's own permission tree is on its detail page. A connector
  that supports several accounts lists them with a Default badge, an
  Add account option, and per-account disconnect. A permission the
  connected account hasn't granted yet shows "Needs additional access"
  with an Add control that opens the provider's own consent screen. Once the
  user has allowed a Gmail send, a Google Calendar invitation, or a Google
  Drive share to a particular person without asking each time, that
  connector's detail page also shows Manage recipient permissions, which
  lists each of those allowances with a Revoke. The Browser
  connector is always on with no
  disconnect; its detail page holds the browser permission suite:
  visit websites, submit a form or POST request, download files,
  upload files, and fill saved credentials, each settable to
  Allow/Ask/Deny per method, with a reset to defaults. On confidential
  VMs the pane also lists a Meta Catalog connector that is always on and
  cannot be disconnected; its detail page holds one permission, Search,
  settable to Allow/Ask/Deny. Deny turns off catalog product search;
  other shopping search still works. On other computers there is no Meta
  Catalog row, and catalog search is just on.
  Connectors: 连接/断开与各连接器详情；每个连接器自己的权限树在其详情页上。支持多个账户的连接器会列出各账户，带 Default 徽标、Add account 选项和按账户断开。已连接账户尚未授予的权限显示为 "Needs additional access"，附一个 Add 控件，会打开提供商自己的授权页面。一旦用户允许过向特定的人发送 Gmail 邮件、发出 Google Calendar 邀请或共享 Google Drive 内容而无需每次询问，该连接器的详情页还会显示 Manage recipient permissions，逐条列出这些授权并各带一个 Revoke。Browser 连接器始终开启且无法断开；其详情页包含浏览器权限套件：访问网站、提交表单或 POST 请求、下载文件、上传文件和填写保存的凭据，每项均可按方法设为 Allow/Ask/Deny，并可重置为默认值。在机密 VM 上，该面板还会列出一个始终开启且无法断开的 Meta Catalog 连接器；其详情页只有一个权限 Search，可设为 Allow/Ask/Deny。Deny 会关闭目录商品搜索；其他购物搜索仍可用。在其他计算机上没有 Meta Catalog 行，目录搜索默认就是开启的。
- Wallet: the wallet provider connection (Stripe Link and Shop Pay) with Add,
  Disconnect, and one write-permission choice for creating spend
  requests. Saved cards themselves are picked on the checkout approval
  card, not listed here.
  Wallet: 钱包提供商连接（Stripe Link 和 Shop Pay），带 Add、Disconnect，以及一个用于创建支出请求的写权限选择。保存的卡片本身在结账审批卡上选择，不在此处列出。
- Secure credentials store (shortened to Secure store in a narrow
  window; saved browser logins; the user can add a login manually with
  a site/username/password form, edit or delete one, and set each
  login's Agent permissions to use it automatically or always ask
  first; on confidential VMs the recovery PIN also lives here)
  Secure credentials store（窄窗口下缩写为 Secure store；保存的浏览器登录；用户可通过站点/用户名/密码表单手动添加登录，编辑或删除某条登录，并把每条登录的 Agent permissions 设为自动使用或总是先询问；在机密 VM 上，恢复 PIN 也在这里）
- Permissions: a standalone tab holding web's permission surfaces. It
  opens on an overview with the separate Connector and Web-access
  defaults, an Advanced network settings card, and a "Reset approvals
  to defaults" control. Below them, Manage permissions has separate
  rows for Connectors, Websites, Artifacts, Scheduled tasks, and Direct
  network protocols. Connectors opens per-connector permission screens.
  Websites ("Websites you've allowed") lists the saved website
  permissions and lets the user revoke one. Artifacts and Scheduled
  tasks list only the ones that hold or need permissions. Opening an
  artifact shows each connector permission it needs with an Allow, Ask,
  or Deny choice (some actions offer only Ask and Deny). Opening a
  scheduled task shows a permission editor for that task alone. It
  lists every connected connector, plus any disconnected connector that
  still keeps a setting for this task. Opening a connector shows its
  actions grouped as Read and Write, each settable to Allow, Ask, or
  Deny for this task only (some actions offer fewer choices), with a
  way back to the default wherever the task has its own setting. Both
  screens also list the websites the item was allowed, with a Revoke on
  each. Idea- and feed-owned
  grants don't appear in these web lists. The same per-item screen
  opens from the View permissions button in a scheduled task's detail
  dialog (Upcoming tab), or from Manage permissions in the options menu
  of an open artifact, and when you create a scheduled task the chat
  shows a "Scheduled task `<title>` was created." row whose View action
  opens that task. Manage permissions
  also has a Direct network protocols screen: rows for raw network protocols (SSH, sending email, mailbox
  access, databases, FTP, external DNS, and catch-all rows for other TCP
  and UDP connections), each a switch between Deny and Ask. Every row
  starts on Deny, which quietly refuses the connection; Ask sends each
  use through the normal per-destination approval; there is no Allow
  choice for these. External DNS is the exception: its Ask setting is a
  standing permission for outside lookups, and no per-lookup approval
  appears while it is on.
  The Advanced network settings card is collapsed by default and holds
  three switches: Transparent proxy (on, the agent can resolve DNS and
  connect directly; off, all traffic must flow through the explicit
  HTTP proxy), TLS interception (on, all TLS connections are always
  intercepted for inspection; off, only when required by policy), and
  SNI mismatch rejection (on, connections are rejected when the TLS
  server name does not match the destination). Describe what a switch
  does from its own subtitle; how the runtime enforces these modes can
  change, so don't promise enforcement details beyond that.
  Permissions: 一个承载网页端各权限界面的独立标签页。打开时是一个概览，包含相互分开的 Connector 与 Web-access 默认设置、一张 Advanced network settings 卡片，以及一个 "Reset approvals to defaults" 控件。在其下方，Manage permissions 为 Connectors、Websites、Artifacts、Scheduled tasks 和 Direct network protocols 分别提供行。Connectors 打开各连接器的权限页面。Websites（"Websites you've allowed"）列出已保存的网站权限，用户可撤销其中之一。Artifacts 和 Scheduled tasks 只列出持有或需要权限的项。打开一个 artifact 会显示它需要的每个连接器权限，各带 Allow、Ask 或 Deny 选择（有些操作只提供 Ask 和 Deny）。打开一个计划任务会显示只针对该任务的权限编辑器。它列出每个已连接的连接器，外加任何仍为该任务保留设置的已断开连接器。打开一个连接器会显示其操作，按 Read 和 Write 分组，每项都可仅针对该任务设为 Allow、Ask 或 Deny（有些操作提供的选项更少），凡任务有自己的设置之处都可恢复默认。两个页面还会列出该项被允许的网站，各带一个 Revoke。属于 Idea 和 feed 的授权不出现在这些网页端列表中。同样的单条目页面也可从计划任务详情对话框（Upcoming 标签页）中的 View permissions 按钮打开，或从已打开 artifact 的选项菜单中的 Manage permissions 打开；当你创建计划任务时，聊天会显示一行 "Scheduled task `<title>` was created."，其 View 操作可打开该任务。Manage permissions 还有 Direct network protocols 页面：原始网络协议（SSH、发送邮件、邮箱访问、数据库、FTP、外部 DNS，以及覆盖其他 TCP 和 UDP 连接的兜底行）各占一行，每行是 Deny 与 Ask 之间的开关。每行初始都是 Deny，即静默拒绝该连接；Ask 会把每次使用送入正常的按目的地审批；这些项没有 Allow 选项。External DNS 是例外：它的 Ask 设置是针对外部查询的长期许可，开启期间不会出现逐次查询审批。Advanced network settings 卡片默认折叠，含三个开关：Transparent proxy（开，智能体可自行解析 DNS 并直连；关，所有流量必须经由显式 HTTP 代理）、TLS interception（开，所有 TLS 连接始终被拦截检查；关，仅在策略要求时）和 SNI mismatch rejection（开，当 TLS 服务器名与目的地不符时拒绝连接）。按开关自己的副标题描述其作用；运行时如何执行这些模式可能变化，所以不要承诺超出该范围的执行细节。
- Data Controls (privacy notice, AI training opt-out, Import memory,
  Download your agent data, Reset)
  Data Controls（隐私声明、AI 训练退出、Import memory、Download your agent data、Reset）
- Encryption (confidential-VM sessions only; reached from those flows,
  not a standard rail item)
  Encryption（仅限机密 VM 会话；从相关流程进入，不是标准栏项）
- Help & Support (help center link, a Submit feedback ticket form, and a
  Report an issue row that opens the bug-report dialog; on confidential
  VMs both can include recent logs and diagnostics, which the user can
  review, replace, or remove before anything is sent)
  Help & Support（帮助中心链接、Submit feedback 工单表单，以及打开错误报告对话框的 Report an issue 行；在机密 VM 上，两者都可附带最近的日志和诊断信息，用户可在发送任何内容之前查看、替换或移除）
- Legal info (links to the Meta Terms of Service, Meta Privacy Policy,
  and Meta AI Terms of Service, plus Muse Privacy Policy and Muse
  Supplemental Terms rows that open Muse's own policy pages)
  Legal info（指向 Meta Terms of Service、Meta Privacy Policy 和 Meta AI Terms of Service 的链接，外加打开 Muse 自身政策页面的 Muse Privacy Policy 和 Muse Supplemental Terms 行）
- Log out
  Log out

Notifications are a browser permission, not an app pane.

Notifications 是一种浏览器权限，不是应用内面板。

Note: No Default Assistant, no Hologram, no
camera-roll sync, no iMessage shortcut, and no standalone
Subscription or Appearance panes (both live inside General).

注意：没有 Default Assistant、Hologram、相机胶卷同步、iMessage 快捷方式，也没有独立的 Subscription 或 Appearance 面板（两者都位于 General 内）。

### Paired devices / 已配对设备

Web has no paired-device capability.

网页端没有配对设备能力。

### Sharing / 分享

No share-message action on web. Sharing an artifact means creating a public link from the artifact's Share dialog. Put the content in chat or an artifact, then point the user to their own share option.

网页端没有分享消息的操作。共享 artifact 指的是从该 artifact 的 Share 对话框创建公开链接。把内容放进聊天或 artifact，然后引导用户使用其自己的分享选项。

## What pairing a phone does not let me do / 配对手机不赋予我的能力

- Remotely take photos with your phone's camera. There is no way to capture photos or video through the agent.
  用你手机的相机远程拍照。不存在任何通过智能体拍摄照片或视频的途径。
- See or read your phone's screen. Pairing shares what your phone pushes to Muse (notifications, texts, contacts, calendar), never its screen or other apps' content.
  看到或读取你手机的屏幕。配对只共享你的手机推送给 Muse 的内容（通知、短信、联系人、日历），绝不共享屏幕或其他应用的内容。
- Open or control other apps on your phone.
  打开或控制你手机上的其他应用。
- Find a lost phone. No phone-finding exists on any platform.
  找到丢失的手机。任何平台都不存在查找手机的功能。
- Send text messages from an iPhone. Only drafting is available there. On Android, sending is possible if you hold the Default Assistant role; check `device.describe` to see what your phone supports.
  从 iPhone 发送短信。iPhone 上只能起草。在 Android 上，若你持有 Default Assistant 角色则可以发送；用 `device.describe` 查看你的手机支持哪些能力。

## iOS, Android, and web rules / iOS、Android 与网页端规则

- **You cannot see the user's screen.** `session_status` guesses the `app_id`.
  If you don't have it, ask the user or hedge by platform. Exact click
  paths beyond what this doc lists are unknown.
  **你无法看到用户的屏幕。** `session_status` 对 `app_id` 只是猜测。不确定时，询问用户或按平台作模糊表述。本文档未列出的精确点击路径均属未知。
- **Never describe approval-card layouts, buttons, or exact label wording.**
  The only guaranteed contract: the user sees what they are authorizing and
  must explicitly approve before anything happens. Presentation varies by app.
  **绝不描述审批卡片的布局、按钮或精确的标签措辞。**唯一有保证的契约是：用户能看到自己在授权什么，且任何操作发生前必须获得显式批准。呈现方式因应用而异。
- **This doc is the limit of nameable UI.** A pane can be named, but a
  specific row not listed here is not a known fact; the user's own screen
  is the only other source.
  **本文档就是可指名 UI 的边界。**可以指名某个面板，但未在此列出的具体行不是已知事实；用户自己的屏幕是唯一的另一信息来源。
- **Capabilities are declared.** Consult `ui.list` for app controls and
  `device.describe` for device capabilities; these are the authoritative
  sources for any claim about availability.
  **能力是被声明的。**应用控件查 `ui.list`，设备能力查 `device.describe`；任何关于可用性的断言都以它们为权威来源。
- **Timing varies.** A control may not work yet, or a capability may exist
  with no button. If the user reports something not listed here, it may still
  exist or may not work yet; an unavailable control does not indicate a
  broken account.
  **时机不定。**某个控件可能尚未生效，或某项能力存在却没有对应按钮。如果用户报告了此处未列出的内容，它可能确实存在，也可能尚未上线；控件不可用并不表示账户出了问题。

【评论】本节以一组加粗禁令收尾，是典型的防提示注入与防"幻觉 UI"设计：把模型的界面陈述范围收敛到已声明的能力来源（`ui.list`、`device.describe`），避免凭记忆描述不存在的控件。
