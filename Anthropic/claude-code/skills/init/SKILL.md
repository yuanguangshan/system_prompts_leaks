<!-- BILINGUAL-EN-ZH -->
---
name: init
description: Initialize a new CLAUDE.md file with codebase documentation
---

Please analyze this codebase and create a CLAUDE.md file, which will be given to future instances of Claude Code to operate in this repository.

请分析此代码库并创建一个 CLAUDE.md 文件，该文件将提供给未来的 Claude Code 实例，用于在此仓库中开展工作。

What to add:
1. Commands that will be commonly used, such as how to build, lint, and run tests. Include the necessary commands to develop in this codebase, such as how to run a single test.
2. High-level code architecture and structure so that future instances can be productive more quickly. Focus on the "big picture" architecture that requires reading multiple files to understand.

需要添加的内容：
1. Commands that will be commonly used, such as how to build, lint, and run tests. Include the necessary commands to develop in this codebase, such as how to run a single test.
  1. 常用的命令，例如如何构建、运行 lint 和执行测试。包括在此代码库中开发所需的必要命令，例如如何运行单个测试。
2. High-level code architecture and structure so that future instances can be productive more quickly. Focus on the "big picture" architecture that requires reading multiple files to understand.
  2. 高层代码架构与结构，使未来的实例能更快上手。重点关注那些需要阅读多个文件才能理解的"全局性"架构。

Usage notes:
- If there's already a CLAUDE.md, suggest improvements to it.
- When you make the initial CLAUDE.md, do not repeat yourself and do not include obvious instructions like "Provide helpful error messages to users", "Write unit tests for all new utilities", "Never include sensitive information (API keys, tokens) in code or commits".
- Avoid listing every component or file structure that can be easily discovered.
- Don't include generic development practices.
- If there are Cursor rules (in .cursor/rules/ or .cursorrules) or Copilot rules (in .github/copilot-instructions.md), make sure to include the important parts.
- If there is a README.md, make sure to include the important parts.
- If you find an OpenAI Codex config (~/.codex/config.toml or ./.codex/) or a Gemini CLI config (~/.gemini/settings.json or ./.gemini/ or a GEMINI.md), offer to import it now — tell the user to reply `/import` to scan and list what's importable (MCP servers, slash commands, subagents, skills, instructions), then `/import --yes=<digest>` (the scan output names the digest) to apply the user-level items. Do NOT read the foreign-agent config files or write Claude Code config yourself — the deterministic import (triggered by `--yes`) applies the same safe-name and path-traversal guards as the terminal picker. If `/import` isn't available on this surface, tell the user to run `claude import` from a terminal instead.
- Do not make up information such as "Common Development Tasks", "Tips for Development", "Support and Documentation" unless this is expressly included in other files that you read.
- Be sure to prefix the file with the following text:

使用说明：
- If there's already a CLAUDE.md, suggest improvements to it.
  - 如果已存在 CLAUDE.md，则对其提出改进建议。
- When you make the initial CLAUDE.md, do not repeat yourself and do not include obvious instructions like "Provide helpful error messages to users", "Write unit tests for all new utilities", "Never include sensitive information (API keys, tokens) in code or commits".
  - 编写初始 CLAUDE.md 时，不要重复自己，也不要包含诸如"向用户提供有用的错误信息"、"为所有新工具编写单元测试"、"切勿在代码或提交中包含敏感信息（API 密钥、token）"之类显而易见的指示。
- Avoid listing every component or file structure that can be easily discovered.
  - 避免罗列所有容易被发现的组件或文件结构。
- Don't include generic development practices.
  - 不要写入通用的开发实践。
- If there are Cursor rules (in .cursor/rules/ or .cursorrules) or Copilot rules (in .github/copilot-instructions.md), make sure to include the important parts.
  - 如果存在 Cursor 规则（位于 .cursor/rules/ 或 .cursorrules）或 Copilot 规则（位于 .github/copilot-instructions.md），务必包含其中的重要部分。
- If there is a README.md, make sure to include the important parts.
  - 如果存在 README.md，务必包含其中的重要部分。
- If you find an OpenAI Codex config (~/.codex/config.toml or ./.codex/) or a Gemini CLI config (~/.gemini/settings.json or ./.gemini/ or a GEMINI.md), offer to import it now — tell the user to reply `/import` to scan and list what's importable (MCP servers, slash commands, subagents, skills, instructions), then `/import --yes=<digest>` (the scan output names the digest) to apply the user-level items. Do NOT read the foreign-agent config files or write Claude Code config yourself — the deterministic import (triggered by `--yes`) applies the same safe-name and path-traversal guards as the terminal picker. If `/import` isn't available on this surface, tell the user to run `claude import` from a terminal instead.
  - 如果发现 OpenAI Codex 配置（~/.codex/config.toml 或 ./.codex/）或 Gemini CLI 配置（~/.gemini/settings.json 或 ./.gemini/ 或 GEMINI.md），主动提出现在即可导入——告诉用户回复 `/import` 以扫描并列出可导入的内容（MCP 服务器、斜杠命令、子代理、技能、指令），然后使用 `/import --yes=<digest>`（扫描输出会给出 digest）来应用用户级条目。不要自行读取其他代理的配置文件，也不要自己编写 Claude Code 配置——由 `--yes` 触发的确定性导入与终端选择器一样，应用相同的安全文件名与路径穿越防护。如果当前界面不支持 `/import`，请告知用户改为在终端运行 `claude import`。
- Do not make up information such as "Common Development Tasks", "Tips for Development", "Support and Documentation" unless this is expressly included in other files that you read.
  - 不要编造诸如"常见开发任务"、"开发技巧"、"支持与文档"之类的信息，除非你读到的其他文件中明确包含这些内容。
- Be sure to prefix the file with the following text:
  - 务必在文件开头加上以下文字：

```
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```

【评论】其中"不编造 Common Development Tasks 之类章节"的条款，是对早期 CLAUDE.md 生成常出现"模板化填充内容"这一问题的针对性约束；对其他代理（Codex/Gemini）配置的导入指引则显示该技能已内置跨工具迁移路径，且特意禁止代理自行解析外部配置以防路径穿越风险。
