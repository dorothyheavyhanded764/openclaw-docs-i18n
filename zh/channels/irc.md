

  消息平台

  
# IRC

想让 OpenClaw 进入经典的 IRC 频道（`#room`）和私聊对话？IRC 就是你要的功能。它以扩展插件的形式提供，但所有配置都在主配置文件的 `channels.irc` 中完成。

## 快速开始

只需三步，让机器人跑起来：

1.  在 `~/.openclaw/openclaw.json` 中启用 IRC 配置。
2.  填入基本连接信息：

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  启动或重启网关：

```bash
openclaw gateway run
```

## 安全默认值

OpenClaw 的 IRC 频道（channel）默认采用较严格的安全策略：

-   `channels.irc.dmPolicy` 默认为 `"pairing"`（需要配对才能私聊）
-   `channels.irc.groupPolicy` 默认为 `"allowlist"`（只响应已配置的频道）
-   当 `groupPolicy="allowlist"` 时，需要通过 `channels.irc.groups` 定义允许的频道
-   建议启用 TLS（`channels.irc.tls=true`），除非你明确知道自己在做什么

## 访问控制

IRC 频道有两道独立的"关卡"：

1.  **频道准入**（`groupPolicy` + `groups`）：机器人是否接受来自某个频道的消息
2.  **发送者权限**（`groupAllowFrom` 或单个频道的 `groups["#channel"].allowFrom`）：频道里谁有资格触发机器人

相关配置项：

-   私聊白名单：`channels.irc.allowFrom`
-   频道发送者白名单：`channels.irc.groupAllowFrom`
-   单频道精细控制（含频道准入、发送者权限、提及规则）：`channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` 允许未配置的频道进入，但**默认仍需提及才会响应**

白名单条目建议使用稳定的发送者身份标识（`nick!user@host`）。仅用昵称匹配是不可靠的，只有设置 `channels.irc.dangerouslyAllowNameMatching: true` 才会启用。

### 常见问题：allowFrom 管的是私聊，不是频道

如果你在日志里看到：

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

这说明发送者没有**群组/频道**消息的权限。解决方法：

-   设置 `channels.irc.groupAllowFrom`（对所有频道生效），或
-   为特定频道设置发送者白名单：`channels.irc.groups["#channel"].allowFrom`

示例：允许 `#tuirc-dev` 频道里的任何人跟机器人对话：

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 回复触发（提及机制）

即使频道已加入白名单、发送者也有权限，OpenClaw 在群聊中默认还是**要求提及**才会响应。所以你可能会看到 `drop channel … (missing-mention)` 这样的日志——意思是消息里没有 @机器人。

如果你希望机器人在频道里"有问必答"、无需提及，可以关闭该频道的提及门控：

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

或者，让机器人加入**所有** IRC 频道，并且都不需要提及：

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## 安全建议（公共频道必读）

如果你在公共频道里设置 `allowFrom: ["*"]`，意味着任何人都能触发机器人。为了降低风险，建议限制该频道可用的工具。

### 频道内所有人使用相同工具

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 不同发送者使用不同工具（管理员权限更大）

用 `toolsBySender` 给普通用户设置严格限制，给自己开更多权限：

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

注意事项：

-   `toolsBySender` 的键需要用 `id:` 前缀指定 IRC 发送者身份：`id:eigen` 或更完整的 `id:eigen!~eigen@174.127.248.171`
-   旧版无前缀的键仍然支持，会被当作 `id:` 处理
-   按顺序匹配，第一个匹配的策略生效；`"*"` 是兜底通配符

想深入了解频道访问和提及门控的关系？参见 [/channels/groups](./groups.md)。

## NickServ

如果需要连接后向 NickServ 验证身份：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

首次连接时自动注册昵称（可选）：

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

昵称注册成功后，记得把 `register` 关掉，避免重复尝试注册。

## 环境变量

默认账户支持以下环境变量：

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS`（逗号分隔）
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## 故障排除

-   **机器人连上了但不回消息**：检查 `channels.irc.groups` 是否正确配置，同时确认消息是否因 `missing-mention` 被丢弃。如果想让机器人无需 @ 就能回复，设置 `requireMention: false`
-   **登录失败**：检查昵称是否已被占用，服务器密码是否正确
-   **TLS 连接失败**：检查主机和端口是否正确，证书配置是否有问题

|[iMessage](./imessage.md)|[LINE](./line.md)|