

  CLI 命令

  
# channels

在网关上管理聊天频道账户及其运行时状态。相关文档：

-   频道指南：[频道](../channels/index.md)
-   网关配置：[配置](../gateway/configuration.md)

## 常用命令

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## 添加 / 移除账户

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

**提示**：运行 `openclaw channels add --help` 可查看每个频道的专属参数（如令牌、应用令牌、signal-cli 路径等）。如果直接运行 `openclaw channels add` 而不带任何参数，交互式向导会引导你完成：

-   选择要配置的频道，并填写账户 ID
-   为这些账户设置可选的显示名称
-   询问「是否立即将已配置的频道账户绑定到智能体（agent）？」

如果你选择立即绑定，向导会让你为每个频道账户指定所属的智能体，并自动写入账户级别的路由绑定规则。之后你也可以通过 `openclaw agents bindings`、`openclaw agents bind` 和 `openclaw agents unbind` 来管理这些规则（详见 [agents](./agents.md)）。

**多账户迁移**：当你向一个仍在使用单账户顶层配置（即还没有 `channels..accounts` 条目）的频道添加非默认账户时，OpenClaw 会自动将单账户的顶层配置迁移到 `channels..accounts.default`，然后再写入新账户。这样可以平滑过渡到多账户结构，原有行为不受影响。迁移后的路由行为保持一致：

-   已有的仅频道级别绑定（不含 `accountId`）仍会匹配默认账户
-   在非交互模式下，`channels add` 不会自动创建或改写绑定规则
-   交互式设置可选择是否添加账户级别的绑定

如果你的配置已经处于混合状态（存在命名账户但缺少 `default`，且顶层仍保留了单账户配置），请运行 `openclaw doctor --fix` 来自动修复。

## 登录 / 登出（交互式）

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## 故障排除

-   运行 `openclaw status --deep` 进行全面诊断
-   使用 `openclaw doctor` 获取引导式修复建议
-   `openclaw channels list` 显示 `Claude: HTTP 403 ... user:profile` 错误？这说明使用情况快照需要 `user:profile` 权限。你可以使用 `--no-usage` 跳过，或提供 claude.ai 会话密钥（`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`），也可以通过 Claude Code CLI 重新授权
-   当网关不可达时，`openclaw channels status` 会回退为仅显示配置摘要。如果某频道凭据通过 SecretRef 配置但在当前命令路径中不可用，该账户会被标记为「已配置（降级）」而非「未配置」

## 能力探测

查询服务商的能力信息（如 intents/scopes）以及静态功能支持：

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

**注意**：

-   `--channel` 参数可选，省略时会列出所有频道（包括扩展频道）
-   `--target` 接受 `channel:` 格式或原始数字频道 ID，仅适用于 Discord
-   探测内容因服务商而异：Discord 会查询 intents 及可选的频道权限；Slack 会查询 bot 和用户的 scopes；Telegram 会查询 bot 标志和 webhook；Signal 会查询守护进程版本；MS Teams 会查询应用令牌和 Graph 角色/scopes（已知部分会标注）。不支持探测的频道会显示 `Probe: unavailable`

## 名称解析

将频道名称或用户名解析为 ID：

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

**注意**：

-   使用 `--kind user|group|auto` 强制指定目标类型
-   当存在多个同名条目时，解析会优先匹配活跃项
-   `channels resolve` 是只读操作。如果所选账户通过 SecretRef 配置但凭据在当前命令路径中不可用，命令会返回带说明的降级结果，而非直接报错退出

[browser](./browser.md)[clawbot](./clawbot.md)