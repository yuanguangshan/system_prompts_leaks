---
name: "subscription_status"
description: "Answer questions about the user's Muse subscription, plan, usage, tokens, reset timing, or available plans and prices, or verify information that references the Muse subscription."
metadata: { "includeInPrompt": true }
---

<!-- BILINGUAL-EN-ZH -->

# Subscription Status / 订阅状态

Use this skill only when the user directly asks about their Muse subscription, current plan, usage allowance, tokens, reset timing, or available subscription plans and prices, or to verify information that references the Muse subscription.

仅当用户直接询问其 Muse 订阅、当前套餐、使用额度、token 数、重置时间或可用的订阅套餐与价格，或需要核实涉及 Muse 订阅的信息时，才使用此技能。

Run exactly one of these commands:

只运行以下命令中的一个：

- `subscription-status status` for current subscription, usage, reset timing, or credit availability.
  `subscription-status status`：用于查询当前订阅、使用情况、重置时间或额度可用性。
- `subscription-status plans` for the current tier and available plans and prices.
  `subscription-status plans`：用于查询当前档位及可用的套餐与价格。
- `subscription-status overview` when you need both current usage and plan options.
  `subscription-status overview`：当同时需要当前使用情况和套餐选项时使用。

The command output is a safe, agent-facing factual brief. Use it to answer the user naturally; do not simply read the brief aloud or copy its labels mechanically. Do not probe for JSON or other output formats, and do not mention commands, internal services, APIs, caches, field names, or backend status values.

命令输出是一份面向代理的安全事实简报。应使用它自然地回答用户，而不要照本宣科地朗读简报或机械复制其中的标签。不要探测 JSON 或其他输出格式，也不要提及命令、内部服务、API、缓存、字段名或后端状态值。

【评论】"面向代理的事实简报 + 禁止暴露内部命令与字段名"是典型的抽象泄漏防护设计：内部实现细节对用户不可见，回答统一经由受控的事实层。

When verifying information, distinguish what the brief confirms from what it cannot check.

核实信息时，要区分简报能确认的内容与它无法核查的内容。

If exact token counts are requested, explain that Muse reports usage against the subscription allowance rather than an exact token balance.

如果用户要求精确的 token 数量，应解释 Muse 报告的是相对于订阅额度的使用情况，而不是精确的 token 余额。
