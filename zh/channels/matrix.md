

  消息平台

  
# Matrix

Matrix 是一个开放、去中心化的消息协议。OpenClaw 会以 Matrix **用户**的身份连接到任意家庭服务器（homeserver），所以你需要为机器人准备一个 Matrix 账户。登录成功后，你可以直接给机器人发私信，或者把它邀请到房间（Matrix 里的"群组"）。Beeper 也是一种可用的客户端，但需要启用 E2EE 加密。

**支持状态**：通过插件 `@vector-im/matrix-bot-sdk` 提供。支持私信、房间、线程、媒体、表情反应、投票（发送 + 投票开始以文本形式）、位置分享，以及 E2EE 端到端加密（需加密模块支持）。

## 需要安装插件

Matrix 以插件形式提供，不包含在核心安装包中。通过 CLI 从 npm 安装：

```bash
openclaw plugins install @openclaw/matrix
```

如果你是从 git 仓库本地运行，可以这样安装：

```bash
openclaw plugins install ./extensions/matrix
```

在配置或引导流程中选择 Matrix 时，如果检测到 git 检出，OpenClaw 会自动提供本地安装路径。详情参见 [插件](../tools/plugin.md)。

## 设置步骤

1. **安装 Matrix 插件**：
   - 从 npm 安装：`openclaw plugins install @openclaw/matrix`
   - 从本地检出安装：`openclaw plugins install ./extensions/matrix`

2. **在家庭服务器上创建 Matrix 账户**：
   - 浏览托管选项：[https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   - 或者自己搭建服务器。

3. **获取机器人账户的访问令牌**：

   使用 `curl` 在你的家庭服务器上调用 Matrix 登录 API：

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   - 把 `matrix.example.org` 替换成你的家庭服务器 URL。
   - 或者直接配置 `channels.matrix.userId` + `channels.matrix.password`：OpenClaw 会自动调用登录接口，把访问令牌存到 `~/.openclaw/credentials/matrix/credentials.json`，下次启动时复用。

4. **配置凭据**：
   - 环境变量方式：`MATRIX_HOMESERVER`、`MATRIX_ACCESS_TOKEN`（或 `MATRIX_USER_ID` + `MATRIX_PASSWORD`）
   - 配置文件方式：`channels.matrix.*`
   - 如果两种都设了，配置文件优先。
   - 使用访问令牌时：用户 ID 会通过 `/whoami` 接口自动获取。
   - 手动设置 `channels.matrix.userId` 时，需要填写完整的 Matrix ID（如 `@bot:example.org`）。

5. **重启网关**（或完成引导流程）。

6. **开始使用**：从任意 Matrix 客户端（Element、Beeper 等，详见 [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)）给机器人发私信，或邀请它加入房间。Beeper 需要 E2EE，所以记得设置 `channels.matrix.encryption: true` 并验证设备。

**最简配置**（使用访问令牌，用户 ID 自动获取）：

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

**E2EE 加密配置**：

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## 端到端加密 (E2EE)

端到端加密**已支持**，使用的是 Rust 加密 SDK。设置 `channels.matrix.encryption: true` 启用：

- 加密模块加载成功后，加密房间的消息会自动解密。
- 发送到加密房间的媒体文件会自动加密。
- 首次连接时，OpenClaw 会向你的其他会话请求设备验证。
- 在另一个 Matrix 客户端（如 Element）中批准验证请求，才能启用密钥共享。
- 如果加密模块加载失败，E2EE 会被禁用，加密房间的消息无法解密，OpenClaw 会记录警告日志。
- 如果看到加密模块缺失的错误（如 `@matrix-org/matrix-sdk-crypto-nodejs-*`），请允许 `@matrix-org/matrix-sdk-crypto-nodejs` 的构建脚本，然后运行 `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs`，或者用 `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` 下载二进制文件。

加密状态按「账户 + 访问令牌」存储在 `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/` 目录下（SQLite 数据库）。同步状态存在同目录的 `bot-storage.json` 中。如果访问令牌（即设备）变了，会创建新的存储，机器人需要重新验证才能访问加密房间。

**设备验证流程**：启用 E2EE 后，机器人启动时会向你的其他会话发起验证请求。打开 Element（或其他客户端），批准验证请求以建立信任关系。验证完成后，机器人就能解密加密房间里的消息了。

## 多账户支持

使用 `channels.matrix.accounts` 配置多账户，每个账户可以有独立的凭据和可选的 `name` 名称。配置模式参考 [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)。每个账户作为独立的 Matrix 用户运行，可以连接到不同的家庭服务器。账户级配置会继承顶层 `channels.matrix` 的设置，也可以覆盖任意选项（如 DM 策略、群组、加密等）。

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "主助手",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "提醒机器人",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

**注意事项**：

- 账户启动是串行的，避免并发导入模块时出现竞争条件。
- 环境变量（`MATRIX_HOMESERVER`、`MATRIX_ACCESS_TOKEN` 等）只对**默认账户**生效。
- 基础频道设置（DM 策略、群组策略、提及触发等）会应用到所有账户，除非在单个账户中被覆盖。
- 使用 `bindings[].match.accountId` 可以把不同账户路由到不同的智能体（agent）。
- 加密状态按账户 + 访问令牌独立存储（每个账户有各自的密钥库）。

## 路由模型

- 回复消息始终发回 Matrix。
- 私信共享智能体（agent）的主会话；房间映射到群组会话。

## 私信访问控制

- **默认配置**：`channels.matrix.dm.policy = "pairing"`。未知的发送者会收到配对码。
- **批准配对**：
  - `openclaw pairing list matrix`
  - `openclaw pairing approve matrix `
- **公开私信模式**：设置 `channels.matrix.dm.policy="open"` 并加上 `channels.matrix.dm.allowFrom=["*"]`。
- `channels.matrix.dm.allowFrom` 接受完整的 Matrix 用户 ID（如 `@user:server`）。配置向导会在目录搜索找到唯一精确匹配时，把显示名解析成用户 ID。
- **不要**用显示名或纯本地部分（如 `"Alice"` 或 `"alice"`），它们有歧义，在允许列表匹配时会被忽略。请使用完整的 `@user:server` ID。

## 房间（群组）

- **默认配置**：`channels.matrix.groupPolicy = "allowlist"`（提及触发模式）。可以用 `channels.defaults.groupPolicy` 设置全局默认值。
- **运行时注意**：如果 `channels.matrix` 完全没配置，房间检查会回退到 `groupPolicy="allowlist"`（即使设置了 `channels.defaults.groupPolicy`）。
- 用 `channels.matrix.groups` 设置房间允许列表（支持房间 ID 或别名；名称会在目录搜索找到唯一精确匹配时解析成 ID）：

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

- 设置 `requireMention: false` 可在该房间启用自动回复（不需要提及）。
- `groups."*"` 可以为所有房间设置提及触发的默认行为。
- `groupAllowFrom` 限制哪些用户可以在房间触发机器人（使用完整的 Matrix 用户 ID）。
- 每个房间的 `users` 允许列表可以进一步限制特定房间内的发送者（使用完整的 Matrix 用户 ID）。
- 配置向导会引导你输入房间允许列表（房间 ID、别名或名称），名称只在精确唯一匹配时才会解析。
- 启动时，OpenClaw 会把允许列表中的房间/用户名称解析成 ID 并记录映射；解析失败的条目在匹配时会被忽略。
- 邀请默认自动接受；用 `channels.matrix.autoJoin` 和 `channels.matrix.autoJoinAllowlist` 控制此行为。
- 要**禁止所有房间**，设置 `channels.matrix.groupPolicy: "disabled"`（或保持允许列表为空）。
- 旧版配置键：`channels.matrix.rooms`（格式同 `groups`）。

## 线程支持

- 支持回复线程。
- `channels.matrix.threadReplies` 控制回复是否留在线程内：
  - `off`、`inbound`（默认）、`always`
- `channels.matrix.replyToMode` 控制不在线程内回复时的回复元数据：
  - `off`（默认）、`first`、`all`

## 功能支持

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 房间 | ✅ 支持 |
| 线程 | ✅ 支持 |
| 媒体 | ✅ 支持 |
| E2EE | ✅ 支持（需加密模块） |
| 表情反应 | ✅ 支持（通过工具发送/读取） |
| 投票 | ✅ 支持发送；入站投票开始转为文本（响应/结束忽略） |
| 位置 | ✅ 支持（geo URI；忽略海拔） |
| 原生命令 | ✅ 支持 |

## 故障排除

先依次运行这些诊断命令：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后根据需要检查私信配对状态：

```bash
openclaw pairing list matrix
```

**常见问题**：

- 已登录但房间消息被忽略：房间被 `groupPolicy` 或房间允许列表阻止。
- 私信被忽略：`channels.matrix.dm.policy="pairing"` 时，发送者等待批准。
- 加密房间失败：加密模块或加密设置有问题。

更多排查流程：[/channels/troubleshooting](./troubleshooting.md)。

## 配置参考 (Matrix)

完整配置说明：[配置](../gateway/configuration.md)。以下是 Matrix 频道（channel）的配置选项：

- `channels.matrix.enabled`：启用/禁用频道。
- `channels.matrix.homeserver`：家庭服务器 URL。
- `channels.matrix.userId`：Matrix 用户 ID（使用访问令牌时可选）。
- `channels.matrix.accessToken`：访问令牌。
- `channels.matrix.password`：登录密码（会自动存储令牌）。
- `channels.matrix.deviceName`：设备显示名称。
- `channels.matrix.encryption`：启用 E2EE（默认：false）。
- `channels.matrix.initialSyncLimit`：初始同步限制。
- `channels.matrix.threadReplies`：`off | inbound | always`（默认：inbound）。
- `channels.matrix.textChunkLimit`：出站文本分块大小（字符数）。
- `channels.matrix.chunkMode`：`length`（默认）或 `newline`（在长度分块前先按空行/段落边界分割）。
- `channels.matrix.dm.policy`：`pairing | allowlist | open | disabled`（默认：pairing）。
- `channels.matrix.dm.allowFrom`：私信允许列表（完整的 Matrix 用户 ID）。`open` 模式需要 `"*"`。配置向导会尽可能把名称解析成 ID。
- `channels.matrix.groupPolicy`：`allowlist | open | disabled`（默认：allowlist）。
- `channels.matrix.groupAllowFrom`：群组消息允许的发送者（完整的 Matrix 用户 ID）。
- `channels.matrix.allowlistOnly`：强制对私信和房间使用允许列表规则。
- `channels.matrix.groups`：群组允许列表 + 每房间设置映射。
- `channels.matrix.rooms`：旧版群组允许列表/配置。
- `channels.matrix.replyToMode`：线程/标签的回复模式。
- `channels.matrix.mediaMaxMb`：入站/出站媒体大小上限（MB）。
- `channels.matrix.autoJoin`：邀请处理方式（`always | allowlist | off`，默认：always）。
- `channels.matrix.autoJoinAllowlist`：允许自动加入的房间 ID/别名。
- `channels.matrix.accounts`：多账户配置，以账户 ID 为键（每个账户继承顶层设置）。
- `channels.matrix.actions`：每个操作的工具门控（reactions/messages/pins/memberInfo/channelInfo）。

[LINE](./line.md)[Mattermost](./mattermost.md)