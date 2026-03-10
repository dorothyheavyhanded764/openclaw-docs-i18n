

  消息平台

  
# iMessage

> **⚠️** 新部署 iMessage 请使用 [BlueBubbles](./bluebubbles.md)。`imsg` 集成属于旧版功能，未来版本可能会移除。

状态：旧版外部 CLI 集成。网关（gateway）会启动 `imsg rpc` 进程，通过 stdio 上的 JSON-RPC 进行通信（无需单独的守护进程或端口）。

## 快速设置

## 要求和权限（macOS）

在 macOS 上运行 iMessage 频道，需要满足以下条件：

-   运行 `imsg` 的 Mac 必须已在 Messages 中登录。
-   运行 OpenClaw 或 `imsg` 的进程上下文需要"完全磁盘访问"权限（Full Disk Access），这是为了访问 Messages 数据库。
-   需要授予"自动化"权限（Automation），以便通过 Messages.app 发送消息。

> **💡** 权限是按进程上下文授予的。如果你的网关以无头模式（headless）运行——比如通过 LaunchAgent 或 SSH——需要在同一上下文中运行一次交互式命令来触发权限提示：
> 
> 复制
> 
> ```
> imsg chats --limit 1
> # 或者
> imsg send <handle> "test"
> ```

## 访问控制与路由

`channels.imessage.dmPolicy` 字段控制直接消息（direct messages）的处理方式：

-   `pairing`（默认值）——需要配对批准后才能交互
-   `allowlist`——只允许白名单中的发送者
-   `open`——开放模式，但要求 `allowFrom` 包含 `"*"`
-   `disabled`——禁用直接消息

白名单字段是 `channels.imessage.allowFrom`。白名单条目可以是用户句柄（handle）或聊天目标，支持的格式包括 `chat_id:*`、`chat_guid:*`、`chat_identifier:*`。

`channels.imessage.groupPolicy` 控制群组消息的处理方式：

-   `allowlist`（配置时的默认值）
-   `open`
-   `disabled`

群组发送者白名单通过 `channels.imessage.groupAllowFrom` 设置。运行时回退机制：如果未设置 `groupAllowFrom`，iMessage 会尝试使用 `allowFrom` 作为群组发送者检查的回退。需要注意：如果完全缺少 `channels.imessage` 配置，运行时会回退到 `groupPolicy="allowlist"` 并记录警告（即使已设置 `channels.defaults.groupPolicy`）。

关于群组的提及门控（mention gating）：

-   iMessage 本身没有原生的提及元数据
-   提及检测依赖正则表达式模式匹配（配置路径：`agents.list[].groupChat.mentionPatterns`，回退使用 `messages.groupChat.mentionPatterns`）
-   如果没有配置任何模式，就无法强制执行提及门控

已授权发送者发送的控制命令可以绕过群组中的提及门控限制。

这部分说明消息如何路由到智能体（agent）的会话中：

-   直接消息使用直接路由，群组消息使用群组路由
-   使用默认设置 `session.dmScope=main` 时，iMessage 直接消息会合并到智能体的主会话中
-   群组会话是相互隔离的，格式为 `agent::imessage:group:<chat_id>`
-   智能体的回复会使用原始的频道/目标元数据路由回 iMessage

"类群组线程"行为：某些多参与者的 iMessage 线程可能以 `is_group=false` 的标记到达。如果该 `chat_id` 在 `channels.imessage.groups` 中被显式配置，OpenClaw 会将其视为群组流量处理（应用群组门控规则 + 群组会话隔离）。

## 部署模式

这种模式使用独立的 Apple ID 和 macOS 用户，让机器人流量与你的个人 Messages 完全隔离。推荐的配置流程如下：

1.  创建或登录一个专用的 macOS 用户账户。
2.  在该用户账户中使用机器人的 Apple ID 登录 Messages。
3.  在该用户环境中安装 `imsg`。
4.  创建 SSH 包装脚本，让 OpenClaw 能够在该用户上下文中运行 `imsg`。
5.  将 `channels.imessage.accounts..cliPath` 和 `.dbPath` 指向该用户目录。

首次运行时可能需要在机器人用户会话中手动批准 GUI 权限提示（自动化 + 完全磁盘访问）。

这是一种常见的网络拓扑结构：

-   网关运行在 Linux 或虚拟机上
-   iMessage 和 `imsg` 运行在你 tailnet 中的某台 Mac 上
-   `cliPath` 包装脚本通过 SSH 远程执行 `imsg`
-   设置 `remoteHost` 后可以启用 SCP 附件获取功能

配置示例：

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

SSH 包装脚本示例：

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

请使用 SSH 密钥认证，确保 SSH 和 SCP 操作都是非交互式的。首次使用前，需要先手动连接一次以信任主机密钥（例如运行 `ssh bot@mac-mini.tailnet-1234.ts.net`），这样 `known_hosts` 文件中才会有相应记录。

iMessage 支持在 `channels.imessage.accounts` 下配置多个账户。每个账户可以独立覆盖以下字段：`cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb`、历史记录设置，以及附件根目录白名单。

## 媒体、分块与发送目标

处理消息附件的相关配置：

-   入站附件获取是可选的，通过 `channels.imessage.includeAttachments` 开启
-   设置 `remoteHost` 后，可以通过 SCP 获取远程附件
-   附件路径必须匹配允许的根目录白名单：
    -   本地模式使用 `channels.imessage.attachmentRoots`
    -   远程 SCP 模式使用 `channels.imessage.remoteAttachmentRoots`
    -   默认的根目录匹配模式是 `/Users/*/Library/Messages/Attachments`
-   SCP 连接使用严格的主机密钥检查（`StrictHostKeyChecking=yes`）
-   出站媒体大小限制通过 `channels.imessage.mediaMaxMb` 设置，默认为 16 MB

当智能体发送较长的消息时，会自动进行分块处理：

-   文本分块限制通过 `channels.imessage.textChunkLimit` 设置，默认 4000 字符
-   分块模式通过 `channels.imessage.chunkMode` 选择：
    -   `length`（默认）——按长度分割
    -   `newline`——优先在段落边界分割

发送消息时，推荐使用以下显式目标格式，可以获得更稳定的路由效果：

-   `chat_id:123`（推荐，路由最稳定）
-   `chat_guid:...`
-   `chat_identifier:...`

也支持直接使用用户句柄作为目标：

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

查看已有聊天的命令：

```bash
imsg chats --limit 20
```

## 配置写入

iMessage 频道默认允许通过频道发起配置写入（当 `commands.config: true` 时，支持 `/config set|unset` 命令）。如果需要禁用此功能：

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## 故障排除

首先验证 `imsg` 二进制文件和 RPC 功能是否正常：

```bash
imsg rpc --help
openclaw channels status --probe
```

如果探测结果显示 RPC 不受支持，请更新 `imsg` 到最新版本。

如果智能体没有响应直接消息，请检查以下配置：

-   `channels.imessage.dmPolicy` —— 策略是否正确
-   `channels.imessage.allowFrom` —— 白名单是否包含发送者
-   配对批准状态（运行 `openclaw pairing list imessage` 查看）

如果群组消息没有响应，请检查：

-   `channels.imessage.groupPolicy` —— 群组策略设置
-   `channels.imessage.groupAllowFrom` —— 群组发送者白名单
-   `channels.imessage.groups` —— 群组白名单配置
-   提及模式配置（`agents.list[].groupChat.mentionPatterns`）

如果无法获取远程附件，请逐一检查：

-   `channels.imessage.remoteHost` —— 远程主机地址是否正确
-   `channels.imessage.remoteAttachmentRoots` —— 远程附件根目录白名单
-   从网关主机到远程 Mac 的 SSH/SCP 密钥认证是否正常
-   主机密钥是否已添加到网关主机的 `~/.ssh/known_hosts` 中
-   运行 Messages 的 Mac 上，远程路径是否可读

如果权限提示被错过或忽略，可以在同一用户/会话上下文中打开交互式 GUI 终端，重新运行命令并批准权限提示：

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

完成后，确认运行 OpenClaw/`imsg` 的进程上下文已被授予"完全磁盘访问"和"自动化"权限。

## 配置参考链接

-   [配置参考 - iMessage](../gateway/configuration-reference.md#imessage)
-   [网关配置](../gateway/configuration.md)
-   [配对](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)