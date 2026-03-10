

  消息平台

  
# 飞书

飞书（Lark）是企业广泛使用的团队协作与即时通讯平台。本插件通过飞书的 WebSocket 长连接事件订阅，将 OpenClaw 接入飞书/Lark 机器人——无需暴露公网 Webhook 地址，即可安全接收消息。

* * *

## 安装插件

使用以下命令安装飞书插件：

```bash
openclaw plugins install @openclaw/feishu
```

如果您是从 Git 仓库本地运行，可以这样安装：

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## 快速开始

添加飞书频道（channel）有两种方式：

### 方法一：使用引导向导（推荐）

如果是首次安装 OpenClaw，直接运行向导：

```bash
openclaw onboard
```

向导会一步步引导您：

1. 创建飞书应用并获取凭证
2. 在 OpenClaw 中配置应用凭证
3. 启动网关

✅ **配置完成后**，可以通过以下命令检查网关状态：

- `openclaw gateway status`
- `openclaw logs --follow`

### 方法二：通过命令行配置

如果已完成初始安装，可以通过 CLI 添加频道（channel）：

```bash
openclaw channels add
```

选择 **Feishu**，然后输入应用 ID（app ID）和 App Secret。✅ **配置完成后**，可以使用以下命令管理网关：

- `openclaw gateway status`
- `openclaw gateway restart`
- `openclaw logs --follow`

* * *

## 步骤 1：创建飞书应用

### 1. 打开飞书开放平台

访问 [飞书开放平台](https://open.feishu.cn/app) 并登录。如果您使用的是 Lark（国际版），请访问 [https://open.larksuite.com/app](https://open.larksuite.com/app)，并在配置中设置 `domain: "lark"`。

### 2. 创建应用

1. 点击 **创建企业自建应用**
2. 填写应用名称和描述
3. 选择应用图标

![创建企业自建应用](../images/channels-feishu-step2-create-app.png.md)

### 3. 复制凭证

在 **凭证与基础信息** 页面，复制以下信息：

- **App ID**（格式：`cli_xxx`）
- **App Secret**

❗ **重要提示：** 请妥善保管 App Secret，切勿泄露。![获取凭证](../images/channels-feishu-step3-credentials.png.md)

### 4. 配置权限

进入 **权限管理** 页面，点击 **批量导入**，粘贴以下内容：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![配置权限](../images/channels-feishu-step4-permissions.png.md)

### 5. 启用机器人能力

在 **应用功能** > **机器人** 页面：

1. 启用机器人能力
2. 设置机器人名称

![启用机器人能力](../images/channels-feishu-step5-bot-capability.png.md)

### 6. 配置事件订阅

⚠️ **重要提示：** 在配置事件订阅之前，请确保：

1. 已运行 `openclaw channels add` 添加飞书频道（channel）
2. 网关正在运行（可通过 `openclaw gateway status` 检查）

在 **事件订阅** 页面：

1. 选择 **使用长连接接收事件**（WebSocket）
2. 添加事件：`im.message.receive_v1`

⚠️ 如果网关未运行，长连接设置可能无法保存成功。![配置事件订阅](../images/channels-feishu-step6-event-subscription.png.md)

### 7. 发布应用

1. 在 **版本管理与发布** 中创建版本
2. 提交审核并发布
3. 等待管理员审批（企业自建应用通常会自动通过）

* * *

## 步骤 2：配置 OpenClaw

### 使用向导配置（推荐）

```bash
openclaw channels add
```

选择 **Feishu**，然后粘贴您的应用 ID（app ID）和 App Secret。

### 通过配置文件配置

编辑 `~/.openclaw/openclaw.json`：

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

如果您使用 `connectionMode: "webhook"` 模式，需要设置 `verificationToken`。飞书 Webhook 服务器默认绑定到 `127.0.0.1`，除非确实需要修改绑定地址，否则无需设置 `webhookHost`。

#### 校验令牌（Webhook 模式）

使用 Webhook 模式时，需要在配置中设置 `channels.feishu.verificationToken`。获取方法：

1. 在飞书开放平台打开您的应用
2. 进入 **开发配置** → **事件与回调**
3. 打开 **加密策略** 标签页
4. 复制 **校验令牌**

![校验令牌位置](../images/channels-feishu-verification-token.png.md)

### 通过环境变量配置

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark（国际版）域名配置

如果您的租户使用 Lark（国际版），需要将域名设置为 `lark`（或完整的域名字符串）。可以在 `channels.feishu.domain` 全局设置，或针对单个账户设置 `channels.feishu.accounts..domain`。

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### 配额优化选项

您可以通过以下两个可选配置减少飞书 API 调用：

- `typingIndicator`（默认 `true`）：设为 `false` 时，跳过"正在输入"状态更新调用
- `resolveSenderNames`（默认 `true`）：设为 `false` 时，跳过发送者信息查询调用

可以在全局或单个账户级别设置：

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## 步骤 3：启动与测试

### 1. 启动网关

```bash
openclaw gateway
```

### 2. 发送测试消息

在飞书中找到您的机器人，发送一条消息。

### 3. 批准配对

默认情况下，机器人会返回一个配对码。运行以下命令批准：

```bash
openclaw pairing approve feishu <CODE>
```

批准后即可正常对话。

* * *

## 功能概览

- **飞书机器人频道（channel）**：由网关管理的飞书机器人
- **确定性路由**：回复始终返回飞书
- **会话隔离**：私聊共享主会话，群聊独立隔离
- **WebSocket 长连接**：通过飞书 SDK 建立，无需公网地址

* * *

## 访问控制

### 私聊消息

- **默认策略**：`dmPolicy: "pairing"`（未知用户会收到配对码）
- **批准配对**：

    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```

- **白名单模式**：设置 `channels.feishu.allowFrom`，填入允许的 Open ID

### 群聊消息

**1. 群聊策略**（`channels.feishu.groupPolicy`）：

- `"open"` = 允许所有群聊（默认）
- `"allowlist"` = 仅允许 `groupAllowFrom` 中的群
- `"disabled"` = 禁用群消息

**2. @提及要求**（`channels.feishu.groups.<chat_id>.requireMention`）：

- `true` = 需要 @提及机器人（默认）
- `false` = 无需 @提及即可响应

* * *

## 群聊配置示例

### 允许所有群聊，需要 @提及（默认行为）

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // 默认 requireMention: true
    },
  },
}
```

### 允许所有群聊，无需 @提及

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### 仅允许特定群聊

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // 飞书群 ID（chat_id）格式：oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### 限制群聊中可发言的用户（发送者白名单）

除了限制群聊本身，还可以进一步限制**群内的所有消息**：只有 `groups.<chat_id>.allowFrom` 中列出的用户发送的消息才会被处理，其他成员的消息将被忽略。这是完整的发送者级别限制，不仅限于 `/reset`、`/new` 等控制命令。

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // 飞书用户 ID（open_id）格式：ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## 获取群聊/用户 ID

### 群聊 ID（chat_id）

群聊 ID 格式为 `oc_xxx`。**方法一（推荐）**

1. 启动网关，在群聊中 @提及机器人
2. 运行 `openclaw logs --follow`，在日志中查找 `chat_id`

**方法二** 使用飞书 API 调试工具列出群聊。

### 用户 ID（open_id）

用户 ID 格式为 `ou_xxx`。**方法一（推荐）**

1. 启动网关，向机器人发送私聊消息
2. 运行 `openclaw logs --follow`，在日志中查找 `open_id`

**方法二** 查看配对请求中的用户 Open ID：

```bash
openclaw pairing list feishu
```

* * *

## 常用命令

| 命令 | 说明 |
| --- | --- |
| `/status` | 显示机器人状态 |
| `/reset` | 重置会话 |
| `/model` | 显示/切换模型 |

> 注意：飞书暂不支持原生命令菜单，命令需以文本形式发送。

## 网关管理命令

| 命令 | 说明 |
| --- | --- |
| `openclaw gateway status` | 查看网关状态 |
| `openclaw gateway install` | 安装/启动网关服务 |
| `openclaw gateway stop` | 停止网关服务 |
| `openclaw gateway restart` | 重启网关服务 |
| `openclaw logs --follow` | 实时查看网关日志 |

* * *

## 故障排除

### 机器人在群聊中不响应

1. 确认机器人已加入群聊
2. 确认已 @提及机器人（默认行为）
3. 检查 `groupPolicy` 是否被设置为 `"disabled"`
4. 查看日志：`openclaw logs --follow`

### 机器人收不到消息

1. 确认应用已发布并通过审批
2. 确认事件订阅中包含 `im.message.receive_v1`
3. 确认 **长连接** 已启用
4. 确认应用权限配置完整
5. 确认网关正在运行：`openclaw gateway status`
6. 查看日志：`openclaw logs --follow`

### App Secret 泄露

1. 在飞书开放平台重置 App Secret
2. 在配置中更新 App Secret
3. 重启网关

### 消息发送失败

1. 确认应用拥有 `im:message:send_as_bot` 权限
2. 确认应用已发布
3. 查看日志获取详细错误信息

* * *

## 高级配置

### 多账户配置

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` 用于指定当出站 API 未明确指定 `accountId` 时使用的默认账户。

### 消息限制

- `textChunkLimit`：出站文本分块大小（默认：2000 字符）
- `mediaMaxMb`：媒体文件上传/下载限制（默认：30MB）

### 流式输出

飞书支持通过交互式卡片实现流式回复。启用后，机器人会在生成文本过程中实时更新卡片内容。

```json
{
  channels: {
    feishu: {
      streaming: true, // 启用流式卡片输出（默认 true）
      blockStreaming: true, // 启用块级流式输出（默认 true）
    },
  },
}
```

如需等待完整回复后再发送，可设置 `streaming: false`。

### 多智能体（agent）路由

使用 `bindings` 可以将飞书私聊或群聊路由到不同的智能体（agent）。

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

路由配置字段说明：

- `match.channel`: `"feishu"`
- `match.peer.kind`: `"direct"`（私聊）或 `"group"`（群聊）
- `match.peer.id`: 用户 Open ID（`ou_xxx`）或群聊 ID（`oc_xxx`）

查看 [获取群聊/用户 ID](#获取群聊用户-id) 了解如何查找这些 ID。

* * *

## 配置参数参考

完整配置请参阅 [网关配置](../gateway/configuration.md)。以下是常用配置项：

| 设置 | 说明 | 默认值 |
| --- | --- | --- |
| `channels.feishu.enabled` | 启用/禁用频道（channel） | `true` |
| `channels.feishu.domain` | API 域名（`feishu` 或 `lark`） | `feishu` |
| `channels.feishu.connectionMode` | 事件传输模式 | `websocket` |
| `channels.feishu.defaultAccount` | 出站路由默认账户 ID | `default` |
| `channels.feishu.verificationToken` | Webhook 模式必需 | - |
| `channels.feishu.webhookPath` | Webhook 路由路径 | `/feishu/events` |
| `channels.feishu.webhookHost` | Webhook 绑定地址 | `127.0.0.1` |
| `channels.feishu.webhookPort` | Webhook 绑定端口 | `3000` |
| `channels.feishu.accounts..appId` | 应用 ID（app ID） | - |
| `channels.feishu.accounts..appSecret` | App Secret | - |
| `channels.feishu.accounts..domain` | 单账户 API 域名覆盖 | `feishu` |
| `channels.feishu.dmPolicy` | 私聊策略 | `pairing` |
| `channels.feishu.allowFrom` | 私聊白名单（open_id 列表） | - |
| `channels.feishu.groupPolicy` | 群聊策略 | `open` |
| `channels.feishu.groupAllowFrom` | 群聊白名单 | - |
| `channels.feishu.groups.<chat_id>.requireMention` | 是否需要 @提及 | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | 是否启用该群聊 | `true` |
| `channels.feishu.textChunkLimit` | 消息分块大小 | `2000` |
| `channels.feishu.mediaMaxMb` | 媒体文件大小限制 | `30` |
| `channels.feishu.streaming` | 启用流式卡片输出 | `true` |
| `channels.feishu.blockStreaming` | 启用块级流式输出 | `true` |

* * *

## dmPolicy 策略说明

| 值 | 行为说明 |
| --- | --- |
| `"pairing"` | **默认值。** 未知用户会收到配对码，需管理员批准后才能对话 |
| `"allowlist"` | 仅 `allowFrom` 中的用户可以对话 |
| `"open"` | 允许所有用户（需在 allowFrom 中包含 `"*"`） |
| `"disabled"` | 禁用私聊功能 |

* * *

## 支持的消息类型

### 接收

- ✅ 文本
- ✅ 富文本（帖子）
- ✅ 图片
- ✅ 文件
- ✅ 音频
- ✅ 视频
- ✅ 表情

### 发送

- ✅ 文本
- ✅ 图片
- ✅ 文件
- ✅ 音频
- ⚠️ 富文本（部分支持）

[Discord](./discord.md)[Google Chat](./googlechat.md)