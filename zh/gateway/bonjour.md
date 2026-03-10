

  网络与发现

  
# Bonjour 发现

OpenClaw 使用 Bonjour（mDNS / DNS-SD）作为一种**仅限局域网的便利功能**来自动发现活跃的网关（WebSocket 端点）。它是尽力而为的服务，**不能**替代 SSH 或 Tailnet 连接方式。

## 通过 Tailscale 的广域网 Bonjour（单播 DNS-SD）

如果节点和网关位于不同网络，组播 mDNS 无法跨越网络边界。你可以切换到通过 Tailscale 的**单播 DNS-SD**（"广域网 Bonjour"）来保持相同的发现体验。大致步骤：

1.  在网关主机上运行 DNS 服务器（可通过 Tailnet 访问）。
2.  在专用区域（如 `openclaw.internal.`）下发布 `_openclaw-gw._tcp` 的 DNS-SD 记录。
3.  配置 Tailscale **拆分 DNS**，让客户端（包括 iOS）通过该 DNS 服务器解析你选择的域名。

OpenClaw 支持任意发现域名；`openclaw.internal.` 只是示例。iOS/Android 节点会同时浏览 `local.` 和你配置的广域网域名。

### 网关配置（推荐）

```json
{
  gateway: { bind: "tailnet" }, // 仅限 tailnet（推荐）
  discovery: { wideArea: { enabled: true } }, // 启用广域网 DNS-SD 发布
}
```

### 一次性 DNS 服务器设置（网关主机）

```bash
openclaw dns setup --apply
```

这会安装 CoreDNS 并配置它：

-   仅在网关的 Tailscale 接口上监听 53 端口
-   从 `~/.openclaw/dns/.db` 为你选择的域名（如 `openclaw.internal.`）提供服务

从一台连接到 tailnet 的机器验证：

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS 设置

在 Tailscale 管理控制台：

-   添加一个指向网关 tailnet IP 的 DNS 服务器（UDP/TCP 53）。
-   添加拆分 DNS 规则，让你的发现域名使用该 DNS 服务器。

一旦客户端接受了 tailnet DNS，iOS 节点就可以在你的发现域名中浏览 `_openclaw-gw._tcp`，无需组播。

### 网关监听器安全（推荐）

网关 WebSocket 端口（默认 `18789`）默认绑定到本地回环地址。如需局域网或 tailnet 访问，请显式配置绑定并保持认证启用。对于仅限 tailnet 的设置：

-   在 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
-   重启网关（或重启 macOS 菜单栏应用）。

## 广播内容

只有网关会广播 `_openclaw-gw._tcp` 服务。

## 服务类型

-   `_openclaw-gw._tcp` — 网关传输信标（由 macOS/iOS/Android 节点使用）。

## TXT 记录键（非敏感提示）

网关广播一些非敏感的提示信息，方便 UI 流程：

-   `role=gateway`
-   `displayName=<友好名称>`
-   `lanHost=<主机名>.local`
-   `gatewayPort=<端口>`（网关 WebSocket + HTTP）
-   `gatewayTls=1`（仅在启用 TLS 时）
-   `gatewayTlsSha256=`（仅在启用 TLS 且指纹可用时）
-   `canvasPort=<端口>`（仅在启用画布主机时；目前与 `gatewayPort` 相同）
-   `sshPort=<端口>`（未覆盖时默认为 22）
-   `transport=gateway`
-   `cliPath=`（可选；可执行的 `openclaw` 入口点绝对路径）
-   `tailnetDns=`（Tailnet 可用时的可选提示）

安全注意事项：

-   Bonjour/mDNS TXT 记录是**未经认证的**。客户端不应将 TXT 内容视为权威的路由信息。
-   客户端应使用解析出的服务端点（SRV + A/AAAA 记录）进行路由。将 `lanHost`、`tailnetDns`、`gatewayPort` 和 `gatewayTlsSha256` 仅作为辅助提示。
-   TLS 证书固定绝不允许广播的 `gatewayTlsSha256` 覆盖已存储的固定值。
-   iOS/Android 节点应将基于发现的直连视为**仅限 TLS**，并在信任首次指纹前要求用户明确确认。

## 在 macOS 上调试

有用的内置工具：

-   浏览服务实例：

    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```

-   解析某个实例（替换 `<实例名>`）：

    ```bash
    dns-sd -L "<实例名>" _openclaw-gw._tcp local.
    ```

如果浏览成功但解析失败，通常是遇到了局域网策略限制或 mDNS 解析器问题。

## 在网关日志中调试

网关会写入滚动日志文件（启动时会打印 `gateway log file: ...`）。查找 `bonjour:` 开头的日志行，特别关注：

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## 在 iOS 节点上调试

iOS 节点使用 `NWBrowser` 来发现 `_openclaw-gw._tcp`。捕获日志的方法：

-   设置 → 网关 → 高级 → **发现调试日志**
-   设置 → 网关 → 高级 → **发现日志** → 重现问题 → **复制**

日志会包含浏览器状态转换和结果集变化。

## 常见故障原因

-   **Bonjour 无法跨网络**：使用 Tailnet 或 SSH。
-   **组播被阻止**：某些 Wi-Fi 网络禁用了 mDNS。
-   **休眠 / 网络接口变化**：macOS 可能暂时丢失 mDNS 结果；重试即可。
-   **浏览成功但解析失败**：保持机器名称简单（避免表情符号或特殊符号），然后重启网关。服务实例名称派生自主机名，过于复杂的名称可能导致某些解析器混乱。

## 转义的实例名称（\\032）

Bonjour/DNS-SD 常将服务实例名称中的字节转义为十进制 `\DDD` 序列（如空格变为 `\032`）。

-   这在协议层面是正常的。
-   UI 层应解码后显示（iOS 使用 `BonjourEscapes.decode`）。

## 禁用与配置

-   `OPENCLAW_DISABLE_BONJOUR=1` 禁用广播。
-   `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制网关绑定模式。
-   `OPENCLAW_SSH_PORT` 覆盖 TXT 中广播的 SSH 端口。
-   `OPENCLAW_TAILNET_DNS` 在 TXT 中发布 MagicDNS 提示。
-   `OPENCLAW_CLI_PATH` 覆盖广播的 CLI 路径。

## 相关文档

-   发现策略和传输选择：[服务发现](./discovery.md)
-   节点配对与审批：[网关配对](./pairing.md)

[发现与传输](./discovery.md)[远程访问](./remote.md)