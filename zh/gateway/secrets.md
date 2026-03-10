

  配置与运维

  
# 密钥管理

OpenClaw 支持可叠加的 SecretRef 机制，让你无需将敏感凭证以明文形式存储在配置文件中。明文方式依然可用，SecretRef 是按凭证可选启用的。

## 目标与运行时模型

密钥会被解析到内存中的运行时快照。

-   解析在激活阶段主动完成，而非在请求路径上延迟执行。
-   当有效活跃的 SecretRef 无法解析时，启动会快速失败。
-   重载使用原子交换：要么全部成功，要么保留上次已知的良好快照。
-   运行时请求只从活跃的内存快照读取。

这样确保密钥提供商的故障不会影响热请求路径。

## 活跃表面过滤

SecretRef 仅在有效活跃的表面上进行验证。

-   已启用的表面：未解析的引用会阻止启动/重载。
-   未启用的表面：未解析的引用不会阻止启动/重载。
-   未激活的引用会发出非致命诊断，错误码为 `SECRETS_REF_IGNORED_INACTIVE_SURFACE`。

未激活表面的示例：

-   已禁用的频道/账号条目。
-   没有被任何已启用账号继承的顶层频道凭证。
-   已禁用的工具/功能表面。
-   未被 `tools.web.search.provider` 选中的 Web 搜索提供商特定密钥。在自动模式下（未设置提供商），提供商特定密钥也会参与提供商自动检测。
-   `gateway.remote.token` / `gateway.remote.password` SecretRef 在以下条件之一成立时为活跃状态（当 `gateway.remote.enabled` 不为 `false`）：
    -   `gateway.mode=remote`
    -   配置了 `gateway.remote.url`
    -   `gateway.tailscale.mode` 为 `serve` 或 `funnel`
    
    在没有这些远程表面的本地模式下：
    -   当令牌认证可能生效且未配置环境/认证令牌时，`gateway.remote.token` 为活跃状态。
    -   仅当密码认证可能生效且未配置环境/认证密码时，`gateway.remote.password` 为活跃状态。
-   当设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `CLAWDBOT_GATEWAY_TOKEN`）时，`gateway.auth.token` SecretRef 在启动认证解析时为非活跃状态，因为环境变量令牌在该运行时中优先。

## 网关认证表面诊断

当在 `gateway.auth.token`、`gateway.auth.password`、`gateway.remote.token` 或 `gateway.remote.password` 上配置了 SecretRef 时，网关启动/重载日志会明确记录表面状态：

-   `active`：该 SecretRef 属于有效认证表面，必须解析成功。
-   `inactive`：该 SecretRef 在此运行时被忽略，因为其他认证表面优先，或远程认证已禁用/未激活。

这些日志条目使用 `SECRETS_GATEWAY_AUTH_SURFACE` 标签，并包含活跃表面策略的原因，方便你了解凭证为何被标记为活跃或非活跃。

## 引导流程预检验证

当引导流程以交互模式运行且你选择 SecretRef 存储方式时，OpenClaw 会在保存前执行预检验证：

-   环境引用：验证环境变量名称，确认引导过程中可见的值非空。
-   提供商引用（`file` 或 `exec`）：验证提供商选择、解析 `id`，并检查解析值的类型。
-   快速启动复用路径：当 `gateway.auth.token` 已是 SecretRef 时，引导流程会在探测/仪表板初始化前解析它（针对 `env`、`file`、`exec` 引用），使用相同的快速失败检查。

如果验证失败，引导流程会显示错误并允许重试。

## SecretRef 契约

所有地方都使用统一的对象格式：

```json
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### source: "env"

```json
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

验证规则：

-   `provider` 必须匹配 `^[a-z][a-z0-9_-]{0,63}$`
-   `id` 必须匹配 `^[A-Z][A-Z0-9_]{0,127}$`

### source: "file"

```json
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

验证规则：

-   `provider` 必须匹配 `^[a-z][a-z0-9_-]{0,63}$`
-   `id` 必须是绝对 JSON 指针（`/...`）
-   段内使用 RFC6901 转义：`~` => `~0`，`/` => `~1`

### source: "exec"

```json
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

验证规则：

-   `provider` 必须匹配 `^[a-z][a-z0-9_-]{0,63}$`
-   `id` 必须匹配 `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## 提供商配置

在 `secrets.providers` 下定义提供商：

```json
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // 或 "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### 环境变量提供商

-   可通过 `allowlist` 设置可选白名单。
-   缺失或空的环境变量值会导致解析失败。

### 文件提供商

-   从 `path` 指定的本地文件读取。
-   `mode: "json"` 期望 JSON 对象负载，并将 `id` 作为指针解析。
-   `mode: "singleValue"` 期望引用 id 为 `"value"`，直接返回文件内容。
-   路径必须通过所有权和权限检查。
-   Windows 故障关闭说明：如果路径的 ACL 验证不可用，解析会失败。对于受信任的路径，可在该提供商上设置 `allowInsecurePath: true` 绕过路径安全检查。

### 执行提供商

-   运行配置的绝对二进制路径，不使用 shell。
-   默认情况下，`command` 必须指向常规文件（不能是符号链接）。
-   设置 `allowSymlinkCommand: true` 可允许符号链接命令路径（如 Homebrew 垫片）。OpenClaw 会验证解析后的目标路径。
-   将 `allowSymlinkCommand` 与 `trustedDirs` 配合使用，用于包管理器路径（如 `["/opt/homebrew"]`）。
-   支持超时、无输出超时、输出字节限制、环境变量白名单和受信任目录配置。
-   Windows 故障关闭说明：如果命令路径的 ACL 验证不可用，解析会失败。对于受信任的路径，可在该提供商上设置 `allowInsecurePath: true` 绕过路径安全检查。

请求负载（stdin）：

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

响应负载（stdout）：

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "<openai-api-key>" } } // pragma: allowlist secret
```

可选的按 id 错误：

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## Exec 集成示例

### 1Password CLI

```json
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // Homebrew 符号链接二进制需要此项
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### HashiCorp Vault CLI

```json
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // Homebrew 符号链接二进制需要此项
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### sops

```json
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // Homebrew 符号链接二进制需要此项
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## 支持的凭证表面

规范的支持和不支持凭证列表请参阅：

-   [SecretRef 凭证表面](../reference/secretref-credential-surface.md)

运行时生成或轮换的凭证以及 OAuth 刷新材料被有意排除在只读 SecretRef 解析之外。

## 必需行为与优先级

-   没有引用的字段：保持不变。
-   有引用的字段：在活跃表面上，激活期间必须解析成功。
-   如果同时存在明文和引用，在支持的优先级路径上引用优先。

警告和审计信号：

-   `SECRETS_REF_OVERRIDES_PLAINTEXT`（运行时警告）
-   `REF_SHADOWED`（当 `auth-profiles.json` 凭证优先于 `openclaw.json` 引用时的审计发现）

Google Chat 兼容性行为：

-   `serviceAccountRef` 优先于明文 `serviceAccount`。
-   当设置了同级引用时，明文值被忽略。

## 激活触发器

密钥激活在以下时机运行：

-   启动（预检 + 最终激活）
-   配置重载热应用路径
-   配置重载重启检查路径
-   通过 `secrets.reload` 手动重载

激活契约：

-   成功时原子性交换快照。
-   启动失败会中止网关启动。
-   运行时重载失败会保留上次已知的良好快照。

## 降级与恢复信号

当重载时激活在健康状态后失败，OpenClaw 进入密钥降级状态。一次性系统事件和日志代码：

-   `SECRETS_RELOADER_DEGRADED`
-   `SECRETS_RELOADER_RECOVERED`

行为说明：

-   降级：运行时保留上次已知的良好快照。
-   恢复：在下一次成功激活后发送一次。
-   已降级状态下的重复失败只记录警告，不会重复发送事件。
-   启动快速失败不会发送降级事件，因为运行时从未进入活跃状态。

## 命令路径解析

命令路径可以通过网关快照 RPC 选择使用 SecretRef 解析。有两种主要行为：

-   严格命令路径（如 `openclaw memory` 远程内存路径和 `openclaw qr --remote`）从活跃快照读取，当必需的 SecretRef 不可用时快速失败。
-   只读命令路径（如 `openclaw status`、`openclaw status --all`、`openclaw channels status`、`openclaw channels resolve` 以及只读的 doctor/config 修复流程）也优先使用活跃快照，但当目标 SecretRef 在该命令路径中不可用时，会降级继续而非中止。

只读行为：

-   当网关运行时，这些命令首先从活跃快照读取。
-   如果网关解析不完整或网关不可用，会尝试针对特定命令表面的本地回退。
-   如果目标 SecretRef 仍不可用，命令会以降级的只读输出继续，并显示明确的诊断信息如"已配置但在此命令路径中不可用"。
-   这种降级行为仅限于命令本地，不会削弱运行时启动、重载或发送/认证路径。

其他说明：

-   后端密钥轮换后的快照刷新由 `openclaw secrets reload` 处理。
-   这些命令路径使用的网关 RPC 方法：`secrets.resolve`。

## 审计与配置工作流

默认操作员工作流：

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### secrets audit

审计发现包括：

-   静态明文值（`openclaw.json`、`auth-profiles.json`、`.env` 和生成的 `agents/*/agent/models.json`）
-   生成的 `models.json` 条目中的敏感提供商请求头残留
-   未解析的引用
-   优先级遮蔽（`auth-profiles.json` 覆盖 `openclaw.json` 引用）
-   遗留残留（`auth.json`、OAuth 提醒）

请求头残留说明：

-   敏感提供商请求头检测基于名称启发式（常见的认证/凭证请求头名称和片段，如 `authorization`、`x-api-key`、`token`、`secret`、`password`、`credential`）。

### secrets configure

交互式配置助手，可以：

-   首先配置 `secrets.providers`（`env`/`file`/`exec`，添加/编辑/删除）
-   让你选择 `openclaw.json` 中支持密钥的字段，以及一个智能体作用域的 `auth-profiles.json`
-   可以直接在目标选择器中创建新的 `auth-profiles.json` 映射
-   捕获 SecretRef 详情（`source`、`provider`、`id`）
-   运行预检解析
-   可以立即应用

有用的模式：

-   `openclaw secrets configure --providers-only`
-   `openclaw secrets configure --skip-provider-setup`
-   `openclaw secrets configure --agent `

`configure` 应用的默认操作：

-   清除 `auth-profiles.json` 中目标提供商的匹配静态凭证
-   清除 `auth.json` 中的遗留静态 `api_key` 条目
-   清除 `<config-dir>/.env` 中匹配的已知密钥行

### secrets apply

应用已保存的计划：

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

严格的目标/路径契约详情和确切的拒绝规则请参阅：

-   [密钥应用计划契约](./secrets-plan-contract.md)

## 单向安全策略

OpenClaw 有意不写入包含历史明文密钥值的回滚备份。安全模型：

-   预检必须在写入模式前成功
-   运行时激活在提交前经过验证
-   应用使用原子文件替换更新文件，失败时尽力恢复

## 遗留认证兼容性说明

对于静态凭证，运行时不再依赖明文遗留认证存储。

-   运行时凭证源是解析后的内存快照。
-   遗留静态 `api_key` 条目在发现时会被清除。
-   OAuth 相关的兼容性行为保持独立。

## Web UI 说明

某些 SecretInput 联合类型在原始编辑器模式下比表单模式更容易配置。

## 相关文档

-   CLI 命令：[secrets](../cli/secrets.md)
-   计划契约详情：[密钥应用计划契约](./secrets-plan-contract.md)
-   凭证表面：[SecretRef 凭证表面](../reference/secretref-credential-surface.md)
-   认证设置：[认证](./authentication.md)
-   安全态势：[安全](./security.md)
-   环境变量优先级：[环境变量](../help/environment.md)

[认证凭证语义](../auth-credential-semantics.md)[密钥应用计划契约](./secrets-plan-contract.md)