

  CLI 命令

  
# agents

管理隔离智能体（agent）——每个智能体拥有独立的工作空间（workspace）、认证信息和路由配置。相关主题：

-   多智能体路由：[多智能体路由](../concepts/multi-agent.md)
-   智能体工作空间：[智能体工作空间](../concepts/agent-workspace.md)

## 示例

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## 路由绑定

路由绑定用于将入站渠道消息固定到特定智能体处理。比如，你可以让 Telegram 某个账号的消息由「工作」智能体响应，而其他消息则由默认智能体处理。

查看当前绑定：

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

添加绑定：

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

如果省略 `accountId`（即只写 `--bind `），OpenClaw 会尝试从频道默认配置或插件钩子中自动解析账户。

### 绑定范围行为

绑定时是否指定账户会影响匹配规则：

-   不带 `accountId` 的绑定只匹配该频道的默认账户
-   `accountId: "*"` 是频道级通配符，会匹配该频道下的所有账户，但优先级低于显式指定的账户绑定
-   如果同一智能体已有不带 `accountId` 的频道绑定，之后又绑定了显式或自动解析的 `accountId`，OpenClaw 会原地升级原有绑定，而不是创建重复项

示例：

```bash
# 先绑定整个频道（不带账户）
openclaw agents bind --agent work --bind telegram

# 再升级到指定账户
openclaw agents bind --agent work --bind telegram:ops
```

升级后，该绑定只会匹配 `telegram:ops`。如果你仍需要默认账户也走这条路由，需要显式添加（例如 `--bind telegram:default`）。

移除绑定：

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## 身份文件

每个智能体的工作空间（workspace）根目录可以放置 `IDENTITY.md` 文件来定义身份信息：

-   示例路径：`~/.openclaw/workspace/IDENTITY.md`
-   执行 `set-identity --from-identity` 时，会从工作空间根目录读取（也可用 `--identity-file` 指定其他路径）

头像路径相对于工作空间根目录解析。

## 设置身份

`set-identity` 命令会将以下字段写入 `agents.list[].identity`：

-   `name` —— 名称
-   `theme` —— 主题
-   `emoji` —— 表情符号
-   `avatar` —— 头像（支持工作空间相对路径、http(s) URL 或 data URI）

从 `IDENTITY.md` 加载身份信息：

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

直接在命令行指定字段值：

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

配置示例：

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)