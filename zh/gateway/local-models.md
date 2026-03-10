

  协议与 API

  
# 本地模型

本地部署完全可行，但 OpenClaw 对模型有较高要求：需要大上下文窗口和强大的提示注入防御能力。显存较小的显卡会导致上下文被截断，进而削弱安全防护。建议配置：**至少 2 台满配 Mac Studio 或同等算力的 GPU 设备（约 3 万美元以上）**。单张 **24 GB** 显存的显卡只能应付较轻量的提示词负载，延迟也会更高。请务必使用**你能运行的最大/完整尺寸模型**；过度量化或"小型"版本会增加提示注入风险（详见[安全](./security.md)）。

## 推荐方案：LM Studio + MiniMax M2.5（Responses API，完整尺寸）

这是目前最佳的本地部署技术栈。在 LM Studio 中加载 MiniMax M2.5，启用本地服务器（默认地址 `http://127.0.0.1:1234`），并使用 Responses API 让推理过程与最终输出分离。

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**配置清单**

-   安装 LM Studio：[https://lmstudio.ai](https://lmstudio.ai)
-   在 LM Studio 中下载**可用的最大 MiniMax M2.5 版本**（避开"小型"或重度量化的变体），启动服务器后，确认 `http://127.0.0.1:1234/v1/models` 能列出该模型。
-   保持模型常驻内存；冷启动会带来额外的延迟。
-   根据你的 LM Studio 版本调整 `contextWindow` 和 `maxTokens` 参数。
-   用于 WhatsApp 时，建议使用 Responses API，这样只会发送最终的文本输出。

即使主要使用本地模型，也建议保留云端模型的配置，并设置 `models.mode: "merge"`，这样在本地模型不可用时仍有备选方案。

### 混合配置：云端为主，本地为备

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### 本地优先，云端兜底

将上面配置中的主模型和备选模型顺序对调即可。保持相同的 providers 配置和 `models.mode: "merge"`，这样当本地服务不可用时可以自动切换到 Sonnet 或 Opus。

### 区域托管 / 数据路由

-   OpenRouter 上也有托管的 MiniMax/Kimi/GLM 变体，支持区域固定端点（如美国托管）。选择这些区域版本可以让流量留在你指定的司法管辖区内，同时仍可通过 `models.mode: "merge"` 获得 Anthropic/OpenAI 的备选能力。
-   纯本地部署仍是隐私保护最强的方案；当你需要云端模型功能但又想控制数据流向时，区域托管路由是一个折中选择。

## 其他 OpenAI 兼容的本地代理

vLLM、LiteLLM、OAI-proxy 或自建网关，只要能暴露 OpenAI 风格的 `/v1` 端点都可以使用。将上面的 provider 配置块替换为你的端点地址和模型 ID：

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

保持 `models.mode: "merge"`，这样云端模型仍可作为备选方案使用。

## 故障排除

-   网关能访问代理吗？运行 `curl http://127.0.0.1:1234/v1/models` 测试。
-   LM Studio 里的模型被卸载了吗？重新加载——冷启动是导致"卡住"的常见原因。
-   遇到上下文错误？降低 `contextWindow` 或提高服务器的内存限制。
-   安全提醒：本地模型不会经过云端提供商的安全过滤；请将智能体的功能范围收窄，并开启压缩功能以限制提示注入的影响范围。

[CLI 后端](./cli-backends.md)[网络模型](./network-model.md)