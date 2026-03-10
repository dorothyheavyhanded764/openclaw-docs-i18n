

  提供商

  
# Z.AI

如果你想通过 API 调用 GLM 模型，Z.AI 是一个不错的选择。作为 GLM 模型的官方 API 平台，它提供标准的 REST API 接口，并通过 API 密钥完成身份验证。你只需要在 Z.AI 控制台创建密钥，然后在 OpenClaw 中配置 `zai` 提供商即可开始使用。

## CLI 设置

```bash
openclaw onboard --auth-choice zai-api-key
# 或非交互式设置
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## 配置片段

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事项

-   GLM 模型通过 `zai/` 格式调用，例如 `zai/glm-5`
-   `tool_stream` 默认开启，用于 Z.AI 工具调用的流式传输。如需关闭，请将 `agents.defaults.models["zai/"].params.tool_stream` 设置为 `false`
-   关于 GLM 模型家族的详细介绍，请查看 [/providers/glm](./glm.md)
-   Z.AI 使用 Bearer 认证方式验证你的 API 密钥

[Xiaomi MiMo](./xiaomi.md)