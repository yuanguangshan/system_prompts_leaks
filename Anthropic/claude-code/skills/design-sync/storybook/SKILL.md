<!-- BILINGUAL-EN-ZH -->

# Storybook source shape / Storybook 源形态

Storybook is the **fidelity oracle, not the runtime**. The converter bundles the package's compiled `dist/` into `_ds_bundle.js` - the same bundle the claude.ai/design agent builds with - and generates each preview by **compiling the story source module itself** (hooks, fixtures, local helpers - the whole closure comes along), with every component import resolved to that shipped bundle (`lib/story-imports.mjs` redirects package *and* relative component imports to `window.<Global>`). The repo's own storybook render is the ground truth those previews must match: a compare harness screenshots each story in the reference storybook and the matching preview render side by side, and you iterate until they match. Nothing from storybook-static is uploaded, and no story code is ever evaluated at build time - stories run only in the browser, against the real artifact.

Storybook 是**保真度基准（oracle），而非运行时**。转换器把软件包编译后的 `dist/` 打包进 `_ds_bundle.js`——与 claude.ai/design 智能体构建所用的 bundle 相同——并通过**编译 story 源码模块本身**（hooks、fixtures、本地辅助函数——整个闭包一并带上）来生成每个预览，所有组件导入都解析到该随包 bundle（`lib/story-imports.mjs` 把包导入*和*相对路径组件导入都重定向到 `window.<Global>`）。仓库自身的 storybook 渲染是这些预览必须匹配的基准真值：一个比对工具会并排截取参考 storybook 中每个 story 的截图与对应预览的渲染截图，你持续迭代直到二者一致。storybook-static 的任何内容都不会上传，story 代码也绝不在构建时求值——story 只在浏览器中、针对真实制品运行。

【评论】该文档把 Storybook 定位为"保真度基准"而非运行时，并以截图比对作为核心验收手段，属于典型的视觉回归测试（visual regression）设计思路。

Requires React 18+. Playwright + chromium are **required** for this shape (the compare loop is the verification), not optional.

要求 React 18+。对于此形态，Playwright + chromium 是**必需**的（比对循环本身就是验证手段），并非可选项。

**First sync or re-sync?** A re-sync is marked by a config whose `projectId` and `pkg` were both in place before this run started - most of this document then doesn't apply; go to §7, where one driver run routes the work and untouched components cost nothing. Everything else takes the full flow (§2 build -> §3 self-heal -> §4 match -> conventions header (base SKILL.md, before upload) -> §6 upload), where every component gets verified and graded once - that includes a partial config left by an aborted run, and a pin this run itself just recorded in the base skill's §1. (Only the old `design-sync.config.json` present? Move it first and commit: `mkdir -p .design-sync && mv -n design-sync.config.json .design-sync/config.json`, then apply the same test.)

**首次同步还是重新同步？**重新同步的标志是：配置中的 `projectId` 与 `pkg` 在本次运行开始前就已就位——此时本文档大部分内容不再适用；转至 §7，在那里一次 driver 运行即可调度全部工作，未改动的组件零成本。其余情况都走完整流程（§2 构建 -> §3 自愈 -> §4 匹配 -> 约定头部（基础 SKILL.md，上传前）-> §6 上传），其中每个组件都要被验证并评分一次——这包括被中止运行留下的不完整配置，以及本次运行刚刚在基础技能 §1 中记录的 pin。（只存在旧的 `design-sync.config.json`？先移动并提交：`mkdir -p .design-sync && mv -n design-sync.config.json .design-sync/config.json`，然后再套用同一判据。）

## 2. Build, then run the converter / 构建，然后运行转换器

1. **Build the DS package *and its workspace dependencies*.** The converter bundles `dist/` into `window.<Global>`. Run `<pm> run build`; in a monorepo use `turbo run build --filter=<pkg>` or `pnpm -F "<pkg>..." build` (the trailing `...` is required - bare `-F <pkg>` skips dependencies and you'll see `Cannot find module '@scope/tokens'`). If `package.json` `module`/`exports['.']` points at TS source, find the actual built entry and pass it via `--entry`. **Do this before step 2** - storybook often imports sibling packages from their built `dist/`.

   1. **构建 DS 包*及其工作区依赖*。**转换器把 `dist/` 打包进 `window.<Global>`。运行 `<pm> run build`；在 monorepo 中使用 `turbo run build --filter=<pkg>` 或 `pnpm -F "<pkg>..." build`（末尾的 `...` 是必需的——裸 `-F <pkg>` 会跳过依赖，你会看到 `Cannot find module '@scope/tokens'`）。如果 `package.json` 的 `module`/`exports['.']` 指向 TS 源码，找到实际构建产物入口并通过 `--entry` 传入。**在第 2 步之前完成此步**——storybook 经常从兄弟包已构建的 `dist/` 导入。

2. **Build the reference storybook ONCE into `.design-sync/sb-reference/`** - NOT under `ds-bundle/` (the converter wipes `--out` on every rebuild, and storybook builds take minutes; the reference must survive the fix loop):

   2. **把参考 storybook 构建一次到 `.design-sync/sb-reference/`**——不要放在 `ds-bundle/` 下（转换器每次重建都会清空 `--out`，而 storybook 构建耗时数分钟；参考构建必须在修复循环中幸存）：

   ```bash
   npx storybook build -c <storybookConfigDir> -o .design-sync/sb-reference
   ```

   Run it from the directory whose `package.json` has the storybook devDependencies - usually the one containing `.storybook/`; monorepos often have several storybooks, so pick the one covering the package you're syncing. **Make `-o` the repo-root path** (e.g. `-o "$(git rev-parse --show-toplevel)/.design-sync/sb-reference"`): the converter and compare resolve `.design-sync/` from the repo root, so a cwd-relative `-o` in a subpackage puts the reference where nothing will find it. Use `npx storybook build` directly, **not** the repo's `npm run build-storybook` script (wrong output dir). Then check `.design-sync/sb-reference/iframe.html` exists and is >10KB - `index.json` alone can exist with a failed build.

   在 `package.json` 含有 storybook devDependencies 的目录中运行——通常是包含 `.storybook/` 的那个目录；monorepo 常有多个 storybook，应选择覆盖你正在同步的软件包的那一个。**把 `-o` 设为仓库根路径**（例如 `-o "$(git rev-parse --show-toplevel)/.design-sync/sb-reference"`）：转换器与比对工具都从仓库根解析 `.design-sync/`，因此在子包中使用相对 cwd 的 `-o` 会把参考构建放到无人能找到的位置。直接使用 `npx storybook build`，**不要**用仓库的 `npm run build-storybook` 脚本（输出目录不对）。然后检查 `.design-sync/sb-reference/iframe.html` 存在且大于 10KB——仅有 `index.json` 也可能出现在构建失败的情况下。

   Long builds: background them **through your shell tool's background mode only** and wait for the completion notification. Never a bare `&` (untracked - the notification never comes), and never a `pgrep -f '<script>'` poll loop (it matches its own command line and spins to timeout). Headless / `-p` sessions: run long commands synchronously instead - there is no task-notification re-invocation there, so a backgrounded run is never resumed.

   长时间构建：**仅通过你的 shell 工具的后台模式**将其置于后台，并等待完成通知。绝不使用裸 `&`（不受追踪——通知永远不会到来），也绝不使用 `pgrep -f '<script>'` 轮询循环（它会匹配到自身命令行并空转到超时）。Headless / `-p` 会话：改为同步运行长命令——那里没有任务通知的再次唤起机制，后台化的运行永远不会被恢复。

   `.gitignore` additions: `.design-sync/sb-reference/`, `.design-sync/learnings/`, `.design-sync/.cache/`, `.design-sync/node_modules` (fork symlink - recreated per clone), `.ds-sync/`, `ds-bundle/` - build artifact, transient scratch, verification working state, the symlink, staged scripts, regenerated output. Committed: the durable set (the rule in non-storybook §2, same here: everything under `.design-sync/` not gitignored - previews/ holds your authored files ONLY; generated story-module wrappers live in `.design-sync/.cache/previews/` and regenerate every build; the converter never writes or deletes anything in `previews/`). Verification state is never committed - cross-machine carry-forward comes from the uploaded project's `_ds_sync.json`. Rebuild the reference only when stories or the DS source change.

   `.gitignore` 新增条目：`.design-sync/sb-reference/`、`.design-sync/learnings/`、`.design-sync/.cache/`、`.design-sync/node_modules`（fork 符号链接——每次克隆时重建）、`.ds-sync/`、`ds-bundle/`——分别是构建产物、临时草稿、验证工作状态、符号链接、暂存脚本、再生成输出。需提交的：持久集合（与非 storybook 形态 §2 的规则相同，此处同理：`.design-sync/` 下所有未被 gitignore 的内容——previews/ 只存放你亲手编写的文件；生成的 story 模块包装器位于 `.design-sync/.cache/previews/`，每次构建都会重新生成；转换器从不在 `previews/` 中写入或删除任何东西）。验证状态永不提交——跨机器的携带传递来自已上传项目的 `_ds_sync.json`。仅当 story 或 DS 源码变化时才重建参考构建。

3. **Write `.design-sync/config.json`** - only `pkg` and `globalName` required. **If it already exists, read it first and keep what's there** - `titleMap`, `overrides`, and `provider` accumulate fixes from prior syncs. Also Read `.design-sync/NOTES.md` first - its **Re-sync risks** section is the prior run's watch-list; re-verify those items instead of assuming carry-forward covers them. The package-shape field table in `../non-storybook/SKILL.md` §2.6 applies verbatim; the fields that matter most here:

   3. **写入 `.design-sync/config.json`**——只有 `pkg` 和 `globalName` 必填。**如果它已存在，先读取并保留其中内容**——`titleMap`、`overrides` 和 `provider` 积累了此前多次同步的修复。同时先 Read `.design-sync/NOTES.md`——其 **Re-sync risks** 小节就是上一次运行的观察清单；要重新验证那些条目，而不是想当然地认为携带传递已覆盖它们。`../non-storybook/SKILL.md` §2.6 的包形态字段表在此逐字适用；此处最重要的字段：

   | Field | Value |
   |---|---|
   | `pkg` / `globalName` | `pkg` required; `globalName` auto-derived from it when omitted |
   | `shape` | `"storybook"` - pins detection |
   | `storybookStatic` | `".design-sync/sb-reference"` - so re-syncs and compare find the reference without flags |
   | `storybookConfigDir` | the `.storybook/` dir (monorepos) |
   | `buildCmd` | what to re-run before the converter on re-sync |
   | `titleMap` | `{title: ExportName}` when story titles don't match export names; `{title: null}` excludes a non-visual/internal component from the sync entirely |
   | `overrides` | `{<Name>: {skip: [storyIds], cardMode: "single"\|"column", primaryStory: "<Export>", viewport: "WxH"}}` - `skip` for stories that can't render statically; `cardMode: "single"` for overlay components (§4a.5, §5), `"column"` for stories wider than a grid cell (the `[GRID_OVERFLOW]` row in §3) |
   | `provider` | usually unnecessary for **previews** - `.storybook/preview` decorators are auto-bundled; set only when that fails. Before §6 upload, distill decorator-provided context into `cfg.provider` - README/prompt.md wrap guidance is generated from config only (decorator-only wrapping ships a generic note). **Setting it also replaces the decorators as the preview wrapper on the next build**: scoped-compare a themed component after the switch - an incomplete distillation regresses previews the decorators rendered fine, and carried-forward grades won't catch it. Format: `{"component": "ThemeProvider", "props": {...}, "inner": {...}}` - a nested chain, outermost first; each `component` must be a bundle export. Literal `props` are for small scalars (`"theme": "light"`) and stable snippets. For data that already exists in the repo - a locale JSON, a theme object - **prefer `{"$ref": "<export>"}`** backed by a 2-line module added via `cfg.extraEntries` (e.g. `export { default as previewI18n } from '../locales/en.json'`): a `$ref` emits `window.<Global>.<export>`, so the data lives once in the bundle and re-reads from its source file on every build. Inlining a copy is acceptable for something tiny and stable, but know the cost - a literal duplicates into every card's html and silently rots when the source file changes, so anything sizable or evolving belongs behind a `$ref`. Path forms for `extraEntries`: a bare name resolves from `node_modules`; a repo-owned module needs an explicit `./`/`../` package-relative path (workspace-bounded - the build logs `! extraEntries: ... skipped` if it escapes). |

   | 字段 | 取值 |
   |---|---|
   | `pkg` / `globalName` | `pkg` 必填；省略时 `globalName` 由它自动推导 |
   | `shape` | `"storybook"`——固定检测形态 |
   | `storybookStatic` | `".design-sync/sb-reference"`——使重新同步与比对无需标志即可找到参考构建 |
   | `storybookConfigDir` | `.storybook/` 目录（monorepo 场景） |
   | `buildCmd` | 重新同步时在转换器之前要重跑什么 |
   | `titleMap` | story 标题与导出名不匹配时用 `{title: ExportName}`；`{title: null}` 把非视觉/内部组件从同步中完全排除 |
   | `overrides` | `{<Name>: {skip: [storyIds], cardMode: "single"\|"column", primaryStory: "<Export>", viewport: "WxH"}}`——`skip` 用于无法静态渲染的 story；覆盖型组件用 `cardMode: "single"`（§4a.5、§5），宽于一个网格单元的 story 用 `"column"`（§3 的 `[GRID_OVERFLOW]` 行） |
   | `provider` | 对**预览**通常不需要——`.storybook/preview` 装饰器会被自动打包；仅当失败时才设置。在 §6 上传前，把装饰器提供的上下文提炼进 `cfg.provider`——README/prompt.md 的包裹指引仅由配置生成（只有装饰器包裹时会发布一段通用说明）。**设置它还会在下一次构建时取代装饰器成为预览包裹层**：切换后对一个主题化组件做范围比对——提炼不完整会让装饰器本可正常渲染的预览退化，而已携带的评分不会捕获这一点。格式：`{"component": "ThemeProvider", "props": {...}, "inner": {...}}`——嵌套链，最外层在前；每个 `component` 必须是 bundle 导出。字面量 `props` 适用于小标量（`"theme": "light"`）和稳定片段。对仓库中已有的数据——locale JSON、主题对象——**优先用 `{"$ref": "<export>"}`**，并由经 `cfg.extraEntries` 添加的两行模块支撑（如 `export { default as previewI18n } from '../locales/en.json'`）：`$ref` 会生成 `window.<Global>.<export>`，数据只在 bundle 中存一份，且每次构建都从其源文件重新读取。对极小且稳定的内容内联一份副本可以接受，但要清楚代价——字面量会复制进每张卡片的 html，并在源文件变化时静默腐化，因此任何体量较大或持续演进的数据都应放在 `$ref` 之后。`extraEntries` 的路径形式：裸名称从 `node_modules` 解析；仓库自有模块需要显式的 `./`/`../` 包相对路径（以工作区为界——若越界，构建日志会打印 `! extraEntries: ... skipped`）。 |

4. **Stage scripts + install converter deps** (isolated in `.ds-sync/`, repo lockfile untouched):

   4. **暂存脚本 + 安装转换器依赖**（隔离在 `.ds-sync/` 中，仓库锁文件不受影响）：

   ```bash
   mkdir -p .ds-sync && cp -r "<skill-base-dir>"/package-build.mjs "<skill-base-dir>"/package-validate.mjs "<skill-base-dir>"/resync.mjs "<skill-base-dir>"/lib "<skill-base-dir>"/storybook "<skill-base-dir>"/non-storybook .ds-sync/
   echo '{"name":"ds-sync-deps","private":true}' > .ds-sync/package.json
   (cd .ds-sync && npm i esbuild ts-morph @types/react playwright && npx playwright install chromium)
   ```

   If chromium install fails, `npx playwright install-deps chromium` first; if the environment can't install chromium, set `DS_CHROMIUM_PATH=<system-chromium>`.

   若 chromium 安装失败，先执行 `npx playwright install-deps chromium`；若环境无法安装 chromium，设置 `DS_CHROMIUM_PATH=<system-chromium>`。

5. **Run the converter, validator, and compare** - synchronously, stopping at the first non-zero exit (compare only runs once build + validate are clean - §3). Large DSes (~100+ components) may need `NODE_OPTIONS=--max-old-space-size=<MB>` for the build; **never pipe the build through `head`/`tail`** (the pipeline masks the exit code - an OOM looks like success); redirect to a file and read it:

   5. **运行转换器、校验器和比对**——同步执行，在第一个非零退出处停止（只有 build + validate 都干净时才运行 compare——§3）。大型 DS（约 100+ 组件）构建时可能需要 `NODE_OPTIONS=--max-old-space-size=<MB>`；**绝不要把构建输出通过管道接给 `head`/`tail`**（管道会掩盖退出码——OOM 看起来像成功）；重定向到文件再读取：

   ```bash
   node .ds-sync/package-build.mjs --config .design-sync/config.json --node-modules <pkg-node-modules> \
     --entry <built-dist-entry> --out ./ds-bundle
   node .ds-sync/package-validate.mjs ./ds-bundle
   node .ds-sync/storybook/compare.mjs --out ./ds-bundle --storybook-static .design-sync/sb-reference \
     --components <solo-phase picks>   # scope the FIRST compare to the §4b solo components
   ```

   In a monorepo, `--node-modules` is the DS package's own `node_modules` - unless hoisting leaves it sparse (yarn's `node-modules` linker keeps `react` only at the repo root): if `react/` or `react-dom/` is missing inside, pass the repo-root `node_modules` instead. In the DS's own source repo `node_modules/<pkg>` doesn't exist, hence `--entry`. The build logs `[ICON_PKG]` / `[TOKENS_PKG]` auto-detections and bundles `.storybook/preview` decorators as the preview wrapper (`preview-decorators.js`) so previews get the same provider chain stories do.

   在 monorepo 中，`--node-modules` 是 DS 包自己的 `node_modules`——除非依赖提升使其变得稀疏（yarn 的 `node-modules` linker 只在仓库根保留 `react`）：如果其中缺少 `react/` 或 `react-dom/`，改为传入仓库根的 `node_modules`。在 DS 自身的源码仓库中不存在 `node_modules/<pkg>`，因此需要 `--entry`。构建会记录 `[ICON_PKG]` / `[TOKENS_PKG]` 自动探测，并把 `.storybook/preview` 装饰器打包为预览包裹层（`preview-decorators.js`），使预览获得与 story 相同的 provider 链。

   Scope the first compare run: a full capture of a large DS is thousands of chromium navigations - pointless before the solo phase has flushed global issues (each global fix invalidates every capture). The first roster-wide run happens per §4b step 3 - and on a DS over 20 storied components even that is size-gated into §4c's scoped batches, so the only mandatory full-roster run is the §4d receipt, which carries graded work forward instead of recapturing it. For a DS with >100 storied components, also tell the user the expected scale (components × stories) before fan-out and let them narrow scope if they want.

   限定首次比对运行的范围：对大型 DS 做全量捕获意味着数千次 chromium 导航——在 solo 阶段清刷全局问题之前毫无意义（每个全局修复都会使所有捕获失效）。首次全花名册运行发生在 §4b 第 3 步——而在拥有 20 个以上含 story 组件的 DS 上，连这一步也会按规模被限流进 §4c 的分批限定运行，因此唯一强制性的全花名册运行是 §4d 回执，它携带已评分的工作前进而非重新捕获。对于含 >100 个含 story 组件的 DS，还要在扇出前告知用户预期规模（组件数 × story 数），并允许其按需缩小范围。

## 3. Self-heal loop (build + validate) / 自愈循环（构建 + 校验）

Fix `[TAG]` errors -> rebuild -> re-validate until both exit 0, **before** starting the compare loop in §4 - there's no point pixel-matching previews while the bundle itself is broken. Shared converter tags (`[NO_DIST]`, `[WORKSPACE_SIBLING]`, `[CSS_*]`, `[FONT_*]`, `[TOKENS_MISSING]`, `[DTS_*]`, `[RENDER*]`, ...) behave identically to the package shape - use the table in `../non-storybook/SKILL.md` §3. Lines printed as `hypothesis:` under an error are leads, not instructions: run their verify step first, and if it doesn't confirm, drop the hypothesis and diagnose from the error text itself. Storybook-specific:

修复 `[TAG]` 错误 -> 重建 -> 重新校验，直到两者都以 0 退出，且要在开始 §4 的比对循环**之前**——bundle 本身还是坏的，做像素级匹配毫无意义。共享转换器标签（`[NO_DIST]`、`[WORKSPACE_SIBLING]`、`[CSS_*]`、`[FONT_*]`、`[TOKENS_MISSING]`、`[DTS_*]`、`[RENDER*]` 等）的行为与包形态完全一致——使用 `../non-storybook/SKILL.md` §3 的表格。错误下方以 `hypothesis:` 打印的行是线索，不是指令：先运行其验证步骤，若未得到确认，就放弃该假设并从错误文本本身入手诊断。Storybook 特有的：

| Tag | Symptom | Fix |
|---|---|---|
| `[SB_REFERENCE_MISSING]` | compare can't find `iframe.html` | Build the reference (§2.2); set `cfg.storybookStatic`. |
| `[SB_BUILD_FAIL]` | converter's own storybook build failed | You skipped §2.2 - build the reference yourself and set `cfg.storybookStatic` so the converter never needs to. |
| `[ZERO_MATCH]` (storybook flavor) | no story entries matched | Check the storybook config's `stories` glob; then `titleMap`. |
| `[TITLE_UNMAPPED]` | N titles don't match an export | `cfg.titleMap {<title-name>: <export-name>}`. |
| `(preview: <Name> ... no story exports paired ...)` | index story names couldn't be matched to module export keys (pairing tries the display name, then the story ID's tail) | the component shows the floor card; fix the pairing - usually an owned `.tsx` re-exporting the stories under matchable names. |
| a preview cell errors with `undefined`-component / wrong-context messages | a story import resolved the wrong way - relative, tsconfig-alias, and bare-workspace imports all go through the same policy (see `lib/story-imports.mjs`'s rules) | `cfg.storyImports.shim` / `cfg.storyImports.bundle` substring patterns force the resolution per resolved path - the cheap fix before forking the seam. |
| `! preview build failed: <Name>` | the story module didn't COMPILE (top-level await, an import of a package esbuild can't resolve, an asset extension with no loader) | read the esbuild error above the line. Unknown asset extension -> `cfg.storyImports.loaders` (merged over the defaults, e.g. `{".yaml": "text"}`); unresolvable import -> own the `.tsx` and drop it. The component shows the floor card until fixed. |
| a story's own stylesheet is missing from its cell | story-local `.css`/`.scss` side-effect imports compile as empty (component styles ship via the bundle css). Exception: `.module.css` IS compiled - classes resolve and `_preview/<Name>.css` is linked automatically | usually nothing - the styles are decoration the storybook page adds. If the story genuinely depends on them, inline the styles in an owned `.tsx`. |
| `[BUNDLE_EXPORT]` | components aren't functions on `window.<Global>` | `extraEntries` for subpath/icon exports; check the dist entry is the full build. |
| `[SCHEDULER_MISSING]` | dist imports `scheduler` | react-dom leaked into the DS dist - check its build's externals. |
| `! preview decorator bundle failed` | decorators couldn't be bundled | Set `cfg.provider` manually, or run `node .ds-sync/storybook/probe.mjs --storybook-static .design-sync/sb-reference` to infer the chain from the live storybook (replace each `$hint` with a real value). |
| previews error at `_vendor/preview-decorators.js` load (storybook-API `undefined` errors) | the `.storybook/preview` import graph reached a storybook-runtime module the stubs don't cover | `manager-api`/`preview-api` are stubbed with functional no-op hooks and every other `@storybook/*`/`msw` module with inert callables (`fn()`, `action()`, `setupWorker()` at module scope all evaluate harmlessly); if some other API still crashes, set `cfg.provider` explicitly - it skips decorator bundling entirely. |
| `[ASSETS_BLOCKED]` from compare | the capture browser inherited a network-sandboxed shell - story assets (CDN images/fonts) failed on **both** panels, so grades can falsely pass while end users see different output | re-run `package-validate.mjs` + `compare.mjs --force` from a shell with egress to the listed hosts: approve running the command without the sandbox when prompted, or add the hosts to the sandbox allowlist. Don't grade image-bearing components while this prints. |

| 标签 | 症状 | 修复 |
|---|---|---|
| `[SB_REFERENCE_MISSING]` | 比对找不到 `iframe.html` | 构建参考（§2.2）；设置 `cfg.storybookStatic`。 |
| `[SB_BUILD_FAIL]` | 转换器自身的 storybook 构建失败 | 你跳过了 §2.2——自己构建参考并设置 `cfg.storybookStatic`，让转换器永远无需再做。 |
| `[ZERO_MATCH]`（storybook 变体） | 没有任何 story 条目匹配 | 检查 storybook 配置的 `stories` glob；然后检查 `titleMap`。 |
| `[TITLE_UNMAPPED]` | N 个标题无法匹配到导出 | `cfg.titleMap {<title-name>: <export-name>}`。 |
| `(preview: <Name> ... no story exports paired ...)` | index 的 story 名称无法与模块导出键配对（配对先尝试显示名，再尝试 story ID 的尾部） | 组件显示底座卡片；修复配对——通常需要一个以可匹配名称再导出 stories 的自有 `.tsx`。 |
| 预览单元报 `undefined`-组件 / 错误上下文类消息 | 某个 story 导入被解析到了错误的方式——相对、tsconfig 别名和裸工作区导入都走同一策略（见 `lib/story-imports.mjs` 的规则） | `cfg.storyImports.shim` / `cfg.storyImports.bundle` 子串模式可按解析后路径强制指定解析方式——这是分叉该接缝之前的廉价修法。 |
| `! preview build failed: <Name>` | story 模块没有编译通过（顶层 await、esbuild 无法解析的导入、无 loader 的资源扩展名） | 阅读该行上方的 esbuild 错误。未知资源扩展名 -> `cfg.storyImports.loaders`（合并覆盖默认值，如 `{".yaml": "text"}`）；无法解析的导入 -> 自有化该 `.tsx` 并删除该导入。修复前组件一直显示底座卡片。 |
| 某个 story 自己的样式表在其单元中缺失 | story 局部 `.css`/`.scss` 副作用导入编译为空（组件样式经由 bundle css 提供）。例外：`.module.css` 会被编译——类名可解析，`_preview/<Name>.css` 会被自动链接 | 通常无需处理——那些样式只是 storybook 页面添加的装饰。若 story 确实依赖它们，把样式内联到一个自有 `.tsx` 中。 |
| `[BUNDLE_EXPORT]` | 组件在 `window.<Global>` 上不是函数 | 子路径/图标导出用 `extraEntries`；检查 dist 入口是否为完整构建。 |
| `[SCHEDULER_MISSING]` | dist 导入了 `scheduler` | react-dom 泄漏进了 DS dist——检查其构建的 externals。 |
| `! preview decorator bundle failed` | 装饰器无法打包 | 手动设置 `cfg.provider`，或运行 `node .ds-sync/storybook/probe.mjs --storybook-static .design-sync/sb-reference` 从活体 storybook 推断 provider 链（把每个 `$hint` 替换为真实值）。 |
| 预览在加载 `_vendor/preview-decorators.js` 时报错（storybook API `undefined` 类错误） | `.storybook/preview` 的导入图触及了 stub 未覆盖的 storybook 运行时模块 | `manager-api`/`preview-api` 以带功能的 no-op 钩子做 stub，其他所有 `@storybook/*`/`msw` 模块则提供惰性可调用对象（模块作用域的 `fn()`、`action()`、`setupWorker()` 都会无害求值）；若仍有其他 API 崩溃，显式设置 `cfg.provider`——它会完全跳过装饰器打包。 |
| compare 报 `[ASSETS_BLOCKED]` | 捕获浏览器继承了网络沙箱 shell——story 资源（CDN 图片/字体）在**两个**面板上都加载失败，因此评分可能虚假通过，而终端用户看到的输出不同 | 从具有所列主机出站访问的 shell 重新运行 `package-validate.mjs` + `compare.mjs --force`：在提示时批准无沙箱运行命令，或把这些主机加入沙箱允许列表。此告警打印期间不要给含图片的组件评分。 |

**Incremental path (base SKILL.md §3) - this is the open-the-channel gate.** The first time build + validate both exit 0, open the upload channel before starting §4: the user approves once here, then watches components land as grading proceeds. Nothing uploads until the first graded batch - the shared base files ride with it - and the batch pushes come from §4b/§4c. (Atomic path: nothing uploads until §6.)

**增量路径（基础 SKILL.md §3）——这是打开通道的闸口。**当 build + validate 首次双双以 0 退出时，在开始 §4 之前打开上传通道：用户在此批准一次，然后随评分推进看着组件逐批落地。在首个已评分批次之前不会有任何上传——共享基础文件随之一起上行——批次推送来自 §4b/§4c。（原子路径：§6 之前不上传任何东西。）

## 4. Match previews to storybook / 让预览与 storybook 匹配

`compare.mjs` is a **capture harness - it photographs, you grade.** It computes no similarity heuristics (pixel/text/font scores mislead whenever framing legitimately differs); the judgment is made from the two true screenshots. Compiled previews capture **per story** - each story renders alone via `?story=<Export>` at the full capture viewport, exactly as storybook frames the reference side - so sibling stories can't interfere (portal stacking, shared radio-group names, focus, container measurement). Two output tiers:

`compare.mjs` 是一个**捕获工具——它负责拍照，你来评分。**它不计算任何相似度启发式（当取景确实不同时，像素/文本/字体分数会误导）；判断由两张真实截图作出。编译式预览**按 story** 捕获——每个 story 通过 `?story=<Export>` 在完整捕获视口下单独渲染，与 storybook 框定参考侧的方式完全一致——因此兄弟 story 无法相互干扰（portal 堆叠、共享的 radio-group 名称、焦点、容器测量）。两类输出：

- **Transient** (under `ds-bundle/`, wiped by rebuilds): `_screenshots/compare/<group>__<Name>.png` - sheet with one row per story: the **true storybook render | the true preview render**, side by side. Sheet images are shrunk to fit; the full-resolution originals are in `.../compare/raw/` (`...__sb.png` / `...__ds.png`) - Read those when the sheet is too small to judge confidently.
  **临时**（位于 `ds-bundle/` 下，重建时清除）：`_screenshots/compare/<group>__<Name>.png`——每行一个 story 的对照表：**真实 storybook 渲染 | 真实预览渲染**，并排呈现。对照表图片会缩放以适配；全分辨率原图在 `.../compare/raw/`（`...__sb.png` / `...__ds.png`）——当对照表太小、难以自信判断时读取原图。
- **Campaign state** (in `.design-sync/.cache/compare/`, gitignored): `<Name>.grade.json` - your verdicts - and `<Name>.json` - capture facts: story<->cell pairing, shot paths, `previewKind`, the component's `srcSha` (story-file fingerprint), spot-check anchors. Reconstructible - absence just means "capture again". The only verdicts the script emits are factual: `sb-error` (story doesn't render in storybook), `unpaired` (no preview cell for the story), `error` (cell threw); every rendered pair is `needs-grade`.
  **战役状态**（位于 `.design-sync/.cache/compare/`，已 gitignore）：`<Name>.grade.json`——你的评判结果；`<Name>.json`——捕获事实：story<->单元配对、截图路径、`previewKind`、组件的 `srcSha`（story 文件指纹）、抽检锚点。均可重建——缺席只意味着"再捕获一次"。脚本给出的唯一自动评判是事实性的：`sb-error`（story 在 storybook 中不渲染）、`unpaired`（该 story 没有预览单元）、`error`（单元抛错）；每一对渲染成功的都是 `needs-grade`。

Compare captures at most 6 stories per component by default - `[STORY_CAP]` in the log names components with more, and `--max-stories <n>` raises the cap. The cap is NOT part of the grade contract: raising it just captures the tail stories for incremental grading, and existing verdicts survive. One consequence to know: a capped component that grades fully `match`/`close` is verified-by-upload in full on future syncs even though its tail stories were never individually graded - raise the cap when those tail stories carry distinct variants worth verifying. Fan-out subagents must not change it mid-wave (sheets would cover different story sets than the orchestrator's worklist assumed).

比对默认每个组件最多捕获 6 个 story——日志中的 `[STORY_CAP]` 会点名超出者，`--max-stories <n>` 可提高上限。该上限不属于评分契约的一部分：提高它只是为增量评分捕获尾部 story，已有评判不受影响。需要知道的一个后果：一个被截断的组件若整体评为 `match`/`close`，未来同步中将整体按"已验证随上传"处理，即便其尾部 story 从未被单独评分——当尾部 story 承载值得验证的独立变体时，应提高上限。扇出子代理不得在波次中途更改它（否则对照表覆盖的 story 集合将与编排者的工作清单假设不一致）。

**State across runs** - the first run verifies everything once; after that, one rule: **grades follow your sources** - the story files, your owned previews, the story set, the preview-affecting config (`provider`/`storyImports`/`extraEntries`/`overrides`/`titleMap`), and committed `.design-sync/overrides/` forks. Pipeline churn (a skill or toolchain update re-rendering everything) is auto-verified by a sampled `[SPOT_CHECK]` with grades kept; your edits re-grade only what they touch. Pixel jitter can never churn grades.

**跨运行状态**——第一次运行把一切验证一遍；此后只有一条规则：**评分跟随你的源**——story 文件、你的自有预览、story 集合、影响预览的配置（`provider`/`storyImports`/`extraEntries`/`overrides`/`titleMap`），以及已提交的 `.design-sync/overrides/` fork。流水线扰动（技能或工具链更新导致全部重新渲染）由抽样的 `[SPOT_CHECK]` 自动验证且保留评分；你的编辑只重新评分它们触及的部分。像素抖动永远不会搅动评分。

- *Sources unchanged* + fully graded `match`/`close` -> **skipped outright** (`carried forward`): no capture, no re-grade - even when the bundle, styling, storybook, or the converter itself were rebuilt. `--force` recaptures everything **and clears all grades** - systemic re-verification, not casual sheet regeneration.
  - *源未变*且已全部评为 `match`/`close` -> **直接跳过**（`carried forward`）：不捕获、不重评——即使 bundle、样式、storybook 或转换器本身被重建过。`--force` 会重新捕获一切**并清除所有评分**——这是系统性重验证，不是随手再生成对照表。
- *Sources changed* (story edited, `.tsx` edited, config/fork edited) -> recapture, grade cleared, re-grade from the fresh sheet. `[STORY_CHANGED]` marks stories whose code moved - those are the ones where an OWNED `.tsx` **must be updated** (generated previews re-derive automatically); a recapture *without* `[STORY_CHANGED]` usually just needs the re-grade.
  - *源已变*（story 被编辑、`.tsx` 被编辑、配置/fork 被编辑）-> 重新捕获、清除评分、按新对照表重评。`[STORY_CHANGED]` 标记代码有变动的 story——那些是自有 `.tsx` **必须更新**的情形（生成的预览会自动重新推导）；*不带* `[STORY_CHANGED]` 的重新捕获通常只需重新评分。
- *`[SPOT_CHECK]`* -> re-captures named components **without clearing their grades**; Read the fresh sheets and confirm they still match the recorded grades. It can arrive driver-triggered after pipeline churn - the normal verification of a skill/toolchain update, not a bug. Divergence remediation scales with the churned set: a couple of components -> re-grade just those; widespread -> stop, diagnose, then `--force` a full pass. `--spot-check N` tunes the full-run random sample (0 disables); `--spot-check-components A,B` names picks explicitly, honored on scoped runs too (the §7 step-4 audit).
  - *`[SPOT_CHECK]`* -> 重新捕获被点名的组件**且不清除其评分**；读取新对照表并确认其仍与已记录评分相符。它可能在流水线扰动后由 driver 触发——这是技能/工具链更新后的正常验证，不是 bug。分歧处理随受扰集合规模伸缩：个别组件 -> 只重评它们；大范围 -> 停下、诊断，然后 `--force` 做全量通过。`--spot-check N` 调整全量运行的随机抽样（0 为禁用）；`--spot-check-components A,B` 显式点名选择对象，限定运行同样遵守（§7 第 4 步审计）。
- *`[REFERENCE_STALE?]`* -> the bundle changed but the reference storybook didn't. If the DS source changed, rebuild `.design-sync/sb-reference` before grading - a stale reference makes every grade a comparison against the *old* design.
  - *`[REFERENCE_STALE?]`* -> bundle 变了而参考 storybook 没变。若 DS 源码已变，评分前先重建 `.design-sync/sb-reference`——陈旧的参考会让每次评分都变成与*旧*设计的比较。
- *A story renders differently every capture* (`new Date()`/`Math.random()` content) -> the fingerprint is the story FILE, so the contract is stable - but the pixels aren't, and grading judges pixels. The frozen capture clock stabilizes date renders; for truly random content, pin values in an owned `.tsx` or `cfg.overrides.<Name>.skip` the story with a NOTES.md line.
  - *某 story 每次捕获渲染都不同*（`new Date()`/`Math.random()` 内容）-> 指纹是 story 文件本身，契约是稳定的——但像素不稳定，而评分判的是像素。冻结的捕获时钟可以稳定日期渲染；对真正的随机内容，在自有 `.tsx` 中固定取值，或用 `cfg.overrides.<Name>.skip` 跳过该 story 并在 NOTES.md 记一行。

Captures are stabilized for grading comparability (animations fast-forwarded, reduced motion, frozen clock - both panels show the same settled frame, the same rendered date). This is verification-only: shipped previews are untouched and fully animated.

捕获为评分可比性做了稳定化（动画快进、减少动态效果、冻结时钟——两个面板显示同一个定格帧、同一个渲染日期）。这仅用于验证：实际发布的预览不受影响，动画齐全。

**Grading is done by whoever is working the component** - you in the solo phase, each subagent for its own components in fan-out. After each compare run: Read the sheet (and raw PNGs when in doubt), judge each story **from the images alone**, Write the verdicts to `.design-sync/.cache/compare/<Name>.grade.json` (campaign-local working state - what makes a verdict durable is the upload: the uploaded `_ds_sync.json` anchors verified-by-upload skips on every future sync, any machine):

**评分由正在处理该组件的人完成**——solo 阶段是你，扇出时是各子代理处理各自的组件。每次比对运行后：读取对照表（存疑时读原图 PNG），**仅凭图像**评判每个 story，把评判写入 `.design-sync/.cache/compare/<Name>.grade.json`（战役本地的工作状态——让评判持久化的是上传：已上传的 `_ds_sync.json` 会在未来每次同步、任何机器上锚定"已验证随上传"的跳过）：

```json
{"stories": {"Default": {"verdict": "match"}, "Compact": {"verdict": "match", "basis": "sibling-trusted"}}}
{"stories": {"Loading": {"verdict": "mismatch", "note": "spinner missing - story uses MSW mock"}}}
```

(Two components' files: a clean one graded under the sampling rule below - `Default` is the image-judged primary story, `match` on a warning-free component, which is what licenses the sibling-trusted entries - and a mismatching one, whose note drives the next fix.)

（两个组件的文件：一个干净的按下方抽样规则评分——`Default` 是图像评判的主 story，在无告警组件上的 `match`，正是它授权了 sibling-trusted 条目；另一个不匹配，其 note 驱动下一次修复。）

Rubric - grade what a designer would care about, looking at the two renders:

评分标准——看着两张渲染图，按设计师关心的维度评分：

- `match` - same content, composition, and styling. Ignore antialiasing fuzz, scrollbar slivers, sub-5px offsets, and framing differences (the storybook canvas and the preview page frame differently - judge the component, not its surroundings).
  - `match` - 内容、构图与样式一致。忽略抗锯齿噪点、滚动条残条、5px 以内的偏移和取景差异（storybook 画布与预览页面的取景方式不同——评判组件本身，而非其周围环境）。
- `close` - recognizably the same rendering with a minor delta (slightly different padding, focus ring, placeholder text). **`close` is still a fix target, not an exit:** if you can name the delta, you can usually name the knob - keep iterating. Accept `close` only after an iteration fails to improve it or no actionable cause remains, and the note must then say both *what's off* and *what you tried / why it's not fixable* (e.g. "focus ring color differs - storybook applies a global focus addon, not part of the DS").
  - `close` - 可辨认是同一渲染，仅有微小差异（padding 略不同、focus ring、占位文本）。**`close` 仍是修复目标，不是出口：**如果你能说出差异，通常也能说出对应的旋钮——继续迭代。只有当一次迭代未能改善、或已无可行动的成因时才接受 `close`，且 note 必须同时说明*哪里不对*和*你试过什么 / 为何无法修复*（例如"focus ring 颜色不同——storybook 应用了全局 focus 插件，不属于 DS"）。
- `mismatch` - wrong/missing content, unstyled output, wrong variant, missing icons/images, default fonts. The note must say *what* differs - it drives the next fix.
  - `mismatch` - 内容错误/缺失、无样式输出、变体错误、图标/图片缺失、默认字体。note 必须说明*什么*不同——它驱动下一次修复。

When the REFERENCE side is the artifact - storybook gates the story behind UI chrome (a theme/control toggle message) while the preview renders the real component - judge the component render on its own and note the gating; a preview that renders *more* than the gated reference is not `close`.

当 REFERENCE 侧才是制品本身时——storybook 把 story 藏在 UI 外壳（主题/控件切换提示）之后，而预览渲染的是真实组件——应单独评判组件渲染并记录该门控；渲染*得比被门控的参考更多*的预览不算 `close`。

**Grade the primary story, trust the rest.** Sibling stories of one component run through the same pipeline - same imports, same provider chain, same CSS - so when one of them renders faithfully the rest almost always do too. On a first sync, judge from images the component's **primary story** only (`cfg.overrides.<Name>.primaryStory` when set - the same story the single-mode card renders - else the sheet's first story). If it grades `match` and the component is clean - no `sb-error`/`unpaired`/`error` cells, no `[PORTAL?]`, no `[RENDER_BLANK]`, no blank or size-anomalous shots - write `match` for the remaining stories with a basis marker, `{"verdict": "match", "basis": "sibling-trusted"}`, so the record says how each verdict was reached (compare reads only the `verdict` string). All of a component's verdicts - the image-judged primary plus every sibling-trusted entry - go in its one `grade.json` Write: trusted siblings cost no image opens and no per-story passes. Grade exhaustively, story by story, when the component has portals/overlays, theme or provider sensitivity, an owned preview, or any warning - and always for the §4b solo set, whose exhaustive grading is what earns the trust in the first place.

**评主 story，信其余。**同一组件的兄弟 story 走同一条流水线——相同导入、相同 provider 链、相同 CSS——所以当其中一个渲染忠实时，其余几乎总是同样忠实。首次同步时，仅凭图像评判组件的**主 story**（若设置了 `cfg.overrides.<Name>.primaryStory` 则用它——即单卡片模式渲染的同一 story——否则取对照表第一个 story）。若其评为 `match` 且组件干净——没有 `sb-error`/`unpaired`/`error` 单元、没有 `[PORTAL?]`、没有 `[RENDER_BLANK]`、没有空白或尺寸异常的截图——就为其余 story 写入带依据标记的 `match`：`{"verdict": "match", "basis": "sibling-trusted"}`，让记录说明每条评判如何得出（compare 只读取 `verdict` 字符串）。一个组件的全部评判——图像评判的主 story 加上每条 sibling-trusted 条目——放进同一次 `grade.json` Write：信任兄弟不花图像打开次数，也不逐 story 走流程。当组件有 portal/覆盖层、主题或 provider 敏感性、自有预览或任何告警时，逐 story 详尽评分——§4b solo 集合则永远详尽评分，正是其详尽评分赢得了这份信任。

Capture photographs every story either way - sampling saves grading attention, not capture time, and the sheets stay available for any deliberate later look (the §7 step-4 carried-grade audit uses the same grades-kept spot-check path). This is the same trust class as `[STORY_CAP]`'s ungraded tail stories, applied deliberately. Sampling never relaxes `[FONT_MISSING]` (§4a) - that check is invisible to the compare images either way.

无论如何捕获都会给每个 story 拍照——抽样节省的是评分注意力，不是捕获时间，对照表也保持可用，供之后有意的查看（§7 第 4 步的携带评分审计走同一条保留评分的抽检路径）。这与 `[STORY_CAP]` 未评分的尾部 story 属同一信任类别，只是有意为之。抽样绝不放宽 `[FONT_MISSING]`（§4a）——该检查对比对图像本来就不可见。

### 4a. Fix decision tree - global first / 修复决策树——全局优先

Work top-down; a global fix repairs every component at once, a per-component fix repairs one:

自上而下处理；全局修复一次修好所有组件，逐组件修复只修一个：

1. **Most/all components wrong the same way** -> global, fix in config + full rebuild:

   1. **多数/全部组件以同一方式出错** -> 全局问题，在配置中修复 + 全量重建：

   - Context/provider errors in cells (`use<X> must be inside <Provider>`) -> decorators didn't bundle (§3 `! preview decorator bundle failed` rows) -> `cfg.provider`.
     - 单元中的 context/provider 错误（`use<X> must be inside <Provider>`）-> 装饰器未打包（§3 的 `! preview decorator bundle failed` 行）-> `cfg.provider`。
   - Everything unstyled / default fonts -> `cfg.cssEntry` (check `[CSS_FROM_STORYBOOK]` in the build log), `cfg.tokensPkg`, `cfg.extraFonts`.
     - 一切无样式 / 默认字体 -> `cfg.cssEntry`（在构建日志中检查 `[CSS_FROM_STORYBOOK]`）、`cfg.tokensPkg`、`cfg.extraFonts`。
   - **`[FONT_MISSING]` - the compare loop cannot see this one.** When neither side ships the font, both panels render the same chromium fallback, so the sheets look "matching" while every claude.ai/design user gets the wrong font - never accept "both sides fall back the same way" as a pass. Resolve per the `[FONT_MISSING]` row in `../non-storybook/SKILL.md` §3; storybook-specific extras: `cfg.extraFonts` paths are bounded by the git repo enclosing `dirname(--node-modules)` - sibling typography packages in the monorepo work as-is; only with no `.git` ancestor does the bound narrow to `dirname(--node-modules)`, and if you add a font the reference lacks, inject the same `@font-face` into `.design-sync/sb-reference/iframe.html` so the oracle verifies with the real font on both sides.
     - **`[FONT_MISSING]`——比对循环看不见这个问题。**当两侧都没带该字体时，两个面板渲染同样的 chromium 回退字体，对照表看起来"匹配"，而每个 claude.ai/design 用户拿到的都是错误字体——绝不要把"两侧以同样方式回退"当作通过。按 `../non-storybook/SKILL.md` §3 的 `[FONT_MISSING]` 行处理；storybook 特有补充：`cfg.extraFonts` 路径以包含 `dirname(--node-modules)` 的 git 仓库为界——monorepo 中的兄弟排版包可直接使用；只有当没有 `.git` 祖先时边界才收窄到 `dirname(--node-modules)`；若你添加了参考构建缺少的字体，要把同一个 `@font-face` 注入 `.design-sync/sb-reference/iframe.html`，让基准在两侧都用真实字体验证。

   【评论】此条点出了自动化视觉比对的固有盲区：当被比较的两侧犯同样的错误时，差异检测会失效。这类"共同基准偏移"需要独立于比对之外的显式检查来兜底。

   - Icons missing everywhere -> `cfg.extraEntries` (check `[ICON_PKG]`).
     - 图标处处缺失 -> `cfg.extraEntries`（检查 `[ICON_PKG]`）。

2. **One component, `unpaired` or `fallback preview`** -> its `.tsx` lacks a cell for that story. Previews compile the story MODULE whole (hooks, fixtures, local helpers all included - closures are not a failure mode), so the causes are: pairing failed (`storyName` override), the wrapper build failed (`! preview build failed` in the build log), or the module threw at load - check the sheet's `(page)` error row for the real exception (module-scope calls into a package the stubs don't cover). Open the wrapper (generated: `.design-sync/.cache/previews/<Name>.tsx`; owned: `.design-sync/previews/<Name>.tsx`), add/rename the export or drop the offending import - and if it's the generated one, save your fix as `.design-sync/previews/<Name>.tsx` WITHOUT the first-line marker (an in-place cache edit is preserved on this machine but gitignored - it vanishes on a fresh clone, and it recompiles without ever re-grading; only the owned copy moves the grade contract, and the rebuild warns about edited cache twins). Story imports use the location-independent `@ds-stories/<repo-relative path>` form, so the file works unchanged from either home.

   2. **单个组件，`unpaired` 或 `fallback preview`** -> 它的 `.tsx` 缺少该 story 的单元。预览把 story 模块整体编译（hooks、fixtures、本地辅助函数全部包含——闭包不是失败模式），所以成因是：配对失败（`storyName` 覆盖）、包装器构建失败（构建日志中的 `! preview build failed`）、或模块在加载时抛错——检查对照表的 `(page)` 错误行获取真实异常（模块作用域调用了 stub 未覆盖的包）。打开包装器（生成的：`.design-sync/.cache/previews/<Name>.tsx`；自有的：`.design-sync/previews/<Name>.tsx`），添加/重命名导出或删掉肇事的导入——如果是生成的那个，把修复保存为 `.design-sync/previews/<Name>.tsx`，**不要**带首行标记（就地编辑缓存文件在本机可保留但被 gitignore——换新克隆就消失，而且重新编译时永远不会重新评分；只有自有副本才改变评分契约，重建时会对被编辑的缓存孪生文件发出警告）。story 导入使用与位置无关的 `@ds-stories/<repo-relative path>` 形式，因此文件放在哪个家都原样可用。

3. **One component, you graded `mismatch`** -> wrong props/composition. Read the story source; mirror it in an owned `.design-sync/previews/<Name>.tsx` (copy the cache wrapper there minus its marker line). That's the only lever for compiled story previews.

   3. **单个组件，你评了 `mismatch`** -> props/组合错误。阅读 story 源码；在一个自有 `.design-sync/previews/<Name>.tsx` 中镜像它（把缓存包装器复制过去，去掉标记行）。这是编译式 story 预览唯一的杠杆。

4. **`sb-error`** -> the story doesn't render in storybook either (data-fetching, interaction-driven). Add its id to `cfg.overrides.<Name>.skip` and note why in NOTES.md.

   4. **`sb-error`** -> 该 story 在 storybook 里也不渲染（数据获取、交互驱动）。把它的 id 加入 `cfg.overrides.<Name>.skip`，并在 NOTES.md 里记下原因。

5. **`[PORTAL?]` / overlay components** (Dialog/Tooltip/Toast) -> grading is already isolated (per-story capture), but the PRODUCT card renders the whole grid html, so open-overlay stories paint over sibling cells there too. Set `cfg.overrides.<Name>.cardMode: "single"` - the card renders one story (`primaryStory` picks it; first export otherwise) full-bleed in a wrapper that contains `position:fixed` descendants, and declares the grading viewport on the card so the product renders at the size you verified. For stories that are merely too WIDE for a grid cell (data tables, full-width bars - validate flags these as `[GRID_OVERFLOW] ... wide`), use `cardMode: "column"` instead: every story keeps full card width, nothing is dropped. Targeted-rebuild that component (`preview-rebuild.mjs --components <Name>`, seconds) - **grades carry** (`cardMode`/`primaryStory` aren't in the grade key or the stamped config slices); only a `viewport` change re-grades (it's the capture viewport) and needs the full build (it moves the slices).

   5. **`[PORTAL?]` / 覆盖层组件**（Dialog/Tooltip/Toast）-> 评分已经隔离（按 story 捕获），但产品卡片渲染的是整个网格 html，因此打开覆盖层的 story 在那里也会涂到兄弟单元上。设置 `cfg.overrides.<Name>.cardMode: "single"`——卡片以满幅方式渲染一个 story（由 `primaryStory` 指定；否则第一个导出），包裹层能收容 `position:fixed` 后代，并在卡片上声明评分视口，使产品按你验证过的尺寸渲染。对只是对网格单元来说太宽的 story（数据表、全宽横条——validate 会标记为 `[GRID_OVERFLOW] ... wide`），改用 `cardMode: "column"`：每个 story 保持整卡宽度，不丢弃任何 story。对该组件做定向重建（`preview-rebuild.mjs --components <Name>`，数秒）——**评分携带**（`cardMode`/`primaryStory` 不在评分键或已盖章的配置切片中）；只有 `viewport` 变化才重新评分（它是捕获视口）且需要全量构建（它会移动切片）。

**Rebuild rules - rebuild only what the change can reach.** Styling changes (css/fonts/tokens) re-render every preview without moving any grade contract - grades carry forward. Provider, `storyImports`, `extraEntries`, and fork edits are part of the grade contract (they change what the preview mounts) - affected grades clear and re-grade on the rebuild.

**重建规则——只重建改动所能波及的范围。**样式变更（css/字体/tokens）会重新渲染每个预览而不触动任何评分契约——评分向前携带。provider、`storyImports`、`extraEntries` 和 fork 编辑属于评分契约的一部分（它们改变预览挂载的内容）——受影响的评分在重建时清除并重评。

| You changed | Rebuild | Compare |
|---|---|---|
| a preview `.tsx` only | targeted loop below (seconds) | scoped `--components <Name>` - its grade cleared, re-grade |
| `overrides` (`skip`/`viewport`) / `titleMap` | full `package-build.mjs` + `package-validate.mjs` (re-stamps the config keys targeted rebuilds check) | full `compare.mjs` - the touched components re-grade; carried `match`/`close` components skip outright, and the still-pending set gets fresh sheets (the full build wiped them - the next wave reads those sheets) |
| `overrides` (`cardMode`/`primaryStory` only) | **targeted loop** (`preview-rebuild.mjs --components <Name>`, seconds) - presentation keys aren't in the stamped config slices, so `[CONFIG_STALE]` doesn't trip; the loop re-emits the card html and patches its renderHash | **no re-grade**: presentation-only keys aren't in the grade contract - grades carry; the changed card html re-ships and a re-sync may spot-check it |
| `provider` / `storyImports` / `.design-sync/overrides/` forks | full build + validate | full `compare.mjs` - affected grades re-grade per the rule above |
| css / fonts / tokens | `package-build.mjs --skip-dts` + validate | full `compare.mjs` - cheap: carried `match`/`close` components skip outright, so only the pending set recaptures against the new styling. Grades carry - zero-regrade, not zero-touch: the changed bytes still re-ship, and a re-sync may surface them as a `verification.canary` spot-check |
| `entry` / `extraEntries` | full build + validate - never `--skip-dts` (they change the bundle and export surface) | full `compare.mjs` - affected grades re-grade |

| 你改了什么 | 重建 | 比对 |
|---|---|---|
| 仅某个预览 `.tsx` | 下方的定向循环（数秒） | 限定 `--components <Name>`——其评分已清除，需重评 |
| `overrides`（`skip`/`viewport`）/ `titleMap` | 全量 `package-build.mjs` + `package-validate.mjs`（重新盖章定向重建所检查的配置键） | 全量 `compare.mjs`——被触及的组件重评；已携带 `match`/`close` 的组件直接跳过，仍待处理的集合获得新对照表（全量构建已将其清除——下一波读取这些新表） |
| `overrides`（仅 `cardMode`/`primaryStory`） | **定向循环**（`preview-rebuild.mjs --components <Name>`，数秒）——呈现类键不在已盖章的配置切片中，因此不会触发 `[CONFIG_STALE]`；循环重新生成卡片 html 并修补其 renderHash | **不重评**：纯呈现类键不在评分契约中——评分携带；变更的卡片 html 重新发布，重新同步可能对其抽检 |
| `provider` / `storyImports` / `.design-sync/overrides/` fork | 全量构建 + 校验 | 全量 `compare.mjs`——受影响的评分按上述规则重评 |
| css / 字体 / tokens | `package-build.mjs --skip-dts` + 校验 | 全量 `compare.mjs`——代价低：已携带 `match`/`close` 的组件直接跳过，因此只有待处理集合针对新样式重新捕获。评分携带——是"零重评"而非"零触碰"：变更的字节仍会重新发布，重新同步可能以 `verification.canary` 抽检的形式让其浮现 |
| `entry` / `extraEntries` | 全量构建 + 校验——绝不 `--skip-dts`（它们改变 bundle 与导出面） | 全量 `compare.mjs`——受影响的评分重评 |

Mid-campaign - §4c waves still pending - read this table's "full `compare.mjs`" as *eventually, via the batches*: the rebuild clears the affected grades either way, the next wave's scoped runs recapture those components, and the §4d receipt is the roster-wide settlement (§4c between-waves step 2). Pay an immediate roster-wide compare only when no waves remain.

战役进行中——§4c 波次尚未完成时——把本表的"全量 `compare.mjs`"理解为*最终会通过各批次完成*：无论如何重建都会清除受影响的评分，下一波的限定运行重新捕获那些组件，§4d 回执才是全花名册的结算（§4c 波间第 2 步）。只有当不再有波次时，才立即支付一次全花名册比对。

`--skip-dts` skips the per-component type extraction - the slow part of a large-DS build - and emits stub `.d.ts` bodies, so its validate fails `[DTS_STUBBED]` by design (the render checks still answer "did the fix work?"); the §4d/§6 gate's validate-exits-0 requirement forces the final build to run without it. Expect stub-build floor cards and README blurbs to look bare - the final build restores them. `--skip-dts` is for fix-loop iteration only: any build that an upload reads - an incremental batch push (base SKILL.md §3) as much as the §6 close-out - must be a real one, so if `.ds-build-meta.json` still carries `dtsStubbed`, rebuild without the flag before pushing (batch pushes upload the on-disk `.d.ts`).

`--skip-dts` 跳过逐组件的类型提取——大型 DS 构建中最慢的部分——并发出 stub 的 `.d.ts` 主体，因此其校验按设计会报 `[DTS_STUBBED]` 失败（渲染检查仍能回答"修复是否生效？"）；§4d/§6 闸口要求校验以 0 退出，迫使最终构建不带该标志运行。预期 stub 构建的底座卡片与 README 简介会显得简陋——最终构建会恢复它们。`--skip-dts` 只用于修复循环迭代：任何被上传读取的构建——增量批次推送（基础 SKILL.md §3）与 §6 收尾同样——必须是真实构建，因此若 `.ds-build-meta.json` 仍带有 `dtsStubbed`，推送前先不带该标志重建（批次推送上传的是磁盘上的 `.d.ts`）。

**Batch config edits into one cycle.** Before paying a rebuild, sweep every pending sheet verdict and known issue for ALL the config edits they imply (`skip`s, `titleMap` entries, `cardMode`s) and apply them together - two edits discovered minutes apart must not cost two rebuild+validate+compare cycles.

**把配置修改攒进一个周期。**在支付一次重建之前，把所有待处理对照表评判与已知问题中隐含的全部配置修改（`skip`、`titleMap` 条目、`cardMode`）扫一遍并一并应用——相隔几分钟发现的两个修改不应付出两次重建+校验+比对的代价。

**Compare run died partway** (browser crash, OOM): the sheets it captured are valid - grade them first, then re-run; carry-forward scopes the recapture to the gap. Never restart a crashed run with `--force` (it clears the grades you just earned).

**比对运行中途死亡**（浏览器崩溃、OOM）：它已捕获的对照表有效——先评分，再重跑；携带传递把重新捕获限定在缺口范围内。绝不要用 `--force` 重启崩溃的运行（它会清除你刚刚赢得的评分）。

**On a large DS, verify the fix is right BEFORE paying the full rebuild**: run the targeted loop below on one affected component (or probe its rendered page) first - a wrong guess validated by a full rebuild costs the whole cycle. **Intermediate validates can sample**: global breakage is systemic by nature, so `--render-sample 10` answers "did the fix work?" at a fraction of the cost; the FULL render-check is required at the §4d/§6 upload gate whenever anything render-affecting moved - on an anchored re-sync the §7 driver applies that rule automatically (the tier rule lives there).

**在大型 DS 上，先验证修复是否正确，再支付全量重建**：先在某个受影响组件上运行下方的定向循环（或探测其渲染页面）——被全量重建背书的错误猜测会烧掉整个周期。**中间校验可以抽样**：全局性破坏本质上是系统性的，`--render-sample 10` 以极小的代价回答"修复是否生效？"；只要任何影响渲染的东西变动了，§4d/§6 上传闸口就要求完整渲染检查——在有锚点的重新同步中，§7 driver 会自动套用该规则（分层规则就在那里）。

The `.tsx`-only targeted loop:
  ```bash
  node .ds-sync/lib/preview-rebuild.mjs --config .design-sync/config.json --node-modules <nm> --out ./ds-bundle --components <Name>
  node .ds-sync/storybook/compare.mjs --out ./ds-bundle --storybook-static .design-sync/sb-reference --components <Name>
  ```

  仅 `.tsx` 的定向循环：

  The targeted loop recompiles previews but does not re-key grade contracts from source: a story-file edit followed by only this loop carries the old grade until the next full build or driver run re-keys it - route story edits through a full build (the driver does that automatically).

  定向循环会重新编译预览，但不会从源重新生成评分契约键：story 文件编辑后若只跑了这个循环，旧评分会一直保留，直到下一次全量构建或 driver 运行重新生成键——story 编辑应经由全量构建处理（driver 会自动这样做）。

### 4b. Solo phase - one, then a few / Solo 阶段——先一个，再几个

Do NOT fan out immediately. Global issues must be flushed into config first, or every subagent rediscovers them.

不要立即扇出。全局问题必须先被清刷进配置，否则每个子代理都会重新发现它们。

1. **One component.** Pick a simple, well-storied one (Button-like: several stories, no portals). Run the §4a loop until you've graded every story `match` from its images - settle for `close` only when an iteration stops improving it (rubric above). **Every fix becomes a bullet in `.design-sync/NOTES.md`**: symptom -> root cause -> fix, marked `[GENERAL]` when it isn't component-specific.

   1. **一个组件。**选一个简单、story 丰富的（Button 类：若干 story，无 portal）。运行 §4a 循环，直到你凭图像把每个 story 都评为 `match`——只有当迭代不再带来改善时才接受 `close`（标准见上）。**每个修复都成为 `.design-sync/NOTES.md` 里的一条要点**：症状 -> 根因 -> 修复，非组件专属时标记 `[GENERAL]`。

2. **Three more, chosen for diversity:** one compound/overlay (Dialog/Tabs), one icon- or asset-heavy **whose stories load remote images** (this is the `[ASSETS_BLOCKED]` canary - §3's row: a network-sandboxed shell blanks assets on BOTH panels, so grades falsely pass; surfacing it here costs one component's recapture, surfacing it after a roster-wide pass costs the whole pass), one theme/provider-sensitive - and make sure the set spans one **text-heavy** component (font/typography bugs hide from button-only solos and then invalidate a whole grading wave). Same loop, solo. *Incremental path:* the solo set, once every story grades `match` (or `close` per the rubric's acceptance bar), is the first verified batch - push it (base SKILL.md §3).

   2. **再选三个，以多样性为准：**一个复合/覆盖层组件（Dialog/Tabs），一个图标或资源重度组件**且其 story 加载远程图片**（这是 `[ASSETS_BLOCKED]` 的金丝雀——§3 那一行：网络沙箱 shell 会让两个面板的资源同时空白，评分因此虚假通过；在这里暴露只需重捕一个组件，在全花名册通过之后暴露则赔上整轮通过），一个主题/provider 敏感组件——并确保这组里包含一个**文本重度**组件（字体/排版 bug 在只有 button 的 solo 中会隐藏，随后让整个评分波次作废）。同样的循环，单人执行。*增量路径：*solo 集合一旦每个 story 都评为 `match`（或按标准接受线评为 `close`），就是第一个已验证批次——推送它（基础 SKILL.md §3）。

3. **First roster-wide capture - size-gated on the storied-component count.**

   3. **首次全花名册捕获——按含 story 组件数量做规模门控。**

   - **20 or fewer:** run one full `compare.mjs` over the roster. Background it through the shell tool's background mode and wait for the completion notification - §2.2's rule, restated here because this is where it gets violated: a foreground `sleep`-poll blocks the very notification that would wake you, and a `pgrep -f` loop matches its own command line and spins to timeout. (Headless / `-p` session: run it synchronously instead - there is no task-notification re-invocation in headless mode, so a backgrounded run is never resumed.) If >=30% of components fail with the *same* reason, that's a global issue you missed - fix it in config and re-run before fanning out. **Batch every skip and pairing fix the listing shows before rebuilding** - each rebuild+compare cycle costs minutes; fixing them one at a time pays that cost per item.
     - **20 个及以下：**对花名册运行一次全量 `compare.mjs`。通过 shell 工具的后台模式将其后台化并等待完成通知——§2.2 的规则在此重申，因为正是在这里它被违反：前台 `sleep` 轮询会阻塞那个本会唤醒你的通知，而 `pgrep -f` 循环会匹配到自身命令行并空转到超时。（Headless / `-p` 会话：改为同步运行——headless 模式下没有任务通知的再次唤起，后台化的运行永远不会被恢复。）若 >=30% 的组件以*相同*原因失败，那是你漏掉的全局问题——在配置中修复并重跑，然后再扇出。**重建前把清单显示的每一个 skip 与配对修复攒批处理**——每个重建+比对周期耗数分钟；逐项修复就是逐项支付该代价。
   - **More than 20: do NOT run a monolithic full capture. Capture happens inside §4c's batches** - each subagent runs one scoped `compare.mjs --components <its batch>` and grades the sheets it just captured. This buys three things: scoped captures run concurrently (the roster renders in a fraction of a serial sweep's wall-clock); grading starts when the first batch's sheets exist instead of after the last component renders; and when a wave surfaces a `[GENERAL]` issue, the work at risk is the few batches graded so far, not the whole roster's captures and grades. The >=30% same-reason check moves with the capture - it becomes the wave-1 learnings review (§4c between-waves). The roster-wide run you do NOT skip is the §4d receipt: by then everything is graded, so it carries components forward instead of recapturing them and costs seconds, not minutes.
     - **超过 20 个：不要运行单体式全量捕获。捕获发生在 §4c 的批次内**——每个子代理运行一次限定的 `compare.mjs --components <its batch>`，并为其刚捕获的对照表评分。这买到三件事：限定的捕获并发运行（花名册的渲染时间只是串行扫描墙钟时间的零头）；第一批的对照表一存在评分即开始，而不必等最后一个组件渲染完；当某波暴露出一个 `[GENERAL]` 问题时，处于风险中的只是至今已评分的少数批次，而不是整个花名册的捕获与评分。>=30% 同因检查随捕获移动——它变为第 1 波的 learnings 复盘（§4c 波间）。你不应跳过的全花名册运行是 §4d 回执：届时一切已评分，它携带组件前进而非重新捕获，代价是秒级而非分钟级。

### 4c. Fan-out - parallel subagents / 扇出——并行子代理

Partition the components that still need work into batches of 5-8 - on a large DS (§4b step 3's >20 gate) that is every component outside the solo set, most with no sheet captured yet; after a small-DS full capture it is the non-matching set. Group related components together (shared providers, shared fixtures - one diagnosis then serves the whole batch). Launch up to 4 subagents per wave (Agent tool, in one message so they run concurrently). Four is also the browser-concurrency cap: each subagent's scoped compare runs its own chromium, and more than ~4 concurrent captures risks launch failures from machine-level contention. For each subagent, fill every `{...}` in this prompt and paste the **current** NOTES.md content in (subagents inherit the solo phase's learnings through it):

把仍需处理的组件划成每批 5-8 个——在大型 DS（§4b 第 3 步的 >20 门控）上这是 solo 集合之外的每个组件，多数还没有捕获对照表；小型 DS 全量捕获之后则是未匹配集合。把相关组件分在一组（共享 provider、共享 fixtures——一次诊断即可服务整批）。每波最多启动 4 个子代理（Agent 工具，放在同一条消息里使其并发运行）。4 也是浏览器并发上限：每个子代理的限定比对运行自己的 chromium，超过约 4 个并发捕获会因机器级资源争用而有启动失败风险。对每个子代理，填写此提示词中的每个 `{...}`，并把**当前**的 NOTES.md 内容粘贴进去（子代理经由它继承 solo 阶段的经验）：

```text
Fix design-sync previews so they match the repo's own storybook render.
Repo: {REPO_ROOT}. Your components (yours alone): {COMPONENT_LIST}.

Why this matters: this design system is being synced to claude.ai/design, where
a design agent will build real UIs from this exact compiled bundle. The
storybook render is the proof of how each component is supposed to look; a
preview that matches it proves the component arrived intact, and one that
doesn't means every design the agent builds with it will be wrong the same way.

Artifacts per component (read these first):
- {OUT}/_screenshots/compare/<group>__<Name>.png - the true storybook render (left) vs the true preview render (right), per story. Full-res originals in {OUT}/_screenshots/compare/raw/.
- .design-sync/.cache/compare/<Name>.json - pairing facts + shot paths (no similarity scores - your eyes are the judge).
- The preview source (real JSX importing from '{PKG}'): .design-sync/previews/<Name>.tsx when owned, else the generated .design-sync/.cache/previews/<Name>.tsx. Your fixes are written to .design-sync/previews/<Name>.tsx (step 2).
- {OUT}/.stories-map.json - maps components to story ids; find each story's source file via its id in .design-sync/sb-reference/index.json (`importPath`). The story source is the authority on intended props/composition.
- .ds-sync/storybook/SKILL.md §4 - the grading rubric and fix decision tree.

First action, once for the whole batch: if any of your components has no compare sheet yet, run
  node .ds-sync/storybook/compare.mjs --out {OUT} --storybook-static {SB_REF} --components {COMPONENT_LIST}
One scoped run captures every missing sheet in your batch (one browser launch, not one per component); components already graded with unchanged sources skip automatically.

Per component (max 3 iterations):
1. Read the sheet; judge the primary story FROM THE TWO IMAGES (raw PNGs when the sheet is too small) per the §4 sampling rule - exhaustively when the component has portals, theme/provider sensitivity, an owned preview, or any warning; diagnose failures via the decision tree.
2. Copy .design-sync/.cache/previews/<Name>.tsx to .design-sync/previews/<Name>.tsx and DELETE its first-line `// @ds-preview generated ...` marker (owned files live in previews/, win over the generated twin, and are durable + committed; an in-place cache edit survives rebuilds on this machine but is gitignored and vanishes on a fresh clone). The `@ds-stories/...` imports work unchanged from the new location. Mirror the story's JSX; inline story-local fixture data.
3. node .ds-sync/lib/preview-rebuild.mjs --config .design-sync/config.json --node-modules {NM} --out {OUT} --components <Name>
4. node .ds-sync/storybook/compare.mjs --out {OUT} --storybook-static {SB_REF} --components <Name>   (your edit changed the component's contract, so this clears its old grade - that's intended)
5. Re-Read the fresh sheet and Write your verdicts to .design-sync/.cache/compare/<Name>.grade.json ({"stories": {"<story>": {"verdict": "match|close|mismatch", "note": "..."}}}); siblings you trust under the §4 sampling rule get {"verdict": "match", "basis": "sibling-trusted"} - written in the same single grade.json Write, no image opens for them. Done when you grade every story match. A close story is still a fix target - if you can name the delta, try the knob for it; accept close only when an iteration didn't improve it or there's no actionable cause, and the note must say what's off AND what you tried. Blocked after 3 iterations -> grade honestly (mismatch/close + note), record the exact blocker, move on.

HARD RULES - violating these corrupts other agents' work:
- Edit ONLY .design-sync/previews/{<your components>}.tsx, your components' .design-sync/.cache/compare/*.grade.json files, and .design-sync/learnings/{BATCH_ID}.md.
- NEVER edit .design-sync/config.json, .design-sync/NOTES.md, .ds-sync/, or any other component's files.
- NEVER run package-build.mjs or package-validate.mjs - they rewrite the shared bundle. preview-rebuild.mjs + compare.mjs scoped via --components are your only build commands.
- NEVER write an image-judged grade for images you haven't Read in this iteration. A sibling-trusted verdict must carry "basis": "sibling-trusted" and is allowed only when the image-judged primary story graded match and the component is warning-free (§4 sampling rule).
- A story that doesn't render in storybook either (sb-error) needs cfg.overrides.<Name>.skip; likewise [PORTAL?] needs cfg.overrides.<Name>.cardMode "single". Both are config edits you may NOT make - record them in your learnings file and final report; the orchestrator applies them. NEVER "fix" overlay bleed by neutralizing a story's open state in the .tsx - that destroys the fidelity being verified.
- If the SAME root cause appears in 2+ of your components - or even once when the cause is config-level (provider/css/font/token/import resolution) - STOP on those components: it's global. Write it to your learnings file `[GENERAL]`, report it, do not work around it per-component. Per-component fixes for a global cause are worse than waste: nothing ever machine-deletes `.design-sync/previews/`, so an owned preview you land for it persists and SHADOWS the corrected generated preview on every future build.

Learnings: append to .design-sync/learnings/{BATCH_ID}.md as you go - one bullet per discovery:
`<Component>: <symptom> -> <root cause> -> <fix>`, prefixed [GENERAL] if it applies beyond that component.

Known repo gotchas (read before starting):
{CURRENT_NOTES_MD_CONTENT}

Final report: per component - match/close/blocked + one-line reason; then any [GENERAL] learnings verbatim.
```

**Between waves (orchestrator) - the learnings fold is mandatory, not optional:**

**波次之间（编排者）——learnings 折叠是强制的，不是可选项：**

1. Read every `.design-sync/learnings/*.md`. Promote `[GENERAL]` bullets into `.design-sync/NOTES.md` (dedup; keep them terse), then delete each learnings file you've folded. Full `compare.mjs` runs print `[LEARNINGS_UNMERGED]` while any learnings file exists, and the §4d driver receipt fails its verdict on the same condition - an overlooked fold can't silently ship.

   1. 读取每个 `.design-sync/learnings/*.md`。把 `[GENERAL]` 要点提升进 `.design-sync/NOTES.md`（去重；保持精炼），然后删除每个已折叠的 learnings 文件。只要任何 learnings 文件存在，全量 `compare.mjs` 运行就会打印 `[LEARNINGS_UNMERGED]`，§4d driver 回执在同样条件下判定失败——被忽视的折叠不可能静默上线。

2. **Act on every `[GENERAL]` learning NOW, before the next wave launches - however few components showed it.** A 2-of-24 incidence is still global; a wave dispatched past an un-actioned `[GENERAL]` re-pays it per component, and those grades wash out when the config fix finally lands. Apply the config fix, **delete any owned previews subagents authored to work around that same cause** (owned files are never machine-deleted - left in place they shadow the fix), then full rebuild (a real one - step 3's batch push uploads the on-disk files, so never a `--skip-dts` stub) + validate. Then prove the fix worked with a scoped `compare.mjs --components` on 1-2 components the issue actually hit - **do not run a roster-wide compare mid-campaign.** The rebuild already cleared whatever grades the fix's contract change touched; those components simply rejoin the queue, the next wave's scoped runs recapture them, and the §4d receipt settles the whole roster at the end. A roster-wide run mid-campaign that *captures* a large share of components is a symptom, not a routine step: either captured components were never graded (each batch must grade everything it captures) or a global-slice config edit cleared grades that were already earned - diagnose before paying for the render time.

   2. **在下一波启动之前，立即处理每一条 `[GENERAL]` 经验——无论有多少组件表现出它。**24 分之 2 的发生率也仍然是全局的；带着未处理的 `[GENERAL]` 派出的一波会按组件重新支付代价，而当配置修复最终落地时那些评分也会被冲掉。应用配置修复，**删除子代理为绕过同一成因而编写的任何自有预览**（自有文件永不被机器删除——留在原地会遮蔽修复），然后全量重建（真实构建——第 3 步的批次推送上传的是磁盘上的文件，因此绝不要 `--skip-dts` 的 stub）+ 校验。然后用限定的 `compare.mjs --components` 在 1-2 个真正受该问题影响的组件上证明修复有效——**战役中途不要运行全花名册比对。**重建已经清除了修复的契约变更所触及的评分；那些组件只是重新加入队列，下一波的限定运行重新捕获它们，§4d 回执最后对整个花名册结算。战役中途一个*捕获*了大部分组件的全花名册运行是症状，不是例行步骤：要么被捕获的组件从未被评分（每批必须给它捕获的一切评分），要么一次全局切片的配置修改清除了已经赢得的评分——在支付渲染时间之前先诊断。

3. *Incremental path:* push the wave's components that now meet the §4d grade bar (every story `match`, or `close` per the rubric) as a verified batch (base SKILL.md §3) - after steps 1-2, so a global fix from this wave rebuilds them first.

   3. *增量路径：*把本波中现已达到 §4d 评分线（每个 story `match`，或按标准评为 `close`）的组件作为已验证批次推送（基础 SKILL.md §3）——在第 1-2 步之后，使本波的全局修复先重建它们。

4. Next wave gets the updated NOTES.md content and the still-failing components. After the last wave, repeat step 1 for whatever remains and delete `.design-sync/learnings/`.

   4. 下一波获得更新后的 NOTES.md 内容与仍然失败的组件。最后一波之后，对剩余内容重复第 1 步并删除 `.design-sync/learnings/`。

### 4d. Done criteria + report / 完成判据 + 报告

- **One §7 driver run is the closing receipt - every path.** Make the session's FINAL build the driver (`resync.mjs`); omit `--remote` when no anchor exists (first syncs, recovered projects) - a full re-verify of an anchored project still passes it. The gate is the driver's verdict: `ok: true` with `verification.pendingGrade` empty. Its capture scope is the capturable subset of its worklist - every storied component on a first sync, the `changed`+`added` set on a re-sync - with carried-forward grades skipped, so the receipt costs a scoped pass, not a full re-capture (uncapturable members re-ship via the upload partition with nothing to grade; verified-by-upload components are outside the gate). The driver checks `.design-sync/learnings/` itself and fails the verdict with `[LEARNINGS_UNMERGED]` while any unfolded learnings file remains (`.compare-report.json` aggregation stays full-run-only). On this final run every in-scope component should print `carried forward` with zero `grade cleared` - that line IS the proof the next sync will be fast. A cleared grade on a no-change run means a nondeterministic source input (volatile story content) - chase it now; a driver-triggered `[SPOT_CHECK]` is not that (pipeline churn being auto-verified - confirm the sheets and move on).
  - **一次 §7 driver 运行就是收尾回执——所有路径皆然。**把本次会话的最终构建交给 driver（`resync.mjs`）；不存在锚点时省略 `--remote`（首次同步、被恢复的项目）——对有锚点项目做完整重验证依然会通过。闸口就是 driver 的判定：`ok: true` 且 `verification.pendingGrade` 为空。其捕获范围是其工作清单中可捕获的子集——首次同步时的每个含 story 组件、重新同步时的 `changed`+`added` 集合——已携带的评分被跳过，因此回执的代价是一次限定通过，而非全量重捕获（不可捕获的成员经由上传分区重新发布，无需评分；"已验证随上传"组件在闸口之外）。driver 会自行检查 `.design-sync/learnings/`，只要还有未折叠的 learnings 文件就以 `[LEARNINGS_UNMERGED]` 判定失败（`.compare-report.json` 聚合仍仅限全量运行）。在这次最终运行中，每个范围内组件都应打印 `carried forward` 且零 `grade cleared`——这行字就是下一次同步会很快的证明。无变化运行中出现被清除的评分意味着某个非确定性源输入（易变的 story 内容）——现在就去追查；driver 触发的 `[SPOT_CHECK]` 不算（那是流水线扰动被自动验证——确认对照表后继续）。
- Every IN-SCOPE storied component has a current `.grade.json` with every story `match` - or `close` meeting the rubric's acceptance bar (§4) - or skipped via `cfg.overrides.<Name>.skip` with a NOTES.md justification. The mechanical check is the driver's `verification.pendingGrade`: a component listed there has stories without current verdicts and is not done (verified-by-upload components are exempt).
  - 每个范围内的含 story 组件都有一份当前的 `.grade.json`，其中每个 story 为 `match`——或达到标准接受线的 `close`（§4）——或经由 `cfg.overrides.<Name>.skip` 跳过并有 NOTES.md 理由。机械检查是 driver 的 `verification.pendingGrade`：被列在那里的组件有尚无当前评判的 story，即未完成（"已验证随上传"组件豁免）。
- `package-validate.mjs` still exits 0 after the final rebuild, with no unresolved `[FONT_MISSING]` (§4a - the one warning the compare oracle can't see).
  - 最终重建后 `package-validate.mjs` 仍以 0 退出，且没有未解决的 `[FONT_MISSING]`（§4a——比对基准看不见的唯一告警）。
- Call `DesignSync({method: 'report_validate', counts: {total, bad, thin, variantsIdentical, iterations}})` from the final `ds-bundle/.render-check.json` (written by `package-validate.mjs`; `iterations` = full rebuild passes). On a driver-scoped receipt (§7) that file is absent (skip tier) or covers only the sample - re-run the driver with `--render-sample 0` first when this call needs full counts; on a no-change re-sync that uploads nothing, skip the call.
  - 从最终的 `ds-bundle/.render-check.json`（由 `package-validate.mjs` 写出；`iterations` = 全量重建遍数）调用 `DesignSync({method: 'report_validate', counts: {total, bad, thin, variantsIdentical, iterations}})`。在 driver 限定范围的回执（§7）上该文件缺席（跳过层）或只覆盖抽样——当此调用需要完整计数时，先用 `--render-sample 0` 重跑 driver；在不发布任何东西的无变化重新同步上，跳过该调用。
- NOTES.md has a current **Re-sync risks** section, written now while you still know them: what can silently go stale (data inlined into config, neutralized story exports, owned previews tied to upstream APIs), what was verified only partially (story caps, accepted `close` rationales), and what the build assumed (toolchain version, CDN-fetched assets). Fixes record what you did; this section tells the next run what to watch.
  - NOTES.md 有一个最新的 **Re-sync risks** 小节，趁你还记得时现在写下：哪些东西可能静默过时（内联进配置的数据、被中和的 story 导出、与上游 API 绑定的自有预览），哪些只得到部分验证（story 上限、被接受的 `close` 理由），构建做了哪些假设（工具链版本、CDN 拉取的资源）。修复记录你做了什么；这一小节告诉下一次运行要盯什么。
- Tell the user: N/M components graded match, which are `close` (and why that's acceptable), which were skipped and why.
  - 告诉用户：N/M 个组件评为 match，哪些是 `close`（以及为何可接受），哪些被跳过及原因。

## 5. When the repo is strange - escape hatches / 当仓库不寻常时——逃生舱口

First runs against unusual repos WILL hit things the defaults don't cover. Every heuristic has a committed override - the rule is: **never hand-patch generated output; put the fix in the file the next run reads.** Map from failure class to knob:

对不寻常仓库的首次运行必然会撞上默认值未覆盖的东西。每条启发式都有对应的已提交覆盖——规则是：**绝不要手工修补生成输出；把修复放进下一次运行会读取的文件。**从失败类别到旋钮的映射：

| The repo's strangeness | Knob | Lives in |
|---|---|---|
| Nonstandard build/entry (`module` points at TS source, exotic dist layout) | `cfg.entry`, `cfg.buildCmd` | config |
| CSS built by a separate pipeline / no dist sidecar / CSS-in-JS | `cfg.cssEntry` if there's a file; otherwise rely on `[CSS_FROM_STORYBOOK]` - the converter scrapes the **compiled** CSS out of `sb-reference`, which is the universal catch-all: however weird the pipeline, its output is in the storybook build | config |
| Tokens shipped as a separate package | `cfg.tokensPkg` | config |
| Fonts from a runtime service / proprietary CDN | `cfg.extraFonts`, `cfg.runtimeFontPrefixes` | config |
| Icons or components on subpath exports | `cfg.extraEntries` | config |
| Naming conventions (story titles != export names) | `cfg.titleMap`; story<->cell pairing also falls back to order | config |
| Decorators/providers that won't bundle (vite-only plugins, MDX, aliases) | `cfg.provider` - an explicit chain beats the decorator bundle; `probe.mjs` infers it from the live storybook; or compose providers **inline in the component's own `.tsx`** (an owned preview can import and wrap anything the package exports) | config / previews |
| Stories that can't render statically (MSW, data fetching, interaction tests) | `cfg.overrides.<Name>.skip` + a NOTES.md line saying why. Skip removes the story's cell, but the wrapper still imports the whole story MODULE - if the file crashes at import (module-scope fetch/worker), own the `.tsx` and drop the import instead | config |
| `[PORTAL?]` - overlay/portal stories paint outside their cells in the grid card | `cfg.overrides.<Name>.cardMode: "single"` (+ optional `primaryStory`, `viewport: "WxH"`) - single-story card, fixed-position containment, declared product viewport. Compare still grades every story via `?story=` | config |
| `[GRID_OVERFLOW]` - validate measured the grid card's geometry: `wide` = stories render wider than their cells (the cell clip crops them in the product); `escape` = fixed/portal content positions outside any cell | apply the override the warn names - `wide` -> `cardMode: "column"` (one story per row, full card width, all stories kept); `escape` -> `cardMode: "single"` + `primaryStory`. Structured copy in `.render-check.json` (`gridOverflow`, `gridOverflowCells`, `suggestedOverride`). Batch every flagged component into ONE targeted rebuild (`preview-rebuild.mjs --components A,B,C`) - presentation-only edits don't trip `[CONFIG_STALE]` and grades carry. Don't chase a clean re-validate to confirm: the applied remedy can't re-flag (single is fully exempt; column can't re-flag `wide` - escape stays monitored, so a portal story added later still surfaces); eyeball `.review.html` if you want visual confirmation | config |
| `[EXPORT_COLLISION]` - a sibling package (icons etc.) exports names the main package also exports | the main package wins the global merge, so stories importing the losing name from the sibling render the wrong thing | the log names the fix: `cfg.storyImports.bundle: ["<sibling>"]` |
| `[FILE_TOO_LARGE]` - a build output exceeds the upload's 12 MB per-file cap | usually a dev-only heavyweight bundled into a preview or the decorator bundle (syntax highlighters, icons-as-code) | slim it NOW, before grading - a post-grade slim of an owned preview re-grades that component |
| `[PROVIDER_UNEXPORTED]` - a `cfg.provider` component isn't a bundle export | the build exits 1 before emitting any component previews or docs - the output dir is left partial; rebuild after fixing | use the exact exported name, or re-export it via `cfg.extraEntries`. The check reads the bundle's own export list, so absence is reliable; names hidden behind bundled CommonJS re-exports can't be enumerated - those build with a `[PROVIDER_UNVERIFIED]` warning instead; if every preview then fails "Element type is invalid", the name is wrong |
| A story import resolves the wrong way (shimmed when it should bundle, or vice versa - any import style) | `cfg.storyImports.shim` / `cfg.storyImports.bundle` - substring patterns matched against resolved paths (bare package imports shim by **specifier**, without resolution - pattern-match the specifier for those). Unknown package subpaths (`<pkg>/utils`) bundle by default; if one should ride the global instead, add it to `cfg.extraEntries`. In the package's own source repo a bundled self-import has nothing to resolve to - symlink `node_modules/<pkg>` -> the built `dist/` first | config |
| Story files import an asset type the defaults can't load (`.yaml`, `?raw`, svg-as-component) | `cfg.storyImports.loaders` - an esbuild loader map merged over the defaults (e.g. `{".yaml": "text"}`) | config |
| Generated preview has wrong props/composition | copy `.design-sync/.cache/previews/<Name>.tsx` to `.design-sync/previews/<Name>.tsx` minus its marker line (owned forever) | previews |
| Source/docs discovery misses (unusual repo layout) | `cfg.componentSrcMap`, `cfg.docsMap`, `cfg.dtsPropsFor`, `cfg.srcDir` | config |
| Anything deeper - custom story format, exotic args extraction, CSS transform | fork the adapter: copy the bundled lib module to `.design-sync/overrides/<name>.mjs` and declare it in `cfg.libOverrides` with a one-line reason (the build cross-checks both directions: `[OVERRIDE_UNDECLARED]` / `[OVERRIDE_MISSING]`). Forks are committed, so re-syncs use them automatically. **`emit.mjs` and `bundle.mjs` are app-contract surface - never fork them.** | `.design-sync/overrides/` |

| 仓库的怪异之处 | 旋钮 | 位于 |
|---|---|---|
| 非标准构建/入口（`module` 指向 TS 源码、奇异的 dist 布局） | `cfg.entry`、`cfg.buildCmd` | config |
| CSS 由独立流水线构建 / 无 dist 伴生文件 / CSS-in-JS | 有文件则用 `cfg.cssEntry`；否则依赖 `[CSS_FROM_STORYBOOK]`——转换器从 `sb-reference` 中刮取**已编译**的 CSS，这是万能兜底：无论流水线多怪，其输出都在 storybook 构建里 | config |
| Tokens 以独立包发布 | `cfg.tokensPkg` | config |
| 字体来自运行时服务 / 专有 CDN | `cfg.extraFonts`、`cfg.runtimeFontPrefixes` | config |
| 图标或组件在子路径导出上 | `cfg.extraEntries` | config |
| 命名约定（story 标题 != 导出名） | `cfg.titleMap`；story<->单元配对还会回退到顺序 | config |
| 无法打包的装饰器/provider（vite 专属插件、MDX、别名） | `cfg.provider`——显式链优于装饰器打包；`probe.mjs` 从活体 storybook 推断；或在组件自己的 `.tsx` 中**内联组合 provider**（自有预览可以导入并包裹包导出的任何东西） | config / previews |
| 无法静态渲染的 story（MSW、数据获取、交互测试） | `cfg.overrides.<Name>.skip` + NOTES.md 一行说明原因。skip 会移除该 story 的单元，但包装器仍会导入整个 story 模块——若文件在导入时崩溃（模块作用域 fetch/worker），改为自有化 `.tsx` 并删除该导入 | config |
| `[PORTAL?]`——覆盖层/portal story 在网格卡片中涂到单元之外 | `cfg.overrides.<Name>.cardMode: "single"`（可选 `primaryStory`、`viewport: "WxH"`）——单 story 卡片、fixed 定位收容、声明产品视口。比对仍通过 `?story=` 给每个 story 评分 | config |
| `[GRID_OVERFLOW]`——validate 量得网格卡片的几何：`wide` = story 渲染宽度超出其单元（产品中单元裁剪会切掉它们）；`escape` = fixed/portal 内容定位到任何单元之外 | 应用该告警点名的覆盖——`wide` -> `cardMode: "column"`（每行一个 story、整卡宽度、保留全部 story）；`escape` -> `cardMode: "single"` + `primaryStory`。结构化副本在 `.render-check.json`（`gridOverflow`、`gridOverflowCells`、`suggestedOverride`）。把所有被点名的组件攒进一次定向重建（`preview-rebuild.mjs --components A,B,C`）——纯呈现类编辑不触发 `[CONFIG_STALE]`，评分携带。不要为了确认而追一次干净的重新校验：所施加的补救不可能再被标记（single 完全豁免；column 不可能再触发 `wide`——escape 仍受监控，所以之后新增的 portal story 仍会浮现）；想要视觉确认就目检 `.review.html` | config |
| `[EXPORT_COLLISION]`——兄弟包（图标等）导出了主包也导出的名称 | 主包在全球合并中获胜，因此从兄弟包导入落败名称的 story 会渲染错误的东西 | 日志点名修复方法：`cfg.storyImports.bundle: ["<sibling>"]` |
| `[FILE_TOO_LARGE]`——某个构建输出超过上传的每文件 12 MB 上限 | 通常是某个被打进预览或装饰器 bundle 的仅开发用重物（语法高亮器、icons-as-code） | 在评分之前立即瘦身——评分后再瘦身自有预览会触发该组件重评 |
| `[PROVIDER_UNEXPORTED]`——`cfg.provider` 的某组件不是 bundle 导出 | 构建以 1 退出，且未产出任何组件预览或文档——输出目录处于部分状态；修复后重建 | 使用确切的导出名，或经由 `cfg.extraEntries` 再导出。该检查读取 bundle 自身的导出清单，因此缺席是可靠的；藏在打包后的 CommonJS 再导出之后的名称无法枚举——那些会带 `[PROVIDER_UNVERIFIED]` 告警构建；若此后每个预览都报 "Element type is invalid"，则名称有误 |
| 某 story 导入被解析到错误方式（该打包的被 shim，或反之——任何导入风格） | `cfg.storyImports.shim` / `cfg.storyImports.bundle`——对解析后路径做子串模式匹配（裸包导入按**说明符** shim，不经解析——对它们按说明符做模式匹配）。未知包子路径（`<pkg>/utils`）默认打包；若某个应改搭全局便车，把它加入 `cfg.extraEntries`。在包自身的源码仓库中，被打包的自导入无处可解析——先 symlink `node_modules/<pkg>` -> 已构建的 `dist/` | config |
| story 文件导入默认值无法加载的资源类型（`.yaml`、`?raw`、svg-as-component） | `cfg.storyImports.loaders`——合并覆盖默认值的 esbuild loader 映射（如 `{".yaml": "text"}`） | config |
| 生成的预览 props/组合错误 | 把 `.design-sync/.cache/previews/<Name>.tsx` 复制为 `.design-sync/previews/<Name>.tsx`，去掉标记行（永久自有） | previews |
| 源码/文档发现遗漏（不寻常的仓库布局） | `cfg.componentSrcMap`、`cfg.docsMap`、`cfg.dtsPropsFor`、`cfg.srcDir` | config |
| 更深层的一切——自定义 story 格式、奇异 args 提取、CSS 变换 | fork 适配器：把打包的 lib 模块复制到 `.design-sync/overrides/<name>.mjs`，并在 `cfg.libOverrides` 中声明，附一行理由（构建会双向交叉检查：`[OVERRIDE_UNDECLARED]` / `[OVERRIDE_MISSING]`）。fork 会被提交，因此重新同步自动使用它们。**`emit.mjs` 与 `bundle.mjs` 是应用契约表面——绝不 fork。** | `.design-sync/overrides/` |

For **story handling** specifically, the fork points by concern: `story-imports.mjs` (ALL import-resolution policy for preview compiles - the seam built for per-repo customization; honored by both the full build and `preview-rebuild.mjs`), `source-storybook.mjs` (index.json discovery, title->component mapping, story-source resolution + export pairing), `preview-gen-storybook.mjs` (the wrapper template / composeStories semantics), `css-fallback.mjs` (CSS/font scraping from the storybook build). Fork the *narrowest* module that owns the breakage, keep its export signature, and record what the repo does differently in NOTES.md - the next sync inherits all of it. A fork loads from `.design-sync/overrides/` while its siblings stay in the staged scripts - repoint the fork's relative imports (`./common.mjs` etc.) at `../../.ds-sync/lib/`. A fork that imports a bare converter dep (`esbuild`) also needs `ln -sfn ../.ds-sync/node_modules .design-sync/node_modules` so node can resolve it from the fork's location - once per clone, not once ever: the link is gitignored (`node_modules` rules) while the committed fork that needs it survives the clone, so recreating it is part of the fresh-clone setup.

专就 **story 处理**而言，按关注点划分的 fork 点：`story-imports.mjs`（预览编译的全部导入解析策略——为按仓库定制而建的接缝；全量构建与 `preview-rebuild.mjs` 都遵守它）、`source-storybook.mjs`（index.json 发现、标题->组件映射、story 源解析 + 导出配对）、`preview-gen-storybook.mjs`（包装器模板 / composeStories 语义）、`css-fallback.mjs`（从 storybook 构建刮取 CSS/字体）。fork *最窄*的拥有该破损的模块，保持其导出签名，并在 NOTES.md 记录该仓库有何不同——下一次同步继承这一切。fork 从 `.design-sync/overrides/` 加载而其兄弟仍留在暂存脚本中——把 fork 的相对导入（`./common.mjs` 等）重指向 `../../.ds-sync/lib/`。导入裸转换器依赖（`esbuild`）的 fork 还需要 `ln -sfn ../.ds-sync/node_modules .design-sync/node_modules`，让 node 能从 fork 所在位置解析——每次克隆一次，不是一生一次：链接被 gitignore（`node_modules` 规则），而需要它的已提交 fork 能在克隆后幸存，因此重建链接属于新克隆设置的一部分。

The ladder's last rung, for repos genuinely outside the converter's envelope: **the upload format is the contract, not the converter** (see the base skill). Generate the layout however the repo allows - but `package-validate.mjs` and the compare/grading gate apply unchanged to whatever you produce. The oracle is never forked.

阶梯的最后一级，给真正超出转换器包络的仓库：**上传格式才是契约，转换器不是**（见基础技能）。用仓库允许的任何方式生成该布局——但 `package-validate.mjs` 与比对/评分闸口对你产出的任何东西都原样适用。基准永不被 fork。

Everything in that table is a committed file, and §2.3 requires reading the existing config + NOTES.md before doing anything - so run N+1 replays every decision run N made. When you fix something on a strange repo, ask: "which committed file makes this automatic next time?" If the answer is none, that's a NOTES.md entry at minimum - and likely a missing row here worth reporting.

那张表里的一切都是已提交文件，而 §2.3 要求在做任何事之前先读现有配置 + NOTES.md——所以第 N+1 次运行会重放第 N 次运行做过的每个决定。在怪异仓库上修好某事后，问自己："哪个已提交文件能让这在未来自动生效？"若答案是没有，那至少是一条 NOTES.md 条目——而且很可能是这里缺失的、值得上报的一行。

## Author the conventions header (before upload) / 撰写约定头部（上传前）

With previews verified - whether newly authored or carried forward by a re-sync - run the conventions-authoring step in the base SKILL.md ("Author the conventions header") - it distills what you just learned making the previews render into `.design-sync/conventions.md`, wired via the `readmeHeader` config key. Ordering matters: author the file and set the key FIRST, then rebuild per the base step's **rebuild rule** (a fresh DRIVER run on every path - first syncs omit `--remote`) so the generated README actually carries the header and the §4d receipt describes the build §6 uploads. Then proceed to Upload below.

预览验证完毕后——无论是新撰写还是由重新同步携带而来——运行基础 SKILL.md 中的约定撰写步骤（"Author the conventions header"）——它把你刚才让预览渲染所学到的内容提炼进 `.design-sync/conventions.md`，经 `readmeHeader` 配置键接线。顺序很重要：先撰写文件并设置键，然后按基础步骤的**重建规则**重建（每条路径都是全新 DRIVER 运行——首次同步省略 `--remote`），使生成的 README 真正携带头部，且 §4d 回执描述的正是 §6 上传的构建。然后进行下方的上传。

## 6. Upload / 上传

Which of the two paths applies was decided by the base skill §1 router (pinned-at-run-start -> atomic; otherwise empty -> incremental, non-empty -> atomic):

适用两条路径中的哪一条由基础技能 §1 路由器决定（运行开始时已 pin -> 原子路径；否则目标为空 -> 增量路径，非空 -> 原子路径）：

**Incremental path** (first sync into an empty project): the plan has been open since this file's §3 gate and verified batches have already landed. After §4d passes and the conventions-header step has run (base SKILL.md - it must precede the upload its rebuild feeds), run the close-out in base SKILL.md §3 - sentinel fence -> full content writes -> reconciliation deletes -> sentinel re-arm -> `_ds_sync.json` last. This section's chunking, hygiene, and stays-local rules apply to those writes; `projectId` was already recorded in §1; the handoff audit at the end of this section still applies. Skip the rest of this section's sequence - it is the atomic path.

**增量路径**（首次同步进空项目）：自本文件 §3 闸口起计划就已打开，已验证批次也已落地。§4d 通过且约定头部步骤运行后（基础 SKILL.md——它必须先于其重建所供的上传），运行基础 SKILL.md §3 的收尾——哨兵围栏 -> 全量内容写入 -> 对账删除 -> 哨兵重新布防 -> `_ds_sync.json` 最后。本节的分块、卫生与保留本地规则适用于那些写入；`projectId` 已在 §1 记录；本节末尾的交接审计仍然适用。跳过本节其余序列——那是原子路径。

**Atomic path** (re-sync, or any non-empty target - it may be in active use, so it updates in one pass after everything is verified): everything below. Only after §4d and the conventions-header step (base SKILL.md). `DesignSync(finalize_plan)` with `localDir: "./ds-bundle"`.

**原子路径**（重新同步，或任何非空目标——它可能正在被使用，因此在一切验证完毕后一次性更新）：以下全部内容。仅在 §4d 与约定头部步骤（基础 SKILL.md）之后。`DesignSync(finalize_plan)` 配 `localDir: "./ds-bundle"`。

- **Writes - everything, always** (full re-verifies and re-syncs alike): `writes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**", "_ds_bundle.js", "_ds_bundle.css", "styles.css", "README.md", "_ds_sync.json", "_ds_needs_recompile"]`. Re-uploading unchanged files is idempotent and cheap. An under-scoped writes list silently and permanently desyncs the project - full writes are the safe default.
  - **写入——一切，永远**（完整重验证与重新同步皆然）：`writes: ["components/**", "tokens/**", "fonts/**", "_vendor/**", "_preview/**", "guidelines/**", "_ds_bundle.js", "_ds_bundle.css", "styles.css", "README.md", "_ds_sync.json", "_ds_needs_recompile"]`。重复上传未变文件是幂等且廉价的。范围不足的 writes 清单会静默且永久地使项目失同步——全量写入是安全默认。
- **Deletes.** Anchored re-syncs: verbatim from the diff - copy `.sync-diff.json`'s `upload.deletePaths` exactly; never hand-derive the list, never pass `[]` when the diff lists paths. No anchor (a re-adopted or recovered non-empty project being fully re-verified): the diff can't see the project's history, so review its `list_files` NOW - before `finalize_plan` - for files this build doesn't produce, and put those reviewed paths in the plan's `deletes` (a delete not named in the plan is rejected).
  - **删除。**有锚点的重新同步：逐字来自 diff——原样照抄 `.sync-diff.json` 的 `upload.deletePaths`；绝不手推清单，diff 列出路径时绝不传 `[]`。无锚点（被重新接管或恢复的非空项目正在做完整重验证）：diff 看不到项目历史，因此现在——`finalize_plan` 之前——审查其 `list_files`，找出本次构建不产出的文件，把这些经审查的路径放进计划的 `deletes`（计划未点名的删除会被拒绝）。
- **The §4d closing receipt doubles as the upload's source of truth.** The session's FINAL build is already a §7 driver run (§4d); bare `package-build.mjs` runs wipe `.sync-diff.json`, and the driver's diff stage regenerates it, so `deletePaths` and `upload.any` describe the exact bytes you upload - one run is both the verification receipt and the upload manifest, with no separate full compare after it.
  - **§4d 收尾回执兼任上传的事实来源。**会话的最终构建已是 §7 driver 运行（§4d）；裸 `package-build.mjs` 运行会抹掉 `.sync-diff.json`，而 driver 的 diff 阶段会重新生成它，因此 `deletePaths` 与 `upload.any` 描述的正是你上传的确切字节——一次运行既是验证回执又是上传清单，其后无需单独的全量比对。
- **`upload.any === false` -> skip the upload entirely** - the project already matches this build. (The handoff audit below still applies.)
  - **`upload.any === false` -> 完全跳过上传**——项目已与本次构建一致。（下方的交接审计仍适用。）
- **`_ds_sync.json` is the absolute final write** - after all content writes, all deletes, and the sentinel re-arm, in its own `write_files` call. Uploaded early, a mid-plan failure leaves the anchor vouching for files the project doesn't have, and deterministic rebuilds mean no later sync would repair them.
  - **`_ds_sync.json` 是绝对的最终写入**——在所有内容写入、所有删除与哨兵重新布防之后，用它自己的 `write_files` 调用。若上传过早，计划中途失败会让锚点为项目并不拥有的文件背书，而确定性重建意味着后续任何同步都不会修复它们。

【评论】哨兵文件先行、内容写入次之、锚点文件绝后的写入顺序，是为了让锚点永远不为尚未真实存在的文件背书——一种常见的事务性发布（write barrier）设计。

- **What stays local**: `_sb/**` (storybook-static is a reference, never uploaded), dot-prefixed entries (`.stories-map.json`, `.compare-report.json`, `.ds-build-meta.json`, `.sb-static/`, `.sync-diff.json`), and `_screenshots/`. `_vendor/` and `_preview/` DO upload - the preview cards load React and the compiled previews from them.
  - **留在本地的**：`_sb/**`（storybook-static 是参考物，永不上传）、点前缀条目（`.stories-map.json`、`.compare-report.json`、`.ds-build-meta.json`、`.sb-static/`、`.sync-diff.json`），以及 `_screenshots/`。`_vendor/` 与 `_preview/` **会**上传——预览卡片从它们加载 React 与编译后的预览。

If `finalize_plan` is denied, **stop** - denial means the session can't approve, not that the arguments were wrong. Tell the user what was denied and ask how they'd like to proceed: try the approval again, or take the validated `ds-bundle/` and run the upload interactively themselves.

若 `finalize_plan` 被拒绝，**停下**——拒绝意味着会话无法批准，而非参数有误。告诉用户被拒绝了什么，并询问希望如何继续：再次尝试批准，或拿走已验证的 `ds-bundle/` 由用户自行交互式执行上传。

After plan approval, the upload is a fixed sequence:

计划获批后，上传是一个固定序列：

1. **Sentinel first**: `DesignSync(write_files, [{path: "_ds_needs_recompile", localPath: "_ds_needs_recompile"}])` - it fences the app's manifest/copy machinery against a half-uploaded state.

   1. **哨兵先行**：`DesignSync(write_files, [{path: "_ds_needs_recompile", localPath: "_ds_needs_recompile"}])`——它为应用的清单/复制机制加围栏，抵御半上传状态。

2. **All content writes**, chunked into <=256-file `write_files` calls under the same `planId`. The server also bounds payload BYTES, not just file count - batch binary-heavy dirs (fonts/, images) into smaller chunks, and on a 500 halve the chunk size and retry.

   2. **所有内容写入**，分块为同一 `planId` 下 <=256 文件的 `write_files` 调用。服务器还按字节（BYTES）限制载荷，不只是文件数——把二进制重的目录（fonts/、images）分批为更小的块，遇到 500 就把块大小减半重试。

3. **All deletes**: `DesignSync(delete_files)` over every path in `upload.deletePaths`. (No anchor: the paths you reviewed into the plan's `deletes` at `finalize_plan` - the deletes bullet above.) If `delete_files` rejects paths that don't exist remotely (floor-card components have no `_preview/` files), retry without the rejected entries - that not-found rejection is the ONLY failure you may continue past.

   3. **所有删除**：对 `upload.deletePaths` 中的每个路径执行 `DesignSync(delete_files)`。（无锚点：`finalize_plan` 时你审查后放进计划 `deletes` 的路径——上方的删除条目。）若 `delete_files` 拒绝远端不存在的路径（底座卡片组件没有 `_preview/` 文件），去掉被拒条目重试——这种 not-found 拒绝是你唯一可以继续越过的失败。

4. **Sentinel re-arm, then `_ds_sync.json` last.** The anchor goes after deletes too - a failed delete would leave remote files the refreshed anchor can no longer see.

   4. **哨兵重新布防，然后 `_ds_sync.json` 最后。**锚点也排在删除之后——一次失败的删除会留下刷新后的锚点再也看不见的远端文件。

Any other write/delete failure that retries don't clear means **STOP** - no sentinel re-arm, no `_ds_sync.json`. An un-anchored project merely re-verifies next sync; a fresh anchor over a half-applied upload is permanent.

任何重试也清不掉的其他写入/删除失败都意味着**停止**——不重新布防哨兵，不写 `_ds_sync.json`。无锚点项目不过是下次同步重新验证；盖在半应用上传之上的新锚点是永久性的。

**Upload hygiene**: keep file lists and chunk manifests under `.design-sync/` - never bare `/tmp` paths, where a stale list from another repo's sync uploads the wrong design system. Regenerate the list from the live `ds-bundle/` immediately before upload, and sanity-check it: component names belong to THIS design system, and the bundle's `window.<globalName>` matches. Finish with `DesignSync(list_files)` to confirm the count.

**上传卫生**：文件清单与分块清单保存在 `.design-sync/` 下——绝不要用裸 `/tmp` 路径，那里另一仓库同步留下的陈旧清单会传错设计系统。上传前一刻从活的 `ds-bundle/` 重新生成清单，并做合理性检查：组件名属于本设计系统，bundle 的 `window.<globalName>` 匹配。最后以 `DesignSync(list_files)` 确认数量。

Only after the post-upload `list_files` count verifies, **record `projectId` in `.design-sync/config.json`** if absent or different (this is a backstop - §1 records the id at target settlement for every route, so it's normally already present; what must never happen is recording an id here before the upload verifies, pinning a config to a project whose content isn't real yet) - it pins which project anchors future re-syncs. When done, tell the user: the project URL (`https://claude.ai/design/p/<projectId>`), component count, compare results summary, and that validate exited clean. The durable set (the rule in the handoff audit below: everything under `.design-sync/` not gitignored) must land in the repo for re-syncs to reuse every fix; verified-state lives with the uploaded `_ds_sync.json`, not in git. The handoff audit below covers the offer to commit.

只有在上传后的 `list_files` 数量核实之后，若 `.design-sync/config.json` 缺失或不同才**记录 `projectId`**（这是兜底——§1 在目标结算时为每条路径记录该 id，所以它通常已在；绝不可发生的是在上传核实前在此记录 id，把配置 pin 到一个内容还不真实的项目上）——它 pin 定哪个项目锚定未来的重新同步。完成后告诉用户：项目 URL（`https://claude.ai/design/p/<projectId>`）、组件数量、比对结果摘要，以及 validate 以 0 退出。持久集合（下方交接审计中的规则：`.design-sync/` 下一切未被 gitignore 的内容）必须落入仓库，重新同步才能复用每个修复；已验证状态随上传的 `_ds_sync.json` 存放，不在 git 里。下方交接审计涵盖提交提议。

**Last step - audit the handoff.** A future run is only as fast and correct as what this one leaves behind; verify it, don't assume it:

**最后一步——审计交接。**未来运行的速度与正确性取决于本次运行留下什么；去核实，不要想当然：

1. `git status` - the durable set (everything under `.design-sync/` that isn't gitignored - today config.json, NOTES.md, `conventions.md`, `previews/`, `overrides/`; the rule is the contract, so future durable files are in the set by construction) is the sync's repo footprint; `sb-reference/`, `learnings/`, `.cache/`, `.ds-sync/` are ignored. If this run created or changed any of the durable files, **offer to commit them and open a PR** (one commit, sync state only - no unrelated files). An uncommitted fix is a fix the next sync doesn't have.

   1. `git status`——持久集合（`.design-sync/` 下一切未被 gitignore 的内容——今天是 config.json、NOTES.md、`conventions.md`、`previews/`、`overrides/`；规则即契约，因此未来的持久文件按构造就在集合内）是同步的仓库足迹；`sb-reference/`、`learnings/`、`.cache/`、`.ds-sync/` 被忽略。若本次运行创建或修改了任何持久文件，**提议提交它们并开 PR**（一次提交，仅同步状态——不含无关文件）。未提交的修复就是下一次同步没有的修复。

2. Re-read NOTES.md as if you were the next agent, knowing nothing from this session: could you skip today's debugging with only what's written? Every owned preview, skip, config knob, and lib fork should trace to a bullet, and the Re-sync risks section should be current (§4d). Write whatever's missing now - it costs a minute today and a re-derivation later.

   2. 以下一个代理的身份重读 NOTES.md，不带本会话的任何记忆：仅凭写下的内容，你能跳过今天的调试吗？每个自有预览、skip、配置旋钮与 lib fork 都应能追溯到一条要点，Re-sync risks 小节应是最新的（§4d）。现在就补写缺的东西——今天花一分钟，省掉日后的重新推导。

3. After a re-sync - however much it changed or re-graded - leave NOTES.md and the git state exactly as you found them unless the run produced something the next run needs to know; only hand the user something to commit when it adds value for a future sync.

   3. 重新同步之后——无论它改动或重评了多少——把 NOTES.md 与 git 状态保持为你发现时的原样，除非本次运行产出了下一次运行需要知道的东西；只有当它为未来同步增加价值时，才把可提交的东西交给用户。

## 7. Re-syncs - one command routes the work / 重新同步——一条命令调度全部工作

The repo carries the sync's inputs (config, owned previews, NOTES.md); the uploaded project carries the anchor (`_ds_sync.json`). Read NOTES.md first (Re-sync risks is the watch-list), then:

仓库承载同步的输入（配置、自有预览、NOTES.md）；已上传项目承载锚点（`_ds_sync.json`）。先读 NOTES.md（Re-sync risks 是观察清单），然后：

1. **Refresh inputs.** Re-copy the staged scripts (§2.4's `cp -r` line - instant; a stale `.ds-sync/` runs an old converter against these instructions). Re-run `buildCmd` **and rebuild `.design-sync/sb-reference`** whenever the DS source may have changed - they must move together; when in doubt rebuild both (deterministic builds make an unnecessary rebuild a no-op; `[REFERENCE_STALE?]` in the capture log means you forgot). Fresh-clone extras: the §2.4 dep install + chromium, the §2.2 sb-reference build, and - if the repo carries `.design-sync/overrides/` forks with bare imports - `ln -sfn ../.ds-sync/node_modules .design-sync/node_modules`.

   1. **刷新输入。**重新复制暂存脚本（§2.4 的 `cp -r` 行——瞬间完成；陈旧的 `.ds-sync/` 会用旧转换器跑这些指令）。只要 DS 源码可能变了，就重跑 `buildCmd` **并重建 `.design-sync/sb-reference`**——它们必须一起移动；拿不准就两个都重建（确定性构建让一次多余的重建等于无操作；捕获日志中的 `[REFERENCE_STALE?]` 意味着你忘了）。新克隆附加项：§2.4 的依赖安装 + chromium、§2.2 的 sb-reference 构建，以及——若仓库带有含裸导入的 `.design-sync/overrides/` fork——`ln -sfn ../.ds-sync/node_modules .design-sync/node_modules`。

2. **Fetch the anchor**: `DesignSync(get_file, path: "_ds_sync.json")` -> save to `.design-sync/.cache/remote-sync.json`. No sidecar in the project -> first-sync scope (omit `--remote` below).

   2. **获取锚点**：`DesignSync(get_file, path: "_ds_sync.json")` -> 保存到 `.design-sync/.cache/remote-sync.json`。项目中没有 sidecar -> 按首次同步范围（省略下方的 `--remote`）。

3. **Run the driver** from the repo root:

   3. **运行 driver**（从仓库根）：

   ```sh
   node .ds-sync/resync.mjs --config .design-sync/config.json --node-modules <nm> \
     [--entry <dist-entry>] --out ./ds-bundle --remote .design-sync/.cache/remote-sync.json
   ```

   It chains build -> diff -> validate -> capture (scoped to new + contract-changed components) and prints one verdict JSON (also written to `ds-bundle/.resync-verdict.json`). Stage logs stream to stderr. The driver is idempotent - re-run it after fixes. For per-component preview iteration use the §4a targeted loop instead (seconds, not a full build + render-check); the driver re-run is the closing receipt.

   它串联 build -> diff -> validate -> capture（限定于新增 + 契约变更组件），打印一个判定 JSON（也写入 `ds-bundle/.resync-verdict.json`）。阶段日志流向 stderr。driver 是幂等的——修复后重跑。逐组件的预览迭代改用 §4a 定向循环（秒级，不是全量构建 + 渲染检查）；driver 重跑才是收尾回执。

   The driver also scopes validate's render check by what the diff proved (explicit `--render-sample` / `--no-render-check` flags always win). With a healthy anchor and the bundle + styling unchanged, every unchanged preview's render inputs are byte-identical to what the last upload render-verified (or explicitly accepted) - the diff pins the anchor to the fresh sidecar, the `[SYNC_STALE]`/bundle-sha recompute pins the render surfaces to disk (styling is pinned by the build that just wrote both), and re-rendering identical bytes tests your chromium install, not the artifacts. So: nothing changed at all -> the render check is **skipped** (the `[RENDER_SKIPPED]` warn on that run is driver-announced and expected - not a new warn to chase); something still ships but nothing that affects rendering moved (docs/guidelines edits, an anchor refresh) -> **sampled** (`--render-sample 10`); anything that could change a render moved - components changed/added/churned, bundle or styling (a `.d.ts`/`.prompt.md` edit lands here: it re-ships the bundle, whose header embeds those files' hashes) - or no healthy anchor -> **full**, as always. The file-shape checks (`[SYNC_STALE]`, bundle header, CSS/fonts, `.d.ts` parse) run in full on every tier; pass `--render-sample 0` to force the full render pass.

   driver 还会按 diff 所证明的内容为 validate 的渲染检查划定范围（显式 `--render-sample` / `--no-render-check` 标志永远优先）。锚点健康且 bundle + 样式未变时，每个未变预览的渲染输入与上次上传渲染验证过（或显式接受过）的字节完全相同——diff 把锚点 pin 到新 sidecar，`[SYNC_STALE]`/bundle-sha 重算把渲染面 pin 到磁盘（样式由刚写出两者的构建 pin 定），而重新渲染相同字节测试的是你的 chromium 安装，不是制品。所以：完全没变 -> 渲染检查**跳过**（该次运行中的 `[RENDER_SKIPPED]` 告警是 driver 主动宣布且符合预期的——不是要追查的新告警）；仍有东西发布但没有影响渲染的变动（docs/guidelines 编辑、锚点刷新）-> **抽样**（`--render-sample 10`）；任何可能改变渲染的东西变了——组件变更/新增/扰动、bundle 或样式（`.d.ts`/`.prompt.md` 的编辑落在这里：它会重新发布 bundle，其头部内嵌这些文件的哈希）——或没有健康锚点 -> **全量**，一如既往。文件形态检查（`[SYNC_STALE]`、bundle 头部、CSS/字体、`.d.ts` 解析）在每一层都全量运行；传 `--render-sample 0` 可强制完整渲染通过。

4. **Act on the verdict** - every field that needs you:

   4. **按判定行动**——每个需要你处理的字段：

   | Field | Your work |
   |---|---|
   | `ok: false` | the failed stage (`stages.<name>`) logged its [TAG]s - fix per that stage's section above, re-run. Every stage green? Check `learningsUnmerged` |
   | `learningsUnmerged` non-empty | unfolded fan-out learnings - fold into NOTES.md, delete the files (§4c step 1), re-run; this alone fails `ok`, and the run preserves the reference-drift canary for the retry |
   | `verification.pendingGrade` | grade those fresh sheets (§4 rubric). In the capture log: `[STORY_CHANGED]` -> mirror the story in the owned `.tsx` first; `unpaired` -> add the export; `extraCells` naming an owned export -> prune it |
   | `verification.canary` | pipeline churn (or a reference-storybook change) with your sources stable - grades kept; confirm the named `[SPOT_CHECK]` sheets against the recorded grades. A couple diverge -> re-grade those components; widespread divergence -> `--force` full pass |
   | warn lines in the validate log (`[RENDER_THIN]` etc.) | check NOTES.md's known list - a warn recorded there was triaged on a prior sync (legitimately-short components read as thin forever); a warn NOT recorded there is new - look at that component, then fix it or record it in NOTES.md |
   | `verification.removed` | components gone upstream - confirm the deletions are intentional |
   | `upload.styling: true` | styling re-ships automatically; grades stay |
   | `upload.any: false` | nothing to upload from THIS verdict - continue to step 5; you're done only after it (a header authored there re-runs the driver) |
   | `upload.any: true` | §6 upload - full writes by default, `deletes` verbatim from `upload.deletePaths` (never scope writes by the verification partition) |

   | 字段 | 你的工作 |
   |---|---|
   | `ok: false` | 失败阶段（`stages.<name>`）记录了它的 [TAG]——按上方对应阶段小节修复后重跑。所有阶段都绿？检查 `learningsUnmerged` |
   | `learningsUnmerged` 非空 | 未折叠的扇出经验——折叠进 NOTES.md、删除文件（§4c 第 1 步）、重跑；仅此一项即可判 `ok` 失败，且本次运行会为重试保留参考漂移金丝雀 |
   | `verification.pendingGrade` | 给那些新对照表评分（§4 标准）。捕获日志中：`[STORY_CHANGED]` -> 先在自有 `.tsx` 中镜像该 story；`unpaired` -> 添加导出；`extraCells` 点名某个自有导出 -> 修剪它 |
   | `verification.canary` | 流水线扰动（或参考 storybook 变更）而你的源稳定——评分保留；对照已记录评分确认被点名的 `[SPOT_CHECK]` 对照表。个别分歧 -> 重评那些组件；大范围分歧 -> `--force` 全量通过 |
   | validate 日志中的 warn 行（`[RENDER_THIN]` 等） | 查 NOTES.md 的已知清单——已记录在案的 warn 在此前同步中已分诊过（合法偏短的组件会永远被读作 thin）；未记录的 warn 是新的——看那个组件，然后修复它或记录进 NOTES.md |
   | `verification.removed` | 组件在上游消失——确认删除是有意的 |
   | `upload.styling: true` | 样式自动重新发布；评分保持 |
   | `upload.any: false` | 本次判定无可上传——继续第 5 步；只有做完它才算完成（在那里撰写的头部会重跑 driver） |
   | `upload.any: true` | §6 上传——默认全量写入，`deletes` 逐字取自 `upload.deletePaths`（绝不要按验证分区缩小写入范围） |

   Grades follow your sources by design - DS source, CSS, and bundle changes carry, and pipeline churn arrives as `verification.canary` rather than re-grades. To deliberately audit carried-forward grades anyway (after a major DS version bump, or on suspicion), run `node .ds-sync/storybook/compare.mjs --out ./ds-bundle --components <A,B> --spot-check-components <A,B>` - fresh sheets, grades kept - and confirm the sheets still match the recorded grades.

   评分按设计跟随你的源——DS 源码、CSS 与 bundle 的变更被携带，流水线扰动以 `verification.canary` 的形式到来而非重评。若仍想有意审计已携带的评分（在 DS 大版本升级后，或有疑心时），运行 `node .ds-sync/storybook/compare.mjs --out ./ds-bundle --components <A,B> --spot-check-components <A,B>`——新对照表、评分保留——并确认对照表仍与已记录评分相符。

5. **Run the conventions-header step** (base SKILL.md "Author the conventions header") - after acting on the verdict, before any upload, and regardless of what the verdict said. On a re-sync it validates an existing `.design-sync/conventions.md` against the fresh build and reports drift; for repos synced before the step existed it authors the file for the first time. If it authored or changed the header, rebuild per the base step's **rebuild rule** (driver run here) and act on the fresh verdict - the prior verdict predates the header.

   5. **运行约定头部步骤**（基础 SKILL.md "Author the conventions header"）——在按判定行动之后、任何上传之前，且无论判定说了什么。重新同步时它以新构建验证既有 `.design-sync/conventions.md` 并报告漂移；对在该步骤存在之前同步过的仓库则首次撰写该文件。若它撰写或修改了头部，按基础步骤的**重建规则**重建（此处为 driver 运行）并按新判定行动——先前判定早于头部。

6. Re-fetch the sidecar right before `finalize_plan`; if it moved (concurrent sync), re-run the driver and act on the fresh verdict.

   6. 在 `finalize_plan` 前一刻重新拉取 sidecar；若它移动了（并发同步），重跑 driver 并按新判定行动。
