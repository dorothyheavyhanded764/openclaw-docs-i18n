

  提供商

  
# 千帆

千帆是百度的模型即服务（MaaS）平台，通过一个**统一 API** 把请求路由到背后众多模型——只需一个端点和 API 密钥就能访问。它兼容 OpenAI，所以大多数 OpenAI SDK 只需切换 `base URL` 就能直接使用。

## 先决条件

在开始之前，请确保你已具备以下条件：

1.  一个已开通千帆 API 访问权限的百度云账户
2.  从千帆控制台获取的 API 密钥
3.  系统上已安装 OpenClaw

## 获取 API 密钥

按照以下步骤获取你的 API 密钥：

1.  访问 [千帆控制台](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  创建新应用或选择已有应用
3.  生成 API 密钥（格式示例：`bce-v3/ALTAK-...`）
4.  复制 API 密钥，稍后配置 OpenClaw 时使用

## CLI 设置

运行以下命令完成配置：

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## 相关文档

-   [OpenClaw 配置](../gateway/configuration.md)
-   [模型提供商](../concepts/model-providers.md)
-   [智能体设置](../concepts/agent.md)
-   [千帆 API 文档](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)