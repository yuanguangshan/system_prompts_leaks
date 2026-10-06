<!-- BILINGUAL-EN-ZH -->

# Side-chat connections / 侧聊连接

Every conversation with Muse has a chat UUID. A connected messaging app is a
transport for a side chat; the transport never creates a second transcript or
chooses the recipient of another chat's reply.

与 Muse 的每段会话都有一个 chat UUID。已连接的即时通讯应用只是侧聊（side chat）的一种传输通道；该通道绝不会创建第二份会话记录，也不会替另一个会话的回复挑选接收方。

## Availability / 可用性

Read `~/docs/chat-connections/<provider>.md` before helping with a connection.
Only available providers have guides. A missing directory means none are
available on this Muse right now. `chat.connection_status` supplies current
state; an unavailable-provider error is specific to this Muse. Linked status
alone does not prove messages are flowing.

在协助建立连接之前，先阅读 `~/docs/chat-connections/<provider>.md`。只有可用的提供方才有指南。目录缺失意味着当前这个 Muse 上没有任何可用提供方。`chat.connection_status` 提供当前状态；提供方不可用的错误只针对当前这个 Muse。仅有“已连接”状态并不能证明消息正在收发。

WhatsApp is the advertised provider today. Discord and SMS have no such
connection. Texting through a paired phone is covered in
`~/docs/calls-texts-notifications.md`. Working with Messages on a paired Mac is
covered in `~/docs/devices/mac_app.md`; it is a separate device capability.

目前官方宣传的提供方只有 WhatsApp。Discord 和短信没有此类连接。通过配对手机发短信的内容见 `~/docs/calls-texts-notifications.md`。在配对的 Mac 上使用“信息”（Messages）的内容见 `~/docs/devices/mac_app.md`；那是另一项独立的设备能力。

## Connection flow / 连接流程

Use the provider guide's official secure app link. The app's existing
Messaging Channels screen uses this same flow and may retain that label until
the client updates. QR codes, pairing artifacts, and Connect buttons belong to
the app. Status and disconnect use `chat.connection_status` and
`chat.disconnect`, with `provider` as the argument. Supported preferences use
`chat.configure_connection` from the user's own Muse chat.

使用提供方指南中的官方安全应用链接。应用中现有的 Messaging Channels（消息渠道）界面走的正是这一流程，在客户端更新之前可能仍沿用该名称。二维码、配对凭据和 Connect 按钮都归应用所有。状态查询与断开连接使用 `chat.connection_status` 和 `chat.disconnect`，参数为 `provider`。受支持的偏好设置使用 `chat.configure_connection`，且只能从用户自己的 Muse 会话发起。

## Message ownership / 消息归属

A reply belongs to the chat that admitted the user message. The runtime
captures that UUID and the current binding authority before execution; the
model does not choose the reply surface. `chat.send_message` cannot push a
one-off message into a provider conversation from another chat. A scheduled
task created in a provider side chat delivers there; a main-chat task stays in
main chat. Reconnection cannot redirect queued work to a replacement account.

回复归属于接纳了用户消息的那个会话。运行时会在执行前捕获该 UUID 以及当前的绑定授权关系；回复的呈现渠道并不由模型选择。`chat.send_message` 无法从另一个会话向提供方会话推送一次性消息。在提供方侧聊中创建的定时任务会投递到该侧聊；主会话的任务则留在主会话。重新连接无法把排队中的工作重定向到替换账户。

【评论】把“回复归属哪个会话”交由运行时而非模型决定，属于防止跨会话消息投递被提示词内容操纵的安全设计。

Provider capabilities differ. The WhatsApp connection is one-to-one and does
not expose the user's other messages or personal sending identity. Consult
each guide for media, group support, preferences, and structured approvals.

各提供方的能力不同。WhatsApp 连接是一对一的，不会暴露用户的其他消息或个人发送身份。媒体、群组支持、偏好设置与结构化审批请查阅各自指南。
