

  CLI 命令

  
# hooks

`hooks` 命令帮你管理智能体钩子（hooks）——这是一类事件驱动的自动化机制，可以在特定事件（如 `/new`、`/reset` 命令或网关启动）发生时自动执行预设操作。

**相关内容：**

-   钩子概述：[Hooks](../automation/hooks.md)
-   插件钩子：[Plugins](../tools/plugin.md#plugin-hooks)

## 列出所有钩子

想看看当前有哪些钩子可用？运行：

```bash
openclaw hooks list
```

这个命令会扫描工作区、托管目录和内置目录，展示所有已发现的钩子。

**选项：**

-   `--eligible`: 只显示符合条件的钩子（满足所有前置要求）
-   `--json`: 以 JSON 格式输出，方便程序解析
-   `-v, --verbose`: 显示详细信息，包括未满足的要求

**示例输出：**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - 在网关启动时运行 BOOT.md
  📎 bootstrap-extra-files ✓ - 在智能体引导期间注入额外的工作区引导文件
  📝 command-logger ✓ - 将所有命令事件记录到集中审计文件
  💾 session-memory ✓ - 当发出 /new 命令时将会话上下文保存到内存
```

**查看详细信息（verbose 模式）：**

```bash
openclaw hooks list --verbose
```

这个模式会显示不符合条件的钩子缺少哪些前置要求。

**JSON 格式输出：**

```bash
openclaw hooks list --json
```

返回结构化的 JSON 数据，适合在脚本或程序中使用。

## 查看钩子详情

想了解某个钩子的具体信息？使用：

```bash
openclaw hooks info <name>
```

**参数：**

-   ``: 钩子名称，例如 `session-memory`

**选项：**

-   `--json`: 以 JSON 格式输出

**示例：**

```bash
openclaw hooks info session-memory
```

**输出：**

```
💾 session-memory ✓ 就绪

当发出 /new 命令时将会话上下文保存到内存

详细信息：
  来源：openclaw-bundled
  路径：/path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  处理器：/path/to/openclaw/hooks/bundled/session-memory/handler.ts
  主页：https://docs.openclaw.ai/automation/hooks#session-memory
  事件：command:new

要求：
  配置：✓ workspace.dir
```

## 检查钩子状态

快速了解整体情况：

```bash
openclaw hooks check
```

显示钩子的就绪状态摘要——有多少已就绪，有多少还未就绪。

**选项：**

-   `--json`: 以 JSON 格式输出

**示例输出：**

```
钩子状态

钩子总数：4
就绪：4
未就绪：0
```

## 启用钩子

找到想用的钩子后，启用它：

```bash
openclaw hooks enable <name>
```

命令会将钩子添加到你的配置文件（`~/.openclaw/config.json`）中。

**参数：**

-   ``: 钩子名称，例如 `session-memory`

> **注意：** 由插件管理的钩子在 `openclaw hooks list` 中显示为 `plugin:`，不能通过此命令启用或禁用。请通过启用或禁用对应的插件来控制这类钩子。

**示例：**

```bash
openclaw hooks enable session-memory
```

**输出：**

```
✓ 已启用钩子：💾 session-memory
```

**命令执行流程：**

1.  检查钩子是否存在且符合条件
2.  在配置中更新 `hooks.internal.entries..enabled = true`
3.  将配置保存到磁盘

**启用后：**

重启网关让钩子生效。在 macOS 上重启菜单栏应用，或在开发环境中重启你的网关进程。

## 禁用钩子

不再需要某个钩子？禁用它：

```bash
openclaw hooks disable <name>
```

**参数：**

-   ``: 钩子名称，例如 `command-logger`

**示例：**

```bash
openclaw hooks disable command-logger
```

**输出：**

```
⏸ 已禁用钩子：📝 command-logger
```

**禁用后：**

重启网关让更改生效。

## 安装钩子

从本地目录、压缩包或 npm 安装新的钩子包：

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

**npm 规范说明：**

npm 安装仅支持注册表规范——包名加可选的**确切版本号**或**分发标签**（dist-tag）。不支持 Git URL、文件路径或 semver 范围（如 `^1.0.0`）。

为了安全，依赖安装会使用 `--ignore-scripts` 参数。裸包名和 `@latest` 会保持在稳定版本轨道上。如果 npm 将这些解析为预发布版本，OpenClaw 会停止并提示你明确选择预发布标签（如 `@beta`/`@rc`）或确切的预发布版本号。

**命令执行流程：**

-   将钩子包复制到 `~/.openclaw/hooks/`
-   在 `hooks.internal.entries.*` 中启用已安装的钩子
-   在 `hooks.internal.installs` 下记录安装信息

**选项：**

-   `-l, --link`: 链接本地目录而非复制（将目录添加到 `hooks.internal.load.extraDirs`）
-   `--pin`: 将 npm 安装记录为确切解析的 `name@version`，存放在 `hooks.internal.installs`

**支持的压缩格式：** `.zip`、`.tgz`、`.tar.gz`、`.tar`

**示例：**

```bash
# 本地目录
openclaw hooks install ./my-hook-pack

# 本地压缩包
openclaw hooks install ./my-hook-pack.zip

# NPM 包
openclaw hooks install @openclaw/my-hook-pack

# 链接本地目录（不复制）
openclaw hooks install -l ./my-hook-pack
```

## 更新钩子

更新已安装的钩子包（仅限通过 npm 安装的）：

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

**选项：**

-   `--all`: 更新所有已跟踪的钩子包
-   `--dry-run`: 只显示会发生的更改，不实际执行

**安全提示：** 当已存储的完整性哈希与下载的工件哈希不一致时，OpenClaw 会打印警告并要求确认后再继续。在 CI 或非交互环境中，使用全局 `--yes` 参数跳过确认提示。

## 内置钩子

OpenClaw 自带几个实用的内置钩子。

### session-memory

当你执行 `/new` 命令时，自动保存当前会话上下文到内存文件中。这对于跨会话保持上下文很有用。

**启用：**

```bash
openclaw hooks enable session-memory
```

**输出位置：** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**详细文档：** [session-memory 文档](../automation/hooks.md#session-memory)

### bootstrap-extra-files

在智能体引导阶段（`agent:bootstrap`），自动注入额外的引导文件。比如在 monorepo 项目中注入本地的 `AGENTS.md` 或 `TOOLS.md`。

**启用：**

```bash
openclaw hooks enable bootstrap-extra-files
```

**详细文档：** [bootstrap-extra-files 文档](../automation/hooks.md#bootstrap-extra-files)

### command-logger

将所有命令事件记录到集中审计日志文件中，便于追踪和审计。

**启用：**

```bash
openclaw hooks enable command-logger
```

**输出位置：** `~/.openclaw/logs/commands.log`

**查看日志：**

```bash
# 查看最近的命令记录
tail -n 20 ~/.openclaw/logs/commands.log

# 美化打印 JSON
cat ~/.openclaw/logs/commands.log | jq .

# 按操作类型过滤
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**详细文档：** [command-logger 文档](../automation/hooks.md#command-logger)

### boot-md

在网关启动时（通道启动完成后）自动运行 `BOOT.md` 脚本。

**监听事件：** `gateway:startup`

**启用：**

```bash
openclaw hooks enable boot-md
```

**详细文档：** [boot-md 文档](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)