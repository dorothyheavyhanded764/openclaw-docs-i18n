

  内置工具

  
# 思考层级

思考层级让你可以控制智能体的推理深度——从快速响应到深度思考，灵活适配不同任务需求。

## 功能说明

你可以在任何入站消息中使用内联指令来调整思考深度：

- `/t `、`/think:` 或 `/thinking `

### 可用层级

| 层级 | 别名 | 说明 |
|------|------|------|
| `off` | - | 关闭思考 |
| `minimal` | "think" | 最小思考 |
| `low` | "think hard" | 低强度思考 |
| `medium` | "think harder" | 中等强度思考 |
| `high` | "ultrathink" | 高强度思考（最大预算） |
| `xhigh` | "ultrathink+" | 超高强度思考（仅限 GPT-5.2 + Codex 模型） |
| `adaptive` | - | 提供商管理的自适应推理预算（支持 Anthropic Claude 4.6 模型系列） |

以下变体会自动映射：`x-high`、`x_high`、`extra-high`、`extra high`、`extra_high` → `xhigh`；`highest`、`max` → `high`。

### 各提供商特殊说明

- **Anthropic Claude 4.6 模型**：未显式设置思考层级时，默认使用 `adaptive`。
- **Z.AI (`zai/*`)**：仅支持开关式思考（`on`/`off`）。任何非 `off` 的层级都被视为 `on`（映射到 `low`）。
- **Moonshot (`moonshot/*`)**：将 `/think off` 映射到 `thinking: { type: "disabled" }`，其他层级映射到 `thinking: { type: "enabled" }`。思考启用时，Moonshot 只接受 `tool_choice` 为 `auto` 或 `none`；OpenClaw 会将不兼容的值规范化为 `auto`。

## 解析顺序

思考层级的生效优先级如下：

1. **消息内联指令**——仅对该条消息生效
2. **会话覆盖**——通过发送纯指令消息设置
3. **全局默认值**——配置中的 `agents.defaults.thinkingDefault`
4. **后备方案**——Anthropic Claude 4.6 模型为 `adaptive`，其他支持推理的模型为 `low`，否则为 `off`

## 设置会话默认值

想让某个思考层级在整个会话中保持有效？发送一条**仅包含指令**的消息（可以有空格）：

```
/think:medium
```

或：

```
/t high
```

这样就会在当前会话中生效（默认按发送者区分）。要清除这个设置，发送 `/think:off` 或等待会话空闲重置。

系统会返回确认消息（如 "Thinking level set to high." 或 "Thinking disabled."）。如果层级无效（比如 `/thinking big`），命令会被拒绝并给出提示，会话状态保持不变。

想查看当前思考层级？发送不带参数的 `/think` 或 `/think:` 即可。

## 按智能体应用

- **嵌入式 Pi**：解析后的层级会传递给进程内的 Pi 智能体运行时。

## 详细日志指令 (`/verbose` 或 `/v`)

想在对话中看到更多工具执行细节？使用详细日志指令。

### 层级

- `on`（最小详细）
- `full`（完整详细）
- `off`（默认关闭）

### 用法

发送一条仅包含指令的消息，即可切换会话的详细日志状态。系统会回复 "Verbose logging enabled." 或 "Verbose logging disabled."。层级无效时会返回提示，状态不变。

- `/verbose off` 会存储一个显式的会话覆盖；可通过会话 UI 选择 `inherit` 来清除。
- 内联指令仅影响当前消息；其他情况使用会话或全局默认值。
- 发送不带参数的 `/verbose` 或 `/verbose:` 可查看当前详细日志级别。

### 详细日志开启时的行为

当详细日志开启时，发出结构化工具结果的智能体（如 Pi 和其他 JSON 智能体）会将每个工具调用作为单独的元数据消息发送，格式为 ` <tool-name>: `（路径或命令可用时）。这些工具摘要会在每个工具启动时立即发送（单独的对话气泡），而不是作为流式增量发送。

工具失败摘要在正常模式下仍然可见，但原始错误详情后缀会被隐藏——除非详细日志级别为 `on` 或 `full`。

当详细日志级别为 `full` 时，工具输出也会在完成后转发（单独的对话气泡，截断到安全长度）。如果在智能体运行过程中切换 `/verbose on|full|off`，后续的工具气泡会遵循新设置。

## 推理可见性 (`/reasoning`)

想在回复中看到智能体的推理过程？使用推理可见性指令。

### 层级

- `on`：显示推理
- `off`：隐藏推理
- `stream`：流式推理（仅限 Telegram）

### 用法

发送仅包含指令的消息可切换是否在回复中显示思考区块。启用后，推理会作为**单独的消息**发送，前缀为 "Reasoning:"。

`stream` 模式（仅 Telegram）：在生成回复时将推理流式传输到 Telegram 的草稿气泡中，然后发送最终答案（不包含推理）。

别名：`/reason`

发送不带参数的 `/reasoning` 或 `/reasoning:` 可查看当前的推理级别。

## 相关链接

- [提升模式](./elevated.md) 文档

## 心跳检测

心跳探测的正文是配置好的心跳提示（默认："Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK."）。心跳消息中的内联指令照常生效，但应避免通过心跳消息更改会话默认值。

心跳传递默认仅发送最终有效载荷。若要同时发送单独的 `Reasoning:` 消息（当可用时），可设置：

- `agents.defaults.heartbeat.includeReasoning: true`（全局）
- `agents.list[].heartbeat.includeReasoning: true`（按智能体）

## 网页聊天界面

网页聊天的思考选择器在页面加载时会显示会话存储/配置中的层级。

选择另一个层级仅对**下一条消息**生效（`thinkingOnce`）；发送后，选择器会跳回存储的会话层级。

要更改会话默认值，请发送 `/think:` 指令，选择器会在下次重新加载后反映新设置。

[反应](./reactions.md)[网页工具](./web.md)