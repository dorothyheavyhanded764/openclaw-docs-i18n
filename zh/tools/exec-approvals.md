

  内置工具

  
# 执行审批（Exec Approvals）

当沙箱化的智能体需要在真实主机（`gateway` 或 `node`）上执行命令时，如何确保安全？执行审批就是为此设计的**安全联锁机制**——只有当策略规则、允许列表、以及（可选的）用户审批三者全部放行时，命令才会执行。

执行审批是工具策略和提升模式门控的**额外安全层**（除非提升模式设为 `full`，否则不会跳过审批）。最终生效的策略取 `tools.exec.*` 与审批默认值中**更严格**的那个；如果审批配置中某字段缺失，则沿用 `tools.exec` 的对应值。当配套应用 UI **不可达**时，任何需要用户确认的请求都会走**回退机制**（默认为拒绝）。

## 适用范围

执行审批在执行主机上本地强制执行：

- **网关主机** → 网关机器上的 `openclaw` 进程
- **节点主机** → 节点运行器（macOS 配套应用或无头节点主机）

macOS 上的分工：

- **节点主机服务**通过本地 IPC 将 `system.run` 请求转发给 **macOS 应用**
- **macOS 应用**负责执行审批，并在 UI 上下文中运行命令

## 配置与存储

审批配置存储在执行主机的本地 JSON 文件中：`~/.openclaw/exec-approvals.json`

示例结构：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 策略选项

### 安全级别（`exec.security`）

- `deny`：阻止所有主机执行请求
- `allowlist`：仅允许已加入允许列表的命令
- `full`：允许所有命令（等同于提升模式）

### 询问策略（`exec.ask`）

- `off`：从不弹出确认提示
- `on-miss`：仅当允许列表未匹配时才提示
- `always`：每个命令都要求确认

### 回退行为（`askFallback`）

当需要用户确认但 UI 不可达时，按此策略处理：

- `deny`：直接拒绝
- `allowlist`：仅当允许列表匹配时放行
- `full`：直接放行

## 允许列表（按智能体配置）

允许列表是**按智能体隔离**的。如果存在多个智能体，可以在 macOS 应用中切换编辑目标。匹配模式采用**不区分大小写的 glob 通配符**，且必须解析为**二进制文件的完整路径**（仅写文件名的条目会被忽略）。

旧的 `agents.default` 条目会在加载时自动迁移到 `agents.main`。

示例模式：

- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

每个允许列表条目会记录：

- `id`：用于 UI 标识的稳定 UUID（可选）
- `lastUsedAt`：最后使用时间戳
- `lastUsedCommand`：最后执行的命令
- `lastResolvedPath`：最后解析的路径

## 自动允许技能 CLI

开启**自动允许技能 CLI** 后，已知技能所引用的可执行文件会在节点（macOS 节点或无头节点主机）上自动视为已允许。该功能通过 Gateway RPC 调用 `skills.bins` 获取技能二进制列表。如果你需要严格的手动控制，请关闭此选项。

## 安全二进制（仅标准输入模式）

`tools.exec.safeBins` 定义了一组**仅从标准输入读取**的二进制文件（如 `jq`），它们可以在允许列表模式下运行，**无需**显式的允许列表条目。

安全二进制会拒绝位置文件参数和路径形式的标记，因此只能处理管道传入的数据流。验证逻辑仅基于命令行参数的形态进行判断（不检查主机文件系统），这样可以防止攻击者通过允许/拒绝的差异来探测文件是否存在。

对于默认的安全二进制，以下面向文件的选项会被拒绝：`sort -o`、`sort --output`、`sort --files0-from`、`sort --compress-program`、`wc --files0-from`、`jq -f/--from-file`、`grep -f/--file`。

安全二进制还会对可能破坏"仅标准输入"行为的选项执行严格的标志策略检查（如 `sort -o/--output/--compress-program` 和 `grep` 的递归标志）。

在执行时，安全二进制会将命令行参数视为**字面文本**，不进行通配符扩展和 `$VARS` 变量展开，因此 `*` 或 `$HOME/...` 这类模式无法被用来绕过限制读取文件。

安全二进制还必须从受信任的二进制目录解析（系统默认目录 + 网关进程启动时的 `PATH`），这可以阻止请求级别的 PATH 劫持攻击。

Shell 链和重定向在允许列表模式下不会自动放行。当每个顶级段落都满足允许列表条件（包括安全二进制或技能自动允许）时，Shell 链（`&&`、`||`、`;`）才被允许。重定向在允许列表模式下仍不支持。命令替换（`$()` 或反引号）在允许列表解析时会被拒绝，即使在双引号内也不例外；如需字面的 `$()` 文本，请使用单引号。

在 macOS 配套应用的审批中，如果原始 shell 文本包含控制或扩展语法（`&&`、`||`、`;`、`|`、反引号、`$`、`<`、`>`、`(`、`)`），将被视为允许列表未命中，除非 shell 二进制本身已加入允许列表。

默认安全二进制：`jq`、`cut`、`uniq`、`head`、`tail`、`tr`、`wc`。

`grep` 和 `sort` 不在默认列表中。如果启用它们，请为非标准输入工作流保留显式的允许列表条目。对于安全二进制模式下的 `grep`，请用 `-e`/`--regexp` 指定模式；位置模式形式会被拒绝，以防止文件操作数伪装成模式参数。

## 通过控制 UI 编辑

打开 **Control UI → Nodes → Exec approvals** 卡片，可以编辑默认配置、各智能体的覆盖设置和允许列表。

选择编辑范围（Defaults 或某个智能体），调整策略，添加或删除允许列表模式，然后点击**保存**。UI 会显示每个模式的**最后使用**信息，方便你清理不再需要的条目。

目标选择器可选择**网关**（本地审批）或某个**节点**。节点必须支持 `system.execApprovals.get/set`（macOS 应用或无头节点主机）。如果节点尚未支持执行审批，可以直接编辑其本地的 `~/.openclaw/exec-approvals.json` 文件。

命令行方式：`openclaw approvals` 支持网关或节点编辑，详见 [审批命令行](../cli/approvals.md)。

## 审批流程

当需要用户确认时，网关会向操作员客户端广播 `exec.approval.requested` 事件。Control UI 和 macOS 应用通过 `exec.approval.resolve` 处理确认，之后网关将获批的请求转发给节点主机执行。

需要审批时，执行工具会立即返回一个审批 ID。你可以用这个 ID 关联后续的系统事件（`Exec finished` 或 `Exec denied`）。如果在超时前没有收到决定，请求会被视为审批超时，并作为拒绝原因显示。

确认对话框包含以下信息：

- 命令及参数
- 当前工作目录（`cwd`）
- 智能体 ID
- 解析后的可执行文件路径
- 主机与策略元数据

操作选项：

- **允许一次** → 本次执行
- **始终允许** → 加入允许列表并执行
- **拒绝** → 阻止执行

## 转发审批到聊天频道

你可以将执行审批提示转发到任意聊天频道（包括插件频道），然后通过 `/approve` 命令进行审批。这使用正常的外发消息管道。

配置示例：

```json
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // 子字符串或正则表达式
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

在聊天中回复：

```bash
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### macOS IPC 流程

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

安全要点：

- Unix 套接字权限为 `0600`，令牌存储在 `exec-approvals.json` 中
- 同 UID 用户检查
- 质询/响应机制（随机数 + HMAC 令牌 + 请求哈希）+ 短 TTL

## 系统事件

执行生命周期会以系统消息形式呈现：

- `Exec running`（仅当命令运行时间超过通知阈值时）
- `Exec finished`
- `Exec denied`

这些消息在节点报告事件后发布到智能体的会话中。网关主机上的执行审批会在命令完成时（以及可选地在运行超时时）发送相同的事件。需要审批的执行会在这些消息中复用审批 ID 作为 `runId`，方便关联。

## 注意事项

- `full` 模式权限很大，建议优先使用允许列表
- `ask` 模式让你保持对智能体行为的掌控，同时支持快速审批
- 每个智能体独立的允许列表可以防止审批策略互相干扰
- 审批仅适用于**授权发送者**的主机执行请求，未授权者无法触发 `/exec`
- `/exec security=full` 是授权操作员的会话级便捷设置，设计上会跳过审批。如需彻底禁止主机执行，请将审批安全级别设为 `deny`，或通过工具策略禁用 `exec` 工具

相关文档：

- [执行工具](./exec.md)
- [提升模式](./elevated.md)
- [技能](./skills.md)

[执行工具](./exec.md)[Firecrawl](./firecrawl.md)