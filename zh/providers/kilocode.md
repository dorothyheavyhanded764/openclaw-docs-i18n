

  提供商

  
# Kilocode

如果你想用一把密钥访问多家 AI 模型服务商，Kilo Gateway 是个好选择。它提供一个**统一 API**，只需一个端点和一套密钥就能调用多个模型。由于它兼容 OpenAI 格式，大多数 OpenAI SDK 只需改一下 base URL 就能直接使用。

## 获取 API 密钥

1. 访问 [app.kilo.ai](https://app.kilo.ai)
2. 登录或注册账户
3. 进入 API Keys 页面，生成一个新密钥

## 命令行配置

运行以下命令快速完成配置：

```bash
openclaw onboard --kilocode-api-key <key>
```

或者直接设置环境变量：

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## 配置示例

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## 默认模型

默认使用 `kilocode/kilo/auto`，这是一款智能路由模型，会根据任务类型自动选择最合适的底层模型：

- 规划、调试和编排任务 → 路由到 Claude Opus
- 代码编写和探索任务 → 路由到 Claude Sonnet

## 可用模型

OpenClaw 启动时会自动从 Kilo Gateway 获取可用模型列表。输入 `/models kilocode` 可以查看你账户下的完整模型清单。网关上的任何模型都可以用 `kilocode/` 前缀来调用：

```
kilocode/kilo/auto              (默认 - 智能路由)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...还有更多
```

## 注意事项

- 模型引用格式：`kilocode/<model-id>`（例如 `kilocode/anthropic/claude-sonnet-4`）
- 默认模型：`kilocode/kilo/auto`
- Base URL：`https://api.kilo.ai/api/gateway/`
- 想了解更多模型和提供商选项？看这里：[/concepts/model-providers](../concepts/model-providers.md)
- Kilo Gateway 底层使用 Bearer Token 认证（你的 API 密钥就是 Token）

[Hugging Face (推理)](./huggingface.md)[Litellm](./litellm.md)