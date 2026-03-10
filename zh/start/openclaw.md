

  指南

  
# 个人助手设置

OpenClaw 是 **Pi** 智能体（agent）的 WhatsApp + Telegram + Discord + iMessage 网关，插件还支持 Mattermost。本指南帮你搭建一个"个人助手"——一个专属的 WhatsApp 号码，让它像你全天候在线的智能体一样工作。

## ⚠️ 安全第一

在开始之前，请理解你将要放权的智能体可以：

-   在你的机器上执行命令（取决于你的 Pi 工具配置）
-   读写你工作空间里的文件
-   通过 WhatsApp/Telegram/Discord/Mattermost（插件）向外发送消息

建议从保守的配置开始：

-   务必设置 `channels.whatsapp.allowFrom`（不要在你的个人 Mac 上开放给全世界）
-   为助手准备一个专用的 WhatsApp 号码
-   心跳目前默认每 30 分钟运行一次。在信任此配置之前，先通过 `agents.defaults.heartbeat.every: "0m"` 禁用它

## 先决条件

-   已安装 OpenClaw 并完成初始设置 — 如未完成，请参阅 [入门指南](./getting-started.md)
-   一个用于助手的独立手机号（SIM/eSIM/预付费卡均可）

## 双手机设置（推荐）

为什么要用两个手机？如果你把个人 WhatsApp 绑定到 OpenClaw，发给你的每条消息都会变成"智能体输入"——这通常不是你想要的。

## 5 分钟快速开始

1.  配对 WhatsApp Web（会显示二维码，用助手手机扫描）：

```bash
openclaw channels login
```

2.  启动网关（保持运行）：

```bash
openclaw gateway --port 18789
```

3.  在 `~/.openclaw/openclaw.json` 中添加最小配置：

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

现在用你已加入白名单的手机向助手号码发送消息。初始设置完成后，我们会自动打开仪表板并打印一个干净的（无令牌）链接。如果提示需要认证，请将 `gateway.auth.token` 中的令牌粘贴到控制台 UI 设置中。之后重新打开：`openclaw dashboard`。

## 为智能体（agent）提供工作空间

OpenClaw 从工作空间目录读取操作指令和"记忆"。默认情况下，OpenClaw 使用 `~/.openclaw/workspace` 作为智能体工作空间，在设置或首次运行智能体时会自动创建它，同时生成初始的 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 文件。`BOOTSTRAP.md` 只在工作空间全新时创建（删除后不会再次出现）。`MEMORY.md` 是可选的（不会自动创建），存在时会在正常会话中加载。子智能体（subagent）会话仅注入 `AGENTS.md` 和 `TOOLS.md`。

小贴士：把这个文件夹当作 OpenClaw 的"记忆"，把它设为 git 仓库（最好是私有的），这样你的 `AGENTS.md` 和记忆文件都有备份。如果安装了 git，全新的工作空间会自动初始化。

```bash
openclaw setup
```

完整的工作空间布局和备份指南：[智能体工作空间](../concepts/agent-workspace.md)。记忆工作流：[记忆](../concepts/memory.md)。

可选：通过 `agents.defaults.workspace` 指定不同的工作空间（支持 `~`）。

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

如果你已经从仓库中提供了自己的工作空间文件，可以完全禁用引导文件的创建：

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## 把它变成"助手"的配置

OpenClaw 默认就提供了不错的助手设置，但你通常需要调整：

-   `SOUL.md` 中的角色/指令
-   思考模式默认值（如需要）
-   心跳（在你信任它之后）

示例：

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 从 0 开始，之后再启用
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## 会话与记忆

会话数据存储位置：

-   会话文件：`~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   会话元数据（令牌使用量、最后路由等）：`~/.openclaw/agents//sessions/sessions.json`（旧版路径：`~/.openclaw/sessions/sessions.json`）

会话管理命令：

-   `/new` 或 `/reset` 为该聊天启动一个新会话（可通过 `resetTriggers` 配置）。如果单独发送，智能体会回复简短问候以确认重置
-   `/compact [指令]` 压缩会话上下文并报告剩余的上下文预算

## 心跳（主动模式）

默认情况下，OpenClaw 每 30 分钟运行一次心跳，提示词为：`如果存在 HEARTBEAT.md 请阅读它（工作空间上下文）。严格遵守它。不要推断或重复先前聊天中的旧任务。如果无需关注任何事项，请回复 HEARTBEAT_OK。` 设置 `agents.defaults.heartbeat.every: "0m"` 可禁用。

关于心跳的行为：

-   如果 `HEARTBEAT.md` 存在但实际上为空（只有空行和 `# 标题` 这样的 Markdown 标题），OpenClaw 会跳过心跳运行以节省 API 调用
-   如果文件缺失，心跳仍会运行，由模型决定做什么
-   如果智能体回复 `HEARTBEAT_OK`（可附带简短填充内容，参见 `agents.defaults.heartbeat.ackMaxChars`），OpenClaw 会抑制该心跳的对外发送
-   默认允许向 DM 风格的 `user:` 目标发送心跳。设置 `agents.defaults.heartbeat.directPolicy: "block"` 可在保持心跳运行的同时，阻止向直接目标发送
-   心跳会运行完整的智能体轮次——间隔越短，消耗的令牌越多

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## 媒体输入与输出

入站附件（图片/音频/文档）可以通过模板传递给你的命令：

-   `{{MediaPath}}`（本地临时文件路径）
-   `{{MediaUrl}}`（伪 URL）
-   `{{Transcript}}`（如启用音频转录）

智能体发送出站附件：在单独一行中包含 `MEDIA:<路径或URL>`（无空格）。示例：

```
这是截图。
MEDIA:https://example.com/screenshot.png
```

OpenClaw 会提取这些内容并将其作为媒体与文本一起发送。

## 运维清单

```bash
openclaw status          # 本地状态（凭证、会话、排队事件）
openclaw status --all    # 完整诊断（只读，可粘贴分享）
openclaw status --deep   # 添加网关健康探测（Telegram + Discord）
openclaw health --json   # 网关健康快照（WebSocket）
```

日志位于 `/tmp/openclaw/` 下（默认：`openclaw-YYYY-MM-DD.log`）。

## 后续步骤

-   WebChat：[WebChat](../web/webchat.md)
-   网关运维：[网关运行手册](../gateway.md)
-   定时任务与唤醒：[Cron 任务](../automation/cron-jobs.md)
-   macOS 菜单栏伴侣：[OpenClaw macOS 应用](../platforms/macos.md)
-   iOS 节点应用：[iOS 应用](../platforms/ios.md)
-   Android 节点应用：[Android 应用](../platforms/android.md)
-   Windows 状态：[Windows (WSL2)](../platforms/windows.md)
-   Linux 状态：[Linux 应用](../platforms/linux.md)
-   安全性：[安全性](../gateway/security.md)

[初始设置：macOS 应用](./onboarding.md)[CLI 参考](./wizard-cli-reference.md)