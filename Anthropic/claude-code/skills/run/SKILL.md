---
name: run
description: Launch and drive this project's app to see a change working. Use when asked to run, start, or screenshot the app, or to confirm a change works in the real app (not just tests). First looks for a project skill that already covers launching the app; otherwise falls back to built-in patterns per project type (CLI, server, TUI, Electron, browser-driven, library).
---
<!-- BILINGUAL-EN-ZH -->

**Running means launching the actual app and interacting with it** -
not the test suite, not an `import` of an internal function and a
`console.log`. The app as a user (human or programmatic) would meet
it: the CLI at its command, the server at its socket, the GUI at its
window.

**运行意味着启动真正的应用并与之交互**——不是测试套件，不是 `import` 一个内部函数再 `console.log` 一下。而是以用户（人类或程序）会接触到的方式面对应用：在命令行处面对 CLI，在套接字处面对服务器，在窗口处面对 GUI。

## First: does a project skill already cover this? / 第一步：项目技能是否已覆盖此事？

A project skill that launches this app is the repo's verified path -
its author already cold-started from a Linux container and committed
what worked: the exact `apt-get` line, the env vars, the patches, the
driver. Use it instead of rediscovering.

启动本应用的项目技能是仓库中经过验证的路径——其作者已经从 Linux 容器完成过冷启动，并提交了验证可行的内容：确切的 `apt-get` 命令行、环境变量、补丁、驱动。直接用它，不要重新摸索。

```bash
d=$PWD; while :; do
  grep -Hm1 '^description:' "$d"/.claude/skills/*/SKILL.md 2>/dev/null
  [ -e "$d/.git" ] || [ "$d" = / ] && break
  d=$(dirname "$d")
done
```

- **One describes launching/driving this app** -> read that SKILL.md
  and follow it verbatim. Don't paraphrase; don't skip the patches.
  - **其中一份描述了启动/驱动本应用** -> 读取那份 SKILL.md 并逐字遵循。不要转述；不要跳过补丁。
- **Mega-repo, several plausible, no clear match** -> ask the user
  which unit to run.
  - **巨型仓库、多个候选、没有明确匹配** -> 询问用户要运行哪个单元。
- **Stale** (fails on mechanics unrelated to your task) -> tell the
  user; offer to refresh it via `/run-skill-generator`.
  - **已过时**（在与任务无关的机械步骤上失败） -> 告知用户；提议通过 `/run-skill-generator` 刷新它。
- **Nothing about running** -> fall back to the patterns below.
  - **没有涉及运行的内容** -> 回退到下方的模式。

## Otherwise: match the shape, use the pattern / 否则：匹配形态，套用模式

Pick the row closest to your project. Each example walks through
launch + first interaction; ignore any trailing "write the skill"
section - you're using the recipe, not authoring one.

选与你的项目最接近的一行。每个示例都完整走过启动 + 首次交互的流程；忽略末尾任何"编写技能"小节——你是在使用配方，不是在编写配方。

| Project type | Handle | Example |
|---|---|---|
| CLI tool | direct invocation, exit code, stdin/stdout | [examples/cli.md](examples/cli.md) |
| Web server / API | background launch + `curl` smoke | [examples/server.md](examples/server.md) |
| TUI / interactive terminal | tmux `send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / desktop GUI | Playwright `_electron` REPL under xvfb | [examples/electron.md](examples/electron.md) |
| Browser-driven | dev server + `chromium-cli` script | [examples/playwright.md](examples/playwright.md) |
| Library / SDK | import-and-call smoke script at the package boundary | [examples/library.md](examples/library.md) |

| Project type / 项目类型 | Handle / 处理方式 | Example / 示例 |
|---|---|---|
| CLI 工具 | 直接调用、退出码、stdin/stdout | [examples/cli.md](examples/cli.md) |
| Web 服务器 / API | 后台启动 + `curl` 冒烟 | [examples/server.md](examples/server.md) |
| TUI / 交互式终端 | tmux `send-keys` / `capture-pane` | [examples/tui.md](examples/tui.md) |
| Electron / 桌面 GUI | xvfb 下的 Playwright `_electron` REPL | [examples/electron.md](examples/electron.md) |
| 浏览器驱动 | 开发服务器 + `chromium-cli` 脚本 | [examples/playwright.md](examples/playwright.md) |
| 库 / SDK | 在包边界做 import 并调用的冒烟脚本 | [examples/library.md](examples/library.md) |

If nothing fits, start from the closest match and adapt. For a web
app, [examples/playwright.md](examples/playwright.md) - drive it with
`chromium-cli`, no custom driver needed. For a desktop app,
[examples/electron.md](examples/electron.md) - it has the `_electron`
REPL driver skeleton and the tmux wrapping.

如果没有合适的，就从最接近的匹配入手再做调整。Web 应用参见 [examples/playwright.md](examples/playwright.md)——用 `chromium-cli` 驱动，无需自定义驱动。桌面应用参见 [examples/electron.md](examples/electron.md)——那里有 `_electron` REPL 驱动骨架和 tmux 包装方法。

## Drive it, don't just launch it / 要驱动它，而不是仅仅启动它

Launching with no interaction proves the entrypoint resolves. That's
not running the app - it's typechecking with extra steps. Drive it to
a point where a user would see something:

不做任何交互的启动只能证明入口点可以解析。那不是运行应用——只是多绕几步的类型检查。要把它驱动到用户能看到某些东西的程度：

- CLI -> type a representative command, check the exit code and output.
  - CLI -> 输入一条有代表性的命令，检查退出码和输出。
- Server -> hit the route the diff touches with `curl`, read the body.
  - 服务器 -> 用 `curl` 请求 diff 触及的路由，读取响应体。
- TUI -> `send-keys` a navigation, `capture-pane` the result.
  - TUI -> 用 `send-keys` 发送一次导航，用 `capture-pane` 捕获结果。
- GUI -> click the button, screenshot the window. **Look at the
  screenshot.** A blank frame is a failure to launch.
  - GUI -> 点击按钮，截取窗口。**要看那张截图。**空白画面就是启动失败。

If the fallback pattern didn't work out of the box - you had to
install packages, set env vars, patch config, or write a driver -
recommend `/run-skill-generator` in your report so that work gets
captured as a project skill. If it just worked, don't.

如果回退模式不是开箱即用——你不得不安装软件包、设置环境变量、修改配置或编写驱动——就在报告中推荐 `/run-skill-generator`，让这些工作沉淀为项目技能。如果直接就跑通了，就不必推荐。
