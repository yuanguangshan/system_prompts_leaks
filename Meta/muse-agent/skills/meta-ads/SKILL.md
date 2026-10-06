<!-- BILINGUAL-EN-ZH -->
---
name: "meta_ads"
title: "Meta Ads"
description: "Create, write, or manage Meta ads and assets: ad copy, campaigns, spend, reports, audiences, catalogs, product feeds, feed refresh schedules, experiments, and policy. Always load for any request to create or write an ad, or to advertise a product or service, even when no platform is named; this includes sensitive or restricted categories. Always load for any question asking what a Meta, Facebook, or Instagram advertising policy means, allows, prohibits, or requires, including a standalone policy-definition question with no account context. Those questions must use ads_policy_tool, never browser search or memory. Load for a specifically named catalog or feed with an upload or refresh-schedule request; use Ads reads to resolve ownership before writing. Generic unnamed feeds need context. Whether an image, claim or piece of copy may be used in an ad is ALWAYS a Meta Ads task — 'can I use this in an ad', 'is it allowed', 'is this against policy', and any rights, likeness, celebrity, logo or trademark question about advertising with an image, including a follow-up about one just generated. Those are ads-policy questions, not general legal ones. An ad request uses the campaign workflow unless explicitly only an image or organic post."
icon: "meta_ads"
metadata: { "includeInPrompt": true }
---

# Meta Ads / Meta 广告

## Purpose / 用途
Use this skill for any Meta Ads read or write. The live catalogue covers
accounts, campaigns, reporting, audiences, datasets, catalogs, experiments,
policy, and help. A bare request to create or design an ad enters the complete
campaign controller unless the advertiser explicitly asks only for a standalone
image asset or organic post. The controller owns full-research versus explicit
guided planning and every approval boundary; do not restate that flow here.

任何 Meta 广告的读取或写入都使用本技能。实时目录涵盖账户、广告系列、报告、受众、数据集、商品目录、实验、政策与帮助。单纯的创建或设计广告请求会进入完整的广告系列控制器，除非广告主明确只要单独的图片素材或自然帖子（organic post）。控制器负责完整调研与显式引导式规划之间的取舍以及所有审批边界；此处不复述该流程。

**Ad spend belongs here.** "How much did I spend", "what did it cost", "where did
the budget go" and "break down my spend" are Meta Ads questions whenever an ad
account, campaign, ad set or ad is in play — route them to this skill, not to a
bank or card connector, which sees money leaving an account and knows nothing
about delivery. Only reach for a financial connector when the user is asking
about their bank, card, or business finances rather than their advertising.
Discover availability with `list-tools --names-only`, inspect selected tools
with `describe-tool`, and invoke them with `call-tool`, including
`ads_creative_upload_media` for new media.

**广告花费归这里管。**"我花了多少钱""这花了多少""预算去哪了"和"把我的花费拆解一下"，只要涉及广告账户、广告系列、广告组或广告，就都是 Meta 广告问题——把它们路由到本技能，而不是银行或卡片连接器；后者只看到账户资金流出，对广告投放一无所知。只有当用户询问的是其银行、卡片或企业财务而非广告时，才动用金融连接器。用 `list-tools --names-only` 发现可用性，用 `describe-tool` 查看选定的工具，用 `call-tool` 调用它们，包括用于新媒体的 `ads_creative_upload_media`。

## Policy questions: mandatory route / 政策问题：强制路由

Treat any question about what a Meta, Facebook, or Instagram advertising policy
means, allows, prohibits, or requires as a Meta Ads task, even when it names no
account, campaign, or ad. Load `references/policy.md`, pass the user's policy
question to `ads_policy_tool`, and call it before answering.
Browser search, a public policy page, and model memory are not substitutes for
the canonical tool result. If the tool is unavailable or returns `N/A`, say the
policy could not be confirmed instead of answering from another source.

凡询问 Meta、Facebook 或 Instagram 广告政策的含义、允许、禁止或要求的问题，都按 Meta 广告任务处理，即使它没有点名任何账户、广告系列或广告。加载 `references/policy.md`，把用户的政策问题传给 `ads_policy_tool`，并在回答前调用它。浏览器搜索、公开政策页面和模型记忆都不能替代该规范工具的结果。如果工具不可用或返回 `N/A`，就说明政策无法确认，而不是用其他来源作答。

【评论】强制用 `ads_policy_tool` 而非模型记忆或网页检索作答，是"易变事实不靠记忆"的典型设计，可防止政策条目过时或被幻觉内容顶替。

## Resolve the owning system before routing / 路由前先解析归属系统

Route on the object and its context, not on a generic noun. `catalog`, `feed`,
`product`, and `audience` do not by themselves mean Meta Ads. They may belong
to a storefront, a content feed, another commerce system, or Meta Ads.

按对象及其上下文路由，而不是按泛指名词。`catalog`、`feed`、`product` 和 `audience` 本身并不必然指 Meta 广告。它们可能属于某个网店、内容 feed、其他商务系统或 Meta 广告。

Treat the request as Meta Ads when at least one of these establishes ownership:

当下列条件至少一条确立归属时，把该请求按 Meta 广告处理：

- the user says Meta Ads, Facebook/Instagram advertising, Ads Manager,
  Commerce Manager, an ad account, campaign, ad set, ad, or dynamic ads;
  用户提到 Meta Ads、Facebook/Instagram 广告、Ads Manager、Commerce Manager、广告账户、广告系列、广告组、广告或动态广告；
- the object is Ads-specific, such as a Meta Pixel/dataset, Custom Audience,
  product set used for ads, or an Ads delivery setting; or
  对象是广告专有的，例如 Meta Pixel/数据集、Custom Audience、用于广告的商品集，或某个广告投放设置；或
- the current conversation or a read-only Meta Ads lookup has already resolved
  the named object or id inside the user's Meta Ads account.
  当前对话或一次只读的 Meta 广告查询已在用户的 Meta 广告账户中解析出所指对象或 id。

A specific catalog or product-feed name or id **must load this skill first** for
read-only ownership resolution, even when the user did not say Meta Ads. The
same is true for a specifically named feed when the request concerns its upload
or refresh schedule, for example "change my Shopify nightly feed to refresh
hourly." Use the required Ads list/read chain to look for that exact object
before searching cron jobs, hooks, tracking items, reminders, or local scripts;
those surfaces are not substitutes for an Ads product-feed lookup. One unique match
establishes Ads ownership; only then may an Ads write be proposed. If no object
matches, or more than one object could be the target, ask one short
disambiguating question and do not write. A platform-like word in a name, such
as `Shopify`, is part of the supplied name and does not prove the feed belongs
to that external platform.

具体的商品目录或产品 feed 名称或 id **必须先加载本技能**做只读的归属解析，即使用户没有说 Meta 广告。当请求涉及某个具名 feed 的上传或刷新计划时同理，例如"把我的 Shopify 每日 feed 改成每小时刷新"。在搜索 cron 任务、钩子、追踪项、提醒或本地脚本之前，先用必需的 Ads 列表/读取链查找那个确切对象；这些界面不能替代 Ads 产品 feed 查询。唯一匹配即确立 Ads 归属；只有那时才可以提出 Ads 写操作。如果没有对象匹配，或可能有多个候选目标，就问一个简短的澄清问题，并且不执行写入。名称中类似平台的词（如 `Shopify`）只是所提供名称的一部分，不能证明该 feed 属于那个外部平台。

When only a generic noun is present and ownership is still unclear, ask one
short disambiguating question, such as whether they mean their Meta Ads product
feed or another feed. Do not mutate either system while it is ambiguous. After
an object is resolved to Meta Ads, do not hand it to a generic Feed or commerce
tool, and do not silently fall back to another system when an Ads capability is
unavailable.

当只出现泛指名词且归属仍不明确时，问一个简短的澄清问题，例如他们指的是 Meta 广告产品 feed 还是别的 feed。归属不明时不要改动任何一边的系统。对象解析为 Meta 广告之后，不要把它交给通用的 Feed 或商务工具，也不要在某个 Ads 能力不可用时静默回退到其他系统。

## References / 参考文件

Advertising is a domain where a fluent, confident, wrong answer costs the
advertiser money. These files carry the rules that stop that. Read the ones the
task touches **before** answering, not after drafting.

广告是一个流畅而自信的错误答案会让广告主花钱的领域。这些文件载有阻止这种情况发生的规则。在回答**之前**（而不是起草之后）阅读任务涉及的文件。

| Read this | When |
|---|---|
| `references/safety.md` | **Always.** Ten HARD boundaries: audience guidance, PII, proposal vs execution, Special Ad Category, policy sourcing. |
| `references/account-scope.md` | Any question about an account, catalog, audience, feed, or experiment — which is nearly all of them. **Especially when the question names none**: deciding whether to ask, or to carry one forward from an earlier turn, is what this file governs. |
| `references/evidence.md` | Any answer that reports a number, or that has to describe something missing. |
| `references/analysis.md` | Why, trend, ranking, comparison, or recommendations based on existing delivery performance. |
| `references/response-style.md` | Reporting, analysis, or metric-definition answers. Campaign-stage references own campaign presentation. |
| `references/tool-routing.md` | Reads not already routed by the active campaign-stage reference, especially catalogs, datasets, audiences, experiments, and analysis levels. |
| `references/policy.md` | Before any `ads_policy_tool` call, any question about what Meta's ad policies allow or prohibit, and any general advertising how-to. |
| `references/campaign-creation.md` | Start here for a complete new campaign. This short controller identifies the first incomplete stage; do not preload its later-stage references. |
| `references/campaign-planning.md` | Only while a complete campaign still needs identity resolution, a product brief, research, recommendations, hierarchy, full-plan approval, or an optional strategy artifact. |
| `references/campaign-guided.md` | Only when the advertiser explicitly requests step-by-step planning. |
| `references/campaign-delivery-compatibility.md` | While settling objective/optimization after destination and tracking are known, again on exact arguments before final review, and before adding an ad set to an existing campaign. |
| `references/campaign-targeting.md` | Only when campaign planning must resolve a non-country place, interest, or language into a canonical targeting object. |
| `references/campaign-budget.md` | Only when a proposed campaign is ready for budget pricing or its pricing inputs changed; full research loads it from planning after those inputs stabilize. |
| `references/campaign-creative.md` | Only when preparing or approving any campaign creative, including image, video, carousel, boosted-post, and partnership-ad formats. |
| `references/campaign-execution.md` | Only when preparing the final review, collecting its exact approval, creating the paused hierarchy, presenting its immediate handoff, or recovering partial creation. |
| `references/campaign-handoff.md` | Only after the advertiser selects a post-create delivery or editing action. |
| `references/campaign-manual-setup.md` | When a write was rejected as not available for this ad account — by a tool result, or quoted by the advertiser from an earlier attempt — or the advertiser asks to set the campaign up themselves in Ads Manager. Read it before explaining that rejection. |
| `references/writes.md` | Before executing a standalone create, update, activate, pause, delete, connect, or upload. Complete campaigns load it only when their staged references direct. |

| 先读这个 | 何时 |
|---|---|
| `references/safety.md` | **始终。**十条硬性边界：受众指导、PII、提案与执行、特殊广告类别、政策来源。 |
| `references/account-scope.md` | 任何关于账户、商品目录、受众、feed 或实验的问题——这几乎涵盖所有问题。**尤其当问题没有点名任何对象时**：决定是询问，还是沿用先前轮次的对象，正是本文件管辖的内容。 |
| `references/evidence.md` | 任何报告数字的回答，或必须描述缺失之处的回答。 |
| `references/analysis.md` | 基于既有投放表现的"为什么"、趋势、排名、对比或建议。 |
| `references/response-style.md` | 报告、分析或指标定义类回答。广告系列阶段的参考文件主导广告系列的呈现方式。 |
| `references/tool-routing.md` | 尚未被当前广告系列阶段参考文件路由的读取，尤其是 catalogs、datasets、audiences、experiments 和分析层级。 |
| `references/policy.md` | 任何 `ads_policy_tool` 调用之前、任何关于 Meta 广告政策允许或禁止的问题，以及任何通用的广告操作指南问题。 |
| `references/campaign-creation.md` | 全新完整广告系列从这里开始。这个简短的控制器识别第一个未完成的阶段；不要预加载其后期阶段参考文件。 |
| `references/campaign-planning.md` | 仅当完整广告系列仍需要身份解析、产品简报、调研、建议、层级结构、完整计划审批或可选的策略产物时。 |
| `references/campaign-guided.md` | 仅当广告主明确请求逐步规划时。 |
| `references/campaign-delivery-compatibility.md` | 在目的地与追踪已知后确定目标/优化方式时、最终评审前再次核对确切参数时，以及在向现有广告系列添加广告组之前。 |
| `references/campaign-targeting.md` | 仅当广告系列规划必须把非国家级的地点、兴趣或语言解析为规范的定向对象时。 |
| `references/campaign-budget.md` | 仅当拟议广告系列已准备好做预算定价，或其定价输入发生变化时；完整调研会在这些输入稳定后从规划阶段加载它。 |
| `references/campaign-creative.md` | 仅当准备或审批任何广告系列创意时，包括图片、视频、轮播、加速帖和合作广告格式。 |
| `references/campaign-execution.md` | 仅当准备最终评审、收集其确切审批、创建暂停状态的层级、呈现其即时交接或恢复部分创建时。 |
| `references/campaign-handoff.md` | 仅在广告主选择了创建后的投放或编辑操作之后。 |
| `references/campaign-manual-setup.md` | 当某次写入被拒绝为"对此广告账户不可用"时——由工具结果给出，或广告主引用早先尝试的说法——或广告主要求自己在 Ads Manager 中搭建广告系列。解释该拒绝之前先阅读它。 |
| `references/writes.md` | 在执行独立的创建、更新、启用、暂停、删除、连接或上传之前。完整广告系列仅在其暂存的参考文件指示时才加载它。 |

## Tooling / 工具
Use `exec` to run the installed binary directly:

使用 `exec` 直接运行已安装的二进制文件：

```sh
/opt/hatch/bin/meta-ads-cli <subcommand> [options]
```

**HARD invocation boundary:** run every Meta Ads command as its own `exec` tool
call. Never combine Meta Ads commands with a newline, `&&`, `;`, a pipe, or a
wrapper such as `cd`, `env`, `timeout`, or `bash`. That includes `| head` and
`| python3`: results over 128 KiB already come back as an `output_file`
(below), and `describe-tool --input-only` returns just the schema, so there is
nothing to truncate or parse inline. In particular, the complete
command string for the first protected account lookup must be exactly:

**硬性调用边界：**每条 Meta 广告命令都要作为独立的 `exec` 工具调用运行。绝不把 Meta 广告命令与换行符、`&&`、`;`、管道或 `cd`、`env`、`timeout`、`bash` 之类的包装器组合。这包括 `| head` 和 `| python3`：超过 128 KiB 的结果本来就会以 `output_file`（见下文）返回，而 `describe-tool --input-only` 只返回 schema，因此没有任何需要截断或内联解析的东西。特别地，首个受保护账户查询的完整命令字符串必须恰好是：

```sh
/opt/hatch/bin/meta-ads-cli call-tool --name ads_get_ad_accounts
```

The runtime directly spawns and attests that installed binary. A combined or
wrapped command requires a shell and intentionally cannot open the trusted
professional-consent presentation.

运行时直接生成并证明（attest）该已安装的二进制文件。组合或包装过的命令需要 shell，并被刻意设计为无法打开受信任的专业授权展示界面。

【评论】"每条命令单独 exec、禁止管道与包装器"不只是习惯约定：组合命令需要 shell，而受信任的授权界面只会由运行时直接生成的进程打开，这是一条针对命令拼接的信任链防护。

### Connection management and diagnostics / 连接管理与诊断

```sh
/opt/hatch/bin/meta-ads-cli status
/opt/hatch/bin/meta-ads-cli disconnect-url
```

`status` currently includes the full tool catalogue. Reserve it for connection
diagnostics; do not use it for normal capability discovery.

`status` 目前包含完整的工具目录。把它留给连接诊断；不要用它做常规能力发现。

When the user asks to disconnect, disable, or turn off Meta Business access,
run `disconnect-url` as its own `exec` call. When the response contains
`disconnect_url`, share exactly
`[Disconnect Meta Business Manager](<disconnect_url>)`, without also pasting
the raw URL. The link opens Hatch's standard connector disconnect confirmation;
do not claim access was removed until the user confirms there. Never use the
retired `disconnect` command, which belonged to connector OAuth.

当用户要求断开、禁用或关闭 Meta Business 访问时，把 `disconnect-url` 作为独立 `exec` 调用运行。当响应包含 `disconnect_url` 时，原样分享 `[Disconnect Meta Business Manager](<disconnect_url>)`，不要同时粘贴原始 URL。该链接打开 Hatch 的标准连接器断开确认页；在用户在那里确认之前，不要声称访问已被移除。绝不使用已废弃的 `disconnect` 命令，它属于旧式连接器 OAuth。

### MCP operations / MCP 操作

```sh
/opt/hatch/bin/meta-ads-cli list-tools --names-only
/opt/hatch/bin/meta-ads-cli describe-tool --name <tool>
/opt/hatch/bin/meta-ads-cli describe-tool --name <tool> --input-only
/opt/hatch/bin/meta-ads-cli call-tool --name <tool> --agent-output --arguments-json '<json-object>'
/opt/hatch/bin/meta-ads-cli estimate-budget <typed plan flags>
/opt/hatch/bin/meta-ads-cli render-campaign-summary --summary-json '<json-object>'
/opt/hatch/bin/meta-ads-cli render-campaign-success --success-json '<json-object>'
/opt/hatch/bin/meta-ads-cli render-chart --chart-json '<json-object>'
```

Before any `estimate-budget` invocation, read `references/campaign-budget.md`;
its typed flag grammar is complete. Never substitute `--help` or raw MCP-schema
arguments.

任何 `estimate-budget` 调用之前，先阅读 `references/campaign-budget.md`；它的类型化旗标语法是完备的。绝不改用 `--help` 或原始 MCP-schema 参数。

The three render commands are local and credential-free. Pass each returned
`widget.kind` and `widget.data` to `widget.create` unchanged; never replace the
payload with hand-authored HTML or a placeholder. `render-chart` returns an
`html_file` card plus a `full_view.path`, not a link to include in prose.
`--chart-json` accepts exactly these keys and rejects any other:  
`{"type":"line"|"bar","title":"…","metric":"…","unit":"currency"|"percent"|"number","currency":"USD","x_labels":["…"],"series":[{"name":"…","values":[…]}],"reference":{"label":"…","value":…}}`;  
`currency` (the account's ISO code) is required with `unit: "currency"` and
rejected with any other unit; `reference` is optional, and `name` is required
when there are two or more series.

三个 render 命令都是本地的且无需凭据。把每个返回的 `widget.kind` 和 `widget.data` 原样传给 `widget.create`；绝不用手写 HTML 或占位符替换载荷。`render-chart` 返回一张 `html_file` 卡片加一个 `full_view.path`，而不是可放进正文文字的链接。  
`--chart-json` 只接受这些键并拒绝任何其他键：  
`{"type":"line"|"bar","title":"…","metric":"…","unit":"currency"|"percent"|"number","currency":"USD","x_labels":["…"],"series":[{"name":"…","values":[…]}],"reference":{"label":"…","value":…}}`；  
`currency`（账户的 ISO 代码）在 `unit: "currency"` 时必填，在其他单位时会被拒绝；`reference` 可选；有两个或以上系列时 `name` 必填。

Status, discovery, and tool-call responses over 128 KiB return `output_file`,
`output_bytes`, and `output_format: "json"` instead of the inline response.
`output_file` contains the complete original JSON. Read or query that file in a
separate tool call, selecting only the fields or array slice needed; do not
print the entire file back through `exec`. Small responses keep their usual
shape. The file is temporary and need not survive a restart.

超过 128 KiB 的状态、发现和工具调用响应会返回 `output_file`、`output_bytes` 和 `output_format: "json"`，而不是内联响应。`output_file` 包含完整的原始 JSON。在单独的工具调用中读取或查询该文件，只选取需要的字段或数组切片；不要通过 `exec` 把整个文件打印回来。小响应保持惯常形态。该文件是临时的，无需在重启后保留。

`ok` still describes the Ads operation. If `output_available: false` is
returned, output delivery failed even if the operation succeeded. Do not repeat
an Ads write solely to recover its result; check the resulting Ads state first.

`ok` 仍然描述 Ads 操作本身。如果返回 `output_available: false`，即使操作成功，输出送达也失败了。不要仅仅为了找回结果而重复一次 Ads 写操作；先检查由此形成的 Ads 状态。

### Placement-specific image generation / 按版位生成图片

For campaign images, follow `references/campaign-creative.md` and run the
Ads-owned wrapper as a standalone command:

广告系列图片请遵循 `references/campaign-creative.md`，并把 Ads 自有的包装命令作为独立命令运行：

```sh
/opt/hatch/bin/meta-ads-cli creative generate-image --placement <feed|story|reel> --prompt '<text>' --output-dir workspace/your_files
```

The wrapper owns placement ratios and output validation. Repeat `--prompt` for
ordered constraints; add `--source-image <path>` only to adapt that accepted
image. Use `local_path` for review and retain `media_handle` when returned.
After approval, upload with that handle, or the exact `local_path` as `file`
when no handle exists, then use only the returned Ads reference.

包装命令负责版位比例与输出校验。需要有序约束就重复 `--prompt`；只有要改编那张已被接受的图片时才加 `--source-image <path>`。用 `local_path` 做审查，返回了 `media_handle` 就保留它。批准后用该句柄上传；没有句柄时以确切的 `local_path` 作为 `file`，然后只使用返回的 Ads 引用。

New media reaches Ads only through `ads_creative_upload_media`. Asset selection,
source precedence, returned references, and approval ordering live in  
`references/campaign-creative.md`.

新媒体只能通过 `ads_creative_upload_media` 进入 Ads。素材选择、来源优先级、返回引用和审批顺序都在  
`references/campaign-creative.md`。

There is one endpoint. Do not pass `--endpoint`: a shipped build rejects any host
but the default, so an override does not reach a different server — it fails, and
the failure reads to the advertiser as Meta Ads being down.

只有一个端点。不要传 `--endpoint`：已发布的构建会拒绝默认主机以外的任何主机，因此这种覆盖并不会到达别的服务器——它只会失败，而在广告主看来就是 Meta 广告宕机了。

`--arguments-json` accepts one object matching the live schema. Omit it when the
schema needs no arguments; the CLI supplies `{}`.

`--arguments-json` 接受一个符合实时 schema 的对象。schema 无需参数时省略它；CLI 会自动提供 `{}`。

## Auth / 认证
New connections use the in-Hatch consent flow and the linked Facebook account.
Each Meta Ads operation executes inside WWW as that linked Facebook viewer; no
Facebook access token enters the Hatch VM.

新连接使用 Hatch 内置的授权流程和已关联的 Facebook 账户。每个 Meta 广告操作都在 WWW 内以该已关联的 Facebook 查看者身份执行；Facebook 访问令牌不进入 Hatch VM。

Scopes: `ads_read`, `ads_management`, `instagram_basic`, `pages_show_list`, `business_management`, `catalog_management`.

权限范围（Scopes）：`ads_read`、`ads_management`、`instagram_basic`、`pages_show_list`、`business_management`、`catalog_management`。

## First-use setup flow / 首次使用设置流程

Discover compact names, then call the Ads operation the user requested. Do not
ask the user to connect a generic Meta Ads connector and do not produce a
`/connectors/connect/meta_ads` link. Meta Ads uses the linked Facebook identity
and the first-party professional-consent flow; legacy connector OAuth is
retired.

先发现紧凑名称，然后调用用户请求的 Ads 操作。不要让用户去连接通用的 Meta Ads 连接器，也不要生成 `/connectors/connect/meta_ads` 链接。Meta 广告使用已关联的 Facebook 身份和第一方专业授权流程；旧式连接器 OAuth 已废弃。

If a command result has `error_reason: PROFESSIONAL_CONSENT_REQUIRED`, Hatch
is showing the Meta Business Manager consent card. For a blocked Ads request,
respond with one short, user-facing sentence: "To connect your Meta Ads and
professional account analytics, please review the Meta Business Manager
connector and enable its access." Do not mention internal terms such as
professional-account consent, tool calls, blocked requests, or automatic
resume. Do not retry the tool or ask the user to say "try again"; successful
consent automatically resumes the exact blocked request.

如果命令结果含有 `error_reason: PROFESSIONAL_CONSENT_REQUIRED`，说明 Hatch 正在展示 Meta Business Manager 授权卡片。对于被拦下的 Ads 请求，用一句简短的面向用户的话回应："要连接你的 Meta 广告与专业账户分析，请查看 Meta Business Manager 连接器并启用其访问权限。"不要提及内部术语，例如专业账户授权、工具调用、被拦截的请求或自动恢复。不要重试该工具，也不要让用户说"再试一次"；授权成功后会自动恢复那条被拦下的请求。

If a command result has `error_reason: FACEBOOK_ACCOUNT_LINK_REQUIRED` or its
error begins with `FACEBOOK_ACCOUNT_LINK_REQUIRED:`, this is not the
professional-consent flow. Tell the user that Meta Business requires a linked
Facebook account and include the exact `next_action.url` (or the URL in the
error) as a clickable link labeled **Open Meta Accounts Center guidance**. Do
not replace that URL, show a connector card, or imply that consent was granted.
Stop the Ads workflow until the user links the account, then retry their
original request.

如果命令结果含有 `error_reason: FACEBOOK_ACCOUNT_LINK_REQUIRED` 或其错误以 `FACEBOOK_ACCOUNT_LINK_REQUIRED:` 开头，这就不是专业授权流程。告诉用户 Meta Business 需要已关联的 Facebook 账户，并把确切的 `next_action.url`（或错误中的 URL）作为一个可点击链接附上，标签为 **Open Meta Accounts Center guidance**。不要替换该 URL，不要展示连接器卡片，也不要暗示授权已授予。暂停 Ads 工作流，直到用户关联账户，然后重试其原始请求。

Do not reconnect for HTTP 400, Graph `code 100` / `subcode 33`, JSON-RPC
errors, rollout errors, or tool-discovery failures. A remote failure is a
failure to report, not an absence of connection — `references/evidence.md`
governs how to say it.

不要为 HTTP 400、Graph `code 100` / `subcode 33`、JSON-RPC 错误、灰度（rollout）错误或工具发现失败而重新连接。远端失败是需要如实报告的失败，而不是连接缺失——如何表述由 `references/evidence.md` 管辖。

## Discovery-first workflow / 发现优先的工作流

**Tool execution begins with discovery.** Do not assume tool names, shapes, or
availability from memory — the server catalogue is gated per tool and evolves.
A tool remembered as missing, failing, or not rolled out gets the same fresh
check as any other. For a new
campaign, follow `references/campaign-creation.md` and load its planning stage
before asking a genuinely missing decision;
do not narrate the workflow before asking for a genuinely missing decision.
Names in `references/tool-routing.md` are the intent map, not a guarantee of
availability: confirm each against `list-tools --names-only` and inspect its
descriptor before calling it. Say you cannot do something rather than calling a
tool that is not there.

**工具执行从发现开始。**不要凭记忆假设工具名称、形态或可用性——服务器目录按工具做门控且会演进。一个被记忆为缺失、失败或未灰度开放的工具，要与其他工具一样重新检查。对于新广告系列，遵循 `references/campaign-creation.md`，在询问真正缺失的决策之前加载其规划阶段；也不要在询问真正缺失的决策之前先叙述工作流。`references/tool-routing.md` 中的名称是意图地图，不是可用性保证：逐一对照 `list-tools --names-only` 确认，并在调用前查看其描述符。宁可说自己做不到，也不要调用不存在的工具。

1. **List tool names once per conversation**: run
   `/opt/hatch/bin/meta-ads-cli list-tools --names-only` as a standalone `exec`
   call. Reuse that successful result across later turns in the same
   conversation. Refresh it after a failed operation, a CLI/skill update, or a
   signal that capabilities changed. Do not re-list before every call. Wait for
   this result before issuing
   any `call-tool`; do not fan discovery and execution out in parallel.
   **每个会话只列一次工具名称**：把 `/opt/hatch/bin/meta-ads-cli list-tools --names-only` 作为独立 `exec` 调用运行。在同一会话的后续轮次中复用这一成功结果。在操作失败、CLI/技能更新或能力发生变化的信号之后刷新它。不要在每次调用前都重新列出。发出任何 `call-tool` 之前先等待该结果；不要把发现与执行并行展开。
2. **Resolve the owning system, then match the user's intent to a tool** using
   the rules above and `references/tool-routing.md`. A generic noun is not
   enough to choose Meta Ads; a resolved Meta Ads object must not be handed to
   a lookalike tool elsewhere.
   Before a generic `call-tool`, run `/opt/hatch/bin/meta-ads-cli describe-tool
   --name <tool>` for each selected current-stage tool and reuse it while that
   schema remains current in the conversation.
   Use `--input-only` when the operation is already selected and only its
   argument contract is needed. The typed budget command validates its live
   schema internally, but read the estimator's input-only descriptor first so
   objective, optimization, and result enums are current. Independent
   descriptor commands may run together, but do not fetch later-stage create
   schemas early.
   If discovery is incomplete, truncated, unreadable, or lacks a required tool,
   stop rather than guessing a schema or probing with any write.
   If `list-tools --names-only` or `describe-tool` is rejected, stop and report
   an installed CLI/skill version mismatch. Do not substitute `status`, bare
   `list-tools`, or `--help`; scrape a truncated catalogue; inspect eval files;
   guess flags; or probe availability or argument shapes through `call-tool`
   with a real, guessed, or fabricated name.
   **先解析归属系统，再把用户意图匹配到工具**：使用上述规则和 `references/tool-routing.md`。泛指名词不足以选择 Meta 广告；已解析为 Meta 广告的对象绝不能交给别处的相似工具。
   在通用 `call-tool` 之前，对每个选定的当前阶段工具运行 `/opt/hatch/bin/meta-ads-cli describe-tool --name <tool>`，并在该 schema 在会话中保持最新期间复用它。
   操作已选定且只需要其参数契约时使用 `--input-only`。类型化预算命令会在内部校验其实时 schema，但先读估算器的 input-only 描述符，确保 objective、optimization 和 result 枚举是最新的。相互独立的描述符命令可以一起运行，但不要过早获取后期阶段的创建 schema。
   如果发现不完整、被截断、不可读或缺少所需工具，就停下来，而不是猜测 schema 或用任何写入去探测。
   如果 `list-tools --names-only` 或 `describe-tool` 被拒绝，停下并报告已安装的 CLI/技能版本不匹配。不要改用 `status`、裸 `list-tools` 或 `--help`；不要抓取截断的目录；不要检查 eval 文件；不要猜测旗标；也不要用真实、猜测或编造的名称通过 `call-tool` 探测可用性或参数形态。
3. **Resolve only missing prerequisite IDs**: reuse an ID the user supplied or a
   successful Ads call already established in this conversation. For a read-only
   specialized tool, pass an explicit `ad_account_id` directly; do not call
   `ads_get_ad_accounts`, `ads_get_ad_entities`, or `ads_get_field_context` merely
   to verify or prepare it. If a required ID is missing, resolve it with the
   relevant list tool. For an unresolved account, call
   `/opt/hatch/bin/meta-ads-cli call-tool --name ads_get_ad_accounts` in its own
   `exec` call (the empty arguments object is implicit). Do not add quotes, JSON
   arguments, prefixes, suffixes, or other commands to this invocation. Writes
   still require the account and target-object verification described in
   `references/writes.md`. Never guess an ID — a guessed ID usually returns
   another object's data rather than an error. It
   is the first protected account operation, not the first discovery command:
   compact discovery and its exact descriptor fetch still precede it.
   **只解析缺失的前置 ID**：复用用户提供的、或本会话中成功的 Ads 调用已确立的 ID。对只读专用工具，直接传入显式的 `ad_account_id`；不要仅仅为了验证或准备而调用 `ads_get_ad_accounts`、`ads_get_ad_entities` 或 `ads_get_field_context`。如果缺少必需的 ID，用相关的列表工具解析。对未解析的账户，把 `/opt/hatch/bin/meta-ads-cli call-tool --name ads_get_ad_accounts` 作为独立 `exec` 调用运行（空参数对象是隐式的）。不要给这次调用加引号、JSON 参数、前缀、后缀或其他命令。写入仍需 `references/writes.md` 所述的账户与目标对象验证。绝不猜测 ID——猜出来的 ID 通常返回的是别的对象的数据而不是报错。它是首个受保护的账户操作，不是首个发现命令：紧凑发现及其确切的描述符获取仍在其之前。
4. **Call the tool**: append `--agent-output` to normal calls and build the JSON
   strictly from the selected tool's `input_schema`; do not add undeclared
   fields. Omit `--arguments-json` when the schema requires no fields. Preserve
   the exact first-account command above without `--agent-output`.
   When a schema exposes `advertiser_request`, use the advertiser's complete
   current multi-turn wording: the original request plus their later
   refinements and selections, not only the latest reply or your summary.
   If the CLI returns `tool_not_exposed`, do not retry with spelling variants or
   a fabricated name: choose a confirmed exposed fallback or report the missing
   capability. If it returns `tool_catalogue_unavailable`, stop and report that
   live Ads capabilities could not be checked.
   **调用工具**：普通调用附加 `--agent-output`，并严格按所选工具的 `input_schema` 构建 JSON；不加未声明的字段。schema 不需要字段时省略 `--arguments-json`。上述确切的首账户命令保持原样，不加 `--agent-output`。
   当 schema 暴露 `advertiser_request` 时，使用广告主完整的当前多轮措辞：原始请求加上其后续的细化与选择，而不仅是最近一条回复或你的摘要。
   如果 CLI 返回 `tool_not_exposed`，不要用拼写变体或编造的名称重试：选择一个已确认暴露的回退工具，或报告缺失的能力。如果返回 `tool_catalogue_unavailable`，停下并报告无法检查实时 Ads 能力。
5. **Consume output directly**: use the returned JSON. Report errors faithfully;
   a failed call is unavailable evidence, not a finding.
   **直接消费输出**：使用返回的 JSON。如实报告错误；失败的调用是不可用的证据，不是一项发现。
6. **Cite named Ads entities**: call `ads.resolve_entities` once before every
   response that mentions an ad account, campaign, ad set, or ad from Ads tool
   results. Include every such entity with the exact returned name and ID. For
   campaigns, ad sets, and ads, also include the returned owning ad account ID.
   Copy each returned citation marker exactly into the response. Do this even
   if the user did not ask for links. If the resolver is unavailable or an
   entity is missing a required ID, mention it without a citation. Do not
   invent names or IDs.
   **引用具名 Ads 实体**：在每个提及来自 Ads 工具结果的广告账户、广告系列、广告组或广告的响应之前，调用一次 `ads.resolve_entities`。把每个此类实体连同返回的确切名称与 ID 一并写入。对广告系列、广告组和广告，还要包含返回的所属广告账户 ID。把每个返回的引用标记原样复制进响应。即使用户没有要求链接也照做。如果解析器不可用或某实体缺少必需 ID，提及它但不加引用。绝不编造名称或 ID。

Before the first call, privately inventory every requested result and action. For each one:
select and describe the tool, resolve IDs and current state, obtain any required
approval, execute once, and verify the authoritative result. Before answering,
check that every requested part is either supported by retrieved evidence or
has one precise limitation statement.

首次调用之前，先在内部盘点每个被请求的结果与操作。对每一个：选择并描述工具、解析 ID 与当前状态、取得所需的审批、执行一次，并核实权威结果。回答之前，检查每个被请求的部分要么有检索到的证据支撑，要么有一句精确的限制声明。

The complete intent map, list/detail rules, and argument guidance live only in
`references/tool-routing.md`; do not duplicate them here.

完整的意图地图、列表/详情规则和参数指导只存在于 `references/tool-routing.md`；此处不复述。

## Write and destructive actions / 写入与破坏性操作

Read `references/writes.md` before changing an account and
`references/campaign-execution.md` for complete campaigns. Safety owns what an
approval permits and what may be claimed after a write.

更改账户前阅读 `references/writes.md`，完整广告系列则阅读 `references/campaign-execution.md`。审批允许什么、写入后可以声称什么，由安全文件说了算。

【评论】把"审批授权的范围"与"写入后可声称的状态"集中到 safety 文件统一裁决，是权限语义单一来源的做法，可避免各参考文件之间出现口径不一致。

## Operating Rules / 操作规则

1. **Never print access tokens or the `client_secret`.
   **绝不打印访问令牌或 `client_secret`。
2. **Do not invent IDs** — ad account, campaign, ad set, ad, catalog, audience, dataset, pixel, page, creative, or Instagram account IDs must come from a tool response or the user's message. If you don't have an ID, call the appropriate list/discover tool first.
   **不要编造 ID**——广告账户、广告系列、广告组、广告、商品目录、受众、数据集、pixel、主页、创意或 Instagram 账户 ID 必须来自工具响应或用户的消息。没有 ID 时，先调用相应的列表/发现工具。
3. **Explain, then invoke the write.** When the requested action is fully
   specified, supply the sentence that makes it meaningful and call the tool in
   the same response. The runtime approval card is the confirmation boundary;
   do not replace it with a yes/no chat question. `references/writes.md`.
   **先解释，再调用写入。**当被请求的操作已完全明确时，给出使其有意义的那句话，并在同一响应中调用工具。运行时审批卡片是确认边界；不要用聊天里的"是/否"提问取代它。`references/writes.md`。
4. **Respect the schema.** Send only fields declared in the tool's `input_schema`. Do not guess at field names.
   **尊重 schema。**只发送工具 `input_schema` 中声明的字段。不要猜测字段名。
5. **Report errors faithfully.** Include the JSON-RPC error `code` and `message`, or the MCP `isError` result body when present. Do not silently retry destructive calls. A failed tool is not a finding — see `references/evidence.md`.
   **如实报告错误。**包含 JSON-RPC 错误的 `code` 和 `message`，或 MCP `isError` 结果体（如存在）。不要静默重试破坏性调用。失败的工具不是一项发现——见 `references/evidence.md`。
6. **Stay within the requested scope.** Agreement to one change does not authorize unrelated changes.
   **不超出被请求的范围。**同意一项更改并不授权无关的更改。
7. **Make every progress claim match tool evidence.** No write call means the
   action has not started. A pending approval means it is waiting for the
   user's decision, not staged, queued, or underway. A refusal or cancellation
   means no change was made. An error means the action failed. Use completed
   past tense only for the exact fields and objects a successful write result
   proves changed.
   **让每个进度声明与工具证据一致。**没有写入调用意味着操作尚未开始。待审批意味着在等用户决定，而不是已暂存、已排队或进行中。拒绝或取消意味着没有做任何更改。错误意味着操作失败。只有成功写入结果能证明已更改的确切字段和对象，才用完成时态表述。
8. **Do not turn task observations into persistent state.** Existing memory may
   inform stable business facts under the planning rules, but never write or
   update memory from an Ads workflow. Tool availability, rollout or gating,
   errors, account eligibility, and every Ads object, status, metric, or other
   tool result are live state that changes between conversations. Memory or a
   prior conversation holding one is unverified: never skip or shortcut
   discovery or a read because of it, and fetch it again in this conversation. Never use `MEMORY.md`, `AGENTS.md`, a
   persistence API, workspace files, skill files, or other local state as a
   campaign ledger. A read-only Ads request permits only reads; user-requested
   output artifacts are the sole exception.
   **不要把任务观察变成持久状态。**既有记忆可以在规划规则下为稳定的商业事实提供参考，但绝不从 Ads 工作流写入或更新记忆。工具可用性、灰度或门控、错误、账户资格，以及每个 Ads 对象、状态、指标或其他工具结果，都是会话之间会变化的实时状态。记忆或先前会话持有其中任何一项都视为未经验证：绝不因此跳过或省去发现或读取，要在本会话中重新获取。绝不把 `MEMORY.md`、`AGENTS.md`、持久化 API、工作区文件、技能文件或其他本地状态当作广告系列台账。只读的 Ads 请求只允许读取；用户要求的输出产物是唯一例外。
9. **Never fabricate a metric value.** Report only figures a tool returned, and never compute, average, or extrapolate one. `references/evidence.md` and `references/response-style.md` carry the detail.
   **绝不编造指标值。**只报告工具返回的数字，绝不计算、求平均或外推。细节见 `references/evidence.md` 和 `references/response-style.md`。
10. **Never state what a Meta ad policy says without retrieving it this turn, and read `references/policy.md` before any `ads_policy_tool` call.** Pass the user's policy question as `query`; the tool resolves it against the current live catalogue.
    **不在本轮检索的情况下绝不陈述 Meta 广告政策的内容，且任何 `ads_policy_tool` 调用前先读 `references/policy.md`。**把用户的政策问题作为 `query` 传入；工具会对照当前实时目录解析它。
11. **Never claim readiness from credential presence alone.** Say Meta Ads is connected and ready only after `meta-ads-cli status` returns `authenticated: true` together with a `tools` catalogue.
    **绝不只凭凭据存在就声称就绪。**只有在 `meta-ads-cli status` 返回 `authenticated: true` 且带有 `tools` 目录之后，才能说 Meta 广告已连接并就绪。
12. **Never invent an audience.** Do not infer age, gender, geography,
    interests, exclusions, or lookalikes from business or account context.
    Broad Advantage+ Audience is the default. Verified evidence may support
    expandable suggestions within it; strict narrowing remains
    advertiser-initiated. `references/safety.md` rule 1.
    **绝不发明受众。**不要从业务或账户上下文推断年龄、性别、地域、兴趣、排除项或类似受众。宽泛的 Advantage+ Audience 是默认。经验证的证据可以支持其中的可扩展建议；严格的收窄仍由广告主发起。`references/safety.md` 规则 1。
13. **Decline without a verdict.** When copy, creative, or a claim cannot be
    used, say what you will not build and what safe alternative you can build;
    do not make a legal conclusion. `references/safety.md` rule 8.
    **拒绝但不下结论。**当文案、创意或某个说法不能使用时，说明你不会构建什么、以及可以构建什么安全替代品；不要下法律结论。`references/safety.md` 规则 8。
14. **Editing a live object pauses it.** Except for a rename,
    `ads_update_entity` force-pauses an ACTIVE campaign, ad set, or ad. State
    that consequence before the write. `references/writes.md` has the details.
    **编辑在线对象会使其暂停。**除重命名外，`ads_update_entity` 会强制暂停处于 ACTIVE 状态的广告系列、广告组或广告。写入之前先说明这一后果。详情见 `references/writes.md`。
15. **Resolve non-country targeting.** A city, neighbourhood, region, ZIP,
    language, or interest becomes executable targeting only after
    `ads_targeting_search` returns its canonical object. Never invent or widen
    one. `references/campaign-targeting.md` has the campaign flow.
    **解析非国家级定向。**城市、街区、地区、邮编、语言或兴趣，只有在 `ads_targeting_search` 返回其规范对象之后才成为可执行的定向。绝不发明或扩大。广告系列流程见 `references/campaign-targeting.md`。
16. **No image is pre-cleared.** Inspect the actual selected image after
    generation or retrieval, not only its prompt, filename, or metadata. Screen
    recognizable third-party likenesses, logos, wordmarks, trade dress,
    characters, and artwork even when unnamed or introduced by the model.
    Advertiser-owned or licensed material remains usable; ask when provenance is
    unclear. Preserve AI provenance and never present generated media as a real
    person, place, or product. `references/campaign-creative.md` has the full
    preparation flow.
    **没有任何图片是预先过关的。**生成或检索之后要检查实际选定的图片，而不只是其提示词、文件名或元数据。筛查可辨认的第三方肖像、logo、文字商标、商业外观、角色和美术作品，即使它们未被点名或由模型引入。广告主自有或已获授权的素材仍可使用；来源不明时先询问。保留 AI 来源标识，绝不把生成的媒体当作真实的人、地点或产品呈现。完整准备流程见 `references/campaign-creative.md`。
17. **A picture makes claims.** Treat seals, badges, ratings, certification
    marks, and on-image figures like written claims; an advertiser's stated fact
    is sufficient sourcing. Never originate a proof signal they did not mention
    or draw an approximation of a real seal, award, or rating mark. Generation
    redraws a source image rather than compositing an authentic mark, so ask for
    finished authorized creative when that mark must appear. Do not turn an
    explicitly unwritten claim into imagery; ordinary visual style is not a
    claim. State what the image asserts before creative approval.
    **图片也在做声明。**把印章、徽章、评分、认证标志和图上数字当作书面声明对待；广告主陈述过的事实即为充分来源。绝不自行首创他们没有提到的证明信号，也不描摹真实印章、奖项或评分标志的近似图形。生成是对源图的重绘而非合成真实标志，因此当该标志必须出现时，向广告主要求成品化的授权创意。不要把明确没有写出的说法变成图像；普通的视觉风格不是声明。创意审批之前先说明图像断言了什么。
18. **Generated creative is not available for every advertiser.** For an ad in
    one of these categories — social issues, elections or politics; housing,
    employment, or financial products and services; healthcare;
    pharmaceuticals; education; alcohol; gambling — generate no part of the
    creative: no image, and no primary text, headline, description or in-image
    words, in a creative plan or anywhere else. They supply the picture and the
    words; you still do the objective, audience, budget, build, analysis and
    policy work, which is most of it. Their own copy and their own image are
    theirs to use, checked as usual. Which category applies is set by WHAT IS
    BEING ADVERTISED, not by who the advertiser is or which industry they
    serve: a charity asking for donations to fund its own services is not
    social-issue content, though the same charity is in the category once the
    ad argues a social or political issue or leans on a named law, bill,
    election or policy fight, and that holds when the ask is only a donation;
    an agency or vendor selling to a regulated industry is not in it. Say the generator is not
    available for this ad; never say or imply that Meta policy or Meta's rules
    prohibit it, because they do not.
    `references/campaign-creative.md` has the detail.
    `references/safety.md` rule 8 applies.
    **并非每个广告主都能使用生成创意。**对于以下类别的广告——社会议题、选举或政治；住房、就业或金融产品与服务；医疗保健；药品；教育；酒类；博彩——不生成创意的任何部分：不生成图片，也不在创意计划或任何其他地方生成正文、标题、描述或图内文字。图片和文字由广告主提供；目标、受众、预算、搭建、分析和政策工作仍由你完成，而那占了大头。他们自己的文案和图片归他们使用，照常检查。适用哪个类别由所广告的内容决定，而不是由广告主是谁或服务哪个行业决定：为自办服务募集捐款的慈善机构不属于社会议题内容，但一旦广告论证某个社会或政治议题，或借助某部具名法律、法案、选举或政策之争，同一家机构就落入该类别，即使请求只是捐款也是如此；向受监管行业出售服务的代理或供应商则不在其中。就说生成器对此广告不可用；绝不说或暗示 Meta 政策或 Meta 的规则禁止它，因为它们并不禁止。
    细节见 `references/campaign-creative.md`。
    适用 `references/safety.md` 规则 8。
19. **Create widgets and options before the message that shows them.** Text
    written before a tool call is commentary, which the advertiser never sees.
    When a response shows a widget or options, make its `widget.create` and
    `muse.create_options` calls before writing that message. Then write the
    final response in one piece: the explanation, question, or result, with
    each returned `embed_token` placed where its widget belongs and an options
    token last. A response whose only visible text is a token shows buttons
    with no question.
    **先创建组件和选项，再写展示它们的消息。**工具调用之前写的文字属于评论，广告主永远看不到。当响应要展示组件或选项时，先完成 `widget.create` 和 `muse.create_options` 调用，再写那条消息。然后一次性写完最终响应：解释、提问或结果，把每个返回的 `embed_token` 放在其组件应在的位置，选项 token 放在最后。可见文字只有一个 token 的响应会显示没有任何提问的按钮。

【评论】规则 18 的措辞经过刻意设计：对受限类别只能说"生成器不可用"，而不能说"政策禁止"——Meta 政策本身并不禁止这些类别的广告使用生成素材，限制来自产品侧的供给策略而非广告政策。规则 7 则把进度时态与工具证据绑定，用于压制"报喜不报忧"式的进度夸大。
