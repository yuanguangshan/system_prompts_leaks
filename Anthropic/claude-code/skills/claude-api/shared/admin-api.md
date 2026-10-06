<!-- BILINGUAL-EN-ZH -->
# Admin API (Organization Management) / 管理 API（组织管理）

Read this file when the user wants to manage their Anthropic organization programmatically: members and roles, invites, workspaces and workspace members, API keys, rate limit reports, service accounts, workload identity federation (WIF), or customer-managed encryption keys (CMEK).

当用户想要以编程方式管理其 Anthropic 组织时阅读本文件：成员与角色、邀请、工作区与工作区成员、API 密钥、速率限制报告、服务账户、工作负载身份联合（WIF），或客户管理的加密密钥（CMEK）。

The Admin API lives under `https://api.anthropic.com/v1/organizations/*`. It manages the organization itself - it does not send messages. As of **August 26, 2026** it is available in all seven SDKs (Python, TypeScript, C#, Go, Java, PHP, Ruby) under `client.beta.organization`, and in the `ant` CLI under `ant beta:organization`. Usage reports, cost reports, and the Claude Enterprise user-management and analytics endpoints are **not** in the SDKs - call those with raw HTTP.

管理 API 位于 `https://api.anthropic.com/v1/organizations/*` 之下。它管理组织本身 —— 不用于发送消息。自 **2026 年 8 月 26 日**起，它在全部七个 SDK（Python、TypeScript、C#、Go、Java、PHP、Ruby）中以 `client.beta.organization` 提供，在 `ant` CLI 中以 `ant beta:organization` 提供。用量报告、成本报告以及 Claude Enterprise 的用户管理和分析端点**不在** SDK 中 —— 这些需要用原生 HTTP 调用。

## Authentication / 身份验证

Two credential types, both read automatically by the default SDK client and the CLI:

两种凭据类型，默认的 SDK 客户端和 CLI 都会自动读取它们：

| Credential | Env var | HTTP header | Covers |
| --- | --- | --- | --- |
| Admin API key (`sk-ant-admin...`) | `ANTHROPIC_API_KEY` | `x-api-key` | Most endpoints |
| `org:admin` OAuth token | `ANTHROPIC_AUTH_TOKEN` | `authorization: Bearer` | Everything, including the OAuth-only endpoints |

| 凭据 | 环境变量 | HTTP 标头 | 覆盖范围 |
| --- | --- | --- | --- |
| 管理 API 密钥（`sk-ant-admin...`） | `ANTHROPIC_API_KEY` | `x-api-key` | 大多数端点 |
| `org:admin` OAuth 令牌 | `ANTHROPIC_AUTH_TOKEN` | `authorization: Bearer` | 全部端点，包括仅支持 OAuth 的端点 |

- **OAuth-only endpoints:** service accounts, federation issuers, and federation rules reject API keys - they require an `org:admin` OAuth token.

- **仅支持 OAuth 的端点：** 服务账户、联合身份签发方（federation issuers）和联合规则拒绝 API 密钥 —— 它们要求 `org:admin` OAuth 令牌。

- **Precedence gotcha:** when both env vars are set, some clients prefer the API key. When using a bearer token, leave `ANTHROPIC_API_KEY` unset in that shell.

- **优先级陷阱：** 当两个环境变量同时设置时，某些客户端会优先使用 API 密钥。使用 Bearer 令牌时，请在该 shell 中保持 `ANTHROPIC_API_KEY` 未设置。

- Admin API keys are created in the Claude Console by organization admins.

- 管理 API 密钥由组织管理员在 Claude Console 中创建。

- Regular (non-admin) API keys do not work on any of these endpoints, and admin credentials do not work on the Messages API.

- 普通（非管理）API 密钥在这些端点上均不可用，而管理凭据也不能用于 Messages API。

- An `org:admin` token grants access to the whole organization regardless of any workspace binding.

- `org:admin` 令牌可访问整个组织，不受任何工作区绑定限制。

**Interactive OAuth token** - log in with the `ant` CLI under a dedicated profile (keeps routine commands from running with elevated access), then export the token. Tokens are short-lived; on 401, re-run the export. Profile and scope mechanics (why `org:admin` needs an explicit `--scope`, switching profiles): `shared/anthropic-cli.md`.

**交互式 OAuth 令牌** —— 在 `ant` CLI 中使用专用配置档（profile）登录（避免日常命令以提升权限运行），然后导出令牌。令牌有效期较短；遇到 401 时，重新执行导出。配置档与作用域机制（为何 `org:admin` 需要显式 `--scope`、切换配置档）：见 `shared/anthropic-cli.md`。

```bash
ant auth login --profile admin --scope "org:admin"
export ANTHROPIC_AUTH_TOKEN=$(ant auth print-credentials --profile admin --access-token)
# When done: unset ANTHROPIC_AUTH_TOKEN && ant profile activate default
```

**Automated workloads (CI)** - don't log in interactively. Create a federation rule with `oauth_scope: org:admin` targeting a service account whose `organization_role` is `admin` (this one rule must be created by a human in the Claude Console), then point the client at it with the federation env vars and construct it with no arguments - the SDK/CLI performs the token exchange automatically and refreshes before expiry:

**自动化工作负载（CI）** —— 不要以交互方式登录。创建一条 `oauth_scope: org:admin` 的联合规则，目标是一个 `organization_role` 为 `admin` 的服务账户（这一条规则必须由人工在 Claude Console 中创建），然后用联合身份环境变量把客户端指向它并无参构造 —— SDK/CLI 会自动执行令牌交换并在过期前刷新：

```bash
export ANTHROPIC_FEDERATION_RULE_ID=fdrl_...       # the org:admin rule
export ANTHROPIC_ORGANIZATION_ID=<org-uuid>
export ANTHROPIC_SERVICE_ACCOUNT_ID=svac_...       # the rule's target service account
export ANTHROPIC_IDENTITY_TOKEN_FILE=/path/to/jwt  # or ANTHROPIC_IDENTITY_TOKEN
```

**curl** also needs `anthropic-version: 2023-06-01` on every request.

**curl** 还需要在每个请求上携带 `anthropic-version: 2023-06-01`。

## Endpoint Coverage / 端点覆盖

SDK accessor shown in Python spelling; see the per-language table below for naming conventions.

SDK 访问器以 Python 拼写展示；各语言命名约定见下表。

| Resource | REST path | SDK accessor (`client.beta.organization` +) | CLI (`ant beta:organization` +) |
| --- | --- | --- | --- |
| Organization info | `GET /v1/organizations/me` | `.retrieve()` | `retrieve` |
| Members | `/v1/organizations/users` | `.users` - `list`, `update`, `remove` | `:users list\|update\|remove` |
| Invites | `/v1/organizations/invites` | `.invites` - `create`, `list`, `delete` | `:invites create\|list\|delete` |
| Workspaces | `/v1/organizations/workspaces` | `.workspaces` - `create`, `retrieve`, `list`, `update`, `archive` | `:workspaces create\|list\|update\|archive` |
| Workspace members | `/v1/organizations/workspaces/{id}/members` | `.workspaces.members` - `add`, `list`, `update`, `remove` | `:workspaces:members add\|list\|update\|remove` |
| API keys | `/v1/organizations/api_keys` | `.api_keys` - `list`, `update` | `:api-keys list\|update` |
| Org rate limits | `GET /v1/organizations/rate_limits` | `.rate_limits.list(model=..., group_type=...)` | `:rate-limits list` |
| Workspace rate limits | `GET /v1/organizations/workspaces/{id}/rate_limits` | `.workspaces.rate_limits.list(workspace_id)` | `:workspaces:rate-limits list` |
| Service accounts (*) | `/v1/organizations/service_accounts` | `.service_accounts` - `create`, `list`, `archive` | `:service-accounts create\|list\|archive` |
| Federation issuers (*) | `/v1/organizations/federation_issuers` | `.federation.issuers` - `create`, `list`, `archive` | `:federation:issuers create\|list\|archive` |
| Federation rules (*) | `/v1/organizations/federation_rules` | `.federation.rules` - `create`, `list`, `archive` | `:federation:rules create\|list\|archive` |
| CMEK external keys | `/v1/organizations/external_keys` | `.external_keys` - `create`, `validate` | - |

| 资源 | REST 路径 | SDK 访问器（`client.beta.organization` 之后） | CLI（`ant beta:organization` 之后） |
| --- | --- | --- | --- |
| 组织信息 | `GET /v1/organizations/me` | `.retrieve()` | `retrieve` |
| 成员 | `/v1/organizations/users` | `.users` - `list`、`update`、`remove` | `:users list\|update\|remove` |
| 邀请 | `/v1/organizations/invites` | `.invites` - `create`、`list`、`delete` | `:invites create\|list\|delete` |
| 工作区 | `/v1/organizations/workspaces` | `.workspaces` - `create`、`retrieve`、`list`、`update`、`archive` | `:workspaces create\|list\|update\|archive` |
| 工作区成员 | `/v1/organizations/workspaces/{id}/members` | `.workspaces.members` - `add`、`list`、`update`、`remove` | `:workspaces:members add\|list\|update\|remove` |
| API 密钥 | `/v1/organizations/api_keys` | `.api_keys` - `list`、`update` | `:api-keys list\|update` |
| 组织速率限制 | `GET /v1/organizations/rate_limits` | `.rate_limits.list(model=..., group_type=...)` | `:rate-limits list` |
| 工作区速率限制 | `GET /v1/organizations/workspaces/{id}/rate_limits` | `.workspaces.rate_limits.list(workspace_id)` | `:workspaces:rate-limits list` |
| 服务账户（*） | `/v1/organizations/service_accounts` | `.service_accounts` - `create`、`list`、`archive` | `:service-accounts create\|list\|archive` |
| 联合身份签发方（*） | `/v1/organizations/federation_issuers` | `.federation.issuers` - `create`、`list`、`archive` | `:federation:issuers create\|list\|archive` |
| 联合规则（*） | `/v1/organizations/federation_rules` | `.federation.rules` - `create`、`list`、`archive` | `:federation:rules create\|list\|archive` |
| CMEK 外部密钥 | `/v1/organizations/external_keys` | `.external_keys` - `create`、`validate` | - |

(*) OAuth-only: requires an `org:admin` bearer token, not an API key.

（*）仅支持 OAuth：需要 `org:admin` Bearer 令牌，不能使用 API 密钥。

Attaching a CMEK external key to a workspace is a workspace update: `client.beta.organization.workspaces.update("<workspace-id>", external_key_id="ekey_...")`.

把 CMEK 外部密钥关联到工作区是一次工作区更新操作：`client.beta.organization.workspaces.update("<workspace-id>", external_key_id="ekey_...")`。

## Per-Language Naming & Pagination / 各语言命名与分页

| Language | Accessor style (list members example) | List behavior |
| --- | --- | --- |
| Python | `client.beta.organization.users.list(limit=10)` | Iterator auto-fetches more pages; `limit` = page size, not total |
| TypeScript | `client.beta.organization.users.list({ limit: 10 })` - camelCase sub-resources: `apiKeys`, `rateLimits`, `serviceAccounts`, `externalKeys` | `for await` auto-pages |
| C# | `client.Beta.Organization.Users.List(new() { Limit = 10 })` | `await foreach (var u in page.Paginate())` auto-pages |
| Go | `client.Beta.Organization.Users.ListAutoPaging(ctx, params)`; org info is `Organization.Get(ctx)` | `.Next()` / `.Current()` auto-pages |
| Java | `client.beta().organization().users().list(params)` with builder params (`UserListParams.builder().limit(10).build()`) | `.autoPager()` auto-pages |
| PHP | `$client->beta->organization->users->list(limit: 10)` | Raw single-page data call - iterate `->getItems()`; the SDK's auto-pagination helpers aren't wired up for these endpoints yet |
| Ruby | `client.beta.organization.users.list(limit: 10)` | Raw single-page data call - iterate `.data`; the SDK's auto-pagination helpers aren't wired up for these endpoints yet |
| CLI | `ant beta:organization:users list --limit 10` | On the member, invite, workspace, workspace-member, and API-key lists, `--limit` caps the results (unlike most `ant` list commands, where `--limit` sets the page size and `--max-items` caps - see `shared/anthropic-cli.md`) |
| curl | `GET /v1/organizations/users?limit=10` | One page per request; cursor pagination per the Admin API reference |

| 语言 | 访问器风格（列出成员示例） | 列举行为 |
| --- | --- | --- |
| Python | `client.beta.organization.users.list(limit=10)` | 迭代器自动抓取更多页；`limit` 是页大小，不是总量 |
| TypeScript | `client.beta.organization.users.list({ limit: 10 })` - 子资源用驼峰命名：`apiKeys`、`rateLimits`、`serviceAccounts`、`externalKeys` | `for await` 自动翻页 |
| C# | `client.Beta.Organization.Users.List(new() { Limit = 10 })` | `await foreach (var u in page.Paginate())` 自动翻页 |
| Go | `client.Beta.Organization.Users.ListAutoPaging(ctx, params)`；组织信息用 `Organization.Get(ctx)` | `.Next()` / `.Current()` 自动翻页 |
| Java | `client.beta().organization().users().list(params)`，参数用构建器（`UserListParams.builder().limit(10).build()`） | `.autoPager()` 自动翻页 |
| PHP | `$client->beta->organization->users->list(limit: 10)` | 原生单页数据调用 —— 遍历 `->getItems()`；SDK 的自动分页辅助工具尚未接入这些端点 |
| Ruby | `client.beta.organization.users.list(limit: 10)` | 原生单页数据调用 —— 遍历 `.data`；SDK 的自动分页辅助工具尚未接入这些端点 |
| CLI | `ant beta:organization:users list --limit 10` | 对成员、邀请、工作区、工作区成员和 API 密钥列表，`--limit` 限制的是结果总数（与大多数 `ant` list 命令不同 —— 那里 `--limit` 设页大小、`--max-items` 限总量，见 `shared/anthropic-cli.md`） |
| curl | `GET /v1/organizations/users?limit=10` | 每次请求一页；按管理 API 参考进行游标分页 |

The rate-limit lists (`rate_limits`, `workspaces.rate_limits`) also support pagination as of launch - page them like the other list endpoints rather than assuming a single response.

速率限制列表（`rate_limits`、`workspaces.rate_limits`）自发布起同样支持分页 —— 请像其他列表端点一样分页获取，不要假定只有单条响应。

Go param types follow the pattern `anthropic.BetaOrganizationUserListParams` (with `anthropic.Int(10)` for `Limit`); Java params use builders from `com.anthropic.models.beta.organization.*` (e.g. `UserListParams.builder().limit(10).build()`). The Go and Java pagination loops:

Go 的参数类型遵循 `anthropic.BetaOrganizationUserListParams` 模式（`Limit` 用 `anthropic.Int(10)`）；Java 的参数使用来自 `com.anthropic.models.beta.organization.*` 的构建器（例如 `UserListParams.builder().limit(10).build()`）。Go 和 Java 的分页循环：

```go
users := client.Beta.Organization.Users.ListAutoPaging(ctx, anthropic.BetaOrganizationUserListParams{Limit: anthropic.Int(10)})
for users.Next() {
	user := users.Current() // ...
}
if err := users.Err(); err != nil { /* handle */ }
```

```java
for (var user : client.beta().organization().users().list(params).autoPager()) { /* ... */ }
```

## Examples / 示例

Common operations (Python spelling; map to other languages with the table above - every operation follows the same shape in each language):

常见操作（Python 拼写；用上表映射到其他语言 —— 每个操作在各语言中形态一致）：

```python
# Organization info
org = client.beta.organization.retrieve()

# List members (iterator auto-fetches more pages; limit = page size)
for user in client.beta.organization.users.list(limit=10):
    print(f"{user.id}: {user.email} ({user.role})")

# Change a member's role / remove a member
client.beta.organization.users.update("user_...", role="developer")
client.beta.organization.users.remove("user_...")

# Invite someone
client.beta.organization.invites.create(email="user@example.com", role="developer")

# Create a workspace and add a member to it
ws = client.beta.organization.workspaces.create(name="Production")
client.beta.organization.workspaces.members.add(
    ws.id, user_id="user_...", workspace_role="workspace_developer"
)

# Deactivate / rename an API key
client.beta.organization.api_keys.update("apikey_...", status="inactive", name="New Key Name")

# Rate limit reports (optional filters: model=..., group_type=...)
client.beta.organization.rate_limits.list(model="claude-opus-5-5")
client.beta.organization.workspaces.rate_limits.list("wrkspc_...")

# Service accounts + WIF (org:admin OAuth token required)
sa = client.beta.organization.service_accounts.create(name="inference-worker", organization_role="developer")
issuer = client.beta.organization.federation.issuers.create(
    name="github-actions",
    issuer_url="https://token.actions.githubusercontent.com",
    jwks={"type": "discovery"},
)
client.beta.organization.federation.rules.create(
    name="gha-deploy",
    issuer_id=issuer.id,
    match={"subject_prefix": "repo:my-org/my-repo:ref:refs/heads/main",
           "claims": {"repository_owner": "my-org"}},
    target={"type": "service_account", "service_account_id": sa.id},
    workspace_id="wrkspc_...",
    oauth_scope="workspace:developer",
    token_lifetime_seconds=600,
)

# CMEK: register, validate, then attach an external key to a workspace
key = client.beta.organization.external_keys.create(
    display_name="prod-key", geo="us",
    provider_config={"type": "aws", "kms_arn": "arn:aws:kms:..."},
)
client.beta.organization.external_keys.validate(key.id)
client.beta.organization.workspaces.update("wrkspc_...", external_key_id=key.id)
```

## Organization Roles / 组织角色

| Role | Permissions |
| --- | --- |
| `user` | Playground |
| `claude_code_user` | Playground + Claude Code |
| `developer` | Playground + manage API keys |
| `billing` | Playground + manage billing |
| `admin` | All of the above + manage users |

| 角色 | 权限 |
| --- | --- |
| `user` | Playground |
| `claude_code_user` | Playground + Claude Code |
| `developer` | Playground + 管理 API 密钥 |
| `billing` | Playground + 管理账单 |
| `admin` | 以上全部 + 管理用户 |

Owners and primary owners have all admin permissions and can also manage admins. Workspace roles are `workspace_user`, `workspace_developer`, `workspace_admin`, and `workspace_billing`.

所有者（owner）和主所有者（primary owner）拥有全部管理员权限，还可以管理管理员。工作区角色为 `workspace_user`、`workspace_developer`、`workspace_admin` 和 `workspace_billing`。

【评论】凭据体系刻意做了双向隔离：普通密钥进不了管理端点，管理凭据也调不了 Messages API，这限制了单一凭据泄漏时的横向影响面。

## Platform Restrictions / 平台限制

- **Claude Platform on AWS:** only the workspace endpoints work. Members, workspace members, invites, API keys, and usage/cost/rate-limit reports are unavailable. CMEK external-key endpoints are not yet available there - register and attach keys in the Claude Console.

- **Claude Platform on AWS：** 只有工作区端点可用。成员、工作区成员、邀请、API 密钥以及用量/成本/速率限制报告均不可用。CMEK 外部密钥端点在该平台上尚未提供 —— 请在 Claude Console 中注册并关联密钥。

- **Claude Enterprise (claude.ai orgs):** only members and invites from this surface, plus Enterprise-only endpoints (group and custom-role reads, spend limits) that are not in the SDKs.

- **Claude Enterprise（claude.ai 组织）：** 此界面仅有成员和邀请功能，外加仅面向 Enterprise 的端点（组和自定义角色读取、支出限额），后者不在 SDK 中。

## Live Docs / 在线文档

| Topic | URL |
| --- | --- |
| Admin API guide | `https://platform.claude.com/docs/en/manage-claude/admin-api.md` |
| Admin API reference | `https://platform.claude.com/docs/en/api/admin.md` |
| Workspaces | `https://platform.claude.com/docs/en/manage-claude/workspaces.md` |
| Rate limits API | `https://platform.claude.com/docs/en/manage-claude/rate-limits-api.md` |
| WIF admin | `https://platform.claude.com/docs/en/manage-claude/wif-admin-api.md` |
| Usage & cost reports (curl-only) | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md` |

| 主题 | URL |
| --- | --- |
| 管理 API 指南 | `https://platform.claude.com/docs/en/manage-claude/admin-api.md` |
| 管理 API 参考 | `https://platform.claude.com/docs/en/api/admin.md` |
| 工作区 | `https://platform.claude.com/docs/en/manage-claude/workspaces.md` |
| 速率限制 API | `https://platform.claude.com/docs/en/manage-claude/rate-limits-api.md` |
| WIF 管理 | `https://platform.claude.com/docs/en/manage-claude/wif-admin-api.md` |
| 用量与成本报告（仅 curl） | `https://platform.claude.com/docs/en/manage-claude/usage-cost-api.md` |
