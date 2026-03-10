

  提供商

  
# OpenCode Zen

本节介绍如何通过 OpenCode Zen 快速接入适合编程智能体（agent）的模型。OpenCode Zen 是 OpenCode 团队精心挑选的模型列表，为编程智能体提供优化过的模型访问路径。作为可选的托管服务，您只需配置 API 密钥并使用 `opencode` 提供商（provider）即可开始。Zen 目前处于测试阶段。

## 命令行设置

```bash
openclaw onboard --auth-choice opencode-zen
# 或非交互式
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## 配置示例

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## 注意事项

- 环境变量 `OPENCODE_ZEN_API_KEY` 也受支持。
- 需要登录 Zen 平台，完善账单信息后获取 API 密钥。
- OpenCode Zen 采用按请求计费模式，具体详情请查看 OpenCode 仪表板。

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)