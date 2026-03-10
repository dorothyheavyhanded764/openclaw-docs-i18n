

  CLI 命令

  
# memory

这一组命令帮你管理语义记忆（memory）的索引和搜索功能。它由当前启用的记忆插件提供支持（默认是 `memory-core`；如果你想完全禁用，可以设置 `plugins.slots.memory = "none"`）。

想深入了解？可以参考：

- 记忆的工作原理：[Memory](../concepts/memory.md)
- 插件机制：[Plugins](../tools/plugin.md)

## 使用示例

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## 命令选项详解

### `memory status` 和 `memory index` 共有选项

- `--agent `：只对指定的智能体（agent）生效。如果不指定，命令会对所有已配置的智能体逐个执行；如果没有配置智能体列表，则使用默认智能体。
- `--verbose`：在探测和索引过程中输出详细的日志信息，方便排查问题。

### `memory status` 专属选项

- `--deep`：深入检查向量数据库和嵌入模型的可用性。
- `--index`：如果存储有更新未同步，自动触发重新索引（此选项隐含了 `--deep`）。
- `--json`：以 JSON 格式输出结果，便于程序解析。

### `memory index` 专属选项

- `--force`：强制执行完整的重新索引，忽略增量更新。

### `memory search` 专属选项

查询内容有两种方式：

- 位置参数：直接在命令后写查询内容，如 `openclaw memory search "关键词"`
- `--query `：使用选项参数，如 `openclaw memory search --query "关键词"`

如果同时提供了两种方式，`--query` 参数优先。如果两者都没提供，命令会报错退出。

其他选项：

- `--agent `：在指定智能体的记忆范围内搜索（默认使用默认智能体）。
- `--max-results `：限制返回结果的最大数量。
- `--min-score `：设置最低相似度阈值，过滤掉分数过低的结果。
- `--json`：以 JSON 格式输出搜索结果。

## 注意事项

- 使用 `memory index --verbose` 可以看到每个阶段的详细处理信息，包括嵌入提供者、模型、数据来源、批处理进度等。
- `memory status` 的输出会包含通过 `memorySearch.extraPaths` 配置的额外索引路径。
- 如果记忆远程 API 的密钥字段配置为 SecretRefs，命令会尝试从当前网关快照中解析这些值。如果网关不可用，命令会立即失败并提示错误。
- **网关版本兼容性**：此命令需要网关支持 `secrets.resolve` 方法。如果你使用的是较旧版本的网关，会收到"未知方法"的错误提示。

[logs](./logs.md)[message](./message.md)