

  CLI 命令

  
# voicecall

`voicecall` 是插件提供的命令。仅在安装并启用了 voice-call 插件时才会出现。主要文档：

-   Voice-call 插件：[Voice Call](../plugins/voice-call.md)

## 常用命令

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## 暴露 webhook（Tailscale）

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

安全提示：仅将 webhook 端点暴露给你信任的网络。尽可能优先使用 Tailscale Serve 而非 Funnel。

[update](./update.md)[webhooks](./webhooks.md)