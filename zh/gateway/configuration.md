

  配置与运维

  
# 配置

网关（Gateway）会从 `~/.openclaw/openclaw.json` 读取一个可选的 **JSON5** 配置文件。如果没有这个文件，也没关系——OpenClaw 会使用安全的默认值运行。

你通常会因为以下原因需要添加或修改配置：

- 连接各种频道，控制谁能给机器人发消息
- 设置模型、工具、沙箱或自动化功能（定时任务、钩子（hooks））
- 调整会话、媒体、网络或界面相关的参数

完整的字段说明请查阅[配置参考文档](./configuration-reference.md)。

> **💡** **第一次接触配置？** 建议先运行 `openclaw onboard` 走一遍交互式设置流程，或者直接查看[配置示例](./configuration-examples.md)页面，里面有可以直接复制使用的完整配置。

## 最小配置

一个最简单的配置示例：

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## 编辑配置的方式

```bash
openclaw onboard       # 完整设置向导
openclaw configure     # 配置向导
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

打开 [http://127.0.0.1:18789](http://127.0.0.1:18789)，切换到 **Config** 标签页。控制界面会根据配置模式自动渲染表单，同时也提供了 **Raw JSON** 编辑器作为备选方案。

直接编辑 `~/.openclaw/openclaw.json` 文件即可。网关（Gateway）会监视该文件，修改后自动生效（详见[热重载](#config-hot-reload)）。

## 严格验证机制

> **⚠️** OpenClaw 对配置有严格要求：必须完全符合预定义的模式。如果存在未知的键、格式错误的类型或无效的值，网关（Gateway）会**拒绝启动**。唯一的例外是根级别的 `$schema` 字段（字符串类型），这是为了让编辑器能够附加 JSON Schema 元数据。

当验证失败时：

- 网关（Gateway）不会启动
- 只有诊断命令可以使用（`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`）
- 运行 `openclaw doctor` 查看具体问题
- 运行 `openclaw doctor --fix`（或 `--yes`）自动修复

## 常见配置任务

每个频道都有自己专属的配置段，位于 `channels.` 下。具体设置步骤请查看对应的频道文档：

- [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
- [Telegram](../channels/telegram.md) — `channels.telegram`
- [Discord](../channels/discord.md) — `channels.discord`
- [Slack](../channels/slack.md) — `channels.slack`
- [Signal](../channels/signal.md) — `channels.signal`
- [iMessage](../channels/imessage.md) — `channels.imessage`
- [Google Chat](../channels/googlechat.md) — `channels.googlechat`
- [Mattermost](../channels/mattermost.md) — `channels.mattermost`
- [MS Teams](../channels/msteams.md) — `channels.msteams`

所有频道的私信访问控制都使用相同的模式：

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // 仅用于 allowlist/open 模式
    },
  },
}
```

想指定主要使用的模型和备用模型？这样配置：

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

几点说明：

- `agents.defaults.models` 定义了模型目录，同时也是 `/model` 命令的允许列表
- 模型引用格式为 `provider/model`（例如 `anthropic/claude-opus-4-6`）
- `agents.defaults.imageMaxDimensionPx` 控制转录/工具图像的缩放（默认 `1200`）；调低这个值通常能减少截图较多时的视觉 token 消耗
- 想在聊天中切换模型，看[模型 CLI](../concepts/models.md)；想了解认证轮换和备用模型行为，看[模型故障转移](../concepts/model-failover.md)
- 如果要用自定义或自托管的提供商，参考配置参考文档中的[自定义提供商](./configuration-reference.md#custom-providers-and-base-urls)

私信访问权限由每个频道的 `dmPolicy` 字段控制：

- `"pairing"`（默认）：陌生发送者会收到一个一次性配对码，需要批准才能继续
- `"allowlist"`：只允许 `allowFrom` 列表中的发送者（或已配对的用户）
- `"open"`：允许所有私信（需要设置 `allowFrom: ["*"]`）
- `"disabled"`：忽略所有私信

群组访问控制则使用 `groupPolicy` + `groupAllowFrom` 或频道专属的允许列表。查看[配置参考文档](./configuration-reference.md#dm-and-group-access)了解各频道的详细设置。

群组消息默认**需要被提及才会响应**。你可以为每个智能体配置触发模式：

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

触发方式有两种：

- **元数据提及**：原生的 @提及功能（WhatsApp 的点击提及、Telegram 的 @bot 等）
- **文本模式**：`mentionPatterns` 中定义的正则表达式

查看[配置参考文档](./configuration-reference.md#group-chat-mention-gating)了解各频道的覆盖规则和自聊天模式。

会话设置决定了对话的连续性和隔离方式：

```json
{
  session: {
    dmScope: "per-channel-peer",  // 多用户场景推荐
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

关键参数说明：

- `dmScope`：可选值有 `main`（共享）、`per-peer`、`per-channel-peer`、`per-account-channel-peer`
- `threadBindings`：线程绑定会话路由的全局默认值（Discord 支持 `/focus`、`/unfocus`、`/agents`、`/session idle` 和 `/session max-age` 命令）
- 更多内容看[会话管理](../concepts/session.md)了解作用域、身份关联和发送策略
- 完整字段说明看[配置参考文档](./configuration-reference.md#session)

想让智能体会话在隔离的 Docker 容器中运行？这样配置：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

使用前需要先构建镜像：`scripts/sandbox-setup.sh`

完整指南看[沙箱](./sandboxing.md)，所有选项看[配置参考文档](./configuration-reference.md#sandbox)。

想让机器人定期主动发消息给你？配置心跳：

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

参数说明：

- `every`：时间间隔字符串（如 `30m`、`2h`）。设为 `0m` 可禁用
- `target`：目标频道，可选 `last`、`whatsapp`、`telegram`、`discord`、`none`
- `directPolicy`：针对私信类心跳目标，可选 `allow`（默认）或 `block`

完整指南看[心跳](./heartbeat.md)。

设置定时执行的任务：

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

参数说明：

- `sessionRetention`：从 `sessions.json` 中清理已完成的隔离运行会话（默认 `24h`；设为 `false` 可禁用清理）
- `runLog`：按大小和保留行数清理 `cron/runs/.jsonl` 日志

功能概述和 CLI 示例看[定时任务](../automation/cron-jobs.md)。

想在网关（Gateway）上启用 HTTP webhook 端点？这样配置：

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

安全提醒：

- 把所有钩子/webhook 的有效载荷内容都当作不可信输入来处理
- 不安全内容绕过标志默认应该保持禁用状态（`hooks.gmail.allowUnsafeExternalContent`、`hooks.mappings[].allowUnsafeExternalContent`），除非你在做严格限定范围的调试
- 对于钩子驱动的智能体，建议使用较强的现代模型级别和严格的工具策略（比如只允许消息传递功能，并在可能的情况下启用沙箱）

所有映射选项和 Gmail 集成看[配置参考文档](./configuration-reference.md#hooks)。

想同时运行多个独立的智能体，各自有独立的工作空间和会话？这样配置：

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

绑定规则和每个智能体的访问配置看[多智能体](../concepts/multi-agent.md)和[配置参考文档](./configuration-reference.md#multi-agent-routing)。

配置文件越来越大？可以用 `$include` 把它拆分成多个文件：

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

规则说明：

- **单个文件**：替换包含它的那个对象
- **文件数组**：按顺序深度合并（后面的覆盖前面的）
- **同级键**：在引入完成后合并（会覆盖被引入的值）
- **嵌套引入**：最多支持 10 层深度
- **相对路径**：相对于引入它的文件来解析
- **错误处理**：对缺失文件、解析错误和循环引用都会给出清晰的错误提示

## 配置热重载

网关（Gateway）会监视 `~/.openclaw/openclaw.json` 文件，修改后自动应用——大多数设置都不需要手动重启。

### 重载模式

| 模式 | 行为 |
| --- | --- |
| **`hybrid`**（默认） | 安全的修改立即热应用。需要重启的修改会自动触发重启。 |
| **`hot`** | 只热应用安全的修改。需要重启时会记录警告——你需要手动处理。 |
| **`restart`** | 任何配置修改都会重启网关（Gateway），无论是否安全。 |
| **`off`** | 禁用文件监视。修改要等到下次手动重启才生效。 |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### 哪些能热应用，哪些需要重启

大多数字段都能热应用，无需停机。在 `hybrid` 模式下，需要重启的修改会自动处理。

| 类别 | 字段 | 需要重启？ |
| --- | --- | --- |
| 频道 | `channels.*`、`web`（WhatsApp）—— 所有内置和扩展频道 | 否 |
| 智能体与模型 | `agent`、`agents`、`models`、`routing` | 否 |
| 自动化 | `hooks`、`cron`、`agent.heartbeat` | 否 |
| 会话与消息 | `session`、`messages` | 否 |
| 工具与媒体 | `tools`、`browser`、`skills`、`audio`、`talk` | 否 |
| 界面与杂项 | `ui`、`logging`、`identity`、`bindings` | 否 |
| 网关服务器 | `gateway.*`（端口、绑定、认证、tailscale、TLS、HTTP） | **是** |
| 基础设施 | `discovery`、`canvasHost`、`plugins` | **是** |

> **ℹ️** `gateway.reload` 和 `gateway.remote` 是例外——修改它们**不会**触发重启。

## 配置 RPC（编程式更新）

> **ℹ️** 控制平面写入 RPC（`config.apply`、`config.patch`、`update.run`）有速率限制：每个 `deviceId+clientIp` 每 60 秒最多 3 个请求。超限时，RPC 会返回 `UNAVAILABLE` 并附带 `retryAfterMs` 字段。

一步完成验证、写入完整配置并重启网关（Gateway）。

`config.apply` 会**替换整个配置**。如果只想改部分内容，用 `config.patch`；如果只改单个键，用 `openclaw config set`。

参数：

- `raw`（字符串）—— 完整配置的 JSON5 内容
- `baseHash`（可选）—— 从 `config.get` 获取的配置哈希（配置已存在时必需）
- `sessionKey`（可选）—— 重启后唤醒 ping 使用的会话键
- `note`（可选）—— 重启标记的备注
- `restartDelayMs`（可选）—— 重启前的延迟时间（默认 2000 毫秒）

当已有重启请求在等待或执行中时，新的请求会被合并。每次重启周期之间有 30 秒冷却时间。

```bash
openclaw gateway call config.get --params '{}'  # 获取 payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

将部分更新合并到现有配置中（JSON merge patch 语义）：

- 对象递归合并
- `null` 删除对应的键
- 数组直接替换

参数：

- `raw`（字符串）—— 只包含要修改的键的 JSON5 内容
- `baseHash`（必需）—— 从 `config.get` 获取的配置哈希
- `sessionKey`、`note`、`restartDelayMs` —— 同 `config.apply`

重启行为与 `config.apply` 相同：合并等待中的重启请求，每次重启周期之间有 30 秒冷却。

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## 环境变量

OpenClaw 会从以下位置读取环境变量：

- 父进程的环境变量
- 当前工作目录下的 `.env` 文件（如果存在）
- `~/.openclaw/.env`（全局回退）

这些文件不会覆盖已存在的环境变量。你也可以直接在配置文件中设置环境变量：

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

如果启用了这个功能，且预期的环境变量没有设置，OpenClaw 会运行你的登录 shell 并只导入缺失的变量：

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

也可以用环境变量开启：`OPENCLAW_LOAD_SHELL_ENV=1`

在配置的任意字符串值中，可以用 `${VAR_NAME}` 引用环境变量：

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

替换规则：

- 只匹配大写名称：`[A-Z_][A-Z0-9_]*`
- 缺失或为空的变量会在加载时报错
- 用 `$${VAR}` 转义，输出字面值
- 在 `$include` 引入的文件中同样有效
- 支持内联替换：`"${BASE}/v1"` → `"https://api.example.com/v1"`

支持 SecretRef 对象的字段，可以用以下方式引用密钥：

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

SecretRef 的详细说明（包括用于 `env`/`file`/`exec` 的 `secrets.providers`）请看[密钥管理](./secrets.md)。支持的凭据路径列表见 [SecretRef 凭据范围](../reference/secretref-credential-surface.md)。

 完整的优先级和来源说明请看[环境变量](../help/environment.md)。

## 完整参考

所有字段的详细说明请查阅**[配置参考文档](./configuration-reference.md)**。

* * *

*相关内容：[配置示例](./configuration-examples.md) · [配置参考文档](./configuration-reference.md) · [Doctor 诊断工具](./doctor.md)*

[网关运行手册](../gateway.md)[配置参考文档](./configuration-reference.md)