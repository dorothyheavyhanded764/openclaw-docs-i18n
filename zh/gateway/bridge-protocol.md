

  协议与 API

  
# Bridge 协议

Bridge 协议是**已弃用**的节点传输方式（TCP JSONL）。新的节点客户端应改用统一的 Gateway WebSocket 协议。如果你正在构建操作端或节点客户端，请使用 [Gateway 协议](./protocol.md)。**注意：** 当前版本的 OpenClaw 已不再内置 TCP bridge 监听器；本文档仅作历史参考。旧版 `bridge.*` 配置键已从配置 schema 中移除。

## 为何曾经同时存在两种协议

-   **安全边界**：bridge 只暴露有限的允许列表，而非完整的网关 API 接口。
-   **配对与节点身份**：节点准入由网关控制，并与每个节点的令牌绑定。
-   **发现体验**：节点可通过 Bonjour 在局域网发现网关，或通过 tailnet 直连。
-   **本地回环 WS**：完整的 WebSocket 控制平面保持本地化，除非通过 SSH 隧道转发。

## 传输层

-   TCP 协议，每行一个 JSON 对象（JSONL 格式）。
-   可选 TLS（当 `bridge.tls.enabled` 为 true 时）。
-   旧版默认监听端口为 `18790`（当前版本不再启动 TCP bridge）。

启用 TLS 时，发现 TXT 记录会包含 `bridgeTls=1` 以及 `bridgeTlsSha256` 作为非敏感提示。注意 Bonjour/mDNS TXT 记录本身未经验证；客户端不应将广播的指纹视为权威固定值，除非用户明确确认或通过其他带外方式验证。

## 握手与配对流程

1.  客户端发送 `hello`，携带节点元数据和令牌（如已配对）。
2.  如未配对，网关回复 `error`（`NOT_PAIRED` 或 `UNAUTHORIZED`）。
3.  客户端发送 `pair-request`。
4.  网关等待批准后，发送 `pair-ok` 和 `hello-ok`。

`hello-ok` 返回 `serverName`，可能包含 `canvasHostUrl`。

## 帧类型

客户端 → 网关：

-   `req` / `res`：限定范围的网关 RPC（聊天、会话、配置、健康检查、voicewake、skills.bins）
-   `event`：节点信号（语音转录、智能体请求、聊天订阅、执行生命周期）

网关 → 客户端：

-   `invoke` / `invoke-res`：节点命令（`canvas.*`、`camera.*`、`screen.record`、`location.get`、`sms.send`）
-   `event`：已订阅会话的聊天更新
-   `ping` / `pong`：心跳保活

旧版允许列表的强制逻辑曾位于 `src/gateway/server-bridge.ts`（已移除）。

## 执行生命周期事件

节点可以发送 `exec.finished` 或 `exec.denied` 事件来报告 system.run 活动。这些事件在网关中映射为系统事件。（旧版节点可能仍发送 `exec.started`。）负载字段（除注明外均为可选）：

-   `sessionKey`（必填）：接收系统事件的智能体会话。
-   `runId`：用于分组的唯一执行 ID。
-   `command`：原始或格式化的命令字符串。
-   `exitCode`、`timedOut`、`success`、`output`：完成详情（仅 finished 事件）。
-   `reason`：拒绝原因（仅 denied 事件）。

## Tailnet 使用

-   将 bridge 绑定到 tailnet IP：在 `~/.openclaw/openclaw.json` 中设置 `bridge.bind: "tailnet"`。
-   客户端通过 MagicDNS 名称或 tailnet IP 连接。
-   Bonjour **无法**跨网络工作；需要时请使用手动主机/端口或广域 DNS-SD。

## 版本控制

Bridge 目前为**隐式 v1**（无最小/最大版本协商）。预期保持向后兼容；任何破坏性变更前应先添加协议版本字段。

[Gateway 协议](./protocol.md)[OpenAI 聊天补全](./openai-http-api.md)