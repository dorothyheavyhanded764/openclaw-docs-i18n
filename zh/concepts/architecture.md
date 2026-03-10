

  基础

  
# 网关架构

最后更新：2026-01-22

## 概述

网关是整个系统的核心枢纽——它负责管理所有消息通道的连接，并协调客户端与节点之间的通信。理解网关架构，能帮助你更好地部署、调试和扩展 OpenClaw。

- 一个长期运行的**网关**统一管理所有消息通道（通过 Baileys 的 WhatsApp、通过 grammY 的 Telegram、Slack、Discord、Signal、iMessage、WebChat）。
- 控制平面客户端（macOS 应用、CLI、Web 管理界面、自动化程序）通过 **WebSocket** 连接到网关，默认绑定地址为 `127.0.0.1:18789`。
- **节点**（macOS/iOS/Android/无头设备）同样通过 **WebSocket** 连接，但需要声明 `role: node` 并明确声明自身的能力和命令。
- 每台主机只运行一个网关实例，它是唯一与 WhatsApp 建立会话的组件。
- **画布服务**由网关的 HTTP 服务器提供，路径包括：
    - `/__openclaw__/canvas/`（智能体可编辑的 HTML/CSS/JS）
    - `/__openclaw__/a2ui/`（A2UI 宿主页面），使用与网关相同的端口（默认 `18789`）。

## 组件与流程

### 网关（守护进程）

网关守护进程是整个系统的"大脑"，负责以下核心职责：

- 维护与各消息平台的连接
- 暴露类型化的 WebSocket API（支持请求、响应、服务器推送事件）
- 使用 JSON Schema 验证所有入站消息帧
- 发出各类事件，如 `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron`

### 客户端（mac 应用 / CLI / Web 管理界面）

客户端是与网关交互的"操控端"，每个客户端维护一个独立的 WebSocket 连接：

- 发送请求命令（`health`、`status`、`send`、`agent`、`system-presence`）
- 订阅并接收事件（`tick`、`agent`、`presence`、`shutdown`）

### 节点（macOS / iOS / Android / 无头设备）

节点是连接到网关的"设备端"，用于扩展系统能力：

- 连接到**同一个 WebSocket 服务器**，声明 `role: node`
- 在 `connect` 时提供设备身份；配对是**基于设备**的（角色为 `node`），批准信息存储在设备配对记录中
- 暴露设备能力命令，如 `canvas.*`、`camera.*`、`screen.record`、`location.get`

想深入了解通信细节？参见 [网关协议](../gateway/protocol.md)。

### WebChat

WebChat 是一个轻量级的 Web 聊天界面：

- 静态前端页面，通过网关 WebSocket API 获取聊天记录和发送消息
- 在远程部署场景下，通过与其他客户端相同的 SSH/Tailscale 隧道连接

## 连接生命周期（单个客户端）

## 通信协议概要

理解消息格式对于调试和开发集成非常重要：

- **传输层**：WebSocket，使用文本帧携带 JSON 负载
- **首帧要求**：第一条消息**必须**是 `connect`
- **握手后的消息格式**：
    - 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
    - 事件：`{type:"event", event, payload, seq?, stateVersion?}`
- **认证**：如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token` 参数），则 `connect.params.auth.token` 必须匹配，否则连接会被关闭
- **幂等性保证**：具有副作用的命令（`send`、`agent`）需要提供幂等键，以支持安全重试；服务端会维护一个短期去重缓存
- **节点声明**：节点必须在 `connect` 中包含 `role: "node"` 以及 caps/commands/permissions

## 配对与本地信任

OpenClaw 使用设备配对机制来确保连接安全：

- 所有 WebSocket 客户端（操作端 + 节点）在 `connect` 时都必须提供**设备身份**
- 新设备 ID 需要经过配对批准；批准后，网关会颁发**设备令牌**供后续连接使用
- **本地连接**（环回地址或网关主机自身的 Tailscale 地址）可以配置为自动批准，简化同主机操作体验
- 所有连接都必须对 `connect.challenge` 随机数进行签名
- 签名负载 `v3` 版本还会绑定 `platform` + `deviceFamily`；网关在重连时会校验已配对的元数据，元数据变更需要重新配对
- **非本地连接**仍需明确的人工批准
- 网关认证策略（`gateway.auth.*`）对**所有**连接生效，无论是本地还是远程

更多细节请参阅：[网关协议](../gateway/protocol.md)、[配对](../channels/pairing.md)、[安全](../gateway/security.md)。

## 协议类型与代码生成

OpenClaw 使用类型优先的开发方式：

- 使用 TypeBox schema 定义协议结构
- 从 schema 自动生成 JSON Schema
- 从 JSON Schema 自动生成 Swift 模型

## 远程访问

需要从外网访问网关？以下是推荐方案：

- **首选方案**：Tailscale 或其他 VPN
- **备选方案**：SSH 隧道

    复制

    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@host
    ```

- 隧道连接同样适用握手和认证令牌验证
- 远程场景下可以为 WebSocket 启用 TLS + 可选证书固定

## 运维要点

日常运维的几个关键点：

- **启动**：`openclaw gateway`（前台运行，日志输出到标准输出）
- **健康检查**：通过 WebSocket 发送 `health` 请求（也包含在 `hello-ok` 响应中）
- **进程守护**：使用 launchd（macOS）或 systemd（Linux）实现自动重启

## 核心约束

系统设计遵循以下不变原则：

- 每台主机上**恰好一个**网关控制一个 Baileys 会话
- 握手是**强制性的**；任何非 JSON 格式或首帧不是 `connect` 的消息都会导致连接立即关闭
- 事件**不会重放**；客户端检测到消息间隙时必须主动刷新状态

[Pi 集成架构](../pi.md)[智能体运行时](./agent.md)