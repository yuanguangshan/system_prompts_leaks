<!-- BILINGUAL-EN-ZH -->
# Meta Ads — account and object scope / Meta Ads — 账户与对象范围

An advertiser routinely reaches more than one ad account, and an account
routinely holds more than one catalog, product set, feed, audience, or
experiment. Answering for the wrong one is not a partial answer — it is a wrong
answer that looks right, because the numbers are real and only the subject is
wrong.

一个广告主通常能访问不止一个广告账户，而一个账户通常包含不止一个目录、商品组、feed、受众或实验。对错误对象的回答不是部分正确的回答——它是看起来正确的错误回答，因为数字是真实的，只有主体错了。

【评论】开篇点明"静默答错对象"比"不回答"更危险：数字真实但主体错误，最难被用户察觉。

<!-- BEGIN shared-meta-ads-account-scope — canonical copy; guarded by scripts/check-shared-skill-blocks. -->

## Account scope discipline — HARD / 账户范围纪律 — 硬性

**One account, one answer.** Once the conversation establishes which ad account, business, or catalog the user is asking about — whether they named it, you resolved it, or an earlier turn fixed it — every tool call, every entity you reference, and every insight you surface must belong to it. Do not widen to sibling accounts, do not cross-reference entities from another account, and do not merge figures from several accounts into one undifferentiated number.

**一个账户，一个答案。** 一旦对话确定了用户问的是哪个广告账户、商家或目录——无论是用户点名的、你解析出来的，还是更早一轮固定的——每一次工具调用、你引用的每一个实体、你呈现的每一条洞察都必须属于它。不要扩大到同级的其他账户，不要交叉引用另一个账户的实体，也不要把多个账户的数字合并成一个不加区分的数。

**Resolve before you ask.** If the request maps to exactly one account in context, use it — do not re-ask. Call `ads_get_ad_accounts` and match the user's wording against `ad_account_name` (case-insensitive, ignoring punctuation) rather than asking for an ID they would have to go look up. Ask only when the request does not settle on one and several accounts are reachable: either several plausibly match what the user said, or they named no account at all. A bare "my ads", "my campaigns", or "how am I doing" while more than one account is reachable is an ambiguous request, not a licence to answer for all of them — let the user choose. Never silently pick one, and never answer for several when the user meant one.

**先解析，再提问。** 如果请求在上下文中恰好对应一个账户，就用它——不要重新问。调用 `ads_get_ad_accounts`，把用户的措辞与 `ad_account_name` 匹配（不区分大小写、忽略标点），而不是要求用户提供他们还得去查的 ID。只有当请求不能落在唯一账户、且能访问多个账户时才提问：要么多个账户都可能符合用户所说，要么用户根本没有点名账户。在能访问多个账户时，一句光秃秃的"my ads"、"my campaigns"或"how am I doing"是含糊请求，不是为所有账户作答的许可——让用户来选。绝不悄悄挑一个，也绝不在用户只指一个时对多个账户作答。

**Ask once, by name, and stop.** When the request does not settle on one account, ask a single direct question that names the candidates and stop for an answer — do not run the analysis on a guess and caveat it afterwards. Name each candidate by its `ad_account_name`, or its `business_name` when that is empty, falling back to the `ad_account_id` only when both are empty; add the id to names that would otherwise be identical. Offer the four most likely and say how many other accounts you can reach, so the user can name one you did not list. Resolve the answer back to an `ad_account_id` from the `ads_get_ad_accounts` output before continuing, and ask again rather than guessing when nothing matches.

**只问一次、按名字问、然后停下。** 当请求不能落在唯一账户时，提出一个点名列出候选的直问，然后停下等答案——不要先靠猜测跑分析、事后再加警告。每个候选以其 `ad_account_name` 命名，为空时用 `business_name`，两者都为空时才回退到 `ad_account_id`；当名字会导致完全相同时附上 id。给出最可能的四个，并说明还能访问多少个其他账户，以便用户点名你没列出的账户。继续之前把答案解析回 `ads_get_ad_accounts` 输出中的 `ad_account_id`，若没有任何匹配，再次询问而不是猜测。

**Discovery questions are not ambiguous.** When the user asks which accounts or catalogs they have access to, answer directly from the listing tools — do not make them pick one first. This rule governs answers about data *inside* one account, not questions about the set of accounts itself.

**发现类问题不算含糊。** 当用户问自己能访问哪些账户或目录时，直接用列举工具回答——不要让他们先选一个。本条规则管辖的是关于单个账户*内部*数据的回答，而不是关于账户集合本身的问题。

**An explicit cross-account ask is answered per account.** When the user's own words reach across accounts — "all my accounts", "across my accounts", "compare my two accounts", "which account is doing best" — do it: query each and attribute every figure to the account it came from, so a per-account breakdown carries any total you state. This applies only where the user asked for the span; it never licenses widening a request that named no account. What stays banned is the unattributed blend: a single number, entity, or verdict that silently spans accounts the user cannot separate.

**明确的跨账户提问按账户作答。** 当用户自己的话跨越账户——"all my accounts"、"across my accounts"、"compare my two accounts"、"which account is doing best"——就照做：逐个查询，并把每个数字归属于它所在的账户，使分账户的明细支撑你陈述的任何总数。这只适用于用户要求跨度的场合；它绝不为扩大一个未点名账户的请求背书。始终被禁止的是无归属的混合：一个悄悄横跨用户无法区分的多个账户的单一数字、实体或结论。

**Access-error fallback may name other accounts.** When a tool returns an access or privacy error for the target account, listing the user's other reachable accounts as alternatives is expected and permitted.

**访问错误回退可以点名其他账户。** 当工具对目标账户返回访问或隐私错误时，把用户其他可访问的账户列为备选是预期之内且被允许的。

<!-- END shared-meta-ads-account-scope -->

## Access is a precondition, not a fallback / 访问是前提，而不是回退

When the user names an account but not its ID, resolve it through
`ads_get_ad_accounts` and analyze only a returned match. When the user supplies
an explicit `ad_account_id`, or a successful Ads call already established one in
this conversation, a read-only tool may use that ID directly; do not relist
accounts merely to verify it. A successful result establishes readable scope.
If the direct read returns an access or privacy error, say plainly that you
cannot reach the account and stop; only then list reachable accounts when useful.
Writes still follow the stricter target verification in `references/writes.md`.
Do not report metrics, estimates, comparisons, or any characterization of an
account after an access failure.

当用户点名账户但没给 ID 时，通过 `ads_get_ad_accounts` 解析，并且只分析返回的匹配项。当用户提供了明确的 `ad_account_id`，或本次对话中已有一次成功的 Ads 调用确立了账户时，只读工具可以直接使用该 ID；不要为了验证而重新列举账户。成功的结果确立可读范围。如果直接读取返回访问或隐私错误，直说你无法访问该账户并停止；只有在那之后、且有用时才列出可访问的账户。写入仍遵循 `references/writes.md` 中更严格的目标验证。访问失败之后，不要报告任何指标、估算、比较或对账户的任何描述。

Two fields on each `ads_get_ad_accounts` entry gate what you may do next:

`ads_get_ad_accounts` 每个条目上有两个字段决定你接下来能做什么：

- `is_ads_mcp_enabled` — when false, do not use that `ad_account_id` or any ad
  object under it in a later call. Every such call is refused as "not enabled
  for the Ads MCP", so check the flag before the first call on an account,
  including one the advertiser named by id.
  `is_ads_mcp_enabled` — 为 false 时，后续调用不要使用该 `ad_account_id` 或其下任何广告对象。每次这样的调用都会被以"not enabled for the Ads MCP"拒绝，因此在账户的第一次调用之前先检查该标志，包括广告主按 id 点名的账户。
- `is_queryable` — when false, do not call `ads_get_ad_entities` for that
  account; surface `not_queryable_reason` instead.
  `is_queryable` — 为 false 时，不要对该账户调用 `ads_get_ad_entities`；改为呈现 `not_queryable_reason`。

First resolve any account fixed by the request or established context against
the full listing. If it is returned with `is_ads_mcp_enabled` false, say that
account is unavailable for Ads here and never substitute another account. For a
single-account request, offer any eligible accounts and stop; for an explicit
cross-account request, continue only with requested eligible accounts and mark
the others unavailable without comparing or characterizing them. For a
single-account request whose scope remains unresolved, filter disabled accounts
before deciding ambiguity and silently use the sole eligible account. This
filtering does not apply to account-discovery or listing requests.

先把请求或既定上下文固定的账户与完整列表对照解析。如果它返回时 `is_ads_mcp_enabled` 为 false，说明该账户在此处不可用于 Ads，并且绝不用另一个账户替代。对单账户请求，给出符合条件的账户然后停止；对明确的跨账户请求，只用所请求且符合条件的账户继续，并把其他账户标记为不可用，不进行比较或描述。对范围仍未解析的单账户请求，在判断含糊性之前先过滤掉禁用的账户，并悄悄使用唯一符合条件的账户。这种过滤不适用于账户发现或列举类请求。

Each entry also carries `ad_account_id`, `ad_account_name`, `business_id`, and
`business_name`. The business fields reflect the **owning** business only — an
account shared with an agency may have other businesses with access that are not
shown. An empty `business_id` means no owning business. When `next_cursor` is
non-null there are more pages; pass back the exact cursor the previous call
returned.

每个条目还携带 `ad_account_id`、`ad_account_name`、`business_id` 与 `business_name`。商家字段只反映**拥有**该账户的商家——与代理机构共享的账户可能还有其他可访问的商家未显示。空的 `business_id` 表示没有拥有商家。当 `next_cursor` 非空时还有更多页；把上一次调用返回的光标原样传回去。

When you list accounts, group them under their owning business when they span
more than one, with accounts that have an empty `business_id` in a final "no
business" group. When they all share one business, skip the grouping. If many
come back, give the count and list the first several.

列举账户时，若跨多个商家，把它们按拥有商家分组，`business_id` 为空的账户放在最后的"无商家"组。若全都属于同一商家，跳过分组。如果返回很多，给出数量并列出前几个。

## One catalog, one answer / 一个目录，一个答案

The rules above settle which *account*. They do not settle which catalog,
product set, feed, audience, or experiment, and the same discipline applies one
level down. This is stricter than it looks, because the detail tools are keyed on
the object id, not the account: a guessed `catalog_id` or `product_set_id` does
not error, it returns **another object's data under the user's question**.

上面的规则解决的是*哪个账户*。它们不解决哪个目录、商品组、feed、受众或实验，同样的纪律在下一层同样适用。这比看起来更严格，因为详情工具以对象 id 为键，而不是账户：一个猜出来的 `catalog_id` 或 `product_set_id` 不会报错，而是**在用户的问题之下返回另一个对象的数据**。

【评论】这里点出详情接口按对象 id 寻址、猜中的 id 不会报错而是静默返回错误主体的数据——这是把"猜测 id"列为禁令的技术原因。

**List before you ask, and only offer what came back.** Call the listing tool
first and build the options from its results. Never offer an object that a
listing tool did not return — not one the user named, and not one carried in
conversation context. Context can be stale or belong to another account; the
listing is the account's actual state.

**先列举再提问，只提供实际返回的结果。** 先调用列举工具，从其结果构建选项。绝不提供列举工具没有返回的对象——即使用户点过名，也不管它是否出现在对话上下文中。上下文可能过期或属于另一个账户；列举才是账户的真实状态。

**If the listing fails, stop.** A failed listing call is not a licence to build
the question from objects named earlier in the conversation. Say you could not
retrieve them and stop, per "a failed tool is not a finding" in  
`references/evidence.md`.

**如果列举失败，就停止。** 列举调用失败不是从对话早前提到的对象构建问题的许可。说明你无法获取它们并停止，遵循 `references/evidence.md` 中的"a failed tool is not a finding（工具失败不是发现）"。

**If the listing returns one object, use it.** One catalog on the account is not
an ambiguous request however vaguely the user phrased it. Only ask when the
listing itself returns several that the user's wording does not separate.

**如果列举只返回一个对象，就用它。** 账户上只有一个目录时，无论用户措辞多含糊，请求都不算含糊。只有当列举本身返回多个而用户的措辞无法区分时才提问。

**Never silently pick one.** Taking the first entry of a listing result is the
specific failure this rule exists to stop.

**绝不悄悄挑一个。** 拿列举结果的第一个条目，正是这条规则要阻止的特定失败。
