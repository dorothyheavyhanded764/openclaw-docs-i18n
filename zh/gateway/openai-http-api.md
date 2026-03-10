

  协议与 API

  
# OpenAI 聊天补全

OpenClaw 网关可以提供一个轻量级的 OpenAI 兼容聊天补全端点。这个端点**默认是关闭的**，需要先在配置中手动启用。

-   `POST /v1/chat/completions`
-   使用与网关相同的端口（WebSocket + HTTP 复用）：`http://<gateway-host>:/v1/chat/completions`

底层实现上，每个请求都会作为一个标准的网关智能体运行来执行（与 `openclaw agent` 走相同的代码路径），因此路由规则、权限设置和配置项都会与你配置的网关保持一致。

## 认证方式

使用网关的认证配置。请求时需要携带 Bearer 令牌：

-   `Authorization: Bearer `

注意事项：

-   当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或环境变量 `OPENCLAW_GATEWAY_TOKEN`）。
-   当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或环境变量 `OPENCLAW_GATEWAY_PASSWORD`）。
-   如果配置了 `gateway.auth.rateLimit` 且认证失败次数过多，端点会返回 `429` 状态码并附带 `Retry-After` 响应头。

## 安全边界（重要！）

请务必将此端点视为网关实例的**完全操作员权限入口**。

-   这里的 HTTP Bearer 认证不是细粒度的用户级权限模型。
-   能通过此端点验证的有效网关令牌/密码，等同于拥有者/操作员凭证，应当妥善保管。
-   请求会经过与受信任操作员操作相同的控制平面智能体路径执行。
-   此端点没有独立的非管理员或用户级工具边界——一旦调用者通过了网关认证，OpenClaw 就会将其视为该网关的受信任操作员。
-   如果目标智能体的策略允许使用敏感工具，此端点就能调用它们。
-   请仅将此端点暴露在本地回环/Tailnet/私有网络入口上，切勿直接暴露到公网。

详见[安全](./security.md)和[远程访问](./remote.md)。

## 选择智能体

无需自定义请求头，只需在 OpenAI 的 `model` 字段中编码智能体 ID：

-   `model: "openclaw:"`（例如：`"openclaw:main"`、`"openclaw:beta"`）
-   `model: "agent:"`（别名形式）

也可以通过请求头指定目标智能体：

-   `x-openclaw-agent-id: `（默认值：`main`）

高级用法：

-   `x-openclaw-session-key: ` 可以完全控制会话路由。

## 启用端点

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `true`：

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## 禁用端点

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `false`：

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## 会话行为

默认情况下，此端点对每个请求都是**无状态的**（每次调用都会生成新的会话密钥）。如果请求中包含 OpenAI 的 `user` 字符串，网关会从中派生一个稳定的会话密钥，这样重复调用就能共享同一个智能体会话。

## 流式传输（SSE）

设置 `stream: true` 即可接收服务器发送事件（SSE）响应：

-   `Content-Type: text/event-stream`
-   每个事件行的格式为 `data: `
-   流结束时发送 `data: [DONE]`

## 示例

非流式请求：

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

流式请求：

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[桥接协议](./bridge-protocol.md)[OpenResponses API](./openresponses-http-api.md)