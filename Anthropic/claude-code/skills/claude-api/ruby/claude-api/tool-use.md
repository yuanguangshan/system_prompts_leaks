<!-- BILINGUAL-EN-ZH -->
# Tool Use - Ruby / 工具使用 - Ruby

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概览（工具定义、工具选择、技巧）参见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Use / 工具使用

The Ruby SDK supports tool use via raw JSON schema definitions and also provides a beta tool runner for automatic tool execution.

Ruby SDK 支持通过原始 JSON schema 定义来使用工具，并提供一个 beta 版工具运行器（tool runner）用于自动执行工具。

### Tool Runner (Beta) / 工具运行器（Beta）

```ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"claude-opus-5-5",
  max_tokens: 16000,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
```

### Manual Loop / 手动循环

See the [shared tool use concepts](../../shared/tool-use-concepts.md) for the tool definition format and agentic loop pattern.

工具定义格式与代理式循环模式参见[共享的工具使用概念文档](../../shared/tool-use-concepts.md)。

---

