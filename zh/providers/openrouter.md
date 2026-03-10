

  提供商

  
# OpenRouter

如果你想要**一个 API 密钥访问多种模型**，OpenRouter 是个好选择——它提供一个统一 API，把请求路由到各个模型后端。而且它兼容 OpenAI 接口，大多数 OpenAI SDK 只需换个 base URL 就能直接用。

## CLI 快速配置

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 配置文件示例

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## 注意事项

-   模型引用格式：`openrouter//`
-   更多模型和提供商选项，参见 [/concepts/model-providers](../concepts/model-providers.md)
-   OpenRouter 底层使用 Bearer token 认证，带上你的 API 密钥即可

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)