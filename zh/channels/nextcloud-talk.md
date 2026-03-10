

  消息平台

  
# Nextcloud Talk

状态：通过插件支持（webhook 机器人）。支持私聊、房间、消息反应和 Markdown 格式消息。

## 需要先安装插件

Nextcloud Talk 以插件形式提供，不包含在核心安装包中。你可以通过 CLI（npm 仓库）安装：

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

如果你是从 git 仓库本地运行的，可以用本地检出路径安装：

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

提示：在配置或引导过程中，如果你选择了 Nextcloud Talk 且系统检测到 git 检出，OpenClaw 会自动提示本地安装路径。更多详情请参阅 [插件](../tools/plugin.md)。

## 快速上手

只需几步就能让 OpenClaw 在 Nextcloud Talk 中运行起来：

1. 安装 Nextcloud Talk 插件。
2. 在 Nextcloud 服务器上创建一个机器人：

   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```

3. 在目标房间的设置中启用这个机器人。
4. 配置 OpenClaw：
   - 通过配置文件：设置 `channels.nextcloud-talk.baseUrl` 和 `channels.nextcloud-talk.botSecret`
   - 或通过环境变量：`NEXTCLOUD_TALK_BOT_SECRET`（仅限默认账户）
5. 重启网关（或完成引导流程）。

最简配置示例：

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## 使用注意事项

在使用 Nextcloud Talk 频道（channel）时，请注意以下几点：

- **机器人无法主动发起私聊**——用户必须先向机器人发送消息。
- **Webhook URL 必须对网关可访问**——如果你的网关在代理后面，请设置 `webhookPublicUrl`。
- **媒体上传有限制**——机器人 API 不支持媒体文件上传，媒体内容会以 URL 形式发送。
- **私聊和房间的区分**——Webhook 负载本身不区分私聊与房间。如果你需要区分，请设置 `apiUser` 和 `apiPassword` 来启用房间类型查询功能，否则私聊会被当作普通房间处理。

## 私聊访问控制

谁可以和机器人私聊？默认配置如下：

- **默认行为**：`channels.nextcloud-talk.dmPolicy = "pairing"`，未知发送者会收到一个配对码。
- **批准配对请求**：
  - 查看待批准列表：`openclaw pairing list nextcloud-talk`
  - 批准请求：`openclaw pairing approve nextcloud-talk `
- **开放私聊**：如果你想允许任何人私聊，设置 `channels.nextcloud-talk.dmPolicy="open"` 并配合 `channels.nextcloud-talk.allowFrom=["*"]`。
- **注意**：`allowFrom` 只匹配 Nextcloud 用户 ID，显示名称不参与匹配。

## 房间（群组）设置

机器人可以加入群组房间并响应消息。以下是配置要点：

- **默认行为**：`channels.nextcloud-talk.groupPolicy = "allowlist"`，即只有被 @ 提及时才会响应。
- **配置允许的房间**：通过 `channels.nextcloud-talk.rooms` 设置：

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

- **禁用所有房间**：保持允许列表为空，或直接设置 `channels.nextcloud-talk.groupPolicy="disabled"`。

## 功能支持一览

| 功能 | 状态 |
| --- | --- |
| 私聊 | 支持 |
| 房间 | 支持 |
| 话题 | 不支持 |
| 媒体 | 仅限 URL |
| 消息反应 | 支持 |
| 原生命令 | 不支持 |

## 配置字段参考

以下是 Nextcloud Talk 频道（channel）的完整配置选项。更多通用配置请参阅 [配置](../gateway/configuration.md)。

**基础设置**

- `channels.nextcloud-talk.enabled`：是否启用该频道（channel）。
- `channels.nextcloud-talk.baseUrl`：Nextcloud 实例的 URL 地址。
- `channels.nextcloud-talk.botSecret`：机器人的共享密钥。
- `channels.nextcloud-talk.botSecretFile`：密钥文件的路径（适合不想在配置中直接写密钥的场景）。

**API 访问（用于私聊检测）**

- `channels.nextcloud-talk.apiUser`：用于房间查询的 API 用户名。
- `channels.nextcloud-talk.apiPassword`：API 或应用密码。
- `channels.nextcloud-talk.apiPasswordFile`：API 密码文件的路径。

**Webhook 设置**

- `channels.nextcloud-talk.webhookPort`：webhook 监听端口（默认：8788）。
- `channels.nextcloud-talk.webhookHost`：webhook 监听主机（默认：0.0.0.0）。
- `channels.nextcloud-talk.webhookPath`：webhook 路径（默认：/nextcloud-talk-webhook）。
- `channels.nextcloud-talk.webhookPublicUrl`：外部可访问的 webhook URL（代理场景必填）。

**访问控制**

- `channels.nextcloud-talk.dmPolicy`：私聊策略，可选值：`pairing | allowlist | open | disabled`。
- `channels.nextcloud-talk.allowFrom`：私聊允许列表（用户 ID）。设为 `open` 时需要配置 `"*"`。
- `channels.nextcloud-talk.groupPolicy`：群组策略，可选值：`allowlist | open | disabled`。
- `channels.nextcloud-talk.groupAllowFrom`：群组允许列表（用户 ID）。
- `channels.nextcloud-talk.rooms`：单个房间的设置和允许列表。

**历史消息**

- `channels.nextcloud-talk.historyLimit`：群组历史消息数量限制（0 表示不加载历史）。
- `channels.nextcloud-talk.dmHistoryLimit`：私聊历史消息数量限制（0 表示不加载历史）。
- `channels.nextcloud-talk.dms`：针对单个私聊的覆盖设置（如 historyLimit）。

**消息处理**

- `channels.nextcloud-talk.textChunkLimit`：出站文本分块大小（按字符数计算）。
- `channels.nextcloud-talk.chunkMode`：分块模式。`length`（默认）按长度分块；`newline` 先按空行（段落边界）分割，再按长度分块。
- `channels.nextcloud-talk.blockStreaming`：是否为此频道（channel）禁用块流式传输。
- `channels.nextcloud-talk.blockStreamingCoalesce`：块流式传输合并的调优参数。
- `channels.nextcloud-talk.mediaMaxMb`：入站媒体文件大小上限（单位：MB）。

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)