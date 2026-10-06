<!-- BILINGUAL-EN-ZH -->
# Claude Platform on AWS / AWS 上的 Claude 平台

**Anthropic-operated** access to the Claude Developer Platform through AWS infrastructure - SigV4 authentication, AWS IAM access control, and AWS Marketplace billing. Because Anthropic operates it, **the API surface matches first-party with same-day parity** - for per-feature exceptions, see `shared/platform-availability.md` (the single source of truth; do not rely on an inline exception list here). Model IDs are the bare first-party strings (`claude-opus-5-5`, `claude-sonnet-5-5`) - **no provider prefix**.

**由 Anthropic 运营**的 Claude 开发者平台访问，经由 AWS 基础设施——SigV4 认证、AWS IAM 访问控制、AWS Marketplace 计费。由于由 Anthropic 运营，**API 接口与第一方保持同日对齐**——各功能的例外情况见 `shared/platform-availability.md`（唯一事实来源；不要依赖此处的内联例外清单）。模型 ID 是不带前缀的第一方字符串（`claude-opus-5-5`、`claude-sonnet-5-5`）——**无提供商前缀**。

> **Not the same as Amazon Bedrock.** Bedrock is partner-operated (AWS runs the service; release schedules vary, feature subset, `anthropic.`-prefixed model IDs). Claude Platform on AWS and Bedrock coexist; pick by whether you need AWS-native IAM/billing with full Anthropic API parity (this page) vs. Bedrock's own ecosystem.

> **与 Amazon Bedrock 不是一回事。** Bedrock 由合作方运营（AWS 运行该服务；发布节奏不同、功能为子集、模型 ID 带 `anthropic.` 前缀）。AWS 上的 Claude 平台与 Bedrock 并存；按你是需要 AWS 原生 IAM/计费且完整对齐 Anthropic API（本页），还是需要 Bedrock 自身的生态来选择。

---

## Client & install / 客户端与安装

| Language | Install | Client |
|---|---|---|
| Python | `pip install -U "anthropic[aws]"` | `from anthropic import AnthropicAWS` -> `AnthropicAWS()` |
| TypeScript | `npm install @anthropic-ai/aws-sdk` | `import AnthropicAws from "@anthropic-ai/aws-sdk"` -> `new AnthropicAws()` |
| Go | `go get github.com/anthropics/anthropic-sdk-go` | `import anthropicaws "github.com/anthropics/anthropic-sdk-go/aws"` -> `anthropicaws.NewClient(ctx, anthropicaws.ClientConfig{})` |
| C# | `dotnet add package Anthropic.Aws` | `new AnthropicAwsClient()` |
| Java | See SDK repo in `shared/live-sources.md` | See SDK repo in `shared/live-sources.md` |
| Ruby | `gem install anthropic aws-sdk-core` | See SDK repo in `shared/live-sources.md` |
| PHP | `composer require anthropic-ai/sdk aws/aws-sdk-php` | See SDK repo in `shared/live-sources.md` |

| 语言 | 安装 | 客户端 |
|---|---|---|
| Python | `pip install -U "anthropic[aws]"` | `from anthropic import AnthropicAWS` -> `AnthropicAWS()` |
| TypeScript | `npm install @anthropic-ai/aws-sdk` | `import AnthropicAws from "@anthropic-ai/aws-sdk"` -> `new AnthropicAws()` |
| Go | `go get github.com/anthropics/anthropic-sdk-go` | `import anthropicaws "github.com/anthropics/anthropic-sdk-go/aws"` -> `anthropicaws.NewClient(ctx, anthropicaws.ClientConfig{})` |
| C# | `dotnet add package Anthropic.Aws` | `new AnthropicAwsClient()` |
| Java | 见 `shared/live-sources.md` 中的 SDK 仓库 | 见 `shared/live-sources.md` 中的 SDK 仓库 |
| Ruby | `gem install anthropic aws-sdk-core` | 见 `shared/live-sources.md` 中的 SDK 仓库 |
| PHP | `composer require anthropic-ai/sdk aws/aws-sdk-php` | 见 `shared/live-sources.md` 中的 SDK 仓库 |

After construction, **use the client exactly as you would `Anthropic()`** - `client.messages.create(...)`, `client.beta.sessions.*`, etc., with bare model IDs.

构建之后，**完全像使用 `Anthropic()` 那样使用该客户端**——`client.messages.create(...)`、`client.beta.sessions.*` 等，模型 ID 不带前缀。

```python
from anthropic import AnthropicAWS

client = AnthropicAWS()  # region + workspace_id from env; see below
client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
```

---

## Required configuration / 必需配置

Two values must be available (constructor args or environment) - **there is no default fallback** for either:

两个值必须可用（构造函数参数或环境变量）——两者都**没有默认回退值**：

| Value | Env var | Notes |
|---|---|---|
| AWS region | `AWS_REGION` | Required. Unlike `AnthropicBedrock`, there is no `us-east-1` fallback. |
| Workspace ID | `ANTHROPIC_AWS_WORKSPACE_ID` | Required. Routes requests to your Claude workspace. |

| 值 | 环境变量 | 说明 |
|---|---|---|
| AWS 区域 | `AWS_REGION` | 必填。与 `AnthropicBedrock` 不同，没有 `us-east-1` 回退。 |
| 工作区 ID | `ANTHROPIC_AWS_WORKSPACE_ID` | 必填。将请求路由到你的 Claude 工作区。 |

Endpoint pattern: `https://aws-external-anthropic.{region}.api.aws/v1/...`. Requests are SigV4-signed with service name `aws-external-anthropic`.

端点模式：`https://aws-external-anthropic.{region}.api.aws/v1/...`。请求以服务名 `aws-external-anthropic` 进行 SigV4 签名。

## Authentication / 认证

The client resolves AWS credentials via the standard precedence chain: explicit constructor args -> environment (`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`/`AWS_SESSION_TOKEN`) -> shared profile -> assumed role / instance metadata.

客户端按标准优先级链解析 AWS 凭证：显式构造函数参数 -> 环境变量（`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`/`AWS_SESSION_TOKEN`）-> 共享配置档案 -> 已代入角色 / 实例元数据。

**Short-term API keys** are also supported for cases where SigV4 isn't practical (e.g., browser, simple scripts). Mint one with the per-language token-generator package; pass it as `api_key` on the client. Lifetime is the **lesser of** the requested duration, the underlying credential's expiry, and **12 hours**. For package names and IAM details, WebFetch the Claude Platform on AWS page in `shared/live-sources.md`.

也支持**短期 API 密钥**，适用于 SigV4 不便使用的场景（如浏览器、简单脚本）。用各语言的令牌生成器包生成一个，作为 `api_key` 传给客户端。有效期为请求时长、底层凭证到期时间与 **12 小时**三者中的**最小值**。包名与 IAM 细节，请通过 WebFetch 获取 `shared/live-sources.md` 中的 Claude Platform on AWS 页面。

---

## What to tell users / 应告知用户的内容

- Treat it as first-party: every section of this skill applies unchanged. Do **not** apply Bedrock's feature-availability mask. Three Managed Agents differences only: (1) a session can run autonomously (no user events) for at most **6 hours** before it needs reauthentication - send any user-role event to continue; (2) sessions on **self-hosted** environments **cannot attach memory stores** (rejected at session create) - cloud environments attach them as usual; (3) self-hosted workers authenticate with IAM/SigV4 or an AWS-Console API key plus the `AnthropicSelfHostedEnvironmentAccess` managed policy - Console-generated environment keys don't work against the AWS endpoint.
  将它当作第一方：本技能的每一节都原样适用。**不要**套用 Bedrock 的功能可用性遮罩。Managed Agents 仅有三处差异：(1) 会话至多可自主运行（无用户事件）**6 小时**，之后需要重新认证——发送任意 user 角色事件即可继续；(2) **自托管**环境上的会话**无法挂载记忆存储**（在会话创建时被拒绝）——云环境照常挂载；(3) 自托管 worker 使用 IAM/SigV4 或 AWS 控制台 API 密钥并配合 `AnthropicSelfHostedEnvironmentAccess` 托管策略认证——控制台生成的环境密钥对 AWS 端点无效。
- Model IDs are bare (`claude-opus-5-5`). Do **not** add an `anthropic.` prefix.
  模型 ID 是裸字符串（`claude-opus-5-5`）。**不要**添加 `anthropic.` 前缀。
- A missing region or `workspace_id` throws at client-construction time (no request is sent). A **403** means the request reached the server - check for a **wrong** `workspace_id` or a missing IAM action on the principal. See the IAM actions reference in `shared/live-sources.md`.
  缺少 region 或 `workspace_id` 会在客户端构造时抛出异常（不会发送请求）。**403** 表示请求已到达服务器——检查 `workspace_id` 是否**有误**，或委托主体是否缺少 IAM 操作权限。IAM 操作参考见 `shared/live-sources.md`。
