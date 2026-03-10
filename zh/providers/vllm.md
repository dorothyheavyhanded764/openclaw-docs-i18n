

  提供商

  
# vLLM

想把开源模型跑在本地，还能用 OpenAI 风格的 API 调用？vLLM 就是为此而生的。它可以通过 **OpenAI 兼容**的 HTTP API 来提供开源（以及部分自定义）模型服务。OpenClaw 通过 `openai-completions` API 即可连接到 vLLM。

更方便的是，OpenClaw 还支持**自动发现** vLLM 中的可用模型——只要你设置 `VLLM_API_KEY`（服务器不强制认证的话，随便填个值都行），且没有手动定义 `models.providers.vllm` 配置项，OpenClaw 就会自动拉取模型列表。

## 快速入门

只需三步，就能让 OpenClaw 用上 vLLM 提供的本地模型：

1.  **启动 vLLM 服务**

    确保你的服务地址包含 `/v1` 端点（比如 `/v1/models`、`/v1/chat/completions`）。vLLM 默认监听：

    -   `http://127.0.0.1:8000/v1`

2.  **设置 API Key（启用自动发现）**

    如果服务器没配认证，随便填个值就行：

    ```bash
    export VLLM_API_KEY="vllm-local"
    ```

3.  **选择模型**

    把 `your-model-id` 换成你 vLLM 里实际跑的模型 ID：

    ```json
    {
      agents: {
        defaults: {
          model: { primary: "vllm/your-model-id" },
        },
      },
    }
    ```

## 模型自动发现（隐式提供商）

当你设置了 `VLLM_API_KEY`（或已有认证配置），且**没有**手动定义 `models.providers.vllm` 时，OpenClaw 会自动请求：

-   `GET http://127.0.0.1:8000/v1/models`

然后把返回的模型 ID 转成可用的模型条目。简单说：只要 vLLM 跑起来了，OpenClaw 就能自动识别你加载了哪些模型。

注意：如果你显式定义了 `models.providers.vllm`，自动发现就会被跳过，你需要自己把模型一个个配进去。

## 手动配置（显式定义模型）

有些场景下，自动发现不够用，你需要手动配置：

-   vLLM 跑在别的机器或端口上
-   想固定 `contextWindow` 和 `maxTokens` 的值
-   服务器需要真实的 API Key，或者你想自定义请求头

这时候就手动配置一下：

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "本地 vLLM 模型",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 故障排除

连不上？先检查这些：

-   **确认服务能访问**：

    ```bash
    curl http://127.0.0.1:8000/v1/models
    ```

-   **认证报错？** 设置一个和服务器配置匹配的真实 `VLLM_API_KEY`，或者在 `models.providers.vllm` 里显式配置提供商。

[Venice AI](./venice.md)[小米 MiMo](./xiaomi.md)