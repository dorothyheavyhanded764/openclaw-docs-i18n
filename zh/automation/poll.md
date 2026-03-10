

  自动化

  
# 投票

本节介绍如何在多个消息平台上发送投票，帮助你快速收集群组意见、做决策或进行趣味互动。

## 支持的渠道

-   Telegram
-   WhatsApp（网页渠道）
-   Discord
-   MS Teams（自适应卡片）

## CLI 命令行

通过 `openclaw message poll` 命令即可在各平台发送投票：

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

### 命令选项说明

-   `--channel`：指定渠道，支持 `whatsapp`（默认）、`telegram`、`discord` 或 `msteams`
-   `--poll-multi`：允许多选
-   `--poll-duration-hours`：仅 Discord 支持，省略时默认 24 小时
-   `--poll-duration-seconds`：仅 Telegram 支持，范围 5-600 秒
-   `--poll-anonymous` / `--poll-public`：仅 Telegram 支持的投票可见性设置

## Gateway RPC 接口

调用 `poll` 方法发送投票，参数如下：

-   `to`（字符串，必需）：目标接收方
-   `question`（字符串，必需）：投票问题
-   `options`（字符串数组，必需）：选项列表
-   `maxSelections`（数字，可选）：最多可选数量
-   `durationHours`（数字，可选）：投票持续时长（小时）
-   `durationSeconds`（数字，可选，仅 Telegram）：投票持续时长（秒）
-   `isAnonymous`（布尔值，可选，仅 Telegram）：是否匿名
-   `channel`（字符串，可选，默认 `whatsapp`）：指定渠道
-   `idempotencyKey`（字符串，必需）：幂等键

## 各渠道差异须知

不同平台对投票的支持各有特点，使用时请注意：

-   **Telegram**：支持 2-10 个选项。可通过 `threadId` 或 `:topic:` 目标发送到论坛主题。时长参数使用 `durationSeconds`（5-600 秒），支持匿名和公开两种投票模式。
-   **WhatsApp**：支持 2-12 个选项，`maxSelections` 不能超过选项总数，不支持时长设置（`durationHours` 会被忽略）。
-   **Discord**：支持 2-10 个选项，`durationHours` 范围为 1-768 小时（默认 24）。当 `maxSelections > 1` 时启用多选，但 Discord 不支持"恰好选 N 项"的严格限制。
-   **MS Teams**：投票以自适应卡片形式呈现，由 OpenClaw 管理（无原生投票 API），`durationHours` 会被忽略。

## 智能体（agent）工具

使用 `message` 工具并设置 `action: "poll"` 即可创建投票。相关字段包括 `to`、`pollQuestion`、`pollOption`，以及可选的 `pollMulti`、`pollDurationHours`、`channel`。

Telegram 额外支持 `pollDurationSeconds`、`pollAnonymous`、`pollPublic` 参数。

:::caution
不要在 `action: "send"` 时传递投票字段，会被拒绝。必须使用 `action: "poll"`。
:::

**注意事项**：

-   Discord 没有"恰好选择 N 个"的模式，`pollMulti` 仅表示启用多选
-   Teams 投票需要网关保持在线才能记录投票，数据保存在 `~/.openclaw/msteams-polls.json`

[Gmail PubSub](./gmail-pubsub.md)[认证监控](./auth-monitoring.md)