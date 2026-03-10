

  配置与运维

  
# 日志

关于面向用户的概述（CLI + 控制界面 + 配置），请参阅 [/logging](../logging.md)。OpenClaw 有两类日志输出：

-   **控制台输出**（终端/调试界面中看到的内容）。
-   **文件日志**（JSON Lines 格式），由网关日志器写入。

## 文件日志器

-   默认的滚动日志文件位于 `/tmp/openclaw/` 目录下（每天一个文件）：`openclaw-YYYY-MM-DD.log`
    -   日期使用网关主机的本地时区。
-   日志文件路径和级别可通过 `~/.openclaw/openclaw.json` 配置：
    -   `logging.file`
    -   `logging.level`

文件格式为每行一个 JSON 对象。控制界面的"日志"标签页通过网关的 `logs.tail` 接口实时追踪此文件。CLI 也可以做到同样效果：

```bash
openclaw logs --follow
```

**详细模式与日志级别的关系**

-   **文件日志**完全由 `logging.level` 控制。
-   `--verbose` 仅影响**控制台输出的详细程度**（以及 WebSocket 日志样式），**不会**提高文件日志级别。
-   如需在文件日志中捕获详细信息，请将 `logging.level` 设为 `debug` 或 `trace`。

## 控制台捕获

CLI 会捕获 `console.log/info/warn/error/debug/trace` 的输出并写入文件日志，同时仍打印到 stdout/stderr。你可以独立调整控制台详细程度：

-   `logging.consoleLevel`（默认 `info`）
-   `logging.consoleStyle`（`pretty` | `compact` | `json`）

## 工具摘要脱敏

详细的工具摘要（如 `🛠️ Exec: ...`）可以在输出到控制台之前屏蔽敏感令牌。此功能**仅针对工具输出**，不会修改文件日志。

-   `logging.redactSensitive`: `off` | `tools`（默认: `tools`）
-   `logging.redactPatterns`: 正则表达式字符串数组（覆盖默认规则）
    -   使用原始正则字符串（自动添加 `gi` 标志），或使用 `/pattern/flags` 格式自定义标志。
    -   匹配项保留前 6 位和后 4 位字符（长度 ≥ 18 时），否则显示为 `***`。
    -   默认规则覆盖常见的键值赋值、CLI 参数、JSON 字段、Bearer 请求头、PEM 块以及常见的令牌前缀。

## 网关 WebSocket 日志

网关以两种模式打印 WebSocket 协议日志：

-   **普通模式（不带 `--verbose`）**：仅打印"值得关注"的 RPC 结果：
    -   错误（`ok=false`）
    -   慢调用（默认阈值：`>= 50ms`）
    -   解析错误
-   **详细模式（带 `--verbose`）**：打印所有 WebSocket 请求/响应流量。

### WebSocket 日志样式

`openclaw gateway` 支持针对每个网关的样式设置：

-   `--ws-log auto`（默认）：普通模式已优化；详细模式使用紧凑输出
-   `--ws-log compact`：详细模式下使用紧凑输出（请求/响应成对显示）
-   `--ws-log full`：详细模式下显示完整的每帧输出
-   `--compact`：`--ws-log compact` 的别名

示例：

```bash
# 优化模式（仅显示错误/慢调用）
openclaw gateway

# 显示所有 WebSocket流量（紧凑格式）
openclaw gateway --verbose --ws-log compact

# 显示所有 WebSocket 流量（完整元数据）
openclaw gateway --verbose --ws-log full
```

## 控制台格式化（子系统日志）

控制台格式化器能够**识别 TTY** 并打印统一、带前缀的日志行。子系统日志器让输出保持分组且易于浏览。特性：

-   **子系统前缀**：每行都有（如 `[gateway]`、`[canvas]`、`[tailscale]`）
-   **子系统颜色**：每个子系统颜色稳定，配合日志级别着色
-   **颜色检测**：当输出是 TTY 或环境看起来像富终端时启用颜色（检测 `TERM`/`COLORTERM`/`TERM_PROGRAM`），尊重 `NO_COLOR` 环境变量
-   **简化子系统前缀**：移除开头的 `gateway/` 和 `channels/`，保留最后 2 段（如 `whatsapp/outbound`）
-   **按子系统划分的子日志器**（自动添加前缀和结构化字段 `{ subsystem }`）
-   **`logRaw()`**：用于二维码/用户界面输出（无前缀、无格式化）
-   **控制台样式**（如 `pretty | compact | json`）
-   **控制台日志级别**与文件日志级别分离（当 `logging.level` 设为 `debug`/`trace` 时，文件日志保留完整详情）
-   **WhatsApp 消息体**在 `debug` 级别记录（使用 `--verbose` 可见）

这样既保持了文件日志的稳定性，又让交互式输出清晰易读。

[Doctor](./doctor.md)[网关锁](./gateway-lock.md)