---
name: deep-research
description: Deep research harness — fan-out web searches, fetch sources, adversarially verify claims, synthesize a cited report.
when_to_use: When the user wants a deep, multi-source, fact-checked research report on any topic. BEFORE invoking, check if the question is specific enough to research directly — if underspecified (e.g., "what car to buy" without budget/use-case/region), ask 2-3 clarifying questions to narrow scope. Then pass the refined question as args, weaving the answers in.
disable-model-invocation: true
---
<!-- BILINGUAL-EN-ZH -->

Run the "deep-research" workflow.

运行 "deep-research" 工作流。

Deep research harness — fan-out web searches, fetch sources, adversarially verify claims, synthesize a cited report.

深度研究调度器——扇出多路网络搜索、抓取来源、以对抗方式核查论断、综合出一份带引用的报告。

When the user wants a deep, multi-source, fact-checked research report on any topic. BEFORE invoking, check if the question is specific enough to research directly — if underspecified (e.g., "what car to buy" without budget/use-case/region), ask 2-3 clarifying questions to narrow scope. Then pass the refined question as args, weaving the answers in.

当用户想要就任何主题获得一份深入、多来源、经事实核查的研究报告时使用。调用之前，先检查问题是否足够具体、可以直接开展研究——如果不够明确（例如没有预算/用途/地区限制的"买什么车好"），先提出 2-3 个澄清问题缩小范围，再把精炼后的问题作为 args 传入，并将澄清得到的答案融入其中。

Phases:

阶段：

- Scope: Decompose question (from args) into 5 search angles
  Scope：把问题（来自 args）分解为 5 个搜索角度
- Search: 5 parallel WebSearch agents, one per angle
  Search：5 个并行的 WebSearch 智能体，每个角度一个
- Fetch: URL-dedup, fetch top 15 sources, extract falsifiable claims
  Fetch：URL 去重，抓取排名前 15 的来源，提取可证伪的论断
- Verify: 3-vote adversarial verification per claim (need 2/3 refutes to kill)
  Verify：每条论断经 3 票对抗式核查（需 2/3 票反驳才能否决）
- Synthesize: Merge semantic dupes, rank by confidence, cite sources
  Synthesize：合并语义重复项，按置信度排序，标注来源引用

Invoke: Workflow({ name: "deep-research" })

调用方式：Workflow({ name: "deep-research" })

If the user asks you to modify this workflow or write a new script, load the `workflow-authoring` skill first.

如果用户要求修改此工作流或编写新脚本，请先加载 `workflow-authoring` 技能。

【评论】"每条论断需 2/3 票反驳才被否决"的对抗式核查是该工作流的显著设计：宁可保留存疑论断也不轻易删除，以降低单一搜索代理出错对结论的影响。
