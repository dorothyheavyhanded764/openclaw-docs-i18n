

  提供商

  
# Ollama

Ollama 是一个本地模型（local model）运行时，让你能够轻松在自己的机器上运行开源大语言模型。OpenClaw 通过 Ollama 的原生 API（`/api/chat`）与其集成，支持流式传输和工具调用。更棒的是，只要你设置了 `OLLAMA_API_KEY`（或认证配置），并且没有显式定义 `models.providers.ollama`，OpenClaw 就能**自动发现支持工具调用的模型**。

> **⚠️** **远程 Ollama 用户请注意**：不要使用 OpenAI 兼容的 `/v1` URL（`http://host:11434/v1`），这会导致工具调用失效，模型可能会把原始工具 JSON 当作普通文本输出。请使用原生 Ollama API URL：`baseUrl: "http://host:11434"`（不要加 `/v1`）。

## 快速开始

只需四步，你就能让 Ollama 在 OpenClaw 中运行起来：

1.  安装 Ollama：[https://ollama.ai](https://ollama.ai)
2.  拉取一个模型：

```bash
ollama pull gpt-oss:20b
# 或
ollama pull llama3.3
# 或
ollama pull qwen2.5-coder:32b
# 或
ollama pull deepseek-r1:32b
```

3.  在 OpenClaw 中启用 Ollama（随便填什么值都行，Ollama 不需要真正的密钥）：

```bash
# 设置环境变量
export OLLAMA_API_KEY="ollama-local"

# 或者在配置文件中设置
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  开始使用 Ollama 模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## 模型自动发现（隐式提供商）

当你设置了 `OLLAMA_API_KEY`（或认证配置），却**没有**定义 `models.providers.ollama` 时，OpenClaw 会自动从本地 `http://127.0.0.1:11434` 的 Ollama 实例中发现可用模型：

-   查询 `/api/tags` 和 `/api/show` 接口获取模型信息
-   只保留声明支持 `tools` 能力的模型
-   当模型报告具有 `thinking` 能力时，自动标记为推理模型
-   从 `model_info[".context_length"]` 读取上下文窗口大小
-   自动设置 `maxTokens` 为上下文窗口的 10 倍
-   所有费用都设为 `0`（毕竟是本地运行）

这样一来，你就不需要手动维护模型列表，OpenClaw 会自动和 Ollama 的能力保持同步。想看看有哪些模型可用？

```bash
ollama list
openclaw models list
```

想添加新模型？直接用 Ollama 拉取就行：

```bash
ollama pull mistral
```

新模型会自动被发现并立即可用。需要注意的是，一旦你显式设置了 `models.providers.ollama`，自动发现就会关闭，你必须手动定义所有模型（详见下文）。

## 配置

### 最简单的方式：环境变量

启用 Ollama 最简单的方法就是设置环境变量：

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 显式配置：手动定义模型

有些场景下，你需要显式配置而非使用自动发现：

-   Ollama 运行在其他主机或端口上
-   你想强制指定特定的上下文窗口大小或模型列表
-   你想使用那些没有报告工具支持的模型

这时候，可以这样配置：

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

如果已经设置了 `OLLAMA_API_KEY` 环境变量，你可以省略配置中的 `apiKey`，OpenClaw 会自动用它来做可用性检查。

### 自定义连接地址

如果你的 Ollama 运行在其他主机或端口上（注意：显式配置会关闭自动发现，所以记得手动定义模型）：

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // 不要加 /v1 - 使用原生 Ollama API URL
        api: "ollama", // 显式设置，确保使用原生工具调用行为
      },
    },
  },
}
```

> **⚠️** 不要在 URL 后面加 `/v1`。`/v1` 路径走的是 OpenAI 兼容模式，工具调用在这个模式下不太靠谱。直接用 Ollama 的基础 URL，不要加任何路径后缀。

### 模型选择

配置完成后，你所有的 Ollama 模型都可以使用了：

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 高级配置

### 推理模型

当 Ollama 在 `/api/show` 接口中报告模型具有 `thinking` 能力时，OpenClaw 会自动将其标记为推理模型：

```bash
ollama pull deepseek-r1:32b
```

### 模型成本

Ollama 完全免费且本地运行，所以所有模型的成本都是 $0。

### 流式传输配置

OpenClaw 的 Ollama 集成默认使用**原生 Ollama API**（`/api/chat`），完美支持流式传输和工具调用同时进行，不需要任何额外配置。

#### 旧版 OpenAI 兼容模式

> **⚠️** **OpenAI 兼容模式下工具调用不可靠。** 只有在你必须使用 OpenAI 格式（比如通过只支持 OpenAI 格式的代理），且不需要原生工具调用行为时，才考虑使用这个模式。

如果你确实需要用 OpenAI 兼容端点（比如代理服务器只支持 OpenAI 格式），可以显式设置 `api: "openai-completions"`：

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // 默认: true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

这个模式可能无法同时支持流式传输和工具调用。如果遇到问题，可以在模型配置里用 `params: { streaming: false }` 关掉流式传输。

另外，当使用 `api: "openai-completions"` 时，OpenClaw 默认会注入 `options.num_ctx` 参数，防止 Ollama 静默回退到 4096 的上下文窗口。如果你的代理或上游服务器不认识这个 `options` 字段，可以这样关掉：

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### 上下文窗口

对于自动发现的模型，OpenClaw 会优先使用 Ollama 报告的上下文窗口大小，如果获取不到就默认用 `8192`。你可以在显式配置中覆盖 `contextWindow` 和 `maxTokens`。

## 故障排除

### Ollama 没被检测到？

确保三件事：Ollama 正在运行、你设置了 `OLLAMA_API_KEY`（或认证配置）、而且你**没有**定义显式的 `models.providers.ollama` 条目。

启动 Ollama：

```bash
ollama serve
```

确认 API 可以访问：

```bash
curl http://localhost:11434/api/tags
```

### 找不到可用模型？

OpenClaw 只会自动发现那些报告支持工具调用的模型。如果你的模型没出现在列表里，要么：

-   拉取一个支持工具调用的模型，或者
-   在 `models.providers.ollama` 里手动定义这个模型

添加模型：

```bash
ollama list  # 看看已安装了哪些
ollama pull gpt-oss:20b  # 拉取一个支持工具调用的模型
ollama pull llama3.3     # 或者其他模型
```

### 连接被拒绝？

检查 Ollama 是否在正确的端口运行：

```bash
# 检查 Ollama 进程
ps aux | grep ollama

# 或者重启 Ollama
ollama serve
```

## 相关文档

-   [模型提供商](../concepts/model-providers.md) - 了解所有可用的提供商（provider）
-   [模型选择](../concepts/models.md) - 如何选择合适的模型
-   [配置参考](../gateway/configuration.md) - 完整的配置文档

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)