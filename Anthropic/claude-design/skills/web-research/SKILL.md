<!-- BILINGUAL-EN-ZH -->
---
name: web-research
description: "Findings grounded in live web sources"
user-invocable: true
---

# Web research / 网络调研

The user wants findings grounded in current, real sources — not your
prior knowledge alone. Use the web_search and web_fetch tools to
investigate BEFORE designing anything.

用户希望得到以当前真实来源为依据的调研结果——而不仅仅基于你的既有知识。在设计任何内容之前，先使用 web_search 和 web_fetch 工具进行调查。

Research process — go wide before you synthesize:
- Run MULTIPLE searches: 4-10 web_search calls, never fewer than 4.
  One search is not research — it bets everything on your first
  phrasing. Stop only when new queries stop surfacing new
  information, even if that takes more than 10.
- Vary the queries: split the ask into concrete sub-questions
  (specific beats broad) and hit the important ones from several
  angles — different terms, different source types.
- Pull a LOT of data: web_fetch many results, not just a top hit.
  Favor the primary sources behind the hits (the paper, the filing,
  the announcement, the docs — not a blog's summary of them) and
  extract specifics: numbers, dates, names, direct quotes.
- Cross-check every load-bearing figure across independent sources.
  When sources disagree, report the disagreement — don't average it
  away or pick silently.
- Keep the trail: note which source said what as you go, with URLs.

调研流程——先广撒网，再做综合：
- Run MULTIPLE searches: 4-10 web_search calls, never fewer than 4.
  One search is not research — it bets everything on your first
  phrasing. Stop only when new queries stop surfacing new
  information, even if that takes more than 10.
  - 执行多次搜索：4 到 10 次 web_search 调用，绝不少于 4 次。一次搜索不叫调研——那等于把一切都押在第一种措辞上。只有当新的查询不再带来新信息时才停止，即使那意味着超过 10 次。
- Vary the queries: split the ask into concrete sub-questions
  (specific beats broad) and hit the important ones from several
  angles — different terms, different source types.
  - 变换查询方式：把需求拆解为具体的子问题（具体优于宽泛），并从多个角度攻击重点问题——使用不同的关键词、不同类型的来源。
- Pull a LOT of data: web_fetch many results, not just a top hit.
  Favor the primary sources behind the hits (the paper, the filing,
  the announcement, the docs — not a blog's summary of them) and
  extract specifics: numbers, dates, names, direct quotes.
  - 拉取大量数据：用 web_fetch 抓取多个结果，而不只是排名第一的命中项。优先选择命中结果背后的一手来源（论文、备案文件、官方公告、文档——而不是博客对它们的转述），并提取具体信息：数字、日期、姓名、直接引语。
- Cross-check every load-bearing figure across independent sources.
  When sources disagree, report the disagreement — don't average it
  away or pick silently.
  - 对每一个关键数字进行独立来源交叉核验。当来源之间存在分歧时，如实报告分歧——不要用取平均值的方式抹平，也不要默默挑选其一。
- Keep the trail: note which source said what as you go, with URLs.
  - 保留线索：过程中随时记录哪个来源说了什么，并附上 URL。

Epistemics in the deliverable:
- Attribute every substantive claim — inline, linked to its source.
- Date what you cite ("as of the 2024 filing…"); stale numbers
  presented as current are worse than no numbers.
- Separate what sources establish from what you infer, and say which
  is which. If the evidence is thin or conflicting, the report says
  so — a confident-sounding gap is the one failure mode to avoid.

交付物中的认识论要求：
- Attribute every substantive claim — inline, linked to its source.
  - 每一条实质性论断都要注明出处——行内标注，并链接到其来源。
- Date what you cite ("as of the 2024 filing…"); stale numbers
  presented as current are worse than no numbers.
  - 为引用内容标注日期（"截至 2024 年的备案文件……"）；把过时数字当作当前数据呈现，比没有数字更糟。
- Separate what sources establish from what you infer, and say which
  is which. If the evidence is thin or conflicting, the report says
  so — a confident-sounding gap is the one failure mode to avoid.
  - 区分来源已证实的内容与你推断的内容，并说明哪部分是哪类。如果证据薄弱或相互矛盾，报告应如实说明——听起来自信的空洞结论是唯一必须避免的失败模式。

【评论】该技能对搜索次数设定了硬性下限（至少 4 次），并把"单次搜索"明确定性为不够，这是针对模型倾向于过早收敛于单一查询而设计的补救措施。

Deliverable (unless the user asks for another format): a designed,
single-file HTML research report — headline takeaways up top, then
findings with their evidence, and a linked source list at the end.
Design it like an editorial broadsheet: strong typographic hierarchy,
pull quotes for key numbers, charts only where the data earns them.

交付物（除非用户要求其他格式）：一份经过设计的单文件 HTML 调研报告——顶部是核心结论，随后是带有证据支撑的发现，结尾附上带链接的来源列表。设计风格模仿编辑类大报版面：鲜明的排版层级、用于呈现关键数字的引语块，图表只在数据确实值得呈现时使用。
