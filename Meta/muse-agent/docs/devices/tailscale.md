---
summary: "Tailscale - private network connecting this VM to the user's own machines"
read_when:
  - Reach a machine on the user's Tailscale network or tailnet
  - Join a Tailscale network from this device, or check whether it has
  - Reach a personal, work, or home machine not on the public internet
  - Access a service on a `100.64.0.0/10` address (`100.64.` through `100.127.`)
title: "Tailscale"
---
<!-- BILINGUAL-EN-ZH -->
# Tailscale / Tailscale

This device requests the name `muse` on your Tailscale network.

该设备在你的 Tailscale 网络中请求使用名称 `muse`。

Tailscale is a private network joining this VM to the user's own machines. Once
connected, they are reachable at their tailnet addresses, which Tailscale
assigns from `100.64.0.0/10` — anything from `100.64.` to `100.127.`, not just  
`100.64.`.

Tailscale 是一个将此虚拟机与用户自己的机器连接起来的私有网络。连接建立后，这些机器可以通过它们的 tailnet 地址访问，这些地址由 Tailscale 从 `100.64.0.0/10` 中分配——即从 `100.64.` 到 `100.127.` 的任意地址，而不仅仅是 `100.64.`。

This device joins as a client only: it makes outgoing connections and accepts
none. It cannot be reached from the tailnet, hosts nothing, and does not route
traffic for other machines.

该设备仅以客户端身份加入：它只发起出站连接，不接受任何入站连接。tailnet 无法访问它，它不托管任何服务，也不为其他机器路由流量。

> Tailscale support is experimental and work in progress.

> Tailscale 支持尚处于实验阶段，仍在开发中。

## Connecting / 连接

```bash
tailscale up
```

Prints a link. **Show it to the user and ask them to open it** — only they can
approve this device. Approval finishes the join on its own; the connection comes
up as soon as they approve, and `tailscale status` is how you confirm it.

该命令会打印一个链接。**把链接展示给用户并请他们打开**——只有他们能批准这台设备。批准后加入流程会自行完成；用户一批准连接就会建立，你可以用 `tailscale status` 加以确认。

Safe to run again while waiting: it shows the same link again and keeps
watching for the approval. The link stays good for five days. This device
stops watching a while after each ask, so if the user needs longer, run it
again; it picks up the same link.

等待期间可以安全地重复运行：它会再次显示同一个链接，并继续等待批准。该链接在五天内有效。每次发起请求后，本设备会在一段时间后停止监听，因此如果用户需要更长时间，再次运行即可，它会重新接上同一个链接。

If the user runs their own control server (headscale or another self-hosted
control plane), pass it the way the vanilla CLI does — an https origin only:

如果用户运行自己的控制服务器（headscale 或其他自托管控制平面），按原版 CLI 的方式传入——仅接受 https 源（origin）：

```bash
tailscale up --login-server https://headscale.example.com
```

The choice persists with the enrollment and `tailscale status` names the login
server in use; switching to a different one needs `tailscale down` first.
Approving a custom-server join never leaves a reusable permission behind: each
approval is single-use and names the server, unlike a plain `tailscale up`,
which can carry a standing allow.

该选择会随注册信息持久保存，`tailscale status` 会显示当前使用的登录服务器；切换到其他服务器需要先执行 `tailscale down`。批准加入自定义服务器绝不会留下可复用的权限：每次批准都是一次性的，并且绑定了具体的服务器，这与普通的 `tailscale up` 不同，后者可能携带长期有效的允许授权。

【评论】此处对比了自定义服务器批准（一次性、绑定服务器）与普通 tailscale up（可能留下长期授权）在权限残留上的差异，属于安全细节说明。

`tailscale down` disconnects this VM and erases its Tailscale identity, so
coming back up needs a fresh approval — use it when the user wants this VM to
stop holding access.

`tailscale down` 会断开此虚拟机并抹除其 Tailscale 身份，因此重新上线需要重新批准——当用户想让此虚拟机不再持有访问权限时使用它。

It cannot remove the device from the Tailscale admin console or their own
control server. Tell the user to delete it there themselves.

它无法把该设备从 Tailscale 管理控制台或用户自己的控制服务器中移除。请告知用户自行在该处删除。

## Reaching Machines / 访问机器

Ordinary network access does not reach the tailnet. Reuse the runtime proxy,
changing only its port to `3130`:

普通网络访问无法到达 tailnet。复用运行时代理，只把端口改为 `3130`：

```bash
tunnel_proxy="${HTTPS_PROXY%:*}:3130"
curl --proxy "$tunnel_proxy" --fail-with-body "http://<tailnet-ip>:<port>/<path>"
```

The port selects Tailscale; the address is just the destination. This carries
any TCP service, not only HTTP — point a client at the same proxy.

端口用于选择 Tailscale；地址只是目标。它可以承载任何 TCP 服务，而不仅是 HTTP——让客户端指向同一个代理即可。

**TCP only, through that proxy.** The tunnel carries connections, not packets,
so `ping`, traceroute, and anything else over ICMP or UDP never reach the
tailnet — they fail or hang even when the machine is up and reachable. A failed
ping is not evidence of anything; do not report it as the machine being down.
Opening a TCP connection through the proxy is the only test.

**仅支持 TCP，且必须经过该代理。** 隧道承载的是连接而不是数据包，因此 `ping`、traceroute 以及其他任何基于 ICMP 或 UDP 的流量都无法到达 tailnet——即使机器在线且可达，它们也会失败或挂起。ping 失败不能说明任何问题；不要将其报告为机器宕机。通过代理建立 TCP 连接是唯一的测试方法。

- The first tunnel request to an address needs its own approval; web permission
  does not carry over. Once approved, the selected permission lifetime covers
  other ports and TCP services, including SSH, on that exact name or IP through
  this proxy. Another name or IP for the same machine (short name, full
  MagicDNS name, IPv4, IPv6) is a different address and needs its own approval;
  keep using one.
  对某个地址的第一个隧道请求需要单独批准；网页权限不会自动沿用。一经批准，所选权限的有效期即覆盖通过此代理访问该确切名称或 IP 上的其他端口和 TCP 服务（包括 SSH）。同一台机器的另一个名称或 IP（短名称、完整 MagicDNS 名称、IPv4、IPv6）属于不同地址，需要单独批准；请坚持使用同一个地址。
- Any request awaiting approval may wait for the user to answer it. Expected: do
  not time out, retry, or read the pause as a failure.
  等待批准的请求可能会一直等待用户回应。这是预期行为：不要超时、重试，也不要把这种停顿当作失败。
- If this device has not joined a network, or the address is not one the
  tailnet reaches, the request fails rather than falling back to the public
  internet. Check `tailscale status`.
  如果本设备尚未加入网络，或该地址不在 tailnet 可达范围内，请求会直接失败，而不会回退到公共互联网。请检查 `tailscale status`。
- Never put credentials in a command.
  绝不在命令中放入凭据。

## Inspecting / 查看状态

`tailscale status` lists the connection state and every device with its
addresses; `tailscale ip` shows this device's own.

`tailscale status` 列出连接状态以及每台设备及其地址；`tailscale ip` 显示本设备自己的地址。

Machines can be addressed by tailnet IP or, when the user's network enables
MagicDNS, by their name. Requests through port `3130` use the network's DNS
settings, including search domains, split DNS, NextDNS over HTTPS, plain DNS,
DNS-over-TLS, and private DNS servers reachable through the tailnet. If a DNS
server is unavailable, the request can try another server configured for that
same DNS route. A blocking or negative answer does not switch DNS providers.
These settings apply to requests through the Tailscale proxy. The connector
does not offer exit-node selection.

机器可以通过 tailnet IP 寻址，或者在用户的网络启用 MagicDNS 时通过其名称寻址。通过端口 `3130` 发出的请求使用网络的 DNS 设置，包括搜索域、split DNS、NextDNS over HTTPS、普通 DNS、DNS-over-TLS，以及可通过 tailnet 访问的私有 DNS 服务器。如果某个 DNS 服务器不可用，请求可以尝试为同一 DNS 路由配置的另一个服务器。阻断性或否定性应答不会导致切换 DNS 提供方。这些设置适用于通过 Tailscale 代理发出的请求。该连接器不提供出口节点（exit node）选择。

`tailscale status` says whether an address is on the network. There is no
reachability probe; test the service with a TCP connection through the proxy.

`tailscale status` 可以说明某个地址是否在网络上。没有可达性探测机制；请通过代理建立 TCP 连接来测试服务。

## User-Facing Language / 面向用户的语言

Say "your Tailscale network" or name the machine. Do not quote raw commands or
`100.64.0.0/10` addresses unless the user asks for detail.

说"你的 Tailscale 网络"或指名具体机器。除非用户索要细节，否则不要引用原始命令或 `100.64.0.0/10` 地址。
