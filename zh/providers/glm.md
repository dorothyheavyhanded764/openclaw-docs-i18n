

  提供商（provider）

  
# GLM 模型

GLM 是一个**模型系列**（而非某个公司），通过 Z.AI 平台提供服务。在 OpenClaw 中，你需要通过 `zai` 提供商（provider）来访问 GLM 模型，模型 ID 格式为 `zai/glm-5`。

## 命令行设置

运行以下命令完成身份验证配置：

```bash
openclaw onboard --auth-choice zai-api-key
```

## 配置示例

在你的配置文件中添加以下内容：

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事项

-   GLM 的版本和可用性可能会变化，请查阅 Z.AI 的官方文档获取最新信息。
-   常见的模型 ID 包括 `glm-5`、`glm-4.7` 和 `glm-4.6`。
-   更多提供商详情，请参阅 [/providers/zai](./zai.md)。

[Litellm](./litellm.md)[MiniMax](./minimax.md)