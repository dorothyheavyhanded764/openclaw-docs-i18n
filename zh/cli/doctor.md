

  CLI 命令

  
# doctor

网关和频道的健康检查与快速修复。相关内容：

-   故障排除：[Troubleshooting](../gateway/troubleshooting.md)
-   安全审计：[Security](../gateway/security.md)

## 示例

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

注意事项：

-   交互式提示（如钥匙串/OAuth 修复）仅在 stdin 是 TTY 且未设置 `--non-interactive` 时运行。无头运行（cron、Telegram、无终端）会跳过提示。
-   `--fix`（`--repair` 的别名）会将备份写入 `~/.openclaw/openclaw.json.bak` 并丢弃未知的配置键，列出每个移除项。
-   状态完整性检查现在可以检测会话目录中的孤立转录文件，并可以将其归档为 `.deleted.` 以安全地回收空间。
-   Doctor 包含内存搜索就绪检查，当嵌入凭据缺失时会推荐 `openclaw configure --section model`。
-   如果沙盒模式已启用但 Docker 不可用，doctor 会报告高价值警告并提供修复建议（`安装 Docker` 或 `openclaw config set agents.defaults.sandbox.mode off`）。

## macOS：launchctl 环境变量覆盖

如果你之前运行过 `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`（或 `...PASSWORD`），该值会覆盖你的配置文件，可能导致持续的"未授权"错误。

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[docs](./docs.md)[gateway](./gateway.md)