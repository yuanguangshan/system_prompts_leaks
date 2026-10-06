<!-- BILINGUAL-EN-ZH -->
# Token Counting / 令牌计数

Use the `count_tokens` endpoint (`POST /v1/messages/count_tokens`) for accurate
token counts against Claude models. Token counts are **model-specific** - pass
the same model ID you'll use for inference.

要对 Claude 模型获得准确的令牌计数，请使用 `count_tokens` 端点（`POST /v1/messages/count_tokens`）。令牌计数是**因模型而异的**——请传入你推理时将使用的同一个模型 ID。

**Do not use `tiktoken`.** It's OpenAI's tokenizer. It undercounts Claude
tokens by ~15-20% on typical text, and by much more on code or non-English
input. Any estimate from `tiktoken`, `gpt-tokenizer`, or similar is wrong for
Claude.

**不要使用 `tiktoken`。**那是 OpenAI 的分词器。对典型文本，它会把 Claude 的令牌数低估约 15-20%，对代码或非英语输入的低估幅度还要大得多。任何来自 `tiktoken`、`gpt-tokenizer` 或类似工具的估计值，对 Claude 来说都是错误的。

## Count a file or string / 统计文件或字符串

```python
from anthropic import Anthropic

client = Anthropic()
resp = client.messages.count_tokens(
    model="claude-opus-5-5",
    messages=[{"role": "user", "content": open("CLAUDE.md").read()}],
)
print(resp.input_tokens)
```

TypeScript: `await client.messages.countTokens({model, messages})` ->  
`.input_tokens`. See `{lang}/claude-api/README.md` for other SDKs.

TypeScript：`await client.messages.countTokens({model, messages})` ->  
`.input_tokens`。其他 SDK 见 `{lang}/claude-api/README.md`。

## CLI / 命令行

```sh
ant messages count-tokens --model claude-opus-5-5 \
  --message '{role: user, content: "@./CLAUDE.md"}' \
  --transform input_tokens -r
```

## Diffing a file across two versions / 比较同一文件两个版本间的令牌差异

The endpoint is stateless - count each version separately and subtract:

该端点是无状态的——请分别统计每个版本再相减：

```python
from anthropic import Anthropic
import subprocess

client = Anthropic()
def count(text: str) -> int:
    return client.messages.count_tokens(
        model="claude-opus-5-5",
        messages=[{"role": "user", "content": text}],
    ).input_tokens

before = subprocess.check_output(["git", "show", "HEAD:CLAUDE.md"], text=True)
after = open("CLAUDE.md").read()
print(count(after) - count(before))
```

Full docs: see the Token Counting entry in `shared/live-sources.md`.

完整文档：见 `shared/live-sources.md` 中的 Token Counting 条目。

【评论】文档明确告诫不要用 OpenAI 的 tiktoken 估算 Claude 的令牌数：不同厂商分词器差异会带来两位数的计数偏差。这类提示面向的是需要生成正确调用代码的模型或开发者。
