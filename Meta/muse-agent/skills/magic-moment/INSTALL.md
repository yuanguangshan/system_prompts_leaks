<!-- BILINGUAL-EN-ZH -->

# Installing magic-moment on a Muse VM / 在 Muse VM 上安装 magic-moment

There is nothing to install. The skill tar ships the code and fonts (~13MB
on disk, ~11MB tar — no avatar; that is per-user and read off the VM at
compose time), and everything else the renders need already ships with the
VM image:

无需安装任何东西。技能 tar 包自带代码和字体（磁盘上约 13MB，tar 约 11MB —— 不含 avatar；avatar 按用户存储，在合成时从 VM 读取），渲染所需的其他一切都已随 VM 镜像提供：

| Piece | Where it ships |
|---|---|
| imaging (PIL 10.2) | cell image (`python3-pil`) |
| ffmpeg / ffprobe | cell image (`/usr/bin`) |
| capture browser driver | the bundle's playwright-core at `/opt/hatch/skills/spaces/ts-runtime/dist/node_modules` (bundle contract), on the cell's node |
| browser | image-baked `/opt/meta-chromium/chrome` |
| transcription | daemon sandbox API → inference-proxy → host ASR service; cell ffmpeg extracts 16 kHz mono audio and ffprobe reads clip duration |

| 组件 | 随什么提供 |
|---|---|
| 图像处理（PIL 10.2） | cell 镜像（`python3-pil`） |
| ffmpeg / ffprobe | cell 镜像（`/usr/bin`） |
| 截图浏览器驱动 | bundle 自带的 playwright-core，位于 `/opt/hatch/skills/spaces/ts-runtime/dist/node_modules`（bundle 契约），跑在 cell 的 node 上 |
| 浏览器 | 镜像内置的 `/opt/meta-chromium/chrome` |
| 转写 | daemon 沙箱 API → inference-proxy → 宿主机 ASR 服务；cell 的 ffmpeg 提取 16 kHz 单声道音频，ffprobe 读取片段时长 |

## Verify / 验证

One command, run by whoever will run the renders — normally the agent
itself, from inside the cell. No sudo, no root, no network:

只需一条命令，由将要执行渲染的人运行——通常就是代理自身，在 cell 内部运行。无需 sudo、无需 root、无需网络：

```bash
bash <skill-dir>/install.sh          # seconds, idempotent
bash <skill-dir>/install.sh --check  # sub-second preflight, never mutates
```

The full run verifies every leg of the shipped stack, does a smoke render
through the real capture path, and reclaims the vendor layer older
installs left on the home volume (WeasyPrint era through the pip-playwright
era, up to ~500MB of dead weight).

完整运行会验证随附技术栈的每一条链路，通过真实截图路径做一次冒烟渲染，并回收旧版安装在 home 卷上遗留的 vendor 层（从 WeasyPrint 时代到 pip-playwright 时代，最多约 500MB 的无用负担）。

No daemon restart: skill discovery is content-hashed, so the agent sees the
skill on its next turn.

无需重启 daemon：技能发现基于内容哈希，代理在下一轮就能看到该技能。

Why the rules exist:
这些规则存在的原因：
- **Never install pieces by hand.** The shipped stack is the contract the
  pipeline was tuned against. If `install.sh` says the stack is incomplete,
  the VM image predates it — report that setup is not possible on this VM;
  do not improvise with pip, npm, or browser downloads.
  **绝不动手单独安装组件。** 随附技术栈就是流水线调优所依据的契约。如果 `install.sh` 说技术栈不完整，说明 VM 镜像早于它——应报告此 VM 无法完成安装；不要用 pip、npm 或浏览器下载临场发挥。
- **Dev machines** (no cell image) override each leg explicitly: `MM_NODE`,
  `MM_PLAYWRIGHT_MODULES`, `JARVIS_CHROMIUM_BINARY`, `MM_FFMPEG`.
  **开发机**（无 cell 镜像）需显式覆盖每条链路：`MM_NODE`、`MM_PLAYWRIGHT_MODULES`、`JARVIS_CHROMIUM_BINARY`、`MM_FFMPEG`。

## Updating / 更新

Update the bundled skill through the supported Jarvis deploy flow. Do not extract a separate skill copy into the user's workspace. The canonical runtime path is `/opt/hatch/skills/magic-moment/`.

通过受支持的 Jarvis 部署流程更新捆绑的技能。不要在用户工作区里另解出一份技能副本。规范运行时路径是 `/opt/hatch/skills/magic-moment/`。

The fast check establishes dependency presence. The full check exercises static capture. Transcription, animated capture, and the final audio/video build still require run-specific validation.

快速检查用于确认依赖存在。完整检查会实际演练静态截图。转写、动态截图和最终音视频合成仍需按具体运行场景单独验证。
