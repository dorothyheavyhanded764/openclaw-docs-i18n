

  智能体协调

  
# 多智能体沙盒与工具

## 概述

当你需要在同一个 OpenClaw 实例中运行多个智能体，并为它们设置不同的安全策略时，现在可以为每个智能体单独配置：

- **沙盒配置** — `agents.list[].sandbox` 会覆盖 `agents.defaults.sandbox`
- **工具限制** — 通过 `tools.allow` / `tools.deny` 以及 `agents.list[].tools` 控制

这意味着你可以轻松实现以下场景：

- 个人助手：拥有完整的系统访问权限
- 家庭/工作智能体：仅能使用受限的工具集
- 公开服务的智能体：运行在严格隔离的沙盒环境中

`setupCommand` 配置项位于 `sandbox.docker` 下（可以是全局配置或按智能体配置），它会在容器创建时执行一次。身份验证是按智能体隔离的：每个智能体从各自的 `agentDir` 目录读取认证信息，路径为：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

凭证**不会**在智能体之间共享。切勿让多个智能体使用同一个 `agentDir`。如果确实需要共享凭证，请手动将 `auth-profiles.json` 复制到另一个智能体的 `agentDir` 中。

想了解沙盒在运行时的具体行为？请参阅 [沙盒](../gateway/sandboxing.md)。遇到权限被阻止的问题需要调试？请参阅 [沙盒 vs 工具策略 vs 提升权限](../gateway/sandbox-vs-tool-policy-vs-elevated.md) 以及 `openclaw sandbox explain` 命令。

* * *

## 配置示例

### 示例 1：个人助手 + 受限家庭智能体

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "个人助手",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "家庭机器人",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**效果说明：**

- `main` 智能体：直接在主机上运行，拥有完整的工具访问权限
- `family` 智能体：在 Docker 容器中运行（每个智能体独占一个容器），仅能使用 `read` 工具

* * *

### 示例 2：工作智能体 + 共享沙盒

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### 示例 2b：全局编码配置 + 仅消息处理的智能体

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**效果说明：**

- 默认智能体获得编码相关工具
- `support` 智能体仅限消息处理功能（额外增加 Slack 工具）

* * *

### 示例 3：为不同智能体设置不同的沙盒模式

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // 全局默认值
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // 覆盖：main 智能体永不进入沙盒
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // 覆盖：public 智能体始终在沙盒中运行
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## 配置优先级

当全局配置 (`agents.defaults.*`) 与智能体特定配置 (`agents.list[].*`) 同时存在时，遵循以下规则：

### 沙盒配置

智能体级别的设置会覆盖全局设置：

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意：**

- 当沙盒的 `scope` 解析为 `"shared"` 时，`agents.list[].sandbox.{docker,browser,prune}.*` 的设置会被忽略。

### 工具限制

工具过滤按以下顺序逐层应用：

1. **工具配置文件** — `tools.profile` 或 `agents.list[].tools.profile`
2. **按提供商的工具配置文件** — `tools.byProvider[provider].profile` 或 `agents.list[].tools.byProvider[provider].profile`
3. **全局工具策略** — `tools.allow` / `tools.deny`
4. **按提供商的工具策略** — `tools.byProvider[provider].allow/deny`
5. **智能体级别的工具策略** — `agents.list[].tools.allow/deny`
6. **智能体按提供商的策略** — `agents.list[].tools.byProvider[provider].allow/deny`
7. **沙盒工具策略** — `tools.sandbox.tools` 或 `agents.list[].tools.sandbox.tools`
8. **子智能体工具策略** — `tools.subagents.tools`（如适用）

每一层都只能进一步限制工具，无法恢复之前层级已拒绝的工具。如果设置了 `agents.list[].tools.sandbox.tools`，它会完全替换该智能体的 `tools.sandbox.tools`。如果设置了 `agents.list[].tools.profile`，它会覆盖该智能体的 `tools.profile`。

提供商工具键支持 `provider`（如 `google-antigravity`）或 `provider/model`（如 `openai/gpt-5.2`）两种格式。

### 工具组（简写形式）

工具策略（全局、智能体、沙盒）支持 `group:*` 形式的简写，会自动展开为多个具体工具：

- `group:runtime`：`exec`、`bash`、`process`
- `group:fs`：`read`、`write`、`edit`、`apply_patch`
- `group:sessions`：`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
- `group:memory`：`memory_search`、`memory_get`
- `group:ui`：`browser`、`canvas`
- `group:automation`：`cron`、`gateway`
- `group:messaging`：`message`
- `group:nodes`：`nodes`
- `group:openclaw`：所有内置 OpenClaw 工具（不包括提供商插件）

### 提升权限模式

`tools.elevated` 是全局的提升权限基线（基于发送者的允许列表）。`agents.list[].tools.elevated` 可以进一步限制特定智能体的提升权限（两级都必须允许才能生效）。

常见的安全加固模式：

- 为不受信任的智能体拒绝 `exec` 工具：`agents.list[].tools.deny: ["exec"]`
- 避免将路由到受限智能体的发送者加入允许列表
- 如果只需要沙盒执行，全局禁用提升权限：`tools.elevated.enabled: false`
- 为敏感场景按智能体禁用提升权限：`agents.list[].tools.elevated.enabled: false`

* * *

## 从单智能体配置迁移

**迁移前（单智能体配置）：**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**迁移后（多智能体配置，不同配置文件）：**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

旧的 `agent.*` 配置会由 `openclaw doctor` 命令自动迁移。建议后续使用 `agents.defaults` + `agents.list` 的新配置方式。

* * *

## 工具限制示例

### 只读智能体

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### 安全执行智能体（禁止文件修改）

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### 仅通信智能体

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## 常见陷阱："non-main" 模式

`agents.defaults.sandbox.mode: "non-main"` 的判断依据是 `session.mainKey`（默认值为 `"main"`），而非智能体的 id。群组/频道会话总是拥有独立的键，因此会被视为非主会话而进入沙盒。如果你希望某个智能体永不进入沙盒，请显式设置 `agents.list[].sandbox.mode: "off"`。

* * *

## 测试验证

配置好多智能体沙盒和工具限制后，按以下步骤验证：

1. **检查智能体解析结果：**

   ```bash
   openclaw agents list --bindings
   ```

2. **验证沙盒容器状态：**

   ```bash
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **测试工具限制是否生效：**
   - 发送一条需要使用受限工具的消息
   - 确认智能体确实无法使用被拒绝的工具

4. **监控运行日志：**

   ```bash
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

* * *

## 故障排除

### 已设置 `mode: "all"` 但智能体仍未进入沙盒

- 检查是否存在全局的 `agents.defaults.sandbox.mode` 覆盖了你的设置
- 智能体级别的配置优先级更高，确保设置了 `agents.list[].sandbox.mode: "all"`

### 已添加拒绝列表但工具仍可用

- 检查工具过滤顺序：全局 → 智能体 → 沙盒 → 子智能体
- 每一层只能进一步限制，无法恢复之前层级已允许的工具
- 通过日志验证：`[tools] filtering tools for agent:${agentId}`

### 容器未按智能体隔离

- 在智能体的沙盒配置中设置 `scope: "agent"`
- 默认值为 `"session"`，会为每个会话创建独立容器

* * *

## 相关资料

- [多智能体路由](../concepts/multi-agent.md)
- [沙盒配置](../gateway/configuration.md#agentsdefaults-sandbox)
- [会话管理](../concepts/session.md)

[ACP 智能体](./acp-agents.md)[创建技能](./creating-skills.md)