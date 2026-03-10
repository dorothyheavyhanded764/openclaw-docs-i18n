

  智能体协调

  
# Agent Send

有时候，你可能需要主动让智能体做点什么——比如定时生成报告、批量处理任务，或者从其他系统触发一个回复。`openclaw agent` 命令就是为这种场景设计的：它无需等待聊天消息传入，就能直接运行一次智能体轮次。默认情况下，命令会**通过网关（Gateway）**执行；如果加 `--local` 参数，则会强制使用本机的嵌入式运行时。

## 命令行为

**必需参数**：`--message <文本内容>`

**会话选择**（三选一）：

- `--to <目标>`：根据目标派生会话密钥。群组/频道会保持各自的隔离会话，而私聊会统一合并到 `main` 会话。
- `--session-id `：直接复用一个已存在的会话。
- `--agent `：指定一个已配置好的智能体（会使用该智能体的 `main` 会话密钥）。

**运行机制**：使用与正常聊天回复相同的嵌入式智能体运行时。思考（thinking）和详细输出（verbose）的设置会持久化保存到会话中。

**输出格式**：

- 默认：直接打印回复文本（如有媒体则会附带 `MEDIA:` 行）
- `--json`：输出结构化的 JSON 数据，包含完整负载和元数据

**投递选项**：加上 `--deliver` 和 `--channel`，可以把回复自动发回指定的聊天渠道（目标格式与 `openclaw message --target` 一致）。还可以用 `--reply-channel`、`--reply-to`、`--reply-account` 来覆盖投递目标，而不影响当前会话。

**容错机制**：如果网关不可达，CLI 会自动**回退**到本地嵌入式运行。

## 使用示例

```bash
# 向手机号发送消息并获取回复
openclaw agent --to +15555550123 --message "status update"

# 让 ops 智能体总结日志
openclaw agent --agent ops --message "Summarize logs"

# 复用已有会话，开启中等思考级别
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium

# 详细模式 + JSON 输出
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json

# 将回复投递回原渠道
openclaw agent --to +15555550123 --message "Summon reply" --deliver

# 让 ops 智能体生成报告，并投递到 Slack 频道
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## 命令标志

| 标志 | 说明 |
|------|------|
| `--local` | 本地运行（需要在 shell 环境中配置好模型提供商的 API 密钥） |
| `--deliver` | 将回复发送到指定渠道 |
| `--channel` | 投递渠道，可选值：`whatsapp`、`telegram`、`discord`、`googlechat`、`slack`、`signal`、`imessage`，默认为 `whatsapp` |
| `--reply-to` | 覆盖投递目标 |
| `--reply-channel` | 覆盖投递渠道 |
| `--reply-account` | 覆盖投递账户 ID |
| `--thinking <级别>` | 持久化思考级别，可选：`off`、`minimal`、`low`、`medium`、`high`、`xhigh`（仅 GPT-5.2 和 Codex 模型支持） |
| `--verbose <级别>` | 持久化详细输出级别，可选：`on`、`full`、`off` |
| `--timeout <秒>` | 覆盖智能体超时时间 |
| `--json` | 输出结构化 JSON |

[浏览器故障排除](./browser-linux-troubleshooting.md)[子智能体](./subagents.md)