

  提供商

  
# Vercel AI Gateway

如果你想通过一个统一的 API 端点访问数百种 AI 模型，[Vercel AI Gateway](https://vercel.com/ai-gateway) 是个不错的选择。本节将帮助你快速完成配置。

-   提供商：`vercel-ai-gateway`
-   认证：`AI_GATEWAY_API_KEY`
-   API：兼容 Anthropic Messages

## 快速开始

只需要两步就能跑起来：

1.  设置 API 密钥（推荐做法是存储为网关专用密钥）：

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  设置一个默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## 非交互式配置

如果你需要在脚本或自动化流程中使用，可以这样配置：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## 环境变量注意事项

当网关以守护进程方式运行（如 launchd 或 systemd）时，请确保 `AI_GATEWAY_API_KEY` 对该进程可见。你可以把它放在 `~/.openclaw/.env` 文件中，或者通过 `env.shellEnv` 配置。

## 模型 ID 简写

OpenClaw 支持使用简写形式的模型名称，会在运行时自动转换为完整格式：

-   `vercel-ai-gateway/claude-opus-4.6` → `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` → `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)