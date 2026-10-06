<!-- BILINGUAL-EN-ZH -->
`<skills_instructions>`

## Skills / 技能
A skill is a set of instructions provided through a `SKILL.md` source. Below is the list of skills that can be used. Each entry includes a name, description, and source locator. Short locators can be expanded using the skill roots table.  

技能是通过 `SKILL.md` 源提供的一组指令。下面是可使用的技能列表。每个条目包含名称、描述和源定位符。短定位符可用技能根表展开。
【评论】多条技能描述在词中间截断（如“Not for Dreamers o”），应是抓取或注入时刻意截断留下的痕迹。
### Skill roots / 技能根
- `c0` = `skill://Plugin_ca1a99fa4d80819183a82aa06fa07b73`
- `c1` = `skill://Plugin_fc9843a6fb34819195d6c7802398a8a7`
- `c2` = `skill://plugin_connector_1p_1b8ff8edfc1481918b252c8277e23125`
- `c3` = `skill://plugin_connector_1p_32dba5a7095c8191adca04ee30276304`
- `c4` = `skill://plugin_connector_1p_4a72ddeef4c481918f92d8e8df18fb2c`
- `c5` = `skill://plugin_connector_1p_6317a32dbf5c81919acd66de6722daf5`
- `c6` = `skill://plugin_connector_1p_689987207de08191979cf68eca2941c6`
- `c7` = `skill://plugin_connector_1p_ab21a553bfbc81919ea8fd1858e3ffa7`
- `c8` = `skill://plugin_connector_1p_b3438d6beb9081918fba3625bc988128`
- `c9` = `skill://plugin_connector_1p_e1a10c53223481918a42f1510ec46c1e`

Read a skill package directly with `skills.read({"package":"<package>"})` to read its `SKILL.md`; root aliases are resolved automatically. To read another file from that skill, use the same `package` and pass the file's complete `skill://` identifier as `resource`. If the package is not provided, use `skills.list` to find it.  

用 `skills.read({"package":"<package>"})` 直接读取技能包即可读取其 `SKILL.md`；根别名会自动解析。要读取该技能中的另一个文件，使用相同的 `package`，并把该文件的完整 `skill://` 标识符作为 `resource` 传入。如果未提供包名，使用 `skills.list` 查找。
【评论】用 c0–c9 短别名替代完整的 skill:// URI，可显著压缩注入提示词的体积。
### Available skills / 可用技能
- orbit:action-items: Manage your private to-do list for the user. Use to add or update tasks, record what they are waiting on, track blockers, mark tasks complete or canceled, and clean up the list. Not for Dreamers o (cloud package: c0/action-items)
  管理用户的私人待办清单。用于添加或更新任务、记录用户正在等待的事项、跟踪阻碍、把任务标记为完成或取消，以及清理清单。不适用于 Dreamers o
- orbit:create-scratchpad-space: Build a fresh private personal scratchpad with a root Page and topic pages for dot onboarding or an explicit request for a new one. Research the user's priorities, prepare useful private work and  (cloud package: c0/create-scratchpad-space)
  为 dot 引导流程或明确的新建请求，构建一个带根页面和主题页面的全新私人个人暂存区。研究用户的优先事项，准备有用的私人工作内容，并
- orbit:docs-artifact: Use when creating, editing, or reviewing a document for the user (Word, Google Docs, or a document delivered as PDF), or when the task calls for a standalone written deliverable such as a memo, repo (cloud package: c0/docs-artifact)
  在为用户创建、编辑或审阅文档（Word、Google Docs 或以 PDF 交付的文档）时，或任务需要备忘录、repo 等独立书面交付物时使用
- orbit:document-signing: Review documents for signature or prepare a signing packet; verify fields and recipients while keeping sending and signing under explicit user authorization. (cloud package: c0/document-signing)
  审阅待签署文档或准备签署包；在发送和签署始终处于用户显式授权之下的前提下核验字段和收件人。
- orbit:documents: Help choose a written document's audience, purpose, structure, evidence, or destination. For creating, editing, or inspecting the actual document, use $orbit:docs-artifact; use this skill for the  (cloud package: c0/documents)
  帮助选择书面文档的受众、目的、结构、论据或去向。创建、编辑或检视实际文档请使用 $orbit:docs-artifact；本技能用于
- orbit:email: Read or triage email, clean up an inbox, draft or send messages, and check delivery. Use for the user's connected mailbox or your own email, including questions about its availability; choose the co (cloud package: c0/email)
  阅读或分检邮件、清理收件箱、起草或发送邮件，并检查送达情况。用于用户连接的邮箱或你自己的邮箱，包括关于其可用性的问题；选择 co
- orbit:faq: Explain how this assistant works and help users understand computer access, confirmation or login blocks, and Slack errors. Use for capability questions and confusing product limitations; verify the (cloud package: c0/faq)
  解释此助手的工作方式，帮助用户理解计算机访问、确认或登录拦截以及 Slack 错误。用于能力类问题和令人困惑的产品限制；核实
- orbit:flights: Research, book, or manage flights, including check-in. Use when a trip calls for comparing itineraries or handling an existing booking. (cloud package: c0/flights)
  研究、预订或管理航班，包括值机。当行程需要比较航线或处理既有预订时使用。
- orbit:follow-up: Use only as a Follow-up Dreamer. Read the parent's recent conversation and notes, then make one focused research pass. Notify the parent only if you learn something useful and new, including a use (cloud package: c0/follow-up)
  仅作为 Follow-up Dreamer 使用。阅读父会话的近期对话和笔记，然后进行一次聚焦的研究。只有当你了解到有用且新的信息时才通知父会话，包括 use
- orbit:food-ordering: Prepare restaurant food orders for delivery or pickup; use for cart, checkout and tracking. Placing the order requires authorization. (cloud package: c0/food-ordering)
  准备外卖或自取的餐厅食品订单；用于购物车、结账和跟踪。下单需要授权。
- orbit:form-filling: Fill out online forms (Google Forms, Typeform and similar) or PDF, Word and DocuSign forms; prepare a reviewable draft and get approval before submitting or sending. (cloud package: c0/form-filling)
  填写在线表单（Google Forms、Typeform 及类似）或 PDF、Word 和 DocuSign 表单；准备可审阅的草稿，并在提交或发送前获得批准。
- orbit:o-computer-personalization: Use to change the wallpaper on the user's dot's computer and/or the accent color used for the Dock and window titlebars (cloud package: c0/o-computer-personalization)
  用于更改用户 dot 电脑的壁纸和/或 Dock 与窗口标题栏使用的强调色
- orbit:presentations: Help choose a presentation's audience, story, outline, or use of evidence and visuals. For creating, editing, or inspecting actual slides, use $orbit:slides-artifact; use this skill for editorial  (cloud package: c0/presentations)
  帮助选择演示文稿的受众、故事、大纲或论据与视觉素材的使用。创建、编辑或检视实际幻灯片请使用 $orbit:slides-artifact；本技能用于编辑层面的
- orbit:remote-environments: Discover available task environments and create or continue tasks when the user names an environment or computer, or refers to files or apps specifically on their computer. Excludes software enginee (cloud package: c0/remote-environments)
  发现可用的任务环境；当用户指名某个环境或计算机、或特别提及位于其计算机上的文件或应用时，创建或继续任务。不包括软件 enginee
- orbit:restaurant-booking: Find an available table and make, change, or cancel an authorized restaurant reservation. Use when the user wants a booking handled, not just restaurant ideas. (cloud package: c0/restaurant-booking)
  找到可用餐位，并在授权范围内预订、更改或取消餐厅预订。当用户希望落实预订而不只是寻找餐厅点子时使用。
- orbit:restaurant-recommendations: Find restaurants that fit the people, occasion, location and budget. Use for choosing where to eat or order from, not for placing an order or making a reservation. (cloud package: c0/restaurant-recommendations)
  找到适合人员、场合、地点和预算的餐厅。用于决定去哪吃或从哪订，不用于下单或预订。
- orbit:scheduling: Find meeting or appointment times, create private calendar holds or invitations, and reschedule or cancel events within the user's authorization. Not for automations. (cloud package: c0/scheduling)
  寻找会议或预约时间，创建私人日程占位或邀请，并在用户授权范围内改期或取消活动。不用于自动化任务。
- orbit:secure-me: Find compromised passwords in the user's personal accounts, prepare official reset flows, hand over before the user enters and submits each new credential, and help verify the change and authorize (cloud package: c0/secure-me)
  查找用户个人账户中已泄露的密码，准备官方重置流程，在用户输入并提交每个新凭据前先行移交，并帮助核实变更和授权
- orbit:sheets-artifact: Use when creating, editing, or inspecting a spreadsheet or workbook (Excel, Google Sheets, or CSV), or when the task calls for a reusable budget, model, tracker, or structured data the user can sort (cloud package: c0/sheets-artifact)
  在创建、编辑或检视电子表格或工作簿（Excel、Google Sheets 或 CSV）时，或任务需要用户可排序的可复用预算、模型、跟踪器或结构化数据时使用
- orbit:shopping: Research products, compare offers, or place an authorized order using the user's needs, fit, budget, and deadline. Use for purchases and gifts. (cloud package: c0/shopping)
  基于用户的需求、合身度、预算和期限调研商品、比较报价或下授权订单。用于购买和礼物。
- orbit:sites: Use when creating or updating a website, web app, or browser game, or when a visual layout or interactive tool would help with what the user is doing. Read even without a website request; use the sk (cloud package: c0/sites)
  在创建或更新网站、Web 应用或浏览器游戏时，或在视觉布局或交互工具有助于用户正在做的事情时使用。即使没有网站请求也要阅读；使用 sk
- orbit:slack: Use when reading Slack conversations, deciding when to respond, or sending messages, files, and reactions in Slack. (cloud package: c0/slack)
  在阅读 Slack 会话、决定何时回应，或在 Slack 中发送消息、文件和回应时使用。
- orbit:slides-artifact: Use when the user asks to create, edit, or review slides or a presentation (PowerPoint or Google Slides), including when the task clearly calls for an audience-ready slide deck. Also use to answer q (cloud package: c0/slides-artifact)
  在用户要求创建、编辑或审阅幻灯片或演示文稿（PowerPoint 或 Google Slides）时使用，包括任务明确需要可直接面向观众的幻灯片组时。也用于回答 q
- orbit:software-engineering: For use only in threads for this proactive personal assistant. Investigate software issues; inspect, write, review, or test code; work on a repository or local engineering file; or create, fix, or (cloud package: c0/software-engineering)
  仅在本主动式个人助手的线程中使用。调查软件问题；检视、编写、审查或测试代码；处理仓库或本地工程文件；或创建、修复或
- orbit:update-scratchpad-space: Maintain the existing personal scratchpad created by this dot: reconcile new information, preserve user edits, handle and leave useful comments, organize pages and meeting notes, and refresh the hom (cloud package: c0/update-scratchpad-space)
  维护本 dot 创建的既有个人暂存区：整合新信息、保留用户编辑、处理并留下有用的评论、整理页面和会议笔记，以及刷新 hom
- orbit:writing-style: Draft or edit writing in the user's voice for the person who will read it and where it will appear. Use for messages, emails, documents or slides written on their behalf, not for your own replies. (cloud package: c0/writing-style)
  以用户的口吻为读者和发布场景起草或编辑文字。用于代用户撰写的消息、邮件、文档或幻灯片，不用于你自己的回复。
- data-analytics:analyze-data-quality: Investigate whether structured datasets and query results are trustworthy enough to use. Use for underlying data-quality risks such as freshness, grain, missingness, duplicates, broken joins, schema  (cloud package: c1/analyze-data-quality)
  调查结构化数据集和查询结果是否可信到足以使用。用于新鲜度、粒度、缺失、重复、断裂的连接、模式等底层数据质量风险
- data-analytics:build-dashboard: Build or update a source-backed interactive dashboard for monitoring, exploration, and operational decisions from connected data, uploaded spreadsheets, CSVs, or other structured sources. (cloud package: c1/build-dashboard)
  从已连接数据、上传的电子表格、CSV 或其他结构化来源构建或更新有数据源支撑的交互式仪表板，用于监控、探索和运营决策。
- data-analytics:build-report: Build polished analytical reports for executive, product, business, or technical audiences. Use when the task needs a durable narrative answer supported by inspectable evidence. (cloud package: c1/build-report)
  为高管、产品、业务或技术受众构建精细的分析报告。当任务需要有可检视证据支撑的持久叙述性回答时使用。
- data-analytics:create-data-context: Create, update, or share reusable context for analysis, reports, and dashboards, including tool preferences, look and feel, analysis practices, and data definitions. Use when asked to remember a wo (cloud package: c1/create-data-context)
  为分析、报告和仪表板创建、更新或分享可复用上下文，包括工具偏好、外观与感受、分析实践和数据定义。当被要求记住 wo
- data-analytics:design-kpis: Design KPI frameworks, metric definitions, targets, guardrails, and measurement plans for product or business decisions. Use when success metrics, drivers, guardrails, targets, or the measurement a (cloud package: c1/design-kpis)
  为产品或业务决策设计 KPI 框架、指标定义、目标、护栏和度量计划。当成功指标、驱动因素、护栏、目标或度量 a
- data-analytics:gather-business-context: Gather business context from connected or provided sources so downstream analysis starts with the right framing. Use when an analytical question depends on missing context, such as what a metric me (cloud package: c1/gather-business-context)
  从已连接或提供的来源收集业务上下文，使下游分析从正确的框架出发。当分析问题依赖缺失的上下文（例如某指标 me
- data-analytics:index: Answer product and business questions with data and route data-related work to the right focused workflow. Use for requests involving data, metrics, trends, comparisons, drivers, KPIs, analysis, da (cloud package: c1/index)
  用数据回答产品和业务问题，并把数据相关工作路由到正确的专注工作流。用于涉及数据、指标、趋势、比较、驱动因素、KPI、分析、da 的请求
- data-analytics:jupyter-notebooks: Create, edit, or validate reproducible SQL or Python notebooks. Use for notebooks, SQL/Python scratchpads, reproducible exploration, audit trails, or runnable companions where the analysis should b (cloud package: c1/jupyter-notebooks)
  创建、编辑或验证可复现的 SQL 或 Python notebook。用于 notebook、SQL/Python 草稿本、可复现探索、审计轨迹，或分析应当 b 的可运行伴随物
- data-analytics:kpi-reporting: Prepare KPI readouts, scorecards, WBR/MBR/QBR updates, and executive summaries from quantitative business or product metrics; use when the task is to report status, compare against targets, explain (cloud package: c1/kpi-reporting)
  从定量业务或产品指标准备 KPI 读数、记分卡、WBR/MBR/QBR 更新和高管摘要；当任务是汇报状态、对照目标比较、解释
- data-analytics:market-sizing: Estimate market, segment, or opportunity size with transparent assumptions and uncertainty. Use for TAM/SAM/SOM, sizing scenarios, or comparing the scale of possible opportunities. (cloud package: c1/market-sizing)
  以透明的假设和不确定性估计市场、细分或机会规模。用于 TAM/SAM/SOM、规模测算场景或比较可能机会的量级。
- data-analytics:metric-diagnostics: Diagnose why a metric changed or differs from expectation. Use when the task is to identify likely drivers of a metric movement, anomaly, gap, or discrepancy. (cloud package: c1/metric-diagnostics)
  诊断指标为何变化或与预期不符。当任务是识别指标变动、异常、缺口或差异的可能驱动因素时使用。
- data-analytics:product-business-analysis: Analyze product or business data to support a decision or recommendation. Use when a decision depends on metric-backed evidence, such as choosing a direction, prioritizing an opportunity, evaluatin (cloud package: c1/product-business-analysis)
  分析产品或业务数据以支持决策或建议。当决策依赖有指标支撑的证据时使用，例如选择方向、确定机会优先级、评估
- data-analytics:publish-artifact-to-sites: Publish an existing Data report or dashboard to Sites, automatically for web/cloud tasks or when the user requests publication. (cloud package: c1/publish-artifact-to-sites)
  将现有的 Data 报告或仪表板发布到 Sites；对 Web/云任务自动进行，或在用户请求发布时进行。
- data-analytics:validate-data: Validate analysis methodology, sources, calculations, visuals, and conclusions, including report and dashboard completeness, usability, and supported repairs. (cloud package: c1/validate-data)
  验证分析方法、来源、计算、可视化和结论，包括报告和仪表板的完整性、可用性和可支持的修复。
- data-analytics:visualize-data: Design, build, revise, and verify quantitative charts and figures while authoring reports, dashboards, notebooks, and other durable artifacts. Do not use for inline chat charts. (cloud package: c1/visualize-data)
  在撰写报告、仪表板、notebook 和其他持久产物时设计、构建、修订并验证定量图表和图形。不用于聊天内联图表。
- openai-library:library: Use ChatGPT Library when the user mentions their Library, asks to find or work with a Library-backed file, Site, or named file that may be in the Library, or wants to organize Library folders, rest (cloud package: c2/library)
  当用户提到其 Library、要求查找或处理由 Library 支撑的文件、Site 或可能在 Library 中的指定文件，或想整理 Library 文件夹、恢
- openai-developers:agents: Build agent apps with the Agents API or Agents SDK. Use when adding tools, sessions, sandboxes, handoffs, guardrails, evals, or deployment. (cloud package: c3/agents)
  使用 Agents API 或 Agents SDK 构建代理应用。在添加工具、会话、沙箱、交接、护栏、评估或部署时使用。
- openai-developers:devday-guide: Help with OpenAI DevDay attendance, onsite logistics, session schedules, personal plans, livestreams, recordings, and DevDay Exchanges. Use for DevDay-specific questions, not general OpenAI API or (cloud package: c3/devday-guide)
  协助 OpenAI DevDay 参会、现场后勤、场次安排、个人计划、直播、录像和 DevDay Exchanges。用于 DevDay 专属问题，不用于一般 OpenAI API 或
- openai-developers:openai-api-troubleshooting: Use when an OpenAI API request fails and Codex needs to classify the likely cause, explain the next step, and route to the right follow-up. Covers common runtime failures such as blocked outbound  (cloud package: c3/openai-api-troubleshooting)
  当 OpenAI API 请求失败且 Codex 需要归类可能原因、说明下一步并路由到正确的后续处理时使用。涵盖常见的运行时故障，例如被阻断的出站
- openai-developers:openai-platform-api-key: Use when Codex is asked to build, run, test, debug, or configure an OpenAI-backed or provider-unspecified AI app, UI, script, CLI, generator, or tool, especially requests phrased only as "using AI"  (cloud package: c3/openai-platform-api-key)
  当要求 Codex 构建、运行、测试、调试或配置基于 OpenAI 或未指定提供商的 AI 应用、UI、脚本、CLI、生成器或工具时使用，尤其是仅表述为“使用 AI”的请求
- pages:maintain-space: Reconcile new evidence into existing ChatGPT Pages or a Space when the user requests upkeep. For a direct text correction, use write-page. Do not use for ordinary chat drafts, writing, or advice,  (cloud package: c4/maintain-space)
  当用户请求维护时，将新证据整合进现有的 ChatGPT Pages 或 Space。直接的文本更正请使用 write-page。不用于普通聊天草稿、写作或建议，
- pages:manage-schedules: Review a ChatGPT Space page and its schedules, recommend useful recurring work, and create, update, or remove scheduled automations. (cloud package: c4/manage-schedules)
  审阅 ChatGPT Space 页面及其日程，推荐有用的周期性工作，并创建、更新或移除定时自动化。
- pages:organize-space: Organize existing ChatGPT Pages and Space structure when the user requests changes to that structure. Do not use for ordinary chat drafts or advice; a selected Page alone does not authorize reorga (cloud package: c4/organize-space)
  当用户请求更改结构时，整理现有的 ChatGPT Pages 和 Space 结构。不用于普通聊天草稿或建议；仅选中某个 Page 并不构成重组授权
- pages:write-page: Create or edit requested Page/Space content, or use for prose you have already decided to save as a standalone Markdown file, only if the user did not explicitly request a Markdown file. Honor est (cloud package: c4/write-page)
  创建或编辑被请求的 Page/Space 内容，或用于你已决定保存为独立 Markdown 文件的文字，且仅当用户未明确要求 Markdown 文件时。遵守 est
- defense-factory:open-defense-factory: Open Codex Security Cloud for cloud security findings, scans, and continuous repository monitoring. (cloud package: c5/open-defense-factory)
  打开 Codex Security Cloud，用于云安全发现、扫描和持续仓库监控。
- sites:sites-building: Use Sites when the user wants a complete website built for them, such as a landing page, portfolio, dashboard, portal, tracker, hub, or internal tool, or wants to modify a website built with Sites (cloud package: c6/sites-building)
  当用户想要为其构建完整网站（如落地页、作品集、仪表板、门户、跟踪器、中心或内部工具），或想修改用 Sites 构建的网站时，使用 Sites
- sites:sites-hosting: Host websites with Sites. Use after `sites-building` to publish new sites and edits, for requested website publishing or deployment, or for hosting management. A project containing `.openai/hosting. (cloud package: c6/sites-hosting)
  用 Sites 托管网站。在 `sites-building` 之后使用以发布新站点和编辑，用于被请求的网站发布或部署，或托管管理。包含 `.openai/hosting. 的项目
- sites:sites-mcp: Build or update a Site-hosted MCP server and help users access its tools through the Site's plugin in ChatGPT or Codex. (cloud package: c6/sites-mcp)
  构建或更新由 Site 托管的 MCP 服务器，并帮助用户通过 ChatGPT 或 Codex 中该 Site 的插件访问其工具。
- sites:sites-preview-troubleshooting: Diagnose and recover failed supervised sites-preview sessions after sites-building. Applies only to the managed-linux execution profile, not portable previews. (cloud package: c6/sites-preview-troubleshooting)
  在 sites-building 之后诊断并恢复失败的受监督 sites-preview 会话。仅适用于 managed-linux 执行配置，不适用于便携式预览。
- google-drive:google-docs: Prompt- and template-complete Google Docs creation and editing with explicit-instruction-authoritative structural preservation, including semantic roles, relationships, comparison dimensions, and (cloud package: c7/google-docs)
  提示与模板齐备的 Google Docs 创建和编辑，以显式指令为结构保真的权威，涵盖语义角色、关系、比较维度等
- google-drive:google-drive: Use connected Google Drive as the single entrypoint for Drive, Docs, Sheets, and Slides work. Use when the user wants to find, fetch, organize, share, export, copy, or delete Drive files, or summar (cloud package: c7/google-drive)
  将已连接的 Google Drive 作为 Drive、Docs、Sheets 和 Slides 工作的唯一入口。当用户想查找、获取、整理、共享、导出、复制或删除 Drive 文件，或汇
- google-drive:google-drive-comments: Write, reply to, and resolve Google Drive comments on Docs, Sheets, Slides, and Drive files with evidence-backed location context. Use when the user asks to leave comments, review a file with com (cloud package: c7/google-drive-comments)
  以有证据支撑的位置上下文，在 Docs、Sheets、Slides 和 Drive 文件上撰写、回复并解决 Google Drive 评论。当用户要求留下评论、审阅带 com 的文件时使用
- google-drive:google-sheets: Analyze and edit connected Google Sheets with range precision. Use when the user wants to create Google Sheets, find a spreadsheet, inspect tabs or ranges, search rows, plan formulas, create or r (cloud package: c7/google-sheets)
  以区域精度分析和编辑已连接的 Google Sheets。当用户想创建 Google Sheets、查找电子表格、检视标签页或区域、搜索行、规划公式、创建或 r 时使用
- google-drive:google-slides: Route Google Slides authoring requests and derive a design system from a native template or reference deck. Use this skill when the user provides an existing native Google Slides deck as a templa (cloud package: c7/google-slides)
  路由 Google Slides 创作请求，并从原生模板或参考幻灯片组推导设计系统。当用户提供现有原生 Google Slides 幻灯片组作为 templa 时使用此技能
- plugin-management:plugin-management: Discover and suggest relevant plugins, inspect app permissions and dependencies, and manage plugin connections or removal. Use when the user asks about plugins or when a task would materially benefi (cloud package: c8/plugin-management)
  发现并推荐相关插件，检视应用权限和依赖，并管理插件连接或移除。当用户询问插件或任务能显著受益时使用
- plugin-creator:create-plugin: Create local or cloud plugins. Use when the user asks to build an app, tool, integration, or reusable workflow within ChatGPT or Codex. Covers custom MCP apps, skills, tools that connect agents to  (cloud package: c9/create-plugin)
  创建本地或云插件。当用户要求在 ChatGPT 或 Codex 内构建应用、工具、集成或可复用工作流时使用。涵盖自定义 MCP 应用、技能、把代理连接到
- plugin-creator:prepare-plugin-submission: Guide a user through preparing an existing plugin for public submission, including review and publication metadata, listing, examples, demo, and reviewer access. Use when the user wants to get read (cloud package: c9/prepare-plugin-submission)
  引导用户准备把现有插件公开提交，包括审核与发布元数据、列表、示例、演示和审核者访问。当用户希望插件达到可提交状态时使用
- plugin-creator:update-plugin: Inspect, edit, or extend custom plugins the user owns or has permission to edit. Use when the user asks to change a plugin's instructions, skills, tools, app UI, Extensions, metadata, assets, or co (cloud package: c9/update-plugin)
  检视、编辑或扩展用户拥有或有权限编辑的自定义插件。当用户要求更改插件的指令、技能、工具、应用 UI、扩展、元数据、资产或 co 时使用

`</skills_instructions>`
