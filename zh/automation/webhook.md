

  自动化

  
# Webhooks

网关可以对外暴露一个 HTTP Webhook 端点（endpoint），让你从外部触发智能体（agent）执行或唤醒现有会话。

## 启用 Webhook

在网关配置中添加 `hooks` 设置即可启用：

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // 可选：将显式的 `agentId` 路由限制在此允许列表中
    // 省略或包含 "*" 表示允许任何智能体
    // 设置为 [] 表示拒绝所有显式的 `agentId` 路由
    allowedAgentIds: ["hooks", "main"],
  },
}
```

注意事项：

-   当 `hooks.enabled=true` 时，`hooks.token` 是必需的
-   `hooks.path` 默认为 `/hooks`

## 认证方式

每个请求都必须携带 Webhook 令牌。推荐通过请求头传递：

-   `Authorization: Bearer `（推荐）
-   `x-openclaw-token: `
-   查询字符串方式会被拒绝（`?token=...` 返回 `400`）

## 端点说明

### POST /hooks/wake

用于唤醒主会话，让它处理新事件。

请求体：

```json
{ "text": "System line", "mode": "now" }
```

字段说明：

-   `text` **必需**（字符串）：事件描述，例如"收到新邮件"
-   `mode` 可选（`now` | `next-heartbeat`）：`now` 表示立即触发心跳检查（默认），`next-heartbeat` 表示等待下一次定期检查

执行效果：

-   向**主会话**入队一个系统事件
-   如果 `mode=now`，会立即触发一次心跳检查

### POST /hooks/agent

让智能体（agent）执行一个独立的任务回合，适合处理特定事件。

请求体：

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

字段说明：

-   `message` **必需**（字符串）：让智能体处理的提示词或消息内容
-   `name` 可选（字符串）：Webhook 的可读名称（如"GitHub"），会作为会话摘要的前缀
-   `agentId` 可选（字符串）：将此 Webhook 路由到指定的智能体。未知的 ID 会回退到默认智能体。设置后，Webhook 会使用该智能体的工作空间和配置运行
-   `sessionKey` 可选（字符串）：用于标识智能体会话的密钥。默认情况下此字段会被拒绝，除非设置 `hooks.allowRequestSessionKey=true`
-   `wakeMode` 可选（`now` | `next-heartbeat`）：`now` 表示立即触发心跳检查（默认），`next-heartbeat` 表示等待下一次定期检查
-   `deliver` 可选（布尔值）：为 `true` 时，智能体的响应会发送到消息通道。默认为 `true`。仅作为心跳确认的响应会自动跳过
-   `channel` 可选（字符串）：消息投递通道，可选值：`last`、`whatsapp`、`telegram`、`discord`、`slack`、`mattermost`（插件）、`signal`、`imessage`、`msteams`。默认为 `last`
-   `to` 可选（字符串）：通道接收者标识符（如 WhatsApp/Signal 的电话号码、Telegram 的聊天 ID、Discord/Slack/Mattermost（插件）的频道 ID、MS Teams 的对话 ID）。默认为主会话中的最后一位接收者
-   `model` 可选（字符串）：模型覆盖（如 `anthropic/claude-3-5-sonnet` 或别名）。如果配置了模型限制，必须使用允许列表中的模型
-   `thinking` 可选（字符串）：思考级别覆盖（如 `low`、`medium`、`high`）
-   `timeoutSeconds` 可选（数字）：智能体运行的最大时长（秒）

执行效果：

-   运行一个**独立**的智能体回合（拥有独立的会话密钥）
-   始终会在**主会话**中记录摘要
-   如果 `wakeMode=now`，会立即触发一次心跳检查

## 会话密钥策略（重要变更）

默认情况下，`/hooks/agent` 请求体中的 `sessionKey` 覆盖是禁用的。

配置建议：

-   **推荐做法**：设置固定的 `hooks.defaultSessionKey`，关闭请求覆盖
-   **可选做法**：仅在确实需要时开启请求覆盖，并限制允许的前缀

推荐配置：

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

兼容性配置（旧行为）：

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // 强烈推荐
  },
}
```

### POST /hooks/&lt;name&gt;（自定义映射）

通过 `hooks.mappings` 配置，你可以创建自定义命名的 Webhook 端点。映射可以将任意请求体转换为 `wake` 或 `agent` 操作，支持模板或代码转换。

映射选项摘要：

-   `hooks.presets: ["gmail"]` 启用内置的 Gmail 映射
-   `hooks.mappings` 允许你在配置中定义 `match`、`action` 和模板
-   `hooks.transformsDir` + `transform.module` 加载 JS/TS 模块实现自定义逻辑
    -   `hooks.transformsDir`（如果设置）必须位于 OpenClaw 配置目录下的 transforms 根目录内（通常是 `~/.openclaw/hooks/transforms`）
    -   `transform.module` 必须在有效的 transforms 目录内解析（路径遍历/逃逸会被拒绝）
-   使用 `match.source` 可以保持一个通用的接收端点（基于请求体的路由）
-   TS 转换需要 TS 加载器（如 `bun` 或 `tsx`），或运行时预编译的 `.js` 文件
-   在映射中设置 `deliver: true` + `channel`/`to` 可将回复路由到聊天界面（`channel` 默认为 `last`，回退到 WhatsApp）
-   `agentId` 将 Webhook 路由到指定智能体；未知的 ID 回退到默认智能体
-   `hooks.allowedAgentIds` 限制显式的 `agentId` 路由。省略或包含 `*` 表示允许任何智能体。设置为 `[]` 表示拒绝显式的 `agentId` 路由
-   `hooks.defaultSessionKey` 设置当没有提供显式密钥时，Webhook 智能体运行的默认会话
-   `hooks.allowRequestSessionKey` 控制 `/hooks/agent` 请求体是否可以设置 `sessionKey`（默认：`false`）
-   `hooks.allowedSessionKeyPrefixes` 可选地限制来自请求体和映射的显式 `sessionKey` 值
-   `allowUnsafeExternalContent: true` 为该 Webhook 禁用外部内容安全包装（危险；仅适用于受信任的内部来源）
-   `openclaw webhooks gmail setup` 会为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。完整的 Gmail 监听流程请参见 [Gmail Pub/Sub](./gmail-pubsub.md)

## 响应状态码

-   `200`：`/hooks/wake` 成功
-   `200`：`/hooks/agent` 异步运行已接受
-   `401`：认证失败
-   `429`：同一客户端多次认证失败后被限速（查看 `Retry-After` 响应头）
-   `400`：请求体无效
-   `413`：请求体过大

## 使用示例

唤醒主会话：

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

让智能体执行任务：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### 指定不同的模型

在智能体请求体（或映射）中添加 `model` 字段，可以覆盖该次运行使用的模型：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

如果你配置了 `agents.defaults.models` 限制，请确保覆盖的模型在允许列表中。

Gmail Webhook 示例：

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## 安全建议

-   将 Webhook 端点（endpoint）部署在本地回环、Tailnet 或受信任的反向代理之后
-   使用专用的 Webhook 令牌，不要复用网关认证令牌
-   同一客户端地址的重复认证失败会被限速，以减缓暴力破解攻击
-   如果使用多智能体路由，请设置 `hooks.allowedAgentIds` 限制显式的 `agentId` 选择
-   除非确实需要调用方指定会话，否则保持 `hooks.allowRequestSessionKey=false`
-   如果启用了请求 `sessionKey`，请限制 `hooks.allowedSessionKeyPrefixes`（如 `["hook:"]`）
-   避免在 Webhook 日志中记录敏感的原始请求体
-   Webhook 请求体默认被视为不可信数据，会用安全边界包装。如果必须为特定 Webhook 禁用此保护，请在该 Webhook 的映射中设置 `allowUnsafeExternalContent: true`（危险操作）

[自动化故障排除](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)