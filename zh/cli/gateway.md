

  CLI 命令

  
# gateway

网关（Gateway）是 OpenClaw 的 WebSocket 服务器，负责管理频道、节点、会话和钩子。本页介绍 `openclaw gateway` 下的所有子命令。相关文档：

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## 启动网关

启动一个本地网关进程：

```bash
openclaw gateway
```

前台运行的别名：

```bash
openclaw gateway run
```

几点注意事项：

-   默认情况下，网关要求在 `~/.openclaw/openclaw.json` 中设置 `gateway.mode=local` 才会启动。临时测试或开发时，可以用 `--allow-unconfigured` 跳过这个检查。
-   出于安全考虑，未经身份验证时无法绑定到环回地址以外的网络接口。
-   `SIGUSR1` 信号可以触发进程内重启（`commands.restart` 默认启用）。如果想禁止手动重启，设置 `commands.restart: false`，但网关工具和配置更新仍然有效。
-   `SIGINT` 和 `SIGTERM` 会停止网关进程，但不会恢复终端状态。如果你用 TUI 或原始模式包装了 CLI，记得在退出前手动恢复终端。

### 启动选项

-   `--port `：WebSocket 端口（默认从配置或环境变量读取，通常是 `18789`）。
-   `--bind <loopback|lan|tailnet|auto|custom>`：监听地址绑定模式。
-   `--auth <token|password>`：身份验证模式覆盖。
-   `--token `：令牌覆盖（同时为当前进程设置 `OPENCLAW_GATEWAY_TOKEN`）。
-   `--password `：密码覆盖（同时为当前进程设置 `OPENCLAW_GATEWAY_PASSWORD`）。
-   `--tailscale <off|serve|funnel>`：通过 Tailscale 暴露网关。
-   `--tailscale-reset-on-exit`：退出时重置 Tailscale serve/funnel 配置。
-   `--allow-unconfigured`：允许在未设置 `gateway.mode=local` 时启动网关。
-   `--dev`：自动创建开发配置和工作区（跳过 BOOTSTRAP.md）。
-   `--reset`：重置开发配置、凭据、会话和工作区（需要配合 `--dev`）。
-   `--force`：启动前强制关闭占用端口的进程。
-   `--verbose`：输出详细日志。
-   `--claude-cli-logs`：只在控制台显示 claude-cli 日志（并启用其 stdout/stderr）。
-   `--ws-log <auto|full|compact>`：WebSocket 日志格式（默认 `auto`）。
-   `--compact`：`--ws-log compact` 的简写。
-   `--raw-stream`：将原始模型流事件记录到 jsonl 文件。
-   `--raw-stream-path `：原始流 jsonl 文件路径。

## 查询运行中的网关

所有查询命令都通过 WebSocket RPC 与网关通信。输出格式支持：

-   默认：人类可读格式（在终端中带颜色）。
-   `--json`：机器可读的 JSON（不带样式和加载动画）。
-   `--no-color`（或设置环境变量 `NO_COLOR=1`）：禁用 ANSI 颜色，但保持人类可读的布局。

这些命令支持以下通用选项：

-   `--url `：网关的 WebSocket URL。
-   `--token `：网关令牌。
-   `--password `：网关密码。
-   `--timeout `：超时时间（不同命令的默认值不同）。
-   `--expect-final`：等待"最终"响应（用于智能体调用）。

注意：一旦设置了 `--url`，CLI 就不会再从配置文件或环境变量中读取凭据。你必须显式传入 `--token` 或 `--password`，否则会报错。

### gateway health

检查网关健康状态：

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` 显示网关服务的状态（launchd/systemd/schtasks），并可选地通过 RPC 探测网关是否响应。

```bash
openclaw gateway status
openclaw gateway status --json
```

选项：

-   `--url `：覆盖探测的 URL。
-   `--token `：探测时使用的令牌认证。
-   `--password `：探测时使用的密码认证。
-   `--timeout `：探测超时（默认 `10000` 毫秒）。
-   `--no-probe`：跳过 RPC 探测（只显示服务状态）。
-   `--deep`：同时扫描系统级服务。

注意：

-   `gateway status` 会尽量解析配置中的 SecretRef 来获取探测所需的认证信息。
-   如果某个 SecretRef 无法解析，探测认证可能会失败。这时请显式传入 `--token` 或 `--password`，或者先解决密钥来源问题。

### gateway probe

`gateway probe` 是一个"诊断神器"，它会同时探测：

-   你配置的远程网关（如果有的话）
-   本地网关（环回地址），**即使你已经配置了远程网关**

如果同时有多个网关可达，它会列出所有结果。当你使用隔离的配置文件或端口时（比如运行一个救援机器人），可能会有多个网关同时运行，但大多数情况下只有一个。

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### 通过 SSH 探测远程网关

macOS 应用的"通过 SSH 远程"模式会建立本地端口转发，让远程网关（可能只绑定了环回地址）在 `ws://127.0.0.1:` 上可访问。CLI 中等效的命令：

```bash
openclaw gateway probe --ssh user@gateway-host
```

选项：

-   `--ssh `：SSH 目标，格式为 `user@host` 或 `user@host:port`（端口默认 `22`）。
-   `--ssh-identity `：SSH 身份文件路径。
-   `--ssh-auto`：自动选择第一个发现的网关主机作为 SSH 目标（仅限 LAN/WAB）。

相关配置（可选，用作默认值）：

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

底层的 RPC 辅助命令，可以调用任意方法：

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## 管理网关服务

使用以下命令管理网关的系统服务：

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

注意事项：

-   `gateway install` 支持 `--port`、`--runtime`、`--token`、`--force`、`--json` 等选项。
-   当令牌认证需要令牌，且 `gateway.auth.token` 使用 SecretRef 管理时，`gateway install` 会验证 SecretRef 是否可解析，但不会把解析后的令牌写入服务环境变量。
-   如果令牌认证需要令牌，但配置中的 SecretRef 无法解析，安装会直接失败，而不会存储明文凭据。
-   在推断认证模式下，只在 shell 中设置的 `OPENCLAW_GATEWAY_PASSWORD` 或 `CLAWDBOT_GATEWAY_PASSWORD` 不会降低安装时的令牌要求。安装托管服务时，请使用持久化配置（`gateway.auth.password` 或配置中的 `env`）。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password`，但 `gateway.auth.mode` 未设置，安装会被阻止，直到你明确设置认证模式。
-   生命周期命令都支持 `--json`，方便脚本调用。

## 发现网关（Bonjour）

`gateway discover` 扫描网络中的网关信标（`_openclaw-gw._tcp`）。

扫描方式：

-   多播 DNS-SD：`local.`
-   单播 DNS-SD（广域 Bonjour）：选择一个域名（如 `openclaw.internal.`）并配置 split DNS 和 DNS 服务器，详见 [/gateway/bonjour](../gateway/bonjour.md)

只有启用了 Bonjour 发现（默认启用）的网关才会广播信标。广域发现记录中包含以下 TXT 字段：

-   `role`：网关角色提示
-   `transport`：传输提示，如 `gateway`
-   `gatewayPort`：WebSocket 端口，通常是 `18789`
-   `sshPort`：SSH 端口，如果不存在则默认 `22`
-   `tailnetDns`：MagicDNS 主机名（如果可用）
-   `gatewayTls` / `gatewayTlsSha256`：TLS 启用状态和证书指纹
-   `cliPath`：远程安装的可选提示路径

### gateway discover

扫描局域网中的网关：

```bash
openclaw gateway discover
```

选项：

-   `--timeout `：命令超时时间（浏览/解析），默认 `2000` 毫秒。
-   `--json`：机器可读输出（同时禁用样式和加载动画）。

示例：

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)