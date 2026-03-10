

  配置

  
# 配对（Pairing）

**配对（pairing）**是 OpenClaw 中一项重要的安全机制——所有者必须显式批准，才能允许特定对象访问系统。它主要用于两个场景：

1. **私信配对**：控制谁有权限与机器人对话
2. **节点配对**：控制哪些设备/节点可以加入网关网络

想了解整体安全模型？请参阅 [安全](../gateway/security.md)。

## 1) 私信配对（入站聊天访问）

当频道（channel）配置了 `pairing` 私信策略后，任何未知的发送者都会收到一个配对码。在您批准之前，他们的消息**不会被处理**。关于默认私信策略的详细说明，请参阅 [安全](../gateway/security.md)。

配对码的特点：

- 8 位大写字母，已排除容易混淆的字符（`0O1I`）
- **1 小时后过期**。机器人只会在新请求创建时发送配对消息（大致每个发送者每小时一次）
- 默认情况下，每个频道的待处理配对请求上限为 **3 个**；超出上限的请求会被忽略，直到有请求过期或被批准

### 批准发送者

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

支持的频道：`telegram`、`whatsapp`、`signal`、`imessage`、`discord`、`slack`、`feishu`。

### 状态文件存储位置

所有状态文件存储在 `~/.openclaw/credentials/` 目录下：

- 待处理请求：`-pairing.json`
- 已批准的允许列表：
  - 默认账户：`-allowFrom.json`
  - 非默认账户：`--allowFrom.json`

账户作用域说明：

- 非默认账户仅读写自己作用域内的允许列表文件
- 默认账户使用频道级别的非作用域允许列表文件

请将这些文件视为敏感信息——它们控制着谁能访问您的智能体（agent）。

## 2) 节点设备配对（iOS/Android/macOS/无头节点）

节点设备会以 `role: node` 的**设备**身份连接到网关。网关会创建一个设备配对请求，您必须批准该请求才能完成配对。

### 通过 Telegram 配对（推荐 iOS 用户）

如果您安装了 `device-pair` 插件，首次配对设备可以完全在 Telegram 内完成：

1. 在 Telegram 中向您的机器人发送：`/pair`
2. 机器人会回复两条消息：一条操作说明和一条单独的**设置代码**消息（方便在 Telegram 中复制/粘贴）
3. 在手机上打开 OpenClaw iOS 应用 → 设置 → 网关
4. 粘贴设置代码，然后点击连接
5. 回到 Telegram 发送：`/pair approve`

设置代码是 base64 编码的 JSON 数据，包含：

- `url`：网关 WebSocket URL（`ws://...` 或 `wss://...`）
- `token`：短期有效的配对令牌

设置代码在有效期内请像密码一样妥善保管。

### 批准节点设备

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### 节点配对状态存储

存储在 `~/.openclaw/devices/` 目录下：

- `pending.json`：短期有效，待处理请求会过期
- `paired.json`：已配对的设备及令牌

### 注意事项

- 遗留的 `node.pair.*` API（CLI 命令：`openclaw nodes pending/approve`）使用的是独立的网关配对存储。WebSocket 节点仍然需要进行设备配对。

## 相关文档

- 安全模型与提示注入防护：[安全](../gateway/security.md)
- 安全更新（运行 doctor）：[更新](../install/updating.md)
- 频道配置：
  - Telegram：[Telegram](./telegram.md)
  - WhatsApp：[WhatsApp](./whatsapp.md)
  - Signal：[Signal](./signal.md)
  - BlueBubbles (iMessage)：[BlueBubbles](./bluebubbles.md)
  - iMessage（旧版）：[iMessage](./imessage.md)
  - Discord：[Discord](./discord.md)
  - Slack：[Slack](./slack.md)

[Zalo 个人版](./zalouser.md) | [群组消息](./group-messages.md)