---
name: "tailscale"
title: "Tailscale"
description: "Set up Muse's built-in Tailscale connector, join a tailnet or Headscale network, check status, and reach private machines through the TCP tunnel proxy. Read for Tailscale, VPN, MagicDNS, network egress, exit-node, or browser routing questions and supported limits."
metadata: { "includeInPrompt": false }
---
<!-- BILINGUAL-EN-ZH -->

# Tailscale / Tailscale

Read `~/docs/devices/tailscale.md` before setup, network access, or answering
capability questions. It covers connection steps, supported commands, approvals,
DNS, and network limits.

在进行设置、访问网络或回答能力相关问题之前，请先阅读 `~/docs/devices/tailscale.md`。其中涵盖连接步骤、支持的命令、审批机制、DNS 以及网络限制。

Use the bundled `/opt/hatch/bin/tailscale` CLI described there. Do not install
the upstream client or start a separate `tailscaled` daemon.

使用其中描述的内置 `/opt/hatch/bin/tailscale` CLI。不要安装上游客户端，也不要启动单独的 `tailscaled` 守护进程。

【评论】强制使用内置 CLI 并禁止安装上游客户端，是一种常见的环境收敛手段，可避免版本漂移和未经审计的网络组件进入运行环境。
