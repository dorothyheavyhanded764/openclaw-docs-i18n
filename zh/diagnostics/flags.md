

  环境与调试

  
# 诊断标志

当你需要调试某个特定子系统（比如 Telegram HTTP 请求）时，不想开启全局详细日志怎么办？诊断标志（diagnostics flags）让你可以精准地为指定模块开启调试日志，而不会淹没其他无关信息。标志默认关闭，只有在对应子系统检查时才会生效。

## 工作原理

- 标志是字符串，不区分大小写
- 可通过配置文件或环境变量启用
- 支持通配符匹配：
    - `telegram.*` 会匹配 `telegram.http`
    - `*` 表示启用所有标志

## 通过配置启用

在配置文件中添加：

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

启用多个标志：

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

修改后需要重启网关才能生效。

## 环境变量覆盖（临时调试）

临时启用标志，无需修改配置文件：

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

禁用所有标志：

```
OPENCLAW_DIAGNOSTICS=0
```

## 日志输出位置

标志产生的日志会写入标准诊断日志文件，默认路径为：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

如果你设置了 `logging.file`，则以该路径为准。日志采用 JSONL 格式（每行一个 JSON 对象），敏感信息脱敏规则（`logging.redactSensitive`）依然生效。

## 提取日志

找到最新的日志文件：

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

过滤 Telegram HTTP 相关的诊断日志：

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

或者在复现问题时实时查看：

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

如果是远程网关，可以使用 `openclaw logs --follow` 命令（详见 [/cli/logs](../cli/logs.md)）。

## 注意事项

- 如果 `logging.level` 设置为高于 `warn` 的级别，这些日志可能会被过滤掉。默认的 `info` 级别没有问题
- 标志可以长期启用，它们只会增加指定子系统的日志量，不会影响其他功能
- 如需修改日志目标路径、级别或脱敏规则，请参考 [/logging](../logging.md)

[Node + tsx 崩溃](../debug/node-issue.md)[Node.js](../install/node.md)