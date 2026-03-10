

  提供商

  
# OpenAI

本节帮助你配置 OpenAI 作为 OpenClaw 的模型提供商。OpenAI 为 GPT 系列模型提供开发者 API，你可以通过两种方式接入：

- **ChatGPT 登录**：使用现有的 ChatGPT/Codex 订阅，适合已订阅用户
- **API 密钥登录**：按使用量计费，适合需要直接 API 访问的场景

需要注意的是，Codex 云端服务必须使用 ChatGPT 登录，而 Codex CLI 两种方式都支持。OpenAI 官方明确支持在 OpenClaw 这类外部工具中使用订阅 OAuth。

## 方式 A：使用 OpenAI API 密钥

**适用场景**：需要直接 API 访问和按使用量计费时，推荐使用此方式。你可以从 OpenAI 控制台获取 API 密钥。

### CLI 设置

```bash
openclaw onboard --auth-choice openai-api-key
# 或使用非交互模式
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### 配置示例

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

OpenAI 当前的 API 文档列出了 `gpt-5.4` 和 `gpt-5.4-pro` 两个模型，OpenClaw 通过 `openai/*` 路径转发这些模型的请求。

## 方式 B：使用 OpenAI Code (Codex) 订阅

**适用场景**：已有 ChatGPT/Codex 订阅，希望直接使用订阅权益而非单独购买 API 使用量时，推荐使用此方式。

### CLI 设置（Codex OAuth）

```bash
# 在配置向导中选择 Codex OAuth
openclaw onboard --auth-choice openai-codex

# 或直接运行 OAuth 登录
openclaw models auth login --provider openai-codex
```

### 配置示例（Codex 订阅）

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

根据 OpenAI 的 Codex 文档，`gpt-5.4` 是当前的 Codex 模型。在 OpenClaw 中，使用 `openai-codex/gpt-5.4` 来调用通过 ChatGPT/Codex OAuth 认证的模型。

## 传输方式配置

OpenClaw 使用 `pi-ai` 进行模型流式传输。对于 `openai/*` 和 `openai-codex/*` 两种路径，默认传输方式为 `"auto"`（优先使用 WebSocket，失败时回退到 SSE）。

你可以通过设置 `agents.defaults.models.<provider/model>.params.transport` 来自定义传输方式：

- `"sse"`：强制使用 SSE（Server-Sent Events）
- `"websocket"`：强制使用 WebSocket
- `"auto"`：先尝试 WebSocket，失败后回退到 SSE

对于 `openai/*`（即 Responses API），当使用 WebSocket 传输时，OpenClaw 默认启用 WebSocket 预热（`openaiWsWarmup: true`）。相关文档请参考：

- [使用 WebSocket 的 Realtime API](https://platform.openai.com/docs/guides/realtime-websocket)
- [流式 API 响应 (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

## WebSocket 预热

根据 OpenAI 文档，WebSocket 预热是可选功能。但在 OpenClaw 中，我们默认为 `openai/*` 路径启用此功能，以减少首次请求的延迟。

### 禁用预热

如果你不需要预热功能，可以手动关闭：

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### 显式启用预热

虽然预热默认已启用，但如果你想显式声明：

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

## 优先级处理

OpenAI API 支持通过 `service_tier=priority` 来启用优先级处理。在 OpenClaw 中，你可以通过设置 `agents.defaults.models["openai/"].params.serviceTier` 来在直接的 `openai/*` Responses 请求中传递此参数。

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

支持的值包括 `auto`、`default`、`flex` 和 `priority`。

## 服务器端压缩

对于直接使用 OpenAI Responses 的模型（即 `openai/*` 路径，配置为 `api: "openai-responses"` 且 `baseUrl` 指向 `api.openai.com`），OpenClaw 现在会自动注入服务器端压缩的相关参数：

- 自动设置 `store: true`（除非模型兼容性配置了 `supportsStore: false`）
- 注入 `context_management: [{ type: "compaction", compact_threshold: ... }]`

默认情况下，`compact_threshold` 设置为模型 `contextWindow` 的 70%（如果无法获取上下文窗口大小，则默认为 `80000`）。

### 显式启用服务器端压缩

当你需要在兼容的 Responses 模型上强制注入 `context_management` 时（例如 Azure OpenAI Responses），可以使用以下配置：

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### 自定义压缩阈值

你可以根据需要自定义压缩阈值：

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### 禁用服务器端压缩

如果你想关闭此功能：

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

需要注意的是，`responsesServerCompaction` 仅控制 `context_management` 的注入。对于直接的 OpenAI Responses 模型，`store: true` 仍然会被强制设置，除非兼容性配置了 `supportsStore: false`。

## 注意事项

- 模型引用始终使用 `提供商/model` 格式（参见 [/concepts/models](../concepts/models.md)）。
- 身份验证详情和重用规则请参考 [/concepts/oauth](../concepts/oauth.md)。

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)