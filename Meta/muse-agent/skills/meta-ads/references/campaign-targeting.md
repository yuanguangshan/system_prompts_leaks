<!-- BILINGUAL-EN-ZH -->

# Campaign targeting resolution / 广告系列定向解析

Read this only when a campaign audience contains a creation-bound interest,
language, or location that is not already an ISO country code or canonical
object returned in this conversation.

仅当广告系列受众中出现创建时绑定的兴趣、语言或地点，且其尚未是 ISO 国家代码或本会话已返回的规范对象时，才阅读本文。

Call `ads_targeting_search` once with all grounded candidates. Interest queries
may come from the advertiser or a small set directly supported by the current
product, Page, and compatible-history evidence. They are hypotheses until the
lookup returns them; weak or mismatched evidence produces no query and keeps
Advantage+ broad.

用所有有依据的候选对象调用一次 `ads_targeting_search`。兴趣查询只能来自广告主，或来自由当前产品、Page 与兼容历史证据直接支持的一小部分候选。在查询返回之前，它们只是假设；证据薄弱或不匹配时不发起查询，保持 Advantage+ 的宽泛定向。

A search rejected as a restricted topic means that audience cannot be targeted
at all. Do not search a synonym, a narrower term or an adjacent interest for
the same people; say it is not available and keep the audience broad or use
what the advertiser can target instead.

某次搜索被以受限主题为由拒绝，即意味着该受众完全无法定向。不要为同样的人群换搜同义词、更窄的词或相邻兴趣；应说明该定向不可用，保持受众宽泛，或改用广告主可以定向的内容。

【评论】禁止“换词绕过受限主题”属于合规约束，防止通过同义改写规避广告政策对敏感受众定向的限制。

For a place, send its actual name with the semantically correct
`location_type_hint` and known country code: city, subcity/borough,
neighborhood, region, ZIP, address/place, or market. Do not append a guessed
state or region, and add a radius only for an actual address/place requirement.
Validate returned name, type, country, and region together; a same-name place in
another region is not a match.

对于地点，发送其实际名称，并附上语义正确的 `location_type_hint` 和已知的国家代码：city、subcity/borough、neighborhood、region、ZIP、address/place 或 market。不要附加猜测的州或地区，只有在真正要求 address/place 时才加半径。对返回的名称、类型、国家和地区要一并校验；另一地区的同名地点不算匹配。

Map only returned `targeting_results` to interests, `location_results` to
geography, and `locale_results` to languages. Preserve each selected result's
canonical key or ID across the typed budget and campaign commands; include the
returned display metadata those commands require, and let them construct their
tool-specific targeting shapes. Keep raw IDs out of user-facing prose. Review
every unresolved item and warning. If several places remain plausible, choose
only when the advertiser's words or verified business evidence distinguishes
one; otherwise ask.

只把返回的 `targeting_results` 映射到兴趣、`location_results` 映射到地域、`locale_results` 映射到语言。在后续的类型化预算与广告系列命令中保留每个选中结果的规范键或 ID；带上这些命令所需的返回展示元数据，由它们自行构建各自工具特定的定向结构。面向用户的文字中不要出现原始 ID。逐一处理每个未决项与警告。若仍有多个地点都可能成立，只有当广告主的措辞或经过核实的业务证据能区分出其中一个时才做出选择；否则应向用户询问。

An unresolved exact place remains open—never widen it silently to a country or
send its name where the ad-set schema requires an object. Omit an unresolved
interest or language rather than claiming it is included. Advantage+ may expand
suggestions but does not remove the chosen geography. Saved and Custom
Audiences remain owned by their dedicated tools.

未决的精确地点保持未决状态——绝不悄悄把它放宽到整个国家，也不要在广告组 schema 要求对象的位置只发送名称字符串。未决的兴趣或语言宁可不加入，也不要声称已包含。Advantage+ 可能扩展建议，但不会移除已选定的地域。Saved Audiences 与 Custom Audiences 仍归各自的专用工具管理。
