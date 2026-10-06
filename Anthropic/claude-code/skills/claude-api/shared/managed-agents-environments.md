<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Environments & Resources / 托管代理 - 环境与资源

## Environments / 环境

Creating a session requires an `environment_id`. Environments are **reusable configuration templates** for spinning up containers in Anthropic's infrastructure - you might create different environments for different use cases (e.g. data visualization vs web development, with different package sets). Anthropic handles scaling, container lifecycle, and work orchestration.

创建会话需要 `environment_id`。环境是用于在 Anthropic 基础设施中启动容器的**可复用配置模板**——你可以为不同用例创建不同的环境（例如数据可视化与 Web 开发分别使用不同的软件包集合）。扩缩容、容器生命周期与工作编排均由 Anthropic 负责。

**Environment names must be unique.** Creating an environment with an existing name returns 409.

**环境名称必须唯一。** 使用已存在的名称创建环境会返回 409。

### Networking / 网络

| Network Policy   | Description                                                   |
| ---------------- | ------------------------------------------------------------- |
| `unrestricted`   | Full egress (except legal blocklist)                          |
| `limited`        | Deny-by-default; opt in via `allowed_hosts` / `allow_package_managers` / `allow_mcp_servers` |

| 网络策略 | 描述 |
| ---------------- | ------------------------------------------------------------- |
| `unrestricted` | 完全出站（法律封锁名单除外） |
| `limited` | 默认拒绝；通过 `allowed_hosts` / `allow_package_managers` / `allow_mcp_servers` 选择性放行 |

```json
{
  "networking": {
    "type": "limited",
    "allow_package_managers": true,
    "allow_mcp_servers": true,
    "allowed_hosts": ["api.example.com"]
  }
}
```

All three `limited` fields are optional. `allow_package_managers` (default `false`) permits PyPI/npm/etc.; `allow_mcp_servers` (default `false`) permits the agent's configured MCP server endpoints without listing them in `allowed_hosts`.

这三个 `limited` 相关字段均为可选。`allow_package_managers`（默认 `false`）允许访问 PyPI/npm 等；`allow_mcp_servers`（默认 `false`）允许代理已配置的 MCP 服务器端点，无需在 `allowed_hosts` 中逐一列出。

**MCP caveat:** Under `limited` networking, either set `allow_mcp_servers: true` or add each MCP server domain to `allowed_hosts`. Otherwise the container can't reach them and tools silently fail.

**MCP 注意事项：** 在 `limited` 网络下，要么设置 `allow_mcp_servers: true`，要么把每个 MCP 服务器域名加入 `allowed_hosts`。否则容器无法访问它们，工具会静默失败。

**Packages caveat:** Under `limited` networking, `packages` requires `allow_package_managers: true`; otherwise the request fails with a 400. Listing the registry in `allowed_hosts` is not enough.

**软件包注意事项：** 在 `limited` 网络下，使用 `packages` 需要 `allow_package_managers: true`；否则请求会以 400 失败。仅在 `allowed_hosts` 中列出镜像源并不够。

**`networking` does not govern `web_search` / `web_fetch`.** Those tools run on Anthropic's servers (in cloud *and* self-hosted environments), so `limited` egress and `allowed_hosts` don't restrict them. To restrict the sites they can reach, set `allowed_domains` / `blocked_domains` on the tool's `configs` entry in the agent toolset - see `shared/managed-agents-tools.md` § Web search & web fetch settings.

**`networking` 不约束 `web_search` / `web_fetch`。** 这些工具运行在 Anthropic 的服务器上（云端*和*自托管环境均是如此），因此 `limited` 出站策略与 `allowed_hosts` 对它们不起限制作用。要限制它们可访问的站点，需在 agent toolset 中该工具的 `configs` 条目上设置 `allowed_domains` / `blocked_domains`——参见 `shared/managed-agents-tools.md` § Web search & web fetch settings。

【评论】值得注意的是，容器级网络策略只覆盖沙箱出站流量，`web_search` / `web_fetch` 因运行在 Anthropic 侧服务器上而处于该策略之外，站点白名单需改用工具级 `allowed_domains` / `blocked_domains` 控制，两者容易混淆。

### Creating an environment / 创建环境

The SDK adds `managed-agents-2026-04-01` automatically. TypeScript:

SDK 会自动添加 `managed-agents-2026-04-01`。TypeScript 示例：

```ts
const env = await client.beta.environments.create({
  name: "my_env",
  config: {
    type: "cloud",
    networking: { type: "unrestricted" },
  },
});
```

### Self-hosted sandboxes / 自托管沙箱

To run tool execution in **your own infrastructure** instead of Anthropic's, set `config: {type: "self_hosted"}` - the agent loop stays on Anthropic's side, but `bash` / file ops / code execute in a container you control via an outbound-polling worker. The `networking` block does not apply (you control egress). Resource mounting (`file`, `github_repository`) and memory stores behave differently - see `shared/managed-agents-self-hosted-sandboxes.md` for the worker, credentials, and cloud-vs-self-hosted comparison.

要在**你自己的基础设施**而非 Anthropic 的设施中运行工具执行，设置 `config: {type: "self_hosted"}`——代理循环仍留在 Anthropic 侧，但 `bash` / 文件操作 / 代码在由你控制、通过出站轮询 worker 连接的容器中执行。`networking` 块不适用（出站由你控制）。资源挂载（`file`、`github_repository`）与记忆存储的行为有所不同——worker、凭据及云端与自托管的对比参见 `shared/managed-agents-self-hosted-sandboxes.md`。

### Environment CRUD / 环境的增删改查

| Operation        | Method   | Path                                       | Notes |
| ---------------- | -------- | ------------------------------------------ | ----- |
| Create           | `POST`   | `/v1/environments`                         | |
| List             | `GET`    | `/v1/environments`                         | Paginated (`limit`, `after_id`, `before_id`) |
| Get              | `GET`    | `/v1/environments/{id}`                    | |
| Update           | `POST`   | `/v1/environments/{id}`                    | Changes apply only to **new** containers; existing sessions keep their original config |
| Delete           | `DELETE` | `/v1/environments/{id}`                    | Returns 204. |
| Archive          | `POST`   | `/v1/environments/{id}/archive`            | Makes it **read-only**; existing sessions continue, new sessions cannot reference it. No unarchive - terminal state. |

| 操作 | 方法 | 路径 | 说明 |
| ---------------- | -------- | ------------------------------------------ | ----- |
| 创建 | `POST` | `/v1/environments` | |
| 列表 | `GET` | `/v1/environments` | 分页（`limit`、`after_id`、`before_id`） |
| 获取 | `GET` | `/v1/environments/{id}` | |
| 更新 | `POST` | `/v1/environments/{id}` | 变更只作用于**新建**容器；已有会话保留其原始配置 |
| 删除 | `DELETE` | `/v1/environments/{id}` | 返回 204。 |
| 归档 | `POST` | `/v1/environments/{id}/archive` | 使其变为**只读**；已有会话继续运行，新会话无法引用它。无法取消归档——终态。 |

---

## Resources / 资源

Attach files, GitHub repositories, and memory stores to a session. Resources are resolved during session creation, so a bad `file_id` or an unreachable repo surfaces on the create call rather than mid-run. Creating a session does **not** by itself start work or provision the sandbox - without `initial_events` the session is only registered, and the sandbox comes up when the session first needs it (see `shared/managed-agents-core.md` -> Seeding a session with `initial_events`). Max **999 file resources** per session. Multiple GitHub repositories per session are supported. For `type: "memory_store"` resources (persistent cross-session memory - max 8 per session), see `shared/managed-agents-memory.md`.

将文件、GitHub 仓库和记忆存储挂载到会话。资源在会话创建时解析，因此错误的 `file_id` 或不可达的仓库会在创建调用时即暴露，而不是运行中途才报错。创建会话本身**不会**启动工作或预配沙箱——没有 `initial_events` 时会话只是被注册，沙箱在会话首次需要时才会拉起（参见 `shared/managed-agents-core.md` -> Seeding a session with `initial_events`）。每个会话最多 **999 个文件资源**。每个会话支持挂载多个 GitHub 仓库。关于 `type: "memory_store"` 资源（跨会话的持久记忆——每个会话最多 8 个），参见 `shared/managed-agents-memory.md`。

### File Uploads (input - host -> agent) / 文件上传（输入：宿主机 -> 代理）

Upload a file first via the Files API, then reference by `file_id` + `mount_path`:

先通过 Files API 上传文件，再通过 `file_id` + `mount_path` 引用：

```ts
// 1. Upload
const file = await client.beta.files.upload({
  file: fs.createReadStream("data.csv"),
  purpose: "agent",
});

// 2. Attach as a session resource
const session = await client.beta.sessions.create({
  agent: agent.id,
  environment_id: envId,
  resources: [
    { type: "file", file_id: file.id, mount_path: "/workspace/data.csv" }
  ],
});
```

**`mount_path` is required** and must be absolute. Parent directories are created automatically. Agent working directory defaults to `/workspace`. Files are mounted read-only - the agent writes modified versions to new paths.

**`mount_path` 为必填**且必须是绝对路径。父目录会自动创建。代理工作目录默认为 `/workspace`。文件以只读方式挂载——代理把修改后的版本写到新路径。

### Session outputs (output - agent -> host) / 会话输出（输出：代理 -> 宿主机）

The agent can write files to `/mnt/session/outputs/` during a session. These are automatically captured by the Files API and can be listed and downloaded afterwards:

代理可以在会话期间把文件写入 `/mnt/session/outputs/`。这些文件由 Files API 自动捕获，之后可以列出并下载：

```ts
// After the turn completes, list output files scoped to this session:
for await (const f of client.beta.files.list({
  scope_id: session.id,
  betas: ["managed-agents-2026-04-01"],
})) {
  console.log(f.filename, f.size_bytes);
  const resp = await client.beta.files.download(f.id);
  const text = await resp.text();
}
```

**Requirements:**

**要求：**

- The `write` tool (or `bash`) must be enabled for the agent to create output files.
  代理必须启用 `write` 工具（或 `bash`）才能创建输出文件。
- Session-scoped `files.list` / `files.download` captures outputs written to `/mnt/session/outputs/`.
  会话级 `files.list` / `files.download` 捕获写入 `/mnt/session/outputs/` 的输出。
- The filter parameter is **`scope_id`** (REST query param `?scope_id=<session_id>`). Filtering by `scope_id` requires the `managed-agents-2026-04-01` header, which `client.beta.files` does not add, so pass `betas: ["managed-agents-2026-04-01"]` explicitly (on raw HTTP, send `anthropic-beta: managed-agents-2026-04-01`); the list call uses the `beta` files namespace only to pass that header, and upload and download also work on `client.files`. Requires `@anthropic-ai/sdk` >= 0.88.0 / `anthropic` (Python) >= 0.92.0 - older versions don't type `scope_id`. In the `ant` CLI, use `ant beta:files list --scope-id <session_id> --beta managed-agents-2026-04-01`.
  过滤参数是 **`scope_id`**（REST 查询参数 `?scope_id=<session_id>`）。按 `scope_id` 过滤需要 `managed-agents-2026-04-01` 请求头，而 `client.beta.files` 不会自动添加该头，因此需显式传入 `betas: ["managed-agents-2026-04-01"]`（使用原始 HTTP 时发送 `anthropic-beta: managed-agents-2026-04-01`）；列表调用使用 `beta` files 命名空间只是为了带上该请求头，上传与下载在 `client.files` 上同样可用。需要 `@anthropic-ai/sdk` >= 0.88.0 / Python 的 `anthropic` >= 0.92.0——更早的版本没有 `scope_id` 的类型定义。在 `ant` CLI 中使用 `ant beta:files list --scope-id <session_id> --beta managed-agents-2026-04-01`。
- Pass the session ID returned by `sessions.create()` verbatim (e.g. `sesn_011CZx...`) - the API validates the prefix.
  原样传入 `sessions.create()` 返回的会话 ID（例如 `sesn_011CZx...`）——API 会校验前缀。
- There's a brief indexing lag (~1-3s) between `session.status_idle` and output files appearing in `files.list`. Retry once or twice if empty.
  从 `session.status_idle` 到输出文件出现在 `files.list` 之间存在短暂的索引延迟（约 1-3 秒）。若列表为空可重试一两次。

> **Fallback when `scope_id` filtering is unavailable** (older SDK, or endpoint returns an error): send a follow-up `user.message` asking the agent to `read` each file under `/mnt/session/outputs/` and return the contents. The agent streams the file bodies back as `agent.message` text. This works for text files only and costs output tokens - use it to unblock, not as the primary path.

> **当 `scope_id` 过滤不可用时**（SDK 较旧，或端点返回错误的兜底方案）：追加一条 `user.message`，要求代理 `read` `/mnt/session/outputs/` 下的每个文件并返回内容。代理会以 `agent.message` 文本的形式流式返回文件正文。该方式仅适用于文本文件，且会消耗输出 token——可作为应急解堵手段，不宜作为主路径。

This gives you a bidirectional file bridge: upload reference data in, download agent artifacts out.

由此形成一条双向文件通道：向内上传参考数据，向外取回代理产物。

### GitHub Repositories / GitHub 仓库

Clones a GitHub repository into the session container during initialization, before the agent begins execution. The agent can read, edit, commit, and push via `bash` (`git`). Multiple repositories per session are supported - add one `resources` entry per repo. Repositories are cached, so future sessions that use the same repository start faster.

在初始化阶段（代理开始执行之前）把 GitHub 仓库克隆进会话容器。代理可以通过 `bash`（`git`）读取、编辑、提交和推送。每个会话支持多个仓库——每个仓库添加一个 `resources` 条目。仓库会被缓存，后续使用同一仓库的会话启动更快。

Mounting a repository also loads any skills stored in its root `.claude/skills` directory - discovered once per session, from the repository state checked out at session start (cloud sandboxes only). See `shared/managed-agents-tools.md` -> Skills from a GitHub repository.

挂载仓库还会加载其根目录 `.claude/skills` 下存放的技能——每个会话发现一次，来源是会话启动时检出的仓库状态（仅限云端沙箱）。参见 `shared/managed-agents-tools.md` -> Skills from a GitHub repository。

Repositories are attached for the lifetime of the session - to change which repositories are mounted, create a new session. You **can** rotate a repository's `authorization_token` on a running session via `client.beta.sessions.resources.update(resource_id, {session_id, authorization_token})`; the resource `id` is returned at session creation and by `resources.list()`.

仓库在整个会话生命周期内保持挂载——要更换挂载的仓库，需创建新会话。不过你**可以**在运行中的会话上轮换仓库的 `authorization_token`：通过 `client.beta.sessions.resources.update(resource_id, {session_id, authorization_token})`；资源 `id` 在会话创建时以及 `resources.list()` 中返回。

**Fields:**

**字段：**

| Field | Required | Notes |
|---|---|---|
| `type` | Yes | `"github_repository"` |
| `url` | Yes | The GitHub repository URL |
| `authorization_token` | Yes | GitHub Personal Access Token with repository access. **Never echoed in API responses.** |
| `mount_path` | No | Path where the repository will be cloned. Defaults to `/workspace/<repo-name>`. |
| `checkout` | No | `{type: "branch", name: "..."}` or `{type: "commit", sha: "..."}`. Defaults to the repo's default branch. |

| 字段 | 是否必填 | 说明 |
|---|---|---|
| `type` | 是 | `"github_repository"` |
| `url` | 是 | GitHub 仓库 URL |
| `authorization_token` | 是 | 具有仓库访问权限的 GitHub 个人访问令牌（PAT）。**绝不会在 API 响应中回显。** |
| `mount_path` | 否 | 仓库的克隆路径。默认为 `/workspace/<repo-name>`。 |
| `checkout` | 否 | `{type: "branch", name: "..."}` 或 `{type: "commit", sha: "..."}`。默认为仓库的默认分支。 |

**Token permission levels** (fine-grained PATs):

**令牌权限级别**（细粒度 PAT）：

- `Contents: Read` - clone only
  `Contents: Read`——仅克隆
- `Contents: Read and write` - push changes and create pull requests
  `Contents: Read and write`——推送更改并创建拉取请求

**How auth works:** `authorization_token` is never placed inside the container. `git pull` / `git push` and GitHub REST calls against the attached repository are routed through an Anthropic-side git proxy that injects the token after the request leaves the sandbox. Code running in the container - including anything the agent writes - cannot read or exfiltrate it.

**鉴权原理：** `authorization_token` 绝不会被放进容器内部。针对所挂载仓库的 `git pull` / `git push` 与 GitHub REST 调用经由 Anthropic 侧的 git 代理路由，该代理在请求离开沙箱之后才注入令牌。容器内运行的代码——包括代理写出的任何代码——都无法读取或外泄该令牌。

【评论】令牌不进入容器、由 Anthropic 侧 git 代理在出站后注入，属于典型的凭据隔离设计，可防止沙箱内代码（包括模型生成代码）窃取或外泄令牌，与 API 响应不回显令牌的措施互为补充。

> Important: **To generate pull requests** you also need GitHub **MCP server** access - the `github_repository` resource gives filesystem + git access only. See `shared/managed-agents-tools.md` -> MCP Servers. The PR workflow is: edit files in the mounted repo -> push branch via `bash` (authenticated via the git proxy using `authorization_token`) -> create PR via the MCP `create_pull_request` tool (authenticated via the vault).

> 重要：**要创建拉取请求**，你还需要 GitHub **MCP 服务器**访问权限——`github_repository` 资源只提供文件系统 + git 访问。参见 `shared/managed-agents-tools.md` -> MCP Servers。PR 工作流为：在挂载的仓库中编辑文件 -> 通过 `bash` 推送分支（经使用 `authorization_token` 的 git 代理鉴权）-> 通过 MCP `create_pull_request` 工具创建 PR（经 vault 鉴权）。

**TypeScript:**

**TypeScript：**

```ts
// 1. Create the agent - declare GitHub MCP (no auth here)
const agent = await client.beta.agents.create(
  {
    name: 'GitHub Agent',
    model: 'claude-opus-5-5',
    mcp_servers: [
      { type: 'url', name: 'github', url: 'https://api.githubcopilot.com/mcp/' },
    ],
    tools: [
      { type: 'agent_toolset_20260401', default_config: { enabled: true } },
      { type: 'mcp_toolset', mcp_server_name: 'github' },
    ],
  },
);

// 2. Start a session - attach vault for MCP auth + mount the repo
const session = await client.beta.sessions.create({
  agent: agent.id,
  environment_id: envId,
  vault_ids: [vaultId],  // vault contains the GitHub MCP OAuth credential
  resources: [
    {
      type: 'github_repository',
      url: 'https://github.com/owner/repo',
      authorization_token: process.env.GITHUB_TOKEN,  // repo clone token (!= MCP auth)
      checkout: { type: 'branch', name: 'main' },
    },
  ],
});
```

**Python:**

**Python：**

```python
import os

agent = client.beta.agents.create(
    name="GitHub Agent",
    model="claude-opus-5-5",
    mcp_servers=[{
        "type": "url",
        "name": "github",
        "url": "https://api.githubcopilot.com/mcp/",
    }],
    tools=[
        {"type": "agent_toolset_20260401", "default_config": {"enabled": True}},
        {"type": "mcp_toolset", "mcp_server_name": "github"},
    ],
)

session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=env_id,
    vault_ids=[vault_id],  # vault contains the GitHub MCP OAuth credential
    resources=[{
        "type": "github_repository",
        "url": "https://github.com/owner/repo",
        "authorization_token": os.environ["GITHUB_TOKEN"],  # repo clone token (!= MCP auth)
        "checkout": {"type": "branch", "name": "main"},
    }],
)
```

---

## Files API / Files API（文件 API）

Upload and manage files for use as session resources, and download files the agent wrote to `/mnt/session/outputs/`.

上传和管理用作会话资源的文件，并下载代理写入 `/mnt/session/outputs/` 的文件。

| Operation        | Method   | Path                                  | SDK |
| ---------------- | -------- | ------------------------------------- | --- |
| Upload           | `POST`   | `/v1/files`                           | `client.beta.files.upload({ file })` |
| List             | `GET`    | `/v1/files?scope_id=...`              | `client.beta.files.list({ scope_id, betas: ["managed-agents-2026-04-01"] })` |
| Get Metadata     | `GET`    | `/v1/files/{id}`                      | `client.beta.files.retrieveMetadata(id)` |
| Download         | `GET`    | `/v1/files/{id}/content`              | `client.beta.files.download(id)` -> `Response` |
| Delete           | `DELETE` | `/v1/files/{id}`                      | `client.beta.files.delete(id)` |

| 操作 | 方法 | 路径 | SDK |
| ---------------- | -------- | ------------------------------------- | --- |
| 上传 | `POST` | `/v1/files` | `client.beta.files.upload({ file })` |
| 列表 | `GET` | `/v1/files?scope_id=...` | `client.beta.files.list({ scope_id, betas: ["managed-agents-2026-04-01"] })` |
| 获取元数据 | `GET` | `/v1/files/{id}` | `client.beta.files.retrieveMetadata(id)` |
| 下载 | `GET` | `/v1/files/{id}/content` | `client.beta.files.download(id)` -> `Response` |
| 删除 | `DELETE` | `/v1/files/{id}` | `client.beta.files.delete(id)` |

The `scope_id` filter on List scopes the results to files written to `/mnt/session/outputs/` by that session. Without the filter, you get all files uploaded to your account.

List 的 `scope_id` 过滤器把结果限定为该会话写入 `/mnt/session/outputs/` 的文件。不使用该过滤器时，返回的是上传到你自己账户的全部文件。
