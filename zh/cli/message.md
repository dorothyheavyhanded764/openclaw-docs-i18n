

  CLI 命令

  
# message

`message` 命令是你与各聊天平台交互的核心工具——无论是发送消息、发起投票、添加表情反应，还是管理频道内容，一个命令全搞定。支持 Discord、Google Chat、Slack、Mattermost（需插件）、Telegram、WhatsApp、Signal、iMessage 和 MS Teams。

## 用法

```bash
openclaw message <子命令> [标志]
```

### 频道选择

配置了多个频道时，必须用 `--channel` 指定使用哪个。如果只配置了一个频道，系统会自动把它作为默认值。

可选值：`whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`（Mattermost 需要插件）

### 目标格式（`--target`）

不同平台的消息目标格式各有差异：

- **WhatsApp**：E.164 格式号码或群组 JID
- **Telegram**：聊天 ID 或 `@用户名`
- **Discord**：`channel:` 或 `user:`（也支持 `<@id>` 提及格式；纯数字 ID 会被当作频道处理）
- **Google Chat**：`spaces/` 或 `users/`
- **Slack**：`channel:` 或 `user:`（也接受原始频道 ID）
- **Mattermost（插件）**：`channel:`、`user:` 或 `@用户名`（裸 ID 会被当作频道处理）
- **Signal**：`+E.164`、`group:`、`signal:+E.164`、`signal:group:` 或 `username:`/`u:`
- **iMessage**：句柄、`chat_id:`、`chat_guid:` 或 `chat_identifier:`
- **MS Teams**：会话 ID（`19:...@thread.tacv2`）或 `conversation:` 或 `user:<aad-object-id>`

### 名称查找

对于 Discord、Slack 等支持的提供商，你可以直接使用频道名称（如 `Help` 或 `#help`），系统会通过目录缓存自动解析。如果缓存里没找到，OpenClaw 会尝试实时查询（前提是提供商支持）。

## 通用标志

以下标志在多数子命令中都会用到：

- `--channel <名称>` — 指定频道
- `--account ` — 指定账号
- `--target <目标>` — 发送/投票/读取等操作的目标频道或用户
- `--targets <名称>` — 广播目标（可重复指定）
- `--json` — JSON 格式输出
- `--dry-run` — 模拟运行，不实际执行
- `--verbose` — 详细输出

## 操作

### 核心操作

#### `send` — 发送消息

**支持频道**：WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost（插件）/Signal/iMessage/MS Teams

**必需参数**：`--target`，以及 `--message` 或 `--media` 二选一

**可选参数**：`--media`、`--reply-to`、`--thread-id`、`--gif-playback`

**平台专属**：
- Telegram：`--buttons`（需启用 `channels.telegram.capabilities.inlineButtons`）、`--thread-id`（论坛主题 ID）
- Slack：`--thread-id`（线程时间戳；`--reply-to` 使用相同字段）
- WhatsApp：`--gif-playback`

#### `poll` — 发起投票

**支持频道**：WhatsApp/Telegram/Discord/Matrix/MS Teams

**必需参数**：`--target`、`--poll-question`、`--poll-option`（可重复指定多个选项）

**可选参数**：`--poll-multi`

**平台专属**：
- Discord：`--poll-duration-hours`、`--silent`、`--message`
- Telegram：`--poll-duration-seconds`（5-600）、`--silent`、`--poll-anonymous` / `--poll-public`、`--thread-id`

#### `react` — 添加/移除表情反应

**支持频道**：Discord/Google Chat/Slack/Telegram/WhatsApp/Signal

**必需参数**：`--message-id`、`--target`

**可选参数**：`--emoji`、`--remove`、`--participant`、`--from-me`、`--target-author`、`--target-author-uuid`

**注意事项**：
- 移除反应时，`--remove` 需配合 `--emoji` 使用；省略 `--emoji` 可清除自己的所有反应（支持的平台）
- WhatsApp：`--participant`、`--from-me`
- Signal 群组：必须指定 `--target-author` 或 `--target-author-uuid`

详见 [表情反应](../tools/reactions.md)。

#### `reactions` — 查看消息反应

**支持频道**：Discord/Google Chat/Slack

**必需参数**：`--message-id`、`--target`

**可选参数**：`--limit`

#### `read` — 读取消息

**支持频道**：Discord/Slack

**必需参数**：`--target`

**可选参数**：`--limit`、`--before`、`--after`

**Discord 专属**：`--around`

#### `edit` — 编辑消息

**支持频道**：Discord/Slack

**必需参数**：`--message-id`、`--message`、`--target`

#### `delete` — 删除消息

**支持频道**：Discord/Slack/Telegram

**必需参数**：`--message-id`、`--target`

#### `pin` / `unpin` — 置顶/取消置顶

**支持频道**：Discord/Slack

**必需参数**：`--message-id`、`--target`

#### `pins` — 查看置顶消息列表

**支持频道**：Discord/Slack

**必需参数**：`--target`

#### `permissions` — 查看权限

**支持频道**：Discord

**必需参数**：`--target`

#### `search` — 搜索消息

**支持频道**：Discord

**必需参数**：`--guild-id`、`--query`

**可选参数**：`--channel-id`、`--channel-ids`（可重复）、`--author-id`、`--author-ids`（可重复）、`--limit`

### 线程操作

#### `thread create` — 创建线程

**支持频道**：Discord

**必需参数**：`--thread-name`、`--target`（频道 ID）

**可选参数**：`--message-id`、`--message`、`--auto-archive-min`

#### `thread list` — 列出线程

**支持频道**：Discord

**必需参数**：`--guild-id`

**可选参数**：`--channel-id`、`--include-archived`、`--before`、`--limit`

#### `thread reply` — 线程回复

**支持频道**：Discord

**必需参数**：`--target`（线程 ID）、`--message`

**可选参数**：`--media`、`--reply-to`

### 表情符号

#### `emoji list` — 列出表情

- Discord：需要 `--guild-id`
- Slack：无需额外标志

#### `emoji upload` — 上传表情

**支持频道**：Discord

**必需参数**：`--guild-id`、`--emoji-name`、`--media`

**可选参数**：`--role-ids`（可重复）

### 贴纸

#### `sticker send` — 发送贴纸

**支持频道**：Discord

**必需参数**：`--target`、`--sticker-id`（可重复）

**可选参数**：`--message`

#### `sticker upload` — 上传贴纸

**支持频道**：Discord

**必需参数**：`--guild-id`、`--sticker-name`、`--sticker-desc`、`--sticker-tags`、`--media`

### 角色 / 频道 / 成员 / 语音

- `role info`（Discord）：`--guild-id`
- `role add` / `role remove`（Discord）：`--guild-id`、`--user-id`、`--role-id`
- `channel info`（Discord）：`--target`
- `channel list`（Discord）：`--guild-id`
- `member info`（Discord/Slack）：`--user-id`（Discord 还需 `--guild-id`）
- `voice status`（Discord）：`--guild-id`、`--user-id`

### 活动

- `event list`（Discord）：`--guild-id`
- `event create`（Discord）：`--guild-id`、`--event-name`、`--start-time`
  - 可选：`--end-time`、`--desc`、`--channel-id`、`--location`、`--event-type`

### 管理（Discord）

- `timeout`：`--guild-id`、`--user-id`（可选 `--duration-min` 或 `--until`；两者都省略则清除超时）
- `kick`：`--guild-id`、`--user-id`（可加 `--reason`）
- `ban`：`--guild-id`、`--user-id`（可加 `--delete-days`、`--reason`）
- `timeout` 也支持 `--reason`

### 广播

#### `broadcast` — 批量广播

**支持频道**：任意已配置频道；使用 `--channel all` 可同时向所有提供商发送

**必需参数**：`--targets`（可重复指定多个目标）

**可选参数**：`--message`、`--media`、`--dry-run`

## 示例

### 发送 Discord 回复

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

### 发送带交互组件的 Discord 消息

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

完整模式请参见 [Discord 组件](../channels/discord.md#interactive-components)。

### 创建 Discord 投票

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

### 创建 Telegram 投票（2 分钟后自动关闭）

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

### 发送 Teams 主动消息

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

### 创建 Teams 投票

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

### 在 Slack 中添加表情反应

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

### 在 Signal 群组中添加表情反应

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

### 发送 Telegram 内联按钮

```bash
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)