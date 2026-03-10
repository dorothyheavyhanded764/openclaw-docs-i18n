

  消息平台

  
# LINE

LINE 通过 LINE Messaging API 与 OpenClaw 建立连接。插件以 Webhook 接收器的形式运行在网关上，使用频道访问令牌（Channel Access Token）和频道密钥（Channel Secret）完成身份验证。

**支持状态**：通过插件支持。支持私聊、群聊、媒体消息、位置分享、Flex 消息、模板消息和快速回复。暂不支持消息反应和话题讨论。

## 安装插件

首先安装 LINE 插件：

```bash
openclaw plugins install @openclaw/line
```

如果你是从 Git 仓库本地运行，可以这样安装：

```bash
openclaw plugins install ./extensions/line
```

## 配置步骤

按照以下步骤在 LINE 开发者控制台完成配置：

1. 创建 LINE Developers 账户并打开控制台：[https://developers.line.biz/console/](https://developers.line.biz/console/)
2. 创建一个 Provider（或使用已有的），然后添加一个 **Messaging API** 频道（channel）。
3. 在频道设置页面复制 **Channel access token** 和 **Channel secret**。
4. 在 Messaging API 设置中启用 **Use webhook** 选项。
5. 将 Webhook URL 设置为你的网关端点（必须使用 HTTPS）：

```
https://gateway-host/line/webhook
```

网关会处理 LINE 发来的 Webhook 验证请求（GET）和消息事件（POST）。如果需要自定义路径，可以设置 `channels.line.webhookPath` 或 `channels.line.accounts..webhookPath`，并相应更新 URL。

:::note 安全说明
LINE 的签名验证基于原始请求体进行 HMAC 计算，因此 OpenClaw 会在验证前对请求体大小和超时进行严格限制。
:::

## 基础配置

最小配置示例：

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

### 环境变量

对于默认账户，可以使用以下环境变量：

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

### 使用文件存储凭据

如果不想在配置文件中直接写入凭据，可以指向文件路径：

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

### 多账户配置

需要同时管理多个 LINE 账号时，可以使用 `accounts` 配置：

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## 访问控制

私聊默认采用配对模式：未知用户会收到一个配对码，在管理员批准之前，他们发送的消息会被忽略。

查看和批准配对请求：

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

### 策略配置选项

以下配置项可以精细控制访问权限：

- `channels.line.dmPolicy`：私聊策略，可选值 `pairing | allowlist | open | disabled`
- `channels.line.allowFrom`：允许私聊的 LINE 用户 ID 白名单
- `channels.line.groupPolicy`：群聊策略，可选值 `allowlist | open | disabled`
- `channels.line.groupAllowFrom`：允许群聊的 LINE 用户 ID 白名单
- `channels.line.groups..allowFrom`：针对特定群组的覆盖配置

:::note 运行时行为
如果 `channels.line` 配置完全缺失，运行时会默认使用 `groupPolicy="allowlist"` 进行群组检查（即使设置了 `channels.defaults.groupPolicy`）。
:::

### LINE ID 格式

LINE ID 区分大小写，格式如下：

- 用户 ID：`U` + 32 位十六进制字符
- 群组 ID：`C` + 32 位十六进制字符
- 房间 ID：`R` + 32 位十六进制字符

## 消息处理方式

- 文本消息会在 5000 字符处自动分割。
- Markdown 格式会被移除；代码块和表格会尽可能转换为 Flex 卡片展示。
- 流式响应会被缓冲，智能体（agent）处理时 LINE 会显示加载动画，完成后一次性发送完整内容。
- 媒体下载大小受 `channels.line.mediaMaxMb` 限制（默认 10MB）。

## 发送富消息

通过 `channelData.line` 可以发送快速回复、位置、Flex 卡片或模板消息：

```json
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

LINE 插件还内置了 `/card` 命令，方便快速发送预设的 Flex 消息：

```bash
/card info "Welcome" "Thanks for joining!"
```

## 常见问题

- **Webhook 验证失败**：确认 Webhook URL 使用 HTTPS，且 `channelSecret` 与 LINE 控制台中的值一致。
- **收不到消息**：检查 Webhook 路径是否与 `channels.line.webhookPath` 匹配，确保网关能被 LINE 服务器访问。
- **媒体下载失败**：如果媒体文件超过默认限制，尝试调大 `channels.line.mediaMaxMb` 值。

[IRC](./irc.md)[Matrix](./matrix.md)