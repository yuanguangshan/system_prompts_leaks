<!-- BILINGUAL-EN-ZH -->
# Tool Use - TypeScript / 工具使用 - TypeScript

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概述（工具定义、工具选择、技巧）请参阅 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Runner (Recommended) / 工具运行器（推荐）

**Beta:** The tool runner is in beta in the TypeScript SDK.

**Beta：** 工具运行器在 TypeScript SDK 中处于 beta 阶段。

Use `betaZodTool` with Zod schemas to define tools with a `run` function, then pass them to `client.beta.messages.toolRunner()`:

使用 `betaZodTool` 配合 Zod schema 来定义带 `run` 函数的工具，然后将它们传递给 `client.beta.messages.toolRunner()`：

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = betaZodTool({
  name: "get_weather",
  description: "Get current weather for a location",
  inputSchema: z.object({
    location: z.string().describe("City and state, e.g., San Francisco, CA"),
    unit: z.enum(["celsius", "fahrenheit"]).optional(),
  }),
  run: async (input) => {
    // Your implementation here
    return `72°F and sunny in ${input.location}`;
  },
});

// The tool runner handles the agentic loop and returns the final message
const finalMessage = await client.beta.messages.toolRunner({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: [getWeather],
  messages: [{ role: "user", content: "What's the weather in Paris?" }],
});

console.log(finalMessage.content);
```

Zod is optional - `betaTool()` from `@anthropic-ai/sdk/helpers/beta/json-schema` accepts a raw JSON Schema `inputSchema` plus a `run` function if you don't want a Zod dependency.

Zod 是可选的——如果你不想引入 Zod 依赖，`@anthropic-ai/sdk/helpers/beta/json-schema` 中的 `betaTool()` 接受原始 JSON Schema 形式的 `inputSchema` 加上一个 `run` 函数。

**Key benefits of the tool runner:**

**工具运行器的主要优点：**

- No manual loop - the SDK handles calling tools and feeding results back
  无需手动循环——SDK 负责调用工具并把结果回传
- Type-safe tool inputs via Zod schemas (or raw JSON Schema via `betaTool()`)
  通过 Zod schema（或经由 `betaTool()` 的原始 JSON Schema）实现类型安全的工具输入
- Tool schemas are generated automatically from Zod definitions
  工具 schema 会从 Zod 定义自动生成
- Iteration stops automatically when Claude has no more tool calls
  当 Claude 不再有工具调用时，迭代会自动停止

### Server tools with the tool runner / 配合工具运行器使用服务器工具

The runner's `tools` array accepts raw server-tool definitions (`web_search_20260209`, `web_fetch_20260209`, code execution) alongside runnable tools - pass the literal tool object; server tools run on Anthropic's servers, so there is no `run` function.

运行器的 `tools` 数组在接受可运行工具的同时，也接受原始的服务器工具定义（`web_search_20260209`、`web_fetch_20260209`、代码执行）——直接传入字面量工具对象即可；服务器工具在 Anthropic 的服务器上运行，因此没有 `run` 函数。

**Caution - the runner does not auto-resume `pause_turn` (as of `@anthropic-ai/sdk` 0.110.0).** A long-running server-tool turn can stop with `stop_reason: "pause_turn"`. The runner only continues after a client tool produces a result, so a paused turn ends the loop and is returned as the final message - no error, no warning, just a silently truncated answer. If you mix server tools into the runner, check `stop_reason` on every iteration and resume by pushing the paused assistant turn back:

**注意——运行器不会自动恢复 `pause_turn`（截至 `@anthropic-ai/sdk` 0.110.0）。** 长时间运行的服务器工具轮次可能以 `stop_reason: "pause_turn"` 停止。运行器只会在客户端工具产出结果后才继续，因此被暂停的轮次会终止循环并作为最终消息返回——没有错误、没有警告，只有一个被静默截断的答案。如果你在运行器中混用服务器工具，请在每次迭代时检查 `stop_reason`，并通过把被暂停的 assistant 轮次推回去来恢复：

【评论】此处点明了一个真实的行为缺口：客户端工具与服务器工具混用时，暂停状态不会被自动恢复，属于容易踩中的静默失败场景，文档要求调用方自行检查 `stop_reason`。

```typescript
const params = {
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: [getWeather, { type: "web_search_20260209", name: "web_search", max_uses: 5 }],
  messages: [{ role: "user", content: "Compare this week's forecasts for Paris across two sources" }],
};

const runner = client.beta.messages.toolRunner(params);

// Non-streaming: each iteration yields a complete message
for await (const message of runner) {
  if (message.stop_reason === "pause_turn") {
    runner.pushMessages({ role: "assistant", content: message.content });
  }
}

// Streaming alternative - construct the runner with `stream: true` (same
// params as above). Each iteration then yields a stream, not a message - a
// bare `message.stop_reason` check never fires. Resolve the stream first:
const streamingRunner = client.beta.messages.toolRunner({ ...params, stream: true });
for await (const stream of streamingRunner) {
  const message = await stream.finalMessage();
  if (message.stop_reason === "pause_turn") {
    streamingRunner.pushMessages({ role: "assistant", content: message.content });
  }
}
```

Each pause-resume consumes a `max_iterations` tick, so a capped run can still end paused - check the final message's `stop_reason` before trusting the result (after the loop, call `.done()` on the runner you iterated to get the final message). Alternatively, use the manual loop below, which handles `pause_turn` explicitly.

每次暂停-恢复都会消耗一个 `max_iterations` 名额，因此达到上限的运行仍可能以暂停状态收尾——在信任结果之前请检查最终消息的 `stop_reason`（循环结束后，在你所迭代的运行器上调用 `.done()` 即可获得最终消息）。或者，改用下文的手动循环，它会显式处理 `pause_turn`。

---

## Manual Agentic Loop / 手动智能体循环

Prefer the tool runner above. Drop to a manual loop only when you need control the runner does not expose (e.g., a custom transport, request shapes the SDK cannot build, or avoiding a beta dependency - the runner is beta, and it supports per-token streaming via `stream: true`). Human-in-the-loop approval does *not* require a manual loop - gate inside the tool's `run()` function (return a "user declined" result) or inspect pending `tool_use` blocks and call `setMessagesParams()` between iterations.

优先使用上文的工具运行器。只有当你需要运行器未暴露的控制能力时（例如自定义传输层、SDK 无法构建的请求形态，或想避免 beta 依赖——运行器本身是 beta，并且它通过 `stream: true` 支持逐 token 流式输出），才退回到手动循环。人在回路中的审批*并*不*需要手动循环——可以在工具的 `run()` 函数内部设置门控（返回 "user declined" 结果），或在迭代之间检查待处理的 `tool_use` 块并调用 `setMessagesParams()`。

If you do need a manual loop:

如果你确实需要手动循环：

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const tools: Anthropic.Tool[] = [...]; // Your tool definitions
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];

while (true) {
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    tools: tools,
    messages: messages,
  });

  if (response.stop_reason === "end_turn") break;

  // Server-side tool hit iteration limit; append assistant turn and re-send to continue
  if (response.stop_reason === "pause_turn") {
    messages.push({ role: "assistant", content: response.content });
    continue;
  }

  const toolUseBlocks = response.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );

  messages.push({ role: "assistant", content: response.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

### Streaming Manual Loop / 流式手动循环

Use `client.messages.stream()` + `finalMessage()` instead of `.create()` when you need streaming within a manual loop. Text deltas are streamed on each iteration; `finalMessage()` collects the complete `Message` so you can inspect `stop_reason` and extract tool-use blocks. Set `eager_input_streaming: true` on each tool so large inputs stream as generated; the server then no longer validates them, so validate each parsed input against the tool's schema before running it, stop on `max_tokens` / `refusal`, and catch only the SDK's JSON error (`shared/tool-use-concepts.md` -> Eager input streaming). Schema validation is not path validation: the model-supplied `path` is untrusted output, so confine it to a project root before writing (the text-editor security note in the same file):

在手动循环中需要流式输出时，用 `client.messages.stream()` + `finalMessage()` 代替 `.create()`。每次迭代都会流式传输文本增量；`finalMessage()` 会收集完整的 `Message`，让你可以检查 `stop_reason` 并提取工具使用块。在每个工具上设置 `eager_input_streaming: true`，使大输入在生成的同时被流式传输；此时服务器不再校验这些输入，因此运行前要依据工具 schema 校验每个解析后的输入，在 `max_tokens` / `refusal` 时停止，并且只捕获 SDK 的 JSON 错误（`shared/tool-use-concepts.md` -> Eager input streaming）。schema 校验不等于路径校验：模型提供的 `path` 是不可信输出，写入前必须把它限制在项目根目录之内（见同一文件中的文本编辑器安全注意事项）：

【评论】该段把"eager 流式输入会使服务器端校验失效"与"模型输出的路径必须限制在项目根目录内"两个安全要点并置，强调客户端侧防御不可省略。

```typescript
import Anthropic from "@anthropic-ai/sdk";
import nodePath from "path";
import { z } from "zod";

const client = new Anthropic();
const ROOT = nodePath.resolve(process.cwd());
const WriteFileInput = z.object({ path: z.string(), contents: z.string() });
const tools: Anthropic.Tool[] = [
  {
    name: "write_file",
    description: "Write text to a file at the given path",
    eager_input_streaming: true, // stream large inputs as generated
    input_schema: {
      type: "object",
      properties: { path: { type: "string" }, contents: { type: "string" } },
      required: ["path", "contents"],
    },
  },
];
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];
let jsonRetries = 0;

while (true) {
  const stream = client.messages.stream({
    model: "claude-opus-5-5",
    max_tokens: 64000,
    tools,
    messages,
  });

  // Stream text deltas on each iteration
  stream.on("text", (delta) => {
    process.stdout.write(delta);
  });

  // finalMessage() resolves with the complete Message - no need to
  // manually wire up .on("message") / .on("error") / .on("abort").
  // With eager input streaming it rejects if a tool input could not be
  // parsed at all. Only that case is retried; API errors are rethrown.
  let message: Anthropic.Message;
  try {
    message = await stream.finalMessage();
    jsonRetries = 0; // the cap is on consecutive failures of one turn
  } catch (err) {
    if (err instanceof Anthropic.APIError || jsonRetries++ >= 2) throw err;
    console.error("tool input was not parseable JSON, re-issuing the turn");
    continue;
  }

  if (message.stop_reason === "end_turn") break;
  // A refusal can cut a tool_use off mid-input; never run that turn's tools.
  if (message.stop_reason === "refusal") break;

  // Server-side tool hit iteration limit; append assistant turn and re-send to continue
  if (message.stop_reason === "pause_turn") {
    messages.push({ role: "assistant", content: message.content });
    continue;
  }

  const toolUseBlocks = message.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );
  if (toolUseBlocks.length === 0) break; // other terminal stop

  // A tool input cut off at max_tokens usually parses as a valid partial
  // object; check the stop reason and retry with a higher max_tokens
  // instead of running the tool on truncated input.
  if (message.stop_reason === "max_tokens") {
    throw new Error("tool input truncated (max_tokens); retry with a higher max_tokens");
  }

  messages.push({ role: "assistant", content: message.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    // The SDK's tolerant parser can return a silently truncated input (for
    // example at an unescaped inner quote), so validate before running.
    const parsed = WriteFileInput.safeParse(tool.input);
    if (!parsed.success) {
      toolResults.push({
        type: "tool_result",
        tool_use_id: tool.id,
        is_error: true,
        content: JSON.stringify({ INVALID_JSON: JSON.stringify(tool.input) }),
      });
      continue;
    }
    // `path` is untrusted model output: resolve it and reject anything that
    // escapes the project root (`..`, absolute paths) before the write -
    // schema validation alone does not check this. This check is lexical; if
    // the root contains symlinked directories, canonicalize with fs.realpath
    // too (shared/tool-use-concepts.md -> the text-editor security note).
    const target = nodePath.resolve(ROOT, parsed.data.path);
    const relative = nodePath.relative(ROOT, target);
    if (relative === ".." || relative.startsWith(".." + nodePath.sep) || nodePath.isAbsolute(relative)) {
      toolResults.push({
        type: "tool_result",
        tool_use_id: tool.id,
        is_error: true,
        content: "path escapes the project root",
      });
      continue;
    }
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: await executeTool(tool.name, { ...parsed.data, path: target }),
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

> **Important:** Don't wrap `.on()` events in `new Promise()` to collect the final message - use `stream.finalMessage()` instead. The SDK handles all error/abort/completion states internally.
> **重要：** 不要把 `.on()` 事件包在 `new Promise()` 里来收集最终消息——请改用 `stream.finalMessage()`。SDK 会在内部处理所有错误/中止/完成状态。

> **Error handling in the loop:** Use the SDK's typed exceptions (e.g., `Anthropic.RateLimitError`, `Anthropic.APIError`) - see [Error Handling](./README.md#error-handling) for examples. Don't check error messages with string matching.
> **循环中的错误处理：** 使用 SDK 的类型化异常（例如 `Anthropic.RateLimitError`、`Anthropic.APIError`）——示例见 [Error Handling](./README.md#error-handling)。不要用字符串匹配来检查错误消息。

> **SDK types:** Use `Anthropic.MessageParam`, `Anthropic.Tool`, `Anthropic.ToolUseBlock`, `Anthropic.ToolResultBlockParam`, `Anthropic.Message`, etc. for all API-related data structures. Don't redefine equivalent interfaces.
> **SDK 类型：** 所有 API 相关的数据结构请使用 `Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.ToolUseBlock`、`Anthropic.ToolResultBlockParam`、`Anthropic.Message` 等。不要重新定义等价接口。

---

## Handling Tool Results / 处理工具结果

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: tools,
  messages: [{ role: "user", content: "What's the weather in Paris?" }],
});

for (const block of response.content) {
  if (block.type === "tool_use") {
    const result = await executeTool(block.name, block.input);

    const followup = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 16000,
      tools: tools,
      messages: [
        { role: "user", content: "What's the weather in Paris?" },
        { role: "assistant", content: response.content },
        {
          role: "user",
          content: [
            { type: "tool_result", tool_use_id: block.id, content: result },
          ],
        },
      ],
    });
  }
}
```

---

## Tool Choice / 工具选择

`tool_choice` is `{ type: "auto" }` by default. Forcing a call (`{ type: "any" }` or `{ type: "tool", name: ... }`) returns a 400 on Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and Claude Mythos 5.1; Claude Opus 5, Claude Sonnet 5, and older models accept it. Steer with the prompt instead, and keep the schema guarantee with `strict: true`:

`tool_choice` 默认为 `{ type: "auto" }`。在 Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.1 与 Claude Mythos 5.1 上，强制调用（`{ type: "any" }` 或 `{ type: "tool", name: ... }`）会返回 400；Claude Opus 5、Claude Sonnet 5 及更早的模型则接受该参数。请改为通过提示词进行引导，并用 `strict: true` 保住 schema 层面的保证：

【评论】强制工具调用在较新模型上返回 400 是明显的接口行为变化，依赖旧用法写成的示例代码会因此失效，迁移时需要改用提示词引导加 `strict: true` 的组合。

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: tools.map((tool) => ({ ...tool, strict: true })), // schemas must set additionalProperties: false
  messages: [{ role: "user", content: "What's the weather in Paris? Use the get_weather tool." }],
});
// auto does not guarantee a call - check for a tool_use block and re-prompt if none came back
```

---

## Anthropic-Defined Tools / Anthropic 定义的工具

Version-suffixed `type` literals; `name` is fixed per interface. Web search and code execution are server-executed; bash and text editor are client-executed (you handle the `tool_use` locally - see `shared/tool-use-concepts.md`). Pass plain object literals - the `ToolUnion` type is satisfied structurally. **The `name`/`type` pair must match the interface**: mixing `str_replace_based_edit_tool` (20250728 name) with `text_editor_20250124` (which expects `str_replace_editor`) is a TS2322.

带版本后缀的 `type` 字面量；`name` 按各接口固定。网络搜索与代码执行由服务器执行；bash 与文本编辑器由客户端执行（`tool_use` 由你在本地处理——见 `shared/tool-use-concepts.md`）。直接传入普通对象字面量即可——`ToolUnion` 类型是按结构满足的。**`name`/`type` 组合必须与接口匹配**：把 `str_replace_based_edit_tool`（20250728 的 name）与 `text_editor_20250124`（它期望的是 `str_replace_editor`）混用会得到 TS2322。

**Don't type-annotate as `Tool[]`** - `Tool` is just the custom-tool variant. Let structural typing infer from the `tools` param, or annotate as `Anthropic.Messages.ToolUnion[]` if you must:

**不要把类型注解写成 `Tool[]`**——`Tool` 只是自定义工具的变体。让结构化类型从 `tools` 参数自行推断，必要时才注解为 `Anthropic.Messages.ToolUnion[]`：

```typescript
// Good: let inference work - no annotation
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: [
    { type: "text_editor_20250728", name: "str_replace_based_edit_tool" },
    { type: "bash_20250124", name: "bash" },
    { type: "web_search_20260209", name: "web_search" },
    { type: "code_execution_20260120", name: "code_execution" },
  ],
  messages: [{ role: "user", content: "..." }],
});

// Bad: this is a TS2352 - Tool is the CUSTOM tool variant only
// const tools: Anthropic.Tool[] = [{ type: "text_editor_20250728", ... }]
```

| Interface | `name` | `type` |
|---|---|---|
| `ToolTextEditor20250124` | `str_replace_editor` | `text_editor_20250124` |
| `ToolTextEditor20250429` | `str_replace_based_edit_tool` | `text_editor_20250429` |
| `ToolTextEditor20250728` | `str_replace_based_edit_tool` | `text_editor_20250728` |
| `ToolBash20250124` | `bash` | `bash_20250124` |
| `WebSearchTool20260209` | `web_search` | `web_search_20260209` |
| `WebFetchTool20260209` | `web_fetch` | `web_fetch_20260209` |
| `CodeExecutionTool20260120` | `code_execution` | `code_execution_20260120` |

中文版表格：

| 接口 | `name` | `type` |
|---|---|---|
| `ToolTextEditor20250124` | `str_replace_editor` | `text_editor_20250124` |
| `ToolTextEditor20250429` | `str_replace_based_edit_tool` | `text_editor_20250429` |
| `ToolTextEditor20250728` | `str_replace_based_edit_tool` | `text_editor_20250728` |
| `ToolBash20250124` | `bash` | `bash_20250124` |
| `WebSearchTool20260209` | `web_search` | `web_search_20260209` |
| `WebFetchTool20260209` | `web_fetch` | `web_fetch_20260209` |
| `CodeExecutionTool20260120` | `code_execution` | `code_execution_20260120` |

**Don't mix beta and non-beta types**: if you call `client.beta.messages.create()`, the response `content` is `BetaContentBlock[]` - you cannot pass that to a non-beta `ContentBlockParam[]` without narrowing each element.

**不要混用 beta 与非 beta 类型**：如果调用 `client.beta.messages.create()`，响应的 `content` 是 `BetaContentBlock[]`——不逐个元素做类型收窄的话，无法把它传给非 beta 的 `ContentBlockParam[]`。

---

## Code Execution / 代码执行

### Basic Usage / 基本用法

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content:
        "Calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

### Reading Local Files (ESM note) / 读取本地文件（ESM 说明）

`__dirname` doesn't exist in ES modules. For script-relative paths use `import.meta.url`:

`__dirname` 在 ES 模块中并不存在。脚本相对路径请使用 `import.meta.url`：

```typescript
import { readFileSync } from "fs";
import { fileURLToPath } from "url";
import { dirname, join } from "path";

const __dirname = dirname(fileURLToPath(import.meta.url));
const pdfBytes = readFileSync(join(__dirname, "sample.pdf"));
```

Or use a CWD-relative path if the script runs from a known directory: `readFileSync("./sample.pdf")`.

如果脚本从已知目录运行，也可以使用相对于 CWD 的路径：`readFileSync("./sample.pdf")`。

### Upload Files for Analysis / 上传文件供分析

```typescript
import Anthropic, { toFile } from "@anthropic-ai/sdk";
import { createReadStream } from "fs";

const client = new Anthropic();

// 1. Upload a file
const uploaded = await client.beta.files.upload({
  file: await toFile(createReadStream("sales_data.csv"), undefined, {
    type: "text/csv",
  }),
});

// 2. Pass to code execution
const response = await client.messages.create(
  {
    model: "claude-opus-5-5",
    max_tokens: 16000,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "Analyze this sales data. Show trends and create a visualization.",
          },
          { type: "container_upload", file_id: uploaded.id },
        ],
      },
    ],
    tools: [{ type: "code_execution_20260120", name: "code_execution" }],
  },
);
```

### Retrieve Generated Files / 取回生成的文件

```typescript
import path from "path";
import fs from "fs";

const OUTPUT_DIR = "./claude_outputs";
await fs.promises.mkdir(OUTPUT_DIR, { recursive: true });

for (const block of response.content) {
  if (block.type === "bash_code_execution_tool_result") {
    const result = block.content;
    if (result.type === "bash_code_execution_result" && result.content) {
      for (const fileRef of result.content) {
        if (fileRef.type === "bash_code_execution_output") {
          const metadata = await client.beta.files.retrieveMetadata(
            fileRef.file_id,
          );
          const downloadResponse = await client.beta.files.download(fileRef.file_id);
          const fileBytes = Buffer.from(await downloadResponse.arrayBuffer());
          const safeName = path.basename(metadata.filename);
          if (!safeName || safeName === "." || safeName === "..") {
            console.warn(`Skipping invalid filename: ${metadata.filename}`);
            continue;
          }
          const outputPath = path.join(OUTPUT_DIR, safeName);
          await fs.promises.writeFile(outputPath, fileBytes);
          console.log(`Saved: ${outputPath}`);
        }
      }
    }
  }
}
```

### Container Reuse / 容器复用

```typescript
// First request: set up environment
const response1 = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: "Install tabulate and create data.json with sample user data",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});

// Reuse container
// container is nullable - set only when using server-side code execution
const containerId = response1.container!.id;

const response2 = await client.messages.create({
  container: containerId,
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: "Read data.json and display as a formatted table",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

---

## Memory Tool / 记忆工具

### Basic Usage / 基本用法

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: "Remember that my preferred language is TypeScript.",
    },
  ],
  tools: [{ type: "memory_20250818", name: "memory" }],
});
```

### SDK Memory Helper / SDK 记忆辅助器

Use `betaMemoryTool` with a `MemoryToolHandlers` implementation:

使用 `betaMemoryTool` 配合一个 `MemoryToolHandlers` 实现：

```typescript
import {
  betaMemoryTool,
  type MemoryToolHandlers,
} from "@anthropic-ai/sdk/helpers/beta/memory";

const handlers: MemoryToolHandlers = {
  async view(command) { ... },
  async create(command) { ... },
  async str_replace(command) { ... },
  async insert(command) { ... },
  async delete(command) { ... },
  async rename(command) { ... },
};

const memory = betaMemoryTool(handlers);

const runner = client.beta.messages.toolRunner({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  tools: [memory],
  messages: [{ role: "user", content: "Remember my preferences" }],
});

for await (const message of runner) {
  console.log(message);
}
```

For full implementation examples, use WebFetch:

完整实现示例请使用 WebFetch 获取：

- `https://github.com/anthropics/anthropic-sdk-typescript/blob/main/examples/tools-helpers-memory.ts`

---

## Structured Outputs / 结构化输出

### JSON Outputs (Zod - Recommended) / JSON 输出（Zod - 推荐）

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { z } from "zod";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";

const ContactInfoSchema = z.object({
  name: z.string(),
  email: z.string(),
  plan: z.string(),
  interests: z.array(z.string()),
  demo_requested: z.boolean(),
});

const client = new Anthropic();

const response = await client.messages.parse({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content:
        "Extract: Jane Doe (jane@co.com) wants Enterprise, interested in API and SDKs, wants a demo.",
    },
  ],
  output_config: {
    format: zodOutputFormat(ContactInfoSchema),
  },
});

// parsed_output is null if parsing failed - assert or guard
console.log(response.parsed_output!.name); // "Jane Doe"
```

### Strict Tool Use / 严格工具使用

```typescript
const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: "Book a flight to Tokyo for 2 passengers on March 15",
    },
  ],
  tools: [
    {
      name: "book_flight",
      description: "Book a flight to a destination",
      strict: true,
      input_schema: {
        type: "object",
        properties: {
          destination: { type: "string" },
          date: { type: "string", format: "date" },
          passengers: {
            type: "integer",
            enum: [1, 2, 3, 4, 5, 6, 7, 8],
          },
        },
        required: ["destination", "date", "passengers"],
        additionalProperties: false,
      },
    },
  ],
});
```

---

## Agent Skills / 智能体技能

Enable an Anthropic-managed skill (e.g., `pptx`) via `container.skills` + the `code_execution` tool on the beta path. Both beta headers are required. Outputs land as files in the response content - download by file ID via the Files API.

通过 `container.skills` 加 `code_execution` 工具，在 beta 路径上启用 Anthropic 托管的技能（例如 `pptx`）。两个 beta 头都必须提供。输出会以文件形式出现在响应内容中——凭文件 ID 经 Files API 下载。

```typescript
const response = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  container: {
    skills: [{ type: "anthropic", skill_id: "pptx", version: "latest" }],
  },
  tools: [{ type: "code_execution_20260521", name: "code_execution" }],
  betas: ["code-execution-2025-08-25"],
  messages: [{ role: "user", content: "Create a 3-slide deck about X." }],
});
// Find the file_id in response.content, then:
// await client.beta.files.download(fileId)
```
