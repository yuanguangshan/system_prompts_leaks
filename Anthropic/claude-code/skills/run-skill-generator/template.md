---
name: run-<unit-name>
description: Build, run, and drive <unit-name>. Use when asked to start <unit-name>, run its tests, build it, take a screenshot of its UI, or interact with the running app.
---
<!-- BILINGUAL-EN-ZH -->

<One-sentence description: what this is and how an agent drives it.
Name the handle here - "drive it via
`.claude/skills/run-<unit-name>/driver.mjs` under xvfb" for a desktop
app, or "start the dev server then drive it via `chromium-cli`" for a
web app - so an agent knows where to look first.>

<一句话描述：这是什么，代理如何驱动它。
在这里写明操作入口——桌面应用写"在 xvfb 下通过
`.claude/skills/run-<unit-name>/driver.mjs` 驱动"，
web 应用写"启动开发服务器后通过 `chromium-cli` 驱动"——
让代理知道首先该看哪里。>

`<If the unit isn't at repo root:>`
<若该单元不在仓库根目录：>
All paths below are relative to `<unit-dir>/`.

以下所有路径均相对于 `<unit-dir>/`。

## Prerequisites / 前提条件

<System-level requirements. The exact `apt-get install` line you ran -
not a generic list, the one that actually worked. Target Ubuntu.>

<系统级依赖。写出你实际运行的那条 `apt-get install` 命令——
不是通用清单，而是真正生效的那一条。目标系统为 Ubuntu。>

```bash
sudo apt-get update
sudo apt-get install -y <packages-you-actually-installed>
```

`<Runtime versions if they matter:>`
<若运行时版本有影响：>

```bash
# Example: Node 20 via nvm, Python 3.12 via uv, etc.
```

## Setup / 安装配置

`<One-time setup after clone: install deps, configure, apply any
patches (feature-gate overrides, config stubs) with the exact command.>`
<克隆后的一次性设置：安装依赖、配置、以精确命令应用
所有补丁（特性开关覆盖、配置桩）。>

```bash
<commands>
```

`<Env vars - required vs optional, with sensible defaults:>`
<环境变量——区分必填与可选，并给出合理默认值：>

```bash
export FOO_API_KEY=...   # required - get from <where>
export BAR_MODE=dev      # optional - default is prod
```

## Build / 构建

`<Skip if no separate build step. Otherwise the exact command:>`
<如无独立构建步骤则省略本节。否则写精确命令：>

```bash
<command>
```

## Run (agent path) / 运行（代理路径）

<This is the section a future agent actually uses. If you built a
driver/REPL/smoke script, this documents how to launch it and what it
does. If the app is simple enough that `curl` or a one-liner suffices,
that one-liner goes here.>

<这是未来的代理真正会使用的章节。如果你写了
driver/REPL/冒烟脚本，这里要写明如何启动它、它做什么。
如果应用简单到 `curl` 或一行命令即可，那一行命令就写在这里。>

```bash
<launch-the-driver-or-smoke-script>
```

`<For REPL-style drivers, show the tmux wrapping. Poll for a ready marker
between send-keys and capture-pane - faster than a fixed sleep and fails
loudly instead of capturing a half-rendered screen:>`
<对于 REPL 式 driver，展示 tmux 包装方式。在 send-keys 与
capture-pane 之间轮询就绪标记——比固定 sleep 更快，
且会明确报错而不是截取到渲染了一半的画面：>

```bash
tmux new-session -d -s app -x 200 -y 50
tmux send-keys -t app '<launch command>' Enter
timeout 30 bash -c 'until tmux capture-pane -t app -p | grep -q "<ready-marker>"; do sleep 0.2; done'
tmux send-keys -t app '<first driver command>' Enter
tmux capture-pane -t app -p
```

`<Where artifacts land (screenshots, logs) - absolute paths:>`
<产物（截图、日志）落盘位置——绝对路径：>

Screenshots -> `/tmp/shots/`. Logs -> `/tmp/<app>.log`.

截图 -> `/tmp/shots/`。日志 -> `/tmp/<app>.log`。

`<If the driver has commands, a table:>`
<若 driver 有命令，用表格：>

| command | what it does |
|---|---|
| `<cmd>` | `<description>` |

| 命令 | 作用 |
|---|---|
| `<cmd>` | `<描述>` |

## Run (human path) / 运行（人工路径）

`<If meaningfully different from the agent path. Brief - agents won't
use this, humans can figure it out.>`
<仅当与代理路径有实质差异时才写。要简短——代理不会用这一节，
人类自己能弄明白。>

```bash
<command>   # -> <what happens>. <how to stop>.
```

## Test / 测试

```bash
<command>
```

`<Expected result - "N suites pass", or specific known-flaky tests.>`
<预期结果——"N 个测试套件通过"，或指出特定的已知不稳定测试。>

---

`<Optional sections below - include only if relevant and only with
content you actually hit, not generic advice.>`
<以下是可选章节——仅在相关时收录，且只写你真正遇到的
内容，不要写泛泛之谈。>

## Gotchas / 陷阱

`<Non-obvious traps. The things that look like they should work but
don't, with the workaround. If this section is generic, delete it.>`
<不明显的坑。那些看起来应该可行实际却不行的事，
连同绕过办法一起写。如果这一节内容很平庸，就删掉它。>

- **`<specific thing>`** - `<why it breaks>` -> `<what to do instead>`
  **`<具体事物>`** - `<为何会坏>` -> `<应当怎么做>`

## Troubleshooting / 故障排查

`<Symptom ->` fix. Only errors you actually encountered.>

`<Symptom ->` 修复方法。只收录你实际遇到的错误。

- **`<exact error message or symptom>`**: `<cause>`. `<fix>`.
  **`<精确的报错信息或症状>`**：`<原因>`。`<修复方法>`。

<---

NOTE ON THE FRONTMATTER ABOVE:
关于上方 front matter 的说明：

- Replace <unit-name> in both `name:` and `description:`. The `name:`
  becomes the slash command (`/run-<unit-name>`) and must match the
  directory name.
- 将 `name:` 和 `description:` 中的 <unit-name> 都替换掉。`name:`
  会成为斜杠命令（`/run-<unit-name>`），必须与目录名一致。

- The `description:` is what Claude scans to decide whether to load this
  skill automatically. Keep the verbs - "start," "run," "build," "test,"
  "screenshot" - they're what an asking agent will actually type.
- `description:` 是 Claude 用来判断是否自动加载本技能的依据。
  保留那些动词——"start"、"run"、"build"、"test"、"screenshot"——
  它们正是发起询问的代理会实际输入的词。

NOTE ON THE DRIVER:
关于 driver 的说明：

- If you wrote a driver script, it lives in this same directory (next
  to this file) by default. Reference it from the Run section.
- 如果你写了 driver 脚本，默认它放在本目录（与本文件同级）。
  在 Run 章节中引用它。

- For a web app there's usually no driver file - the `chromium-cli`
  heredoc in the Run section is the harness.
- 对 web 应用通常没有 driver 文件——Run 章节中的 `chromium-cli`
  heredoc 就是测试装置。

- If the driver grows into something the project's test suite wants -
  shared launch helpers, a real e2e harness - move it to scripts/ or
  e2e/ in the unit, and update the paths here. The skill stays put.
- 如果 driver 成长为项目测试套件需要的东西——共享的启动辅助、
  真正的 e2e 装置——把它移到该单元的 scripts/ 或 e2e/ 中，
  并更新此处的路径。本技能文件保持在原地。

Delete everything from `---` above onwards before committing. --->

提交前删除从上面的 `---` 起及其后的全部内容。 --->
