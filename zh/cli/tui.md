

  CLI 命令

  
# tui

打开连接到网关的终端界面。相关内容：

-   TUI 指南：[TUI](../web/tui.md)

注意事项：

-   `tui` 在可能的情况下会解析配置的网关认证 SecretRef 用于令牌/密码认证（`env`/`file`/`exec` 提供商）。

## 示例

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)