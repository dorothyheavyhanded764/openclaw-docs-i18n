

  消息平台

  
# WhatsApp

状态：通过 WhatsApp Web（Baileys）达到生产就绪。网关管理已链接的会话。

## 快速设置

本节带你完成 WhatsApp 频道的基础配置，四步即可启动运行。

### 步骤 1：配置 WhatsApp 访问策略

首先在配置文件中设置谁可以与你的智能体通信：

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### 步骤 2：链接 WhatsApp（扫码）

运行以下命令，终端会显示二维码，用 WhatsApp 扫码即可链接：

```bash
openclaw channels login --channel whatsapp
```

如果有多个账户，指定账户名称：

```bash
openclaw channels login --channel whatsapp --account work
```

### 步骤 3：启动网关

```bash
openclaw gateway
```

### 步骤 4：批准首次配对请求（使用配对模式时）

如果你配置了 `dmPolicy: "pairing"`，首次有人给你发消息时会生成配对请求，需要手动批准：

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

配对请求 1 小时后过期。每个频道的待处理请求最多保留 3 个。

> **ℹ️** OpenClaw 建议尽可能使用独立的手机号码运行 WhatsApp。虽然频道元数据和入门流程已针对这种场景优化，但使用个人号码同样受支持。

## 部署模式

这是最简洁的运营模式：

- 为 OpenClaw 使用独立的 WhatsApp 身份
- 私聊白名单和路由边界更清晰
- 减少与自己聊天的混淆

最简策略配置：

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

入门流程支持个人号码模式，会自动生成对"自聊"友好的基础配置：

- `dmPolicy: "allowlist"`
- `allowFrom` 包含你的个人号码
- `selfChatMode: true`

运行时，自聊保护机制会根据已链接的本机号码和 `allowFrom` 配置自动触发。

当前 OpenClaw 频道架构中，WhatsApp 频道基于 WhatsApp Web（Baileys）实现。内置的聊天频道注册表中没有单独的 Twilio WhatsApp 消息频道。

## 运行时模型

了解 WhatsApp 频道在运行时的行为特点：

- 网关管理 WhatsApp 套接字连接和自动重连
- 发送消息需要目标账户有活跃的 WhatsApp 监听器
- 状态广播和官方广播会被忽略（`@status`、`@broadcast`）
- 私聊使用 DM 会话规则（`session.dmScope`；默认 `main` 会把私聊合并到智能体主会话）
- 群聊会话相互隔离（格式：`agent::whatsapp:group:`）

## 访问控制与激活

`channels.whatsapp.dmPolicy` 控制谁能发起私聊：

- `pairing`（默认）：首次发消息需要配对批准
- `allowlist`：仅白名单用户可发消息
- `open`：所有人可发消息（需配置 `allowFrom: ["*"]`）
- `disabled`：禁用私聊

`allowFrom` 接受 E.164 格式的电话号码（系统会自动规范化）。

多账户覆盖：`channels.whatsapp.accounts..dmPolicy` 和 `allowFrom` 会覆盖该账户的频道级默认值。

运行时行为细节：

- 配对记录持久化到频道允许存储中，与配置的 `allowFrom` 合并生效
- 如果没有配置白名单，默认允许已链接的本机号码
- 自己发出的 `fromMe` 私聊消息不会自动配对

群组访问控制分为两层：

1. **群组成员白名单**（`channels.whatsapp.groups`）
   - 如果省略 `groups`，所有群组都符合条件
   - 如果配置了 `groups`，则作为群组白名单（支持 `"*"` 通配）
2. **群组发送者策略**（`channels.whatsapp.groupPolicy` + `groupAllowFrom`）
   - `open`：绕过发送者白名单检查
   - `allowlist`：发送者必须匹配 `groupAllowFrom`（或 `*`）
   - `disabled`：阻止所有群组入站消息

发送者白名单回退逻辑：

- 如果未设置 `groupAllowFrom`，运行时会回退使用 `allowFrom`
- 发送者白名单检查在提及/回复激活之前执行

注意：如果完全没有 `channels.whatsapp` 配置块，群组策略会回退到 `allowlist`（并输出警告日志），即使配置了 `channels.defaults.groupPolicy` 也不会生效。

默认情况下，群聊中的机器人需要被 @ 提及才会回复。提及检测包括：

- 显式的 WhatsApp @ 提及（提及了机器人身份）
- 配置的提及正则模式（`agents.list[].groupChat.mentionPatterns`，回退到 `messages.groupChat.mentionPatterns`）
- 隐式的回复检测（回复的发送者与机器人身份匹配）

安全提示：

- 引用/回复只满足提及触发条件，**不会**授予发送者权限
- 配置 `groupPolicy: "allowlist"` 时，非白名单发送者即使回复了白名单用户的消息也会被拦截

会话级激活命令：

- `/activation mention`：仅响应提及
- `/activation always`：总是响应

`activation` 命令更新的是会话状态（不是全局配置），且需要所有者权限。

## 个人号码与自聊行为

当已链接的本机号码也在 `allowFrom` 中时，WhatsApp 自聊保护机制会自动激活：

- 跳过自聊消息的已读回执
- 忽略会 ping 到自己的提及 JID 自动触发行为
- 如果未设置 `messages.responsePrefix`，自聊回复默认添加 `[{identity.name}]` 或 `[openclaw]` 前缀

## 消息规范化与上下文

收到的 WhatsApp 消息会被封装成统一的消息格式。如果消息引用了其他消息，会附带上下文信息：

```json
[回复 <sender> id:<stanzaId>]
<被引用的消息正文或媒体占位符>
[/回复]
```

回复相关的元数据字段也会填充（`ReplyToId`、`ReplyToBody`、`ReplyToSender`、发送者 JID/E.164）。

纯媒体的入站消息会规范化为占位符格式：

- `<media:image>`
- `<media:video>`
- `<media:audio>`
- `<media:document>`
- `<media:sticker>`

位置和联系人消息在路由前会转换为文本上下文。

在群聊中，机器人被触发前的消息可以被缓冲，触发时作为上下文注入：

- 默认上限：`50` 条
- 配置项：`channels.whatsapp.historyLimit`
- 回退配置：`messages.groupChat.historyLimit`
- 设为 `0` 可禁用

注入时会添加标记：

- `[自上次回复以来的聊天消息 - 供参考]`
- `[当前消息 - 请回复此条]`

默认对已接受的入站消息发送已读回执。全局禁用：

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

按账户单独配置：

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

自聊消息即使全局启用已读回执也会跳过。

## 消息送达、分块与媒体

长消息会自动分块发送：

- 默认分块长度：`channels.whatsapp.textChunkLimit = 4000`
- 分块模式：`channels.whatsapp.chunkMode = "length" | "newline"`
- `newline` 模式优先在段落边界（空行）处分块，不够时再按长度分块

支持发送多种媒体类型：

- 图片、视频、音频（PTT 语音消息）、文档
- `audio/ogg` 会自动转换为 `audio/ogg; codecs=opus` 以兼容语音消息
- 通过 `gifPlayback: true` 支持动态 GIF 播放
- 发送多媒体时，标题会应用到第一个媒体项
- 媒体源支持 HTTP(S)、`file://` 或本地路径

- 入站媒体保存上限：`channels.whatsapp.mediaMaxMb`（默认 `50` MB）
- 出站媒体发送上限：`channels.whatsapp.mediaMaxMb`（默认 `50` MB）
- 按账户覆盖：`channels.whatsapp.accounts..mediaMaxMb`
- 图片会自动优化（调整尺寸/压缩质量）以适应限制
- 媒体发送失败时，会发送文本警告提示，而不是静默丢弃响应

## 确认表情反应

WhatsApp 支持在收到消息时立即回复一个表情作为确认：

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

行为说明：

- 在入站消息被接受后、回复发送前立即发送
- 发送失败会记录日志，但不会阻止正常回复的送达
- 群聊模式 `mentions` 仅在提及触发的回合反应；群组激活设为 `always` 可绕过此检查
- WhatsApp 频道使用 `channels.whatsapp.ackReaction`（旧版 `messages.ackReaction` 不适用）

## 多账户与凭据管理

- 账户 ID 在 `channels.whatsapp.accounts` 中定义
- 默认账户选择：优先使用 `default`，不存在则使用第一个配置的账户 ID（按字母排序）
- 账户 ID 内部会规范化处理用于查找

- 当前认证文件路径：`~/.openclaw/credentials/whatsapp//creds.json`
- 备份文件：`creds.json.bak`
- 旧版默认认证目录 `~/.openclaw/credentials/` 中的凭据在默认账户流程中仍可识别和迁移

`openclaw channels logout --channel whatsapp [--account ]` 会清除指定账户的 WhatsApp 认证状态。

在旧版认证目录中，登出时会保留 `oauth.json`，仅删除 Baileys 认证文件。

## 工具、操作与配置写入

- 智能体工具支持 WhatsApp 表情反应操作（`react`）
- 操作权限门控：
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- 频道发起的配置写入默认启用（通过 `channels.whatsapp.configWrites=false` 可禁用）

## 故障排除

症状：频道状态显示未链接。

解决方法：

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

症状：账户已链接但反复断开或尝试重连。

解决方法：

```bash
openclaw doctor
openclaw logs --follow
```

如需重新链接，运行 `channels login`。

出站消息发送失败，提示没有活跃监听器。

确保网关正在运行且账户已正确链接。

按以下顺序排查：

- `groupPolicy` 配置
- `groupAllowFrom` / `allowFrom` 白名单
- `groups` 群组白名单条目
- 提及门控（`requireMention` + 提及模式）
- `openclaw.json`（JSON5 格式）中的重复键：后面的配置会覆盖前面的，确保每个作用域只有一个 `groupPolicy`

WhatsApp 网关运行时应使用 Node.js。Bun 运行时被标记为与稳定的 WhatsApp/Telegram 网关操作不兼容。

## 配置参考

主要参考文档：

- [配置参考 - WhatsApp](../gateway/configuration-reference.md#whatsapp)

常用 WhatsApp 配置字段：

- 访问控制：`dmPolicy`、`allowFrom`、`groupPolicy`、`groupAllowFrom`、`groups`
- 消息送达：`textChunkLimit`、`chunkMode`、`mediaMaxMb`、`sendReadReceipts`、`ackReaction`
- 多账户：`accounts..enabled`、`accounts..authDir`、账户级覆盖
- 运维配置：`configWrites`、`debounceMs`、`web.enabled`、`web.heartbeatSeconds`、`web.reconnect.*`
- 会话行为：`session.dmScope`、`historyLimit`、`dmHistoryLimit`、`dms..historyLimit`

## 相关链接

- [配对机制](./pairing.md)
- [频道路由](./channel-routing.md)
- [多智能体路由](../concepts/multi-agent.md)
- [故障排除](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)