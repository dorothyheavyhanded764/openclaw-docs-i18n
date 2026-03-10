

  智能体协调

  
# 子智能体（Sub-Agents）

子智能体是从现有智能体运行中派生出的后台运行实例。它们在独立的会话（`agent::subagent:`）中执行，完成后会向请求方的聊天频道**汇报**结果。

## 斜杠命令

使用 `/subagents` 命令可以查看或控制**当前会话**中的子智能体：

-   `/subagents list`
-   `/subagents kill <id|#|all>`
-   `/subagents log <id|#> [limit] [tools]`
-   `/subagents info <id|#>`
-   `/subagents send <id|#> `
-   `/subagents steer <id|#> `
-   `/subagents spawn   [--model ] [--thinking ]`

线程绑定控制：以下命令适用于支持持久线程绑定的频道，详见后文**支持线程的频道**一节。

-   `/focus <subagent-label|session-key|session-id|session-label>`
-   `/unfocus`
-   `/agents`
-   `/session idle <duration|off>`
-   `/session max-age <duration|off>`

`/subagents info` 用于查看运行元数据（状态、时间戳、会话 ID、记录文件路径、清理状态）。

### 派生行为

`/subagents spawn` 会以用户命令的形式启动一个后台子智能体，而非内部转发。运行结束时，它会向请求方的聊天频道发送一条完成通知。

-   派生命令是非阻塞的，会立即返回一个运行 ID。
-   子智能体完成后，会向请求方聊天频道汇报一条摘要或结果消息。
-   手动派生的消息投递具有容错机制：
    -   OpenClaw 首先尝试使用稳定的幂等键进行直接 `agent` 投递。
    -   若直接投递失败，则回退到队列路由。
    -   若队列路由仍不可用，会在最终放弃前以短间隔指数退避重试。
-   向请求方会话传递完成信息时，使用的是运行时生成的内部上下文（而非用户撰写的文本），包括：
    -   `Result`：`assistant` 回复文本，若助手回复为空则取最新的 `toolResult`
    -   `Status`：`completed successfully` / `failed` / `timed out` / `unknown`
    -   精简的运行时和 Token 统计
    -   一条投递指令，告知请求方智能体以正常助手语气重写内容（而非直接转发原始内部元数据）
-   `--model` 和 `--thinking` 可覆盖该次运行的默认值。
-   完成后可使用 `info`/`log` 查看详情和输出。
-   `/subagents spawn` 是一次性模式（`mode: "run"`）。如需持久线程绑定会话，请使用带 `thread: true` 和 `mode: "session"` 的 `sessions_spawn`。
-   对于 ACP 工具会话（Codex、Claude Code、Gemini CLI），请使用带 `runtime: "acp"` 的 `sessions_spawn`，详见 [ACP 智能体](./acp-agents.md)。

核心设计目标：

-   并行处理「研究/长任务/慢工具」类工作，不阻塞主运行。
-   默认保持子智能体相互隔离（会话分离 + 可选沙箱化）。
-   工具接口设计安全防滥用：子智能体默认**不**获取会话类工具。
-   支持可配置的嵌套深度，适用于编排器模式。

成本提示：每个子智能体有**独立的**上下文和 Token 用量。对于繁重或重复性任务，建议给子智能体配置较便宜的模型，主智能体则使用更高质量的模型。可通过 `agents.defaults.subagents.model` 或逐个智能体覆盖配置。

## 工具

使用 `sessions_spawn` 工具：

-   启动一个子智能体运行（`deliver: false`，全局通道：`subagent`）
-   执行汇报步骤，将汇报回复发布到请求方聊天频道
-   默认模型：继承调用方，除非设置了 `agents.defaults.subagents.model`（或逐智能体的 `agents.list[].subagents.model`）；显式指定的 `sessions_spawn.model` 优先级最高。
-   默认思考级别：继承调用方，除非设置了 `agents.defaults.subagents.thinking`（或逐智能体的 `agents.list[].subagents.thinking`）；显式指定的 `sessions_spawn.thinking` 优先级最高。
-   默认运行超时：若省略 `sessions_spawn.runTimeoutSeconds`，则当 `agents.defaults.subagents.runTimeoutSeconds` 已设置时使用该值，否则回退到 `0`（无超时）。

工具参数：

-   `task`（必需）
-   `label?`（可选）
-   `agentId?`（可选；若允许，可在其他智能体 ID 下派生）
-   `model?`（可选；覆盖子智能体模型；无效值会被跳过，子智能体以默认模型运行并在工具结果中给出警告）
-   `thinking?`（可选；覆盖子智能体运行的思考级别）
-   `runTimeoutSeconds?`（默认为已配置时的 `agents.defaults.subagents.runTimeoutSeconds`，否则为 `0`；设置后，子智能体运行将在 N 秒后中止）
-   `thread?`（默认 `false`；设为 `true` 时，为该子智能体会话请求频道线程绑定）
-   `mode?`（`run|session`）
    -   默认为 `run`
    -   若 `thread: true` 且省略 `mode`，则默认变为 `session`
    -   `mode: "session"` 要求 `thread: true`
-   `cleanup?`（`delete|keep`，默认 `keep`）
-   `sandbox?`（`inherit|require`，默认 `inherit`；`require` 会拒绝向非沙箱化的目标运行时派生）
-   `sessions_spawn` **不**接受频道投递参数（`target`、`channel`、`to`、`threadId`、`replyTo`、`transport`）。如需投递消息，请在派生的运行中使用 `message`/`sessions_send`。

## 线程绑定会话

当频道启用了线程绑定时，子智能体可以持续绑定到某个线程，使该线程中的后续用户消息继续路由到同一个子智能体会话。

### 支持线程的频道

-   Discord（目前唯一支持的频道）：支持持久线程绑定子智能体会话（带 `thread: true` 的 `sessions_spawn`）、手动线程控制（`/focus`、`/unfocus`、`/agents`、`/session idle`、`/session max-age`），以及适配器配置项 `channels.discord.threadBindings.enabled`、`channels.discord.threadBindings.idleHours`、`channels.discord.threadBindings.maxAgeHours` 和 `channels.discord.threadBindings.spawnSubagentSessions`。

基本流程：

1.  使用带 `thread: true`（可选 `mode: "session"`）的 `sessions_spawn` 进行派生。
2.  OpenClaw 在活动频道中为该会话目标创建或绑定一个线程。
3.  该线程内的回复和后续消息将路由到绑定的会话。
4.  使用 `/session idle` 查看/设置不活动自动解绑，使用 `/session max-age` 控制硬性时长上限。
5.  使用 `/unfocus` 手动解绑。

手动控制：

-   `/focus ` 将当前线程（或新建线程）绑定到指定的子智能体/会话目标。
-   `/unfocus` 移除当前已绑定线程的绑定关系。
-   `/agents` 列出活动运行及绑定状态（`thread:` 或 `unbound`）。
-   `/session idle` 和 `/session max-age` 仅对已聚焦的绑定线程有效。

配置开关：

-   全局默认值：`session.threadBindings.enabled`、`session.threadBindings.idleHours`、`session.threadBindings.maxAgeHours`
-   频道覆盖和派生自动绑定的配置项因适配器而异，参见上方**支持线程的频道**。

更多适配器详情请参阅[配置参考](../gateway/configuration-reference.md)和[斜杠命令](./slash-commands.md)。允许列表：

-   `agents.list[].subagents.allowAgents`：可通过 `agentId` 指定的智能体 ID 列表（`["*"]` 表示允许任意）。默认：仅请求方智能体。
-   沙箱继承保护：若请求方会话处于沙箱中，`sessions_spawn` 会拒绝向非沙箱化目标派生。

发现：

-   使用 `agents_list` 可查看当前哪些智能体 ID 允许被 `sessions_spawn` 调用。

自动归档：

-   子智能体会话会在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动归档。
-   归档使用 `sessions.delete` 并将记录文件重命名为 `*.deleted.`（同一目录）。
-   `cleanup: "delete"` 会在汇报后立即归档（仍通过重命名保留记录）。
-   自动归档是尽力而为的；若网关重启，待处理的定时器会丢失。
-   `runTimeoutSeconds` **不会**触发自动归档，它只中止运行。会话会保留至自动归档。
-   自动归档同等适用于深度 1 和深度 2 的会话。

## 嵌套子智能体

默认情况下，子智能体不能再派生自己的子智能体（`maxSpawnDepth: 1`）。设置 `maxSpawnDepth: 2` 可启用一层嵌套，从而支持**编排器模式**：主智能体 → 编排器子智能体 → 工作器子子智能体。

### 启用方法

```json
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // 允许子智能体派生子级（默认：1）
        maxChildrenPerAgent: 5, // 每个智能体会话的最大活动子级数（默认：5）
        maxConcurrent: 8, // 全局并发通道上限（默认：8）
        runTimeoutSeconds: 900, // sessions_spawn 省略时的默认超时（0 = 无超时）
      },
    },
  },
}
```

### 深度层级

| 深度 | 会话键格式 | 角色 | 可否派生 |
| --- | --- | --- | --- |
| 0 | `agent::main` | 主智能体 | 始终可以 |
| 1 | `agent::subagent:` | 子智能体（当允许深度 2 时可作为编排器） | 仅当 `maxSpawnDepth >= 2` |
| 2 | `agent::subagent::subagent:` | 子子智能体（叶工作器） | 永远不可 |

### 汇报链

结果沿链向上传递：

1.  深度 2 工作器完成 → 向其父级（深度 1 编排器）汇报
2.  深度 1 编排器接收汇报、综合结果、完成 → 向主智能体汇报
3.  主智能体接收汇报并交付给用户

每一级只能看到其直接子级的汇报。

### 按深度区分的工具策略

-   **深度 1（编排器，当 `maxSpawnDepth >= 2` 时）**：拥有 `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history`，可管理其子级。其他会话/系统工具仍被拒绝。
-   **深度 1（叶节点，当 `maxSpawnDepth == 1` 时）**：无会话工具（当前默认行为）。
-   **深度 2（叶工作器）**：无会话工具 — `sessions_spawn` 在深度 2 始终被拒绝，无法继续派生子级。

### 单智能体派生上限

每个智能体会话（任意深度）最多可同时拥有 `maxChildrenPerAgent`（默认：5）个活动子级，防止单个编排器失控扩展。

### 级联停止

停止深度 1 编排器会自动停止其所有深度 2 子级：

-   主聊天中发送 `/stop` 会停止所有深度 1 智能体，并级联停止其深度 2 子级。
-   `/subagents kill ` 停止指定子智能体，并级联停止其子级。
-   `/subagents kill all` 停止请求方的所有子智能体，并级联停止。

## 认证

子智能体认证按**智能体 ID** 解析，与会话类型无关：

-   子智能体会话键为 `agent::subagent:`。
-   认证存储从该智能体的 `agentDir` 加载。
-   主智能体的认证配置会作为**后备**合并进来；冲突时智能体配置覆盖主配置。

注意：合并是叠加式的，主配置始终作为后备可用。目前尚不支持完全隔离的智能体级认证。

## 汇报机制

子智能体通过汇报步骤返回结果：

-   汇报步骤在子智能体会话内部执行（而非请求方会话）。
-   若子智能体回复恰好是 `ANNOUNCE_SKIP`，则不发布任何内容。
-   否则，投递方式取决于请求方深度：
    -   顶层请求方会话使用带外部投递（`deliver=true`）的后续 `agent` 调用
    -   嵌套的请求方子智能体会话接收内部后续注入（`deliver=false`），以便编排器可在会话内综合子级结果
    -   若嵌套的请求方子智能体会话已不存在，OpenClaw 会在可用时回退到该会话的请求方
-   构建嵌套完成结果时，子级完成聚合限定在当前请求方运行内，防止过时的前次运行子级输出混入当前汇报。
-   频道适配器支持时，汇报回复会保留线程/主题路由。
-   汇报上下文规范化为稳定的内部事件块：
    -   来源（`subagent` 或 `cron`）
    -   子会话键/ID
    -   汇报类型 + 任务标签
    -   由运行时结果推导的状态行（`success`、`error`、`timeout` 或 `unknown`）
    -   汇报步骤的结果内容（缺失时显示 `(no output)`）
    -   一条后续指令，描述何时应回复、何时应静默
-   `Status` 不是从模型输出推断的，而是来自运行时结果信号。

汇报负载末尾包含一行统计信息（即使被包装也包含）：

-   运行时长（如 `runtime 5m12s`）
-   Token 用量（输入/输出/总计）
-   配置了模型定价时的预估成本（`models.providers.*.models[].cost`）
-   `sessionKey`、`sessionId` 和记录文件路径（以便主智能体通过 `sessions_history` 获取历史或在磁盘上检查文件）
-   内部元数据仅供编排使用；面向用户的回复应以正常助手语气重写。

## 工具策略（子智能体工具）

默认情况下，子智能体拥有**除会话工具和系统工具外的所有工具**：

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

当 `maxSpawnDepth >= 2` 时，深度 1 编排器子智能体额外获得 `sessions_spawn`、`subagents`、`sessions_list` 和 `sessions_history`，以便管理其子级。可通过配置覆盖：

```json
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny 优先
        deny: ["gateway", "cron"],
        // 若设置了 allow，则变为白名单模式（deny 仍优先）
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## 并发

子智能体使用专用的进程内队列通道：

-   通道名称：`subagent`
-   并发数：`agents.defaults.subagents.maxConcurrent`（默认 `8`）

## 停止

-   在请求方聊天中发送 `/stop` 会中止请求方会话，并停止由其派生的所有活动子智能体运行，级联至嵌套子级。
-   `/subagents kill ` 停止指定子智能体，并级联停止其子级。

## 限制

-   子智能体汇报是**尽力而为**的。若网关重启，待处理的「汇报返回」工作会丢失。
-   子智能体仍共享同一网关进程资源；请将 `maxConcurrent` 视为安全阀。
-   `sessions_spawn` 始终是非阻塞的：立即返回 `{ status: "accepted", runId, childSessionKey }`。
-   子智能体上下文仅注入 `AGENTS.md` + `TOOLS.md`（不含 `SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
-   最大嵌套深度为 5（`maxSpawnDepth` 范围：1–5）。推荐大多数场景使用深度 2。
-   `maxChildrenPerAgent` 限制每个会话的活动子级数（默认：5，范围：1–20）。

[智能体发送](./agent-send.md)[ACP 智能体](./acp-agents.md)