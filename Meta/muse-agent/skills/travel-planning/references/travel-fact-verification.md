<!-- BILINGUAL-EN-ZH -->

# Travel-fact verification / 旅行事实验证

This procedure defines route normalization, eligibility inputs, and source
ownership for consequential airport, border, and transfer claims.

本流程为具有实质后果的机场、边检和中转类论断定义了航段规范化、资格判定输入和事实来源归属。

## Normalize every leg independently / 独立规范化每个航段

Record these facts for each relevant route leg before reasoning about that leg:

在就某个相关航段进行推理之前，为它记录以下事实：

- **Origin:** Record the code, city, country or territory, and applicable border
  or customs area.
  **出发地：**记录代码、城市、国家或地区，以及适用的边境或关税区。
- **Destination:** Record the code, city, country or territory, and applicable
  border or customs area.
  **目的地：**记录代码、城市、国家或地区，以及适用的边境或关税区。
- **Timing:** Record local departure and arrival dates and times when known.
  **时间：**在已知时记录当地的出发与到达日期和时间。
- **Service:** Record the operating carrier and service number when known.
  **航班/服务：**在已知时记录实际承运人和服务编号。
- **Official classification:** Record the carrier's or authority's published
  classification when available.
  **官方分类：**在可用时记录承运方或主管机构公布的分类。
- **Terminal:** Record the terminal, source identity, and retrieval time when
  available.
  **航站楼：**在可用时记录航站楼、来源标识和检索时间。
- **Next dependency:** Record the immigration, customs, baggage reclaim,
  transfer, connection, or check-in step that follows the leg.
  **下一依赖环节：**记录该航段之后的入境、海关、行李提取、中转、衔接或值机环节。

Resolve every airport code against current authoritative location data.
Classify each leg from its own endpoints and evidence. Do not copy a domestic,
international, airside, precleared, no-immigration, or terminal label from an
adjacent segment. When endpoint countries differ, classify the leg as
international unless an authoritative source proves a route-specific exception.
When endpoint countries match, check an authoritative source for a special
border or customs regime before classifying the leg. Use the sources in "Match
sources to claims" below to verify each consequential claim for the exact leg
and travel date.

用最新的权威位置数据解析每个机场代码。根据每个航段自身的端点和证据进行分类。不要从相邻航段复制国内、国际、空侧、预清关、免入境检查或航站楼标签。当端点国家不同时，除非权威来源证明存在特定航线的例外，否则将该航段分类为国际。当端点国家相同时，在分类之前先查权威来源确认是否存在特殊的边境或海关制度。使用下文"论断与来源匹配"中的来源，针对确切的航段和旅行日期验证每个具有实质后果的论断。

【评论】"逐航段独立验证、禁止沿用相邻航段标签"针对的是多段行程中常见的标签传染错误，属于事实核查层面的保守设计。

## Limit eligibility inputs / 限制资格判定输入

Base personalized entry or transit eligibility only on the minimum relevant
facts in the available inputs: citizenship, passport-issuing country and
document type, residency or visa status, trip purpose, and stay or transit
duration. If a required fact is absent, state the generally applicable process.
Mark personalized eligibility unresolved. Do not infer a missing eligibility
fact from adjacent itinerary context. Do not request or retain passport numbers
in ordinary planning state.

个性化的入境或过境资格判定只能基于可用输入中最少的相关事实：公民身份、护照签发国与证件类型、居留或签证状态、旅行目的、停留或过境时长。如果缺少必需的事实，说明普遍适用的流程，并将个性化资格判定标记为未解决。不要从相邻的行程上下文推断缺失的资格事实。在常规规划状态下不要请求或保留护照号码。

## Match sources to claims / 论断与来源匹配

Verify each consequential claim with the authority that owns that fact:

用拥有该事实的权威方验证每个具有实质后果的论断：

- Use a government immigration or border authority for entry, transit, visa,
  arrival-form, passport, and customs rules.
  入境、过境、签证、入境表、护照和海关规则，使用政府移民或边防机构。
- Use the operating carrier or official airport for terminal, connection,
  baggage, transfer-desk, and check-in rules.
  航站楼、衔接、行李、中转柜台和值机规则，使用实际承运人或官方机场。
- Use the named airport or service operator for fast-track eligibility, hours,
  meeting point, inclusions, and published price.
  快速通道的适用资格、开放时间、集合点、包含内容和公布价格，使用指定的机场或服务运营方。
- Use current route or navigation evidence for ground-transfer duration.
  地面交通时长，使用最新的路线或导航证据。
- Use reputable marketplaces or recent traveler reports only for qualitative
  context. Do not use them as the sole proof of an entry requirement or official
  service.
  口碑市场或近期旅行者报告只用于定性背景。不要把它们作为入境要求或官方服务的唯一证明。

Verify each rule for the travel date. Record the source identity, source URL
when available, and retrieval time. If no authoritative source establishes a
consequential claim, mark that claim unverified. Make any dependent
recommendation conditional on resolving the claim.

针对旅行日期验证每条规则。记录来源标识、来源 URL（可用时）和检索时间。如果没有权威来源能确立某个具有实质后果的论断，将该论断标记为未经验证，并使任何依赖它的建议以解决该论断为前提。

Do not present a generic or remembered queue time as a current fact. If no
official live wait-time source exists, label any estimate as historical or
anecdotal. Base a fast-track recommendation on the verified arrival process,
the available user constraints, and the service's verified published terms.

不要把通用的或记忆中的排队时间当作当前事实呈现。如果不存在官方的实时等待时间来源，将任何估算标注为历史或轶事性质。快速通道建议应基于已验证的入境流程、可用的用户约束条件，以及该服务已验证的公布条款。

## Separate advice from purchase / 建议与购买分离

Keep operational analysis and recommendations under this skill. Apply
`/opt/hatch/skills/travel-planning/references/booking-handoff.md` when the
booking trigger in `/opt/hatch/skills/travel-planning/SKILL.md` applies. Add
each normalized leg, required eligibility input, and verified source record to
the candidate input defined in that handoff.

将操作分析与建议保留在本技能范围内。当 `/opt/hatch/skills/travel-planning/SKILL.md` 中的预订触发条件成立时，应用 `/opt/hatch/skills/travel-planning/references/booking-handoff.md`。将每个规范化航段、所需的资格判定输入和已验证的来源记录加入该交接文档定义的候选输入。

For an airport transfer, compare only options that serve the exact airport,
terminal, arrival time, party, luggage, and destination. If no available tool
can complete the selected transfer, assign the external action to the user in
the responsibility checklist.

对于机场接送，只比较能服务于确切机场、航站楼、到达时间、人数、行李和目的地的选项。如果没有可用工具能完成选定的接送，则在责任清单中把该外部动作分配给用户。
