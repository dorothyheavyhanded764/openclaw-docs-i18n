

  会话与内存

  
# 压缩（Compaction）

你有没有发现，和智能体（agent）聊久了，对话越来越长，响应却越来越慢？这通常是因为**上下文窗口（context window）**快满了——每个模型能"看到"的 token 都有上限。当对话累积的消息和工具结果超出这个限制时，OpenClaw 会自动**压缩**较早的历史记录，让对话继续顺畅进行。

## 压缩是如何工作的

简单来说，压缩会把**较早的对话内容总结成一条摘要**，同时保留最近的消息不变。压缩后的摘要会保存在会话历史中，所以后续请求只会用到：

-   压缩生成的摘要
-   压缩点之后的最近消息

最重要的是，压缩结果会**持久化保存**到会话的 JSONL 历史文件中，不会丢失。

## 配置压缩行为

在 `openclaw.json` 中，通过 `agents.defaults.compaction` 设置可以控制压缩的各种行为，比如压缩模式、目标 token 数等。

默认情况下，压缩摘要会保留原有的标识符（`identifierPolicy: "strict"`）。你也可以：

-   设置 `identifierPolicy: "off"` 关闭标识符保留
-   设置 `identifierPolicy: "custom"` 并配合 `identifierInstructions` 提供自定义的压缩指令

## 自动压缩（默认开启）

当会话接近或超过模型的上下文窗口限制时，OpenClaw 会自动触发压缩，然后用压缩后的上下文重新处理你的请求。你可以通过以下方式确认压缩已执行：

-   在详细模式下看到 `🧹 Auto-compaction complete` 提示
-   运行 `/status` 查看 `🧹 Compactions: ` 计数

值得一提的是，压缩前 OpenClaw 可能会先执行一次**静默内存刷新**，把需要持久化的笔记存到磁盘。详见 [Memory](./memory.md)。

## 手动压缩

想主动清理一下对话？用 `/compact` 命令（可以附带具体指令）：

```bash
/compact Focus on decisions and open questions
```

这会让压缩时重点关注你指定的内容。

## 上下文窗口从哪来

上下文窗口的大小取决于具体模型——OpenClaw 会根据你配置的提供商目录中的模型定义来确定这个限制。

## 压缩 vs 修剪

这两个概念容易混淆，但它们做的事情不同：

-   **压缩**：总结历史内容并**持久化保存**到 JSONL 文件
-   **会话修剪**：只在内存中临时裁剪旧的**工具结果**，每次请求单独处理

修剪的详细说明见 [/concepts/session-pruning](./session-pruning.md)。

## OpenAI 服务器端压缩

除了本地压缩，OpenClaw 还支持 OpenAI 的服务器端压缩提示（针对兼容的直接 OpenAI 模型）。这两种压缩可以同时使用：

-   **本地压缩**：OpenClaw 在本地总结并保存到会话 JSONL
-   **服务器端压缩**：启用 `store` + `context_management` 后，OpenAI 在服务端压缩上下文

模型参数和配置覆盖详见 [OpenAI provider](../providers/openai.md)。

## 小贴士

-   觉得对话"变笨"或上下文太冗长时，试试 `/compact` 手动压缩
-   大型工具输出会被自动截断，配合修剪可以进一步减少工具结果的堆积
-   想彻底重新开始？`/new` 或 `/reset` 会创建新的会话 ID

[Memory](./memory.md)[Multi-Agent Routing](./multi-agent.md)