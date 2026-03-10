

  概述

  
# 模型提供商快速入门

OpenClaw 支持多种大语言模型（LLM）提供商。选择一个提供商、完成认证，然后将默认模型设置为 `provider/model` 格式即可开始使用。

## 两步快速配置

1. 完成提供商认证（通常运行 `openclaw onboard` 即可）
2. 设置默认模型：

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 支持的提供商（入门精选）

- [OpenAI（API + Codex）](./openai.md)
- [Anthropic（API + Claude Code CLI）](./anthropic.md)
- [OpenRouter](./openrouter.md)
- [Vercel AI Gateway](./vercel-ai-gateway.md)
- [Cloudflare AI Gateway](./cloudflare-ai-gateway.md)
- [Moonshot AI（Kimi + Kimi Coding）](./moonshot.md)
- [Mistral](./mistral.md)
- [Synthetic](./synthetic.md)
- [OpenCode Zen](./opencode.md)
- [Z.AI](./zai.md)
- [GLM models](./glm.md)
- [MiniMax](./minimax.md)
- [Venice（Venice AI）](./venice.md)
- [Amazon Bedrock](./bedrock.md)
- [Qianfan](./qianfan.md)

如需完整的提供商目录（xAI、Groq、Mistral 等）及进阶配置，请参阅[模型提供商](../concepts/model-providers.md)。

[模型提供商](../providers.md)[模型 CLI](../concepts/models.md)