<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Self-Hosted Sandboxes / 托管智能体 - 自托管沙箱

With `config.type: "self_hosted"`, the **agent loop stays on Anthropic's orchestration layer** but **tool execution moves to infrastructure you control** - bash, file ops, and code run inside your container, so filesystem contents and the sandbox's network egress never leave your environment. (`web_search` / `web_fetch` are the exception: they run on Anthropic's servers in both environment types - restrict them with `allowed_domains` / `blocked_domains` in the agent toolset, `shared/managed-agents-tools.md` § Web search & web fetch settings.) Tool inputs/outputs still flow to Anthropic's control plane so the model can see results; the agent's skills and the contents of any attached memory stores are stored by Anthropic and copied into your sandbox for the session (memory changes sync back - see § Memory stores). Contrast with `config.type: "cloud"`, where Anthropic runs the container. Connectivity is **outbound-only**: your worker long-polls Anthropic's work queue; Anthropic never dials into your network.

使用 `config.type: "self_hosted"` 时，**智能体循环仍留在 Anthropic 的编排层**，但**工具执行转移到你掌控的基础设施上**——bash、文件操作和代码都在你的容器内运行，因此文件系统内容和沙箱的网络出站流量都不会离开你的环境。（`web_search` / `web_fetch` 是例外：无论哪种环境类型，它们都在 Anthropic 的服务器上运行——请通过智能体工具集中的 `allowed_domains` / `blocked_domains` 加以限制，参见 `shared/managed-agents-tools.md` § Web search & web fetch settings。）工具的输入/输出仍会流向 Anthropic 的控制平面，以便模型看到结果；智能体的技能以及任何挂载的内存存储的内容由 Anthropic 存储，并在会话期间复制到你的沙箱中（内存更改会同步回去——参见 § Memory stores）。与之对比，`config.type: "cloud"` 由 Anthropic 运行容器。连接是**仅出站（outbound-only）**的：你的 worker 长轮询 Anthropic 的工作队列；Anthropic 绝不会拨入你的网络。

【评论】"仅出站"连接设计意味着宿主机无需开放任何入站端口，这是自托管方案中收缩攻击面的常见做法；代价是取活与回传都要靠客户端主动轮询。

## Flow / 流程

```
1. Create environment:      config: {type: "self_hosted"}        -> env_...
2. Generate environment key (Console, on the environment page)   -> sk-ant-oat01-...  as ANTHROPIC_ENVIRONMENT_KEY
3. Run a worker:            EnvironmentWorker.run()  or  ant beta:worker poll
4. Sessions reference       environment_id=env_... exactly as for cloud
```

## Create the environment / 创建环境

```python
client = anthropic.Anthropic()

environment = client.beta.environments.create(
    name="self-hosted", config={"type": "self_hosted"}
)
```

`{"type": "self_hosted"}` is the entire config - there are no pool, capacity, or networking sub-fields; you control those on your side.

`{"type": "self_hosted"}` 就是全部配置——没有 pool、capacity 或 networking 子字段；这些由你在自己一侧控制。

## Run a worker - SDK (primary path) / 运行 worker - SDK（主要路径）

`EnvironmentWorker` wraps the poll -> dispatch -> tool-execute loop. `.run()` is the always-on loop (loops until cancelled). `.handle_item()` / `.handleItem()` / `.HandleItem()` services **one already-claimed** work item without polling - IDs fall back to `ANTHROPIC_WORK_ID` / `ANTHROPIC_ENVIRONMENT_ID` / `ANTHROPIC_SESSION_ID`, the key to the worker's own `environment_key` and then `ANTHROPIC_ENVIRONMENT_KEY`, and the per-session secret to `ANTHROPIC_WORK_SECRET`, so inside an `ant beta:worker poll --on-work` container it needs no arguments. It ignores (and force-stops) non-session work items itself. There is no `run_one()`; claiming is done by `.run()` or by the mid-level poller (below).

`EnvironmentWorker` 封装了轮询 -> 分发 -> 执行工具的循环。`.run()` 是常驻循环（一直运行直到被取消）。`.handle_item()` / `.handleItem()` / `.HandleItem()` 在不轮询的情况下处理**单个已认领**的工作项——ID 依次回退到 `ANTHROPIC_WORK_ID` / `ANTHROPIC_ENVIRONMENT_ID` / `ANTHROPIC_SESSION_ID`，密钥回退到 worker 自身的 `environment_key`，再回退到 `ANTHROPIC_ENVIRONMENT_KEY`，会话级密钥回退到 `ANTHROPIC_WORK_SECRET`，因此在 `ant beta:worker poll --on-work` 容器内调用时无需任何参数。它会自行忽略（并强制停止）非会话类工作项。不存在 `run_one()`；认领由 `.run()` 或中层轮询器（见下文）完成。

**Python - always-on: / Python - 常驻模式：**

```python
import asyncio
import contextlib
import os
import signal
from anthropic import AsyncAnthropic
from anthropic.lib.environments import EnvironmentWorker


async def main() -> None:
    environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
    environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
    async with AsyncAnthropic(auth_token=environment_key) as client:
        worker = EnvironmentWorker(
            client,
            environment_id=environment_id,
            environment_key=environment_key,
            workdir="/workspace",
        )
        task = asyncio.create_task(worker.run())
        # Cancel the task (don't kill the process): the worker stops its in-flight
        # work item and uploads changed memory files before exiting.
        loop = asyncio.get_running_loop()
        for signum in (signal.SIGINT, signal.SIGTERM):
            loop.add_signal_handler(signum, task.cancel)
        with contextlib.suppress(asyncio.CancelledError):
            await task


asyncio.run(main())
```

**TypeScript - always-on: / TypeScript - 常驻模式：**

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { EnvironmentWorker } from "@anthropic-ai/sdk/helpers/beta/environments";

const environmentKey = process.env.ANTHROPIC_ENVIRONMENT_KEY!;
const environmentId = process.env.ANTHROPIC_ENVIRONMENT_ID!;
const client = new Anthropic({ authToken: environmentKey });
const ctrl = new AbortController();
process.once("SIGTERM", () => ctrl.abort());
process.once("SIGINT", () => ctrl.abort());

await new EnvironmentWorker({
  client,
  environmentId,
  environmentKey,
  workdir: "/workspace",
  signal: ctrl.signal
}).run();
```

**Customizing tools.** `EnvironmentWorker` runs the built-in toolset by default. To add or replace tools, use `AgentToolContext(workdir=, client=, session_id=)` with `beta_agent_toolset(env)` / `betaAgentToolset(env)` and pass the resulting tools to the lower-level `tool_runner()`. Skills attached to the agent are downloaded into `{workdir}/skills/<name>/` before tool calls begin (`AgentToolContext` handles this when given `client` and `session_id`). Downloaded skill files are marked executable automatically by the CLI and SDK; if you implement skills download yourself, you set permissions.

**自定义工具。**`EnvironmentWorker` 默认运行内置工具集。要新增或替换工具，请使用 `AgentToolContext(workdir=, client=, session_id=)` 配合 `beta_agent_toolset(env)` / `betaAgentToolset(env)`，并把生成的工具传给更低层的 `tool_runner()`。挂载到智能体的技能会在工具调用开始前下载到 `{workdir}/skills/<name>/`（当给定 `client` 和 `session_id` 时，`AgentToolContext` 会处理这一步）。下载的技能文件会被 CLI 和 SDK 自动标记为可执行；如果你自行实现技能下载，则需自行设置权限。

> **Runtime deps:** the SDK helpers require `/bin/bash` at that exact path (not consulted via `PATH`). The TypeScript SDK additionally requires `unzip` and `tar` on `PATH` and Node.js 22+; Python and Go use their standard libraries for archive extraction. Memory stores additionally need a POSIX host (Linux or macOS - not Windows, the worker opens memory files with `O_NOFOLLOW`) with a writable `/mnt/memory` - see § Memory stores.

> **Runtime deps / 运行时依赖：** SDK 辅助工具要求 `/bin/bash` 位于该确切路径（不通过 `PATH` 查找）。TypeScript SDK 还要求 `PATH` 上有 `unzip` 和 `tar`，以及 Node.js 22+；Python 和 Go 使用各自的标准库进行归档解压。内存存储还需要 POSIX 宿主机（Linux 或 macOS——不支持 Windows，worker 以 `O_NOFOLLOW` 打开内存文件）以及可写的 `/mnt/memory`——参见 § Memory stores。

**File-tool confinement.** `AgentToolContext` confines `read`/`write`/`edit`/`glob`/`grep` to the working directory plus `allowed_roots` (`allowedRoots` / `AllowedRoots`); `write` and `edit` also refuse paths under `read_only_roots` (`readOnlyRoots` / `ReadOnlyRoots`). `EnvironmentWorker` adds the session's memory store directories to these lists itself. This is a guardrail for the file tools only - it does **not** constrain `bash`. The old `unrestricted_paths` option is no longer accepted (passing it raises); add directories to `allowed_roots` instead.

**文件工具限制。**`AgentToolContext` 将 `read`/`write`/`edit`/`glob`/`grep` 限制在工作目录及 `allowed_roots`（`allowedRoots` / `AllowedRoots`）之内；`write` 和 `edit` 还会拒绝 `read_only_roots`（`readOnlyRoots` / `ReadOnlyRoots`）下的路径。`EnvironmentWorker` 会自行把会话的内存存储目录加入这些列表。这只是针对文件工具的防护栏——**并不**约束 `bash`。旧的 `unrestricted_paths` 选项不再被接受（传入会抛出异常）；请改为把目录加入 `allowed_roots`。

【评论】文件工具的路径边界对 bash 不生效，说明该限制属于工具层约定而非操作系统级隔离；真正的文件访问隔离仍依赖容器自身的加固。

## Run a worker - `ant` CLI (fixed tools) / 运行 worker - `ant` CLI（固定工具）

The `ant` CLI ships a worker with the fixed built-in toolset (`bash`, `read`, `write`, `edit`, `glob`, `grep`). Install per `shared/anthropic-cli.md`, then:

`ant` CLI 自带一个使用固定内置工具集（`bash`、`read`、`write`、`edit`、`glob`、`grep`）的 worker。按 `shared/anthropic-cli.md` 安装后执行：

```sh
export ANTHROPIC_ENVIRONMENT_KEY=sk-ant-oat01-...
ant beta:worker poll --environment-id env_... --workdir /workspace
```

- `--workdir` is the directory tools operate in (default `.`); tool calls are sandboxed to it.
  `--workdir` 是工具的工作目录（默认 `.`）；工具调用被沙箱限定在该目录内。
- `--environment-key` overrides the env var.
  `--environment-key` 覆盖对应的环境变量。
- `--on-work <script>` runs your script per work item (e.g. to spin a fresh container per session - see Container orchestration below).
  `--on-work <script>` 对每个工作项运行你的脚本（例如为每个会话启动一个全新容器——参见下文 Container orchestration）。
- `--unrestricted-paths`, `--max-idle` (default `60s`), `--log-format` - see `ant beta:worker poll --help`.
  `--unrestricted-paths`、`--max-idle`（默认 `60s`）、`--log-format`——参见 `ant beta:worker poll --help`。
- Flags fall back to env vars (`ANTHROPIC_ENVIRONMENT_ID`, `ANTHROPIC_ENVIRONMENT_KEY`).
  各标志可回退到环境变量（`ANTHROPIC_ENVIRONMENT_ID`、`ANTHROPIC_ENVIRONMENT_KEY`）。
- Exits cleanly on SIGTERM/SIGINT after draining in-flight work.
  收到 SIGTERM/SIGINT 后，会在排空进行中的工作后干净退出。
- **Fixed toolset** - for custom tools, use the SDK worker above.
  **固定工具集**——如需自定义工具，请使用上文的 SDK worker。
- **Does not mount memory stores.** A session that attaches one still runs, but the agent finds nothing at the store's `/mnt/memory/<store-name>/` directory and nothing syncs back. To combine the CLI poller with memory stores, keep `ant beta:worker poll --on-work` on the host and run the **SDK** worker (`EnvironmentWorker.handle_item()`) inside the per-session sandbox - see § Memory stores -> Sandbox-per-session.
  **不会挂载内存存储。**挂载了内存存储的会话仍会运行，但智能体在存储的 `/mnt/memory/<store-name>/` 目录下找不到任何内容，也不会有任何内容同步回去。要把 CLI 轮询器与内存存储结合使用，请在宿主机上保留 `ant beta:worker poll --on-work`，并在每会话沙箱内运行 **SDK** worker（`EnvironmentWorker.handle_item()`）——参见 § Memory stores -> Sandbox-per-session。

Inside an `--on-work` container, run `ant beta:worker run --workdir <dir>` as the entrypoint (or the SDK worker, if the session needs memory stores).

在 `--on-work` 容器内，以 `ant beta:worker run --workdir <dir>` 作为入口点运行（若会话需要内存存储，则改用 SDK worker）。

## Webhook-driven wake (instead of always-on) / 基于 Webhook 的唤醒（替代常驻模式）

Register a webhook for `session.status_run_started` (see `shared/managed-agents-webhooks.md`), verify the delivery, then **drain** the queue with the poller (`drain=True` stops when it's empty; `block_ms=None` is non-blocking; `auto_stop=False` because `handle_item` force-stops the item itself) and hand each claimed item to `handle_item()`. **Don't `await` the drain inside the HTTP handler** - a session run outlives the webhook delivery timeout, so acknowledge the delivery and run the drain as a background task (`asyncio.create_task` / a detached promise / a goroutine off `context.Background()`), keeping the process alive until it finishes:

为 `session.status_run_started` 注册一个 webhook（参见 `shared/managed-agents-webhooks.md`），校验送达后，用轮询器**排空**队列（`drain=True` 表示队列空时停止；`block_ms=None` 表示非阻塞；`auto_stop=False` 是因为 `handle_item` 会自行强制停止该项），并把每个已认领的项交给 `handle_item()`。**不要在 HTTP 处理器内 `await` 排空过程**——一次会话运行的时长会超过 webhook 的送达超时，因此应确认送达后把排空作为后台任务运行（`asyncio.create_task` / 分离的 promise / 基于 `context.Background()` 的 goroutine），并保持进程存活直至其完成：

```python
import asyncio
import os
import anthropic

environment_key = os.environ["ANTHROPIC_ENVIRONMENT_KEY"]
environment_id = os.environ["ANTHROPIC_ENVIRONMENT_ID"]
client = anthropic.AsyncAnthropic(
    auth_token=environment_key,
)  # reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env for webhooks.unwrap()


async def handle(raw: bytes, headers: dict[str, str]) -> dict:
    event = client.beta.webhooks.unwrap(raw.decode(), headers=headers)
    if event.data.type != "session.status_run_started":
        return {"status": "ignored"}
    asyncio.create_task(drain())  # keep a reference if your framework may GC it
    return {"status": "accepted"}


async def drain() -> None:
    async for work in client.beta.environments.work.poller(
        environment_id=environment_id,
        environment_key=environment_key,
        block_ms=None,
        reclaim_older_than_ms=2000,
        drain=True,
        auto_stop=False,
    ):
        await client.beta.environments.work.worker(workdir="/workspace").handle_item(
            work_id=work.id,
            environment_id=environment_id,
            session_id=work.data.id,
            environment_key=environment_key,
            work_secret=work.secret,  # lets the worker mount the session's memory stores
        )
```

TypeScript: same shape with `client.beta.webhooks.unwrap(body, {headers})`, `client.beta.environments.work.poller({environmentId, environmentKey, blockMs: null, reclaimOlderThanMs: 2000, drain: true, autoStop: false})`, and `client.beta.environments.work.worker({workdir}).handleItem({workId, environmentId, sessionId, environmentKey, workSecret: work.secret})`. Go: no `RunOne` convenience either - `environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{EnvironmentID, EnvironmentKey, BlockMs: param.Null[int64](), ReclaimOlderThanMs: param.NewOpt[int64](2000), Drain: true, AutoStop: param.NewOpt(false)})`, then `worker.HandleItem(ctx, environments.HandleItemOptions{WorkID: item.ID, EnvironmentID: item.EnvironmentID, SessionID: item.Data.ID, EnvironmentKey, WorkSecret: item.Secret})` per `poller.Next()` item, in a goroutine off `context.Background()`. Always pass the work item's `secret` through, or sessions with memory stores fail at claim time. `handle_item` skips non-session work items itself, so the drain loop needs no `work.data.type` check.

TypeScript：结构相同，使用 `client.beta.webhooks.unwrap(body, {headers})`、`client.beta.environments.work.poller({environmentId, environmentKey, blockMs: null, reclaimOlderThanMs: 2000, drain: true, autoStop: false})`，以及 `client.beta.environments.work.worker({workdir}).handleItem({workId, environmentId, sessionId, environmentKey, workSecret: work.secret})`。Go：同样没有 `RunOne` 便捷方法——使用 `environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{EnvironmentID, EnvironmentKey, BlockMs: param.Null[int64](), ReclaimOlderThanMs: param.NewOpt[int64](2000), Drain: true, AutoStop: param.NewOpt(false)})`，然后对每个 `poller.Next()` 项，在一个基于 `context.Background()` 的 goroutine 中调用 `worker.HandleItem(ctx, environments.HandleItemOptions{WorkID: item.ID, EnvironmentID: item.EnvironmentID, SessionID: item.Data.ID, EnvironmentKey, WorkSecret: item.Secret})`。务必把工作项的 `secret` 传递下去，否则挂载内存存储的会话会在认领时失败。`handle_item` 会自行跳过非会话工作项，因此排空循环无需检查 `work.data.type`。

## Container orchestration (mid-level) / 容器编排（中层）

`EnvironmentWorker.run()` polls and executes tools in the same process. To run each session in its **own** container, use the mid-level poller in a thin orchestrator - Python `client.beta.environments.work.poller(environment_id=, environment_key=, drain=, block_ms=, reclaim_older_than_ms=, auto_stop=)`; TypeScript `new WorkPoller({client, environmentId, environmentKey, autoStop})` from `@anthropic-ai/sdk/helpers/beta/environments` - and, for each yielded `work` item, start a fresh container with these env vars injected, whose entrypoint runs `ant beta:worker run` or an `EnvironmentWorker(...).handle_item()` (required if the session attaches memory stores). `block_ms` is 1-999 (or `None` for non-blocking); `reclaim_older_than_ms` re-claims items leased to a dead worker; `drain` stops once the queue is empty; `auto_stop` posts a stop signal after the iterator exits (set `False` when the launched container owns the stop call). Go: `environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{EnvironmentID, EnvironmentKey, BlockMs, ReclaimOlderThanMs, Drain, AutoStop: param.NewOpt(false)})` with `poller.Next()` / `poller.Current()` / `poller.Err()`.

`EnvironmentWorker.run()` 在同一进程中完成轮询和工具执行。要在**独立的**容器中运行每个会话，请在一个轻量编排器中使用中层轮询器——Python 的 `client.beta.environments.work.poller(environment_id=, environment_key=, drain=, block_ms=, reclaim_older_than_ms=, auto_stop=)`；TypeScript 的 `@anthropic-ai/sdk/helpers/beta/environments` 中的 `new WorkPoller({client, environmentId, environmentKey, autoStop})`——并对每个产出的 `work` 项，在注入这些环境变量后启动一个全新容器，其入口点运行 `ant beta:worker run` 或 `EnvironmentWorker(...).handle_item()`（若会话挂载了内存存储则为必需）。`block_ms` 取值 1-999（`None` 表示非阻塞）；`reclaim_older_than_ms` 用于重新认领租约给已死亡 worker 的项；`drain` 在队列清空后停止；`auto_stop` 在迭代器退出后发送停止信号（当由启动的容器自行负责停止调用时设为 `False`）。Go：`environments.NewWorkPoller(ctx, client, environments.WorkPollerOptions{EnvironmentID, EnvironmentKey, BlockMs, ReclaimOlderThanMs, Drain, AutoStop: param.NewOpt(false)})`，配合 `poller.Next()` / `poller.Current()` / `poller.Err()`。

| Env var | Value |
|---|---|
| `ANTHROPIC_SESSION_ID` | `work.data.id` |
| `ANTHROPIC_WORK_ID` | `work.id` |
| `ANTHROPIC_ENVIRONMENT_ID` | `work.environment_id` |
| `ANTHROPIC_ENVIRONMENT_KEY` | pass through |
| `ANTHROPIC_BASE_URL` | pass through |
| `ANTHROPIC_WORK_SECRET` | `work.secret` - the per-session credential the worker inside needs to mount memory stores. `ant beta:worker poll --on-work` does **not** set it for the spawned script; read it from the work-item JSON on stdin (`jq -r '.secret // empty'`) and pass it in. Only into the sandbox serving that session; never log it. |

| 环境变量 | 值 |
|---|---|
| `ANTHROPIC_SESSION_ID` | `work.data.id` |
| `ANTHROPIC_WORK_ID` | `work.id` |
| `ANTHROPIC_ENVIRONMENT_ID` | `work.environment_id` |
| `ANTHROPIC_ENVIRONMENT_KEY` | 原样传递 |
| `ANTHROPIC_BASE_URL` | 原样传递 |
| `ANTHROPIC_WORK_SECRET` | `work.secret`——容器内的 worker 挂载内存存储所需的会话级凭据。`ant beta:worker poll --on-work` **不会**为被启动的脚本设置它；请从 stdin 上的工作项 JSON 读取（`jq -r '.secret // empty'`）并传入。只传入服务于该会话的沙箱；绝不写入日志。 |

Skip items where `work.data.type != "session"` when you dispatch containers yourself (`handle_item` does this check for you).

自行分发容器时，跳过 `work.data.type != "session"` 的项（`handle_item` 会替你做这个检查）。

## Memory stores / 内存存储

Sessions on a self-hosted environment attach memory stores exactly like cloud sessions - `resources=[{"type": "memory_store", "memory_store_id": ..., "access": ...}]` at session create, up to 8 per session (see `shared/managed-agents-memory.md`). The difference is *who materializes them*: on cloud, Anthropic mounts a live FUSE filesystem; on self-hosted, the **SDK worker** (`EnvironmentWorker`, or its `handle_item()` / `handleItem()` / `HandleItem()`) downloads a working copy and syncs it. Requires the Python, TypeScript, or Go SDK; the `ant` CLI worker and the C#/Java/PHP/Ruby SDKs don't mount stores. Not available on Claude Platform on AWS.

自托管环境上的会话挂载内存存储的方式与云会话完全相同——在创建会话时传入 `resources=[{"type": "memory_store", "memory_store_id": ..., "access": ...}]`，每个会话最多 8 个（参见 `shared/managed-agents-memory.md`）。区别在于*由谁来落地它们*：云上由 Anthropic 挂载一个实时 FUSE 文件系统；自托管则由 **SDK worker**（`EnvironmentWorker`，或其 `handle_item()` / `handleItem()` / `HandleItem()`）下载一份工作副本并同步。需要 Python、TypeScript 或 Go SDK；`ant` CLI worker 以及 C#/Java/PHP/Ruby SDK 不会挂载存储。Claude Platform on AWS 上不可用。

**What the worker does** when it claims a work item whose session has stores attached:

当 worker 认领的工作项所属会话挂载了存储时，**它会做什么**：

1. Downloads each store to its mount path under `/mnt/memory/` - derived from the store's name, not a settable field (e.g. `/mnt/memory/user-preferences/` for a store named "User Preferences"); the same path cloud sessions use, and the session's system prompt describes it to the agent. Authenticates with the work item's per-session `secret`.
   把每个存储下载到 `/mnt/memory/` 下各自的挂载路径——路径由存储名称派生，不是可设置字段（例如名为 "User Preferences" 的存储对应 `/mnt/memory/user-preferences/`）；与云会话使用的路径相同，且会话的系统提示词会向智能体描述该路径。使用工作项的会话级 `secret` 进行认证。
2. Adds those directories to the file tools' `allowed_roots`, and `access: "read_only"` stores to `read_only_roots`, so the agent uses the ordinary `read`/`write`/`edit`/`glob`/`grep` tools on memories.
   把这些目录加入文件工具的 `allowed_roots`，`access: "read_only"` 的存储加入 `read_only_roots`，使智能体用普通的 `read`/`write`/`edit`/`glob`/`grep` 工具操作记忆。
3. Reconciles after tool calls, at most once per sync interval (default 15 s): remote changes are written to disk, files the agent changed are uploaded.
   在工具调用后进行对账，每个同步间隔最多一次（默认 15 秒）：远端更改写入磁盘，智能体更改过的文件被上传。
4. On session end: final sync, flushes pending uploads for up to 30 s, removes the directories. A worker that is *cancelled* mid-session skips the final sync but still uploads changed files and removes the directories; a worker that is *killed* runs no teardown at all.
   会话结束时：执行最终同步，最多用 30 秒冲刷待上传内容，然后删除这些目录。会话中途被*取消*的 worker 会跳过最终同步，但仍会上传已更改的文件并删除目录；被*杀死*的 worker 则完全不执行任何清理。

The store on Anthropic's side remains the source of truth - memory versions, redaction, and Console viewing/editing work as for cloud sessions, and the agent's memory reads/writes appear in the event stream as ordinary tool events. Because sync is interval-based, a change written by one self-hosted session is visible to another running session only after both have synced (typically well under a minute); cloud sessions see each other's changes almost immediately. Each store directory holds a marker file `.anthropic-memory-store` - leave it alone; the worker won't sync a directory whose marker is missing or altered.

Anthropic 一侧的存储始终是事实来源——内存版本、脱敏以及 Console 查看/编辑都与云会话一致，智能体的内存读写会以普通工具事件的形式出现在事件流中。由于同步基于间隔，一个自托管会话写入的更改，要等双方都完成同步后才能被另一个运行中的会话看到（通常远小于一分钟）；云会话则几乎立即看到彼此的更改。每个存储目录中都有一个标记文件 `.anthropic-memory-store`——不要动它；标记缺失或被改动的目录不会被 worker 同步。

**Prepare the host.** POSIX (Linux/macOS) only; a case-sensitive filesystem is recommended. Before starting the worker:

**准备宿主机。**仅支持 POSIX（Linux/macOS）；建议使用大小写敏感的文件系统。启动 worker 之前：

```bash
sudo mkdir -p /mnt/memory && sudo chown "$USER" /mnt/memory
```

Do **not** create the per-store directories yourself - the worker creates each store's directory when a session starts, **refuses the work item if something already exists at that path**, and removes it at session end. Two rules follow: (a) two sessions can't mount the same store on one host simultaneously (they need the same path) - give each session its own sandbox; (b) stop workers gracefully. `EnvironmentWorker` installs no signal handlers: wire SIGTERM/SIGINT to cancellation yourself (abort the `signal` in TypeScript, cancel the context in Go, cancel the task running `run()` / `handle_item()` in Python), send SIGTERM, and allow >= 30 s before any hard kill. If a worker is killed before teardown, remove the leftover directory under `/mnt/memory/` before the next session that attaches that store - unsynced edits in it are lost.

**不要**自行创建各存储的目录——worker 在会话开始时创建每个存储的目录，**若该路径已有内容则拒绝该工作项**，并在会话结束时删除目录。由此得出两条规则：(a) 两个会话不能在一台主机上同时挂载同一个存储（它们需要同一路径）——给每个会话分配自己的沙箱；(b) 优雅地停止 worker。`EnvironmentWorker` 不安装任何信号处理器：请自行把 SIGTERM/SIGINT 接到取消操作上（TypeScript 中 abort `signal`，Go 中取消 context，Python 中取消运行 `run()` / `handle_item()` 的任务），发送 SIGTERM，并在任何强杀之前留出 >= 30 秒。如果 worker 在清理前被杀死，请在该存储的下一次会话开始前移除 `/mnt/memory/` 下的残留目录——其中未同步的编辑会丢失。

**Sandbox-per-session** (the pattern from § Container orchestration) satisfies rule (a) automatically. Keep `ant beta:worker poll --on-work` (or the SDK poller) on the host; build the per-session image around the SDK worker instead of `ant beta:worker run` - its entrypoint constructs `EnvironmentWorker` and calls `handle_item()`, which reads the session/work/environment IDs from the `ANTHROPIC_*` vars and the per-session secret from `ANTHROPIC_WORK_SECRET` (or pass `work_secret=` / `workSecret` / `WorkSecret` explicitly). `--on-work` does not set `ANTHROPIC_WORK_SECRET` for the spawn script, so read it from the work-item JSON on stdin:

**每会话一个沙箱**（§ Container orchestration 的模式）自动满足规则 (a)。在宿主机上保留 `ant beta:worker poll --on-work`（或 SDK 轮询器）；为每个会话构建围绕 SDK worker（而非 `ant beta:worker run`）的镜像——其入口点构造 `EnvironmentWorker` 并调用 `handle_item()`，后者从 `ANTHROPIC_*` 变量读取会话/工作/环境 ID，从 `ANTHROPIC_WORK_SECRET` 读取会话级密钥（或显式传入 `work_secret=` / `workSecret` / `WorkSecret`）。`--on-work` 不会为被启动的脚本设置 `ANTHROPIC_WORK_SECRET`，因此请从 stdin 上的工作项 JSON 中读取：

```bash
#!/bin/bash
# spawn.sh - called once per claimed work item; the work item arrives as JSON on stdin
ANTHROPIC_WORK_SECRET="$(jq -r '.secret // empty')"
export ANTHROPIC_WORK_SECRET
exec docker run --rm \
  -e ANTHROPIC_SESSION_ID -e ANTHROPIC_WORK_ID -e ANTHROPIC_ENVIRONMENT_ID \
  -e ANTHROPIC_ENVIRONMENT_KEY -e ANTHROPIC_BASE_URL -e ANTHROPIC_WORK_SECRET \
  my-sdk-worker-image
```

The per-session entrypoint is a few lines - no arguments needed, `handle_item()` reads the forwarded `ANTHROPIC_*` vars including `ANTHROPIC_WORK_SECRET`; wire signals to cancellation so a stopped container still uploads:

每会话入口点只需几行——无需参数，`handle_item()` 会读取转发的 `ANTHROPIC_*` 变量（包括 `ANTHROPIC_WORK_SECRET`）；请把信号接到取消操作上，使被停止的容器仍能完成上传：

```python
import asyncio, contextlib, os, signal
from anthropic import AsyncAnthropic
from anthropic.lib.environments import EnvironmentWorker


async def main() -> None:
    async with AsyncAnthropic(auth_token=os.environ["ANTHROPIC_ENVIRONMENT_KEY"]) as client:
        task = asyncio.create_task(EnvironmentWorker(client, workdir="/workspace").handle_item())
        loop = asyncio.get_running_loop()
        for signum in (signal.SIGINT, signal.SIGTERM):
            loop.add_signal_handler(signum, task.cancel)
        with contextlib.suppress(asyncio.CancelledError):
            await task


asyncio.run(main())
```

TypeScript: `new EnvironmentWorker({ client, workdir: "/workspace", signal: controller.signal }).handleItem()` with `process.once("SIGTERM"/"SIGINT", () => controller.abort())`. Go: `signal.NotifyContext(ctx, os.Interrupt, syscall.SIGTERM)` then `environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{Workdir: "/workspace"}).HandleItem(ctx, environments.HandleItemOptions{})`.

TypeScript：`new EnvironmentWorker({ client, workdir: "/workspace", signal: controller.signal }).handleItem()`，配合 `process.once("SIGTERM"/"SIGINT", () => controller.abort())`。Go：先 `signal.NotifyContext(ctx, os.Interrupt, syscall.SIGTERM)`，再 `environments.NewEnvironmentWorker(client, environments.EnvironmentWorkerOptions{Workdir: "/workspace"}).HandleItem(ctx, environments.HandleItemOptions{})`。

The image needs a writable `/mnt/memory`; the memory directories need **not** be bind-mounted to the host - the worker uploads before the sandbox exits, and a discarded sandbox leaves nothing to clean up. Stop a container early with a signal the entrypoint turns into cancellation, not a kill, so that upload still runs.

镜像需要可写的 `/mnt/memory`；内存目录**不需要**绑定挂载到宿主机——worker 会在沙箱退出前完成上传，被丢弃的沙箱不会留下需要清理的东西。提前停止容器时，请使用入口点会转换为取消操作的信号，而不是直接杀死，以便上传仍能执行。

**Configure sync** - two `EnvironmentWorker` options (constructor or `client.beta.environments.work.worker()` factory in Python; the options object in TypeScript; `environments.EnvironmentWorkerOptions` in Go):

**配置同步**——两个 `EnvironmentWorker` 选项（Python 中通过构造函数或 `client.beta.environments.work.worker()` 工厂；TypeScript 中是选项对象；Go 中是 `environments.EnvironmentWorkerOptions`）：

| Option | Python / TypeScript / Go | Behavior |
|---|---|---|
| Sync interval | `memory_sync_interval` (seconds) / `memorySyncIntervalMs` (ms) / `MemorySyncInterval` (duration) | Default 15 s, minimum 5 s. Shorter narrows the stale window at the cost of more memory-store requests. `None` / `null` / negative duration **disables memory support entirely** - stores are neither downloaded nor synced, and a session with stores attached runs without them even though its system prompt still describes them. Only disable on workers whose sessions never attach stores. While enabled, a work item that arrives without a `secret` for a session with stores **fails** rather than running memory-less. |
| Delete propagation | `memory_sync_deletes` / `memorySyncDeletes` / `MemorySyncDeletes` | `"enabled"` (default - deletes from the store once a later sync confirms the file is still gone), `"log_only"` (same checks, only logs what it would delete - use to audit before trusting `enabled`), `"disabled"` (never deletes from the store). Go: `environments.MemorySyncDeletesEnabled` (zero value) / `LogOnly` / `Disabled`. Uploads/downloads are unaffected. |

| 选项 | Python / TypeScript / Go | 行为 |
|---|---|---|
| 同步间隔 | `memory_sync_interval`（秒）/ `memorySyncIntervalMs`（毫秒）/ `MemorySyncInterval`（时长） | 默认 15 秒，最小 5 秒。更短可缩小数据过期窗口，代价是内存存储请求更多。`None` / `null` / 负时长会**完全禁用内存支持**——存储既不下载也不同步，挂载了存储的会话照常运行但没有存储可用，尽管其系统提示词仍在描述它们。仅在会话从不挂载存储的 worker 上禁用。启用时，若到达的工作项缺少 `secret` 而其会话挂载了存储，该工作项将**失败**，而不是在没有内存的情况下运行。 |
| 删除传播 | `memory_sync_deletes` / `memorySyncDeletes` / `MemorySyncDeletes` | `"enabled"`（默认——后续某次同步确认文件确实已删除后，才从存储中删除）、`"log_only"`（同样的检查，只记录本来会删除的内容——用于在信任 `enabled` 之前先行审计）、`"disabled"`（从不从存储中删除）。Go：`environments.MemorySyncDeletesEnabled`（零值）/ `LogOnly` / `Disabled`。上传/下载不受影响。 |

For example, sync every 10 s and only *log* would-be deletes: Python `EnvironmentWorker(client, environment_id=..., environment_key=..., workdir="/workspace", memory_sync_interval=10, memory_sync_deletes="log_only")`; TypeScript `new EnvironmentWorker({ client, environmentId, environmentKey, workdir: "/workspace", memorySyncIntervalMs: 10_000, memorySyncDeletes: "log_only" })`; Go `environments.EnvironmentWorkerOptions{..., MemorySyncInterval: 10 * time.Second, MemorySyncDeletes: environments.MemorySyncDeletesLogOnly}`.

例如，每 10 秒同步一次且仅*记录*本应执行的删除：Python 用 `EnvironmentWorker(client, environment_id=..., environment_key=..., workdir="/workspace", memory_sync_interval=10, memory_sync_deletes="log_only")`；TypeScript 用 `new EnvironmentWorker({ client, environmentId, environmentKey, workdir: "/workspace", memorySyncIntervalMs: 10_000, memorySyncDeletes: "log_only" })`；Go 用 `environments.EnvironmentWorkerOptions{..., MemorySyncInterval: 10 * time.Second, MemorySyncDeletes: environments.MemorySyncDeletesLogOnly}`。

**Read-only stores and conflicts.** For `access: "read_only"`, `write`/`edit` refuse changes under the directory (the only memory errors that reach the agent, as tool errors) and nothing uploads; the memory-store endpoints also reject writes made with the session's `secret`. `bash` edits aren't blocked locally - they never sync and the next remote change overwrites them. Conflicts resolve **in favor of the store**: if the agent changes a file that also changed remotely since the last sync, the worker keeps the store's version at the next sync, overwrites the local file, and logs a warning - `write`/`edit` still succeed and no error reaches the agent; it can re-read and re-apply.

**只读存储与冲突。**对于 `access: "read_only"`，`write`/`edit` 拒绝对该目录下的更改（这是唯一能到达智能体的内存错误，以工具错误形式出现），且不会上传任何内容；内存存储端点也会拒绝使用该会话 `secret` 发起的写入。`bash` 的修改在本地不受阻止——但它们永远不会同步，下一次远端更改会覆盖它们。冲突**以存储为准**解决：如果智能体修改了某个自上次同步以来远端也更改过的文件，worker 会在下次同步时保留存储版本、覆盖本地文件并记录警告——`write`/`edit` 仍然成功，没有任何错误到达智能体；它可以重新读取并重新应用。

【评论】冲突"以存储为准"意味着智能体的本地修改可能被静默回滚，只有日志警告可见；依赖内存做决策的自动化流程需要把这种最终一致性纳入考虑。

**Troubleshooting.** Mount and background-sync failures are *logged*, not reported to the session. If a store can't be mounted at claim time the worker fails the work item - the session emits no error event and sits `idle` (`requires_action` stop reason).

**故障排查。**挂载和后台同步失败只会被*记录日志*，不会上报给会话。如果认领时某个存储无法挂载，worker 会使该工作项失败——会话不发出错误事件，停留在 `idle` 状态（停止原因为 `requires_action`）。

| Log line / symptom | Cause | Fix |
|---|---|---|
| `the work item carried no sessions token` (Go: `ErrSessionMemoryNoToken`), work item fails | The per-session `secret` didn't reach the worker - memory on self-hosted isn't enabled for your org, or your spawn script didn't forward it | Forward `ANTHROPIC_WORK_SECRET` into the sandbox. If the in-process worker (poll + run in one process) still logs this, contact support |
| `something already exists at the memory store's path` | Leftover directory from a killed worker | Remove the named directory (unsynced edits are lost) |
| `cannot create the memory store's folder` + `the worker host must make this mount path writable` | Worker user can't create dirs under `/mnt/memory` | `mkdir -p /mnt/memory && chown <worker-user> /mnt/memory` |
| Session `idle` with `requires_action`, no error event, shortly after a claim | Worker failed the work item on a mount error above | Fix the host, then send `user.interrupt` - the work is re-queued and the next claim retries the mount |

| 日志行 / 症状 | 原因 | 修复 |
|---|---|---|
| `the work item carried no sessions token`（Go 中为 `ErrSessionMemoryNoToken`），工作项失败 | 会话级 `secret` 没有到达 worker——你的组织未启用自托管内存，或你的启动脚本没有转发它 | 把 `ANTHROPIC_WORK_SECRET` 转发进沙箱。如果进程内 worker（轮询与运行在同一进程）仍记录此日志，请联系支持 |
| `something already exists at the memory store's path` | 被杀死的 worker 留下的残留目录 | 移除所指目录（未同步的编辑会丢失） |
| `cannot create the memory store's folder` + `the worker host must make this mount path writable` | worker 用户无法在 `/mnt/memory` 下创建目录 | `mkdir -p /mnt/memory && chown <worker-user> /mnt/memory` |
| 认领后不久会话 `idle` 且 `requires_action`，无错误事件 | worker 因上述某个挂载错误使工作项失败 | 修复宿主机，然后发送 `user.interrupt`——工作会重新入队，下次认领重试挂载 |

## Monitoring & control / 监控与控制

These are **control-plane** calls - authenticate with `x-api-key` (not the environment key); `managed-agents-2026-04-01` beta header. **Call them from outside the worker host** - setting `ANTHROPIC_API_KEY` on the worker host exposes an organization-scoped credential to agent tool calls.

这些是**控制平面**调用——使用 `x-api-key` 认证（而非环境密钥）；需要 `managed-agents-2026-04-01` beta 头。**请从 worker 宿主机之外调用它们**——在 worker 宿主机上设置 `ANTHROPIC_API_KEY` 会把一个组织级范围的凭据暴露给智能体工具调用。

【评论】组织级 API 密钥与运行不可信工具代码的宿主机相隔离，是防止凭据经工具输入输出外泄的常规做法。

| SDK (`client.beta.environments.work.*`) | REST | CLI | Returns |
|---|---|---|---|
| `stats(environment_id)` | `GET /v1/environments/{id}/work/stats` | `ant beta:environments:work stats` | `{type:"work_queue_stats", depth, pending, oldest_queued_at, workers_polling}` |
| `stop(work_id, environment_id=)` | `POST /v1/environments/{id}/work/{work_id}/stop` | `ant beta:environments:work stop` | `work.state` |

| SDK（`client.beta.environments.work.*`） | REST | CLI | 返回 |
|---|---|---|---|
| `stats(environment_id)` | `GET /v1/environments/{id}/work/stats` | `ant beta:environments:work stats` | `{type:"work_queue_stats", depth, pending, oldest_queued_at, workers_polling}` |
| `stop(work_id, environment_id=)` | `POST /v1/environments/{id}/work/{work_id}/stop` | `ant beta:environments:work stop` | `work.state` |

## What changes vs `cloud` / 与 `cloud` 的差异

| Concern | `cloud` | `self_hosted` |
|---|---|---|
| Container lifecycle, hardening, networking | Anthropic | **You** - run non-root, read-only rootfs, drop caps; egress is whatever your VPC/firewall allows - except `web_search` / `web_fetch`, which run on Anthropic's servers either way (restrict them per tool with `allowed_domains` / `blocked_domains`) |
| `file` / `github_repository` resource mounting | Anthropic mounts into the container | **You** - pass pointers via `sessions.create(metadata={...})` and have your orchestrator fetch/clone before dispatch |
| `memory_store` resources | Mounted by Anthropic at `/mnt/memory/<name>/` (live FUSE mount) | **Supported via the SDK worker** (Python / TypeScript / Go `EnvironmentWorker`), which downloads each store to `/mnt/memory/<store-name>/` and syncs on an interval - see § Memory stores. Not mounted by the `ant` CLI worker; not available in the C#, Java, PHP, or Ruby SDKs. `memory_store` is the **only** resource type self-hosted environments accept - `file` / `github_repository` are still rejected with the 400 message "Environment env_... is a self-hosted environment. `resources` are not supported with self-hosted environments." (deployments targeting a self-hosted environment follow the same rule; the Console deployment form doesn't offer memory stores for them - use the API/SDK). |
| Vault `environment_variable` credentials | Supported (substituted at Anthropic-managed egress) | **Not yet supported** - egress is yours, so there's nowhere to substitute the secret. Use MCP credentials or a host-side custom tool (`shared/managed-agents-client-patterns.md` Pattern 9) |
| Built-in tools | Via `agent_toolset_20260401` | Supplied by your worker (`EnvironmentWorker` default / `beta_agent_toolset(env)` / `ant` CLI fixed set) |
| Skills download | Automatic | `EnvironmentWorker` / `AgentToolContext` fetch into `{workdir}/skills/` (needs `client` + `session_id`) |
| Claude Platform on AWS | Supported | Supported - the worker authenticates with AWS IAM (SigV4) or an AWS-Console-generated API key (Console-generated environment keys don't work against the AWS endpoint); attach the `AnthropicSelfHostedEnvironmentAccess` managed policy to the worker's principal. **Memory stores cannot be attached** to sessions on self-hosted environments there (rejected at session create); cloud environments attach them as usual. |
| SDK worker helpers | All SDKs | **Python, TypeScript, Go only** (`EnvironmentWorker` / poller not in Java, Ruby, PHP, or C#) - use one of those three or the `ant` CLI |

| 关注点 | `cloud` | `self_hosted` |
|---|---|---|
| 容器生命周期、加固、网络 | Anthropic | **你**——以非 root 运行、只读 rootfs、裁剪 capabilities；出站流量由你的 VPC/防火墙决定——但 `web_search` / `web_fetch` 无论如何都运行在 Anthropic 的服务器上（用 `allowed_domains` / `blocked_domains` 按工具限制） |
| `file` / `github_repository` 资源挂载 | Anthropic 挂载进容器 | **你**——通过 `sessions.create(metadata={...})` 传递指针，并让你的编排器在分发前抓取/克隆 |
| `memory_store` 资源 | 由 Anthropic 挂载到 `/mnt/memory/<name>/`（实时 FUSE 挂载） | **通过 SDK worker 支持**（Python / TypeScript / Go 的 `EnvironmentWorker`），它把每个存储下载到 `/mnt/memory/<store-name>/` 并按间隔同步——参见 § Memory stores。`ant` CLI worker 不挂载；C#、Java、PHP、Ruby SDK 中不可用。`memory_store` 是自托管环境接受的**唯一**资源类型——`file` / `github_repository` 仍会被 400 消息拒绝："Environment env_... is a self-hosted environment. `resources` are not supported with self-hosted environments."（面向自托管环境的部署遵循同一规则；Console 部署表单不为它们提供内存存储——请使用 API/SDK）。 |
| Vault `environment_variable` 凭据 | 支持（在 Anthropic 管理的出站处替换） | **暂不支持**——出站由你负责，没有替换密钥的位置。请使用 MCP 凭据或宿主机侧自定义工具（`shared/managed-agents-client-patterns.md` Pattern 9） |
| 内置工具 | 经 `agent_toolset_20260401` | 由你的 worker 提供（`EnvironmentWorker` 默认 / `beta_agent_toolset(env)` / `ant` CLI 固定集合） |
| 技能下载 | 自动 | `EnvironmentWorker` / `AgentToolContext` 抓取到 `{workdir}/skills/`（需要 `client` + `session_id`） |
| Claude Platform on AWS | 支持 | 支持——worker 用 AWS IAM（SigV4）或 AWS Console 生成的 API 密钥认证（Console 生成的环境密钥对 AWS 端点无效）；为 worker 的主体附加 `AnthropicSelfHostedEnvironmentAccess` 托管策略。在该处的自托管环境上**无法为会话挂载内存存储**（创建会话时被拒绝）；云环境照常挂载。 |
| SDK worker 辅助 | 所有 SDK | **仅 Python、TypeScript、Go**（`EnvironmentWorker` / 轮询器不在 Java、Ruby、PHP、C# 中）——请使用这三者之一或 `ant` CLI |

## Credentials / 凭据

| Credential | Format | Scope |
|---|---|---|
| `ANTHROPIC_ENVIRONMENT_KEY` | `sk-ant-oat01-...` | One environment's work queue. Generate in Console ("Generate environment key"). Pass as `auth_token=` / `authToken` on the client **and** as `environment_key=` / `environmentKey` on `EnvironmentWorker`. Store in a secrets manager; rotate on exposure. |
| `ANTHROPIC_WEBHOOK_SIGNING_KEY` | `whsec_...` | Webhook signature verification (if using webhook-driven wake). The SDK reads this env var automatically for `client.beta.webhooks.unwrap()`. |
| Work-item `secret` (`ANTHROPIC_WORK_SECRET`) | per-session, issued by Anthropic on the claimed work item | Posts that session's events and reads/writes the memory stores attached to it. You don't generate it; the in-process worker picks it up from the work item, and in the sandbox-per-session pattern you forward it into the sandbox yourself (or pass `work_secret=` / `workSecret` / `WorkSecret` explicitly). Treat like the environment key: only into the sandbox serving that session, never in images, shared volumes, or logs. |

| 凭据 | 格式 | 作用范围 |
|---|---|---|
| `ANTHROPIC_ENVIRONMENT_KEY` | `sk-ant-oat01-...` | 单个环境的工作队列。在 Console 中生成（"Generate environment key"）。在客户端上作为 `auth_token=` / `authToken` 传入，**并且**在 `EnvironmentWorker` 上作为 `environment_key=` / `environmentKey` 传入。存放在密钥管理器中；暴露后轮换。 |
| `ANTHROPIC_WEBHOOK_SIGNING_KEY` | `whsec_...` | Webhook 签名验证（如使用 webhook 驱动唤醒）。SDK 会为 `client.beta.webhooks.unwrap()` 自动读取该环境变量。 |
| 工作项 `secret`（`ANTHROPIC_WORK_SECRET`） | 会话级，由 Anthropic 在被认领的工作项上签发 | 发布该会话的事件，并读写其挂载的内存存储。它不由你生成；进程内 worker 从工作项中取得，而在每会话沙箱模式中由你转发进沙箱（或显式传入 `work_secret=` / `workSecret` / `WorkSecret`）。像对待环境密钥一样对待它：只进入服务于该会话的沙箱，绝不进入镜像、共享卷或日志。 |

## Security - what you own / 安全——你负责的部分

Container hardening; egress restriction for the sandbox (there is no default; the server-side `web_search` / `web_fetch` are governed only by their `allowed_domains` / `blocked_domains`); `ANTHROPIC_ENVIRONMENT_KEY` custody and rotation; one workspace + environment per trust boundary when running untrusted code; least-privilege for the tool process; log retention and redaction. **Anthropic cannot**: fast-revoke a leaked environment key, verify your image or supply chain, sandbox tool execution inside your container, or enforce retention after tool output reaches your infrastructure. **Memory stores** stay hosted by Anthropic (with version history), but the working copy under `/mnt/memory/` is yours for the session's duration: the worker deletes it on teardown, a killed worker leaves it behind, and permissions/isolation between sessions sharing a filesystem are your responsibility. A `read_only` store is protected from *upload*, not from local modification - `bash` can still change the local copy (later tool calls in that session read the changed copy until the store next changes that memory); disable `bash` or mount the path read-only if the agent must not alter even its local view. See the Self-Hosted Sandboxes Security page in `shared/live-sources.md` for the full checklist.

容器加固；沙箱的出站限制（没有默认值；服务器端的 `web_search` / `web_fetch` 仅受其 `allowed_domains` / `blocked_domains` 约束）；`ANTHROPIC_ENVIRONMENT_KEY` 的保管与轮换；运行不可信代码时每个信任边界一个工作区加一个环境；工具进程的最小权限；日志留存与脱敏。**Anthropic 无法**：快速吊销已泄露的环境密钥、校验你的镜像或供应链、在你容器内部对工具执行做沙箱隔离、或在工具输出到达你的基础设施之后强制留存。**内存存储**仍由 Anthropic 托管（带版本历史），但 `/mnt/memory/` 下的工作副本在会话期间归你所有：worker 会在清理时删除它，被杀死的 worker 会留下它，而共享文件系统的会话之间的权限/隔离由你负责。`read_only` 存储防的是*上传*，不防本地修改——`bash` 仍可更改本地副本（该会话后续的工具调用会读到更改后的副本，直到存储下次更改该记忆）；如果连智能体的本地视图都不允许被更改，请禁用 `bash` 或把该路径以只读方式挂载。完整检查清单见 `shared/live-sources.md` 中的 Self-Hosted Sandboxes Security 页面。
