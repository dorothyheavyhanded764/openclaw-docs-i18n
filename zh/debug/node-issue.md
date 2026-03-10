

  环境与调试（debug）

  
# Node + tsx 崩溃

运行 OpenClaw 网关或 CLI 时，如果遇到 **Node.js 或 tsx 崩溃**（比如 `node --import tsx` 或使用 tsx 的脚本），本页会帮你排查问题。

## 常见原因

- **Node 版本**：OpenClaw 需要 Node 22 或更高版本。版本过低可能导致运行时错误或原生模块崩溃。
- **tsx / 加载器**：启动时崩溃或加载 TypeScript 失败，可能是 tsx 版本问题、`--import` 参数顺序不对，或者与其他加载器冲突。
- **内存 / 原生模块**：大负载或原生插件可能触发内存溢出（OOM）或段错误。版本和环境配置建议请参考 [Node.js](../install/node.md)。

## 快速排查步骤

1. **检查 Node 版本**：运行 `node -v`，确认显示 v22 或更高版本。
2. **不用 tsx 复现**：用纯 `node` 运行预编译的 JS，看是否还会崩溃。这样可以判断问题是否与 tsx 相关。
3. **开启诊断日志**：使用 [诊断标志](../diagnostics/flags.md)（如 `OPENCLAW_DEBUG_*`），在崩溃前获取更多日志信息。

## 相关链接

- [调试（debug）](../help/debugging.md) — 运行时覆盖、网关 watch 模式、开发配置
- [脚本](../help/scripts.md) — 辅助脚本与约定
- [Node.js](../install/node.md) — Node 版本与 PATH 配置
- [诊断标志](../diagnostics/flags.md) — 调试（debug）与跟踪标志