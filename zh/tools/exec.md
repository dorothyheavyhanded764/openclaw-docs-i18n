

  内置工具

  
# Exec 工具

Exec 工具让你能在工作区中执行 Shell 命令。它支持前台执行和后台执行（通过 `process` 工具配合）。如果 `process` 工具被禁用，`exec` 会以同步方式运行，并忽略 `yieldMs` 和 `background` 参数。后台会话按智能体（agent）隔离——每个智能体只能看到自己创建的会话。

## 参数说明

以下是 `exec` 工具支持的所有参数：

- `command`（必需）：要执行的命令
- `workdir`（默认：当前工作目录）：命令执行的工作目录
- `env`（可选）：环境变量键值对，用于覆盖默认环境
- `yieldMs`（默认：10000）：超时自动转入后台的毫秒数
- `background`（布尔值）：设为 `true` 可立即在后台执行
- `timeout`（秒，默认：1800）：命令超时时间，到期后自动终止
- `pty`（布尔值）：在伪终端中运行，适用于需要 TTY 的 CLI 工具、编码智能体、终端 UI 程序
- `host`（`sandbox | gateway | node`）：命令执行的目标位置
- `security`（`deny | allowlist | full`）：`gateway` 和 `node` 模式下的安全策略
- `ask`（`off | on-miss | always`）：`gateway` 和 `node` 模式下的审批提示策略
- `node`（字符串）：当 `host=node` 时，指定节点的 ID 或名称
- `elevated`（布尔值）：请求提升权限模式（仅网关主机）；只有当 `elevated` 实际生效为 `full` 时，才会强制 `security=full`

### 重要说明

- `host` 默认为 `sandbox`（沙盒环境）
- 当沙盒功能关闭时，`elevated` 参数会被忽略（因为 `exec` 已经在主机上运行了）
- `gateway` 和 `node` 的审批策略由 `~/.openclaw/exec-approvals.json` 文件控制
- 使用 `node` 需要先配对节点（伴侣应用或无头节点主机）
- 如果有多个可用节点，需要设置 `exec.node` 或 `tools.exec.node` 来指定
- 在非 Windows 系统上，`exec` 会优先使用 `SHELL` 环境变量；但如果 `SHELL` 是 `fish`，会改为使用 `PATH` 中的 `bash`（或 `sh`），以避免 fish 脚本兼容性问题；如果都找不到，才回退到 `SHELL`
- 在 Windows 系统上，`exec` 会优先查找 PowerShell 7 (`pwsh`)（搜索顺序：Program Files、ProgramW6432、PATH），找不到则回退到 Windows PowerShell 5.1
- 主机执行模式（`gateway`/`node`）会拒绝 `env.PATH` 和加载器覆盖（`LD_*`/`DYLD_*`），以防止二进制劫持或代码注入
- OpenClaw 会在执行的命令环境中设置 `OPENCLAW_SHELL=exec`（包括 PTY 和沙盒执行），这样你的 shell 配置文件就能检测到这是 `exec` 工具触发的
- **重要**：沙盒功能**默认关闭**。如果沙盒关闭但仍显式配置或请求 `host=sandbox`，现在会直接报错，而不是静默地回退到网关主机执行。请启用沙盒功能，或改用 `host=gateway` 并配置审批策略
- 脚本预检（用于检测常见的 Python/Node shell 语法错误）只检查 `workdir` 范围内的文件。如果脚本路径解析到了 `workdir` 之外，该文件会跳过预检

## 配置选项

你可以在配置文件中设置以下选项：

- `tools.exec.notifyOnExit`（默认：`true`）：设为 `true` 时，后台执行的会话在退出时会发送系统事件通知并请求心跳
- `tools.exec.approvalRunningNoticeMs`（默认：`10000`）：当需要审批的命令运行超过此时长后，发出一条"运行中"通知（设为 `0` 可禁用）
- `tools.exec.host`（默认：`sandbox`）
- `tools.exec.security`（默认：沙盒模式为 `deny`，网关和节点模式未设置时为 `allowlist`）
- `tools.exec.ask`（默认：`on-miss`）
- `tools.exec.node`（默认：未设置）
- `tools.exec.pathPrepend`：在执行前预置到 `PATH` 的目录列表（仅限 `gateway` 和 `sandbox` 模式）
- `tools.exec.safeBins`：仅从标准输入读取的安全二进制文件列表，无需显式加入允许列表即可运行。详见[安全二进制文件](./exec-approvals.md#safe-bins-stdin-only)
- `tools.exec.safeBinTrustedDirs`：为安全二进制文件路径检查额外指定信任目录。`PATH` 中的条目不会自动受信任。内置默认值为 `/bin` 和 `/usr/bin`
- `tools.exec.safeBinProfiles`：为自定义安全二进制文件配置 argv 策略（支持 `minPositional`、`maxPositional`、`allowedValueFlags`、`deniedFlags`）

配置示例：

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH 环境变量处理

不同执行模式下 `PATH` 的处理方式有所不同：

- **`host=gateway`**：会合并你登录 shell 的 `PATH` 到执行环境中。主机执行模式会拒绝 `env.PATH` 覆盖。守护进程本身使用精简的 `PATH`：
  - macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
  - Linux：`/usr/local/bin`、`/usr/bin`、`/bin`
- **`host=sandbox`**：在容器内以登录 shell 模式运行 `sh -lc`，所以 `/etc/profile` 可能会重置 `PATH`。OpenClaw 会通过内部环境变量（不做 shell 插值）在配置文件加载后预置 `env.PATH`；`tools.exec.pathPrepend` 在这里也生效
- **`host=node`**：只发送你传递的非阻塞环境变量覆盖到节点。主机执行拒绝 `env.PATH` 覆盖，节点主机也会忽略它们。如果需要在节点上添加额外的 PATH 条目，请配置节点主机服务环境（systemd/launchd）或将工具安装到标准位置

为特定智能体绑定节点（使用配置中的智能体列表索引）：

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

在控制 UI 的节点选项卡中，也有一个小型的"Exec 节点绑定"面板可以进行相同的设置。

## 会话覆盖（/exec 命令）

使用 `/exec` 命令可以为当前会话设置 `host`、`security`、`ask` 和 `node` 的默认值。直接发送 `/exec` 不带参数可查看当前设置。

示例：

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## 授权模型

`/exec` 命令只对**授权发送者**生效（即通过通道允许列表/配对且启用了 `commands.useAccessGroups` 的用户）。它只更新**会话状态**，不会写入配置文件。如果你想完全禁用 `exec`，需要通过工具策略拒绝（设置 `tools.deny: ["exec"]` 或按智能体配置）。除非你显式设置 `security=full` 和 `ask=off`，否则主机审批策略仍然有效。

## Exec 审批（伴侣应用 / 节点主机）

沙盒中的智能体可以配置为在网关或节点主机上执行命令前需要审批。详见 [Exec 审批](./exec-approvals.md)了解策略、允许列表和审批流程。

当需要审批时，`exec` 工具会立即返回 `status: "approval-pending"` 状态和一个审批 ID。审批通过（或被拒绝/超时）后，网关会发送系统事件通知（`Exec finished` / `Exec denied`）。如果命令在 `tools.exec.approvalRunningNoticeMs` 配置的时间后仍在运行，会发出一条 `Exec running` 通知。

## 允许列表与安全二进制文件

手动允许列表只匹配**解析后的二进制路径**（不支持基本名称匹配）。当设置 `security=allowlist` 时，shell 命令只有在管道的每一段都已加入允许列表或是安全二进制文件时才会自动放行。命令链接（`;`、`&&`、`||`）和重定向在允许列表模式下会被拒绝，除非每个顶级段都满足允许列表条件（包括安全二进制文件）。重定向目前不受支持。

`autoAllowSkills` 是 exec 审批中的一个独立便捷功能，与手动路径允许列表不同。如果你需要严格的显式信任控制，请保持 `autoAllowSkills` 禁用。

这两个控制项用于不同的场景：

- `tools.exec.safeBins`：小型、仅从标准输入读取的流式过滤工具
- `tools.exec.safeBinTrustedDirs`：为安全二进制文件的可执行路径额外指定信任目录
- `tools.exec.safeBinProfiles`：为自定义安全二进制文件配置 argv 策略
- 允许列表：对可执行路径的显式信任

**注意**：不要把 `safeBins` 当作通用允许列表使用，也不要把解释器或运行时二进制文件（如 `python3`、`node`、`ruby`、`bash`）加入其中。如果需要这些，请使用显式允许列表条目并保持审批提示启用。

`openclaw security audit` 命令会在解释器/运行时类型的 `safeBins` 条目缺少显式配置文件时发出警告，`openclaw doctor --fix` 可以帮你生成缺失的自定义 `safeBinProfiles` 条目。完整的策略说明和示例请参阅 [Exec 审批](./exec-approvals.md#safe-bins-stdin-only)和[安全二进制文件与允许列表](./exec-approvals.md#safe-bins-versus-allowlist)。

## 使用示例

前台执行：

```json
{ "tool": "exec", "command": "ls -la" }
```

后台执行 + 轮询：

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

发送按键（tmux 风格）：

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

提交（仅发送回车）：

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

粘贴（默认使用括号模式）：

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch（实验性功能）

`apply_patch` 是 `exec` 的一个子工具，用于结构化的多文件编辑。需要显式启用：

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

注意事项：

- 仅支持 OpenAI/OpenAI Codex 模型
- 工具策略仍然适用；`allow: ["exec"]` 会隐式允许 `apply_patch`
- 配置项位于 `tools.exec.applyPatch` 下
- `tools.exec.applyPatch.workspaceOnly` 默认为 `true`（限制在工作区内）。只有在你确实需要 `apply_patch` 在工作区目录外写入或删除文件时，才应设为 `false`

[提升模式](./elevated.md)[Exec 审批](./exec-approvals.md)