

  技能

  
# 斜杠命令

命令由网关统一处理。大多数命令必须以 `/` 开头，作为一条**独立**的消息发送。仅限主机使用的 bash 聊天命令则使用 `! ` 格式（`/bash ` 是它的别名）。这里涉及两个相关系统：

-   **命令**：独立的 `/...` 消息
-   **指令**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/exec`、`/model`、`/queue`
    -   指令会在消息到达模型之前被剥离处理
    -   在普通聊天消息（非纯指令消息）中，它们被视为"内联提示"，**不会**持久化到会话设置中
    -   在纯指令消息（消息只包含指令）中，它们会持久化到会话并返回确认信息
    -   指令仅对**授权发送者**生效。如果设置了 `commands.allowFrom`，它就是唯一的授权来源；否则授权来自频道允许列表/配对加上 `commands.useAccessGroups`。未授权发送者发出的指令会被当作普通文本处理

此外还有一些**内联快捷命令**（仅限允许列表或授权发送者使用）：`/help`、`/commands`、`/status`、`/whoami`（别名 `/id`）。它们会立即执行，在模型看到消息前就被剥离，剩余文本则继续按正常流程处理。

## 配置选项

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text`（默认 `true`）：启用解析聊天消息中的 `/...` 命令
    -   在不支持原生命令的平台（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams）上，即使设置为 `false`，文本命令仍然有效
-   `commands.native`（默认 `"auto"`）：注册原生命令
    -   自动模式：Discord/Telegram 上启用；Slack 上禁用（直到你手动添加斜杠命令）；不支持原生的平台则忽略此设置
    -   可通过 `channels.discord.commands.native`、`channels.telegram.commands.native` 或 `channels.slack.commands.native` 按平台单独覆盖（布尔值或 `"auto"`）
    -   设为 `false` 时，启动时会清除 Discord/Telegram 上之前注册的命令。Slack 命令在 Slack 应用中管理，不会自动移除
-   `commands.nativeSkills`（默认 `"auto"`）：在支持的平台原生注册**技能**命令
    -   自动模式：Discord/Telegram 上启用；Slack 上禁用（Slack 需要为每个技能单独创建斜杠命令）
    -   可通过 `channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills` 或 `channels.slack.commands.nativeSkills` 按平台单独覆盖（布尔值或 `"auto"`）
-   `commands.bash`（默认 `false`）：启用 `! ` 来运行主机 shell 命令（`/bash ` 是别名；需要配置 `tools.elevated` 允许列表）
-   `commands.bashForegroundMs`（默认 `2000`）：控制 bash 在切换到后台模式前等待的毫秒数（`0` 表示立即后台运行）
-   `commands.config`（默认 `false`）：启用 `/config` 命令（读取/写入 `openclaw.json`）
-   `commands.debug`（默认 `false`）：启用 `/debug` 命令（仅运行时覆盖，不写盘）
-   `commands.allowFrom`（可选）：为命令授权设置按平台的允许列表。配置后，它成为命令和指令的唯一授权来源（频道允许列表/配对和 `commands.useAccessGroups` 将被忽略）。使用 `"*"` 作为全局默认值；特定平台的键会覆盖它
-   `commands.useAccessGroups`（默认 `true`）：当未设置 `commands.allowFrom` 时，对命令强制执行允许列表/策略

## 命令列表

文本命令 + 原生命令（启用时）：

-   `/help`
-   `/commands`
-   `/skill  [input]`（按名称运行技能）
-   `/status`（显示当前状态；包含当前模型提供商的使用量/配额信息，如可用）
-   `/allowlist`（列出/添加/删除允许列表条目）
-   `/approve  allow-once|allow-always|deny`（处理 exec 审批提示）
-   `/context [list|detail|json]`（解释"上下文"；`detail` 显示各文件、工具、技能和系统提示的大小）
-   `/export-session [path]`（别名：`/export`）（将当前会话导出为 HTML，包含完整系统提示）
-   `/whoami`（显示你的发送者 ID；别名：`/id`）
-   `/session idle <duration|off>`（管理已绑定线程的空闲自动取消聚焦）
-   `/session max-age <duration|off>`（管理已绑定线程的最大存活时间自动取消聚焦）
-   `/subagents list|kill|log|info|send|steer|spawn`（检查、控制或生成当前会话的子智能体运行）
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions`（检查和控制 ACP 运行时会话）
-   `/agents`（列出此会话的线程绑定智能体）
-   `/focus `（Discord 专用：将当前线程或新线程绑定到会话/子智能体目标）
-   `/unfocus`（Discord 专用：移除当前线程绑定）
-   `/kill <id|#|all>`（立即中止当前会话的一个或所有正在运行的子智能体；无确认消息）
-   `/steer <id|#> `（立即引导正在运行的子智能体：尽可能在运行中引导，否则中止当前工作并根据引导消息重启）
-   `/tell <id|#> `（`/steer` 的别名）
-   `/config show|get|set|unset`（将配置持久化到磁盘；仅限所有者；需要 `commands.config: true`）
-   `/debug show|set|unset|reset`（运行时配置覆盖；仅限所有者；需要 `commands.debug: true`）
-   `/usage off|tokens|full|cost`（控制每次回复的使用情况页脚，或显示本地成本摘要）
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（控制 TTS；详见 [/tts](../tts.md)）
    -   Discord：原生命令为 `/voice`（Discord 保留了 `/tts`）；文本命令 `/tts` 仍可用
-   `/stop`
-   `/restart`
-   `/dock-telegram`（别名：`/dock_telegram`）（将回复切换到 Telegram）
-   `/dock-discord`（别名：`/dock_discord`）（将回复切换到 Discord）
-   `/dock-slack`（别名：`/dock_slack`）（将回复切换到 Slack）
-   `/activation mention|always`（仅限群组）
-   `/send on|off|inherit`（仅限所有者）
-   `/reset` 或 `/new [model]`（可选的模型提示；其余内容会被传递）
-   `/think <off|minimal|low|medium|high|xhigh>`（选项根据模型/提供商动态生成；别名：`/thinking`、`/t`）
-   `/verbose on|full|off`（别名：`/v`）
-   `/reasoning on|off|stream`（别名：`/reason`；启用时发送单独的消息，前缀为 `Reasoning:`；`stream` 仅限 Telegram 草稿）
-   `/elevated on|off|ask|full`（别名：`/elev`；`full` 跳过 exec 审批）
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=`（发送 `/exec` 查看当前设置）
-   `/model `（别名：`/models`；或使用 `agents.defaults.models.*.alias` 中定义的 `/`）
-   `/queue `（支持选项如 `debounce:2s cap:25 drop:summarize`；发送 `/queue` 查看当前设置）
-   `/bash `（仅限主机；`! ` 的别名；需要 `commands.bash: true` + `tools.elevated` 允许列表）

仅文本命令：

-   `/compact [instructions]`（详见 [/concepts/compaction](../concepts/compaction.md)）
-   `! `（仅限主机；一次只能运行一个；长时间任务使用 `!poll` + `!stop`）
-   `!poll`（检查输出/状态；接受可选的 `sessionId`；`/bash poll` 也有效）
-   `!stop`（停止正在运行的 bash 任务；接受可选的 `sessionId`；`/bash stop` 也有效）

注意事项：

-   命令支持在命令和参数之间使用可选的 `:`（例如 `/think: high`、`/send: on`、`/help:`）
-   `/new ` 接受模型别名、`provider/model` 格式或提供商名称（模糊匹配）；如果无匹配，文本会被当作消息正文处理
-   要获取完整的提供商使用量明细，请使用 `openclaw status --usage`
-   `/allowlist add|remove` 需要 `commands.config=true` 并遵循频道的 `configWrites` 设置
-   `/usage` 控制每次回复的使用情况页脚；`/usage cost` 从 OpenClaw 会话日志中打印本地成本摘要
-   `/restart` 默认启用；设置 `commands.restart: false` 可禁用
-   Discord 专属原生命令：`/vc join|leave|status` 控制语音频道（需要 `channels.discord.voice` 和原生命令；不支持文本形式）
-   Discord 线程绑定命令（`/focus`、`/unfocus`、`/agents`、`/session idle`、`/session max-age`）需要启用有效的线程绑定功能（`session.threadBindings.enabled` 和/或 `channels.discord.threadBindings.enabled`）
-   ACP 命令参考和运行时行为：详见 [ACP 智能体](./acp-agents.md)
-   `/verbose` 用于调试和额外可见性；正常使用时请保持**关闭**
-   工具失败摘要在相关时仍会显示，但详细的失败文本仅在 `/verbose` 为 `on` 或 `full` 时包含
-   `/reasoning`（和 `/verbose`）在群组环境中存在风险：可能会泄露你不打算公开的内部推理或工具输出。建议保持关闭，尤其在群聊中
-   **快速路径**：来自允许列表发送者的纯命令消息会立即处理（绕过队列和模型）
-   **群组提及门控**：来自允许列表发送者的纯命令消息绕过提及要求
-   **内联快捷命令（仅限允许列表发送者）**：某些命令嵌入在普通消息中也能工作，会在模型看到剩余文本前被剥离
    -   示例：`hey /status` 会触发状态回复，剩余文本继续按正常流程处理
-   当前支持：`/help`、`/commands`、`/status`、`/whoami`（`/id`）
-   未授权的纯命令消息会被静默忽略，内联的 `/...` 标记会被当作普通文本
-   **技能命令**：`user-invocable` 技能会以斜杠命令形式暴露。名称会被清理为 `a-z0-9_`（最多 32 个字符）；冲突时添加数字后缀（如 `_2`）
    -   `/skill  [input]` 按名称运行技能（当原生命令限制阻止为每个技能创建独立命令时很有用）
    -   默认情况下，技能命令会作为普通请求转发给模型
    -   技能可以声明 `command-dispatch: tool` 将命令直接路由到工具（确定性执行，无需模型）
    -   示例：`/prose`（OpenProse 插件）—— 详见 [OpenProse](../prose.md)
-   **原生命令参数**：Discord 使用自动补全提供动态选项（省略必需参数时会显示按钮菜单）。Telegram 和 Slack 在命令支持选择且省略参数时会显示按钮菜单

## 使用情况展示（各平台显示什么）

-   **提供商使用量/配额**（例如："Claude 剩余 80%"）在启用使用量跟踪时，会显示在 `/status` 命令的输出中
-   **每次回复的令牌数/成本** 由 `/usage off|tokens|full` 控制（附加在正常回复后）
-   `/model status` 显示的是**模型/认证/端点**信息，而非使用量

## 模型选择 (/model)

`/model` 作为指令实现。示例：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

注意事项：

-   `/model` 和 `/model list` 显示紧凑的编号选择器（包含模型系列和可用提供商）
-   在 Discord 上，`/model` 和 `/models` 打开交互式选择器，包含提供商和模型下拉菜单，以及一个提交步骤
-   `/model <#>` 从选择器中选择（并尽可能优先使用当前提供商）
-   `/model status` 显示详细视图，包括配置的提供商端点（`baseUrl`）和 API 模式（`api`），如可用

## 调试覆盖

`/debug` 用于设置**仅运行时**的配置覆盖（存在内存中，不写磁盘）。仅限所有者。默认禁用；通过 `commands.debug: true` 启用。示例：

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注意事项：

-   覆盖会立即应用于新的配置读取，但**不会**写入 `openclaw.json`
-   使用 `/debug reset` 清除所有覆盖并返回磁盘配置

## 配置更新

`/config` 用于写入磁盘配置（`openclaw.json`）。仅限所有者。默认禁用；通过 `commands.config: true` 启用。示例：

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

注意事项：

-   配置在写入前会进行验证；无效的更改会被拒绝
-   `/config` 的更新会在重启后保留

## 平台注意事项

-   **文本命令** 在正常聊天会话中运行（私信共享 `main` 会话，群组有各自的会话）
-   **原生命令** 使用独立的会话：
    -   Discord：`agent::discord:slash:`
    -   Slack：`agent::slack:slash:`（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）
    -   Telegram：`telegram:slash:`（通过 `CommandTargetSessionKey` 定位聊天会话）
-   **`/stop`** 针对活动的聊天会话，以便中止当前运行
-   **Slack：** `channels.slack.slashCommand` 仍支持单个 `/openclaw` 风格的命令。如果你启用 `commands.native`，则必须为每个内置命令创建一个 Slack 斜杠命令（名称与 `/help` 等相同）。Slack 的命令参数菜单以临时 Block Kit 按钮形式呈现
    -   Slack 原生例外：请注册 `/agentstatus`（而非 `/status`），因为 Slack 保留了 `/status`。文本命令 `/status` 在 Slack 消息中仍可用

[创建技能](./creating-skills.md)[技能](./skills.md)