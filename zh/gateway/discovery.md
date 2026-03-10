

  网络与发现

  
# 发现与传输

OpenClaw 面临两个表面上相似、但本质完全不同的问题：

1.  **操作员远程控制**：macOS 菜单栏应用需要控制运行在别处的网关。
2.  **节点配对**：iOS/Android（以及未来的节点客户端）需要发现网关并完成安全配对。

设计原则是：将所有网络发现和广播功能集中在**节点网关**（`openclaw gateway`）中，让客户端（Mac 应用、iOS 应用）只负责消费这些服务。

## 术语说明

-   **网关（Gateway）**：一个长期运行的网关进程，持有所有状态（会话、配对信息、节点注册表）并运行频道连接。大多数部署在每个主机上运行一个网关；也支持隔离的多网关部署。
-   **网关 WebSocket（控制平面）**：默认监听 `127.0.0.1:18789` 的 WebSocket 端点；可通过 `gateway.bind` 绑定到局域网或 tailnet。
-   **直连 WebSocket 传输**：面向局域网或 tailnet 的网关 WebSocket 端点（无需 SSH）。
-   **SSH 传输（后备方案）**：通过 SSH 隧道转发 `127.0.0.1:18789` 实现远程控制。
-   **传统 TCP 桥接（已弃用/移除）**：旧版节点传输方式（参见[桥接协议](./bridge-protocol.md)）；已不再用于服务发现。

协议详情：

-   [网关协议](./protocol.md)
-   [桥接协议（旧版）](./bridge-protocol.md)

## 为何同时保留直连和 SSH 两种传输方式

-   **直连 WebSocket** 在同一网络和 tailnet 内提供最佳体验：
    -   通过 Bonjour 在局域网内自动发现
    -   配对令牌和访问控制由网关统一管理
    -   无需 shell 访问权限；协议接口精简且易于审计
-   **SSH** 作为通用的后备方案：
    -   只要能 SSH 连接就能工作（即使跨越不同网络）
    -   能应对组播/mDNS 不可用的情况
    -   除了 SSH 本身，无需开放额外的入站端口

## 发现机制（客户端如何找到网关）

### 1) Bonjour / mDNS（仅限局域网）

Bonjour 是尽力而为的服务，无法跨网络工作，仅用于同一局域网内的便捷发现。目标流程：

-   **网关**通过 Bonjour 广播其 WebSocket 端点。
-   客户端浏览并显示"选择网关"列表，然后保存用户选择的端点。

故障排除和广播详情：[Bonjour](./bonjour.md)。

#### 服务广播详情

-   服务类型：
    -   `_openclaw-gw._tcp`（网关传输广播）
-   TXT 记录键（非敏感信息）：
    -   `role=gateway`
    -   `lanHost=.local`
    -   `sshPort=22`（或实际配置的端口）
    -   `gatewayPort=18789`（网关 WebSocket + HTTP）
    -   `gatewayTls=1`（仅当 TLS 启用时）
    -   `gatewayTlsSha256=`（仅当 TLS 启用且指纹可用时）
    -   `canvasPort=`（画布主机端口；启用画布主机时与 `gatewayPort` 相同）
    -   `cliPath=`（可选；可执行的 `openclaw` 入口或二进制文件的绝对路径）
    -   `tailnetDns=`（可选提示；当 Tailscale 可用时自动检测）

安全注意事项：

-   Bonjour/mDNS TXT 记录是**未经认证的**。客户端必须仅将 TXT 值作为用户体验提示，不能信任其内容。
-   路由（主机/端口）应优先使用**解析后的服务端点**（SRV + A/AAAA 记录），而非 TXT 中提供的 `lanHost`、`tailnetDns` 或 `gatewayPort`。
-   TLS 证书固定绝不能允许广播的 `gatewayTlsSha256` 覆盖已存储的固定值。
-   iOS/Android 节点应将基于发现的直连视为**仅限 TLS**，并在存储首次固定值之前要求用户明确确认"信任此指纹"（带外验证）。

禁用/覆盖：

-   `OPENCLAW_DISABLE_BONJOUR=1` 禁用广播。
-   `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制网关绑定模式。
-   `OPENCLAW_SSH_PORT` 覆盖 TXT 中广播的 SSH 端口（默认 22）。
-   `OPENCLAW_TAILNET_DNS` 发布 `tailnetDns` 提示（MagicDNS）。
-   `OPENCLAW_CLI_PATH` 覆盖广播的 CLI 路径。

### 2) Tailnet（跨网络）

对于伦敦/维也纳这类跨地域部署，Bonjour 无法发挥作用。推荐的"直连"目标是：

-   Tailscale MagicDNS 名称（首选）或稳定的 tailnet IP 地址。

如果网关检测到自己在 Tailscale 环境中运行，它会发布 `tailnetDns` 作为给客户端的可选提示（包括广域广播）。

### 3) 手动指定 / SSH 目标

当没有直连路由（或直连被禁用）时，客户端始终可以通过 SSH 隧道转发本地网关端口来连接。参见[远程访问](./remote.md)。

## 传输选择（客户端策略）

推荐的客户端行为顺序：

1.  如果已配置配对的直连端点且可访问，优先使用它。
2.  否则，如果 Bonjour 在局域网内发现网关，提供一键"使用此网关"选项，并将其保存为直连端点。
3.  否则，如果配置了 tailnet DNS/IP，尝试直连。
4.  都不行时，回退到 SSH。

## 配对与认证（直连传输）

网关是节点和客户端准入的唯一权威来源。

-   配对请求在网关中创建、批准或拒绝（参见[网关配对](./pairing.md)）。
-   网关负责强制执行：
    -   认证（令牌/密钥对）
    -   权限范围和访问控制列表（网关不是对所有方法的无差别代理）
    -   速率限制

## 各组件职责分工

-   **网关**：广播发现信号，掌控配对决策，托管 WebSocket 端点。
-   **macOS 应用**：帮助用户选择网关，显示配对提示，仅将 SSH 作为后备方案。
-   **iOS/Android 节点**：将 Bonjour 浏览作为便捷功能，连接到已配对的网关 WebSocket。

[网关配对](./pairing.md)[Bonjour 发现](./bonjour.md)