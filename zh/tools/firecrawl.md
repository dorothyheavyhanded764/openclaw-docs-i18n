

  内置工具

  
# Firecrawl

当 `web_fetch` 的本地提取器无法处理某些网站时，OpenClaw 可以使用 **Firecrawl** 作为备用方案。Firecrawl 是一个托管的内容提取服务，专门应对两类棘手场景：

- **JavaScript 密集型网站**：内容需要执行 JS 才能加载
- **有反爬机制的页面**：会拦截普通 HTTP 请求

它内置了绕过机器人检测的能力和缓存机制，能有效提升网页内容提取的成功率。

## 获取 API 密钥

1.  注册 Firecrawl 账号并生成 API 密钥
2.  将密钥存入配置文件，或在网关环境变量中设置 `FIRECRAWL_API_KEY`

## 配置 Firecrawl

在配置文件中添加以下内容：

```json
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

几个要点：

-   只要检测到 API 密钥，`firecrawl.enabled` 就会自动启用，无需手动开启
-   `maxAgeMs` 控制缓存的有效时长（单位：毫秒），默认值为 2 天

## 隐身模式与机器人规避

Firecrawl 提供了 **代理模式** 参数来绕过机器人检测，可选值包括 `basic`、`stealth` 和 `auto`。

OpenClaw 在调用 Firecrawl 时会自动使用 `proxy: "auto"` 配合 `storeInCache: true`。`auto` 模式的行为是：先用基础方式尝试，如果失败则自动切换到隐身代理重试。这种机制能提高成功率，但可能会比纯基础模式消耗更多积分。

## web_fetch 的提取优先级

`web_fetch` 会按以下顺序依次尝试提取网页内容：

1.  **Readability**（本地提取，速度快）
2.  **Firecrawl**（如果已配置，处理复杂页面）
3.  **基础 HTML 清理**（最后的兜底方案）

完整的网页工具配置请参阅 [Web 工具](./web.md)。

[执行审批](./exec-approvals.md)[LLM 任务](./llm-task.md)