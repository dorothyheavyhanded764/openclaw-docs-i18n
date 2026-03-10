

  媒体与设备

  
# 音频与语音笔记

本节介绍如何配置 OpenClaw 处理音频和语音笔记，包括转录、自动检测、回退机制以及群组中的提及检测。

## 功能概述

- **媒体理解（音频）**：启用音频理解后（或自动检测到），OpenClaw 会：
  1. 定位第一个音频附件（本地路径或 URL），必要时下载
  2. 发送给每个模型前检查 `maxBytes` 限制
  3. 按顺序运行第一个符合条件的模型条目（提供商或 CLI）
  4. 失败或跳过时（大小/超时），自动尝试下一个条目
  5. 成功后，用 `[Audio]` 块替换 `Body` 并设置 `{{Transcript}}` 变量

- **命令解析**：转录成功后，`CommandBody`/`RawBody` 会被设置为转录文本，确保斜杠命令正常工作

- **详细日志**：`--verbose` 模式下会记录转录执行和正文替换的时机

## 自动检测（默认行为）

如果**未配置模型**且 `tools.media.audio.enabled` 未设置为 `false`，OpenClaw 会按以下顺序自动检测，**在第一个可用选项处停止**：

1. **本地 CLI**（需已安装）
   - `sherpa-onnx-offline`（需要设置 `SHERPA_ONNX_MODEL_DIR`，包含编码器/解码器/连接器/词表）
   - `whisper-cli`（来自 `whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或内置 tiny 模型）
   - `whisper`（Python CLI；自动下载模型）

2. **Gemini CLI**（`gemini`）使用 `read_many_files`

3. **提供商 API**（OpenAI → Groq → Deepgram → Google）

禁用自动检测：设置 `tools.media.audio.enabled: false`。自定义配置：设置 `tools.media.audio.models`。

注意：二进制检测在 macOS/Linux/Windows 上是尽力而为的；确保 CLI 在 `PATH` 中（我们会展开 `~`），或使用完整命令路径配置 CLI 模型。

## 配置示例

### 提供商 + CLI 回退（OpenAI + Whisper CLI）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### 仅提供商 + 范围控制

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### 仅提供商（Deepgram）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

### 仅提供商（Mistral Voxtral）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

### 转录文本回显到聊天（可选）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // 默认为 false
        echoFormat: '📝 "{transcript}"', // 可选，支持 {transcript} 占位符
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## 注意事项与限制

- 提供商认证遵循标准模型认证顺序（认证配置、环境变量、`models.providers.*.apiKey`）
- 使用 `provider: "deepgram"` 时，会读取 `DEEPGRAM_API_KEY`
- Deepgram 配置详情：[Deepgram（音频转录）](../providers/deepgram.md)
- Mistral 配置详情：[Mistral](../providers/mistral.md)
- 音频提供商可通过 `tools.media.audio` 覆盖 `baseUrl`、`headers` 和 `providerOptions`
- 默认大小上限 20MB（`tools.media.audio.maxBytes`）。超大音频会跳过该模型，尝试下一个条目
- 小于 1024 字节的空音频文件会在提供商/CLI 转录前直接跳过
- 音频的 `maxChars` 默认**不设置**（完整转录）。设置 `tools.media.audio.maxChars` 或条目级 `maxChars` 可截断输出
- OpenAI 自动默认使用 `gpt-4o-mini-transcribe`；设置 `model: "gpt-4o-transcribe"` 可获得更高准确度
- 使用 `tools.media.audio.attachments` 处理多个语音笔记（`mode: "all"` + `maxAttachments`）
- 转录文本在模板中可通过 `{{Transcript}}` 访问
- `tools.media.audio.echoTranscript` 默认关闭；启用后会在智能体（agent）处理前将转录确认发送回原聊天
- `tools.media.audio.echoFormat` 可自定义回显文本（占位符：`{transcript}`）
- CLI 标准输出有 5MB 上限；请保持 CLI 输出简洁

### 代理环境支持

基于提供商的音频转录遵循标准出站代理环境变量：

- `HTTPS_PROXY`
- `HTTP_PROXY`
- `https_proxy`
- `http_proxy`

未设置代理变量时使用直连。代理配置格式错误时，OpenClaw 会记录警告并回退到直连。

## 群组中的提及检测

当群组聊天设置了 `requireMention: true` 时，OpenClaw 会在检查提及**之前**先转录音频。这样语音笔记中包含的提及也能被正确识别。

**工作流程：**

1. 语音消息没有文本正文且群组要求提及时，OpenClaw 执行"预检"转录
2. 检查转录文本中的提及模式（如 `@BotName`、表情符号触发器）
3. 找到提及时，消息进入完整回复流程
4. 转录文本用于提及检测，让语音笔记能通过提及门控

**回退行为：**

- 预检转录失败时（超时、API 错误等），按纯文本提及检测处理消息
- 确保混合消息（文本 + 音频）不会被错误丢弃

**按 Telegram 群组/主题退出：**

- 设置 `channels.telegram.groups..disableAudioPreflight: true` 跳过该群组的预检转录提及检查
- 设置 `channels.telegram.groups..topics..disableAudioPreflight` 按主题覆盖（`true` 跳过，`false` 强制启用）
- 默认为 `false`（满足提及门控条件时启用预检）

**示例**：用户在设置了 `requireMention: true` 的 Telegram 群组中发送语音笔记说"嘿 @Claude，天气怎么样？"。语音笔记被转录，检测到提及，智能体（agent）回复。

## 常见问题

- 范围规则采用首次匹配优先。`chatType` 规范化为 `direct`、`group` 或 `room`
- 确保 CLI 以退出码 0 退出并输出纯文本；JSON 需通过 `jq -r .text` 处理
- 对于 `parakeet-mlx`，传递 `--output-dir` 时，`--output-format` 为 `txt`（或省略）时 OpenClaw 会读取 `<output-dir>/<media-basename>.txt`；非 `txt` 格式回退到标准输出解析
- 设置合理的超时时间（`timeoutSeconds`，默认 60 秒），避免阻塞回复队列
- 预检转录只处理**第一个**音频附件进行提及检测，其他音频在主媒体理解阶段处理

[图片和媒体支持](./images.md)[摄像头捕获](./camera.md)