

  CLI 命令

  
# system

网关的系统级辅助工具：入队系统事件、控制心跳以及查看在线状态。

## 常用命令

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## system event

在**主**会话上入队系统事件。下一次心跳会将其作为 `System:` 行注入到提示中。使用 `--mode now` 立即触发心跳；`next-heartbeat` 等待下一次预定时间点。标志：

-   `--text `：必需的系统事件文本。
-   `--mode `：`now` 或 `next-heartbeat`（默认）。
-   `--json`：机器可读输出。

## system heartbeat last|enable|disable

心跳控制：

-   `last`：显示最后一次心跳事件。
-   `enable`：重新开启心跳（如果之前被禁用请使用此命令）。
-   `disable`：暂停心跳。

标志：

-   `--json`：机器可读输出。

## system presence

列出网关当前知道的系统在线状态条目（节点、实例及类似状态行）。标志：

-   `--json`：机器可读输出。

## 注意事项

-   需要运行中的网关可通过当前配置访问（本地或远程）。
-   系统事件是临时的，重启后不会持久化。

[status](./status.md)[tui](./tui.md)