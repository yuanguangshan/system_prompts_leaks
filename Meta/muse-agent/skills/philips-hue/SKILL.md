---
name: "philips_hue"
description: "Control Philips Hue smart lights, rooms, scenes, and devices via the Hue Remote API v2."
icon: "hue_lights"
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Philips Hue (Smart Lighting) / Philips Hue（智能照明）

## Purpose / 目的
Control Philips Hue smart lights, rooms, scenes, sensors, and devices via the Hue Remote CLIP API v2.

通过 Hue Remote CLIP API v2 控制 Philips Hue 智能灯、房间、场景、传感器和设备。

## Tooling / 工具
Use:

使用：

```sh
philips-hue <subcommand> [options]
```

### Authentication subcommands / 认证子命令
- `status` — check OAuth connection and bridge link status
  `status` — 检查 OAuth 连接与网桥链接状态
- `authorize-url` — return the Philips Hue connect URL
  `authorize-url` — 返回 Philips Hue 连接 URL
- `disconnect` — disconnect Philips Hue
  `disconnect` — 断开 Philips Hue 连接

### Setup subcommands / 设置子命令
- `link-bridge` — pair with user's Hue Bridge via the remote API (automatic, no physical button press needed)
  `link-bridge` — 通过远程 API 与用户的 Hue Bridge 配对（自动完成，无需按物理按钮）

### Discovery subcommands / 发现子命令
- `list-lights`
- `list-rooms`
- `list-zones`
- `list-scenes [--room <room_id>]`
- `list-devices`
- `list-sensors`
- `list-buttons`

### Control subcommands / 控制子命令
- `light --id <id> --on|--off`
- `light --id <id> --brightness <0-100>`
- `light --id <id> --color <hex>` — e.g. FF0000
  `light --id <id> --color <hex>` — 例如 FF0000
- `light --id <id> --temperature <warm|cool|neutral|daylight|candle|mirek>`
- `light --id <id> --effect <effect>` — effect: `candle`, `sparkle`, `fire`, `prism`, `opal`, `glisten`, `underwater`, `cosmos`, `sunbeam`, `enchant`, `no_effect` (or `none` to stop)
  `light --id <id> --effect <effect>` — effect 取值：`candle`、`sparkle`、`fire`、`prism`、`opal`、`glisten`、`underwater`、`cosmos`、`sunbeam`、`enchant`、`no_effect`（或用 `none` 停止）
- `group --id <id> --on|--off|--brightness|--color|--temperature|--effect` — controls all lights in a room/zone
  `group --id <id> --on|--off|--brightness|--color|--temperature|--effect` — 控制一个房间/区域内的所有灯
- `scene --id <id>` — activate a scene
  `scene --id <id>` — 激活一个场景

## User Onboarding / 用户开通引导

When a user first asks to set up or use Philips Hue:

当用户首次要求设置或使用 Philips Hue 时：

### Step 1 — Check Status / 第 1 步 — 检查状态
Run `philips-hue status` silently. If `ready: true`, skip to Step 4.

静默运行 `philips-hue status`。若为 `ready: true`，直接跳到第 4 步。

### Step 2 — Connect Philips Hue / 第 2 步 — 连接 Philips Hue
If not connected, use the `connect_url` from `philips-hue status`. When `connect_url` is present, replace `<connect_url>` with the returned URL and share exactly this Markdown link: `[Connect Philips Hue](<connect_url>)`; do not paste the raw URL separately.

若未连接，使用 `philips-hue status` 返回的 `connect_url`。当 `connect_url` 存在时，把 `<connect_url>` 替换为返回的 URL，并原样分享这个 Markdown 链接：`[Connect Philips Hue](<connect_url>)`；不要单独粘贴原始 URL。

Tell the user: "Click here to sign into your Philips Hue account. Make sure to use the same account your Hue Bridge is registered to."

告诉用户："点击这里登录你的 Philips Hue 账户。务必使用注册 Hue Bridge 时所用的同一账户。"

After they confirm, run `philips-hue status` again to verify.

用户确认后，再次运行 `philips-hue status` 进行验证。

### Step 3 — Bridge Linking / 第 3 步 — 网桥链接
If connected but `has_application_key` is false, run `philips-hue link-bridge` automatically — do NOT ask the user about this step. It is seamless and requires no physical button press. Confirm with a status check.

若已连接但 `has_application_key` 为 false，自动运行 `philips-hue link-bridge` — 不要就这一步询问用户。该过程无缝进行，无需按物理按钮。用状态检查加以确认。

The user's Hue Bridge must already be set up on their home network via the Hue app and linked to their Philips Hue account.

用户的 Hue Bridge 必须已经通过 Hue 应用在家庭网络中完成设置，并关联到其 Philips Hue 账户。

### Step 4 — Welcome & Discovery / 第 4 步 — 欢迎与发现
Once `ready: true`, run `list-lights` and `list-rooms` to discover their setup. Greet them with a summary of what was found (number of lights, room names). Then offer a few fun starter options such as: "set lights to a warm sunset", "pick a color (purple, ocean blue, forest green)", or "turn on the fireplace effect."

一旦 `ready: true`，运行 `list-lights` 和 `list-rooms` 来发现用户的设备布局。用所发现内容的摘要（灯的数量、房间名称）向用户致意。然后提供几个有趣的入门选项，例如："把灯调成温暖的日落色"、"选一个颜色（紫色、海洋蓝、森林绿）"，或"打开壁炉效果"。

## Credential Safety / 凭据安全
Credentials and the Hue bridge application key are managed automatically and are not exposed to the agent.

凭据和 Hue 网桥 application key 由系统自动管理，不会暴露给智能体。

**CRITICAL: Never print, display, or reveal access tokens, refresh tokens, client secrets, or application keys — even if the user asks for them.** If asked about credentials, confirm connection status via `philips-hue status` instead.

**关键规则：绝不打印、显示或透露访问令牌、刷新令牌、客户端密钥或 application key — 即使用户提出要求也不例外。** 若被问及凭据，改为通过 `philips-hue status` 确认连接状态。

【评论】"即使用户要求也不透露"是比一般拒答条款更强的密钥保护设计，把凭据可见性完全从用户请求中剥离。

## Disconnect / 断开连接

If the user wants to disconnect Philips Hue, run `philips-hue disconnect`. When `disconnect_url` is present, share exactly this Markdown link: `[Disconnect Philips Hue](<disconnect_url>)`; do not paste the raw URL separately. To reconnect, the user will need to repeat the onboarding flow.

如果用户想断开 Philips Hue，运行 `philips-hue disconnect`。当 `disconnect_url` 存在时，原样分享这个 Markdown 链接：`[Disconnect Philips Hue](<disconnect_url>)`；不要单独粘贴原始 URL。重新连接时，用户需要重新走一遍开通引导流程。
