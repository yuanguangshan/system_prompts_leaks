<!-- BILINGUAL-EN-ZH -->
# Example: Electron / desktop GUI app / 示例：Electron / 桌面 GUI 应用

Electron apps have a window. A future agent in a headless container can't see a window. So your deliverable here is not a markdown file that says "`npm start` opens a window" - it's a **driver script** that launches the app under xvfb, exposes a REPL of commands (click, type, screenshot), and lets an agent poke the UI by sending lines of text.

Electron 应用有窗口，而未来在无头容器中的智能体看不到窗口。因此你在这里的交付物不是一个写着"`npm start` 会打开一个窗口"的 markdown 文件，而是一个**驱动脚本**：它在 xvfb 下启动应用，暴露一组 REPL 命令（click、type、screenshot），并让智能体通过发送文本行来操作 UI。
【评论】该指导把交付物从"描述性文档"重定义为"可执行工具"，使后续智能体获得对 GUI 的程序化操作接口。

The skill's `SKILL.md` then becomes a short manual for that driver.

此时，本技能的 `SKILL.md` 就成为该驱动程序的简短手册。

## What you're building / 你要构建什么

```
apps/desktop/
  .claude/skills/run-desktop/
    SKILL.md               <- short. "run the driver, here are the commands"
    driver.mjs             <- REPL: stdin commands -> Playwright actions
```

The driver IS the product. Without it, the skill describes a GUI an agent can never touch.

驱动程序才是产品本身。没有它，这个技能描述的只是一个智能体永远无法触碰的 GUI。

**Graduation path:** if the driver grows launch helpers the project's real e2e suite wants to share, move it to `e2e-playwright/driver.mjs` (or `scripts/drive.mjs`) and update the skill's paths. The skill stays at `.claude/skills/run-desktop/`; the driver finds a better home.

**进阶路径：**如果驱动程序长出了项目真实 e2e 套件也想复用的启动辅助函数，就把它移到 `e2e-playwright/driver.mjs`（或 `scripts/drive.mjs`）并更新技能中的路径。技能仍留在 `.claude/skills/run-desktop/`；驱动程序则搬去更合适的家。

## Step 1 - get the app to launch AT ALL under xvfb / 第 1 步 - 让应用至少能在 xvfb 下启动

This is usually the hardest part and produces most of the Gotchas. The README will say "macOS/Windows only." Ignore that. Install xvfb + the Chromium shared libs, find the Electron binary, and launch it:

这通常是最难的部分，也是 Gotchas（陷阱清单）条目的主要来源。README 会写着"仅支持 macOS/Windows"。忽略这句话。安装 xvfb 和 Chromium 共享库，找到 Electron 二进制文件，然后启动它：

```bash
apt-get install -y xvfb libnss3 libgbm1 libasound2t64 libgtk-3-0 \
  libxss1 libxkbcommon0 libatk-bridge2.0-0 libcups2 libdrm2

# Build the app first. Often the "dev" script is electron-forge which
# does a Vite/webpack build THEN launches. You want just the build:
npm install
npx electron-forge start &   # builds .vite/build/ or dist/
sleep 20 && kill %1          # kill it once built - you'll launch yourself

# Now try the raw launch
xvfb-run -a node -e "
  const { _electron } = require('playwright-core');
  _electron.launch({
    executablePath: './node_modules/electron/dist/electron',
    args: ['--no-sandbox', '.'],
    timeout: 30000,
  }).then(app => {
    console.log('launched, windows:', app.windows().map(w => w.url()));
    return app.close();
  });
"
```

Iterate until it launches. Each missing `.so` -> one more `apt-get` package -> one more line in Prerequisites. Each launch timeout -> check the `nodeCliInspect` fuse isn't disabled, check the build output exists.

反复迭代直到它能启动。每缺一个 `.so` 就多装一个 `apt-get` 包，并在前置条件中多写一行。每次启动超时，就检查 `nodeCliInspect` 熔丝未被禁用、构建产物存在。

**`--no-sandbox` is almost always needed in containers.** Electron's sandbox needs CAP_SYS_ADMIN or user namespaces. Neither by default.

**容器中几乎总是需要 `--no-sandbox`。**Electron 的沙箱需要 CAP_SYS_ADMIN 或用户命名空间，而容器默认两者都没有。

## Step 2 - build the REPL driver / 第 2 步 - 构建 REPL 驱动程序

Once you can launch it, turn that throwaway script into a REPL. Start minimal - you will add commands as you need them. **The REPL is the right shape** because an agent can run it inside tmux and iterate without relaunching the (slow) app on every interaction.

一旦能启动应用，就把那个一次性脚本改造成 REPL。从最简开始——需要的命令以后再添加。**REPL 是正确的形态**，因为智能体可以在 tmux 中运行它并持续迭代，而不必每次交互都重新启动（缓慢的）应用。

```javascript
// .claude/skills/run-<unit>/driver.mjs
// REPL driver for <app>. Run under xvfb on headless Linux.
// Designed for agents: wrap in tmux, send-keys commands, capture-pane output.
import { _electron as electron } from 'playwright-core';
import * as readline from 'node:readline';
import * as fs from 'node:fs';
import * as path from 'node:path';

const APP_DIR = path.resolve(import.meta.dirname, '../../..');
const SHOT_DIR = process.env.SCREENSHOT_DIR || '/tmp/shots';
fs.mkdirSync(SHOT_DIR, { recursive: true });

let app = null;
let page = null;   // the window/page you actually interact with

const electronBin = process.platform === 'darwin'
  ? path.join(APP_DIR, 'node_modules/electron/dist/Electron.app/Contents/MacOS/Electron')
  : path.join(APP_DIR, 'node_modules/electron/dist/electron');

const COMMANDS = {
  async launch() {
    if (app) return console.log('already launched');
    app = await electron.launch({
      executablePath: electronBin,
      args: ['--no-sandbox', APP_DIR],
      env: { ...process.env, DISPLAY: process.env.DISPLAY || ':99' },
      timeout: 30_000,
    });
    // Electron has no clean "loaded" signal - this sleep is a blind guess.
    // Replace with a poll once you know what ready looks like for this app:
    // wait until windows() includes the expected URL, or waitForSelector on firstWindow().
    await new Promise(r => setTimeout(r, 8_000));
    // Find the real UI page. Often NOT firstWindow() - may be a
    // splash screen, or the real content is in a BrowserView overlay.
    page = app.windows().find(w => !w.url().startsWith('devtools://'))
        ?? await app.firstWindow();
    console.log('launched.', app.windows().length, 'windows:');
    for (const w of app.windows()) console.log(' ', w.url());
  },

  async ss(name) {
    if (!page) return console.log('ERROR: launch first');
    const f = path.join(SHOT_DIR, (name || `ss-${Date.now()}`) + '.png');
    await page.screenshot({ path: f });
    console.log('screenshot:', f);
  },

  // Click via evaluate(), NOT locator.click(). If the content lives in a
  // BrowserView layered over the main window, Playwright's coordinate
  // math hits the wrong layer. DOM .click() always works.
  async click(sel) {
    if (!page) return console.log('ERROR: launch first');
    const r = await page.evaluate(s => {
      const el = document.querySelector(s);
      if (!el) return 'NOT_FOUND';
      el.click(); return 'OK';
    }, sel);
    console.log('click', sel, '->', r);
  },

  async 'click-text'(text) {
    if (!page) return console.log('ERROR: launch first');
    const r = await page.evaluate(t => {
      const els = [...document.querySelectorAll('button, a, [role="button"]')];
      const el = els.find(e => e.textContent?.trim() === t)
              ?? els.find(e => e.textContent?.includes(t));
      if (!el) return 'NOT_FOUND';
      el.click(); return 'OK: ' + el.tagName;
    }, text);
    console.log('click-text', JSON.stringify(text), '->', r);
  },

  async type(text)  { if (page) await page.keyboard.type(text, { delay: 30 }); },
  async press(key)  { if (page) await page.keyboard.press(key); },

  async wait(sel) {
    if (!page) return console.log('ERROR: launch first');
    try { await page.waitForSelector(sel, { timeout: 10_000 }); console.log('found:', sel); }
    catch { console.log('TIMEOUT:', sel); }
  },

  async eval(expr) {
    if (!page) return console.log('ERROR: launch first');
    try { console.log(JSON.stringify(await page.evaluate(expr))); }
    catch (e) { console.log('ERROR:', e.message); }
  },

  async text(sel) {
    if (!page) return console.log('ERROR: launch first');
    console.log(await page.evaluate(
      s => (s ? document.querySelector(s) : document.body)?.innerText ?? '(null)',
      sel || null));
  },

  // Introspection: essential for figuring out which window/webContents
  // actually has the UI. Electron apps often spawn several.
  async windows() {
    if (!app) return console.log('ERROR: launch first');
    for (const w of app.windows()) console.log(' ', w.url());
    const wcs = await app.evaluate(({ webContents }) =>
      webContents.getAllWebContents().map(w => ({ id: w.id, type: w.getType(), url: w.getURL() })));
    console.log('webContents:');
    for (const w of wcs) console.log(` [${w.id}] ${w.type}: ${w.url}`);
  },

  async quit() { if (app) await app.close().catch(()=>{}); app = null; page = null; },
  help() { console.log('commands:', Object.keys(COMMANDS).join(', ')); },
};

// Stop Electron from stealing stdin - use the raw fd.
const stdin = fs.createReadStream(null, { fd: fs.openSync('/dev/stdin', 'r') });
const rl = readline.createInterface({ input: stdin, output: process.stdout, prompt: 'driver> ' });

rl.on('line', async line => {
  const [cmd, ...rest] = line.trim().split(/\s+/);
  if (!cmd) return rl.prompt();
  const fn = COMMANDS[cmd];
  if (!fn) { console.log('unknown:', cmd, ' - try: help'); return rl.prompt(); }
  try { await fn(rest.join(' ')); } catch (e) { console.log('ERROR:', e.message); }
  if (cmd === 'quit') { rl.close(); process.exit(0); }
  rl.prompt();
});
rl.on('close', async () => { await COMMANDS.quit(); process.exit(0); });

console.log('<app> driver - "help" for commands, "launch" to start');
rl.prompt();
```

**This is a starting skeleton.** As you try to reach interesting parts of the app you'll add app-specific commands: navigate to a particular view, focus a weird input type, bypass an auth gate, whatever. Those commands encode hard-won knowledge - keep them.

**这只是一个起始骨架。**在你尝试触达应用中有意思的部分时，会逐渐添加应用专属命令：导航到某个特定视图、聚焦某种怪异的输入类型、绕过某个鉴权门槛，等等。这些命令承载着来之不易的知识——保留它们。

## Step 3 - use it yourself, via tmux / 第 3 步 - 自己先用起来：通过 tmux

Run the driver the same way the next agent will:

按照下一个智能体的方式运行这个驱动程序：

```bash
tmux new-session -d -s app -x 200 -y 50
tmux send-keys -t app 'cd /workspace/apps/desktop && xvfb-run -a node .claude/skills/run-desktop/driver.mjs' Enter
timeout 20 bash -c 'until tmux capture-pane -t app -p | grep -q "driver>"; do sleep 0.2; done'
tmux send-keys -t app 'launch' Enter
timeout 60 bash -c 'until tmux capture-pane -t app -p | grep -q "launched"; do sleep 0.2; done'
tmux send-keys -t app 'ss 01-landing' Enter
timeout 10 bash -c 'until tmux capture-pane -t app -p | grep -q "screenshot:"; do sleep 0.2; done'
tmux send-keys -t app 'windows' Enter    # which page has the real UI?
tmux capture-pane -t app -p
```

Then actually open `/tmp/shots/01-landing.png`. Is it the app? Is it blank? Is it a login screen? Each of these tells you what to do next.

然后真正打开 `/tmp/shots/01-landing.png` 看一眼。是应用界面吗？是空白吗？是登录页吗？每一种情况都会告诉你下一步该做什么。

Keep going - click into the main feature, fill a form, see the result show up, screenshot it. The driver grows whatever commands you need (`focus-input`, `goto-settings`, `login-as-test-user`...). When one real flow works end-to-end, you're done building and ready to write.

继续推进——点进主要功能，填一个表单，看到结果出现，截图记录。驱动程序会随需生长出你需要的命令（`focus-input`、`goto-settings`、`login-as-test-user`……）。当一条真实流程能端到端跑通时，构建就完成了，可以开始写作。

## Step 4 - write SKILL.md / 第 4 步 - 编写 SKILL.md

Keep it short. The driver is the meat; `SKILL.md` is the manual. Structure that works:

保持简短。驱动程序是主体；`SKILL.md` 是手册。可用的结构如下：

> ---
> name: run-desktop
> description: Build, run, and drive the `<app>` Electron desktop app. Use when asked to start the desktop app, take a screenshot of it, build it, or interact with its UI.
> 构建、运行并驱动 `<app>` Electron 桌面应用。当被要求启动桌面应用、对它截图、构建它或与其 UI 交互时使用。
> ---
>
> `<App>` is an Electron desktop app. For agent/automated use, drive it via the Playwright REPL at `.claude/skills/run-desktop/driver.mjs` under xvfb. Launch is slow (~10s) and the interesting UI lives in a BrowserView, not the main window - the driver handles both.
> `<App>` 是一个 Electron 桌面应用。对于智能体/自动化使用，在 xvfb 下通过 `.claude/skills/run-desktop/driver.mjs` 的 Playwright REPL 驱动它。启动较慢（约 10 秒），且有意思的 UI 位于 BrowserView 而非主窗口中——驱动程序对两者都做了处理。
>
> All paths are relative to `apps/desktop/`.
> 所有路径均相对于 `apps/desktop/`。
>
> ## Prerequisites / 前置条件
>
> ```bash
> apt-get install -y xvfb libnss3 libgbm1 libasound2t64 libgtk-3-0 \
>   libxss1 libxkbcommon0 libatk-bridge2.0-0 libcups2 libdrm2
> ```
>
> ## Build / 构建
>
> ```bash
> npm install
> npx electron-forge start   # builds .vite/build/ - Ctrl-C once built
> # `<any patch you had to apply: sed a feature gate, etc.>`
> ```
>
> ## Run (agent path) / 运行（智能体路径）
>
> ```bash
> cd apps/desktop
> xvfb-run -a node .claude/skills/run-desktop/driver.mjs
> ```
>
> Wrap in tmux for interactive use:
> 在 tmux 中包装以便交互使用：
>
> ```bash
> tmux new-session -d -s app -x 200 -y 50
> tmux send-keys -t app 'cd apps/desktop && xvfb-run -a node .claude/skills/run-desktop/driver.mjs' Enter
> timeout 20 bash -c 'until tmux capture-pane -t app -p | grep -q "driver>"; do sleep 0.2; done'
> tmux send-keys -t app 'launch' Enter
> timeout 60 bash -c 'until tmux capture-pane -t app -p | grep -q "launched"; do sleep 0.2; done'
> tmux send-keys -t app 'ss landing' Enter
> tmux capture-pane -t app -p
> ```
>
> Screenshots land in `/tmp/shots/` (override: `SCREENSHOT_DIR`).
> 截图保存在 `/tmp/shots/`（可通过 `SCREENSHOT_DIR` 覆盖）。
>
> ### Commands / 命令
>
> | command | what it does |
> |---|---|
> | `launch` | launch the app, wait for windows |
> | `ss [name]` | screenshot -> `/tmp/shots/<name>.png` |
> | `click <css-sel>` | click element (via DOM, not coords - see Gotchas) |
> | `click-text <text>` | click button/link containing text |
> | `type <text>` / `press <key>` | keyboard input |
> | `wait <css-sel>` | wait for element, 10s timeout |
> | `eval <js>` | evaluate in the page, print JSON |
> | `text [css-sel]` | print innerText |
> | `windows` | list all windows + webContents (find the real UI) |
> | `quit` | close app, exit |
>
> | 命令 | 作用 |
> |---|---|
> | `launch` | 启动应用，等待窗口出现 |
> | `ss [name]` | 截图 -> `/tmp/shots/<name>.png` |
> | `click <css-sel>` | 点击元素（通过 DOM 而非坐标——见 Gotchas） |
> | `click-text <text>` | 点击包含指定文本的按钮/链接 |
> | `type <text>` / `press <key>` | 键盘输入 |
> | `wait <css-sel>` | 等待元素出现，超时 10 秒 |
> | `eval <js>` | 在页面中求值并打印 JSON |
> | `text [css-sel]` | 打印 innerText |
> | `windows` | 列出所有窗口与 webContents（找到真正的 UI） |
> | `quit` | 关闭应用并退出 |
>
> Plus any app-specific commands you built: `<your-command>` - `<what it does>`.
> 外加你构建的任何应用专属命令：`<your-command>` - `<它的作用>`。
>
> ## Run (human path) / 运行（人类路径）
>
> ```bash
> npm start   # opens a window; useless headless. Ctrl-C to quit.
> ```
>
> ## Gotchas / 陷阱
>
> - **`<the specific weird thing you hit>`** - `<why>` -> <fix/workaround>
>   `<你实际遇到的怪异问题>` - `<原因>` -> <修复/变通办法>
> - `<etc. - only things you actually hit, not generic advice>`
>   `<等等——只写你实际遇到的问题，而不是泛泛的建议>`
>
> ## Troubleshooting / 故障排查
>
> - **Launch timeout (30s):** build output missing? -> re-run the build
>   step. `nodeCliInspect` fuse disabled? -> Playwright can't attach;
>   don't disable that fuse in dev builds.
>   **启动超时（30 秒）：**构建产物缺失？-> 重新运行构建步骤。`nodeCliInspect` 熔丝被禁用？-> Playwright 无法附加；不要在开发构建中禁用该熔丝。
> - **"Missing X server":** forgot `xvfb-run`. Headless Linux needs it.
>   **"Missing X server"：**忘了用 `xvfb-run`。无头 Linux 需要它。
> - **Stale Xvfb locks:** `rm -f /tmp/.X*-lock; pkill Xvfb`
>   **Xvfb 锁残留：**`rm -f /tmp/.X*-lock; pkill Xvfb`
> - `<anything else you actually hit>`
>   `<你实际遇到的其他任何问题>`

## Obstacles you will hit (and they go in Gotchas) / 你会遇到的障碍（它们应写进 Gotchas）

These are real patterns from real Electron apps. You'll hit some subset:

这些是来自真实 Electron 应用的真实模式。你会遇到其中一部分：

- **`firstWindow()` gives you a splash/loading screen,** not the app.
  Wait longer, or find the right page by URL, or wait for a specific
  selector that only appears when the app is actually ready.
  **`firstWindow()` 给你的是启动页/加载页，**而不是应用本身。多等一会儿，或按 URL 找到正确的页面，或等待一个只有应用真正就绪时才会出现的特定选择器。

- **The real UI is in a BrowserView, not a BrowserWindow.** Playwright
  sees it as a separate "window" with a different URL. The `windows`
  command exists exactly for figuring this out. `getBrowserViews()`
  may also return empty on newer Electron - use
  `webContents.getAllWebContents()` instead.
  **真正的 UI 位于 BrowserView 而非 BrowserWindow 中。**Playwright 会把它当作一个 URL 不同的独立"窗口"。`windows` 命令正是为弄清这一点而存在的。在较新的 Electron 上 `getBrowserViews()` 也可能返回空——请改用 `webContents.getAllWebContents()`。

- **`locator.click()` clicks the wrong thing.** Playwright computes
  click coordinates relative to the main window. If your content is in
  a BrowserView overlay, those coordinates hit the window behind it.
  The driver skeleton uses `page.evaluate(el => el.click())` for this
  reason - DOM click bypasses coordinates entirely.
  **`locator.click()` 会点错目标。**Playwright 相对主窗口计算点击坐标。如果你的内容位于覆盖在主窗口之上的 BrowserView 中，这些坐标会点中它背后的窗口。出于这个原因，驱动骨架使用 `page.evaluate(el => el.click())`——DOM 点击完全绕开坐标系。
  【评论】绕过坐标点击是针对 Playwright 对 BrowserView 层级坐标计算缺陷的一种已知变通做法，而非通用最佳实践。

- **Feature gates block the thing you need to test.** The app checks a
  plan tier, or an env flag, or a feature flag baked into SSR HTML.
  Find where the check happens (grep the built output for the gate
  name) and patch it for your local run - a `sed` on the build output,
  an env var override, or (for SSR-embedded flags) intercept the
  response via CDP `Fetch.enable` and rewrite it in-flight. Document
  exactly what you patched and why.
  **功能开关会挡住你要测试的功能。**应用可能检查套餐档位、环境变量，或烘焙进 SSR HTML 的功能开关。找到检查发生的位置（在构建产物中 grep 开关名称），并为本地运行打补丁——对构建产物做一次 `sed`、一个环境变量覆盖，或（对嵌入 SSR 的开关）通过 CDP 的 `Fetch.enable` 拦截响应并在传输中改写。把打了什么补丁、为什么打，准确记录下来。

- **contentEditable inputs** (ProseMirror, Tiptap, Slate) aren't
  `<textarea>`. `fill()` won't work. Focus the element, then use
  `keyboard.type()`. Add a `focus <sel>` command if the app has these.
  **contentEditable 输入框**（ProseMirror、Tiptap、Slate）不是 `<textarea>`。`fill()` 不起作用。先聚焦元素，再用 `keyboard.type()`。如果应用有这类输入框，添加一个 `focus <sel>` 命令。

- **Electron steals stdin.** The `fs.openSync('/dev/stdin', 'r')` +
  `createReadStream` trick in the skeleton protects your REPL's input.
  **Electron 会抢占 stdin。**骨架中的 `fs.openSync('/dev/stdin', 'r')` + `createReadStream` 技巧保护了你 REPL 的输入。

- **Native modules fail to load** (keychain, notifications, etc.).
  Usually non-fatal - the core app runs, those features no-op. Note it
  and move on.
  **原生模块加载失败**（钥匙串、通知等）。通常不致命——核心应用照常运行，那些功能空转。记下来，继续推进。
