

  内置工具

  
# PDF 工具

需要让智能体阅读和分析 PDF 文档？`pdf` 工具可以帮你提取 PDF 中的文本内容。它的核心特性：

-   **原生模式**：针对 Anthropic 和 Google 等提供商，直接发送原始 PDF 数据。
-   **回退模式**：针对其他提供商，先提取文本，必要时再提取页面图像。
-   **灵活输入**：支持单个 (`pdf`) 或多个 (`pdfs`) PDF 文档，单次调用最多 10 个。

## 工具可用性

`pdf` 工具只有在 OpenClaw 能为智能体找到支持 PDF 的模型时才会启用。查找顺序如下：

1.  `agents.defaults.pdfModel`
2.  回退到 `agents.defaults.imageModel`
3.  最后尝试基于可用认证选择最佳提供商默认值

如果找不到可用的模型配置，`pdf` 工具就不会被注册。

## 输入参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `pdf` | `string` | 单个 PDF 的路径或 URL |
| `pdfs` | `string[]` | 多个 PDF 的路径或 URL，最多 10 个 |
| `prompt` | `string` | 分析提示词，默认为 `Analyze this PDF document.` |
| `pages` | `string` | 页面范围过滤，格式如 `1-5` 或 `1,3,7-9` |
| `model` | `string` | 可选，指定使用的模型 (`provider/model`) |
| `maxBytesMb` | `number` | 每个 PDF 的大小上限（MB） |

**注意事项：**

-   `pdf` 和 `pdfs` 参数可以同时使用，系统会在加载前合并并去重。
-   如果没有提供任何 PDF 输入，工具会报错。
-   `pages` 使用从 1 开始的页码，系统会自动去重、排序并限制在最大页数内。
-   `maxBytesMb` 默认值为 `agents.defaults.pdfMaxBytesMb` 或 `10`。

## 支持的文件来源

你可以通过以下方式引用 PDF 文档：

-   本地文件路径（支持 `~` 扩展）
-   `file://` URL
-   `http://` 和 `https://` URL

**限制说明：**

-   不支持其他 URI 方案（如 `ftp://`），会返回 `unsupported_pdf_reference` 错误。
-   沙盒模式下，远程 `http(s)` URL 会被拒绝。
-   启用仅工作空间文件策略时，工作空间目录外的本地文件路径会被拒绝。

## 执行模式详解

### 原生提供商模式

当使用 `anthropic` 或 `google` 提供商时，工具采用原生模式——直接将原始 PDF 字节发送给提供商 API。

**限制：** 原生模式不支持 `pages` 参数，如果设置了会返回错误。

### 提取回退模式

对于其他提供商，工具采用提取回退模式，处理流程如下：

1.  从选定页面提取文本（最多 `agents.defaults.pdfMaxPages` 页，默认 `20`）。
2.  如果提取的文本不足 200 个字符，则将选定页面渲染为 PNG 图像一并提交。
3.  将提取的内容和提示词一起发送给目标模型。

**补充说明：**

-   页面图像提取的像素预算为 `4,000,000`。
-   如果目标模型不支持图像输入且没有可提取的文本，工具会报错。
-   提取回退模式依赖 `pdfjs-dist` 库，图像渲染还需要 `@napi-rs/canvas`。

## 配置示例

```json
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

完整的配置字段说明请参阅[配置参考](../gateway/configuration-reference.md)。

## 输出结构

工具返回的文本在 `content[0].text` 中，结构化元数据在 `details` 中。常见的 `details` 字段：

-   `model`: 实际使用的模型 (`provider/model`)
-   `native`: `true` 表示原生模式，`false` 表示回退模式
-   `attempts`: 成功前失败的回退尝试次数

**路径相关字段：**

-   单个 PDF 输入：`details.pdf`
-   多个 PDF 输入：`details.pdfs[]`（包含多个 `pdf` 条目）
-   沙盒路径重写元数据（如适用）：`rewrittenFrom`

## 错误处理

| 情况 | 行为 |
| --- | --- |
| 缺少 PDF 输入 | 抛出错误：`pdf required: provide a path or URL to a PDF document` |
| PDF 数量超限 | 返回结构化错误：`details.error = "too_many_pdfs"` |
| 不支持的引用方案 | 返回：`details.error = "unsupported_pdf_reference"` |
| 原生模式使用 `pages` | 抛出错误：`pages is not supported with native PDF providers` |

## 使用示例

**分析单个 PDF：**

```json
{
  "pdf": "/tmp/report.pdf",
  "prompt": "Summarize this report in 5 bullets"
}
```

**比较多个 PDF：**

```json
{
  "pdfs": ["/tmp/q1.pdf", "/tmp/q2.pdf"],
  "prompt": "Compare risks and timeline changes across both documents"
}
```

**指定页面和模型（回退模式）：**

```json
{
  "pdf": "https://example.com/report.pdf",
  "pages": "1-3,7",
  "model": "openai/gpt-5-mini",
  "prompt": "Extract only customer-impacting incidents"
}
```

[差异](./diffs.md)[提升模式](./elevated.md)