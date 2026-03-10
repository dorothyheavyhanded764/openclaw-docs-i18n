

  内置工具

  
# Lobster

Lobster 是一个工作流 Shell，让 OpenClaw 能够以单次确定性操作运行多步骤工具序列，并在关键节点设置明确的审批检查点。

## 引言

你的助手可以构建管理自身的工具——只需描述你想要的工作流，30 分钟后你就拥有了一个 CLI 和可单次调用运行的管道。Lobster 就是那个缺失的拼图：确定性管道、明确审批、可恢复状态。

## 为什么需要 Lobster

当前，复杂工作流需要多次往返的工具调用。每次调用都消耗 Token，LLM 还要负责编排每个步骤。Lobster 把这种编排移入一个类型化的运行时：

- **一次调用代替多次往返**：OpenClaw 只需调用一次 Lobster 工具，就能获得结构化结果
- **审批机制内置**：副作用操作（发邮件、发评论）会暂停工作流，等待明确批准
- **可恢复执行**：暂停的工作流返回一个令牌；批准后可直接恢复，无需重新运行

## 为什么用 DSL 而不是普通程序

Lobster 的设计刻意保持小巧。目标不是"发明一门新语言"，而是提供一个可预测、对 AI 友好的管道规范，把审批和恢复令牌作为一等公民：

- **审批/恢复开箱即用**：普通程序当然可以提示用户，但要实现*带持久令牌的暂停-恢复*，你得自己写这套运行时
- **确定性 + 可审计**：管道即数据，天然易于记录、对比、重放、审查
- **为 AI 约束接口**：极简语法 + JSON 管道减少了"创造性"代码路径，让验证切实可行
- **安全策略内置**：超时、输出上限、沙箱检查、允许列表都由运行时统一执行，不需要每个脚本自己实现
- **依然可编程**：每个步骤可以调用任意 CLI 或脚本。需要 JS/TS？可以从代码生成 `.lobster` 文件

## 工作原理

OpenClaw 以**工具模式**启动本地的 `lobster` CLI，并从 stdout 解析 JSON 信封。如果管道因等待审批而暂停，工具会返回一个 `resumeToken`，供后续恢复使用。

## 模式：小型 CLI + JSON 管道 + 审批

构建输出 JSON 的小型命令，然后用 Lobster 把它们串联成一次调用。（下方示例命令名称仅供参考——换成你自己的即可。）

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

如果管道请求审批，用令牌恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI 触发工作流，Lobster 执行步骤。审批关卡让副作用变得明确且可审计。示例：把输入项映射成工具调用：

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## 纯 JSON 的 LLM 步骤 (llm-task)

当工作流需要**结构化的 LLM 处理**时，可以启用可选的 `llm-task` 插件工具，在 Lobster 中调用。这样既能保持工作流的确定性，又能用模型做分类/摘要/起草。启用方法：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

在管道中使用：

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

详见 [LLM 任务](./llm-task.md)。

## 工作流文件 (.lobster)

Lobster 支持运行 YAML/JSON 格式的工作流文件，可包含 `name`、`args`、`steps`、`env`、`condition`、`approval` 等字段。在 OpenClaw 工具调用中，把 `pipeline` 设为文件路径即可：

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

要点：

- `stdin: $step.stdout` 和 `stdin: $step.json` 用于传递前一步的输出
- `condition`（或 `when`）可以根据 `$step.approved` 条件化执行步骤

## 安装 Lobster

在运行 OpenClaw Gateway 的**同一台主机**上安装 Lobster CLI（参见 [Lobster 仓库](https://github.com/openclaw/lobster)），并确保 `lobster` 在 `PATH` 中。

## 启用工具

Lobster 是**可选**插件工具（默认不启用）。推荐配置（增量式，安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

或按智能体（agent）配置：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

除非你确实要用严格的允许列表模式，否则避免使用 `tools.allow: ["lobster"]`。注意：可选插件的允许列表是选择性加入的。如果你的允许列表只列出了插件工具（比如 `lobster`），OpenClaw 会保持核心工具启用。要限制核心工具，请在允许列表中一并包含想要的核心工具或工具组。

## 示例：邮件分类处理

不用 Lobster 时：

```
用户："检查我的邮件并起草回复"
→ openclaw 调用 gmail.list
→ LLM 总结
→ 用户："给 #2 和 #5 起草回复"
→ LLM 起草
→ 用户："发送 #2"
→ openclaw 调用 gmail.send
（每天都重复这套流程，完全不记得处理过哪些邮件）
```

用 Lobster：

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

返回 JSON 信封（已截断）：

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

用户批准 → 恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

一个工作流搞定。确定性。安全。

## 工具参数

### run

以工具模式运行管道。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

带参数运行工作流文件：

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

审批后继续已暂停的工作流。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### 可选参数

- `cwd`：管道的相对工作目录（必须在当前进程工作目录范围内）
- `timeoutMs`：子进程超时时间，超时则终止（默认：20000）
- `maxStdoutBytes`：stdout 大小上限，超限则终止子进程（默认：512000）
- `argsJson`：传给 `lobster run --args-json` 的 JSON 字符串（仅工作流文件可用）

## 输出信封

Lobster 返回一个 JSON 信封，状态为以下三者之一：

- `ok` → 成功完成
- `needs_approval` → 已暂停，需要 `requiresApproval.resumeToken` 才能恢复
- `cancelled` → 明确拒绝或取消

工具会在 `content`（美化 JSON）和 `details`（原始对象）两个字段中展示此信封。

## 审批

当返回值中存在 `requiresApproval` 时，检查提示内容后决定：

- `approve: true` → 恢复执行，继续副作用操作
- `approve: false` → 取消并终结工作流

用 `approve --preview-from-stdin --limit N` 可以在审批请求中附加 JSON 预览，无需手写 jq/heredoc 胶水代码。恢复令牌现在是紧凑格式：Lobster 把工作流恢复状态存在其状态目录下，只返回一个短令牌。

## OpenProse

OpenProse 与 Lobster 配合默契：用 `/prose` 编排多智能体（agent）准备阶段，再运行 Lobster 管道做确定性审批。如果 Prose 程序需要用到 Lobster，可以通过 `tools.subagents.tools` 为子智能体（agent）启用 `lobster` 工具。参见 [OpenProse](../prose.md)。

## 安全性

- **仅本地子进程** — 插件本身不发网络请求
- **不管理密钥** — Lobster 不处理 OAuth，它调用的是 OpenClaw 中负责此事的工具
- **沙箱感知** — 工具上下文处于沙箱中时自动禁用
- **加固措施** — 固定可执行文件名（`lobster`）在 `PATH` 中；强制超时和输出上限

## 故障排除

- **`lobster subprocess timed out`** → 增加 `timeoutMs`，或拆分长管道
- **`lobster output exceeded maxStdoutBytes`** → 调大 `maxStdoutBytes` 或减小输出量
- **`lobster returned invalid JSON`** → 确保管道在工具模式下运行且只输出 JSON
- **`lobster failed (code …)`** → 在终端中运行相同管道，查看 stderr

## 延伸阅读

- [插件](./plugin.md)
- [插件工具开发](../plugins/agent-tools.md)

## 案例：社区工作流

一个公开案例："第二大脑" CLI + Lobster 管道，管理三个 Markdown 知识库（个人、伴侣、共享）。CLI 输出 JSON 用于统计、收件箱清单、过时扫描；Lobster 把这些命令串联成 `weekly-review`、`inbox-triage`、`memory-consolidation`、`shared-task-sync` 等工作流，每个都带审批关卡。AI 可用时处理判断逻辑（分类），不可用时回退到确定性规则。

- 推文：[https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
- 仓库：[https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM 任务](./llm-task.md)[工具循环检测](./loop-detection.md)