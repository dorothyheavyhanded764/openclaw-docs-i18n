

  自动化

  
# 认证监控

OpenClaw 通过 `openclaw models status` 命令暴露 OAuth 过期状态，你可以直接用它来做自动化监控和告警；页面下方还提供了一些可选脚本，方便你在手机等移动端处理认证流程。

## 推荐方式：CLI 检查（通用、可移植）

```bash
openclaw models status --check
```

退出码含义：

-   `0`: 正常
-   `1`: 凭证已过期或缺失
-   `2`: 即将过期（24 小时内）

这个命令可以直接用在 cron 或 systemd 定时任务里，无需依赖任何额外脚本，是推荐的自动化方案。

## 可选脚本（运维场景 / 手机工作流）

这些脚本位于 `scripts/` 目录下，均为**可选**。它们假设你能通过 SSH 访问网关主机，并针对 systemd 和 Termux 环境做了优化：

-   `scripts/claude-auth-status.sh` — 现已使用 `openclaw models status --json` 作为数据源（CLI 不可用时回退到直接读取文件），因此请确保 `openclaw` 在 `PATH` 环境变量中，以便定时器调用
-   `scripts/auth-monitor.sh` — cron/systemd 定时任务入口脚本，发送告警通知（支持 ntfy 或手机推送）
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}` — systemd 用户定时器配置文件
-   `scripts/claude-auth-status.sh` — Claude Code + OpenClaw 认证状态检查器（支持 full/json/simple 三种输出格式）
-   `scripts/mobile-reauth.sh` — 通过 SSH 完成引导式重新认证流程
-   `scripts/termux-quick-auth.sh` — 一键查看状态并打开认证链接
-   `scripts/termux-auth-widget.sh` — 完整的 Termux 小部件引导流程
-   `scripts/termux-sync-widget.sh` — 同步 Claude Code 凭证到 OpenClaw

如果你不需要手机端自动化或 systemd 定时器，可以跳过这些脚本。

[轮询](./poll.md)[节点](../nodes.md)