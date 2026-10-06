---
name: claude-api-in-prototypes
description: "Call Claude from your HTML artifacts via window.claude.complete"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Claude API in prototypes / 原型中的 Claude API

Your HTML artifacts can call Claude via a built-in helper. No SDK or API key needed.

你的 HTML artifact 可以通过一个内置辅助函数调用 Claude，无需 SDK 或 API 密钥。

```html
<script>
(async () => {
  const text = await window.claude.complete("Summarize this: ...");
  // or with a messages array:
  const text2 = await window.claude.complete({
    messages: [{ role: 'user', content: '...' }],
  });
})();
</script>
```

Calls default to `claude-haiku-4-5` with a 1024-token output cap. The body may also set `model` (haiku/sonnet families only), `max_tokens` (up to 32000), `system`, `tool_choice`, and client `tools` — standard Messages API shapes, except each tool also carries `run: async (input) => string` and the helper executes tool calls in-page and loops (max 8 model calls), resolving with the final text. Handler throws become is_error tool_results. Server tools (web search etc.) are rejected; no streaming; rate-limited 15 calls/minute per user, loop iterations included. Shared artifacts run under the viewer's quota.

调用默认使用 `claude-haiku-4-5`，输出上限为 1024 个 token。请求体还可以设置 `model`（仅限 haiku/sonnet 系列）、`max_tokens`（最高 32000）、`system`、`tool_choice` 以及客户端 `tools`——均为标准 Messages API 结构，区别在于每个工具还携带 `run: async (input) => string`，辅助函数会在页面内执行工具调用并循环（最多 8 次模型调用），最终以最终文本完成兑现。处理函数抛出的异常会变成 is_error 的 tool_result。服务端工具（网页搜索等）会被拒绝；不支持流式传输；每个用户限速每分钟 15 次调用（循环迭代也计入）。共享 artifact 按查看者的配额运行。

【评论】这是面向原型开发的内嵌调用通道：在浏览器 artifact 中暴露 `window.claude.complete` 免去了密钥管理，同时以模型白名单、输出上限、频次限制和"按查看者配额计费"等手段收敛滥用面。
