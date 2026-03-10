

  媒体与设备

  
# 图片与媒体支持

本文档说明 WhatsApp 频道如何处理各类媒体文件，包括发送、接收、自动回复等场景的处理规则。WhatsApp 频道基于 **Baileys Web** 运行。

## 目标

- 通过 `openclaw message send --media` 发送媒体，支持附带说明文字
- 支持网页收件箱的自动回复携带媒体和文本
- 为各类媒体设置合理、可预测的大小限制

## CLI 接口

```bash
openclaw message send --media <path-or-url> [--message <caption>]
```

- `--media`: 可选参数；纯媒体发送时说明文字可以为空
- `--dry-run`: 打印解析后的负载内容
- `--json`: 输出 `{ channel, to, messageId, mediaUrl, caption }` 格式

## WhatsApp Web 频道行为

### 输入源

接受本地文件路径或 HTTP(S) URL。

### 处理流程

将媒体加载到 Buffer，检测类型后构建对应负载：

- **图片**: 调整尺寸并重新压缩为 JPEG（最大边长 2048px），目标大小由 `agents.defaults.mediaMaxMb` 控制（默认 5 MB），上限 6 MB
- **音频/语音/视频**: 直通传输，上限 16 MB；音频作为语音消息发送（`ptt: true`）
- **文档**: 其他所有文件类型，上限 100 MB，保留原始文件名

### 特殊处理

- **GIF 式播放**: 发送带 `gifPlayback: true` 的 MP4 文件（CLI 参数：`--gif-playback`），移动端会内联循环播放
- **MIME 检测优先级**: 魔数字节 > HTTP 头部 > 文件扩展名
- **说明文字来源**: `--message` 参数或 `reply.text`，允许为空
- **日志输出**: 简洁模式显示 `↩️`/`✅`；详细模式包含文件大小和来源路径/URL

## 自动回复流程

`getReplyFromConfig` 返回 `{ text?, mediaUrl?, mediaUrls? }` 结构。当包含媒体时，网页发送器使用与 `openclaw message send` 相同的流程解析本地路径或 URL。多个媒体条目会按顺序逐一发送。

## 入站媒体处理

当收到的网页消息包含媒体时，OpenClaw 会将其下载到临时文件并暴露以下模板变量：

- `{{MediaUrl}}`: 入站媒体的伪 URL
- `{{MediaPath}}`: 命令执行前的本地临时路径

### Docker 沙箱场景

启用会话级 Docker 沙箱时，入站媒体会被复制到沙箱工作区，`MediaPath`/`MediaUrl` 重写为相对路径（如 `media/inbound/`）。

### 媒体理解

通过 `tools.media.*` 或共享的 `tools.media.models` 配置后，媒体理解会在模板化之前运行，可将 `[Image]`、`[Audio]`、`[Video]` 块插入到 `Body` 中：

- 音频会设置 `{{Transcript}}` 变量，转录文本用于命令解析，确保斜杠命令正常工作
- 视频和图片描述会保留原有说明文字用于命令解析

默认只处理第一个匹配的图片/音频/视频附件；如需处理多个附件，请设置 `tools.media..attachments`。

## 限制与错误

### 出站发送上限（WhatsApp web 发送）

| 类型 | 大小限制 |
|------|----------|
| 图片 | 约 6 MB（重新压缩后） |
| 音频/语音/视频 | 16 MB |
| 文档 | 100 MB |

超大或无法读取的媒体会在日志中显示明确错误，并跳过该回复。

### 媒体理解上限（转录/描述）

| 类型 | 默认限制 | 配置项 |
|------|----------|--------|
| 图片 | 10 MB | `tools.media.image.maxBytes` |
| 音频 | 20 MB | `tools.media.audio.maxBytes` |
| 视频 | 50 MB | `tools.media.video.maxBytes` |

超大媒体会跳过理解处理，但回复仍会以原始正文发送。

## 测试要点

- 覆盖图片、音频、文档三种类型的发送和回复流程
- 验证图片重新压缩符合大小限制，音频正确标记为语音消息
- 确认多媒体回复按顺序逐一发送

[媒体理解](./media-understanding.md)[音频和语音笔记](./audio.md)