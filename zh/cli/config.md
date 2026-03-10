

  CLI 命令

  
# config

配置辅助工具：通过路径获取/设置/取消设置/验证值，并打印当前使用的配置文件路径。不带子命令运行时会打开配置向导（与 `openclaw configure` 相同）。

## 示例

```bash
openclaw config file
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
openclaw config validate
openclaw config validate --json
```

## 路径

路径使用点表示法或括号表示法：

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

使用智能体列表索引来指定特定的智能体：

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## 值

值在可能的情况下会被解析为 JSON5；否则会被视为字符串。使用 `--strict-json` 来要求 JSON5 解析。`--json` 作为传统别名仍然受支持。

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

## 子命令

-   `config file`：打印当前使用的配置文件路径（从 `OPENCLAW_CONFIG_PATH` 或默认位置解析）。

编辑后需要重启网关。

## 验证

在不启动网关的情况下，根据当前 schema 验证配置。

```bash
openclaw config validate
openclaw config validate --json
```

[completion](./completion.md)[configure](./configure.md)