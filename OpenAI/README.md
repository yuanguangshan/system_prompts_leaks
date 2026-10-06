<!-- BILINGUAL-EN-ZH -->
# OpenAI — which file is which product? / OpenAI —— 哪个文件对应哪个产品？

**The `gpt-<version>-thinking/instant.md` files are the ChatGPT app system prompts** — what chatgpt.com serves for that model. Files ending `-api.md` are the hidden system messages OpenAI injects on raw API calls (undocumented). `Codex/` is the Codex CLI/agent.

**`gpt-<version>-thinking/instant.md` 文件是 ChatGPT 应用的系统提示词** —— 即 chatgpt.com 为相应模型下发的内容。以 `-api.md` 结尾的文件是 OpenAI 在原始 API 调用中注入的隐藏系统消息（未见于官方文档）。`Codex/` 是 Codex CLI/智能体。

| File pattern | Product |
|---|---|
| `gpt-5.6-sol-extra-high.md`, `gpt-5.5-thinking.md`, `gpt-5.5-instant.md`, … | **ChatGPT** app system prompt for that model |
| `chatgpt-4.5.md`, `chatgpt-atlas.md`, `chatgpt-gpt-5-agent-mode.md` | ChatGPT app (older captures / Atlas browser / agent mode) |
| `gpt-*-api.md` | Hidden system message injected on **API** calls |
| `Codex/` | Codex CLI / coding agent |
| `gpt-4o.md` | ChatGPT 4o (includes the deprecation self-funeral protocol, L226+) |
| `gpt-5-*-personality.md`, `gpt-5.1-*.md` | ChatGPT personality variants |
| `tool-*.md` | ChatGPT tool-specific fragments |
| `Old/` | Superseded versions · `deprecated/` — killed features |

| 文件名模式 | 产品 |
|---|---|
| `gpt-5.6-sol-extra-high.md`, `gpt-5.5-thinking.md`, `gpt-5.5-instant.md`, … | 该模型的 **ChatGPT** 应用系统提示词 |
| `chatgpt-4.5.md`, `chatgpt-atlas.md`, `chatgpt-gpt-5-agent-mode.md` | ChatGPT 应用（较早抓取的版本 / Atlas 浏览器 / 智能体模式） |
| `gpt-*-api.md` | 在 **API** 调用中注入的隐藏系统消息 |
| `Codex/` | Codex CLI / 编程智能体 |
| `gpt-4o.md` | ChatGPT 4o（包含"弃用自我葬礼协议"，自 L226 行起） |
| `gpt-5-*-personality.md`, `gpt-5.1-*.md` | ChatGPT 个性变体 |
| `tool-*.md` | ChatGPT 特定工具的片段 |
| `Old/` | 已被取代的版本 · `deprecated/` —— 已移除的功能 |
