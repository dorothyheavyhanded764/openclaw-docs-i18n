

  CLI 命令

  
# cron

管理 Gateway 调度器的定时任务（cron）。相关主题：

-   定时任务详解：[定时任务（cron）](../automation/cron-jobs.md)

**小贴士**：运行 `openclaw cron --help` 可查看完整的命令列表。

以下是几个重要的默认行为说明：

-   **独立任务的投递**：使用 `cron add` 创建的独立（isolated）任务默认开启 `--announce` 投递模式。如果想将输出保持在内部而不发送，请使用 `--no-deliver` 参数。`--deliver` 是 `--announce` 的已弃用别名，请避免使用。

-   **一次性任务的保留**：一次性（`--at`）任务执行成功后会自动删除。如果需要保留任务记录，请使用 `--keep-after-run` 参数。

-   **循环任务的自动重试**：循环任务在连续出错时会采用指数退避策略进行重试（30秒 → 1分钟 → 5分钟 → 15分钟 → 60分钟），当再次成功执行后恢复正常调度周期。

-   **日志清理配置**：通过配置文件控制会话和日志的保留策略：
    -   `cron.sessionRetention`（默认 `24h`）：清理已完成的独立任务运行会话
    -   `cron.runLog.maxBytes` + `cron.runLog.keepLines`：清理 `~/.openclaw/cron/runs/.jsonl` 日志文件

## 常用操作示例

**更新投递设置而不修改任务内容**：

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

**禁用独立任务的消息投递**：

```bash
openclaw cron edit <job-id> --no-deliver
```

**为独立任务启用轻量级启动上下文**：

```bash
openclaw cron edit <job-id> --light-context
```

**将结果通知发送到指定频道**：

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

**创建带轻量级上下文的独立任务**：

```bash
openclaw cron add \
  --name "轻量级晨间简报" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "总结夜间更新。" \
  --light-context \
  --no-deliver
```

`--light-context` 参数仅适用于独立的智能体（agent）单轮任务。启用后，定时任务运行时将使用空的启动上下文，而不是注入完整的工作空间引导信息集。

[配置](./configure.md)[守护进程](./daemon.md)