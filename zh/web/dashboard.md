

  Web 界面

  
# 仪表板（dashboard）

仪表板是网关提供的浏览器控制界面，默认路径为 `/`（可通过 `gateway.controlUi.basePath` 自定义）。

**本地网关快速访问：**

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) 或 [http://localhost:18789/](http://localhost:18789/)

**延伸阅读：**

- [控制界面](./control-ui.md) — 了解使用方法和界面功能
- [Tailscale](../gateway/tailscale.md) — 了解 Serve/Funnel 自动化
- [Web 界面](../web.md) — 了解绑定模式和安全性说明

## 关于安全性

身份验证在 WebSocket 握手阶段通过 `connect.params.auth`（令牌或密码）强制执行，具体配置参见 [网关配置](../gateway/configuration.md) 中的 `gateway.auth`。

:::caution
控制界面属于**管理界面**，支持聊天、配置修改、执行审批等敏感操作。**切勿将其暴露在公网上。**

界面会将仪表板 URL 中的令牌保留在当前标签页的内存中，并在页面加载后自动从 URL 中移除。推荐使用本地访问、Tailscale Serve 或 SSH 隧道。
:::

## 快速访问（推荐）

完成初始设置后，CLI 会自动打开仪表板并打印一个干净的（不含令牌的）链接。

**随时重新打开：**

```bash
openclaw dashboard
```

该命令会复制链接、尝试打开浏览器，若在无头环境下则显示 SSH 提示。

如果界面提示需要身份验证，将 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）中的令牌粘贴到控制界面设置中即可。

## 令牌使用（本地 vs 远程）

**本地访问：**

直接打开 `http://127.0.0.1:18789/` 即可。

**令牌来源：**

令牌来自 `gateway.auth.token`（或环境变量 `OPENCLAW_GATEWAY_TOKEN`）。`openclaw dashboard` 命令可以通过 URL 片段传递令牌完成一次性引导，但控制界面不会将网关令牌持久化存储在 localStorage 中。

**SecretRef 管理的令牌：**

如果 `gateway.auth.token` 由 SecretRef 管理，`openclaw dashboard` 会打印/复制/打开一个不含令牌的 URL。这样设计是为了避免在 shell 日志、剪贴板历史或浏览器启动参数中暴露外部管理的令牌。

如果 `gateway.auth.token` 配置为 SecretRef 但在当前 shell 中未解析，`openclaw dashboard` 仍会打印不含令牌的 URL，并给出身份验证设置指导。

**远程访问：**

非本地主机环境下，建议使用以下方式：

- **Tailscale Serve** — 若配置 `gateway.auth.allowTailscale: true`，控制界面和 WebSocket 连接无需令牌（前提是网关主机可信；HTTP API 仍需令牌/密码）
- **Tailnet 绑定** — 配合令牌使用
- **SSH 隧道** — 安全可靠

详见 [Web 界面](../web.md)。

## 遇到"未授权"/ 1008 错误？

按以下步骤排查：

**1. 确认网关可达**

- 本地：运行 `openclaw status`
- 远程：建立 SSH 隧道 `ssh -N -L 18789:127.0.0.1:18789 user@host`，然后打开 `http://127.0.0.1:18789/`

**2. 获取令牌**

从网关主机检索或提供令牌：

- 明文配置：`openclaw config get gateway.auth.token`
- SecretRef 管理的配置：解析外部密钥提供程序，或在此 shell 中导出 `OPENCLAW_GATEWAY_TOKEN`，然后重新运行 `openclaw dashboard`
- 未配置令牌：`openclaw doctor --generate-gateway-token`

**3. 完成认证**

在仪表板设置中将令牌粘贴到身份验证字段，然后连接。

[控制界面](./control-ui.md)[WebChat](./webchat.md)