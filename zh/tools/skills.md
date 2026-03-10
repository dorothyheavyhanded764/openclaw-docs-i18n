

  技能（skills）

  
# 技能（skills）

OpenClaw 采用 **[AgentSkills](https://agentskills.io) 兼容**的技能文件夹来教会智能体（agent）如何使用工具。每个技能（skills）就是一个目录，里面放着 `SKILL.md` 文件，包含 YAML 前置元数据和具体的使用说明。OpenClaw 会加载**内置技能**，同时支持本地覆盖，并在加载时根据运行环境、配置项和二进制依赖自动过滤。

## 存放位置与优先级

技能从**三个地方**加载：

1. **内置技能**：随安装包一起提供（npm 包或 OpenClaw.app）
2. **托管/本地技能**：`~/.openclaw/skills`
3. **工作区技能**：`/skills`

如果出现同名技能，优先级依次是：`/skills`（最高）→ `~/.openclaw/skills` → 内置技能（最低）。你还可以通过 `~/.openclaw/openclaw.json` 中的 `skills.load.extraDirs` 配置额外的技能目录（优先级最低）。

## 独享技能 vs 共享技能

在**多智能体**环境下，每个智能体都有自己专属的工作区。这意味着：

- **独享技能**放在该智能体的 `/skills` 目录，只有它能看到。
- **共享技能**放在 `~/.openclaw/skills`（托管/本地），同一台机器上的**所有智能体**都能访问。
- **共享目录**也可以通过 `skills.load.extraDirs` 添加（优先级最低），方便多个智能体共用一套技能包。

如果同一个技能名称出现在多个位置，还是按优先级来：工作区优先，其次是托管/本地，最后是内置。

## 插件与技能

插件也可以自带技能——只要在 `openclaw.plugin.json` 里列出 `skills` 目录（路径相对于插件根目录）。插件启用后，这些技能就会加载进来，同样遵循优先级规则。你还能通过插件配置项上的 `metadata.openclaw.requires.config` 对它们进行门控。更多内容参见[插件](./plugin.md)和[工具](../tools.md)。

## ClawHub（安装与同步）

ClawHub 是 OpenClaw 的公共技能仓库，访问 [https://clawhub.com](https://clawhub.com) 即可浏览。你可以用它来发现、安装、更新和备份技能。完整指南请看 [ClawHub](./clawhub.md)。常用操作：

- 把技能安装到工作区：
  - `clawhub install <skill-slug>`
- 更新所有已安装的技能：
  - `clawhub update --all`
- 同步（扫描并发布更新）：
  - `clawhub sync --all`

默认情况下，`clawhub` 会把技能安装到当前工作目录下的 `./skills`（如果不在工作区，则回退到配置的 OpenClaw 工作区）。OpenClaw 在下一次会话启动时会自动识别为 `/skills`。

## 安全须知

- 把第三方技能当作**不可信代码**对待——启用前务必仔细阅读。
- 对于不可信输入或高风险工具，优先在沙盒中运行。参见[沙盒化](../gateway/sandboxing.md)。
- 工作区和额外目录的技能发现，只接受解析后的真实路径仍在配置根目录内的技能目录和 `SKILL.md` 文件。
- `skills.entries.*.env` 和 `skills.entries.*.apiKey` 会把密钥注入到该智能体轮次的**主机**进程中（不是沙盒）。千万别把密钥写进提示词或日志里。
- 更全面的威胁模型和检查清单，参见[安全](../gateway/security.md)。

## 格式规范（AgentSkills + Pi 兼容）

`SKILL.md` 至少要包含：

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

几点注意：

- 我们遵循 AgentSkills 规范的布局和意图定义。
- 内嵌智能体使用的解析器只支持**单行**前置元数据键。
- `metadata` 必须是**单行 JSON 对象**。
- 在说明文字里用 `{baseDir}` 可以引用技能文件夹的路径。
- 可选的前置元数据键：
  - `homepage` — 在 macOS 技能界面中显示为"网站"的 URL（也可以通过 `metadata.openclaw.homepage` 设置）。
  - `user-invocable` — `true|false`（默认 `true`）。设为 `true` 时，该技能会作为用户斜杠命令暴露出来。
  - `disable-model-invocation` — `true|false`（默认 `false`）。设为 `true` 时，该技能不会出现在模型提示词里（但仍可通过用户调用使用）。
  - `command-dispatch` — `tool`（可选）。设为 `tool` 时，斜杠命令会绕过模型，直接分派给工具。
  - `command-tool` — 当 `command-dispatch: tool` 时要调用的工具名称。
  - `command-arg-mode` — `raw`（默认）。对于工具分派，会把原始参数字符串直接传给工具（不做核心解析）。工具调用时的参数格式为：`{ command: "", commandName: "", skillName: "" }`。

## 门控机制（加载时过滤）

OpenClaw 会在**加载时根据 `metadata` 过滤技能**（单行 JSON）：

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

`metadata.openclaw` 下支持这些字段：

- `always: true` — 始终包含该技能（跳过其他门控条件）。
- `emoji` — macOS 技能界面使用的可选表情符号。
- `homepage` — 在 macOS 技能界面中显示为"网站"的可选 URL。
- `os` — 可选平台列表（`darwin`、`linux`、`win32`）。设置后，该技能只在这些操作系统上可用。
- `requires.bins` — 列表；每个都必须在 `PATH` 中存在。
- `requires.anyBins` — 列表；至少有一个要在 `PATH` 中存在。
- `requires.env` — 列表；环境变量必须存在**或者**在配置中提供。
- `requires.config` — `openclaw.json` 中必须为真值的路径列表。
- `primaryEnv` — 与 `skills.entries..apiKey` 关联的环境变量名。
- `install` — macOS 技能界面使用的可选安装规范数组（brew/node/go/uv/download）。

关于沙盒的一点说明：

- `requires.bins` 是在技能加载时于**主机**上检查的。
- 如果智能体在沙盒中运行，容器内也必须有对应的二进制文件。可以通过 `agents.defaults.sandbox.docker.setupCommand`（或自定义镜像）来安装。`setupCommand` 会在容器创建后执行一次。包安装还需要网络出口、可写的根文件系统以及沙盒中的 root 用户。比如 `summarize` 技能（`skills/summarize/SKILL.md`）就需要沙盒容器里有 `summarize` CLI 才能运行。

安装规范示例：

```
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

注意事项：

- 如果列出了多个安装选项，网关会**只选一个**首选方案（有 brew 就用 brew，否则用 node）。
- 如果所有安装选项都是 `download`，OpenClaw 会列出每个条目，让你看到所有可用的工件。
- 安装规范可以包含 `os: ["darwin"|"linux"|"win32"]` 来按平台筛选。
- Node 安装会遵循 `openclaw.json` 中的 `skills.install.nodeManager`（默认 npm；可选 npm/pnpm/yarn/bun）。这只影响**技能安装**；网关运行时仍建议用 Node（Bun 不推荐用于 WhatsApp/Telegram）。
- Go 安装：如果没有 `go` 但有 `brew`，网关会先用 Homebrew 安装 Go，并尽可能把 `GOBIN` 设成 Homebrew 的 `bin`。
- 下载安装的字段：`url`（必需）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（默认：检测到压缩包时自动解压）、`stripComponents`、`targetDir`（默认 `~/.openclaw/tools/`）。

如果没有 `metadata.openclaw`，该技能就始终可用（除非在配置中禁用，或内置技能被 `skills.allowBundled` 阻止）。

## 配置覆盖（~/.openclaw/openclaw.json）

你可以开关内置/托管技能，并为它们提供环境变量：

```json
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // 或直接写明文字符串
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

注意：如果技能名称里有连字符，要用引号把键名括起来（JSON5 支持引号键）。配置键默认匹配**技能名称**。如果技能定义了 `metadata.openclaw.skillKey`，则在 `skills.entries` 下用那个键。规则如下：

- `enabled: false` 会禁用该技能，即使它是内置或已安装的。
- `env`：只有在进程里还没设置该变量时才会注入。
- `apiKey`：为声明了 `metadata.openclaw.primaryEnv` 的技能提供便利。支持明文字符串或 SecretRef 对象（`{ source, provider, id }`）。
- `config`：用于存放自定义的每技能字段；自定义键必须放这里。
- `allowBundled`：仅针对**内置**技能的可选白名单。设置后，只有列表里的内置技能可用（托管/工作区技能不受影响）。

## 环境变量注入（每次智能体运行）

每次智能体开始运行时，OpenClaw 会：

1. 读取技能元数据。
2. 把 `skills.entries..env` 或 `skills.entries..apiKey` 应用到 `process.env`。
3. 用**有资格的**技能构建系统提示词。
4. 运行结束后恢复原始环境。

这是**限定在单次智能体运行范围内**的，不是全局 shell 环境。

## 会话快照（性能优化）

OpenClaw 会在**会话启动时**对有资格的技能做快照，同一会话后续的轮次会复用这个列表。技能或配置的改动要到下一次新会话才会生效。不过，如果开启了技能监视器，或者出现了新的符合条件的远程节点（见下文），技能也可以在会话中刷新。可以理解为**热重载**：刷新后的列表会在下一个智能体轮次生效。

## 远程 macOS 节点（Linux 网关）

如果网关跑在 Linux 上，但连着**macOS 节点**且**允许 `system.run`**（执行审批安全策略没设成 `deny`），那么当所需二进制文件在该节点上存在时，OpenClaw 可以把仅限 macOS 的技能也视为可用。智能体应该通过 `nodes` 工具（通常是 `nodes.run`）来执行这些技能。这依赖于节点报告其命令支持，以及通过 `system.run` 进行的二进制探测。如果 macOS 节点后来离线了，技能仍然可见，但调用可能会失败，直到节点重新连上。

## 技能监视器（自动刷新）

默认情况下，OpenClaw 会监视技能文件夹，在 `SKILL.md` 文件变更时更新技能快照。你可以在 `skills.load` 下配置：

```json
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Token 消耗（技能列表）

当有技能符合条件时，OpenClaw 会把一个紧凑的可用技能 XML 列表注入系统提示词（通过 `pi-coding-agent` 里的 `formatSkillsForPrompt`）。消耗是可预测的：

- **基础开销（只要有 ≥1 个技能）：** 195 个字符。
- **每个技能：** 97 个字符 + XML 转义后的 ``、`` 和 `` 值的长度。

计算公式（字符数）：

```bash
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

几点说明：

- XML 转义会把 `& < > " '` 变成实体（`&amp;`、`<` 等），增加长度。
- Token 数量取决于模型的分词器。粗略按 OpenAI 风格估算约 4 字符/Token，所以**97 字符 ≈ 24 Token** 每个技能，再加上你实际字段的长度。

## 托管技能的生命周期

OpenClaw 作为安装包（npm 包或 OpenClaw.app）的一部分，内置了一套基准技能。`~/.openclaw/skills` 用于本地覆盖——比如你想固定或修补某个技能，又不想动内置的副本。工作区技能则完全由你掌控，遇到同名时会覆盖前两者。

## 配置参考

完整的配置模式请参阅[技能配置](./skills-config.md)。

## 想要更多技能？

去 [https://clawhub.com](https://clawhub.com) 逛逛吧。

* * *

[斜杠命令](./slash-commands.md)[技能配置](./skills-config.md)