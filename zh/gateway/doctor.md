

  配置与运维

  
# Doctor

`openclaw doctor` 是 OpenClaw 的诊断修复与迁移工具。它能修复过时的配置和状态、检查系统健康状况，并提供可操作的修复建议。

## 快速开始

```bash
openclaw doctor
```

### 无界面模式 / 自动化脚本

```bash
openclaw doctor --yes
```

自动接受默认选项（包括适用的重启、服务、沙箱修复步骤），无需交互确认。

```bash
openclaw doctor --repair
```

自动应用推荐的修复措施（包括安全的修复和重启操作），无需交互确认。

```bash
openclaw doctor --repair --force
```

同时应用激进的修复措施（会覆盖自定义的 supervisor 配置）。

```bash
openclaw doctor --non-interactive
```

无交互运行，仅应用安全的迁移操作（配置规范化 + 磁盘状态移动）。跳过需要人工确认的重启、服务、沙箱操作。检测到遗留状态迁移时会自动执行。

```bash
openclaw doctor --deep
```

扫描系统服务以查找额外的网关安装（launchd/systemd/schtasks）。如果想在写入前预览更改，可以先查看配置文件：

```bash
cat ~/.openclaw/openclaw.json
```

## 功能一览

-   Git 源码安装的可选预检更新（仅交互模式）。
-   UI 协议版本检查（当协议 schema 更新时重建控制界面）。
-   健康检查 + 重启提示。
-   技能状态摘要（可用/缺失/被阻止）。
-   遗留配置值的规范化处理。
-   OpenCode Zen provider 覆盖警告（`models.providers.opencode`）。
-   遗留磁盘状态迁移（会话/智能体目录/WhatsApp 认证）。
-   状态完整性和权限检查（会话、转录、状态目录）。
-   本地运行时的配置文件权限检查（chmod 600）。
-   模型认证健康检查：检查 OAuth 过期情况，可刷新即将过期的令牌，报告认证配置的冷却/禁用状态。
-   额外工作空间目录检测（`~/openclaw`）。
-   启用沙箱时的沙箱镜像修复。
-   遗留服务迁移和额外网关检测。
-   网关运行时检查（服务已安装但未运行；缓存的 launchd 标签）。
-   频道状态警告（从运行中的网关探测）。
-   Supervisor 配置审计（launchd/systemd/schtasks）及可选修复。
-   网关运行时最佳实践检查（Node vs Bun、版本管理器路径）。
-   网关端口冲突诊断（默认 `18789`）。
-   开放私信策略的安全警告。
-   本地令牌模式的网关认证检查（无令牌源时提供生成选项；不覆盖 SecretRef 配置的令牌）。
-   Linux 上的 systemd linger 检查。
-   源码安装检查（pnpm workspace 不匹配、缺少 UI 资源、缺少 tsx 二进制）。
-   写入更新后的配置 + 向导元数据。

## 详细行为与原理

### 0) 可选更新（Git 安装）

如果是 Git 检出目录且 doctor 以交互模式运行，它会在执行诊断前询问是否更新（fetch/rebase/build）。

### 1) 配置规范化

如果配置文件包含遗留的值结构（例如没有频道覆盖的 `messages.ackReaction`），doctor 会将其规范化为当前的 schema 格式。

### 2) 遗留配置键迁移

当配置包含已弃用的键时，其他命令会拒绝运行并提示执行 `openclaw doctor`。Doctor 会：

-   说明发现了哪些遗留键。
-   展示将要应用的迁移。
-   用更新后的 schema 重写 `~/.openclaw/openclaw.json`。

网关启动时如果检测到遗留配置格式也会自动运行 doctor 迁移，因此过时配置无需手动干预即可修复。当前的迁移包括：

-   `routing.allowFrom` → `channels.whatsapp.allowFrom`
-   `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
-   `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
-   `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
-   `routing.queue` → `messages.queue`
-   `routing.bindings` → 顶级 `bindings`
-   `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
-   `routing.agentToAgent` → `tools.agentToAgent`
-   `routing.transcribeAudio` → `tools.media.audio.models`
-   `bindings[].match.accountID` → `bindings[].match.accountId`
-   对于有命名 `accounts` 但缺少 `accounts.default` 的频道，将账号级别的单账号频道值移入 `channels..accounts.default`（如果存在）
-   `identity` → `agents.list[].identity`
-   `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
-   `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
-   `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

Doctor 还会针对多账号频道的默认账号配置发出指导警告：

-   如果配置了 2 个或更多 `channels..accounts` 条目，但没有设置 `channels..defaultAccount` 或 `accounts.default`，doctor 会警告回退路由可能选择意外的账号。
-   如果 `channels..defaultAccount` 设置为未知的账号 ID，doctor 会警告并列出已配置的账号 ID。

### 2b) OpenCode Zen provider 覆盖

如果你手动添加了 `models.providers.opencode`（或 `opencode-zen`），它会覆盖 `@mariozechner/pi-ai` 提供的内置 OpenCode Zen 目录。这可能导致所有模型被强制路由到单一 API 或成本显示为零。Doctor 会发出警告，方便你移除覆盖并恢复按模型的 API 路由和成本计算。

### 3) 遗留状态迁移（磁盘布局）

Doctor 可以将较旧的磁盘布局迁移到当前结构：

-   会话存储 + 转录文件：
    -   从 `~/.openclaw/sessions/` 迁移到 `~/.openclaw/agents//sessions/`
-   智能体目录：
    -   从 `~/.openclaw/agent/` 迁移到 `~/.openclaw/agents//agent/`
-   WhatsApp 认证状态（Baileys）：
    -   从遗留的 `~/.openclaw/credentials/*.json`（`oauth.json` 除外）
    -   迁移到 `~/.openclaw/credentials/whatsapp//...`（默认账号 ID：`default`）

这些迁移是尽力而为且幂等的；当 doctor 留下任何遗留文件夹作为备份时会发出警告。网关/CLI 启动时也会自动迁移遗留的会话和智能体目录，因此历史记录/认证/模型信息会进入每个智能体的路径，无需手动运行 doctor。WhatsApp 认证故意只通过 `openclaw doctor` 迁移。

### 4) 状态完整性检查（会话持久化、路由和安全）

状态目录是整个系统的核心。如果它丢失，你将失去会话、凭证、日志和配置（除非你有备份）。Doctor 会检查：

-   **状态目录缺失**：警告灾难性的状态丢失，提示重建目录，并提醒无法恢复丢失的数据。
-   **状态目录权限**：验证可写性；提供修复权限的选项（检测到所有者/组不匹配时给出 `chown` 提示）。
-   **macOS 云同步状态目录**：当状态目录位于 iCloud Drive (`~/Library/Mobile Documents/com~apple~CloudDocs/...`) 或 `~/Library/CloudStorage/...` 下时发出警告，因为同步路径可能导致 I/O 变慢以及锁/同步竞争问题。
-   **Linux SD 卡或 eMMC 状态目录**：当状态目录解析到 `mmcblk*` 挂载源时发出警告，因为 SD 卡或 eMMC 的随机 I/O 在会话和凭证写入下可能较慢且磨损更快。
-   **会话目录缺失**：`sessions/` 和会话存储目录是持久化历史和避免 `ENOENT` 崩溃所必需的。
-   **转录文件不匹配**：当最近的会话条目缺少转录文件时发出警告。
-   **主会话"单行 JSONL"**：标记主转录文件只有一行的情况（历史记录未正常累积）。
-   **多个状态目录**：当多个 `~/.openclaw` 文件夹存在于不同主目录下，或 `OPENCLAW_STATE_DIR` 指向其他位置时发出警告（历史记录可能在安装之间分散）。
-   **远程模式提醒**：如果 `gateway.mode=remote`，doctor 会提醒在远程主机上运行（状态在那里）。
-   **配置文件权限**：如果 `~/.openclaw/openclaw.json` 对组/其他用户可读，发出警告并提供收紧权限至 `600` 的选项。

### 5) 模型认证健康（OAuth 过期）

Doctor 检查认证存储中的 OAuth 配置，在令牌即将过期或已过期时发出警告，并可在安全情况下刷新它们。如果 Anthropic Claude Code 配置已过时，会建议运行 `claude setup-token`（或粘贴 setup-token）。刷新提示仅在交互式终端（TTY）中出现；`--non-interactive` 会跳过刷新尝试。Doctor 还会报告因以下原因暂时不可用的认证配置：

-   短期冷却（速率限制/超时/认证失败）
-   长期禁用（计费/额度失败）

### 6) Hooks 模型验证

如果设置了 `hooks.gmail.model`，doctor 会根据目录和允许列表验证模型引用，在无法解析或被禁止时发出警告。

### 7) 沙箱镜像修复

启用沙箱时，doctor 检查 Docker 镜像，在当前镜像缺失时提供构建或切换到遗留名称的选项。

### 8) 网关服务迁移和清理提示

Doctor 检测遗留的网关服务（launchd/systemd/schtasks），提供移除它们并使用当前网关端口安装 OpenClaw 服务的选项。还可以扫描额外的网关类服务并打印清理提示。以 profile 命名的 OpenClaw 网关服务被视为一等公民，不会被标记为"额外"。

### 9) 安全警告

当 provider 对私信开放且没有允许列表，或策略配置存在安全风险时，Doctor 会发出警告。

### 10) systemd linger（Linux）

如果作为 systemd 用户服务运行，doctor 确保启用了 lingering，以便网关在注销后仍保持运行。

### 11) 技能状态

Doctor 为当前工作空间打印一份可用/缺失/被阻止技能的快速摘要。

### 12) 网关认证检查（本地令牌）

Doctor 检查本地网关令牌认证的准备情况。

-   如果令牌模式需要令牌且不存在令牌源，doctor 提供生成选项。
-   如果 `gateway.auth.token` 由 SecretRef 管理但不可用，doctor 会警告但不会用明文覆盖它。
-   `openclaw doctor --generate-gateway-token` 仅在未配置令牌 SecretRef 时强制生成令牌。

### 12b) 只读 SecretRef 感知修复

某些修复流程需要在不影响运行时快速失败行为的前提下检查已配置的凭证。

-   `openclaw doctor --fix` 现在使用与 status 系列命令相同的只读 SecretRef 摘要模型进行针对性配置修复。
-   示例：Telegram `allowFrom` / `groupAllowFrom` `@username` 修复会尝试使用已配置的机器人凭证（如果可用）。
-   如果 Telegram 机器人令牌通过 SecretRef 配置但在当前命令路径中不可用，doctor 会报告凭证已配置但不可用，并跳过自动解析，而非崩溃或错误报告令牌缺失。

### 13) 网关健康检查 + 重启

Doctor 运行健康检查，在网关看起来不健康时提供重启选项。

### 14) 频道状态警告

如果网关健康，doctor 运行频道状态探测并报告带有建议修复方案的警告。

### 15) Supervisor 配置审计 + 修复

Doctor 检查已安装的 supervisor 配置（launchd/systemd/schtasks）是否存在缺失或过时的默认值（如 systemd 的 network-online 依赖和重启延迟）。发现不匹配时会建议更新，并可将服务文件/任务重写为当前默认值。注意：

-   `openclaw doctor` 重写 supervisor 配置前会提示确认。
-   `openclaw doctor --yes` 接受默认的修复提示。
-   `openclaw doctor --repair` 无需提示自动应用推荐的修复。
-   `openclaw doctor --repair --force` 覆盖自定义的 supervisor 配置。
-   如果令牌认证需要令牌且 `gateway.auth.token` 由 SecretRef 管理，doctor 服务安装/修复会验证 SecretRef 但不会将解析出的明文令牌持久化到 supervisor 服务环境元数据中。
-   如果令牌认证需要令牌且配置的令牌 SecretRef 未解析，doctor 会阻止安装/修复并提供可操作的指导。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，doctor 会阻止安装/修复直到明确设置模式。
-   始终可通过 `openclaw gateway install --force` 强制完全重写。

### 16) 网关运行时 + 端口诊断

Doctor 检查服务运行时（PID、上次退出状态），在服务已安装但实际未运行时发出警告。还会检查网关端口（默认 `18789`）的端口冲突，并报告可能的原因（网关已在运行、SSH 隧道占用等）。

### 17) 网关运行时最佳实践

Doctor 在网关服务运行于 Bun 或版本管理的 Node 路径（`nvm`、`fnm`、`volta`、`asdf` 等）时发出警告。WhatsApp + Telegram 频道需要 Node，而版本管理器路径在升级后可能失效，因为服务不会加载你的 shell 初始化脚本。当系统 Node 安装（Homebrew/apt/choco）可用时，doctor 提供迁移选项。

### 18) 配置写入 + 向导元数据

Doctor 持久化任何配置更改，并标记向导元数据以记录本次 doctor 运行。

### 19) 工作空间提示（备份 + 记忆系统）

工作空间缺失记忆系统时 Doctor 会给出建议，如果工作空间尚未使用 git 管理则打印备份提示。关于工作空间结构和 git 备份（推荐使用私有 GitHub 或 GitLab）的完整指南，请参阅 [/concepts/agent-workspace](../concepts/agent-workspace.md)。

[心跳](./heartbeat.md)[日志](./logging.md)