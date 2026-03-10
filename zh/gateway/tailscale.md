

  远程访问

  
# Tailscale

本节介绍如何通过 Tailscale 安全地远程访问网关（Gateway）的控制面板和 WebSocket 端口。OpenClaw 支持自动配置 Tailscale 的 **Serve**（仅限 Tailnet 网络）或 **Funnel**（公网访问）功能——网关本身仍绑定在本地回环地址，而由 Tailscale 负责提供 HTTPS、路由以及身份头信息（Serve 模式）。

## 模式

Tailscale 支持三种运行模式：

- `serve`：通过 `tailscale serve` 实现仅限 Tailnet 网络内部访问。网关保持在 `127.0.0.1` 上监听。
- `funnel`：通过 `tailscale funnel` 对公网开放 HTTPS 访问。OpenClaw 需要配置共享密码。
- `off`：默认值，不启用 Tailscale 自动化功能。

## 身份验证

通过 `gateway.auth.mode` 控制握手认证方式：

- `token`：当设置了 `OPENCLAW_GATEWAY_TOKEN` 环境变量时的默认选项
- `password`：通过 `OPENCLAW_GATEWAY_PASSWORD` 环境变量或配置文件指定的共享密钥

**无令牌认证（仅限 Serve 模式）**

当 `tailscale.mode = "serve"` 且 `gateway.auth.allowTailscale` 为 `true` 时，控制面板和 WebSocket 可以直接使用 Tailscale 的身份头（`tailscale-user-login`）进行认证，无需手动提供令牌或密码。

OpenClaw 会通过本地 Tailscale 守护进程（`tailscale whois`）解析请求来源地址，验证其与身份头是否匹配后才接受请求。只有同时满足以下条件的请求才会被视为合法的 Serve 请求：来自本地回环地址、且携带 Tailscale 的 `x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host` 头。

注意：HTTP API 端点（如 `/v1/*`、`/tools/invoke` 和 `/api/channels/*`）仍需要令牌或密码认证。

安全提示：这种无令牌认证流程假设网关所在主机是可信环境。如果该主机上可能运行不受信任的本地代码，请禁用 `gateway.auth.allowTailscale` 并强制使用令牌或密码认证。如需显式凭据验证，设置 `gateway.auth.allowTailscale: false` 或强制指定 `gateway.auth.mode: "password"`。

## 配置示例

### Tailnet 网络内部访问（Serve 模式）

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

访问地址：`https:///`（或您配置的 `gateway.controlUi.basePath`）

### 直接绑定 Tailnet IP

当您希望网关直接监听 Tailnet IP 而不使用 Serve/Funnel 时，可使用此配置：

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

从同一 Tailnet 网络中的其他设备连接：

- 控制面板：`http://<tailscale-ip>:18789/`
- WebSocket：`ws://<tailscale-ip>:18789`

注意：此模式下本地回环地址（`http://127.0.0.1:18789`）将**无法**访问。

### 公网访问（Funnel + 共享密码）

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

安全建议：推荐使用 `OPENCLAW_GATEWAY_PASSWORD` 环境变量传递密码，避免将密码写入配置文件。

## CLI 示例

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 注意事项

- Tailscale Serve/Funnel 需要先安装并登录 `tailscale` CLI 工具。
- `tailscale.mode: "funnel"` 必须配合 `password` 认证模式才能启动，以防止未授权的公网暴露。
- 如需在 OpenClaw 关闭时自动撤销 `tailscale serve` 或 `tailscale funnel` 配置，可设置 `gateway.tailscale.resetOnExit`。
- `gateway.bind: "tailnet"` 是直接绑定 Tailnet IP 的方式，不提供 HTTPS，也不使用 Serve/Funnel。
- `gateway.bind: "auto"` 默认优先使用回环地址；如需仅限 Tailnet 网络访问，应明确指定 `tailnet`。
- Serve/Funnel 仅暴露**网关控制面板和 WebSocket**。由于节点通过同一 WebSocket 端点连接，Serve 模式也支持节点访问。

## 浏览器控制（远程网关 + 本地浏览器）

如果您在一台机器上运行网关，但需要在另一台机器上操控浏览器，可以在浏览器所在的机器上运行一个**节点主机**，并确保两台机器在同一个 Tailnet 网络中。网关会将浏览器操作代理到节点，无需额外的控制服务器或 Serve URL。

安全建议：避免使用 Funnel 进行浏览器控制，节点配对应视为操作员级别的内部访问。

## Tailscale 前置条件与限制

- Serve 模式要求为 Tailnet 网络启用 HTTPS；如未启用，CLI 会提示您开启。
- Serve 会注入 Tailscale 身份头，Funnel 不会。
- Funnel 需要 Tailscale v1.38.3+、启用 MagicDNS 和 HTTPS，以及 funnel 节点属性。
- Funnel 仅支持 TLS 端口 `443`、`8443` 和 `10000`。
- macOS 上使用 Funnel 需要安装开源版本的 Tailscale 应用。

## 延伸阅读

- Tailscale Serve 概述：[https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
- `tailscale serve` 命令：[https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
- Tailscale Funnel 概述：[https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
- `tailscale funnel` 命令：[https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

|[远程网关设置](./remote-gateway-readme.md)|[形式化验证（安全模型）](../security/formal-verification.md)|