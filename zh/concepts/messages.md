

  消息与投递

  
# 消息

本页面将带你了解 OpenClaw 如何处理入站消息、会话、队列、流式传输以及推理内容的可见性控制。

## 消息处理流程

消息从进入系统到最终回复，大致经历以下环节：

```
入站消息
  -> 路由/绑定 -> 会话标识
  -> 排队等待（若当前有正在执行的任务）
  -> 智能体执行（流式输出 + 工具调用）
  -> 出站回复（遵守通道限制 + 分块发送）
```

相关配置项分布在以下几个位置：

- `messages.*`：控制消息前缀、排队策略和群聊行为
- `agents.defaults.*`：设置流式输出和分块的默认参数
- 通道级配置（如 `channels.whatsapp.*`、`channels.telegram.*`）：覆盖各通道的限制和流式开关

完整配置说明请参阅[配置参考](../gateway/configuration.md)。

## 入站消息去重

通道在断线重连后可能会重复投递同一条消息。OpenClaw 通过一个短期缓存来解决这一问题——缓存以"通道/账户/对端/会话/消息ID"组合为键，确保重复投递不会触发新的智能体执行。

## 入站消息防抖

当同一发送者快速连续发送多条消息时，你可以通过 `messages.inbound` 将它们合并为一次智能体处理。防抖按"通道 + 会话"维度生效，并以最新消息作为回复的线程锚点。配置示例如下：

```json
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

几点说明：

- 防抖仅对**纯文本消息**生效，媒体和附件会立即处理
- 控制命令不受防抖影响，始终独立执行

## 会话与设备

会话由网关管理，而非客户端。

- 私聊会归入智能体的主会话
- 群组/频道各自拥有独立的会话
- 会话存储和对话记录保存在网关主机上

多个设备或通道可以指向同一个会话，但历史记录不会完全同步到每个客户端。建议长时间对话使用同一主设备，避免上下文出现分歧。控制界面和 TUI 显示的是网关上的会话记录，是可信的数据来源。详见[会话管理](./session.md)。

## 消息正文与历史上下文

OpenClaw 区分两种消息正文：

- `Body`：发送给智能体的提示文本，可能包含通道信封和历史上下文包装
- `CommandBody`：用户的原始输入文本，用于解析指令/命令
- `RawBody`：`CommandBody` 的旧别名（保留用于兼容）

当通道提供历史上下文时，会使用统一的包装格式：

- `[自您上次回复以来的聊天消息 - 供上下文参考]`
- `[当前消息 - 请回复此消息]`

对于**群组/频道/房间等非私聊场景**，当前消息会加上发送者标签（与历史消息格式一致），确保智能体能清晰区分不同来源。历史缓冲区只包含**未触发执行的消息**（如仅提及但未满足触发条件的消息），**不包含**已在会话记录中的消息。指令解析仅作用于当前消息，历史内容保持原样。

如果通道需要包装历史记录，应将 `CommandBody`（或 `RawBody`）设为原始消息文本，`Body` 设为完整的提示内容。历史缓冲区大小可通过以下配置调整：`messages.groupChat.historyLimit`（全局默认）以及各通道的覆盖项如 `channels.slack.historyLimit` 或 `channels.telegram.accounts..historyLimit`（设为 `0` 可禁用）。

## 消息队列与后续处理

当智能体正在执行时，新到达的消息可以排队等待、合并到当前执行，或收集后触发下一轮处理。

- 通过 `messages.queue` 和 `messages.queue.byChannel` 配置
- 支持的模式：`interrupt`（打断）、`steer`（引导）、`followup`（后续）、`collect`（收集），以及积压处理变体

详见[消息队列](./queue.md)。

## 流式输出、分块与批处理

流式输出让智能体在生成文本块时就能发送部分回复，提升响应感。分块功能则确保输出符合各通道的文本长度限制，并避免截断代码块。

关键配置项：

- `agents.defaults.blockStreamingDefault`（`on|off`，默认 `off`）
- `agents.defaults.blockStreamingBreak`（`text_end|message_end`）
- `agents.defaults.blockStreamingChunk`（`minChars|maxChars|breakPreference`）
- `agents.defaults.blockStreamingCoalesce`（基于空闲时间的批处理）
- `agents.defaults.humanDelay`（块回复间的拟人化延迟）
- 通道覆盖项：`*.blockStreaming` 和 `*.blockStreamingCoalesce`（非 Telegram 通道需显式设置 `*.blockStreaming: true`）

详见[流式输出与分块](./streaming.md)。

## 推理内容可见性与令牌

OpenClaw 支持控制模型推理过程的可见性：

- 使用 `/reasoning on|off|stream` 切换可见性
- 即使隐藏，推理内容仍会消耗令牌
- Telegram 支持将推理过程流式显示在输入提示气泡中

详见[思考与推理指令](../tools/thinking.md)和[令牌使用](../reference/token-use.md)。

## 前缀、线程与回复

出站消息的格式化配置集中在 `messages` 下：

- 前缀级联：`messages.responsePrefix` → `channels..responsePrefix` → `channels..accounts..responsePrefix`；另有 `channels.whatsapp.messagePrefix`（WhatsApp 入站前缀）
- 回复线程：通过 `replyToMode` 和各通道默认值控制

详见[配置参考](../gateway/configuration.md#messages)及各通道文档。

[在线状态](./presence.md)[流式输出与分块](./streaming.md)