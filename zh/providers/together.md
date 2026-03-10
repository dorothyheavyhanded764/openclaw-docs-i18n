

  提供商

  
# Together

本节介绍如何配置 Together AI 提供商，让你可以通过统一的 API 接入 Llama、DeepSeek、Kimi 等主流开源模型。

- 提供商：`together`
- 认证：`TOGETHER_API_KEY`
- API：OpenAI 兼容

## 快速开始

1. 设置 API 密钥（API key）。推荐为网关存储密钥：

```bash
openclaw onboard --auth-choice together-api-key
```

2. 设置默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 非交互式配置

如果你需要在脚本或自动化流程中使用，可以这样配置：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

这条命令会将 `together/moonshotai/Kimi-K2.5` 设为默认模型。

## 环境变量注意

当网关以守护进程方式运行时（如 launchd 或 systemd），需要确保 `TOGETHER_API_KEY` 对该进程可用。你可以将它放在 `~/.openclaw/.env` 文件中，或通过 `env.shellEnv` 配置。

## 可用模型

Together AI 提供了丰富的开源模型选择：

- **GLM 4.7 Fp8** — 默认模型，支持 200K 上下文窗口
- **Llama 3.3 70B Instruct Turbo** — 快速高效，擅长指令跟随
- **Llama 4 Scout** — 视觉模型，支持图像理解
- **Llama 4 Maverick** — 高级视觉与推理能力
- **DeepSeek V3.1** — 编码和推理能力出色
- **DeepSeek R1** — 高级推理模型
- **Kimi K2 Instruct** — 高性能模型，支持 262K 上下文窗口

所有模型都支持标准的聊天补全接口，并与 OpenAI API 完全兼容。

[合成](./synthetic.md)[Vercel AI 网关](./vercel-ai-gateway.md)