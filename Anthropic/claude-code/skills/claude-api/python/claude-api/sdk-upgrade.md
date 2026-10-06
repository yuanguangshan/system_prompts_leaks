<!-- BILINGUAL-EN-ZH -->
# Upgrading the `anthropic` Python SDK: 0.x -> 1.x / 将 `anthropic` Python SDK 从 0.x 升级到 1.x

> **If you arrived via `/claude-api upgrade`:** this is the right file. Execute the steps below in order - do not summarize them back to the user. Start with Step 0 before touching any file.

> **如果你是通过 `/claude-api upgrade` 来到这里的：**这个文件就是正确入口。按顺序执行以下步骤——不要把步骤复述总结给用户。在改动任何文件之前，先从第 0 步开始。

`anthropic` 1.x is deliberately a small step from the last 0.x release: no method was restructured and no new pattern is required. Long-deprecated surface was removed, the HTTP layer moved from `httpx` to its maintained fork `httpx2`, and the minimum Python version is now 3.10. Almost every required edit is mechanical, and a type checker flags nearly all of them once 1.x is installed - which makes `pyright` / `mypy` output a good cross-check for the inventory below.

`anthropic` 1.x 相对最后一个 0.x 版本刻意只是一小步：没有任何方法被重构，也不要求任何新模式。移除的是早已弃用的表面 API，HTTP 层从 `httpx` 迁移到其持续维护的分支 `httpx2`，最低 Python 版本现在是 3.10。几乎所有必需的修改都是机械性的，且安装 1.x 后类型检查器几乎能把它们全部标出——这使得 `pyright` / `mypy` 的输出成为下文清单的良好交叉校验。

The SDK repository's `MIGRATION.md` is the authoritative change list - WebFetch it (URL in `shared/live-sources.md` -> SDK major-version upgrade guides) when you can, and if it disagrees with this file, follow `MIGRATION.md` and say so in your report. The other Python files in this skill may still show 0.x-era details; for a project on 1.x, this file takes precedence.

SDK 仓库的 `MIGRATION.md` 是权威的变更清单——可能时用 WebFetch 获取它（URL 见 `shared/live-sources.md` -> SDK major-version upgrade guides）；若它与本文件不一致，以 `MIGRATION.md` 为准并在报告中说明。本技能的其他 Python 文件可能仍显示 0.x 时代的细节；对于已升到 1.x 的项目，以本文件为准。


---

## Step 0: Confirm scope, current version, and target / 第 0 步：确认范围、当前版本与目标

**Scope - ask before editing unless it is already unambiguous.** Same rule as model migration: if the request does not name an exact file, a specific directory, or an explicit file list, ask one question offering (1) the whole working directory, (2) a specific subdirectory, (3) specific files - and wait. `upgrade`, `upgrade python`, "move my project to anthropic v1" are all scope-ambiguous. A trailing path in the subcommand (`upgrade python src/`) is a scope. Dependency manifests and lockfiles at the project root (`pyproject.toml`, `requirements*.txt`, `setup.py`/`setup.cfg`, `Pipfile`, `uv.lock`, `poetry.lock`) count as in scope whenever any code under them is - say so when you confirm the scope.

**范围——除非已经毫无歧义，否则先问再改。**与模型迁移相同的规则：如果请求没有指明确切的文件、特定的目录或明确的文件列表，就提一个问题，给出选项（1）整个工作目录、（2）某个特定子目录、（3）特定文件——然后等待。`upgrade`、`upgrade python`、"move my project to anthropic v1" 都属于范围不明。子命令末尾的路径（`upgrade python src/`）即是范围。项目根目录的依赖清单与锁文件（`pyproject.toml`、`requirements*.txt`、`setup.py`/`setup.cfg`、`Pipfile`、`uv.lock`、`poetry.lock`）只要其下有任何代码在范围内，就同样算在范围内——确认范围时说明这一点。

**Current version.** Read the declared requirement (`anthropic...` in the manifests above) and, if a project environment is available, the installed one (`python -c "import anthropic; print(anthropic.__version__)"`). If the project is already on 1.x, skip the dependency bump and treat this as a call-site cleanup. If nothing in scope declares the dependency (a bare scripts directory, or `anthropic` arrives transitively), don't invent a manifest - upgrade the code and put the install command in the report.

**当前版本。**读取声明的依赖要求（上文清单中的 `anthropic...`），若项目环境可用，再读已安装的版本（`python -c "import anthropic; print(anthropic.__version__)"`）。如果项目已经在 1.x 上，跳过依赖升级，把这次当作调用点清理来处理。如果范围内没有任何地方声明该依赖（一个纯粹的脚本目录，或 `anthropic` 是传递依赖），不要凭空造一个清单——直接升级代码，并把安装命令写进报告。

**Target version.** Before writing any pin, confirm a 1.x release is actually published: `pip index versions anthropic` (or `curl -s https://pypi.org/pypi/anthropic/json` and read `info.version`). Use the newest 1.x you find. If no 1.x release exists yet, stop and tell the user - do not write an uninstallable requirement. If you cannot check (no network), proceed with `>=1,<2` and list the unverified pin in your report.

**目标版本。**写入任何版本钉之前，先确认 1.x 发行版确实已发布：`pip index versions anthropic`（或 `curl -s https://pypi.org/pypi/anthropic/json` 并读取 `info.version`）。用你找到的最新 1.x。若 1.x 尚无任何发行版，停下来告诉用户——不要写一个装不上的依赖要求。若无法检查（无网络），按 `>=1,<2` 继续，并在报告中列出这个未经验证的版本钉。

If the scope is under git, check `git status` before editing - unexpected modifications mean a concurrent process; stop and investigate before proceeding.

如果范围在 git 管理之下，编辑前先检查 `git status`——意外的改动意味着有并发进程在动它；先停下来查清再继续。

## Step 1: Inventory the call sites / 第 1 步：清点调用点

Search the scope for each signal below (`rg -n -F` for the literal strings; exclude virtualenvs, `.git`, build output and vendored code) and keep the hit list - it is your checklist and, re-run at the end, your verification.

在范围内搜索下表中的每个信号（字面字符串用 `rg -n -F`；排除虚拟环境、`.git`、构建输出与内嵌的第三方代码），并保留命中清单——它是你的核对清单，结束时重跑一遍即是你的验证。

| Signal | What it finds | Section |
|---|---|---|
| `requires-python`, `python_requires`, `python-version`, `py39`, `3.9` in manifests, CI config, `tox.ini`, `noxfile.py`, `.python-version`, `Dockerfile` | a Python 3.9 floor | Step 2 |
| `anthropic` entries in manifests / lockfiles; `httpx-aiohttp`, `httpx_aiohttp` | the pins to change | Step 2 |
| `import httpx`, `from httpx` | modules that may hand `httpx` objects to the SDK | Step 3 |
| `respx`, `pytest_httpx` / `httpx_mock`, `vcr`, `MockTransport`; `HTTPXClientInstrumentor` / `opentelemetry.instrumentation.httpx`, `HttpxIntegration` (Sentry) | HTTP mocking and tracing / APM instrumentation that patch `httpx` and silently stop seeing SDK traffic | Step 3 |
| `with_raw_response` | raw-response call sites | Step 4 |
| `LegacyAPIResponse`, `_legacy_response` | annotations / imports of the removed class | Step 4 |
| `completions.create`, `HUMAN_PROMPT`, `AI_PROMPT`, `max_tokens_to_sample` | the removed Text Completions API | Step 5 |
| `temperature`, `top_p`, `top_k` (keyword arguments and quoted dict keys) | removed sampling parameters - only hits that feed Anthropic SDK calls count | Step 6 |
| `output_format` | raw `output_format={...}` dicts vs the unchanged `output_format=Model` helper argument | Step 6 |
| `BetaBase64PDFBlockParam`, `READ_MAX_BYTES`, `ProxiesTypes` / `Transport` imported from `anthropic`, `AsyncTransport` / `ProxiesDict` imported from `anthropic._types` | renamed / removed exports | Step 7 |
| `.parse(` calls that pass `stream=` | `messages.parse(stream=...)` | Step 8 |
| `compaction_control` | client-side tool-runner compaction | Step 8 |
| `body=` on `client.get` / `post` / `put` / `patch` / `delete` calls whose value is `bytes` (`b"..."`, `.encode()`, a bytes variable) | raw bytes passed as `body=` | Step 8 |
| `isinstance(` checks against `Stream` / `AsyncStream` | checks aimed at message streams | Step 8 |
| `default_headers`, `extra_headers`, `ANTHROPIC_CUSTOM_HEADERS` | header maps to check for duplicate casings / `bytes` values | Step 9 |
| `AnthropicBedrock(`, `AsyncAnthropicBedrock(` | Bedrock clients that may rely on the old region fallback | Step 10 |

| 信号 | 找到的内容 | 对应章节 |
|---|---|---|
| 清单、CI 配置、`tox.ini`、`noxfile.py`、`.python-version`、`Dockerfile` 中的 `requires-python`、`python_requires`、`python-version`、`py39`、`3.9` | Python 3.9 下限 | 第 2 步 |
| 清单 / 锁文件中的 `anthropic` 条目；`httpx-aiohttp`、`httpx_aiohttp` | 需要修改的版本钉 | 第 2 步 |
| `import httpx`、`from httpx` | 可能把 `httpx` 对象交给 SDK 的模块 | 第 3 步 |
| `respx`、`pytest_httpx` / `httpx_mock`、`vcr`、`MockTransport`；`HTTPXClientInstrumentor` / `opentelemetry.instrumentation.httpx`、`HttpxIntegration`（Sentry） | 会给 `httpx` 打补丁、从而悄然再也看不到 SDK 流量的 HTTP 模拟与追踪 / APM 埋点 | 第 3 步 |
| `with_raw_response` | 原始响应调用点 | 第 4 步 |
| `LegacyAPIResponse`、`_legacy_response` | 对被移除类的注解 / 导入 | 第 4 步 |
| `completions.create`、`HUMAN_PROMPT`、`AI_PROMPT`、`max_tokens_to_sample` | 被移除的文本补全（Text Completions）API | 第 5 步 |
| `temperature`、`top_p`、`top_k`（关键字参数与带引号的字典键） | 被移除的采样参数——只统计真正输入 Anthropic SDK 调用的命中 | 第 6 步 |
| `output_format` | 区分原始 `output_format={...}` 字典与未变的 `output_format=Model` 辅助参数 | 第 6 步 |
| `BetaBase64PDFBlockParam`、`READ_MAX_BYTES`、从 `anthropic` 导入的 `ProxiesTypes` / `Transport`、从 `anthropic._types` 导入的 `AsyncTransport` / `ProxiesDict` | 被改名 / 移除的导出 | 第 7 步 |
| 传了 `stream=` 的 `.parse(` 调用 | `messages.parse(stream=...)` | 第 8 步 |
| `compaction_control` | 客户端 tool-runner 压缩 | 第 8 步 |
| `client.get` / `post` / `put` / `patch` / `delete` 调用上值为 `bytes`（`b"..."`、`.encode()`、bytes 变量）的 `body=` | 以 `body=` 传入的原始字节 | 第 8 步 |
| 针对 `Stream` / `AsyncStream` 的 `isinstance(` 检查 | 面向消息流的检查 | 第 8 步 |
| `default_headers`、`extra_headers`、`ANTHROPIC_CUSTOM_HEADERS` | 需检查重复大小写 / `bytes` 值的头部映射 | 第 9 步 |
| `AnthropicBedrock(`、`AsyncAnthropicBedrock(` | 可能依赖旧区域回退的 Bedrock 客户端 | 第 10 步 |

Classify each hit before editing: **SDK call site** (edit), **unrelated use of the same name** (leave - e.g. `httpx` calls to other services, `urllib.parse`, a pydantic `.parse_obj`, a `temperature` variable for a thermostat), **test** (edit, and keep the test meaningful), **docs / README snippet or notebook inside the scope** (edit - for `.ipynb`, the greps match inside the JSON cell sources; edit the source strings, `%pip install` lines included, and keep the JSON valid). Never touch installed packages or vendored third-party code.

编辑前先给每个命中分类：**SDK 调用点**（修改）、**同名但无关的使用**（不动——例如对其他服务的 `httpx` 调用、`urllib.parse`、pydantic 的 `.parse_obj`、恒温器的 `temperature` 变量）、**测试**（修改，并保持测试仍有意义）、**范围内的文档 / README 片段或 notebook**（修改——对 `.ipynb`，grep 会命中 JSON 单元格源码内的内容；修改源字符串，包括 `%pip install` 行，并保持 JSON 有效）。绝不碰已安装的包或内嵌的第三方代码。

## Step 2: Environment - Python >= 3.10 and the dependency pins / 第 2 步：环境——Python >= 3.10 与依赖钉

- **[DECIDE] Python floor.** 1.x requires Python 3.10+. If the project still declares or tests 3.9 (`requires-python = ">=3.9"`, trove classifiers, a `3.9` CI matrix entry, tox/nox envs, a `python:3.9` base image), that is the user's decision, not a silent edit: propose the floor bump and the CI-matrix change as their own hunk and call it out in the report. On 3.9, `pip` simply keeps resolving the last 0.x release, so nothing breaks until they move.
  **[需决策] Python 下限。**1.x 要求 Python 3.10+。如果项目仍声明或测试 3.9（`requires-python = ">=3.9"`、trove 分类器、CI 矩阵中的 `3.9` 条目、tox/nox 环境、`python:3.9` 基础镜像），那是用户的决定，不是可以悄悄改的：把下限提升与 CI 矩阵修改作为独立的补丁块提出，并在报告中点明。在 3.9 上，`pip` 只会继续解析到最后一个 0.x 版本，因此在用户行动之前什么都不会坏。
- **[BREAKS] The `anthropic` requirement.** Rewrite it in the file's existing style - `anthropic>=1,<2` for a range, `anthropic~=1.0` / Poetry `^1.0` for compatible-release styles, `anthropic==<latest 1.x from Step 0>` where the project pins exactly. Extras (`anthropic[bedrock]`, `[vertex]`, `[aiohttp]`) are unchanged. Regenerate the lockfile with the project's own tool (`uv lock`, `poetry lock`, `pip-compile`, `pipenv lock`) if you can run it; otherwise give the user the exact command.
  **[破坏性] `anthropic` 依赖要求。**按文件既有的风格重写——范围式用 `anthropic>=1,<2`，兼容发布式用 `anthropic~=1.0` / Poetry 的 `^1.0`，项目精确钉版时用 `anthropic==<Step 0 得到的最新 1.x>`。Extras（`anthropic[bedrock]`、`[vertex]`、`[aiohttp]`）不变。能运行时用项目自己的工具重新生成锁文件（`uv lock`、`poetry lock`、`pip-compile`、`pipenv lock`）；否则把确切命令交给用户。
- **`httpx-aiohttp`.** If it is pinned only so `DefaultAioHttpClient()` works, remove it - the aiohttp transport now ships inside the SDK and the `aiohttp` extra installs only `aiohttp`.
  **`httpx-aiohttp`。**如果钉它只是为了让 `DefaultAioHttpClient()` 能用，就移除它——aiohttp 传输现已内置在 SDK 中，`aiohttp` extra 只安装 `aiohttp`。
- **`httpx2` / `httpx`.** After Step 3, if any project module imports `httpx2` directly, add `httpx2` to the declared dependencies (it arrives transitively with `anthropic`, but direct imports should be declared). `httpx2` has its own version line starting at 2.0 - write `httpx2>=2.0` (or match what `anthropic` resolved: `pip index versions httpx2`), never a specifier copied from the old `httpx` pin such as `>=0.27`. Keep `httpx` declared only if the project still uses it for something other than the SDK.
  **`httpx2` / `httpx`。**第 3 步之后，如果项目中有模块直接导入 `httpx2`，就把 `httpx2` 加进声明的依赖（它随 `anthropic` 传递而来，但直接导入应当显式声明）。`httpx2` 有自己的版本线，从 2.0 开始——写 `httpx2>=2.0`（或与 `anthropic` 的解析结果一致：`pip index versions httpx2`），绝不要照抄旧 `httpx` 钉里的限定符（如 `>=0.27`）。只有当项目还在为 SDK 之外的目的使用 `httpx` 时才保留它的声明。

Pydantic v1 and v2 both remain supported; nothing else about the environment changes.

Pydantic v1 与 v2 都仍受支持；环境方面没有其他变化。

## Step 3: `httpx` -> `httpx2`, only where objects cross the SDK boundary / 第 3 步：`httpx` -> `httpx2`，仅当对象跨越 SDK 边界时

`httpx2` is the API-compatible, maintained fork of `httpx` (same classes, same behaviour). The change only matters for `httpx` objects handed **to** the SDK or received **from** it; plain values (`timeout=30.0`, `max_retries=3`) need nothing.

`httpx2` 是 `httpx` 的 API 兼容、持续维护的分支（相同的类、相同的行为）。这一变更只影响交给 **SDK** 或从 SDK 收到的 `httpx` 对象；普通值（`timeout=30.0`、`max_retries=3`）无需任何处理。

- **[BREAKS] Objects passed in.** `httpx.Timeout`, `httpx.Limits`, transports (`httpx.HTTPTransport(...)`, `AsyncHTTPTransport`, `MockTransport`), and whole clients (`httpx.Client` / `AsyncClient` as `http_client=`) must come from `httpx2`. An old-`httpx` client passed as `http_client=` raises `TypeError` at construction. This includes the project's own middleware, not just the outermost object handed to `Anthropic(...)`: a `class TracingTransport(httpx.BaseTransport)` subclass, the inner `httpx.HTTPTransport()` a wrapper delegates to, an `httpx.Auth` flow, and the annotations on `event_hooks` callables all re-base onto `httpx2` - a wrapper left delegating to an old-`httpx` transport hands the SDK `httpx.Response` objects. If the module uses `httpx` only for the SDK, alias the import (`import httpx2 as httpx`) and nothing else changes; if it also talks to other services with `httpx`, import both and switch only the SDK-bound objects to `httpx2`. Prefer the SDK's own re-exports where they let you drop the import entirely: `anthropic.Timeout`, `anthropic.DefaultHttpxClient`, `anthropic.DefaultAsyncHttpxClient`, `anthropic.DefaultAioHttpClient` (all already `httpx2`-based, all unchanged).
  **[破坏性] 传入的对象。**`httpx.Timeout`、`httpx.Limits`、传输器（`httpx.HTTPTransport(...)`、`AsyncHTTPTransport`、`MockTransport`）以及整个客户端（作为 `http_client=` 的 `httpx.Client` / `AsyncClient`）都必须来自 `httpx2`。把旧 `httpx` 客户端作为 `http_client=` 传入会在构造时抛出 `TypeError`。这包括项目自己的中间件，而不只是交给 `Anthropic(...)` 的最外层对象：`class TracingTransport(httpx.BaseTransport)` 子类、包装器所委托的内部 `httpx.HTTPTransport()`、`httpx.Auth` 流程，以及 `event_hooks` 可调用对象上的注解，全部要改基到 `httpx2`——一个仍在委托旧 `httpx` 传输器的包装器会把 `httpx.Response` 对象交给 SDK。如果该模块使用 `httpx` 仅为 SDK，就对导入起别名（`import httpx2 as httpx`），其余无需改动；如果它还用 `httpx` 与其他服务通信，就两者都导入，只把绑定 SDK 的对象换成 `httpx2`。在能彻底省掉导入之处，优先用 SDK 自己的再导出：`anthropic.Timeout`、`anthropic.DefaultHttpxClient`、`anthropic.DefaultAsyncHttpxClient`、`anthropic.DefaultAioHttpClient`（全部已基于 `httpx2`，全部未变）。

  ```python
  # Before
  import httpx
  from anthropic import Anthropic, DefaultHttpxClient

  client = Anthropic(
      timeout=httpx.Timeout(60.0, connect=5.0),
      http_client=DefaultHttpxClient(proxy="http://proxy.example", transport=httpx.HTTPTransport(retries=1)),
  )

  # After
  import httpx2 as httpx
  from anthropic import Anthropic, DefaultHttpxClient

  client = Anthropic(
      timeout=httpx.Timeout(60.0, connect=5.0),
      http_client=DefaultHttpxClient(proxy="http://proxy.example", transport=httpx.HTTPTransport(retries=1)),
  )
  ```

- **[DECIDE] Or alias process-wide, for applications.** `httpx2.alias_httpx()` makes `import httpx` / `import httpcore` resolve to `httpx2` / `httpcore2` for the whole process, so nothing else needs editing. Reach for it instead of the import edits when the scope is an **application** that shares clients, transports or exception types between the SDK and other `httpx` code, or that relies on tooling which patches `httpx` itself (tracing / APM instrumentation, HTTP mocking - see **Instrumentation and tests** below). Two hard rules: it must run before anything imports `httpx` or `httpcore` (otherwise it raises `RuntimeError`; calling it twice is a no-op), so it goes at the very top of the entry point; and it is for applications only - never add it to a **library's** import path on behalf of that library's users (edit the imports there instead). Say which you chose and why in the report.
  **[需决策] 或对整个进程起别名——面向应用。**`httpx2.alias_httpx()` 会让整个进程内 `import httpx` / `import httpcore` 解析到 `httpx2` / `httpcore2`，其余代码无需编辑。当范围是一个在 SDK 与其他 `httpx` 代码之间共享客户端、传输器或异常类型的**应用**，或依赖给 `httpx` 本身打补丁的工具（追踪 / APM 埋点、HTTP 模拟——见下文**埋点与测试**）时，用它替代逐个改导入。两条硬规则：它必须先于任何导入 `httpx` 或 `httpcore` 的代码运行（否则抛出 `RuntimeError`；调用两次是无操作），所以放在入口点的最顶端；且它只用于应用——绝不要替某个**库**的用户把它加进库的导入路径（在库中改为直接编辑导入）。在报告中说明你选了哪种方式及原因。

  ```python
  # the very first lines of the application's entry point
  import httpx2

  httpx2.alias_httpx()

  import httpx  # now the httpx2 module: httpx.Client is httpx2.Client
  ```

- **[BREAKS] Objects coming out.** `APIStatusError.response`, `APIConnectionError.request`, `.http_response` / `.headers` / `.url` on raw and streaming responses, the `request` / `response` arguments your `http_client` event hooks receive, and `cast_to=httpx.Response` on the low-level `client.get/post/...` methods are now `httpx2` types with identical attributes. Only `isinstance` checks and annotations naming `httpx.Response` / `httpx.Request` / `httpx.Headers` / `httpx.URL` change (`httpx2.Response`, ...).
  **[破坏性] 传出的对象。**`APIStatusError.response`、`APIConnectionError.request`、原始与流式响应上的 `.http_response` / `.headers` / `.url`、你的 `http_client` 事件钩子收到的 `request` / `response` 参数，以及低层 `client.get/post/...` 方法上的 `cast_to=httpx.Response`，现在都是属性完全相同的 `httpx2` 类型。只有点名 `httpx.Response` / `httpx.Request` / `httpx.Headers` / `httpx.URL` 的 `isinstance` 检查与注解需要改（`httpx2.Response`，依此类推）。
- **Removed re-exports.** `anthropic.Transport` and `anthropic.ProxiesTypes` (and `AsyncTransport` / `ProxiesDict` from `anthropic._types`) are gone; use `httpx2.BaseTransport`, `httpx2.AsyncBaseTransport`, `httpx2.Proxy` (or a proxy URL string).
  **移除的再导出。**`anthropic.Transport` 与 `anthropic.ProxiesTypes`（以及 `anthropic._types` 的 `AsyncTransport` / `ProxiesDict`）已不存在；改用 `httpx2.BaseTransport`、`httpx2.AsyncBaseTransport`、`httpx2.Proxy`（或代理 URL 字符串）。
- **Instrumentation and tests.** Libraries that observe or stub HTTP by patching `httpx` - OpenTelemetry's `HTTPXClientInstrumentor`, Sentry's `httpx` integration, `respx`, `pytest-httpx`, `vcrpy` - keep importing fine but silently stop seeing the SDK's requests, so nothing fails loudly. The fix is the same `httpx2.alias_httpx()` call - not swapping in some `*-httpx2` instrumentation package (verify any such name is a real, populated release before depending on it) - made before any of them (or `httpx`) is imported: at the top of the application entry point for instrumentation, and under pytest as an early plugin so it runs before `respx` / `pytest-httpx` and the test modules load:
  **埋点与测试。**通过给 `httpx` 打补丁来观测或拦截 HTTP 的库——OpenTelemetry 的 `HTTPXClientInstrumentor`、Sentry 的 `httpx` 集成、`respx`、`pytest-httpx`、`vcrpy`——导入仍正常，但会悄然再也看不到 SDK 的请求，没有任何东西会大声报错。解决办法是同一个 `httpx2.alias_httpx()` 调用——而不是换装某个 `*-httpx2` 埋点包（依赖任何此类名称前，先核实它是一个真实、有实际内容的发行版）——并在它们（或 `httpx`）被导入之前完成：埋点放在应用入口点顶端，pytest 下则作为早期插件，让它先于 `respx` / `pytest-httpx` 与测试模块加载：

  ```python
  # tests/_alias_httpx.py
  import httpx2

  httpx2.alias_httpx()  # `import httpx` / `import httpcore` now resolve to httpx2 / httpcore2
  ```

  ```toml
  # pyproject.toml
  [tool.pytest.ini_options]
  addopts = "-p tests._alias_httpx"
  pythonpath = ["."]
  ```

  Merge into an existing `addopts` rather than replacing it (`pytest.ini` / `setup.cfg` / `tox.ini` equivalents work the same way). Transport-level fakes (`httpx2.Client(transport=httpx2.MockTransport(handler))`, a handler typed `httpx2.Request -> httpx2.Response`) only need the import swap.
  合并进既有的 `addopts` 而不是替换它（`pytest.ini` / `setup.cfg` / `tox.ini` 中的等价写法同理）。传输层假件（`httpx2.Client(transport=httpx2.MockTransport(handler))`、类型为 `httpx2.Request -> httpx2.Response` 的 handler）只需换导入。

## Step 4: `.with_raw_response` returns `APIResponse` / `AsyncAPIResponse` / 第 4 步：`.with_raw_response` 返回 `APIResponse` / `AsyncAPIResponse`

`.with_raw_response` used to return `LegacyAPIResponse` on both clients; it now returns the same classes `.with_streaming_response` already used. Two consequences:

`.with_raw_response` 过去在两个客户端上都返回 `LegacyAPIResponse`；现在它返回 `.with_streaming_response` 早已使用的同一批类。两个后果：

- **[BREAKS] On async clients, reading the body is awaited** - `parse()`, `json()`, `text()`, `read()` are coroutines. Decide sync vs async from the client the accessor hangs off (`AsyncAnthropic` and the other `Async*` platform clients) or an `await` on the `.with_raw_response...(...)` call itself - not from the enclosing function alone.
  **[破坏性] 在异步客户端上，读取响应体需要 await**——`parse()`、`json()`、`text()`、`read()` 都是协程。以访问器所挂的客户端（`AsyncAnthropic` 及其他 `Async*` 平台客户端）或对 `.with_raw_response...(...)` 调用本身的 `await` 来判断同步还是异步——不要只看外层函数。
- **[BREAKS] `.text` and `.content` are methods now, on the sync client too:** `.text` -> `.text()`, `.content` -> `.read()`. The new classes also expose `json()` and the `iter_bytes()` / `iter_text()` / `iter_lines()` iterators directly; 0.x code reached those through `r.http_response`, which still works and need not be rewritten.
  **[破坏性] `.text` 与 `.content` 现在是方法，同步客户端也不例外：**`.text` -> `.text()`，`.content` -> `.read()`。新类还直接暴露 `json()` 与 `iter_bytes()` / `iter_text()` / `iter_lines()` 迭代器；0.x 代码通过 `r.http_response` 访问它们，这条路径仍然可用，无需重写。

| 0.x (`LegacyAPIResponse`) | 1.x sync (`APIResponse`) | 1.x async (`AsyncAPIResponse`) |
|---|---|---|
| `r.parse()` | `r.parse()` | `await r.parse()` |
| `r.text` | `r.text()` | `await r.text()` |
| `r.content` | `r.read()` | `await r.read()` |
| - (only `r.http_response.json()`) | `r.json()` | `await r.json()` |
| - (only `r.http_response.iter_bytes()` ...) | `r.iter_bytes()` / `.iter_text()` / `.iter_lines()` | `async for chunk in r.iter_bytes():` ... |
| `.headers`, `.status_code`, `.url`, `.request_id`, `.retries_taken`, `.http_response`, `.elapsed` | unchanged | unchanged (plain attributes - never awaited) |

| 0.x（`LegacyAPIResponse`） | 1.x 同步（`APIResponse`） | 1.x 异步（`AsyncAPIResponse`） |
|---|---|---|
| `r.parse()` | `r.parse()` | `await r.parse()` |
| `r.text` | `r.text()` | `await r.text()` |
| `r.content` | `r.read()` | `await r.read()` |
| -（仅 `r.http_response.json()`） | `r.json()` | `await r.json()` |
| -（仅 `r.http_response.iter_bytes()`……） | `r.iter_bytes()` / `.iter_text()` / `.iter_lines()` | `async for chunk in r.iter_bytes():`…… |
| `.headers`、`.status_code`、`.url`、`.request_id`、`.retries_taken`、`.http_response`、`.elapsed` | 未变 | 未变（普通属性——永不 await） |

```python
# Before (async client)
raw = await client.messages.with_raw_response.create(...)
print(raw.headers["request-id"], raw.text)
message = raw.parse()

# After
raw = await client.messages.with_raw_response.create(...)
print(raw.headers["request-id"], await raw.text())
message = await raw.parse()
```

Anchor every edit on a value that demonstrably comes from a `.with_raw_response.` call (follow it through variables, return values and fixtures); do not touch `.parse()` / `.text` on unrelated objects, and do not double-await. Annotations and imports of `anthropic._legacy_response.LegacyAPIResponse` become `anthropic.APIResponse` / `anthropic.AsyncAPIResponse`. `.with_streaming_response` code is unchanged.

每一处修改都要锚定在一个可证明来自 `.with_raw_response.` 调用的值上（沿变量、返回值与 fixture 追踪）；不要碰无关对象上的 `.parse()` / `.text`，也不要重复 await。`anthropic._legacy_response.LegacyAPIResponse` 的注解与导入改为 `anthropic.APIResponse` / `anthropic.AsyncAPIResponse`。`.with_streaming_response` 代码不变。

## Step 5: Text Completions -> Messages (the one non-mechanical change) / 第 5 步：文本补全 -> Messages（唯一非机械性的变更）

**[BREAKS]** `client.completions.create()` (`/v1/complete`), the `Completion` types, and the `anthropic.HUMAN_PROMPT` / `anthropic.AI_PROMPT` constants are removed (also from `AnthropicBedrock`). Port each call to `client.messages.create()`:

**[破坏性]** `client.completions.create()`（`/v1/complete`）、`Completion` 类型以及 `anthropic.HUMAN_PROMPT` / `anthropic.AI_PROMPT` 常量已被移除（`AnthropicBedrock` 上同样移除）。把每个调用移植到 `client.messages.create()`：

- the `f"{HUMAN_PROMPT} ...{AI_PROMPT}"` prompt string becomes `messages=[{"role": "user", "content": "..."}]`; text that preceded the first `HUMAN_PROMPT` as instructions becomes `system=`; alternating `HUMAN_PROMPT`/`AI_PROMPT` turns become alternating `user`/`assistant` messages;
  `f"{HUMAN_PROMPT} ...{AI_PROMPT}"` 提示词字符串变为 `messages=[{"role": "user", "content": "..."}]`；第一个 `HUMAN_PROMPT` 之前作为指令的文本变为 `system=`；交替的 `HUMAN_PROMPT`/`AI_PROMPT` 回合变为交替的 `user`/`assistant` 消息；
- `max_tokens_to_sample=` -> `max_tokens=`; `stop_sequences=` carries over; drop `temperature`/`top_p`/`top_k` (Step 6);
  `max_tokens_to_sample=` -> `max_tokens=`；`stop_sequences=` 原样沿用；去掉 `temperature`/`top_p`/`top_k`（第 6 步）；
- `completion.completion` -> the text blocks of `message.content` (`"".join(b.text for b in message.content if b.type == "text")`); `stop_reason` values carry over (`"stop_sequence"`, `"max_tokens"`), with `"end_turn"` as the new normal-completion value;
  `completion.completion` -> `message.content` 的文本块（`"".join(b.text for b in message.content if b.type == "text")`）；`stop_reason` 值原样沿用（`"stop_sequence"`、`"max_tokens"`），正常完成改用 `"end_turn"`；
- `stream=True` completions -> `client.messages.stream(...)` and its `text_stream`.
  `stream=True` 的补全 -> `client.messages.stream(...)` 及其 `text_stream`。

```python
# Before
from anthropic import AI_PROMPT, HUMAN_PROMPT

completion = client.completions.create(
    model="claude-2.1",
    max_tokens_to_sample=256,
    prompt=f"{HUMAN_PROMPT} Why is the sky blue?{AI_PROMPT}",
)
print(completion.completion)

# After
message = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=256,
    messages=[{"role": "user", "content": "Why is the sky blue?"}],
)
print("".join(block.text for block in message.content if block.type == "text"))
```

**[DECIDE] The model.** Code still on Text Completions usually pins a retired model (`claude-2.x`, `claude-instant-*`), which 404s regardless of SDK version. Keep a model that is still served; otherwise switch to `claude-opus-5-5` so the code runs, say so prominently in the report, and point the user at `/claude-api migrate` for validating prompts against the new model - a completions-era prompt is exactly what `shared/prompt-audit.md` exists for.

**[需决策] 模型。**仍停留在文本补全上的代码通常钉着一个已退役的模型（`claude-2.x`、`claude-instant-*`），无论 SDK 版本如何都会 404。保留一个仍在服务的模型；否则切换到 `claude-opus-5-5` 让代码能跑，并在报告中醒目说明，同时引导用户用 `/claude-api migrate` 针对新模型校验提示词——补全时代的提示词正是 `shared/prompt-audit.md` 的用武之地。

## Step 6: Removed request parameters / 第 6 步：被移除的请求参数

- **[BREAKS] `temperature`, `top_p`, `top_k`** are no longer accepted by `messages.create()` / `.stream()` / `.parse()`, their `beta.messages` counterparts, or `beta.messages.tool_runner()` (passing them is a `TypeError`), and are gone from the per-request `params` TypedDict of `messages.batches.create()` (a type checker flags the key; at runtime the SDK still forwards it). Delete them - they are gone from the 1.x signatures, not from the API, and whether a model still honours them is a model question (`shared/model-migration.md`): Opus 4.7 and later return a 400 for any request that carries one (the default value included), Claude Sonnet 5.5 and Claude Sonnet 5 reject non-default values, and every still-served model before those accepts them - the Claude 4.6 / 4.5 line (Opus 4.6, Sonnet 4.6, Opus 4.5, Sonnet 4.5, Haiku 4.5) and the deprecated-but-still-served Claude 4 models (`shared/models.md` -> Deprecated Models). So **[DECIDE]** when the call pins one of those accepting models and visibly depends on the setting (a documented determinism requirement, an A/B on temperature), move it into `extra_body` instead of deleting it - `extra_body={"temperature": 0.2}` is merged into the request JSON as-is - and for a `messages.batches.create()` request leave the key in that request's `params` dict (it is forwarded, see above). A call that pins a retired model (`shared/models.md` -> Retired Models) is the `migrate` flow's problem first: it needs a replacement model, and the replacement decides whether the setting survives. Say which calls kept a setting this way in the report. When a test existed only to assert that these parameters pass through, keep it meaningful by asserting on parameters that still exist (`stop_sequences`, `metadata`, `service_tier`, `max_tokens`) rather than deleting it.
  **[破坏性] `temperature`、`top_p`、`top_k`** 不再被 `messages.create()` / `.stream()` / `.parse()`、对应的 `beta.messages` 方法或 `beta.messages.tool_runner()` 接受（传入即 `TypeError`），并已从 `messages.batches.create()` 的每请求 `params` TypedDict 中移除（类型检查器会标出该键；运行时 SDK 仍会转发它）。删除它们——它们是从 1.x 的签名中消失，而不是从 API 中消失；模型是否仍遵循它们是模型层面的问题（`shared/model-migration.md`）：Opus 4.7 及以后对任何携带这些参数的请求（含默认值）返回 400，Claude Sonnet 5.5 与 Claude Sonnet 5 拒绝非默认值，而在此之前所有仍在服务的模型都接受它们——Claude 4.6 / 4.5 线（Opus 4.6、Sonnet 4.6、Opus 4.5、Sonnet 4.5、Haiku 4.5）以及已弃用但仍在服务的 Claude 4 模型（`shared/models.md` -> Deprecated Models）。因此**[需决策]**：当调用钉住的是上述接受这些参数的模型、且代码明显依赖该设置（有成文的确定性要求、基于 temperature 的 A/B 实验等）时，把它移进 `extra_body` 而不是删除——`extra_body={"temperature": 0.2}` 会原样合并进请求 JSON——而对 `messages.batches.create()` 请求则把键留在该请求的 `params` 字典里（会被转发，见上）。钉住已退役模型（`shared/models.md` -> Retired Models）的调用首先属于 `migrate` 流程要解决的问题：它需要替换模型，而替换后的模型决定该设置是否保留。在报告中说明哪些调用以这种方式保留了设置。如果某个测试存在的意义只是断言这些参数能透传，就改为断言仍然存在的参数（`stop_sequences`、`metadata`、`service_tier`、`max_tokens`）以保持其意义，而不是直接删掉。

【评论】本步骤区分了"从 SDK 签名移除"与"从 API 移除"，并提供 `extra_body` 作为显式逃生通道；这是签名层面清理与线上行为兼容之间的折中，把是否保留旧设置的判断权明确交还给用户。

  ```python
  # Before
  client.messages.create(..., model="claude-sonnet-4-6", temperature=0.2)

  # After (only when the pinned model accepts it and the code depends on it)
  client.messages.create(..., model="claude-sonnet-4-6", extra_body={"temperature": 0.2})
  ```

- **[BREAKS] `output_format={...}` as a raw dict/TypedDict** - on `beta.messages.create()`, `beta.messages.count_tokens()` and batch params (where the parameter is gone) and on the `messages.stream()` / `messages.count_tokens()` / `beta.messages.stream()` helpers (which used to accept a dict as well and now raise `TypeError` for one) -> `output_config={"format": {...}}` (merge into an existing `output_config` if one is already passed, e.g. alongside `effort`). **Leave `output_format=SomeModel` alone** when the value is a *type* (a Pydantic model / class passed to the `parse()`, `stream()` or `tool_runner()` helpers, or to the non-beta `messages.count_tokens()`) - that is the one form the helpers still take (`beta.messages.count_tokens()` only ever took the dict form, and has no `output_format` at all now). Tell them apart by the value: dict literal / `{"type": "json_schema", ...}` -> migrate; a class name -> keep.
  **[破坏性] 作为原始字典/TypedDict 的 `output_format={...}`**——在 `beta.messages.create()`、`beta.messages.count_tokens()` 与 batch 参数上（该参数已被移除），以及在 `messages.stream()` / `messages.count_tokens()` / `beta.messages.stream()` 辅助方法上（它们过去也接受字典，现在遇到字典会抛 `TypeError`）-> 改为 `output_config={"format": {...}}`（如果已经传了 `output_config`，就合并进去，例如与 `effort` 并列）。**当值是*类型*时，不要动 `output_format=SomeModel`**（指传给 `parse()`、`stream()` 或 `tool_runner()` 辅助方法、或非 beta `messages.count_tokens()` 的 Pydantic 模型/类）——这是辅助方法仍接受的唯一形式（`beta.messages.count_tokens()` 只接受过字典形式，现在根本没有 `output_format` 了）。按值区分两者：字典字面量 / `{"type": "json_schema", ...}` -> 迁移；类名 -> 保留。

  ```python
  # Before
  client.beta.messages.create(..., temperature=0.2, output_format={"type": "json_schema", "schema": Order.model_json_schema()})

  # After
  client.beta.messages.create(..., output_config={"format": {"type": "json_schema", "schema": Order.model_json_schema()}})
  # or, usually better: client.beta.messages.parse(..., output_format=Order)
  ```

## Step 7: Renamed and removed names (pure renames) / 第 7 步：改名与移除的名称（纯改名）

**[BREAKS]** Replace imports and every reference; the replacement types are identical.

**[破坏性]** 替换导入与所有引用；替换后的类型完全相同。

| Removed | Replacement |
|---|---|
| `anthropic.types.beta.BetaBase64PDFBlockParam` | `anthropic.types.beta.BetaRequestDocumentBlockParam` |
| `anthropic.Transport` / `anthropic.ProxiesTypes` (and `anthropic._types.AsyncTransport` / `ProxiesDict`) | `httpx2.BaseTransport` / `httpx2.Proxy` (`httpx2.AsyncBaseTransport`) |
| `anthropic.HUMAN_PROMPT` / `anthropic.AI_PROMPT` | none - Step 5 |
| `anthropic.lib.tools.agent_toolset.READ_MAX_BYTES` | `anthropic.lib.tools.agent_toolset.DEFAULT_MAX_FILE_BYTES` |

| 被移除 | 替代 |
|---|---|
| `anthropic.types.beta.BetaBase64PDFBlockParam` | `anthropic.types.beta.BetaRequestDocumentBlockParam` |
| `anthropic.Transport` / `anthropic.ProxiesTypes`（及 `anthropic._types.AsyncTransport` / `ProxiesDict`） | `httpx2.BaseTransport` / `httpx2.Proxy`（`httpx2.AsyncBaseTransport`） |
| `anthropic.HUMAN_PROMPT` / `anthropic.AI_PROMPT` | 无——见第 5 步 |
| `anthropic.lib.tools.agent_toolset.READ_MAX_BYTES` | `anthropic.lib.tools.agent_toolset.DEFAULT_MAX_FILE_BYTES` |

## Step 8: Removed helper arguments and behaviour / 第 8 步：被移除的辅助参数与行为

- **[BREAKS] `messages.parse(..., stream=True)`** (and `beta.messages.parse`): the argument is gone (it never streamed). Use the streaming helper, which supports the same structured-output types:
  **[破坏性] `messages.parse(..., stream=True)`**（及 `beta.messages.parse`）：该参数已移除（它从未真正流式过）。改用支持相同结构化输出类型的流式辅助方法：

  ```python
  # Before
  result = client.messages.parse(..., output_format=Order, stream=True)

  # After
  with client.messages.stream(..., output_format=Order) as stream:
      order = stream.get_final_message().parsed_output
  ```

  A `parse(..., stream=False)` just loses the argument.
  `parse(..., stream=False)` 只是失去该参数。
- **[BREAKS] `tool_runner(compaction_control=...)`** - client-side compaction is removed in favour of server-side compaction. Carry the old `context_token_threshold` over as the trigger value (the API minimum is 50,000; raise smaller values to that and mention it):
  **[破坏性] `tool_runner(compaction_control=...)`**——客户端压缩已移除，改为服务端压缩。把旧的 `context_token_threshold` 作为触发值沿用（API 最小值为 50,000；更小的值提升到该值并加以说明）：

  ```python
  # Before
  runner = client.beta.messages.tool_runner(..., compaction_control={"enabled": True, "context_token_threshold": 100_000})

  # After
  runner = client.beta.messages.tool_runner(
      ...,
      betas=["compact-2026-01-12"],
      context_management={"edits": [{"type": "compact_20260112", "trigger": {"type": "input_tokens", "value": 100_000}}]},
  )
  ```

  If the loop around the runner rebuilds `messages` itself, make sure it appends the full `message.content` (compaction blocks included) - see the Compaction section of `python/claude-api/README.md`.
  如果围绕 runner 的循环自行重建 `messages`，确保它追加完整的 `message.content`（含压缩块）——见 `python/claude-api/README.md` 的 Compaction 一节。
- **[BREAKS] Raw `bytes` as `body=`** on `client.get/post/put/patch/delete`: `body=` is always JSON-serialised now; raw payloads (and iterators, for streaming uploads) go through `content=`:
  **[破坏性] 把原始 `bytes` 作为 `body=`** 用在 `client.get/post/put/patch/delete` 上：`body=` 现在总是被 JSON 序列化；原始负载（以及用于流式上传的迭代器）改走 `content=`：

  ```python
  # Before
  client.post("/v1/example", body=b"raw payload", cast_to=httpx.Response)

  # After
  client.post("/v1/example", content=b"raw payload", cast_to=httpx2.Response)
  ```

- **[BREAKS] `isinstance(x, anthropic.Stream)` / `AsyncStream` meant to match `client.messages.stream()` objects** now returns `False` (the compatibility shim and its `DeprecationWarning` are gone). Check for `anthropic.lib.streaming.MessageStream` / `AsyncMessageStream` instead; keep `Stream` only where the value really is a raw `create(stream=True)` stream.
  **[破坏性] 本意为匹配 `client.messages.stream()` 对象的 `isinstance(x, anthropic.Stream)` / `AsyncStream` 检查**现在返回 `False`（兼容垫片及其 `DeprecationWarning` 已移除）。改为检查 `anthropic.lib.streaming.MessageStream` / `AsyncMessageStream`；仅当值确实是原始 `create(stream=True)` 流时才保留 `Stream`。

## Step 9: Header names are matched case-insensitively / 第 9 步：头部名称按大小写不敏感匹配

Usually nothing to edit. The SDK now merges `default_headers`, `extra_headers`, `with_options(default_headers=...)` and `ANTHROPIC_CUSTOM_HEADERS` case-insensitively: a later entry replaces an earlier header of the same name whatever its casing (including headers the SDK sets itself), and `omit` removes one the same way. Scan the Step 1 hits for two things and fix only those: **[DECIDE]** the same header name spelled with two casings where the code relied on both lines being sent (send one comma-joined value instead), and **[BREAKS]** `bytes` header values, which now raise - `.decode()` them.

通常无可修改。SDK 现在以大小写不敏感的方式合并 `default_headers`、`extra_headers`、`with_options(default_headers=...)` 与 `ANTHROPIC_CUSTOM_HEADERS`：后出现的条目会替换同名头部，无论其大小写如何（包括 SDK 自己设置的头部），`omit` 也以同样方式移除。在第 1 步的命中中只查两件事并只修这两件：**[需决策]**同一头部名以两种大小写拼写、且代码依赖两行都被发送的情况（改为发送一个逗号连接的值），以及 **[破坏性]**`bytes` 头部值——现在会抛错，对它们 `.decode()`。

## Step 10: Bedrock - a region is required / 第 10 步：Bedrock——必须提供区域

**[DECIDE]** `AnthropicBedrock()` / `AsyncAnthropicBedrock()` used to warn and fall back to `us-east-1` when no region was configured; they now raise `ValueError` at construction. Resolution order: `aws_region=` -> `AWS_REGION` / `AWS_DEFAULT_REGION` -> the region configured for the boto3 session / `aws_profile` (the profile is now honoured for region lookup). For each construction without `aws_region=`, check whether the deployment provides a region (env files, Dockerfiles, deployment manifests, AWS profile config in the repo). If it demonstrably does, nothing to do; if you cannot tell, do **not** invent a region - list the call site in the report as needing `aws_region=` or `AWS_REGION`, and only hardcode `"us-east-1"` if the user confirms that the old implicit default is what they were actually using.

**[需决策]** `AnthropicBedrock()` / `AsyncAnthropicBedrock()` 过去在未配置区域时只警告并回退到 `us-east-1`；现在它们在构造时抛出 `ValueError`。解析顺序：`aws_region=` -> `AWS_REGION` / `AWS_DEFAULT_REGION` -> boto3 会话 / `aws_profile` 配置的区域（现在区域查找会遵循 profile）。对每个没有 `aws_region=` 的构造，检查部署是否提供了区域（env 文件、Dockerfile、部署清单、仓库内的 AWS profile 配置）。若能证明提供了，则无需处理；若无法判断，**不要**凭空编一个区域——在报告中把该调用点列为需要 `aws_region=` 或 `AWS_REGION`，只有当用户确认旧隐式默认正是他们实际使用的区域时才硬编码 `"us-east-1"`。

Streaming from Bedrock also changes: event types the SDK does not know are now skipped instead of yielded - the only known case is the `amazon-bedrock-invocationMetrics` frame. Code that filtered those frames out can be deleted; code that *consumed* invocation metrics loses them on 1.x - **[DECIDE]** list it in the report (the SDK asks such users to open an issue).

从 Bedrock 流式读取也有变化：SDK 不认识的事件类型现在会被跳过而不是产出——已知唯一的例子是 `amazon-bedrock-invocationMetrics` 帧。把这些帧过滤掉的代码可以删除；*消费*调用指标的代码在 1.x 上会拿不到它们——**[需决策]**在报告中列出（SDK 请这类用户提 issue）。

## Step 11: Verify / 第 11 步：验证

1. Re-run the Step 1 greps over the scope. Every remaining hit needs a reason (unrelated `httpx` use, `Raw*` names, helper `output_format=Model`, ...) - put the reasons in the report.
   在范围内重跑第 1 步的 grep。每个残留命中都需要一个理由（无关的 `httpx` 使用、`Raw*` 名称、辅助方法的 `output_format=Model` 等）——把理由写进报告。
2. `python -m compileall -q <scope>` must pass. If the project has a type checker configured, run it - nearly every missed call site is a type error on 1.x. Run the test suite if it is runnable without credentials.
   `python -m compileall -q <scope>` 必须通过。如果项目配置了类型检查器，运行它——在 1.x 上几乎每个漏改的调用点都是类型错误。测试套件能在无凭据下运行就运行。
3. If 1.x is installed in the environment: `python -c "import anthropic, httpx2; print(anthropic.__version__)"`.
   如果环境中已安装 1.x：`python -c "import anthropic, httpx2; print(anthropic.__version__)"`。

## Step 12: Report / 第 12 步：报告

Lead with the outcome, then:

先说结果，然后：

- what changed, grouped by the steps above, with file counts and the notable files;
  改了什么，按上述步骤分组，给出文件数量与值得注意的文件；
- **decisions the user owns** - Python floor / CI matrix (Step 2), import edits vs `alias_httpx()` (Step 3), sampling-parameter reliance (Step 6), the model chosen for ported completions calls (Step 5), duplicate-casing headers (Step 9), Bedrock regions and invocation metrics (Step 10);
  **属于用户的决策**——Python 下限 / CI 矩阵（第 2 步）、改导入还是 `alias_httpx()`（第 3 步）、对采样参数的依赖（第 6 步）、移植的补全调用所选的模型（第 5 步）、重复大小写的头部（第 9 步）、Bedrock 区域与调用指标（第 10 步）；
- if you introduced `httpx2` anywhere, one provenance line, because reviewers and supply-chain scanners flag unfamiliar package names as possible typosquats: it is the SDK's own HTTP dependency, the maintained fork of `httpx` by its original author, published by Pydantic (`github.com/pydantic/httpx2`), version line 2.x;
  如果你在任何地方引入了 `httpx2`，写一行出处说明，因为评审者与供应链扫描器会把陌生的包名标记为可能的 typosquat（仿冒包）：它是 SDK 自己的 HTTP 依赖、`httpx` 原作者维护的分支、由 Pydantic 发布（`github.com/pydantic/httpx2`），版本线 2.x；
- what you could not verify (offline PyPI check, no type checker, tests not runnable, pre-commit hooks that need the new packages installed) and the exact commands to finish: the install / lock command and, if relevant, `pip uninstall httpx-aiohttp`.
  你无法验证的内容（离线无法查 PyPI、无类型检查器、测试无法运行、需要先装新包的 pre-commit 钩子）以及完成所需的精确命令：安装 / 锁定命令，以及（如相关）`pip uninstall httpx-aiohttp`。

## Checklist / 核对清单

- [ ] **[BREAKS]** `anthropic` requirement moved to 1.x in the project's pin style; lockfile regenerated or command given
  **[破坏性]** `anthropic` 依赖要求已按项目的钉版风格移到 1.x；锁文件已重新生成或已给出命令
- [ ] **[DECIDE]** Python >= 3.10 floor and CI matrix proposed as a separate hunk
  **[需决策]** Python >= 3.10 下限与 CI 矩阵已作为独立补丁块提出
- [ ] **[BREAKS]** `httpx` objects passed to / received from the SDK (custom transports, auth flows and event hooks included) come from `httpx2` - or **[DECIDE]** `httpx2.alias_httpx()` at the top of an application entry point; `httpx`-patching instrumentation / mocking (`respx`, `pytest-httpx`, `vcrpy`, OpenTelemetry, Sentry) covered by the alias; `httpx-aiohttp` dropped; `httpx2>=2.0` declared if imported
  **[破坏性]** 传给 / 来自 SDK 的 `httpx` 对象（含自定义传输器、认证流程与事件钩子）均来自 `httpx2`——或 **[需决策]** 在应用入口点顶端使用 `httpx2.alias_httpx()`；给 `httpx` 打补丁的埋点 / 模拟（`respx`、`pytest-httpx`、`vcrpy`、OpenTelemetry、Sentry）已被别名覆盖；`httpx-aiohttp` 已移除；若直接导入则声明 `httpx2>=2.0`
- [ ] **[BREAKS]** async `.with_raw_response`: `await` on `parse()/json()/text()/read()`; `.text` -> `.text()`, `.content` -> `.read()` everywhere; `LegacyAPIResponse` annotations replaced
  **[破坏性]** 异步 `.with_raw_response`：对 `parse()/json()/text()/read()` 加 `await`；所有位置 `.text` -> `.text()`、`.content` -> `.read()`；`LegacyAPIResponse` 注解已替换
- [ ] **[BREAKS]** `completions.create` / `HUMAN_PROMPT` / `AI_PROMPT` ported to Messages; **[DECIDE]** model choice surfaced
  **[破坏性]** `completions.create` / `HUMAN_PROMPT` / `AI_PROMPT` 已移植到 Messages；**[需决策]** 模型选择已呈现给用户
- [ ] **[BREAKS]** `temperature` / `top_p` / `top_k` removed from SDK calls - or, **[DECIDE]**, moved to `extra_body` only where the call pins an older model *and* visibly depends on the setting; raw `output_format={...}` -> `output_config={"format": ...}` everywhere (helpers included); helper `output_format=Model` untouched
  **[破坏性]** SDK 调用中的 `temperature` / `top_p` / `top_k` 已移除——或 **[需决策]** 仅当调用钉住较旧模型*且*明显依赖该设置时移入 `extra_body`；原始 `output_format={...}` -> `output_config={"format": ...}` 全面完成（含辅助方法）；辅助方法的 `output_format=Model` 未动
- [ ] **[BREAKS]** `BetaBase64PDFBlockParam` -> `BetaRequestDocumentBlockParam`; `Transport`/`AsyncTransport`/`ProxiesTypes` -> `httpx2` names; `READ_MAX_BYTES` -> `DEFAULT_MAX_FILE_BYTES`
  **[破坏性]** `BetaBase64PDFBlockParam` -> `BetaRequestDocumentBlockParam`；`Transport`/`AsyncTransport`/`ProxiesTypes` -> `httpx2` 名称；`READ_MAX_BYTES` -> `DEFAULT_MAX_FILE_BYTES`
- [ ] **[BREAKS]** `parse(stream=)` -> `messages.stream()`; `compaction_control` -> server-side compaction; `body=bytes` -> `content=`; `Stream` isinstance checks retargeted
  **[破坏性]** `parse(stream=)` -> `messages.stream()`；`compaction_control` -> 服务端压缩；`body=bytes` -> `content=`；`Stream` isinstance 检查已改目标
- [ ] **[DECIDE]** duplicate-casing headers joined; **[BREAKS]** `bytes` header values decoded
  **[需决策]** 重复大小写的头部已合并；**[破坏性]** `bytes` 头部值已解码
- [ ] **[DECIDE]** Bedrock constructions without a discoverable region listed, not guessed; invocation-metrics consumers flagged
  **[需决策]** 无法发现区域的 Bedrock 构造已列出而非猜测；调用指标的消费者已标记
- [ ] Step 11 verification run and Step 12 report written
  第 11 步验证已执行，第 12 步报告已写好
