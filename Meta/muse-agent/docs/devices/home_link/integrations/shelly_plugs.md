<!-- BILINGUAL-EN-ZH -->
# Shelly Plugs (Gen 4): Switching and Power Metering / Shelly 智能插座（第四代）：开关控制与功率计量

**Last verified:** 2026-09-21 on a Shelly Plug US Gen 4 (model S4PL-00116US,
firmware 2.0.0)

**最后验证时间：** 2026-09-21，在 Shelly Plug US 第四代（型号 S4PL-00116US，固件 2.0.0）上验证

Use this guide when fresh discovery identifies a Shelly smart plug and the user
asks to switch it or to read what it is drawing.

当全新发现流程识别出 Shelly 智能插座、且用户要求对其进行开关操作或读取其当前用电情况时，使用本指南。

## Identifying the Plug / 识别插座

- **mDNS** is the strongest signal. Shelly plugs advertise over `_http._tcp`
  with a hostname built from the model and the device MAC, such as
  `ShellyPlugUSG4-<MAC>.local`, where `<MAC>` is the twelve hex digits with no
  separators. The device reports its own MAC the same way, without colons.
- **mDNS** 是最强的信号。Shelly 插座通过 `_http._tcp` 对外通告，其主机名由型号和设备 MAC 构成，例如 `ShellyPlugUSG4-<MAC>.local`，其中 `<MAC>` 是不带分隔符的十二位十六进制字符。设备报告自身 MAC 时也采用同样格式，不带冒号。
- **The MAC vendor prefix** is supporting evidence, not proof. Shelly hardware
  has shipped under more than one prefix, so a match is a positive signal and a
  miss proves nothing. Resolve the current prefixes the way
  `~/docs/devices/home_link.md` requires: look up the manufacturer's public
  vendor prefixes, then compare them against the discovery result locally.
  Send only a vendor prefix to an online lookup, never a full MAC address.
- **MAC 厂商前缀** 是辅助证据，而非证明。Shelly 硬件曾在多个前缀下出厂，因此匹配是正面信号，而不匹配什么也证明不了。按 `~/docs/devices/home_link.md` 的要求解析当前前缀：查询制造商公开的厂商前缀，然后在本地将其与发现结果比对。在线查询时只发送厂商前缀，绝不发送完整 MAC 地址。
- **Confirm before acting.** Request `Shelly.GetDeviceInfo` and check `model`,
  `app`, and `gen`. That answer, not the hostname, is what establishes the
  device is what you think it is.
- **行动之前先确认。** 请求 `Shelly.GetDeviceInfo` 并检查 `model`、`app` 和 `gen`。是这一应答——而不是主机名——确立了设备确实是你所认为的设备。

Addresses are DHCP by default. Identify the plug by MAC or hostname and take
its address from the current discovery result every time. Never reuse a
remembered address.

地址默认由 DHCP 分配。通过 MAC 或主机名识别插座，且每次都从当前发现结果中取其地址。绝不复用记忆中的地址。

## Check the Generation First / 先确认代数

The API depends on the generation, and the two are not compatible.

API 取决于设备代数，两代之间互不兼容。

- `gen` 2 or higher, which covers Gen 4, uses the RPC API described below.
- `gen` 为 2 或更高（涵盖第四代）时，使用下文所述的 RPC API。
- `gen` 1 predates RPC entirely and uses `/relay/0?turn=on`. If you find a
  Gen 1 device, the rest of this guide does not apply.
- `gen` 1 完全早于 RPC，使用 `/relay/0?turn=on`。如果发现的是第一代设备，本指南其余内容不适用。

## Recommended Path / 推荐路径

1. Re-discover the plug and take its address and port from the current result.
2. 重新发现插座，并从当前结果中获取其地址和端口。
2. Reach it through the Home Link CONNECT proxy. It serves a documented HTTP
   API, so the HTTP section of `~/docs/devices/home_link.md` applies,
   including its approval and redirect rules.
3. 通过 Home Link CONNECT 代理访问它。它提供有文档说明的 HTTP API，因此 `~/docs/devices/home_link.md` 的 HTTP 部分适用，包括其中的批准与重定向规则。
3. Call `Shelly.GetDeviceInfo` to confirm the model and generation, and to read  
   `auth_en`.
4. 调用 `Shelly.GetDeviceInfo` 确认型号和代数，并读取 `auth_en`。
4. Read `Switch.GetStatus?id=0` before acting, so you know the state you are
   changing and can tell a no-op from a real change. A plug has one switch, at  
   `id=0`.
5. 在操作之前读取 `Switch.GetStatus?id=0`，从而了解将要改变的状态，并能区分无效果的操作与真实变更。插座只有一个开关，位于 `id=0`。
5. Switch with `Switch.Set?id=0&on=true` or `on=false`. The reply is
   `{"was_on": <bool>}`, which is the state *before* the call, not the result.
   Treating it as the new state inverts the meaning.
6. 使用 `Switch.Set?id=0&on=true` 或 `on=false` 进行开关。应答为 `{"was_on": <bool>}`，它是调用*之前*的状态，而不是结果。把它当作新状态会把含义颠倒。
6. Read the state back. `Switch.GetStatus?id=0` gives `output` for the relay
   and `apower` for watts actually drawn. `output` alone says the relay moved;
   `apower` is what shows the attached load responded. Confirm both before
   telling the user it worked.
7. 回读状态。`Switch.GetStatus?id=0` 提供继电器的 `output` 以及实际汲取功率（瓦特）`apower`。仅有 `output` 只说明继电器动作了；`apower` 才表明所接负载作出了响应。在告诉用户操作成功之前，两者都要确认。

For an action that should not be left latched on, `Switch.Set` also accepts
`toggle_after=<seconds>`, which reverts the plug without needing a second call
to survive.

对于不应保持开启状态的操作，`Switch.Set` 还接受 `toggle_after=<seconds>`，它无需第二次调用即可将插座恢复原状，且该恢复持续生效。

## Endpoints / 端点

| Action | Path |
|---|---|
| Device model, generation, firmware, `auth_en` | `/rpc/Shelly.GetDeviceInfo` |
| Full device status | `/rpc/Shelly.GetStatus` |
| One switch's status | `/rpc/Switch.GetStatus?id=0` |
| Turn on or off | `/rpc/Switch.Set?id=0&on=true` |
| Turn on or off, reverting later | `/rpc/Switch.Set?id=0&on=true&toggle_after=60` |
| Toggle | `/rpc/Switch.Toggle?id=0` |
| Switch configuration, including auto-off | `/rpc/Switch.SetConfig` |
| Schedules | `/rpc/Schedule.List`, `Schedule.Create`, `Schedule.Delete` |

| 操作 | 路径 |
|---|---|
| 设备型号、代数、固件、`auth_en` | `/rpc/Shelly.GetDeviceInfo` |
| 完整设备状态 | `/rpc/Shelly.GetStatus` |
| 单个开关的状态 | `/rpc/Switch.GetStatus?id=0` |
| 打开或关闭 | `/rpc/Switch.Set?id=0&on=true` |
| 打开或关闭，稍后恢复 | `/rpc/Switch.Set?id=0&on=true&toggle_after=60` |
| 切换 | `/rpc/Switch.Toggle?id=0` |
| 开关配置，含自动关闭 | `/rpc/Switch.SetConfig` |
| 定时计划 | `/rpc/Schedule.List`、`Schedule.Create`、`Schedule.Delete` |

## Auth / 认证

Read `auth_en` from `Shelly.GetDeviceInfo` rather than assuming. A plug on a
home network is often left unauthenticated, and plain HTTP works. That is a
property of the individual device, not of the model.

应从 `Shelly.GetDeviceInfo` 读取 `auth_en`，而不是凭假设。家庭网络中的插座常处于未认证状态，且纯 HTTP 可用。这是个别设备的属性，不是该型号的属性。

When `auth_en` is true the device wants HTTP digest auth, and the user name is
always `admin`. Do not put the password in the command. If no stored credential
exists for the plug, follow the credential rules in
`~/docs/devices/home_link.md` and explain that setup is required.

当 `auth_en` 为 true 时，设备要求 HTTP 摘要认证，且用户名始终为 `admin`。不要把密码写进命令。如果该插座没有已存储的凭据，遵循 `~/docs/devices/home_link.md` 中的凭据规则，并说明需要先行设置。

## Other Capabilities / 其他能力

Worth knowing about, though most requests will not need them:

值得了解，尽管多数请求用不到：

- Power metering beyond `apower`: voltage, current, frequency, and cumulative
  energy in watt-hours under `aenergy`, with recent per-minute history.
- `apower` 之外的功率计量：电压、电流、频率，以及 `aenergy` 下以瓦时为单位的累计电能，并附带最近的每分钟历史。
- A built-in light sensor, reported under `illuminance:0`.
- 内置光照传感器，通过 `illuminance:0` 报告。
- Overpower and overcurrent protection thresholds, in the switch config.
- 过功率与过电流保护阈值，位于开关配置中。
- BLE and BTHome, MQTT, the vendor cloud, and Matter. These are alternative
  control planes. Prefer the local RPC API over the network the Link already
  reaches.
- BLE 和 BTHome、MQTT、厂商云以及 Matter。这些是备选控制平面。优先使用 Link 已可达网络之上的本地 RPC API。
