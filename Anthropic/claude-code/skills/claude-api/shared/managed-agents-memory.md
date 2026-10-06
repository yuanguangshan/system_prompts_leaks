<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Memory Stores / 托管智能体 - 记忆存储

> **Public beta.** Memory stores ship under the `agent-memory-2026-07-22` beta header; the SDK sets it automatically on all `client.beta.memory_stores.*` calls. Don't add `managed-agents-2026-04-01` to these calls - sending both headers on a memory store request returns a 400. Attaching a store to a session is a session call and still uses `managed-agents-2026-04-01`. If `client.beta.memory_stores` is missing, upgrade to the latest SDK release.

> **公开测试版。** 记忆存储（memory stores）通过 `agent-memory-2026-07-22` 测试版标头提供；SDK 会在所有 `client.beta.memory_stores.*` 调用上自动设置该标头。不要在这些调用上添加 `managed-agents-2026-04-01` —— 在记忆存储请求上同时发送两个标头会返回 400。把存储挂载到会话属于会话调用，仍然使用 `managed-agents-2026-04-01`。如果 `client.beta.memory_stores` 不存在，请升级到最新版 SDK。

Sessions are ephemeral by default - when one ends, anything the agent learned is gone. A **memory store** is a workspace-scoped collection of small text documents that persists across sessions. When a store is attached to a session (via `resources[]`), it is mounted into the container as a filesystem directory; the agent reads and writes it with the ordinary file tools, and a system-prompt note tells it the mount is there.

会话默认是临时的 —— 会话结束后，智能体学到的任何内容都会消失。**记忆存储（memory store）** 是一个以工作区为作用域、可跨会话持久存在的小型文本文档集合。当存储被挂载到某个会话时（通过 `resources[]`），它会以文件系统目录的形式挂载进容器；智能体用普通文件工具读写它，同时系统提示词中会有一条说明告知它该挂载点存在。

Every mutation to a memory produces an immutable **memory version** (`memver_...`), giving you an audit trail and point-in-time rollback/redact.

对记忆的每次变更都会产生一个不可变的**记忆版本**（`memver_...`），为你提供审计轨迹以及按时间点回滚/抹除的能力。

> Warning: **Never store credentials, API keys, or tokens in memory stores.** Memories persist across sessions and are returned verbatim into future contexts - a key written once is replayed into every later session that mounts the store. Use vault `environment_variable` credentials instead (`shared/managed-agents-tools.md` -> Vaults). If a secret has already been written, delete the memory and redact the affected versions (see "Redact a version" below).

> 警告：**绝不要在记忆存储中保存凭据、API 密钥或令牌。** 记忆会跨会话持久存在，并被逐字返回到未来的上下文中 —— 一个只写入过一次的密钥会被重放到之后每一个挂载该存储的会话中。请改用保管库（vault）的 `environment_variable` 凭据（见 `shared/managed-agents-tools.md` -> Vaults）。如果密钥已被写入，请删除该记忆并抹除受影响的版本（见下文"Redact a version"）。

【评论】该警告源于记忆跨会话持久并逐字回灌未来上下文的机制：任何写入的机密都会被自动重放，属于针对数据泄漏面的一条关键防护条款。

## Object model / 对象模型

| Object | ID prefix | Scope | Notes |
| --- | --- | --- | --- |
| Memory store | `memstore_...` | Workspace | Attach to sessions via `resources[]` |
| Memory | `mem_...` | Store | One text file, addressed by `path` (<= 100KB each - prefer many small files) |
| Memory version | `memver_...` | Memory | Immutable snapshot per mutation; `operation` in `created` / `modified` / `deleted` |

| 对象 | ID 前缀 | 作用域 | 说明 |
| --- | --- | --- | --- |
| 记忆存储 | `memstore_...` | 工作区 | 通过 `resources[]` 挂载到会话 |
| 记忆 | `mem_...` | 存储内 | 一个文本文件，以 `path` 寻址（每个 ≤ 100KB —— 建议多用小文件） |
| 记忆版本 | `memver_...` | 记忆 | 每次变更对应一个不可变快照；`operation` 取值为 `created` / `modified` / `deleted` |

## Create a store / 创建存储

`description` is passed to the agent so it knows what the store contains - write it for the model, not for humans.

`description` 会传给智能体，让它知道存储里有什么 —— 请为模型而不是为人类来写这段描述。

```python
store = client.beta.memory_stores.create(
    name="User Preferences",
    description="Per-user preferences and project context.",
)
print(store.id)  # memstore_01Hx...
```

Other SDKs: TypeScript `client.beta.memoryStores.create({...})`; Go `client.Beta.MemoryStores.New(ctx, ...)`. See `shared/managed-agents-api-reference.md` -> SDK Method Reference for the full per-language table.

其他 SDK：TypeScript 使用 `client.beta.memoryStores.create({...})`；Go 使用 `client.Beta.MemoryStores.New(ctx, ...)`。完整的各语言对照表见 `shared/managed-agents-api-reference.md` -> SDK Method Reference。

Stores support `retrieve` / `update` / `list` (with `include_archived`, `created_at_{gte,lte}` filters) / `delete` / **`archive`**. Archive makes the store read-only - existing session attachments continue, new sessions cannot reference it; no unarchive.

存储支持 `retrieve` / `update` / `list`（带 `include_archived`、`created_at_{gte,lte}` 过滤器）/ `delete` / **`archive`**。归档会让存储变为只读 —— 已存在的会话挂载继续有效，新会话无法再引用它；且不支持取消归档。

### Seed with content (optional) / 预填充内容（可选）

Pre-load reference material before any session runs. `memories.create` creates a memory at the given `path`; if a memory already exists there the call returns `409` (`memory_path_conflict_error`, with the `conflicting_memory_id`). The store ID is the first positional argument.

在任何会话运行之前预加载参考资料。`memories.create` 会在给定 `path` 处创建一条记忆；如果该路径已存在记忆，调用会返回 `409`（`memory_path_conflict_error`，附带 `conflicting_memory_id`）。存储 ID 是第一个位置参数。

```python
client.beta.memory_stores.memories.create(
    store.id,
    path="/formatting_standards.md",
    content="All reports use GAAP formatting. Dates are ISO-8601...",
)
```

## Attach to a session / 挂载到会话

Memory stores go in the session's `resources[]` array alongside `file` and `github_repository` resources (see `shared/managed-agents-environments.md` -> Resources). Memory stores attach at **session create time only** - `sessions.resources.add()` does not accept `memory_store`. Sessions on **self-hosted** environments attach them the same way (and `memory_store` is the *only* resource type those environments accept) - see the self-hosted note below.

记忆存储与 `file`、`github_repository` 资源一起放入会话的 `resources[]` 数组（见 `shared/managed-agents-environments.md` -> Resources）。记忆存储**只能在创建会话时挂载** —— `sessions.resources.add()` 不接受 `memory_store`。**自托管（self-hosted）** 环境上的会话以相同方式挂载（并且 `memory_store` 是这些环境接受的*唯一*资源类型）—— 见下文自托管说明。

```python
session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=environment.id,
    resources=[
        {
            "type": "memory_store",
            "memory_store_id": store.id,
            "access": "read_write",  # or "read_only"; default is "read_write"
            "instructions": "User preferences and project context. Check before starting any task.",
        }
    ],
)
```

| Field | Required | Notes |
| --- | --- | --- |
| `type` | Yes | `"memory_store"` |
| `memory_store_id` | Yes | `memstore_...` |
| `access` | - | `"read_write"` (default) or `"read_only"` - enforced at the filesystem level on the cloud mount; on self-hosted sandboxes enforced by the worker's `write`/`edit` tools and by the upload path (see below) |
| `instructions` | - | Session-specific guidance for this store, in addition to the store's `name`/`description`. <= 4,096 chars. |

| 字段 | 是否必需 | 说明 |
| --- | --- | --- |
| `type` | 是 | `"memory_store"` |
| `memory_store_id` | 是 | `memstore_...` |
| `access` | - | `"read_write"`（默认）或 `"read_only"` —— 在云端挂载点由文件系统层强制执行；在自托管沙箱中由 worker 的 `write`/`edit` 工具及上传路径强制执行（见下文） |
| `instructions` | - | 针对该存储的会话级指引，是对存储自身 `name`/`description` 的补充。≤ 4,096 字符。 |

**Max 8 memory stores per session.** Attach multiple when different slices of memory have different owners or lifecycles - e.g. one read-only shared-reference store plus one read-write per-user store, or one store per end-user/team/project sharing a single agent config.

**每个会话最多 8 个记忆存储。** 当不同的记忆切片拥有不同的所有者或生命周期时，可以挂载多个存储 —— 例如一个只读的共享参考资料存储加一个按用户划分的可读写存储；或在共享同一智能体配置时，按最终用户/团队/项目各挂载一个存储。

### How the agent sees it (FUSE mount) / 智能体如何感知它（FUSE 挂载）

Each attached store is mounted in the session container at `/mnt/memory/<store-name>/`. The agent interacts with it using the standard file tools (`bash`, `read`, `write`, `edit`, `glob`, `grep`) - there are no dedicated memory tools. On cloud sandboxes `access: "read_only"` makes the mount read-only at the filesystem level (on self-hosted sandboxes it is enforced by the worker's `write`/`edit` tools and the upload path - see below); `"read_write"` allows the agent to create, edit, and delete files under it. A short description of each mount (name, path, `instructions`, access) is automatically injected into the system prompt so the agent knows the store exists without you having to mention it.

每个挂载的存储会以 `/mnt/memory/<store-name>/` 路径挂载到会话容器中。智能体使用标准文件工具（`bash`、`read`、`write`、`edit`、`glob`、`grep`）与之交互 —— 没有专用的记忆工具。在云端沙箱上，`access: "read_only"` 会在文件系统层面把挂载点设为只读（在自托管沙箱上则由 worker 的 `write`/`edit` 工具及上传路径强制执行 —— 见下文）；`"read_write"` 允许智能体在其下创建、编辑和删除文件。每个挂载点的一段简短描述（名称、路径、`instructions`、访问权限）会自动注入系统提示词，因此无需你提及，智能体也知道该存储存在。

Writes the agent makes under the mount are persisted back to the store and produce memory versions just like host-side `memories.update` calls.

智能体在挂载点下所做的写入会持久化回存储，并像宿主侧的 `memories.update` 调用一样产生记忆版本。

**Self-hosted sandboxes: a synced local copy, not a live mount.** On a `self_hosted` environment the SDK worker (`EnvironmentWorker` - Python, TypeScript, Go; the `ant` CLI worker does not mount stores) downloads each attached store to the same `/mnt/memory/<store-name>/` path and reconciles it with the store on an interval, so writes are visible to other sessions only after sync, conflicts resolve in favor of the store, and `read_only` is enforced by the worker's tools rather than the filesystem (`bash` can still alter the local copy). Everything else - sync interval, per-session `secret`, host prep, troubleshooting - lives in `shared/managed-agents-self-hosted-sandboxes.md` § Memory stores. Not available on self-hosted environments on Claude Platform on AWS.

**自托管沙箱：是同步的本地副本，不是实时挂载。** 在 `self_hosted` 环境上，SDK worker（`EnvironmentWorker` —— Python、TypeScript、Go；`ant` CLI worker 不挂载存储）会把每个挂载的存储下载到同样的 `/mnt/memory/<store-name>/` 路径，并按固定间隔与存储进行协调同步，因此写入只有在同步之后才对其他会话可见，冲突以存储侧为准，而 `read_only` 由 worker 的工具而非文件系统强制执行（`bash` 仍可修改本地副本）。其余内容 —— 同步间隔、每会话 `secret`、宿主机准备、故障排查 —— 见 `shared/managed-agents-self-hosted-sandboxes.md` § Memory stores。Claude Platform on AWS 上的自托管环境不支持此功能。

【评论】自托管模式下只读约束依赖 worker 工具而非文件系统，`bash` 仍可绕过修改本地副本，这是文档明示的防护边界差异，而非隐藏限制。

## Manage memories directly (host-side) / 直接管理记忆（宿主侧）

Use these for review workflows, correcting bad memories, or seeding stores out-of-band.

这些操作可用于审阅流程、纠正不良记忆，或在带外预填充存储。

### List / 列出

Returns `Memory | MemoryPrefix` entries - a `MemoryPrefix` (`type: "memory_prefix"`, just a `path`) is a directory-like node when listing hierarchically. Use `path_prefix` to scope (include a trailing slash: `"/notes/"` matches `/notes/a.md` but not `/notes_backup/old.md`) and `depth` to bound the tree walk. Pass `view="full"` to include `content` in each item; the default `"basic"` returns metadata only.

返回 `Memory | MemoryPrefix` 条目 —— `MemoryPrefix`（`type: "memory_prefix"`，仅含一个 `path`）在层级列举时表现为类似目录的节点。用 `path_prefix` 划定范围（要包含末尾斜杠：`"/notes/"` 匹配 `/notes/a.md` 但不匹配 `/notes_backup/old.md`），用 `depth` 限制树遍历深度。传 `view="full"` 可让每个条目包含 `content`；默认的 `"basic"` 只返回元数据。

```python
for m in client.beta.memory_stores.memories.list(store.id, path_prefix="/"):
    if m.type == "memory":
        print(f"{m.path}  ({m.content_size_bytes} bytes, sha={m.content_sha256[:8]})")
    else:  # "memory_prefix"
        print(f"{m.path}/")
```

### Read / 读取

```python
mem = client.beta.memory_stores.memories.retrieve(memory_id, memory_store_id=store.id)
print(mem.content)
```

`retrieve` defaults to `view="full"` (content included); `view` matters mainly on list endpoints.

`retrieve` 默认为 `view="full"`（包含内容）；`view` 主要在列举（list）端点上才有意义。

### Create vs. update / 创建与更新

| Operation | Addressed by | Semantics |
| --- | --- | --- |
| `memories.create(store_id, path=..., content=...)` | **Path** | Create at `path`. `409` (`memory_path_conflict_error`, includes `conflicting_memory_id`) if the path is already occupied. |
| `memories.update(mem_id, memory_store_id=..., path=..., content=...)` | **`mem_...` ID** | Mutate existing memory. Change `content`, `path` (rename), or both. Renaming onto an occupied path returns the same `409 memory_path_conflict_error`. |

| 操作 | 寻址方式 | 语义 |
| --- | --- | --- |
| `memories.create(store_id, path=..., content=...)` | **路径** | 在 `path` 处创建。若路径已被占用，返回 `409`（`memory_path_conflict_error`，包含 `conflicting_memory_id`）。 |
| `memories.update(mem_id, memory_store_id=..., path=..., content=...)` | **`mem_...` ID** | 变更现有记忆。可修改 `content`、`path`（重命名）或两者同时修改。重命名到已占用路径会返回同样的 `409 memory_path_conflict_error`。 |

```python
mem = client.beta.memory_stores.memories.create(
    store.id,
    path="/preferences/formatting.md",
    content="Always use tabs, not spaces.",
)

client.beta.memory_stores.memories.update(
    mem.id,
    memory_store_id=store.id,
    path="/archive/2026_q1_formatting.md",  # rename
)
```

### Optimistic concurrency (precondition on `update`) / 乐观并发控制（`update` 的前置条件）

`memories.update` accepts a `precondition` so you can read -> modify -> write back without clobbering a concurrent writer. The only supported type is `content_sha256`. On mismatch the API returns `409` (`memory_precondition_failed_error`) - re-read and retry against fresh state.

`memories.update` 接受一个 `precondition`，让你可以读取 -> 修改 -> 写回而不覆盖并发写入者的更改。目前唯一支持的类型是 `content_sha256`。不匹配时 API 返回 `409`（`memory_precondition_failed_error`）—— 请重新读取并基于最新状态重试。

```python
client.beta.memory_stores.memories.update(
    mem.id,
    memory_store_id=store.id,
    content="CORRECTED: Always use 2-space indentation.",
    precondition={"type": "content_sha256", "content_sha256": mem.content_sha256},
)
```

### Delete / 删除

```python
client.beta.memory_stores.memories.delete(mem.id, memory_store_id=store.id)
```

Pass `expected_content_sha256` for a conditional delete.

传入 `expected_content_sha256` 可执行条件删除。

## Audit and rollback - memory versions / 审计与回滚 - 记忆版本

Every mutation creates an immutable `memver_...` snapshot. Versions accumulate for the lifetime of the parent memory; `memories.retrieve` always returns the current head, the version endpoints give you history.

每次变更都会创建一个不可变的 `memver_...` 快照。版本在其父记忆的生命周期内持续累积；`memories.retrieve` 始终返回当前最新版本，而版本端点为你提供历史记录。

| Operation that triggers it | `operation` field on the version |
| --- | --- |
| `memories.create` at a new path | `"created"` |
| `memories.update` changing `content`, `path`, or both (or an agent-side write to the mount) | `"modified"` |
| `memories.delete` | `"deleted"` |

| 触发该版本的操作 | 版本上的 `operation` 字段 |
| --- | --- |
| 在新路径上执行 `memories.create` | `"created"` |
| `memories.update` 修改 `content`、`path` 或两者（或智能体侧对挂载点的写入） | `"modified"` |
| `memories.delete` | `"deleted"` |

Each version also records `created_by` - an actor object with `type` in `session_actor` / `api_actor` / `user_actor` - and, after redaction, `redacted_at` + `redacted_by`.

每个版本还会记录 `created_by` —— 一个 `type` 取值为 `session_actor` / `api_actor` / `user_actor` 的操作者对象 —— 以及在抹除之后的 `redacted_at` + `redacted_by`。

### List versions / 列出版本

Newest-first, paginated. Filter by `memory_id`, `operation`, `session_id`, `api_key_id`, or `created_at_gte` / `created_at_lte`. Pass `view="full"` to include `content`; default is metadata-only.

按最新在前排列，分页返回。可按 `memory_id`、`operation`、`session_id`、`api_key_id` 或 `created_at_gte` / `created_at_lte` 过滤。传 `view="full"` 可包含 `content`；默认仅返回元数据。

```python
for v in client.beta.memory_stores.memory_versions.list(store.id, memory_id=mem.id):
    print(f"{v.id}: {v.operation}")
```

### Retrieve a version / 获取单个版本

```python
version = client.beta.memory_stores.memory_versions.retrieve(
    version_id, memory_store_id=store.id
)
print(version.content)
```

### Redact a version / 抹除某个版本

Scrubs content from a historical version while preserving the audit trail (actor + timestamps). Clears `content`, `content_sha256`, `content_size_bytes`, and `path`; everything else stays. Use for leaked secrets, PII, or user-deletion requests.

从历史版本中清除内容，同时保留审计轨迹（操作者 + 时间戳）。会清空 `content`、`content_sha256`、`content_size_bytes` 和 `path`；其余字段保持不变。适用于泄漏的机密、PII（个人身份信息）或用户删除请求。

```python
client.beta.memory_stores.memory_versions.redact(version_id, memory_store_id=store.id)
```

## Endpoint reference / 端点参考

See `shared/managed-agents-api-reference.md` -> Memory Stores / Memories / Memory Versions for the full HTTP method/path tables. Raw HTTP base path:

完整的 HTTP 方法/路径表见 `shared/managed-agents-api-reference.md` -> Memory Stores / Memories / Memory Versions。原始 HTTP 基础路径：

```
POST   /v1/memory_stores
POST   /v1/memory_stores/{memory_store_id}/archive
GET    /v1/memory_stores/{memory_store_id}/memories
PATCH  /v1/memory_stores/{memory_store_id}/memories/{memory_id}
GET    /v1/memory_stores/{memory_store_id}/memory_versions
POST   /v1/memory_stores/{memory_store_id}/memory_versions/{version_id}/redact
```

For cURL examples and the CLI (`ant beta:memory-stores ...`), WebFetch the Memory URL in `shared/live-sources.md` -> Managed Agents.

如需 cURL 示例和 CLI（`ant beta:memory-stores ...`），请通过 WebFetch 访问 `shared/live-sources.md` -> Managed Agents 中的 Memory URL。
