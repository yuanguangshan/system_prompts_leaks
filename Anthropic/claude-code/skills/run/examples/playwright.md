<!-- BILINGUAL-EN-ZH -->
# Example: Browser-driven web app / 示例：由浏览器驱动的 Web 应用

You have a dev server that serves HTML to a browser. An agent in a
headless container can't open a browser window - so "run the app" means
launching the dev server, driving a headless Chromium against it, and
producing a screenshot that proves the page rendered.

你有一个向浏览器提供 HTML 的开发服务器。无头容器中的 agent 无法打开浏览器窗口——因此"运行这个应用"意味着：启动开发服务器，用无头 Chromium 对其进行驱动，并产出一张能证明页面已渲染的截图。

Don't write a browser driver. Use `chromium-cli`.

不要自己写浏览器驱动。使用 `chromium-cli`。

## Dev server / 开发服务器

Find the dev command (`package.json` `scripts.dev`, `Makefile`,
README), start it in the background, and wait for it to actually serve:

找到开发命令（`package.json` 的 `scripts.dev`、`Makefile`、README），后台启动它，并等待它真正开始提供服务：

```bash
npm run dev &   # or yarn dev, pnpm dev, make serve, ./dev.sh
timeout 30 bash -c 'until curl -sf http://localhost:3000 >/dev/null; do sleep 1; done'
```

Don't `sleep 5` - poll the port. Stop by killing the port's listener
-- `lsof -ti:3000 -sTCP:LISTEN | xargs -r kill` - before relaunching,
or the next run hits `EADDRINUSE`. (`$!` after `npm run dev &` is only
the npm wrapper; npm doesn't forward SIGTERM to the server it spawned,
so the port kill is what actually frees it.) Avoid `pkill -f` with a
broad pattern - it can match the agent's own command line and kill the
session.

不要 `sleep 5`——要轮询端口。重启之前，先杀掉占用端口的监听进程来停止服务——`lsof -ti:3000 -sTCP:LISTEN | xargs -r kill`——否则下一次运行会撞上 `EADDRINUSE`。（`npm run dev &` 之后的 `$!` 只是 npm 包装进程；npm 不会把 SIGTERM 转发给它派生的服务器，所以杀端口才真正释放它。）避免把 `pkill -f` 配宽泛模式使用——它可能匹配到 agent 自己的命令行，把会话杀掉。

## Drive / 驱动

`chromium-cli` is a headless-Chromium REPL. Pipe a script to stdin:

`chromium-cli` 是一个无头 Chromium 的 REPL。把脚本通过管道送入 stdin：

```bash
chromium-cli --session app <<'EOF'
nav http://localhost:3000
wait-for text=Dashboard
screenshot
click button:has-text("New item")
fill input[name="title"] Smoke test
press Enter
wait-for text=Smoke test
screenshot
console --errors
EOF
```

Screenshots land in `chromium_cli/sessions/app/screenshots/` (latest
symlinked as `screenshot.png`). That's the whole loop: `nav` ->
`wait-for` the element you need -> act (`click` / `fill` / `type` /
`press`) -> `screenshot` -> `console --errors` to check nothing threw.
Full command reference: `chromium-cli` skill, or `help` at the prompt.

截图落在 `chromium_cli/sessions/app/screenshots/`（最新一张以符号链接 `screenshot.png` 指向）。整个循环就是：`nav` -> `wait-for` 你需要的元素 -> 操作（`click` / `fill` / `type` / `press`）-> `screenshot` -> `console --errors` 检查没有抛出异常。完整命令参考见 `chromium-cli` 技能，或在提示符下输入 `help`。

For iterative debugging, run it under tmux and `send-keys` one command
at a time - same commands, same session.

要做迭代式调试，可在 tmux 下运行，并用 `send-keys` 一次发送一条命令——命令相同，会话相同。

**If `chromium-cli` isn't available:** adapt
[electron.md](electron.md)'s REPL driver - the structure and commands
transfer, but it's `_electron`-specific:
import `{ chromium }` instead, launch with
`chromium.launch({ args: ['--no-sandbox'] })`, acquire the page via
`(await app.newContext()).newPage()` then `goto()` your dev URL, and
drop the Electron-only window introspection
(`.windows()`/`.firstWindow()`/the `windows` command).

**如果 `chromium-cli` 不可用：**改造 [electron.md](electron.md) 的 REPL 驱动——结构和命令可以照搬，但那份代码是 `_electron` 专用的：改为 import `{ chromium }`，用 `chromium.launch({ args: ['--no-sandbox'] })` 启动，通过 `(await app.newContext()).newPage()` 获取页面再 `goto()` 你的开发 URL，并去掉 Electron 专属的窗口内省（`.windows()`/`.firstWindow()`/`windows` 命令）。

## What to put in the skill / 技能中应写什么

The project-specific bits only. `chromium-cli` handles the mechanics.

只写项目特有的部分。机械操作交给 `chromium-cli`。

- **Dev command + port + stop.** The exact start line, any env vars it
  needs, and the `kill` to stop it.
  - **开发命令 + 端口 + 停止。**确切的启动命令行、所需的环境变量，以及用于停止的 `kill`。
- **Auth.** Whatever gets a logged-in session - a `set-cookie` line, a
  `fill`/`click` login sequence, or a helper script that does the API
  dance and emits the cookie.
  - **认证。**任何能拿到已登录会话的手段——一行 `set-cookie`、一组 `fill`/`click` 登录序列，或一个完成 API 流程并产出 cookie 的辅助脚本。
- **One representative interaction.** Not the whole app - one path that
  proves it's running, ending in a screenshot.
  - **一个有代表性的交互。**不是整个应用——一条能证明它跑起来了的路径，以一张截图收尾。
- **App-specific gotchas.** Only the ones you actually hit.
  - **应用特有的坑。**只写你真正踩到的那些。

## Gotchas that recur / 反复出现的坑

- **React controlled inputs.** `eval el.value = '...'` doesn't fire
  React's onChange. Use `fill` / `type` - they go through Playwright's
  input pipeline.
  - **React 受控输入。**`eval el.value = '...'` 不会触发 React 的 onChange。使用 `fill` / `type`——它们走 Playwright 的输入管线。
- **Websockets / long-poll.** `wait-idle` never settles. `wait-for` the
  element you actually need.
  - **Websocket / 长轮询。**`wait-idle` 永远等不到空闲。`wait-for` 你真正需要的那个元素。
- **Slow first paint.** Vite/Next compile routes on demand; the first
  `nav` can take 10s+. `wait-for` handles it; raw `sleep` doesn't.
  - **首次绘制慢。**Vite/Next 按需编译路由；第一次 `nav` 可能耗时 10 秒以上。`wait-for` 能应对；裸 `sleep` 不行。
- **`screenshot-element <sel>`** crops to one element - use it when the
  diff is in a specific component, not the whole page.
  - **`screenshot-element <sel>`** 只截取单个元素——当差异出在某个具体组件而非整页时使用它。
- **Check `console --errors` before declaring success.** A page can
  render its shell while every data fetch 500s.
  - **宣告成功之前先检查 `console --errors`。**页面可以在外壳渲染正常的同时，让每一个数据请求都返回 500。
