<!-- BILINGUAL-EN-ZH -->
# Anthropic — which file is which product? / Anthropic —— 哪个文件对应哪个产品？

**The bare `claude-<model>.md` files in this folder are the claude.ai system prompts** — the prompt served to that model in the Claude web/mobile app (claude.ai). They are not API prompts (the Claude API injects no system prompt) and not Claude Code prompts.

**本文件夹中不带附加后缀的 `claude-<model>.md` 文件就是 claude.ai 的系统提示词** —— 即 Claude 网页/移动应用（claude.ai）向该模型下发的提示词。它们不是 API 提示词（Claude API 不注入系统提示词），也不是 Claude Code 提示词。

【评论】文件名中的 `claude-<model>.md` 是模式写法，`<model>` 为模型名占位符；"API 不注入系统提示词"这一点是区分各类提示词来源的关键背景。

| File pattern | Product |
|---|---|
| `claude-fable-5.md`, `claude-opus-4.8.md`, `claude-sonnet-5.md`, … | **claude.ai** app system prompt for that model |
| `claude-*-no-tools.md` | claude.ai with tools disabled |
| `claude-code/` | Claude Code (the CLI/agent harness) |
| `claude-design.md` | Claude Design |
| `claude-cowork/` | Claude Cowork |
| `claude-for-excel.md`, `claude-for-word.md`, `claude-in-powerpoint.md` | Claude in Microsoft 365 |
| `claude-in-chrome.md` | Claude in Chrome extension |
| `claude-mobile-ios.md` | claude.ai iOS app |
| `anthropic_reminders.md`, `sonnet-4.6-reminders.md`, `research_instructions.md`, `visualize.md` | claude.ai injected fragments (reminders, research, artifacts) |
| `official/` | Prompts Anthropic publishes themselves (release-notes versions — shorter than the real served prompts above) |

| 文件名模式 | 产品 |
|---|---|
| `claude-fable-5.md`, `claude-opus-4.8.md`, `claude-sonnet-5.md`, … | 该模型的 **claude.ai** 应用系统提示词 |
| `claude-*-no-tools.md` | 禁用工具的 claude.ai |
| `claude-code/` | Claude Code（CLI/智能体框架） |
| `claude-design.md` | Claude Design |
| `claude-cowork/` | Claude Cowork |
| `claude-for-excel.md`, `claude-for-word.md`, `claude-in-powerpoint.md` | Microsoft 365 中的 Claude |
| `claude-in-chrome.md` | Claude in Chrome 浏览器扩展 |
| `claude-mobile-ios.md` | claude.ai iOS 应用 |
| `anthropic_reminders.md`, `sonnet-4.6-reminders.md`, `research_instructions.md`, `visualize.md` | claude.ai 注入的片段（提醒、研究、artifacts） |
| `official/` | Anthropic 官方自行发布的提示词（发布说明版本 —— 比上表中实际下发的提示词更短） |
