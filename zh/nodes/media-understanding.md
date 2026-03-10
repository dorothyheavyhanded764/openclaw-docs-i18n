

  媒体与设备

  
# 媒体理解

OpenClaw 可以在回复流程运行前，**对入站的媒体文件（图片/音频/视频）进行预处理和总结**。系统会自动检测可用的本地工具或提供商 API，你也可以禁用或自定义这一功能。即使媒体理解功能关闭，模型仍然可以正常接收原始文件或 URL。

## 目标

- **可选预处理**：将入站媒体转换为简短文本描述，加快路由决策、改善命令解析
- **保留原始媒体**：始终将原始媒体传递给模型
- **灵活后端**：支持提供商 API 和本地 CLI 回退
- **多模型容错**：支持配置多个模型，按顺序回退（处理错误、文件过大、超时等情况）

## 整体流程

1. 收集入站附件（`MediaPaths`、`MediaUrls`、`MediaTypes`）
2. 对每种启用的能力（图片/音频/视频），按策略选择附件（默认：**第一个**）
3. 选择第一个符合条件的模型条目（检查大小限制、能力匹配、认证状态）
4. 如果模型失败或媒体过大，**自动尝试下一个条目**
5. 成功时：
   - `Body` 变为 `[Image]`、`[Audio]` 或 `[Video]` 块
   - 音频会设置 `{{Transcript}}` 变量；命令解析优先使用说明文字，没有时使用转录文本
   - 说明文字会作为 `User text:` 保留在块内

如果理解失败或被禁用，**回复流程仍会继续**，使用原始正文和附件。

## 配置概览

`tools.media` 支持**共享模型列表** + **按能力单独覆盖**：

- `tools.media.models`: 共享模型列表（用 `capabilities` 字段控制适用范围）
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - 默认值（`prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`）
  - 提供商覆盖（`baseUrl`、`headers`、`providerOptions`）
  - Deepgram 音频选项通过 `tools.media.audio.providerOptions.deepgram` 配置
  - 音频转录回显控制（`echoTranscript`，默认 `false`；`echoFormat`）
  - 可选的**按能力 `models` 列表**（优先于共享模型）
  - `attachments` 策略（`mode`、`maxAttachments`、`prefer`）
  - `scope`（可选，按频道/聊天类型/会话键进行门控）
- `tools.media.concurrency`: 最大并发处理数（默认 **2**）

```json
{
  tools: {
    media: {
      models: [
        /* 共享模型列表 */
      ],
      image: {
        /* 可选覆盖 */
      },
      audio: {
        /* 可选覆盖 */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* 可选覆盖 */
      },
    },
  },
}
```

### 模型条目格式

每个 `models[]` 条目可以是**提供商类型**或**CLI 类型**：

**提供商类型**：

```json
{
  type: "provider", // 省略时的默认值
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // 可选，用于多模态条目
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

**CLI 类型**：

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLI 模板还支持以下变量：

- `{{MediaDir}}`（媒体文件所在目录）
- `{{OutputDir}}`（本次运行创建的临时目录）
- `{{OutputBase}}`（临时文件基本路径，不含扩展名）

## 默认值与限制

推荐默认值：

- `maxChars`: 图片/视频为 **500**（简短，便于命令解析）
- `maxChars`: 音频为 **不设置**（完整转录，除非你主动设限）
- `maxBytes`:
  - 图片：**10MB**
  - 音频：**20MB**
  - 视频：**50MB**

规则说明：

- 媒体超过 `maxBytes` 时，跳过该模型并**尝试下一个模型**
- 小于 **1024 字节**的音频文件视为空文件或损坏，在提供商/CLI 转录前直接跳过
- 模型返回内容超过 `maxChars` 时会被截断
- `prompt` 默认为简单的"Describe the ."加上 `maxChars` 提示（仅图片/视频）
- 如果 `.enabled: true` 但未配置模型，OpenClaw 会在当前回复模型支持该能力时尝试使用它

### 自动检测媒体理解（默认行为）

如果 `tools.media..enabled` 未设置为 `false` 且你没有配置模型，OpenClaw 会按以下顺序自动检测，**在第一个可用选项处停止**：

1. **本地 CLI**（仅音频；需已安装）
   - `sherpa-onnx-offline`（需要设置 `SHERPA_ONNX_MODEL_DIR`，包含编码器/解码器/连接器/词表）
   - `whisper-cli`（`whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或内置 tiny 模型）
   - `whisper`（Python CLI；自动下载模型）
2. **Gemini CLI**（`gemini`）使用 `read_many_files`
3. **提供商 API**
   - 音频：OpenAI → Groq → Deepgram → Google
   - 图片：OpenAI → Anthropic → Google → MiniMax
   - 视频：Google

禁用自动检测：

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

注意：二进制检测在 macOS/Linux/Windows 上是尽力而为的；确保 CLI 在 `PATH` 中（我们会展开 `~`），或者使用完整命令路径配置 CLI 模型。

### 代理环境支持（提供商模型）

启用基于提供商的**音频**和**视频**媒体理解时，OpenClaw 会遵循标准的出站代理环境变量：

- `HTTPS_PROXY`
- `HTTP_PROXY`
- `https_proxy`
- `http_proxy`

如果未设置代理变量，媒体理解使用直连。如果代理值格式错误，OpenClaw 会记录警告并回退到直连。

## 能力字段（可选）

设置了 `capabilities` 后，该条目只对指定的媒体类型生效。对于共享列表，OpenClaw 可以推断默认值：

| 提供商 | 默认能力 |
|--------|----------|
| `openai`、`anthropic`、`minimax` | 图片 |
| `google`（Gemini API） | 图片 + 音频 + 视频 |
| `groq` | 音频 |
| `deepgram` | 音频 |

对于 CLI 条目，**请显式设置 `capabilities`** 以避免意外匹配。省略 `capabilities` 时，条目对所在列表的所有类型都有效。

## 提供商支持矩阵（OpenClaw 集成）

| 能力 | 提供商集成 | 备注 |
| --- | --- | --- |
| 图片 | OpenAI / Anthropic / Google / 其他通过 `pi-ai` | 注册表中任何支持图片的模型都可用 |
| 音频 | OpenAI, Groq, Deepgram, Google, Mistral | 提供商转录（Whisper/Deepgram/Gemini/Voxtral） |
| 视频 | Google（Gemini API） | 提供商视频理解 |

## 模型选择建议

- 追求质量和安全性时，为每种媒体能力选择可用的最强最新一代模型
- 处理不可信输入的工具型智能体（agent），避免使用较旧或较弱的媒体模型
- 为每种能力至少保留一个回退模型（高质量模型 + 更快/更便宜的模型）
- 当提供商 API 不可用时，CLI 回退（`whisper-cli`、`whisper`、`gemini`）很有用
- `parakeet-mlx` 注意：使用 `--output-dir` 时，输出格式为 `txt`（或未指定）时，OpenClaw 会读取 `<output-dir>/<media-basename>.txt`；非 `txt` 格式则回退到 stdout

## 附件策略

按能力的 `attachments` 配置控制处理哪些附件：

- `mode`: `first`（默认）或 `all`
- `maxAttachments`: 限制处理数量（默认 **1**）
- `prefer`: `first`、`last`、`path`、`url`

当 `mode: "all"` 时，输出会标记为 `[Image 1/2]`、`[Audio 2/2]` 等。

## 配置示例

### 1) 共享模型列表 + 覆盖

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) 仅音频 + 视频（图片关闭）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) 可选的图片理解

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) 多模态单一条目（显式能力）

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## 状态输出

媒体理解运行时，`/status` 会显示简要摘要：

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

展示每种能力的结果，以及选用的提供商/模型。

## 注意事项

- 理解是**尽力而为**的，错误不会阻塞回复流程
- 即使理解被禁用，附件仍会传递给模型
- 使用 `scope` 可限制理解运行的范围（例如仅限私信）

## 相关文档

- [配置](../gateway/configuration.md)
- [图片和媒体支持](./images.md)

[节点故障排除](./troubleshooting.md)[图片和媒体支持](./images.md)