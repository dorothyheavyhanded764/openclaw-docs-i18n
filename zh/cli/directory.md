

  CLI 命令

  
# directory

为支持该功能的频道提供目录查找（联系人/peers、群组以及"我"）。

## 常用标志

-   `--channel `：频道 ID/别名（配置多个频道时必填；仅配置一个时自动使用）
-   `--account `：账户 ID（accountId）（默认：频道默认值）
-   `--json`：输出 JSON 格式

## 注意事项

-   `directory` 旨在帮助你找到可以粘贴到其他命令中的 ID（特别是 `openclaw message send --target ...`）。
-   对于许多频道，结果来自配置（允许列表/配置的群组）而非实时的提供商目录。
-   默认输出是 `id`（有时还有 `name`）用制表符分隔；脚本场景请使用 `--json`。

## 将结果用于 message send

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID 格式（按频道）

-   WhatsApp：`+15551234567`（直接消息），`1234567890-1234567890@g.us`（群组）
-   Telegram：`@username` 或数字聊天 ID；群组为数字 ID
-   Slack：`user:U…` 和 `channel:C…`
-   Discord：`user:` 和 `channel:`
-   Matrix（插件）：`user:@user:server`、`room:!roomId:server` 或 `#alias:server`
-   Microsoft Teams（插件）：`user:` 和 `conversation:`
-   Zalo（插件）：用户 ID（Bot API）
-   Zalo Personal / `zalouser`（插件）：来自 `zca` 的线程 ID（`me`、`friend list`、`group list`）（直接消息/群组）

## 自我信息（"me"）

```bash
openclaw directory self --channel zalouser
```

## Peers（联系人/用户）

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## 群组

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)