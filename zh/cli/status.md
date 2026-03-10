

  CLI 命令

  
# status

频道与会话诊断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

注意事项：

-   `--deep` 运行实时探测（WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal）。
-   当配置了多个智能体时，输出包含各智能体的会话存储信息。
-   概览在可用时包含网关 + 节点主机服务安装/运行时状态。
-   概览包含更新渠道 + git SHA（适用于源码检出）。
-   更新信息会显示在概览中；如果有可用更新，status 会打印提示运行 `openclaw update`（参见 [Updating](../install/updating.md)）。
-   只读状态命令（`status`、`status --json`、`status --all`）在可能的情况下会解析其目标配置路径支持的 SecretRef。
-   如果配置了支持的频道 SecretRef 但在当前命令路径中不可用，status 保持只读并报告降级输出而不是崩溃。人类可读输出会显示警告如"配置的令牌在当前命令路径中不可用"，JSON 输出包含 `secretDiagnostics`。

[skills](./skills.md)[system](./system.md)