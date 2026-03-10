

  配置

  
# 群组消息

本节解决的问题是：让 Clawd 能够驻留在 WhatsApp 群组中，仅在收到 @提及 时才唤醒响应，同时确保群聊会话与个人私聊会话完全隔离。

需要注意的是，`agents.list[].groupChat.mentionPatterns` 这个配置项现已支持 Telegram、Discord、Slack 和 iMessage，但本文档主要聚焦于 WhatsApp 的特定行为。如果你需要配置多个智能体，请为每个智能体单独设置 `agents.list[].groupChat.mentionPatterns`，或者使用 `messages.groupChat.mentionPatterns` 作为全局回退配置。

## 已实现功能（截至 2025-12-03）

- **激活模式**：支持 `mention`（默认）或 `always` 两种模式。`mention` 模式要求收到"召唤"才会响应——可以是 WhatsApp 原生的 @提及（通过 `mentionedJids` 字段），也可以是正则表达式匹配的内容，或消息正文任意位置出现的机器人 E.164 号码。`always` 模式会在每条消息时都唤醒智能体，但智能体应仅在能够提供有价值回复时才响应，否则返回静默令牌 `NO_REPLY`。默认值可在 `channels.whatsapp.groups` 中配置，并可通过 `/activation` 命令针对单个群组进行覆盖。当设置了 `channels.whatsapp.groups` 时，它同时充当群组白名单（添加 `"*"` 表示允许所有群组）。

- **群组策略**：`channels.whatsapp.groupPolicy` 控制是否接受群组消息，可选值为 `open`、`disabled` 或 `allowlist`。`allowlist` 模式会使用 `channels.whatsapp.groupAllowFrom` 配置（若未设置则回退到 `channels.whatsapp.allowFrom`）。默认值为 `allowlist`，即需要先添加发送者才能触发响应。

- **独立会话**：每个群组拥有独立的会话键（session key），格式为 `agent::whatsapp:group:`。这意味着 `/verbose on` 或 `/think high` 等命令（需作为独立消息发送）只会影响当前群组的会话，不会影响你的个人私聊会话。群组会话会跳过心跳检测。

- **上下文注入**：对于未触发运行的待处理群组消息（默认保留最近 50 条），系统会将其作为上下文注入到提示词中，标注为 `[Chat messages since your last reply - for context]`，而当前触发响应的消息则标注为 `[Current message - respond to this]`。已在会话历史中的消息不会被重复注入。

- **发送者标识**：每个群组消息批次末尾都会标注 `[from: Sender Name (+E164)]`，让智能体知道是谁在发言。

- **阅后即焚消息**：系统会在提取文本和提及信息之前先解包阅后即焚/单次查看消息，因此这类消息中的 @提及 仍可正常触发响应。

- **群组系统提示**：在群组会话的首次对话（以及每次通过 `/activation` 更改激活模式时），系统会向提示词中注入一段简短说明，例如：`You are replying inside the WhatsApp group "<群组名>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` 即使元数据不可用，系统也会告知智能体当前处于群组聊天环境。

## 配置示例（WhatsApp）

在 `~/.openclaw/openclaw.json` 中添加 `groupChat` 配置块，这样即使 WhatsApp 在消息正文中移除了可视的 `@` 符号，基于显示名称的 @提及 仍然可以正常工作：

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

几点说明：

- 正则表达式匹配不区分大小写，可以覆盖 `@openclaw` 这样的显示名称 @提及，也可以匹配带或不带 `+` 号和空格的电话号码。
- 当用户通过点击联系人卡片来 @提及 时，WhatsApp 会通过 `mentionedJids` 字段发送规范的提及信息，因此号码匹配只是作为备用方案，但确实是个有用的安全网。

### 激活命令（仅限所有者）

在群聊中使用以下命令：

- `/activation mention`
- `/activation always`

只有所有者号码（来自 `channels.whatsapp.allowFrom` 配置，或未设置时使用机器人自身的 E.164 号码）可以更改激活模式。在群组中发送 `/status` 命令（作为独立消息）可查看当前激活模式。

## 使用方法

1. 将运行 OpenClaw 的 WhatsApp 账号添加到目标群组。
2. 发送 `@openclaw …`（或在消息中包含机器人的电话号码）。注意：除非你设置了 `groupPolicy: "open"`，否则只有白名单中的发送者才能触发响应。
3. 智能体的提示词会包含最近的群组上下文以及末尾的 `[from: …]` 标记，使其能够回复正确的发送者。
4. 会话级别的指令（如 `/verbose on`、`/think high`、`/new` 或 `/reset`、`/compact`）只会作用于当前群组的会话；请将这些指令作为独立消息发送以便系统识别。你的个人私聊会话保持独立，不受影响。

## 测试与验证

- **手动冒烟测试**：
  - 在群组中发送 `@openclaw` @提及，确认智能体的回复正确引用了发送者姓名。
  - 发送第二条 @提及 消息，验证历史上下文块是否正确包含，并在下一轮对话后被清除。
- **检查网关日志**：使用 `--verbose` 参数运行，查看 `inbound web message` 日志条目，确认其中显示 `from: ` 和 `[from: …]` 后缀。

## 已知注意事项

- 群组会话有意跳过心跳检测，以避免在群组中产生干扰性广播消息。
- 回声抑制基于组合的批次字符串；如果你连续发送两条不含 @提及 的相同文本，只有第一条会得到回复。
- 会话存储条目在会话存储文件（默认位于 `~/.openclaw/agents//sessions/sessions.json`）中显示为 `agent::whatsapp:group:`；如果找不到某群组的条目，仅表示该群组尚未触发过运行。
- 群组中的输入指示器遵循 `agents.defaults.typingMode` 配置（默认值：未 @提及 时为 `message`）。

[配对](./pairing.md)[群组](./groups.md)