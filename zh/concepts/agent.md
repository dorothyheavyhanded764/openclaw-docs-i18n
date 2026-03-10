

  基础（Fundamentals）

  
# 智能体运行时（Agent Runtime）

OpenClaw 内部运行着一个嵌入式的**智能体运行时（agent runtime）**，它基于 **pi‑mono** 代码库构建而来。

---

## 工作空间（Workspace，必需）

每个智能体运行时需要一个统一的工作目录，用于存放工具和上下文。这个目录通过 `agents.defaults.workspace` 配置，是智能体执行所有操作的**唯一工作目录（`cwd`）**。

**推荐做法：**

- 运行 `openclaw setup`，它会自动：
  - 创建 `~/.openclaw/openclaw.json`（如果不存在）
  - 初始化工作空间内的一组引导文件

完整的工作空间目录结构和备份说明，请参考 [智能体工作空间](./agent-workspace.md)。

如果启用了沙箱配置 `agents.defaults.sandbox`，非主会话可以在 `agents.defaults.sandbox.workspaceRoot` 下为**每个会话**建立独立的工作空间，这样同一个智能体的不同会话就能在隔离的目录中运行工具。具体字段含义与示例，见 [网关配置](../gateway/configuration.md)。

---

## 引导文件（Bootstrap files）

在工作空间目录下，OpenClaw 会读取一组**可编辑的 Markdown 文件**，作为智能体的"说明书 + 记忆 + 设定"：

| 文件 | 用途 |
| --- | --- |
| `AGENTS.md` | 操作说明 + "记忆"（长期信息与习惯） |
| `SOUL.md` | 人设（persona）、边界（boundaries）、语气（tone） |
| `TOOLS.md` | 工具使用笔记（例如 `imsg`、`sag` 以及你自定义的约定） |
| `BOOTSTRAP.md` | 首次运行仪式（完成后可删除） |
| `IDENTITY.md` | 智能体名称 / 气质 / 表情符号 |
| `USER.md` | 用户画像与首选称呼方式 |

### 注入时机

- 在**新会话的第一个回合**，OpenClaw 会将上述文件的内容**直接注入到智能体上下文**中
- 空文件会被跳过
- 过大的文件会被**裁剪并附带截断标记**，以保持提示（prompt）长度可控；完整内容仍保留在磁盘文件中

如果某个文件**缺失**：

- OpenClaw 会注入一行"缺失文件（missing file）"标记
- 下次运行 `openclaw setup` 时，会为这些文件生成安全的默认模板

### `BOOTSTRAP.md` 的生命周期

- 只有在**全新工作空间**（没有其他引导文件）时，才会首次创建 `BOOTSTRAP.md`
- 完成引导仪式后，你可以删除该文件，后续重启时**不会自动重新生成**
- 如果你已有一套预置好的工作空间，不希望 OpenClaw 创建任何引导文件，可以关闭这一行为：

```json
{ "agent": { "skipBootstrap": true } }
```

---

## 内置工具（Built‑in tools）

OpenClaw 自带一组核心工具（读写文件、执行命令等），它们**始终可用**，但会受**工具策略（tool policy）**约束。其中 `apply_patch` 是可选功能，由 `tools.exec.applyPatch` 开关控制。

需要特别注意的是：`TOOLS.md` **不会**决定"有哪些工具可用"，它更像是"给未来的自己和智能体看的使用说明"。真正的工具启用/禁用，是在配置文件中的 `tools.*` 和 `agents.*.tools` 下进行控制。

---

## 技能（Skills）

除了内置工具，OpenClaw 还支持通过 **Skill** 机制扩展功能。技能加载遵循以下优先级（**同名时工作空间优先**）：

1. 随安装包自带的技能（bundled skills）
2. 用户级技能：`~/.openclaw/skills`
3. 工作空间内技能：`/skills`

是否启用某些技能，可以通过配置或环境变量进行控制，详见 [网关配置](../gateway/configuration.md) 中的 `skills` 部分。

---

## 与 pi‑mono 的关系

OpenClaw 复用了 pi‑mono 代码库中的部分能力（模型适配和工具实现），但在运行时层面，两者有明确的职责边界：

- **pi‑mono 提供**：模型调用、部分工具实现等基础能力
- **OpenClaw 负责**：会话生命周期、多智能体路由、工具接线与权限策略

因此：

- 不存在单独的 "pi‑coding agent runtime"
- OpenClaw **不会读取** `~/.pi/agent` 或 `/.pi` 下的设置，一切以 `~/.openclaw` 与工作空间配置为准

---

## 会话与记录（Sessions）

每个智能体会话的对话记录以 **JSON Lines（JSONL）** 形式存储在本地：

- 路径格式：`~/.openclaw/agents//sessions/.jsonl`
- 会话 ID 由 OpenClaw 生成并保持稳定，便于后续调试或审计
- 旧版 Pi / Tau 的会话文件夹不会被读取或迁移

---

## 流式过程中的"转向"（Steering while streaming）

当启用**流式输出（streaming）**与**消息队列（queue）**功能时，可以让智能体在"说话说到一半"时根据新消息调整行为——这需要理解 `queue.mode` 相关的两种模式：

### 模式一：`steer`

- 新到的用户消息会被**插入到当前运行中**
- 队列在**每次工具调用结束后**检查：
  - 如果发现排队消息：
    - 当前助手消息剩余的工具调用会被跳过，错误结果标记为 `"Skipped due to queued user message."`
    - 在下一次助手回复前，将排队的用户消息注入上下文
- 适用场景：希望"打断当前计划"，让智能体**立即转向**新的问题或指令

### 模式二：`followup` / `collect`

- 新消息会被**暂存到队列中**，直到当前回合结束
- 当前回合结束后，OpenClaw 会基于队列中的负载开启一个新的智能体回合
- 适用场景：希望当前任务先跑完，再整体处理后续问题

更多关于队列模式、防抖与上限配置，详见 [队列](./queue.md)。

---

## 块流式输出（Block streaming）

为了在流式输出与可读性之间取得平衡，OpenClaw 支持"块级流式"配置：

- **默认关闭**：`agents.defaults.blockStreamingDefault: "off"`
- 打开后，会在一个"块（block）"完成时立刻发送，而不是等整条消息结束

你可以通过以下字段精细控制：

1. **边界类型** —— `agents.defaults.blockStreamingBreak`
   - 可选：`text_end` 或 `message_end`（默认 `text_end`）
   - `text_end` 以"文本段落结束"为边界，`message_end` 以"整条消息结束"为边界

2. **块大小** —— `agents.defaults.blockStreamingChunk`
   - 默认区间：约 800–1200 字符
   - 优先在**段落分隔**处切块，其次是换行，最后才是句子边界

3. **块合并** —— `agents.defaults.blockStreamingCoalesce`
   - 通过"空闲时间"阈值，将短小的流式块合并后再发，减少单行消息刷屏

4. **按频道启用**
   - 非 Telegram 频道需要显式设置 `*.blockStreaming: true` 才会启用块回复

其他细节：

- 工具调用的详细摘要会在**工具开始时**立即写入日志（不做防抖）
- 控制面板（Control UI）会在可用时通过智能体事件实时流式展示工具输出

更多详情见 [流式传输与分块](./streaming.md)。

---

## 模型引用（Model refs）

配置中常见的模型引用字段：

- `agents.defaults.model`
- `agents.defaults.models`

这些"模型引用"遵循一个简单的解析规则：

- 按**第一个 `/`** 进行分割
- 推荐写成 `provider/model` 的形式，例如：
  - `openai/gpt-4.1`
  - `anthropic/claude-3.6-sonnet`

**特殊情况：**

- 如果模型 ID 本身包含 `/`（例如 OpenRouter 风格），需要带上提供商前缀：
  - 如：`openrouter/moonshotai/kimi-k2`
- 如果**完全省略提供商**，且模型 ID 中不包含 `/`：
  - OpenClaw 会将其视作默认提供商（default provider）下的别名或模型名

---

## 最小必需配置

要让智能体运行时可靠工作，建议至少设置：

- `agents.defaults.workspace` —— 指定工作空间路径
- `channels.whatsapp.allowFrom` —— WhatsApp 频道的允许列表（强烈推荐，用于限制能与网关交互的号码）

---

*下一步：[群组聊天](../channels/group-messages.md)* 🦞

[网关架构](./architecture.md) · [智能体循环](./agent-loop.md)