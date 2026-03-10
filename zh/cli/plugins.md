

  CLI 命令

  
# plugins

本节介绍如何管理 Gateway 的插件（plugin）和扩展——它们会在进程内加载运行。

在深入命令之前，你可能需要了解这些相关内容：

-   插件系统概述：[插件](../tools/plugin.md)
-   插件清单格式与配置模式：[插件清单](../plugins/manifest.md)
-   安全加固指南：[安全性](../gateway/security.md)

## 命令一览

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

OpenClaw 自带了一些捆绑插件，但它们默认处于禁用状态。你可以用 `plugins enable` 命令来激活它们。

每个插件都必须提供一个 `openclaw.plugin.json` 清单文件，里面要包含内联的 JSON Schema（放在 `configSchema` 字段中，即使为空也必须提供）。如果清单缺失、格式错误或 Schema 无效，插件将无法加载，配置验证也会失败。

### 安装插件

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

**安全提醒**：安装插件本质上就是在你的系统上运行代码，请谨慎选择来源。建议优先使用固定版本。

**npm 安装规范**：只接受注册表包名，格式为「包名 + 可选的精确版本或 dist-tag」。以下格式会被拒绝：Git 仓库地址、URL、本地文件路径、semver 版本范围。为了安全起见，依赖安装时会自动加上 `--ignore-scripts` 参数。

**版本选择**：如果你只写包名或使用 `@latest` 标签，OpenClaw 会追踪稳定版本。如果 npm 解析到了预发布版本（比如 beta 或 rc），OpenClaw 会暂停并提示你明确选择——你需要用 `@beta`、`@rc` 这样的预发布标签，或者精确的预发布版本号（如 `@1.2.3-beta.4`）来确认。

**捆绑插件优先**：如果你指定的包名恰好和某个捆绑插件的 ID 相同（比如 `diffs`），OpenClaw 会直接安装那个捆绑插件。如果你想安装同名的 npm 包，需要加上作用域前缀（如 `@scope/diffs`）。

**支持的归档格式**：`.zip`、`.tgz`、`.tar.gz`、`.tar`

**本地开发**：用 `--link` 参数可以避免复制本地目录，插件会以链接方式添加到 `plugins.load.paths` 中：

```bash
openclaw plugins install -l ./my-plugin
```

**固定版本**：在 npm 安装时加上 `--pin` 参数，OpenClaw 会把解析出的精确版本（`name@version`）保存到 `plugins.installs` 配置中。这样既享受了固定版本的确定性，又不影响默认的灵活升级行为。

### 卸载插件

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

卸载插件时，OpenClaw 会清理以下位置的相关记录：`plugins.entries`、`plugins.installs`、插件白名单，以及（如果是链接插件）`plugins.load.paths` 中的对应条目。如果被卸载的是正在使用的内存插件，内存槽会自动重置回 `memory-core`。

默认情况下，OpenClaw 还会删除插件安装目录（位于 `$OPENCLAW_STATE_DIR/extensions/`）。如果你想保留磁盘上的文件（比如稍后还要重新安装），可以用 `--keep-files` 参数。另外，`--keep-config` 作为 `--keep-files` 的旧别名仍然可用，但已不推荐使用。

### 更新插件

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新功能只对通过 npm 安装的插件有效（这些插件会被记录在 `plugins.installs` 中）。

**完整性校验**：如果 OpenClaw 检测到远程包的哈希值与本地存储的不一致，会打印警告并要求你确认后才会继续更新。在 CI 环境或非交互式脚本中，你可以用全局 `--yes` 参数来跳过确认提示。

[pairing](./pairing.md)[qr](./qr.md)