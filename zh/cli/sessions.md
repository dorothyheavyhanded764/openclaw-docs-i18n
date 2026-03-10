

  CLI 命令

  
# sessions

当你需要查看当前存储了哪些对话会话，或者想要清理过期的会话数据时，就可以使用 `sessions` 命令。

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

你可以通过以下选项来指定查询范围：

-   默认：使用配置中的默认智能体（agent）存储
-   `--agent `：指定某个已配置的智能体存储
-   `--all-agents`：聚合所有已配置的智能体存储
-   `--store `：显式指定存储路径（不能与 `--agent` 或 `--all-agents` 同时使用）

下面是 `openclaw sessions --all-agents --json` 的输出示例：

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## 清理维护

如果你想立即执行会话清理（而不必等待下次写入周期），可以使用 `cleanup` 子命令：

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` 会按照配置文件中的 `session.maintenance` 设置来执行清理。以下是各选项的说明：

-   **范围说明**：`cleanup` 只维护会话存储和记录，不会清理定时任务（cron）的运行日志（`cron/runs/.jsonl`）。后者由 [Cron 配置](../automation/cron-jobs.md#configuration) 中的 `cron.runLog.maxBytes` 和 `cron.runLog.keepLines` 管理，详见 [Cron 维护](../automation/cron-jobs.md#maintenance)。
-   `--dry-run`：预览将要执行的操作，显示有多少条目会被修剪或限制，但不会实际写入。在文本模式下，会打印一张按会话分类的操作表（包含 `Action`、`Key`、`Age`、`Model`、`Flags` 列），让你清楚看到哪些会保留、哪些会移除。
-   `--enforce`：即使 `session.maintenance.mode` 设置为 `warn`，也强制执行维护。
-   `--active-key `：保护指定的活动会话键（session key），使其不会被磁盘预算机制驱逐。
-   `--agent `：仅对指定的智能体存储执行清理。
-   `--all-agents`：对所有已配置的智能体存储执行清理。
-   `--store `：针对指定的 `sessions.json` 文件执行清理。
-   `--json`：以 JSON 格式输出摘要。配合 `--all-agents` 使用时，会为每个存储分别输出一份摘要。

`openclaw sessions cleanup --all-agents --dry-run --json` 的输出示例：

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

**相关链接：**

-   会话配置：[配置参考](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)