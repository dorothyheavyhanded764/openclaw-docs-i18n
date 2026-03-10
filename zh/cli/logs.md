

  CLI 命令

  
# logs

通过 RPC 追踪网关日志文件（支持远程模式）。相关内容：

-   日志概述：[Logging](../logging.md)

## 示例

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

使用 `--local-time` 可以按本地时区显示时间戳。

[hooks](./hooks.md)[memory](./memory.md)