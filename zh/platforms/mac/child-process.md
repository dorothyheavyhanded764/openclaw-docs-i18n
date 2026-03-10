

  macOS 伴侣应用

  
# 网关生命周期

macOS 应用默认**通过 launchd 管理网关**，而不会将网关作为子进程启动。它会先尝试连接到已在配置端口上运行的网关；如果连接失败，则通过外部的 `openclaw` CLI（无嵌入式运行时）启动 launchd 服务。这样你就能获得可靠的登录自动启动和崩溃重启功能。子进程模式（由应用直接启动网关）**目前未使用**。如果你需要与 UI 更紧密的耦合，请在终端中手动运行网关。

## 默认行为 (launchd)

-   应用会安装一个每用户 LaunchAgent，标签为 `ai.openclaw.gateway`（或使用 `--profile`/`OPENCLAW_PROFILE` 时为 `ai.openclaw.`；旧版 `com.openclaw.*` 仍受支持）。
-   当启用本地模式时，应用会确保 LaunchAgent 已加载并在需要时启动网关。
-   日志写入 launchd 网关日志路径（可在调试设置中查看）。

常用命令：

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

运行命名配置文件时，请将标签替换为 `ai.openclaw.`。

## 未签名的开发构建

`scripts/restart-mac.sh --no-sign` 用于在没有签名密钥时进行快速的本地构建。为防止 launchd 指向未签名的中继二进制文件，它会：

-   写入 `~/.openclaw/disable-launchagent`。

已签名的 `scripts/restart-mac.sh` 运行会清除此覆盖（如果标记存在）。要手动重置：

```bash
rm ~/.openclaw/disable-launchagent
```

## 仅附加模式

要强制 macOS 应用**永不安装或管理 launchd**，请使用 `--attach-only`（或 `--no-launchd`）启动它。这会设置 `~/.openclaw/disable-launchagent`，因此应用只会连接到已运行的网关。你可以在调试设置中切换相同的行为。

## 远程模式

远程模式从不启动本地网关。应用通过 SSH 隧道连接到远程主机，并通过该隧道进行通信。

## 我们为何首选 launchd

-   登录时自动启动。
-   内置重启/KeepAlive 机制。
-   可预测的日志和监督。

如果将来确实需要真正的子进程模式，应将其记录为单独的、明确的仅开发模式。

[Canvas](./canvas.md)[健康检查](./health.md)