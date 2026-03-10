

  提供商

  
# LiteLLM

[LiteLLM](https://litellm.ai) 是一款开源的 LLM 网关，为 100 多个模型提供商（provider）提供统一的 API。把 OpenClaw 接入 LiteLLM，你可以集中追踪成本、记录日志，还能随时切换后端而不必修改 OpenClaw 的配置。

## 为什么选择 LiteLLM？

接入 LiteLLM 后，你可以获得以下能力：

- **成本追踪** — 精确掌握 OpenClaw 在各模型上的开支明细
- **模型路由** — 无需改配置，即可在 Claude、GPT-4、Gemini、Bedrock 之间自由切换
- **虚拟密钥** — 为 OpenClaw 创建带预算上限的专用密钥
- **日志记录** — 完整的请求/响应日志，排查问题更方便
- **故障转移** — 主提供商（provider）不可用时自动切换备用方案

## 快速上手

### 通过引导流程

```bash
openclaw onboard --auth-choice litellm-api-key
```

### 手动配置

1. 启动 LiteLLM Proxy：

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. 让 OpenClaw 连接到 LiteLLM：

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

就这么简单，OpenClaw 已经通过 LiteLLM 进行路由了。

## 配置详解

### 环境变量

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### 配置文件

在 OpenClaw 的配置文件中添加 `litellm` 提供商（provider）：

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## 虚拟密钥

你可以为 OpenClaw 创建一个带预算限制的专用密钥，方便控制成本：

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

将返回的密钥设置为 `LITELLM_API_KEY` 即可使用。

## 模型路由

LiteLLM 可以把模型请求路由到不同的后端。你只需要在 LiteLLM 的 `config.yaml` 中配置：

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

这样 OpenClaw 只管请求 `claude-opus-4-6`，具体路由由 LiteLLM 负责处理。

## 查看使用情况

你可以通过 LiteLLM 的仪表板或 API 查看用量和支出：

```bash
# 查看密钥信息
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# 查看支出日志
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## 注意事项

- LiteLLM 默认运行在 `http://localhost:4000`
- OpenClaw 通过 OpenAI 兼容的 `/v1/chat/completions` 端点连接
- 所有 OpenClaw 功能都能正常通过 LiteLLM 工作，没有任何限制

## 相关资源

- [LiteLLM 官方文档](https://docs.litellm.ai)
- [模型提供商（provider）](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[GLM 模型](./glm.md)