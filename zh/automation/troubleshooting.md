

  自动化

  
# 自动化故障排除（troubleshooting）

本页帮助你解决调度器和消息交付相关的问题，主要涉及 `cron` 和 `heartbeat` 两个功能。

## 命令阶梯

先从这些基础诊断命令开始：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后运行自动化相关的检查：

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron 未触发

如果定时任务没有按预期执行，运行以下命令排查：

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

正常的输出应该具备以下特征：

- `cron status` 显示调度器已启用，且 `nextWakeAtMs` 是一个未来的时间点
- 任务处于启用状态，计划表达式和时区配置正确
- `cron runs` 显示状态为 `ok`，或者有明确的跳过原因

常见的错误信息：

- `cron: scheduler disabled; jobs will not run automatically` → 调度器在配置或环境变量中被禁用了
- `cron: timer tick failed` → 调度器的计时器崩溃了，需要查看日志中的堆栈信息和上下文
- `reason: not-due` 出现在运行输出中 → 手动执行时没有加 `--force` 参数，而任务还没有到执行时间

## Cron 已触发但消息未送达

定时任务执行了，但没有收到预期的消息，按以下步骤检查：

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

正常的输出应该具备以下特征：

- 运行状态显示为 `ok`
- 独立任务已正确配置交付模式和目标
- 通道探测显示目标通道已连接

常见的错误信息：

- 运行成功但交付模式为 `none` → 这种情况下不会产生外部消息，属于预期行为
- 交付目标缺失或无效（`channel`/`to`）→ 任务内部执行成功了，但跳过了消息发送
- 通道认证错误（`unauthorized`、`missing_scope`、`Forbidden`）→ 通道的凭据或权限不足，导致消息发送被阻止

## Heartbeat 被抑制或跳过

如果心跳消息没有按时发送，运行以下命令排查：

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

正常的输出应该具备以下特征：

- Heartbeat 已启用，且间隔时间大于零
- 最后一次 heartbeat 的结果为 `ran`，或者跳过原因符合预期

常见的错误信息：

- `heartbeat skipped` 且 `reason=quiet-hours` → 当前时间不在 `activeHours` 配置的活跃时段内
- `requests-in-flight` → 主通道正在处理其他请求，heartbeat 被延后
- `empty-heartbeat-file` → `HEARTBEAT.md` 文件中没有可操作的内容，且队列中没有带标签的 cron 事件，因此跳过了本次间隔 heartbeat
- `alerts-disabled` → 可见性设置禁止了外发的 heartbeat 消息

## 时区和 activeHours 的常见陷阱

时区配置不当是导致定时任务和心跳异常的常见原因。运行以下命令检查配置：

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

快速判断规则：

- `Config path not found: agents.defaults.userTimezone` 表示该配置项未设置，heartbeat 会回退使用主机时区（如果设置了 `activeHours.timezone` 则优先使用）
- 未指定 `--tz` 参数的 cron 任务使用网关主机的时区
- Heartbeat 的 `activeHours` 使用配置中指定的时区解析方式（`user`、`local` 或显式的 IANA 时区）
- 在 cron 的 `at` 计划中，不带时区的 ISO 时间戳会被视为 UTC

常见的错误模式：

- 主机时区变更后，任务在"错误"的实际时间执行了
- 你的白天时段 heartbeat 总是被跳过，原因是 `activeHours.timezone` 设置成了其他时区

相关链接：

- [/automation/cron-jobs](./cron-jobs.md)
- [/gateway/heartbeat](../gateway/heartbeat.md)
- [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
- [/concepts/timezone](../concepts/timezone.md)

[Cron vs Heartbeat](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)