

  CLI 命令

  
# nodes

管理已配对节点（设备）并调用节点功能。相关内容：

-   节点概述：[Nodes](../nodes.md)
-   相机：[Camera nodes](../nodes/camera.md)
-   图像：[Image nodes](../nodes/images.md)

常用选项：

-   `--url`、`--token`、`--timeout`、`--json`

## 常用命令

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` 打印待处理/已配对的表格。已配对行包含最近的连接时间（Last Connect）。使用 `--connected` 仅显示当前连接的节点。使用 `--last-connected ` 筛选在指定时长内连接过的节点（例如 `24h`、`7d`）。

## Invoke / run

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Invoke 标志：

-   `--params `：JSON 对象字符串（默认 `{}`）。
-   `--invoke-timeout `：节点调用超时（默认 `15000`）。
-   `--idempotency-key `：可选的幂等键。

### Exec 风格默认值

`nodes run` 镜像模型的 exec 行为（默认值 + 审批）：

-   读取 `tools.exec.*`（以及 `agents.list[].tools.exec.*` 覆盖）。
-   在调用 `system.run` 之前使用 exec 审批（`exec.approval.request`）。
-   当设置了 `tools.exec.node` 时可以省略 `--node`。
-   需要声明 `system.run` 的节点（macOS 配套应用或无头节点主机）。

标志：

-   `--cwd `：工作目录。
-   `--env <key=val>`：环境变量覆盖（可重复）。注意：节点主机会忽略 `PATH` 覆盖（且 `tools.exec.pathPrepend` 不会应用于节点主机）。
-   `--command-timeout `：命令超时。
-   `--invoke-timeout `：节点调用超时（默认 `30000`）。
-   `--needs-screen-recording`：需要屏幕录制权限。
-   `--raw `：运行 shell 字符串（`/bin/sh -lc` 或 `cmd.exe /c`）。在 Windows 节点主机的允许列表模式下，`cmd.exe /c` shell 包装器运行需要审批（仅允许列表条目不会自动允许包装器形式）。
-   `--agent `：智能体范围的审批/允许列表（默认为配置的智能体）。
-   `--ask <off|on-miss|always>`、`--security <deny|allowlist|full>`：覆盖设置。

[node](./node.md)[onboard](./onboard.md)