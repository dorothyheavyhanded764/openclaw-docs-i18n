

  开发者设置

  
# 设置（setup）

> **ℹ️** 如果你是首次设置，请先阅读 [入门指南](./getting-started.md)。关于向导的详细说明，请参阅 [入门向导](./wizard.md)。

 最后更新：2026-01-01

## 速览

-   **定制内容放在仓库外面：** `~/.openclaw/workspace`（工作区）+ `~/.openclaw/openclaw.json`（配置文件）。
-   **稳定工作流：** 安装 macOS 应用，让它自动运行内置的网关（Gateway）。
-   **前沿工作流：** 通过 `pnpm gateway:watch` 自己运行网关（Gateway），然后让 macOS 应用以本地模式连接。

## 前提条件（从源码安装）

-   Node `>=22`
-   `pnpm`
-   Docker（可选，仅用于容器化设置或端到端测试 — 详见 [Docker](../install/docker.md)）

## 定制策略（让更新不破坏你的配置）

想要「完全为我定制」同时还能轻松更新？把你的个性化内容放在以下位置：

-   **配置：** `~/.openclaw/openclaw.json`（JSON 或类 JSON5 格式）
-   **工作区：** `~/.openclaw/workspace`（技能、提示词、记忆等；建议设为私有 git 仓库）

执行一次引导命令：

```bash
openclaw setup
```

在仓库内部，可以使用本地 CLI 入口：

```bash
openclaw setup
```

如果还没全局安装，可以通过 `pnpm openclaw setup` 运行。

## 从本仓库运行网关（Gateway）

完成 `pnpm build` 后，可以直接运行打包好的 CLI：

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## 稳定工作流（先装 macOS 应用）

1.  安装并启动 **OpenClaw.app**（菜单栏应用）。
2.  完成入门向导和权限检查清单（会弹出 TCC 权限提示）。
3.  确保网关（Gateway）处于 **本地（Local）** 模式并正在运行（应用会自动管理）。
4.  连接通信渠道（以 WhatsApp 为例）：

```bash
openclaw channels login
```

5.  健康检查：

```bash
openclaw health
```

如果你的构建版本没有入门向导：

-   先运行 `openclaw setup`，再运行 `openclaw channels login`，最后手动启动网关（Gateway）（`openclaw gateway`）。

## 前沿工作流（在终端中运行网关）

目标是：在 TypeScript 版网关（Gateway）上开发，获得热重载能力，同时保持 macOS 应用的 UI 连接。

### 0）（可选）也从源码运行 macOS 应用

如果你想让 macOS 应用也保持前沿版本：

```bash
./scripts/restart-mac.sh
```

### 1）启动开发版网关

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` 会以监视模式运行网关（Gateway），TypeScript 代码变更时自动重新加载。

### 2）让 macOS 应用连接你的网关

在 **OpenClaw.app** 中：

-   连接模式选择 **本地（Local）**，应用会连接到指定端口上正在运行的网关（Gateway）。

### 3）验证

-   应用内的网关状态应显示 **「正在使用现有 gateway …」**
-   或通过 CLI 检查：

```bash
openclaw health
```

### 常见坑

-   **端口搞错了：** 网关（Gateway）的 WebSocket 默认使用 `ws://127.0.0.1:18789`；确保应用和 CLI 使用相同的端口。
-   **状态存储在哪：**
    -   凭证：`~/.openclaw/credentials/`
    -   会话：`~/.openclaw/agents//sessions/`
    -   日志：`/tmp/openclaw/`

## 凭证存储位置一览

调试认证问题或决定备份内容时，可以参考这个清单：

-   **WhatsApp：** `~/.openclaw/credentials/whatsapp//creds.json`
-   **Telegram 机器人令牌：** 配置文件/环境变量，或 `channels.telegram.tokenFile`
-   **Discord 机器人令牌：** 配置文件/环境变量，或 SecretRef（支持环境变量/文件/命令执行等提供者）
-   **Slack 令牌：** 配置文件/环境变量（`channels.slack.*`）
-   **配对允许列表：**
    -   `~/.openclaw/credentials/-allowFrom.json`（默认账户）
    -   `~/.openclaw/credentials/--allowFrom.json`（非默认账户）
-   **模型认证配置：** `~/.openclaw/agents//agent/auth-profiles.json`
-   **文件存储的密钥载荷（可选）：** `~/.openclaw/secrets.json`
-   **旧版 OAuth 导入：** `~/.openclaw/credentials/oauth.json` 更多详情见 [安全](../gateway/security.md#credential-storage-map)。

## 更新（别搞坏你的配置）

-   把 `~/.openclaw/workspace` 和 `~/.openclaw/` 视为「你的东西」；不要把个人提示词或配置放进 `openclaw` 仓库里。
-   更新源码的方法：`git pull` + `pnpm install`（当 lockfile 有变化时）+ 继续用 `pnpm gateway:watch`。

## Linux（systemd 用户服务）

Linux 安装使用 systemd **用户**服务。默认情况下，systemd 会在用户登出或空闲时停止用户服务，这会终止网关（Gateway）。入门向导会尝试为你启用 lingering（可能需要输入 sudo 密码）。如果还是没启用，运行：

```bash
sudo loginctl enable-linger $USER
```

对于需要常开或多用户的服务器，建议使用 **系统**服务而非用户服务（不需要 lingering）。详见 [网关运行手册](../gateway.md) 中的 systemd 说明。

## 相关文档

-   [网关运行手册](../gateway.md)（启动参数、进程管理、端口配置）
-   [网关配置](../gateway/configuration.md)（配置模式与示例）
-   [Discord](../channels/discord.md) 和 [Telegram](../channels/telegram.md)（回复标签与 `replyToMode` 设置）
-   [OpenClaw 助手设置](./openclaw.md)
-   [macOS 应用](../platforms/macos.md)（网关生命周期管理）

[会话管理深度解析](../reference/session-management-compaction.md)[Pi 开发工作流](../pi-dev.md)