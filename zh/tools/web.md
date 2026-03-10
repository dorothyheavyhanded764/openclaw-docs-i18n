

  内置工具

  
# 网页工具

想让你的智能体（agent）具备联网能力？OpenClaw 内置了两个轻量级网页工具：

- `web_search` — 网页搜索。支持 Perplexity Search API、Brave Search API、Gemini（带 Google 搜索联网）、Grok 或 Kimi。
- `web_fetch` — 网页获取（fetch）。发起 HTTP 请求并提取可读内容（HTML → Markdown/文本）。

这两个工具适合大多数常规网页操作场景。但请注意，它们**不是**浏览器自动化工具——如果你的目标网站重度依赖 JavaScript 渲染，或者需要登录才能访问内容，请改用[浏览器工具](./browser.md)。

## 工作原理

在深入配置细节之前，先了解一下这两个工具的工作方式：

- `web_search` 会调用你配置好的搜索服务商 API，返回搜索结果。
- 搜索结果会按查询词缓存 15 分钟（可在配置中调整）。
- `web_fetch` 执行普通的 HTTP GET 请求，然后提取页面的可读内容（HTML → Markdown/文本）。它**不会**执行 JavaScript。
- `web_fetch` 默认启用，除非你显式禁用它。

各服务商的具体配置方法，请参阅 [Perplexity 搜索设置](../perplexity.md) 和 [Brave 搜索设置](../brave-search.md)。

## 如何选择搜索服务商

下表帮你快速对比各服务商的特点：

| 服务商 | 优点 | 缺点 | API 密钥 |
| --- | --- | --- | --- |
| **Perplexity Search API** | 响应快，结果结构化；支持域名、语言、地区和时效性过滤；自带内容提取 | — | `PERPLEXITY_API_KEY` |
| **Brave Search API** | 响应快，结果结构化 | 过滤选项较少；受 AI 使用条款限制 | `BRAVE_API_KEY` |
| **Gemini** | Google 搜索联网，AI 综合回答 | 需要 Gemini API 密钥 | `GEMINI_API_KEY` |
| **Grok** | xAI 联网回答 | 需要 xAI API 密钥 | `XAI_API_KEY` |
| **Kimi** | Moonshot 网页搜索能力 | 需要 Moonshot API 密钥 | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### 自动检测机制

如果你没有在配置中显式指定 `provider`，OpenClaw 会根据环境中可用的 API 密钥自动选择服务商，检测顺序如下：

1. **Brave** — 检查 `BRAVE_API_KEY` 环境变量或 `tools.web.search.apiKey` 配置
2. **Gemini** — 检查 `GEMINI_API_KEY` 环境变量或 `tools.web.search.gemini.apiKey` 配置
3. **Kimi** — 检查 `KIMI_API_KEY` / `MOONSHOT_API_KEY` 环境变量或 `tools.web.search.kimi.apiKey` 配置
4. **Perplexity** — 检查 `PERPLEXITY_API_KEY` 环境变量或 `tools.web.search.perplexity.apiKey` 配置
5. **Grok** — 检查 `XAI_API_KEY` 环境变量或 `tools.web.search.grok.apiKey` 配置

如果找不到任何密钥，系统会回退到 Brave（此时会提示缺少密钥，引导你进行配置）。

## 设置网页搜索

最简单的方式是运行 `openclaw configure --section web`，按提示设置 API 密钥并选择服务商。

### Perplexity 搜索

1. 访问 [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) 创建 Perplexity 账户
2. 在控制面板中生成 API 密钥
3. 运行 `openclaw configure --section web` 将密钥存入配置，或者直接设置环境变量 `PERPLEXITY_API_KEY`

更多细节请参阅 [Perplexity Search API 文档](https://docs.perplexity.ai/guides/search-quickstart)。

### Brave 搜索

1. 访问 [brave.com/search/api](https://brave.com/search/api/) 创建 Brave Search API 账户
2. 在控制面板中选择 **Data for Search** 计划（注意不是 "Data for AI"），然后生成 API 密钥
3. 运行 `openclaw configure --section web` 将密钥存入配置（推荐），或者设置环境变量 `BRAVE_API_KEY`

Brave 提供付费计划，具体的额度和定价请查看 Brave API 门户。

### 密钥存放在哪里？

**推荐方式（配置文件）：** 运行 `openclaw configure --section web`，密钥会存储在 `tools.web.search.perplexity.apiKey` 或 `tools.web.search.apiKey` 字段中。

**替代方式（环境变量）：** 在 Gateway 进程环境中设置 `PERPLEXITY_API_KEY` 或 `BRAVE_API_KEY`。如果是以网关服务形式安装，可以放在 `~/.openclaw/.env` 文件中。详见[环境变量](../help/faq.md#how-does-openclaw-load-environment-variables)。

### 配置示例

**Perplexity 搜索：**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // 如果已设置 PERPLEXITY_API_KEY 环境变量，这里可以省略
        },
      },
    },
  },
}
```

**Brave 搜索：**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // 如果已设置 BRAVE_API_KEY 环境变量，这里可以省略 // pragma: allowlist secret
      },
    },
  },
}
```

## 使用 Gemini（Google 搜索联网）

Gemini 模型内置了 [Google 搜索联网](https://ai.google.dev/gemini-api/docs/grounding)功能——它会基于实时 Google 搜索结果，生成带有引用来源的 AI 综合回答。

### 获取 Gemini API 密钥

1. 访问 [Google AI Studio](https://aistudio.google.com/apikey)
2. 创建 API 密钥
3. 在 Gateway 环境中设置 `GEMINI_API_KEY`，或在配置中设置 `tools.web.search.gemini.apiKey`

### 配置 Gemini 搜索

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // API 密钥（如果已设置 GEMINI_API_KEY 环境变量，这里可以省略）
          apiKey: "AIza...",
          // 使用的模型（默认为 "gemini-2.5-flash"）
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**环境变量替代方案：** 在 Gateway 环境中设置 `GEMINI_API_KEY`。如果是以网关服务形式安装，可以放在 `~/.openclaw/.env` 文件中。

### 注意事项

- Gemini 联网返回的引用 URL 会自动从 Google 的重定向 URL 解析为直接 URL。
- 重定向解析会经过 SSRF 防护检查（HEAD 请求 + 重定向检查 + http/https 验证），确保返回的是安全的最终 URL。
- 重定向解析使用严格的 SSRF 默认策略，因此重定向到私有地址或内部网络会被阻止。
- 默认模型 `gemini-2.5-flash` 速度快且性价比高。你可以使用任何支持联网功能的 Gemini 模型。

## web_search 工具详解

使用你配置好的服务商进行网页搜索。

### 使用前提

- `tools.web.search.enabled` 不能设为 `false`（默认启用）
- 需要配置对应服务商的 API 密钥：
  - **Brave**: `BRAVE_API_KEY` 或 `tools.web.search.apiKey`
  - **Perplexity**: `PERPLEXITY_API_KEY` 或 `tools.web.search.perplexity.apiKey`
  - **Gemini**: `GEMINI_API_KEY` 或 `tools.web.search.gemini.apiKey`
  - **Grok**: `XAI_API_KEY` 或 `tools.web.search.grok.apiKey`
  - **Kimi**: `KIMI_API_KEY`、`MOONSHOT_API_KEY` 或 `tools.web.search.kimi.apiKey`

### 完整配置

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // 如果已设置 BRAVE_API_KEY 环境变量，这里可以省略
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### 工具参数

下表列出所有可用的搜索参数。除非特别说明，这些参数同时适用于 Brave 和 Perplexity。

| 参数 | 说明 |
| --- | --- |
| `query` | 搜索关键词（必填） |
| `count` | 返回结果数量（1-10，默认 5） |
| `country` | 2 字母 ISO 国家代码（如 "US"、"CN"、"DE"） |
| `language` | ISO 639-1 语言代码（如 "en"、"zh"、"de"） |
| `freshness` | 时效性过滤：`day`、`week`、`month` 或 `year` |
| `date_after` | 仅返回此日期之后的结果（格式：YYYY-MM-DD） |
| `date_before` | 仅返回此日期之前的结果（格式：YYYY-MM-DD） |
| `ui_lang` | UI 语言代码（仅 Brave 支持） |
| `domain_filter` | 域名白名单/黑名单数组（仅 Perplexity 支持） |
| `max_tokens` | 内容总预算，默认 25000（仅 Perplexity 支持） |
| `max_tokens_per_page` | 每页内容上限，默认 2048（仅 Perplexity 支持） |

**使用示例：**

```
// 限定德语和德国地区
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// 只要最近一周的结果
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// 按日期范围搜索
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// 仅搜索特定域名（仅 Perplexity）
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// 排除特定域名（仅 Perplexity）
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// 获取更多内容（仅 Perplexity）
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web_fetch 工具详解

获取（fetch）指定 URL 的内容，并提取其中的可读文本。

### 使用前提

- `tools.web.fetch.enabled` 不能设为 `false`（默认启用）
- 可选：配置 Firecrawl 作为备选方案，设置 `tools.web.fetch.firecrawl.apiKey` 或环境变量 `FIRECRAWL_API_KEY`

### 完整配置

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // 如果已设置 FIRECRAWL_API_KEY 环境变量，这里可以省略
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // 毫秒（1 天）
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### 工具参数

- `url` — 必填，目标地址（仅支持 http/https）
- `extractMode` — 提取模式：`markdown` 或 `text`
- `maxChars` — 截断超长页面

### 工作流程与注意事项

- `web_fetch` 会优先使用 Readability（主内容提取算法），如果失败则尝试 Firecrawl（需配置）。两者都失败时，工具返回错误。
- Firecrawl 请求默认启用绕过机器人检测模式，并缓存结果。
- `web_fetch` 默认发送模拟 Chrome 的 User-Agent 和 `Accept-Language` 头，如需自定义可以覆盖 `userAgent`。
- `web_fetch` 会阻止访问私有/内部主机名，并对重定向进行二次检查（重定向次数受 `maxRedirects` 限制）。
- `maxChars` 的实际值会被限制在 `tools.web.fetch.maxCharsCap` 以内。
- `web_fetch` 在解析前会将响应体大小限制在 `tools.web.fetch.maxResponseBytes` 以内，超大的响应会被截断并附带警告。
- 这是一个尽力而为的提取工具——某些复杂网站可能需要浏览器工具才能正确处理。
- 关于 Firecrawl 的密钥设置和服务详情，请参阅 [Firecrawl](./firecrawl.md)。
- 响应默认缓存 15 分钟，避免重复获取。
- 如果你使用工具配置文件或白名单，需要添加 `web_search`/`web_fetch` 或 `group:web`。
- 如果缺少 API 密钥，`web_search` 会返回简短的配置提示和文档链接。

[思考层级](./thinking.md)[浏览器（OpenClaw 托管）](./browser.md)