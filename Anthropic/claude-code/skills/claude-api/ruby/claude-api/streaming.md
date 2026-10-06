<!-- BILINGUAL-EN-ZH -->
# Streaming - Ruby / 流式传输 - Ruby

## Streaming / 流式传输

```ruby
stream = client.messages.stream(
  model: :"claude-opus-5-5",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```

---
