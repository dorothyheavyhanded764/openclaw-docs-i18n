

  智能体协调

  
# ACP 智能体（agent）

[Agent Client Protocol (ACP)](https://agentclientprotocol.com/) 会话让 OpenClaw 能够通过 ACP 后端插件运行外部编码工具（如 Pi、Claude Code、Codex、OpenCode 和 Gemini CLI）。当你用自然语言让 OpenClaw "在 Codex 中运行这个"或"在话题中启动 Claude Code"时，OpenClaw 会把请求路由到 ACP 运行时，而不是原生的子智能体运行时。

## 快速操作手册

这是一个实用的 `/acp` 操作速查流程：

1. 创建会话：
   - `/acp spawn codex --mode persistent --thread auto`
2. 在绑定的话题中工作（或直接指定会话键）。
3. 查看运行时状态：
   - `/acp status`
4. 按需调整运行时选项：
   - `/acp model <provider/model>`
   - `/acp permissions `
   - `/acp timeout `
5. 在不替换上下文的情况下引导正在运行的会话：
   - `/acp steer tighten logging and continue`
6. 结束工作：
   - `/acp cancel`（停止当前轮次），或
   - `/acp close`（关闭会话并移除绑定）

## 用户快速入门

想象一下，你可以直接用自然语言来控制外部编码工具：

- "在这个话题里启动一个持久的 Codex 会话，让它专注处理这个任务。"
- "用一次性 Claude Code ACP 会话运行这个，然后总结结果。"
- "在这个话题里用 Gemini CLI 处理这个任务，后续跟进也保持在同一个话题。"

OpenClaw 会帮你完成这些步骤：

1. 选择 `runtime: "acp"`。
2. 解析你要用的编码工具（`agentId`，比如 `codex`）。
3. 如果需要话题绑定且当前频道支持，就把 ACP 会话绑定到该话题。
4. 后续的话题消息会自动路由到同一个 ACP 会话，直到你取消焦点、关闭或会话过期。

## ACP 与子智能体的区别

简单来说：想要外部编码工具的运行时，用 ACP；想要 OpenClaw 原生的委托运行，用子智能体。

| 方面 | ACP 会话 | 子智能体运行 |
| --- | --- | --- |
| 运行时 | ACP 后端插件（如 acpx） | OpenClaw 原生子智能体运行时 |
| 会话键 | `agent::acp:` | `agent::subagent:` |
| 主要命令 | `/acp ...` | `/subagents ...` |
| 创建工具 | `sessions_spawn` 并设置 `runtime:"acp"` | `sessions_spawn`（默认运行时） |

相关内容请参阅[子智能体](./subagents.md)。

## 话题绑定会话（跨频道通用）

当频道适配器启用了话题绑定功能，ACP 会话就可以绑定到特定话题：

- OpenClaw 把话题绑定到目标 ACP 会话。
- 该话题中的后续消息会路由到绑定的 ACP 会话。
- ACP 的输出会传回同一个话题。
- 取消焦点、关闭、归档、空闲超时或达到最大存活时间都会移除绑定。

话题绑定的支持情况取决于具体的适配器。如果当前频道适配器不支持话题绑定，OpenClaw 会返回清晰的"不支持/不可用"提示。启用话题绑定 ACP 需要这些功能标志：

- `acp.enabled=true`
- `acp.dispatch.enabled` 默认开启（设为 `false` 可暂停 ACP 分发）
- 频道适配器需要启用 ACP 话题创建标志（各适配器配置不同）
  - Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
  - Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

### 支持话题绑定的频道

- 任何具备会话/话题绑定能力的频道适配器。
- 目前内置支持的有：
  - Discord 话题/频道
  - Telegram 话题（群组/超级群组的论坛话题，以及私信话题）
- 插件频道可以通过相同的绑定接口添加支持。

## 频道特定配置

对于需要长期运行的工作流，你可以在顶层 `bindings[]` 条目中配置持久的 ACP 绑定。

### 绑定模型

`bindings` 配置项用来定义持久的 ACP 对话绑定：

- `bindings[].type="acp"` —— 标记这是一个持久的 ACP 对话绑定。
- `bindings[].match` —— 指定目标对话：
  - Discord 频道或话题：`match.channel="discord"` + `match.peer.id=""`
  - Telegram 论坛话题：`match.channel="telegram"` + `match.peer.id=":topic:"`
- `bindings[].agentId` —— 所属的 OpenClaw 智能体 ID。
- `bindings[].acp` 下可以设置可选的 ACP 覆盖配置：
  - `mode`（`persistent` 或 `oneshot`）
  - `label`
  - `cwd`
  - `backend`

### 每个智能体的运行时默认值

使用 `agents.list[].runtime` 可以为每个智能体统一设置 ACP 默认值：

- `agents.list[].runtime.type="acp"`
- `agents.list[].runtime.acp.agent`（编码工具 ID，如 `codex` 或 `claude`）
- `agents.list[].runtime.acp.backend`
- `agents.list[].runtime.acp.mode`
- `agents.list[].runtime.acp.cwd`

ACP 绑定会话的配置覆盖优先级：

1. `bindings[].acp.*`（最高优先级）
2. `agents.list[].runtime.acp.*`
3. 全局 ACP 默认值（如 `acp.backend`）

配置示例：

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

运行时行为说明：

- OpenClaw 会在使用前确保配置的 ACP 会话存在。
- 该频道或话题中的消息会路由到配置的 ACP 会话。
- 在绑定的对话中，`/new` 和 `/reset` 会在原地重置同一个 ACP 会话键。
- 临时运行时绑定（比如话题聚焦流程创建的）在存在时仍然生效。

## 启动 ACP 会话

### 通过 sessions_spawn

从智能体轮次或工具调用中启动 ACP 会话时，设置 `runtime: "acp"`：

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

注意事项：

- `runtime` 默认是 `subagent`，所以 ACP 会话必须显式设置 `runtime: "acp"`。
- 如果省略 `agentId`，OpenClaw 会使用配置中的 `acp.defaultAgent`。
- `mode: "session"` 需要 `thread: true` 来保持持久的绑定对话。

接口参数详解：

- `task`（必需）：发送给 ACP 会话的初始提示。
- `runtime`（ACP 必需）：必须是 `"acp"`。
- `agentId`（可选）：ACP 目标编码工具 ID。如果未设置，会回退到 `acp.defaultAgent`。
- `thread`（可选，默认 `false`）：请求话题绑定流程（在支持的地方）。
- `mode`（可选）：`run`（一次性）或 `session`（持久）。
  - 默认是 `run`
  - 如果 `thread: true` 且省略 mode，OpenClaw 可能根据运行时路径默认采用持久行为
  - `mode: "session"` 需要 `thread: true`
- `cwd`（可选）：请求的运行时工作目录（由后端/运行时策略验证）。
- `label`（可选）：操作员可见的标签，用于会话/横幅文本。
- `streamTo`（可选）：设置为 `"parent"` 可以把初始 ACP 运行进度摘要作为系统事件流式传回请求者会话。
  - 可用时，响应会包含 `streamLogPath`，指向会话级别的 JSONL 日志（`.acp-stream.jsonl`），你可以跟踪它获取完整的转发历史。

## 沙盒兼容性

ACP 会话目前运行在主机运行时上，而不是 OpenClaw 沙盒内。这带来了一些限制：

- 如果请求者会话是沙盒化的，ACP 创建会被阻止。
  - 错误信息：`Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
- `sessions_spawn` 设置 `runtime: "acp"` 时不支持 `sandbox: "require"`。
  - 错误信息：`sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

如果你需要沙盒强制执行，请使用 `runtime: "subagent"`。

### 通过 /acp 命令

需要更直接的控制时，可以用 `/acp spawn` 命令：

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

主要标志说明：

- `--mode persistent|oneshot`
- `--thread auto|here|off`
- `--cwd <absolute-path>`
- `--label `

详情请参阅[斜杠命令](./slash-commands.md)。

## 会话目标解析

大多数 `/acp` 命令接受可选的会话目标参数（`session-key`、`session-id` 或 `session-label`）。解析顺序如下：

1. 显式指定的目标参数（或 `/acp steer` 的 `--session` 标志）
   - 先尝试匹配会话键
   - 再尝试匹配 UUID 格式的会话 ID
   - 最后尝试匹配标签
2. 当前话题绑定（如果当前对话/话题绑定了 ACP 会话）
3. 当前请求者会话作为回退

如果无法解析到任何目标，OpenClaw 会返回明确的错误提示（`Unable to resolve session target: ...`）。

## 创建会话的话题模式

`/acp spawn` 支持 `--thread auto|here|off` 三种模式：

| 模式 | 行为 |
| --- | --- |
| `auto` | 如果在活动话题中：绑定该话题。如果不在话题中：在支持时创建/绑定子话题。 |
| `here` | 要求当前处于活动话题中；如果不在话题中则失败。 |
| `off` | 不绑定。会话以未绑定状态启动。 |

注意事项：

- 在不支持话题绑定的界面上，默认行为相当于 `off`。
- 话题绑定创建需要频道策略支持：
  - Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
  - Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## ACP 控制命令

完整的命令列表：

- `/acp spawn`
- `/acp cancel`
- `/acp steer`
- `/acp close`
- `/acp status`
- `/acp set-mode`
- `/acp set`
- `/acp cwd`
- `/acp permissions`
- `/acp timeout`
- `/acp model`
- `/acp reset-options`
- `/acp sessions`
- `/acp doctor`
- `/acp install`

`/acp status` 会显示当前有效的运行时选项，以及（如果可用）运行时级别和后端级别的会话标识符。部分控制命令依赖后端能力。如果后端不支持某个控制，OpenClaw 会返回明确的"不支持该控制"错误。

## ACP 命令速查表

| 命令 | 功能 | 示例 |
| --- | --- | --- |
| `/acp spawn` | 创建 ACP 会话；可选话题绑定。 | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | 取消目标会话中正在进行的轮次。 | `/acp cancel agent:codex:acp:` |
| `/acp steer` | 向运行中的会话发送引导指令。 | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | 关闭会话并解除话题绑定。 | `/acp close` |
| `/acp status` | 显示后端、模式、状态、运行时选项、能力。 | `/acp status` |
| `/acp set-mode` | 设置目标会话的运行时模式。 | `/acp set-mode plan` |
| `/acp set` | 通用运行时配置选项写入。 | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | 设置运行时工作目录覆盖。 | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | 设置审批策略配置。 | `/acp permissions strict` |
| `/acp timeout` | 设置运行时超时（秒）。 | `/acp timeout 120` |
| `/acp model` | 设置运行时模型覆盖。 | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | 移除会话的运行时选项覆盖。 | `/acp reset-options` |
| `/acp sessions` | 列出存储中的最近 ACP 会话。 | `/acp sessions` |
| `/acp doctor` | 后端健康状态、能力、可操作的修复建议。 | `/acp doctor` |
| `/acp install` | 打印安装和启用的详细步骤。 | `/acp install` |

## 运行时选项映射

`/acp` 提供了便捷命令和通用设置器。以下是等效操作：

- `/acp model ` 对应运行时配置键 `model`。
- `/acp permissions ` 对应运行时配置键 `approval_policy`。
- `/acp timeout ` 对应运行时配置键 `timeout`。
- `/acp cwd ` 直接更新运行时 cwd 覆盖。
- `/acp set  ` 是通用设置路径。
  - 特殊情况：`key=cwd` 时使用 cwd 覆盖路径。
- `/acp reset-options` 清除目标会话的所有运行时覆盖。

## acpx 编码工具支持（当前）

acpx 当前内置的编码工具别名：

- `pi`
- `claude`
- `codex`
- `opencode`
- `gemini`
- `kimi`

当 OpenClaw 使用 acpx 后端时，`agentId` 优先使用这些值，除非你的 acpx 配置定义了自定义智能体别名。直接使用 acpx CLI 时也可以通过 `--agent ` 指向任意适配器，但这是 acpx CLI 的原生功能，不是 OpenClaw 的常规 `agentId` 路径。

## 必需配置

核心 ACP 基础配置：

```json
{
  acp: {
    enabled: true,
    // 可选。默认为 true；设为 false 可在保留 /acp 控制的同时暂停 ACP 分发
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

话题绑定配置是频道适配器特定的。Discord 示例：

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

如果话题绑定 ACP 创建不工作，请先检查适配器功能标志：

- Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

详情请参阅[配置参考](../gateway/configuration-reference.md)。

## acpx 后端插件设置

安装并启用插件：

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

开发时的本地工作区安装：

```bash
openclaw plugins install ./extensions/acpx
```

然后验证后端健康状态：

```bash
/acp doctor
```

### acpx 命令和版本配置

默认情况下，acpx 插件（发布为 `@openclaw/acpx`）使用插件本地固定的二进制文件：

1. 命令默认为 `extensions/acpx/node_modules/.bin/acpx`。
2. 预期版本默认为扩展固定的版本。
3. 启动时立即将 ACP 后端注册为"未就绪"状态。
4. 后台确保任务验证 `acpx --version`。
5. 如果插件本地二进制文件缺失或版本不匹配，会运行：`npm install --omit=dev --no-save acpx@` 然后重新验证。

你可以在插件配置中覆盖命令/版本：

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

注意事项：

- `command` 接受绝对路径、相对路径或命令名（`acpx`）。
- 相对路径从 OpenClaw 工作区目录开始解析。
- `expectedVersion: "any"` 禁用严格的版本匹配。
- 当 `command` 指向自定义二进制文件/路径时，插件本地的自动安装会被禁用。
- OpenClaw 启动过程在后端健康检查运行时保持非阻塞。

详情请参阅[插件](./plugin.md)。

## 权限配置

ACP 会话以非交互方式运行——没有 TTY 来审批文件写入和 shell 执行的权限提示。acpx 插件提供两个配置键来控制权限处理：

### permissionMode

控制编码工具智能体可以在不提示的情况下执行哪些操作。

| 值 | 行为 |
| --- | --- |
| `approve-all` | 自动批准所有文件写入和 shell 命令。 |
| `approve-reads` | 仅自动批准读取操作；写入和执行需要提示。 |
| `deny-all` | 拒绝所有权限提示。 |

### nonInteractivePermissions

控制当权限提示需要显示但没有交互式 TTY 可用时会发生什么（ACP 会话始终是这种情况）。

| 值 | 行为 |
| --- | --- |
| `fail` | 以 `AcpRuntimeError` 中止会话。**（默认）** |
| `deny` | 静默拒绝权限并继续（优雅降级）。 |

### 配置方法

通过插件配置设置：

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

修改这些值后需要重启网关。

> **重要提示：** OpenClaw 当前默认为 `permissionMode=approve-reads` 和 `nonInteractivePermissions=fail`。在非交互式 ACP 会话中，任何触发权限提示的写入或执行操作都可能失败并报错 `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`。如果你需要限制权限，请将 `nonInteractivePermissions` 设置为 `deny`，这样会话会优雅降级而不是直接崩溃。

## 故障排除

| 症状 | 可能原因 | 解决方法 |
| --- | --- | --- |
| `ACP runtime backend is not configured` | 后端插件缺失或未启用。 | 安装并启用后端插件，然后运行 `/acp doctor`。 |
| `ACP is disabled by policy (acp.enabled=false)` | ACP 全局禁用。 | 设置 `acp.enabled=true`。 |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | 从普通话题消息的分发已禁用。 | 设置 `acp.dispatch.enabled=true`。 |
| `ACP agent "" is not allowed by policy` | 智能体不在允许列表中。 | 使用允许的 `agentId` 或更新 `acp.allowedAgents`。 |
| `Unable to resolve session target: ...` | 键/ID/标签令牌有误。 | 运行 `/acp sessions`，复制正确的键/标签后重试。 |
| `--thread here requires running /acp spawn inside an active ... thread` | 在话题上下文外使用了 `--thread here`。 | 移动到目标话题，或使用 `--thread auto`/`off`。 |
| `Only <user-id> can rebind this thread.` | 其他用户拥有话题绑定。 | 作为所有者重新绑定，或使用其他话题。 |
| `Thread bindings are unavailable for .` | 适配器不支持话题绑定。 | 使用 `--thread off` 或切换到支持的适配器/频道。 |
| `Sandboxed sessions cannot spawn ACP sessions ...` | ACP 运行时在主机端；请求者会话是沙盒化的。 | 从沙盒化会话使用 `runtime="subagent"`，或从非沙盒化会话运行 ACP 创建。 |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | 为 ACP 运行时请求了 `sandbox="require"`。 | 需要沙盒化时使用 `runtime="subagent"`，或从非沙盒化会话使用带 `sandbox="inherit"` 的 ACP。 |
| 绑定会话缺少 ACP 元数据 | ACP 会话元数据过期/已删除。 | 用 `/acp spawn` 重新创建，然后重新绑定/聚焦话题。 |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` 在非交互式 ACP 会话中阻止了写入/执行。 | 将 `plugins.entries.acpx.config.permissionMode` 设置为 `approve-all` 并重启网关。请参阅[权限配置](#permission-configuration)。 |
| ACP 会话早期失败且输出很少 | 权限提示被 `permissionMode`/`nonInteractivePermissions` 阻止。 | 检查网关日志中的 `AcpRuntimeError`。完全权限时设置 `permissionMode=approve-all`；优雅降级时设置 `nonInteractivePermissions=deny`。 |
| ACP 会话完成后无限期卡住 | 编码工具进程已完成但 ACP 会话未报告完成。 | 用 `ps aux \| grep acpx` 监控；手动终止残留进程。 |

[子智能体](./subagents.md)[多智能体沙盒与工具](./multi-agent-sandbox-tools.md)