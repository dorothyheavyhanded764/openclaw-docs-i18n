

  基础概念

  
# 智能体工作空间

工作空间（workspace）是智能体（agent）的「家」——它是智能体使用文件工具时唯一的工作目录，也是理解上下文的核心场所。请把它当作私密的「记忆库」来对待。

需要注意的是，工作空间与 `~/.openclaw/` 是分开的：后者存储配置、凭据和会话数据。**关键区别：** 工作空间是**默认的工作目录**，并不是一个严格的沙箱。工具会以工作空间为基准解析相对路径，但绝对路径仍然可以访问主机上的其他位置——除非你启用了沙箱隔离。

如果你需要严格的隔离环境，请使用 [`agents.defaults.sandbox`](../gateway/sandboxing.md)（或为每个智能体单独配置沙箱）。当沙箱启用且 `workspaceAccess` 不是 `"rw"` 时，工具会在 `~/.openclaw/sandboxes` 下的沙箱工作空间内运行，而非你的主机工作空间。

## 默认位置

工作空间默认存放在以下位置：

-   默认路径：`~/.openclaw/workspace`
-   如果设置了 `OPENCLAW_PROFILE` 环境变量且值不为 `"default"`，则默认路径变为 `~/.openclaw/workspace-`
-   你也可以在 `~/.openclaw/openclaw.json` 中自定义：

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

当你运行 `openclaw onboard`、`openclaw configure` 或 `openclaw setup` 时，OpenClaw 会自动创建工作空间并生成引导文件（如果文件不存在）。沙箱种子复制只会接受工作空间内的常规文件——解析到工作空间外部的符号链接或硬链接会被忽略。

如果你已经自行管理这些文件，可以禁用引导文件的自动创建：

```json
{ agent: { skipBootstrap: true } }
```

## 多余的工作空间目录

早期版本的安装可能会创建 `~/openclaw` 目录。保留多个工作空间目录容易造成认证混乱或状态不同步，因为同一时间只有一个工作空间是活动的。

**建议：** 只保留一个活动的工作空间。如果不再使用多余的目录，可以归档或删除（例如 `trash ~/openclaw`）。如果你确实需要维护多个工作空间，请确保 `agents.defaults.workspace` 指向当前活动的那个。`openclaw doctor` 检测到多余的工作空间目录时会发出警告。

## 工作空间文件一览

OpenClaw 期望在工作空间中找到以下标准文件。每个文件都有特定的用途：

### 核心配置文件

-   `AGENTS.md` — 智能体的操作指南，定义它应该如何使用记忆。每次会话开始时加载。适合放置规则、优先级和「行为准则」。
-   `SOUL.md` — 人格设定、语气风格和边界。每次会话加载。
-   `USER.md` — 用户画像：你是谁，智能体该如何称呼你。每次会话加载。
-   `IDENTITY.md` — 智能体的名字、风格和代表表情。在引导仪式期间创建和更新。

### 工具与自动化

-   `TOOLS.md` — 本地工具和约定的说明笔记。注意：这只是指导性文档，不会控制工具的实际可用性。
-   `HEARTBEAT.md` — 可选的心跳任务清单。保持简短，避免消耗过多 token。
-   `BOOT.md` — 可选的启动清单，当内部钩子启用时会在网关重启时执行。保持简短，使用消息工具进行外发通信。

### 初始化

-   `BOOTSTRAP.md` — 一次性首次运行仪式。只会在全新的工作空间中创建。仪式完成后可以删除。

### 记忆系统

-   `memory/YYYY-MM-DD.md` — 每日记忆日志，每天一个文件。建议会话开始时读取今天和昨天的日志。
-   `MEMORY.md`（可选）— 精选的长期记忆。只在主要的私有会话中加载（不在共享或群组上下文中）。

详细的工作流程和自动记忆清理机制，请参阅 [记忆](./memory.md)。

### 扩展目录

-   `skills/`（可选）— 工作空间专属技能。当名称冲突时，会覆盖托管/内置技能。
-   `canvas/`（可选）— Canvas UI 文件，用于节点显示（如 `canvas/index.html`）。

如果任何引导文件缺失，OpenClaw 会在会话中注入「文件缺失」标记并继续运行。过大的引导文件会被截断——你可以通过 `agents.defaults.bootstrapMaxChars`（默认 20000）和 `agents.defaults.bootstrapTotalMaxChars`（默认 150000）调整限制。`openclaw setup` 可以重新创建缺失的默认文件，不会覆盖已有文件。

## 什么不应该放在工作空间里

以下内容存放在 `~/.openclaw/` 下，**绝不应该**提交到工作空间仓库：

-   `~/.openclaw/openclaw.json` — 配置文件
-   `~/.openclaw/credentials/` — OAuth 令牌、API 密钥等凭据
-   `~/.openclaw/agents//sessions/` — 会话记录和元数据
-   `~/.openclaw/skills/` — 托管技能

如果需要迁移会话或配置，请单独复制，并确保它们不被纳入版本控制。

## Git 备份（强烈推荐）

把工作空间当作私密的「记忆库」，用 Git 进行版本备份和恢复是最佳实践。请将其放入一个**私有**仓库中。

以下操作请在运行网关的机器上执行（工作空间就存在于那台机器上）。

### 1) 初始化仓库

如果已安装 Git，全新的工作空间会自动初始化仓库。如果当前工作空间还不是 Git 仓库，请运行：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) 添加私有远程仓库

#### 选项 A：GitHub 网页界面

1.  在 GitHub 上创建一个新的**私有**仓库
2.  不要勾选「使用 README 初始化」（避免合并冲突）
3.  复制 HTTPS 远程 URL
4.  添加远程仓库并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

#### 选项 B：GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

#### 选项 C：GitLab 网页界面

1.  在 GitLab 上创建一个新的**私有**仓库
2.  不要使用 README 初始化
3.  复制 HTTPS 远程 URL
4.  添加远程仓库并推送：

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) 日常更新

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## 切勿提交机密信息

即使在私有仓库中，也要避免在工作空间中存储敏感信息：

-   API 密钥、OAuth 令牌、密码或其他凭据
-   `~/.openclaw/` 下的任何内容
-   聊天记录或敏感附件的原始数据

如果确实需要引用敏感信息，请使用占位符，将真实的机密存放在密码管理器、环境变量或 `~/.openclaw/` 中。

建议的 `.gitignore` 模板：

```
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## 迁移工作空间到新机器

1.  将仓库克隆到目标路径（默认为 `~/.openclaw/workspace`）
2.  在 `~/.openclaw/openclaw.json` 中设置 `agents.defaults.workspace` 指向该路径
3.  运行 `openclaw setup --workspace ` 生成缺失的文件
4.  如需迁移会话数据，请单独从旧机器复制 `~/.openclaw/agents//sessions/`

## 进阶说明

-   多智能体路由可以为不同智能体配置不同的工作空间。详见 [通道路由](../channels/channel-routing.md)。
-   如果启用了 `agents.defaults.sandbox`，非主会话可以使用 `agents.defaults.sandbox.workspaceRoot` 下的独立沙箱工作空间。

[上下文](./context.md)[OAuth](./oauth.md)