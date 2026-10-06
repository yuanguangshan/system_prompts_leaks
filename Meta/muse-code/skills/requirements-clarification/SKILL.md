---
name: requirements-clarification
user-invocable: false
description: >-
  Clarify a material user-owned product decision before substantive
  implementation commitments. Use only when request and named/governing evidence
  still leave a choice that would materially change scope, platform support, user-facing
  surface, data/API contract, domain values, project placement, or cross-system
  architecture; also use when a truncated request omits such a choice.
  On the initial turn, inspect only named or governing requirements evidence,
  then ask directly without loading: prefer structured input; ask one blocking
  outcome question or at most three coupled questions; wait. After the answer,
  load the body before write or delegation. Do not use for
  implement/build/fix/debug/refactor work when expected behavior is settled,
  plan/design deliverables owned by Plan, complexity/greenfield/integration
  alone, agent-owned technical choices, locally discoverable facts, reversible
  defaults, permission to begin, or stop/no-tool turns. Never re-ask supplied
  facts or turn clarification into refusal.
---

<!-- BILINGUAL-EN-ZH -->
# Requirements Clarification / 需求澄清

Resolve only a remaining user-owned product decision. Do not turn normal
implementation uncertainty into a question.

只解决尚存的、归用户所有的产品决策。不要把正常的实现层面不确定性变成提问。

1. Read the request and any explicitly named or governing requirements source.
   Do not scan broadly for hypothetical ambiguity. If this evidence settles the
   outcome, proceed without asking.
   阅读请求以及任何被明确点名或具有约束力的需求来源。不要宽泛地搜寻假设性的歧义。如果这些证据足以确定结果，直接继续，无需提问。
2. Separate user-owned outcomes and authoritative inputs from agent-owned
   implementation choices. Research discoverable facts yourself. Never ask the
   user to choose a library, framework, or method the agent can determine.
   把归用户所有的结果与权威输入，同归助手所有的实现选择区分开。可自行查明的事实要自己调研。绝不要求用户去挑选库、框架或助手自己能确定的方法。
3. If a narrow reversible default safely satisfies the request, state it and
   proceed. Complexity, greenfield status, file count, or integration work alone
   never requires clarification.
   如果一个范围窄、可回退的默认选择能安全满足请求，说明该选择并继续。复杂度、全新项目状态、文件数量或集成工作本身绝不构成需要澄清的理由。
4. Otherwise ask before substantive commitment. Reads, notes, or one small
   branch-neutral probe are allowed; a scaffold, public contract, embedded domain
   values, or cross-system architecture is a commitment.
   否则，在做出实质承诺之前先提问。读取、记笔记或一次与分支无关的小型探查是允许的；搭建脚手架、确定公开契约、内嵌领域取值或跨系统架构则属于承诺。
5. Prefer `request_user_input` when available. Ask one blocking outcome question;
   group at most three tightly coupled questions. Offer concrete outcome options
   and a recommendation. Never ask merely for permission to begin.
   在可用时优先使用 `request_user_input`。问一个阻塞性的结果问题；至多把三个紧密耦合的问题归为一组。给出具体的结果选项和一项推荐。绝不仅为"获准开始"而提问。
6. After the answer, do not ask again unless later evidence reveals a distinct
   material fork. Apply the answer and continue.
   得到回答后，除非后续证据揭示出另一个不同的重大分叉，否则不再追问。应用该回答并继续。

`plan` owns standalone plan and design deliverables. This skill does not create
an approval gate or stop a specified implementation merely because planning
would be useful. Never re-ask facts already supplied by the request or governing
evidence, and never turn clarification into a refusal.

独立的计划与设计交付物归 `plan` 所有。本技能不设立审批关卡，也不应仅因为"做规划可能有好处"就叫停一项已明确的实现。绝不重复询问请求或权威证据已给出的事实，也绝不要把澄清变成拒答。
【评论】该技能把"何时该问、何时不该问"写成硬边界，用于抑制以澄清为名的过度打断与变相拖延。
