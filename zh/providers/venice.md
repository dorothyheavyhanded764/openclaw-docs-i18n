

  提供商

  
# Venice AI

如果你在寻找一种既能保护隐私、又能使用主流大模型的方案，**Venice AI** 值得关注。它主打隐私优先的 AI 推理服务，支持无审查模型，还可以通过匿名代理访问 Claude、GPT、Gemini 等专有模型。最重要的是，所有推理默认都是私密的——你的数据不会被用于训练，也不会留下日志。

## 为什么选择 Venice？

在 OpenClaw 中使用 Venice，主要有这些优势：

-   **私密推理**：开源模型完全不记录日志，你的对话内容不会被存储
-   **无审查模型**：需要不受内容限制的模型？Venice 提供了专门的选择
-   **匿名访问专有模型**：当你需要 Claude、GPT、Gemini 的质量，但又不想暴露身份时，Venice 的匿名代理可以帮你
-   **OpenAI 兼容**：标准的 `/v1` 端点，接入简单

## 隐私模式详解

Venice 提供两种隐私级别，理解它们的区别能帮你做出正确的模型选择：

| 模式 | 说明 | 模型示例 |
| --- | --- | --- |
| **私密模式** | 完全私密。提示词和响应**从不存储或记录**，用完即销毁 | Llama、Qwen、DeepSeek、Kimi、MiniMax、Venice Uncensored 等 |
| **匿名模式** | 通过 Venice 代理访问，元数据会被剥离。底层提供商（OpenAI、Anthropic、Google、xAI）只能看到匿名请求 | Claude、GPT、Gemini、Grok |

简单来说：追求绝对隐私，选"私密模式"的模型；需要顶级模型能力，选"匿名模式"通过代理访问。

## 功能特性

-   **隐私优先**：在"私密"（完全私密）和"匿名"（代理访问）两种模式间自由选择
-   **无审查模型**：可访问无内容限制的模型
-   **主流模型支持**：通过 Venice 的匿名代理使用 Claude、GPT、Gemini、Grok
-   **OpenAI 兼容 API**：标准 `/v1` 端点，轻松集成
-   **流式输出**：✅ 所有模型均支持
-   **函数调用**：✅ 部分模型支持（请检查模型能力）
-   **视觉能力**：✅ 具备视觉能力的模型支持图像输入
-   **无硬性速率限制**：极端使用可能触发公平使用节流

## 快速开始

### 第一步：获取 API 密钥

1.  前往 [venice.ai](https://venice.ai) 注册账号
2.  进入 **Settings → API Keys → Create new key**
3.  复制你的 API 密钥（格式为 `vapi_xxxxxxxxxxxx`）

### 第二步：配置 OpenClaw

有三种方式可选：

**方式 A：设置环境变量**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**方式 B：交互式配置（推荐）**

```bash
openclaw onboard --auth-choice venice-api-key
```

这个命令会引导你完成配置：
1.  提示输入 API 密钥（或使用已有的 `VENICE_API_KEY` 环境变量）
2.  显示所有可用的 Venice 模型
3.  让你选择默认模型
4.  自动完成提供商配置

**方式 C：非交互式配置**

适合脚本或自动化场景：

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 第三步：验证配置

运行以下命令确认配置成功：

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## 如何选择模型？

配置完成后，OpenClaw 会列出所有可用的 Venice 模型。以下是选择建议：

| 使用场景 | 推荐模型 | 理由 |
| --- | --- | --- |
| **日常对话（默认）** | `kimi-k2-5` | 私密推理能力强，还支持视觉 |
| **追求最高质量** | `claude-opus-4-6` | 匿名模式下最强的选择 |
| **隐私 + 编程** | `qwen3-coder-480b-a35b-instruct` | 私密编程模型，上下文窗口大 |
| **私密视觉任务** | `kimi-k2-5` | 支持图像输入，无需离开私密模式 |
| **快速且省钱** | `qwen3-4b` | 轻量级推理模型，响应快 |
| **复杂私密任务** | `deepseek-v3.2` | 推理能力强，但无 Venice 工具支持 |
| **无内容限制** | `venice-uncensored` | 无内容审查 |

你可以随时更改默认模型：

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

查看所有可用模型：

```bash
openclaw models list | grep venice
```

## 通过 `openclaw configure` 配置

也可以使用配置工具：

1.  运行 `openclaw configure`
2.  选择 **Model/auth**
3.  选择 **Venice AI**

## 可用模型列表（共 41 个）

### 私密模型（26 个）——完全私密，无日志记录

| 模型 ID | 名称 | 上下文窗口 | 特性 |
| --- | --- | --- | --- |
| `kimi-k2-5` | Kimi K2.5 | 256k | 默认，推理，视觉 |
| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | 推理 |
| `llama-3.3-70b` | Llama 3.3 70B | 128k | 通用 |
| `llama-3.2-3b` | Llama 3.2 3B | 128k | 通用 |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | 通用，工具禁用 |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | 推理 |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | 通用 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | 编程 |
| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | 编程 |
| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | 推理，视觉 |
| `qwen3-next-80b` | Qwen3 Next 80B | 256k | 通用 |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (视觉) | 256k | 视觉 |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | 快速，推理 |
| `deepseek-v3.2` | DeepSeek V3.2 | 160k | 推理，工具禁用 |
| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | 无审查，工具禁用 |
| `mistral-31-24b` | Venice Medium (Mistral) | 128k | 视觉 |
| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | 视觉 |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | 通用 |
| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | 通用 |
| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | 推理 |
| `zai-org-glm-4.6` | GLM 4.6 | 198k | 通用 |
| `zai-org-glm-4.7` | GLM 4.7 | 198k | 推理 |
| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | 推理 |
| `zai-org-glm-5` | GLM 5 | 198k | 推理 |
| `minimax-m21` | MiniMax M2.1 | 198k | 推理 |
| `minimax-m25` | MiniMax M2.5 | 198k | 推理 |

### 匿名模型（15 个）——通过 Venice 代理访问

| 模型 ID | 名称 | 上下文窗口 | 特性 |
| --- | --- | --- | --- |
| `claude-opus-4-6` | Claude Opus 4.6 (通过 Venice) | 1M | 推理，视觉 |
| `claude-opus-4-5` | Claude Opus 4.5 (通过 Venice) | 198k | 推理，视觉 |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 (通过 Venice) | 1M | 推理，视觉 |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 (通过 Venice) | 198k | 推理，视觉 |
| `openai-gpt-54` | GPT-5.4 (通过 Venice) | 1M | 推理，视觉 |
| `openai-gpt-53-codex` | GPT-5.3 Codex (通过 Venice) | 400k | 推理，视觉，编程 |
| `openai-gpt-52` | GPT-5.2 (通过 Venice) | 256k | 推理 |
| `openai-gpt-52-codex` | GPT-5.2 Codex (通过 Venice) | 256k | 推理，视觉，编程 |
| `openai-gpt-4o-2024-11-20` | GPT-4o (通过 Venice) | 128k | 视觉 |
| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (通过 Venice) | 128k | 视觉 |
| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (通过 Venice) | 1M | 推理，视觉 |
| `gemini-3-pro-preview` | Gemini 3 Pro (通过 Venice) | 198k | 推理，视觉 |
| `gemini-3-flash-preview` | Gemini 3 Flash (通过 Venice) | 256k | 推理，视觉 |
| `grok-41-fast` | Grok 4.1 Fast (通过 Venice) | 1M | 推理，视觉 |
| `grok-code-fast-1` | Grok Code Fast 1 (通过 Venice) | 256k | 推理，编程 |

## 模型发现机制

当设置了 `VENICE_API_KEY` 环境变量后，OpenClaw 会自动从 Venice API 获取可用模型列表。如果 API 暂时无法访问，会回退到内置的静态模型目录。`/models` 端点是公开的（无需认证即可查看模型列表），但推理请求需要有效的 API 密钥。

## 流式输出与工具支持

| 特性 | 支持情况 |
| --- | --- |
| **流式输出** | ✅ 所有模型 |
| **函数调用** | ✅ 大多数模型支持（查看 API 中的 `supportsFunctionCalling` 字段） |
| **视觉/图像** | ✅ 标注了"视觉"特性的模型 |
| **JSON 模式** | ✅ 通过 `response_format` 参数支持 |

## 定价说明

Venice 采用积分计费制。具体费率请查看 [venice.ai/pricing](https://venice.ai/pricing)：

-   **私密模型**：通常费用较低
-   **匿名模型**：与直接调用原厂 API 价格相近，外加少量 Venice 服务费

## Venice 匿名模式 vs 直接调用 API

| 对比项 | Venice 匿名模式 | 直接调用 API |
| --- | --- | --- |
| **隐私保护** | 元数据被剥离，完全匿名 | 与你的账户绑定 |
| **响应延迟** | 增加 10-50ms（代理转发） | 直连 |
| **功能支持** | 支持大部分功能 | 完整功能 |
| **计费方式** | Venice 积分 | 原厂计费 |

## 使用示例

```bash
# 使用默认私密模型
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"

# 通过 Venice 匿名访问 Claude Opus
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"

# 使用无审查模型
openclaw agent --model venice/venice-uncensored --message "Draft options"

# 使用视觉模型分析图像
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# 使用编程专用模型
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## 常见问题

### API 密钥未被识别

检查环境变量和模型列表：

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

确认密钥格式正确，应以 `vapi_` 开头。

### 模型显示不可用

Venice 的模型目录会动态更新。运行 `openclaw models list` 查看当前可用模型。部分模型可能因维护暂时下线。

### 连接失败

Venice API 地址为 `https://api.venice.ai/api/v1`，请确认你的网络可以访问 HTTPS。

## 配置文件示例

```json
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

## 相关链接

-   [Venice AI 官网](https://venice.ai)
-   [API 文档](https://docs.venice.ai)
-   [定价页面](https://venice.ai/pricing)
-   [服务状态](https://status.venice.ai)

[Vercel AI Gateway](./vercel-ai-gateway.md)[vLLM](./vllm.md)