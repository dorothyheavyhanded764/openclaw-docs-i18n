

  提供商（provider）

  
# Moonshot AI

本节帮助你快速接入 Moonshot 的 Kimi API。Moonshot 提供与 OpenAI 兼容的 API 端点，你可以将默认模型设置为 `moonshot/kimi-k2.5`，或者使用 `kimi-coding/k2p5` 启用 Kimi Coding 服务。

当前可用的 Kimi K2 模型 ID：

- `kimi-k2.5`
- `kimi-k2-0905-preview`
- `kimi-k2-turbo-preview`
- `kimi-k2-thinking`
- `kimi-k2-thinking-turbo`

## 快速配置

使用以下命令启动 Moonshot API 的配置向导：

```bash
openclaw onboard --auth-choice moonshot-api-key
```

如果你需要配置 Kimi Coding，请使用：

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

> **重要提示**：Moonshot 和 Kimi Coding 是两个独立的提供商（provider）。它们的 API 密钥（API key）不能互换，端点地址不同，模型引用方式也有区别——Moonshot 使用 `moonshot/...` 前缀，而 Kimi Coding 使用 `kimi-coding/...` 前缀。

## Moonshot API 配置示例

以下是一个完整的 Moonshot API 配置片段：

```json
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
        // moonshot-kimi-k2-aliases:end
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          // moonshot-kimi-k2-models:end
        ],
      },
    },
  },
}
```

## Kimi Coding 配置示例

如果你使用的是 Kimi Coding 服务，配置会更加简洁：

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: {
        "kimi-coding/k2p5": { alias: "Kimi K2.5" },
      },
    },
  },
}
```

## 注意事项

- **模型引用格式**：Moonshot 模型使用 `moonshot/` 格式，Kimi Coding 模型使用 `kimi-coding/` 格式。
- **自定义定价**：如有需要，可以在 `models.providers` 中覆盖默认的定价和上下文元数据。
- **上下文窗口**：如果 Moonshot 对某个模型发布了不同的上下文限制，请相应调整 `contextWindow` 的值。
- **端点选择**：国际用户使用 `https://api.moonshot.ai/v1`，国内用户使用 `https://api.moonshot.cn/v1`。

[MiniMax](./minimax.md)[Mistral](./mistral.md)