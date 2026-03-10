

  消息平台

  
# Twitch

通过 IRC 连接实现 Twitch 聊天支持。OpenClaw 以 Twitch 用户身份（机器人账户）连接，在频道（channel）中收发消息。

## 安装插件

Twitch 以插件形式提供，不包含在核心安装中。通过 CLI 安装（npm 注册表）：

```bash
openclaw plugins install @openclaw/twitch
```

从本地 git 仓库运行时：

```bash
openclaw plugins install ./extensions/twitch
```

详情参阅：[插件](../tools/plugin.md)

## 快速上手

按以下步骤完成基础配置：

1. 创建一个专用的 Twitch 账户作为机器人（或使用现有账户）。
2. 生成凭证：访问 [Twitch Token Generator](https://twitchtokengenerator.com/)
   - 选择 **Bot Token**
   - 确认已勾选 `chat:read` 和 `chat:write` 权限范围
   - 复制 **Client ID** 和 **Access Token**
3. 获取您的 Twitch 用户 ID：[https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4. 配置令牌：
   - 环境变量方式：`OPENCLAW_TWITCH_ACCESS_TOKEN=...`（仅适用于默认账户）
   - 配置文件方式：`channels.twitch.accessToken`
   - 两者同时设置时，配置文件优先（环境变量回退仅适用于默认账户）
5. 启动网关。

**⚠️ 重要：** 务必配置访问控制（`allowFrom` 或 `allowedRoles`），防止未授权用户触发机器人。`requireMention` 默认为 `true`。最小配置示例：

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // 机器人的 Twitch 账户
      accessToken: "oauth:abc123...", // OAuth 访问令牌（或使用 OPENCLAW_TWITCH_ACCESS_TOKEN 环境变量）
      clientId: "xyz789...", // 来自令牌生成器的 Client ID
      channel: "vevisk", // 要加入的 Twitch 聊天频道（必需）
      allowFrom: ["123456789"], // （推荐）仅允许您的 Twitch 用户 ID - 从 https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/ 获取
    },
  },
}
```

## 工作原理

- 网关拥有一个 Twitch 频道连接
- 确定性路由：回复始终发回 Twitch
- 每个账户映射到独立的会话键 `agent::twitch:`
- `username` 是机器人账户（用于身份验证），`channel` 是要加入的聊天室

## 详细配置

### 生成凭证

使用 [Twitch Token Generator](https://twitchtokengenerator.com/)：

- 选择 **Bot Token**
- 确认已勾选 `chat:read` 和 `chat:write` 权限范围
- 复制 **Client ID** 和 **Access Token**

无需手动注册应用。令牌会在数小时后过期。

### 配置机器人

**环境变量方式（仅适用于默认账户）：**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**配置文件方式：**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

环境变量和配置文件同时设置时，配置文件优先。

### 访问控制（强烈推荐）

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // （推荐）仅允许您的 Twitch 用户 ID
    },
  },
}
```

推荐使用 `allowFrom` 作为硬性允许列表。如需基于角色的访问控制，请改用 `allowedRoles`。

**可用角色：** `"moderator"`、`"owner"`、`"vip"`、`"subscriber"`、`"all"`

**为什么要用用户 ID？** 用户名可能会变更，存在被冒充的风险。用户 ID 是永久不变的。获取您的 Twitch 用户 ID：[https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)

## 令牌自动刷新（可选）

通过 [Twitch Token Generator](https://twitchtokengenerator.com/) 获取的令牌无法自动刷新，过期后需重新生成。

如需自动刷新令牌，请在 [Twitch 开发者控制台](https://dev.twitch.tv/console) 创建自己的 Twitch 应用，并在配置中添加：

```json
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

机器人会在令牌过期前自动刷新，并记录刷新事件。

## 多账户支持

使用 `channels.twitch.accounts` 为每个账户配置独立令牌。共享配置模式请参阅 [`gateway/configuration`](../gateway/configuration.md)。示例（一个机器人账户连接两个频道）：

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**注意：** 每个账户需要独立的令牌（每个频道一个令牌）。

## 访问控制详解

### 基于角色的限制

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### 用户 ID 允许列表（最安全）

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### 基于角色的访问控制（替代方案）

`allowFrom` 是硬性允许列表，设置后仅允许列表中的用户 ID 访问。如需基于角色控制访问，请勿设置 `allowFrom`，改用 `allowedRoles`：

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### 禁用 @提及 要求

默认情况下 `requireMention` 为 `true`，机器人只响应 @提及 它的消息。如需响应所有消息：

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## 故障排除

首先运行诊断命令：

```bash
openclaw doctor
openclaw channels status --probe
```

### 机器人不响应消息

**检查访问控制：** 确保您的用户 ID 在 `allowFrom` 列表中，或临时移除 `allowFrom` 并设置 `allowedRoles: ["all"]` 进行测试。

**确认机器人已加入频道：** 机器人必须已加入 `channel` 配置中指定的频道。

### 令牌问题

**"连接失败"或身份验证错误：**

- 验证 `accessToken` 是 OAuth 访问令牌值（通常以 `oauth:` 前缀开头）
- 检查令牌是否具有 `chat:read` 和 `chat:write` 权限范围
- 如使用令牌刷新功能，确认 `clientSecret` 和 `refreshToken` 已正确配置

### 令牌刷新不工作

**检查日志中的刷新事件：**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

如看到"token refresh disabled (no refresh token)"：

- 确保已提供 `clientSecret`
- 确保已提供 `refreshToken`

## 配置参考

**账户配置字段：**

- `username` - 机器人用户名
- `accessToken` - 具有 `chat:read` 和 `chat:write` 权限的 OAuth 访问令牌
- `clientId` - Twitch 客户端 ID（来自令牌生成器或您的应用）
- `channel` - 要加入的频道（必需）
- `enabled` - 启用此账户（默认：`true`）
- `clientSecret` - 可选：用于自动令牌刷新
- `refreshToken` - 可选：用于自动令牌刷新
- `expiresIn` - 令牌过期时间（秒）
- `obtainmentTimestamp` - 令牌获取时间戳
- `allowFrom` - 用户 ID 允许列表
- `allowedRoles` - 基于角色的访问控制（`"moderator" | "owner" | "vip" | "subscriber" | "all"`）
- `requireMention` - 是否需要 @提及（默认：`true`）

**频道配置选项：**

- `channels.twitch.enabled` - 启用/禁用频道启动
- `channels.twitch.username` - 机器人用户名（简化单账户配置）
- `channels.twitch.accessToken` - OAuth 访问令牌（简化单账户配置）
- `channels.twitch.clientId` - Twitch 客户端 ID（简化单账户配置）
- `channels.twitch.channel` - 要加入的频道（简化单账户配置）
- `channels.twitch.accounts.` - 多账户配置（包含上述所有账户字段）

完整配置示例：

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## 工具操作

智能体（agent）可调用 `twitch` 工具执行以下操作：

- `send` - 向频道发送消息

示例：

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## 安全与运维建议

- **像保护密码一样保护令牌** - 切勿将令牌提交到 git 仓库
- **长期运行的机器人请启用自动令牌刷新**
- **访问控制使用用户 ID 允许列表**，而非用户名
- **监控日志** 关注令牌刷新事件和连接状态
- **最小权限原则** - 仅请求 `chat:read` 和 `chat:write` 权限
- **如遇问题**：确认无其他进程占用会话后重启网关

## 限制说明

- **每条消息最多 500 字符**（超长消息在单词边界处自动分块）
- 分块前会自动去除 Markdown 格式
- 无额外速率限制（使用 Twitch 内置的速率限制机制）

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)