

  远程访问

  
# 远程访问

OpenClaw 支持在专用主机（桌面/服务器）上运行单一网关（主节点），让客户端远程连接到它。

-   对于**操作员（你 / macOS 应用）**：SSH 隧道是通用的后备方案。
-   对于**节点（iOS/Android 及未来设备）**：连接到网关的 **WebSocket**（根据需要通过局域网/tailnet 或 SSH 隧道）。

## 核心理念

-   网关 WebSocket 默认绑定到你配置端口的**本地回环地址**（默认 18789）。
-   远程使用时，通过 SSH 转发该本地端口（或使用 tailnet/VPN 减少隧道需求）。

## 常见 VPN/tailnet 部署模式（智能体所在位置）

把**网关主机**理解为"智能体所在的地方"。它持有会话、认证配置、频道和状态。你的笔记本/台式机（以及节点）连接到这台主机。

### 1) Tailnet 内的始终在线网关（VPS 或家庭服务器）

在持久运行的主机上部署网关，通过 **Tailscale** 或 SSH 访问。

-   **最佳体验**：保持 `gateway.bind: "loopback"`，使用 **Tailscale Serve** 提供控制界面。
-   **后备方案**：保持本地回环，从需要访问的机器建立 SSH 隧道。
-   **示例**：[exe.dev](../install/exe-dev.md)（简易 VM）或 [Hetzner](../install/hetzner.md)（生产级 VPS）。

当你的笔记本经常休眠但又希望智能体始终在线时，这是理想方案。

### 2) 家里的台式机运行网关，笔记本远程控制

笔记本**不运行**智能体，而是远程连接：

-   使用 macOS 应用的**通过 SSH 远程**模式（设置 → 通用 → "OpenClaw 运行位置"）。
-   应用会自动建立和管理隧道，让 WebChat 和健康检查"开箱即用"。

操作手册：[macOS 远程访问](../platforms/mac/remote.md)。

### 3) 笔记本运行网关，其他机器远程访问

保持网关在本地运行，但安全地暴露出来：

-   从其他机器通过 SSH 隧道连接笔记本，或
-   使用 Tailscale Serve 提供控制界面，网关保持仅限本地回环。

指南：[Tailscale](./tailscale.md) 和 [Web 概览](../web.md)。

## 命令流程（各组件在哪里运行）

一个网关服务持有状态和频道。节点是外围设备。流程示例（Telegram → 节点）：

-   Telegram 消息到达**网关**。
-   网关运行**智能体**，决定是否调用节点工具。
-   网关通过网关 WebSocket（`node.*` RPC）调用**节点**。
-   节点返回结果，网关将回复发回 Telegram。

注意：

-   **节点不运行网关服务。** 除非你故意运行隔离的 profile（见[多网关](./multiple-gateways.md)），否则每台主机只应运行一个网关。
-   macOS 应用的"节点模式"只是通过网关 WebSocket 连接的节点客户端。

## SSH 隧道（CLI + 工具）

建立到远程网关 WebSocket 的本地隧道：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

隧道建立后：

-   `openclaw health` 和 `openclaw status --deep` 可以通过 `ws://127.0.0.1:18789` 访问远程网关。
-   `openclaw gateway {status,health,send,agent,call}` 在需要时也可以通过 `--url` 指向转发的地址。

注意：将 `18789` 替换为你配置的 `gateway.port`（或 `--port`/`OPENCLAW_GATEWAY_PORT`）。注意：传入 `--url` 时，CLI 不会回退到配置或环境变量中的凭证。请显式包含 `--token` 或 `--password`，缺少显式凭证会报错。

## CLI 远程默认配置

你可以持久化远程目标，让 CLI 命令默认使用它：

```json
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

当网关仅限本地回环时，URL 保持为 `ws://127.0.0.1:18789`，并确保先建立 SSH 隧道。

## 凭证优先级

网关调用/探测的凭证解析遵循统一的优先级规则：

-   显式凭证（`--token`、`--password` 或工具的 `gatewayToken`）始终最高优先。
-   本地模式默认值：
    -   token: `OPENCLAW_GATEWAY_TOKEN` → `gateway.auth.token` → `gateway.remote.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` → `gateway.auth.password` → `gateway.remote.password`
-   远程模式默认值：
    -   token: `gateway.remote.token` → `OPENCLAW_GATEWAY_TOKEN` → `gateway.auth.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` → `gateway.remote.password` → `gateway.auth.password`
-   远程探测/状态的令牌检查默认是严格的：针对远程模式时只使用 `gateway.remote.token`（无本地令牌回退）。
-   遗留的 `CLAWDBOT_GATEWAY_*` 环境变量仅用于兼容性调用路径；探测/状态/认证解析只使用 `OPENCLAW_GATEWAY_*`。

## 通过 SSH 的聊天界面

WebChat 不再使用独立的 HTTP 端口。SwiftUI 聊天界面直接连接网关 WebSocket。

-   通过 SSH 转发 `18789`（见上文），然后将客户端连接到 `ws://127.0.0.1:18789`。
-   在 macOS 上，推荐使用应用的"通过 SSH 远程"模式，它会自动管理隧道。

## macOS 应用"通过 SSH 远程"

macOS 菜单栏应用可以端到端驱动相同配置（远程状态检查、WebChat 和语音唤醒转发）。操作手册：[macOS 远程访问](../platforms/mac/remote.md)。

## 安全规则（远程/VPN）

简短版：**保持网关仅限本地回环**，除非你确定需要绑定到其他地址。

-   **本地回环 + SSH/Tailscale Serve** 是最安全的默认配置（无公网暴露）。
-   明文 `ws://` 默认仅限本地回环。对于受信任的私有网络，可在客户端进程设置 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` 作为应急方案。
-   **非本地回环绑定**（`lan`/`tailnet`/`custom`，或本地回环不可用时的 `auto`）必须使用认证令牌/密码。
-   `gateway.remote.token` / `.password` 是客户端凭证来源，它们**本身不配置**服务端认证。
-   当 `gateway.auth.*` 未设置时，本地调用路径可以使用 `gateway.remote.*` 作为回退。
-   使用 `wss://` 时，`gateway.remote.tlsFingerprint` 用于固定远程 TLS 证书。
-   当 `gateway.auth.allowTailscale: true` 时，**Tailscale Serve** 可以通过身份头认证控制界面/WebSocket 流量；HTTP API 端点仍需令牌/密码认证。这种无令牌流程假设网关主机是受信任的。若希望在所有地方都使用令牌/密码，设为 `false`。
-   将浏览器控制视为操作员访问：仅限 tailnet + 有意识的节点配对。

深入讲解：[安全](./security.md)。

[Bonjour 发现](./bonjour.md)[远程网关设置](./remote-gateway-readme.md)