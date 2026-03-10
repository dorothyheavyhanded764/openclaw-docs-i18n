

  CLI 命令

  
# dashboard

使用当前的认证信息打开控制界面（Control UI）。

```bash
openclaw dashboard
openclaw dashboard --no-open
```

注意事项：

-   `dashboard` 在可能的情况下会解析配置的 `gateway.auth.token` SecretRef。
-   对于 SecretRef 管理的令牌（无论已解析还是未解析），`dashboard` 会打印/复制/打开一个不含令牌的 URL，以避免在终端输出、剪贴板历史或浏览器启动参数中暴露外部密钥。
-   如果 `gateway.auth.token` 由 SecretRef 管理但在当前命令路径中未解析，命令会打印一个不含令牌的 URL 以及明确的修复指导，而不是嵌入无效的令牌占位符。

[daemon](./daemon.md)[devices](./devices.md)