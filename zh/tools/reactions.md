

  内置工具

  
# Reactions

Reactions 工具让你可以在消息上添加或移除表情反应，支持 Discord、Slack、Telegram、WhatsApp 等多种平台。本节介绍各平台通用的语义规则和注意事项。

所有渠道共享以下语义规则：

- 添加反应时，`emoji` 字段必须填写。
- 当平台支持时，设置 `emoji=""`（空字符串）会移除智能体（agent）在该消息上的所有反应。
- 当平台支持时，设置 `remove: true` 会移除指定的表情符号（此选项必须配合 `emoji` 使用）。

各平台的实现细节：

- **Discord/Slack**：空的 `emoji` 会移除智能体在该消息上的全部反应；`remove: true` 仅移除指定的单个表情。
- **Google Chat**：空的 `emoji` 会移除应用在该消息上的全部反应；`remove: true` 仅移除指定的单个表情。
- **Telegram**：空的 `emoji` 会移除智能体的全部反应；`remove: true` 同样可以移除反应，但工具验证仍要求 `emoji` 为非空值。
- **WhatsApp**：空的 `emoji` 会移除智能体的反应；`remove: true` 在内部映射为空表情（但仍需提供 `emoji` 参数）。
- **Zalo Personal (`zalouser`)**：必须提供非空的 `emoji`；`remove: true` 用于移除该特定表情的反应。
- **Signal**：当启用 `channels.signal.reactionNotifications` 配置时，收到的反应通知会触发系统事件。

[工具循环检测](./loop-detection.md)[思考层级](./thinking.md)