

  提供商

  
# NVIDIA

想使用 NVIDIA 的 Nemotron 和 NeMo 模型？NVIDIA 提供了 OpenAI 兼容的 API 端点 `https://integrate.api.nvidia.com/v1`，只需从 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) 获取 API 密钥（API key）即可开始。

## CLI 设置

只需三步：设置环境变量、跳过认证流程、选择模型：

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

如果还想用 `--token` 参数，请注意它会留在 shell 历史记录和 `ps` 输出里。能用环境变量就尽量用环境变量。

## 配置片段

```json
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## 模型 ID

-   `nvidia/llama-3.1-nemotron-70b-instruct`（默认）
-   `meta/llama-3.3-70b-instruct`
-   `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 注意事项

-   使用 OpenAI 兼容的 `/v1` 端点，API 密钥（API key）需从 NVIDIA NGC 获取。
-   设置 `NVIDIA_API_KEY` 后，提供商（provider）会自动启用。默认配置为 131,072 token 的上下文窗口和 4,096 token 的最大输出长度。

[Mistral](./mistral.md)[Ollama](./ollama.md)