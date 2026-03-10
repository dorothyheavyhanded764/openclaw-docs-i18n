

  消息平台

  
# Synology Chat

状态：通过插件支持，作为使用 Synology Chat webhook 的私信频道（channel）。该插件接收来自 Synology Chat 出站 webhook 的入站消息，并通过 Synology Chat 入站 webhook 发送回复。

## 需要安装插件

Synology Chat 是基于插件的频道（channel），不包含在默认核心安装中。你需要从本地代码目录安装：

```bash
openclaw plugins install ./extensions/synology-chat
```

更多信息请参考 [插件](../tools/plugin.md) 文档。

## 快速配置

按照以下步骤快速完成 Synology Chat 频道（channel）的配置：

1. 安装并启用 Synology Chat 插件。
2. 在 Synology Chat 的集成设置中：
    - 创建一个入站 webhook，复制其 URL。
    - 创建一个出站 webhook，设置你的密钥令牌。
3. 将出站 webhook 的 URL 指向你的 OpenClaw 网关：
    - 默认路径为 `https://gateway-host/webhook/synology`。
    - 或自定义 `channels.synology-chat.webhookPath` 配置项。
4. 在 OpenClaw 中配置 `channels.synology-chat` 相关参数。
5. 重启网关，然后向 Synology Chat 机器人发送一条私信进行测试。

最简配置示例：

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## 环境变量

对于默认账户，你可以通过环境变量进行配置：

- `SYNOLOGY_CHAT_TOKEN`
- `SYNOLOGY_CHAT_INCOMING_URL`
- `SYNOLOGY_NAS_HOST`
- `SYNOLOGY_ALLOWED_USER_IDS`（多个 ID 用逗号分隔）
- `SYNOLOGY_RATE_LIMIT`
- `OPENCLAW_BOT_NAME`

注意：配置文件中的值优先级高于环境变量。

## 私信策略与访问控制

通过私信策略（DM Policy）控制哪些用户可以向智能体（agent）发送消息：

- `dmPolicy: "allowlist"` 是推荐的默认设置，只允许白名单中的用户发送消息。
- `allowedUserIds` 接受 Synology 用户 ID 列表（数组或逗号分隔的字符串）。
- 在 `allowlist` 模式下，如果 `allowedUserIds` 为空列表，将被视为配置错误，webhook 路由不会启动。如需允许所有用户，请使用 `dmPolicy: "open"`。
- `dmPolicy: "open"` 允许任何发送者。
- `dmPolicy: "disabled"` 禁用私信功能。

配对审批相关命令：

```bash
openclaw pairing list synology-chat
openclaw pairing approve synology-chat <CODE>
```

## 出站消息发送

使用数字形式的 Synology Chat 用户 ID 作为消息目标。示例：

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

媒体文件支持通过 URL 方式发送。

## 多账户配置

支持在 `channels.synology-chat.accounts` 下配置多个 Synology Chat 账户。每个账户可以独立设置令牌、入站 URL、webhook 路径、私信策略和速率限制。

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## 安全注意事项

- 妥善保管 `token`，一旦泄露应立即轮换。
- 除非你明确信任本地 NAS 的自签名证书，否则保持 `allowInsecureSsl: false`。
- 入站 webhook 请求会进行令牌验证，并按发送者进行速率限制。
- 生产环境强烈建议使用 `dmPolicy: "allowlist"`。

[Signal](./signal.md)[Slack](./slack.md)