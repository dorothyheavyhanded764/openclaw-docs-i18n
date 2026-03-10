

  消息平台

  
# Signal

状态：外部 CLI 集成。网关通过 HTTP JSON-RPC + SSE 与 `signal-cli` 通信。

## 先决条件

在开始之前，请确保你具备以下条件：

-   服务器上已安装 OpenClaw（以下 Linux 流程在 Ubuntu 24 上测试通过）
-   网关运行的主机上已安装 `signal-cli`
-   一个可以接收验证短信的手机号码（用于短信注册方式）
-   注册时需要浏览器访问 Signal 验证码网站 (`signalcaptchas.org`)

## 快速设置（新手友好）

按照以下步骤快速启动你的 Signal 机器人：

1.  为机器人准备一个**独立的 Signal 号码**（强烈推荐）
2.  安装 `signal-cli`（如果使用 JVM 版本，需要先安装 Java）
3.  选择一种设置方式：
    -   **方式 A（二维码链接）：** 运行 `signal-cli link -n "OpenClaw"`，然后用 Signal App 扫描二维码
    -   **方式 B（短信注册）：** 通过验证码 + 短信验证注册专用号码
4.  配置 OpenClaw 并重启网关
5.  发送第一条私信，然后批准配对（`openclaw pairing approve signal `）

最小配置示例：

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

核心字段说明：

| 字段 | 说明 |
| --- | --- |
| `account` | 机器人的电话号码，E.164 格式（如 `+15551234567`） |
| `cliPath` | `signal-cli` 的路径；如果在 `PATH` 环境变量中，直接填 `signal-cli` 即可 |
| `dmPolicy` | 私信访问策略（推荐使用 `pairing`） |
| `allowFrom` | 允许发送私信的电话号码或 `uuid:` 值列表 |

## 什么是 Signal 频道？

本节帮你理解 Signal 频道的工作原理：

-   通过 `signal-cli` 实现 Signal 消息收发（非嵌入式 libsignal 方案）
-   确定性路由：回复消息始终返回给同一个 Signal 用户或群组
-   私信会话共享智能体（agent）的主会话；群组会话相互隔离（格式为 `agent::signal:group:`）

## 配置写入权限

默认情况下，Signal 频道允许通过 `/config set|unset` 命令触发配置更新（需要 `commands.config: true`）。如果需要禁用此功能：

```json
{
  channels: { signal: { configWrites: false } },
}
```

## 号码模型（重要概念）

理解这一点很重要，能帮你避免常见问题：

-   网关连接的是一个 **Signal 设备**（即 `signal-cli` 所在的账户）
-   如果你在**个人 Signal 账户**上运行机器人，它会自动忽略你自己发送的消息（防止消息循环）
-   想要实现"我给机器人发消息，它回复我"的场景，请使用**独立的机器人号码**

## 设置方式 A：链接现有 Signal 账户（二维码扫描）

适合已有 Signal 账户、想快速添加机器人设备的用户。

1.  安装 `signal-cli`（JVM 版本或原生版本均可）
2.  链接机器人账户：
    -   运行 `signal-cli link -n "OpenClaw"`，然后用 Signal App 扫描生成的二维码
3.  配置 Signal 并启动网关

配置示例：

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

**多账户支持：** 使用 `channels.signal.accounts` 为每个账户单独配置，可指定 `name` 字段。共享模式详见 [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。

## 设置方式 B：注册专用机器人号码（短信验证，Linux）

适合需要独立机器人号码、不想链接个人 Signal 账户的场景。

1.  准备一个可以接收短信的号码（固定电话可使用语音验证）
    -   使用独立号码可避免账户/会话冲突
2.  在网关主机上安装 `signal-cli`：

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

如果使用 JVM 版本（`signal-cli-${VERSION}.tar.gz`），需要先安装 JRE 25+。请保持 `signal-cli` 更新——官方提示旧版本可能因 Signal 服务器 API 变更而失效。

3.  注册并验证号码：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

如果提示需要验证码：

1.  访问 `https://signalcaptchas.org/registration/generate.html`
2.  完成验证码验证，复制"Open Signal"按钮中的 `signalcaptcha://...` 链接
3.  尽量从与浏览器相同的外部 IP 执行后续命令
4.  立即重新运行注册命令（验证码令牌过期很快）：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4.  配置 OpenClaw、重启网关、验证频道状态：

```bash
# 如果你以用户 systemd 服务方式运行网关：
systemctl --user restart openclaw-gateway

# 然后验证：
openclaw doctor
openclaw channels status --probe
```

5.  配对私信发送者：
    -   向机器人号码发送任意消息
    -   在服务器上批准配对码：`openclaw pairing approve signal <PAIRING_CODE>`
    -   将机器人号码保存为手机联系人，避免显示"未知联系人"

**重要提醒：** 使用 `signal-cli` 注册电话号码账户可能导致该号码的主 Signal App 会话失效。建议使用独立的机器人号码；如果需要保留现有手机 App 设置，请使用二维码链接方式。

上游参考资料：

-   `signal-cli` README: `https://github.com/AsamK/signal-cli`
-   验证码流程: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   设备链接流程: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## 外部守护进程模式（httpUrl）

如果你希望自己管理 `signal-cli`（解决 JVM 冷启动慢、容器初始化、共享 CPU 等问题），可以单独运行守护进程，让 OpenClaw 连接到它：

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

这样配置后，OpenClaw 会跳过自动启动守护进程和启动等待。如果自动启动时启动较慢，可设置 `channels.signal.startupTimeoutMs` 延长超时时间。

## 访问控制（私信 + 群组）

### 私信访问控制

-   默认策略：`channels.signal.dmPolicy = "pairing"`
-   未知发送者会收到配对码；消息在批准前会被忽略（配对码 1 小时后过期）
-   批准配对的方式：
    -   `openclaw pairing list signal` 查看待批准列表
    -   `openclaw pairing approve signal ` 批准配对
-   配对是 Signal 私信的默认令牌交换方式。详情请参阅 [配对](./pairing.md)
-   仅提供 UUID 的发送者（来自 `sourceUuid`）会以 `uuid:` 格式存储在 `channels.signal.allowFrom` 中

### 群组访问控制

-   `channels.signal.groupPolicy` 可选值：`open | allowlist | disabled`
-   `channels.signal.groupAllowFrom` 控制当 `groupPolicy` 设为 `allowlist` 时，哪些用户可以在群组中触发机器人
-   运行时注意：如果 `channels.signal` 配置完全缺失，运行时会回退到 `groupPolicy="allowlist"` 进行群组检查（即使设置了 `channels.defaults.groupPolicy`）

## 工作原理

了解 Signal 频道的内部工作机制：

-   `signal-cli` 以守护进程方式运行；网关通过 SSE（Server-Sent Events）读取事件
-   入站消息被规范化为统一的频道信封格式
-   回复消息始终路由回原始发送者号码或群组

## 媒体处理与限制

以下是 Signal 频道的媒体处理规则：

-   出站文本按 `channels.signal.textChunkLimit` 分块（默认 4000 字符）
-   可选的换行分块模式：设置 `channels.signal.chunkMode="newline"` 在长度分块前先按空行（段落边界）分割
-   支持附件（从 `signal-cli` 获取 base64 编码）
-   默认媒体大小上限：`channels.signal.mediaMaxMb`（默认 8 MB）
-   设置 `channels.signal.ignoreAttachments` 可跳过下载媒体文件
-   群组历史上下文使用 `channels.signal.historyLimit`（或 `channels.signal.accounts.*.historyLimit`），回退到 `messages.groupChat.historyLimit`。设为 `0` 可禁用（默认 50 条）

## 输入提示与已读回执

Signal 频道支持以下交互提示：

-   **输入中指示器：** OpenClaw 通过 `signal-cli sendTyping` 发送输入提示，并在智能体（agent）生成回复时持续刷新
-   **已读回执：** 当 `channels.signal.sendReadReceipts` 为 `true` 时，OpenClaw 会为已批准的私信转发已读回执
-   注意：`signal-cli` 不支持群组的已读回执

## 消息反应（Reaction）

使用消息工具添加表情反应：

-   使用 `message action=react` 命令，指定 `channel=signal`
-   目标参数：发送者 E.164 号码或 UUID（使用配对输出中的 `uuid:` 格式；裸 UUID 也可以）
-   `messageId` 是目标消息的 Signal 时间戳
-   群组反应需要额外指定 `targetAuthor` 或 `targetAuthorUuid`

示例：

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

配置选项：

-   `channels.signal.actions.reactions`：启用/禁用反应功能（默认 `true`）
-   `channels.signal.reactionLevel`：可选值 `off | ack | minimal | extensive`
    -   `off`/`ack` 禁用智能体（agent）反应功能（调用消息工具 `react` 会报错）
    -   `minimal`/`extensive` 启用智能体反应功能，并设置指导级别
-   单账户覆盖：`channels.signal.accounts..actions.reactions`、`channels.signal.accounts..reactionLevel`

## 投递目标（CLI/cron 调用）

从命令行或定时任务发送消息时，使用以下目标格式：

-   私信：`signal:+15551234567`（或直接使用 E.164 格式号码）
-   UUID 私信：`uuid:`（或直接使用 UUID）
-   群组：`signal:group:`
-   用户名：`username:`（如果你的 Signal 账户支持用户名功能）

## 故障排除

遇到问题时，按以下顺序排查：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

如果私信有问题，检查配对状态：

```bash
openclaw pairing list signal
```

### 常见问题

-   **守护进程可达但无回复：** 检查账户/守护进程设置（`httpUrl`、`account`）和接收模式
-   **私信被忽略：** 发送者还在等待配对批准
-   **群组消息被忽略：** 群组发送者/提及权限阻止了消息投递
-   **编辑配置后验证报错：** 运行 `openclaw doctor --fix` 修复
-   **诊断信息中缺少 Signal：** 确认 `channels.signal.enabled: true`

### 进阶排查

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

完整故障排除流程请参考：[/channels/troubleshooting](./troubleshooting.md)

## 安全注意事项

保护你的 Signal 机器人账户：

-   `signal-cli` 在本地存储账户密钥（通常在 `~/.local/share/signal-cli/data/` 目录）
-   服务器迁移或重建前，务必备份 Signal 账户状态
-   除非明确需要开放私信访问，否则保持 `channels.signal.dmPolicy: "pairing"`
-   短信验证仅在注册或恢复流程中使用，但失去对号码/账户的控制会使重新注册变得复杂

## Signal 频道配置参考

完整配置请参阅 [配置](../gateway/configuration.md)。以下是 Signal 特有的配置选项：

### 基础配置

-   `channels.signal.enabled`：启用/禁用 Signal 频道
-   `channels.signal.account`：机器人账户的 E.164 格式号码
-   `channels.signal.cliPath`：`signal-cli` 可执行文件路径
-   `channels.signal.httpUrl`：完整的守护进程 URL（会覆盖 host/port 设置）
-   `channels.signal.httpHost`、`channels.signal.httpPort`：守护进程绑定地址（默认 127.0.0.1:8080）
-   `channels.signal.autoStart`：是否自动启动守护进程（未设置 `httpUrl` 时默认 `true`）
-   `channels.signal.startupTimeoutMs`：启动等待超时时间（毫秒，上限 120000）

### 接收与消息处理

-   `channels.signal.receiveMode`：接收模式，可选 `on-start | manual`
-   `channels.signal.ignoreAttachments`：跳过附件下载
-   `channels.signal.ignoreStories`：忽略 Stories 消息
-   `channels.signal.sendReadReceipts`：转发已读回执

### 访问控制

-   `channels.signal.dmPolicy`：私信策略，可选 `pairing | allowlist | open | disabled`（默认 `pairing`）
-   `channels.signal.allowFrom`：私信允许列表（E.164 号码或 `uuid:`）。设为 `open` 需要 `"*"`
-   `channels.signal.groupPolicy`：群组策略，可选 `open | allowlist | disabled`（默认 `allowlist`）
-   `channels.signal.groupAllowFrom`：群组发送者允许列表

### 历史与分块

-   `channels.signal.historyLimit`：群组历史消息上下文数量（`0` 禁用）
-   `channels.signal.dmHistoryLimit`：私信历史消息限制（用户轮次）。单用户覆盖：`channels.signal.dms["<phone_or_uuid>"].historyLimit`
-   `channels.signal.textChunkLimit`：出站消息分块大小（字符数）
-   `channels.signal.chunkMode`：分块模式，`length`（默认）或 `newline`（先按空行/段落边界分割，再按长度分块）
-   `channels.signal.mediaMaxMb`：入站/出站媒体大小上限（MB）

### 相关全局选项

-   `agents.list[].groupChat.mentionPatterns`（Signal 不支持原生提及功能）
-   `messages.groupChat.mentionPatterns`（全局回退配置）
-   `messages.responsePrefix`

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)