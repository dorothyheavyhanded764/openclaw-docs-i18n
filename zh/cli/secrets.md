

  CLI 命令

  
# secrets

本节介绍如何使用 `openclaw secrets` 命令来管理 SecretRef 引用，确保运行时快照中的密钥配置正确且安全。这些命令帮助你完成密钥的审计、配置和应用，让你的系统始终保持健康的密钥状态。

## 命令概览

`openclaw secrets` 提供四个子命令，分别承担不同的职责：

- **`reload`**：重新解析密钥引用并以原子方式交换运行时快照。通过网关 RPC（`secrets.reload`）执行，仅在全部解析成功时才会更新快照，不会写入配置文件。
- **`audit`**：只读扫描，检查配置文件、认证信息和生成模型存储中的明文密钥、未解析引用、优先级漂移问题，以及遗留残留数据。
- **`configure`**：交互式规划器，引导你完成提供商设置、密钥字段映射和预检验证（需要终端 TTY 支持）。
- **`apply`**：执行已保存的配置计划，支持 `--dry-run` 仅做验证，执行后会清除目标位置的明文残留。

## 推荐操作流程

日常运维中，建议按以下顺序使用这些命令：

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

用于 CI/流水线时，注意以下退出代码：

- `audit --check` 发现问题时返回退出码 `1`
- 存在未解析引用时返回退出码 `2`

## 相关资源

- 密钥管理详解：[密钥管理](../gateway/secrets.md)
- 凭证范围规范：[SecretRef 凭证范围](../reference/secretref-credential-surface.md)
- 安全最佳实践：[安全](../gateway/security.md)

---

## 重新加载运行时快照

当密钥配置发生变化后，使用 `reload` 命令让运行时立即生效。它会重新解析所有密钥引用，并以原子方式交换运行时快照。

```bash
openclaw secrets reload
openclaw secrets reload --json
```

工作原理：

- 调用网关 RPC 方法 `secrets.reload` 执行
- 如果解析过程中任何一步失败，网关会保留上一个有效快照并返回错误，不会出现"部分激活"的中间状态
- 使用 `--json` 参数时，响应中会包含 `warningCount` 字段

---

## 审计密钥状态

`audit` 命令帮你全面检查系统中的密钥安全隐患。它会扫描以下问题：

- **明文密钥存储**：直接写在配置文件中的密钥
- **未解析的引用**：配置了 SecretRef 但无法解析的密钥引用
- **优先级漂移**：`auth-profiles.json` 中的凭证覆盖了 `openclaw.json` 中的引用配置
- **生成文件残留**：`agents/*/agent/models.json` 中遗留的 `apiKey` 值和敏感请求头
- **遗留数据残留**：旧版认证存储条目、OAuth 提醒信息等

关于请求头残留检测：系统使用名称启发式规则来识别敏感请求头，包括常见的认证/凭证相关字段名和片段，如 `authorization`、`x-api-key`、`token`、`secret`、`password`、`credential` 等。

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

退出行为：

- 使用 `--check` 参数时，如果发现问题会以非零状态退出，适合在 CI 中使用
- 未解析引用会以更高优先级的非零代码退出

审计报告包含以下核心字段：

- `status`：整体状态，可能的值为 `clean | findings | unresolved`
- `summary`：问题统计，包括 `plaintextCount`（明文数量）、`unresolvedRefCount`（未解析引用数）、`shadowedRefCount`（被覆盖引用数）、`legacyResidueCount`（遗留残留数）
- 发现代码：
  - `PLAINTEXT_FOUND` - 发现明文密钥
  - `REF_UNRESOLVED` - 引用无法解析
  - `REF_SHADOWED` - 引用被覆盖
  - `LEGACY_RESIDUE` - 遗留数据残留

---

## 交互式配置助手

`configure` 命令提供交互式的密钥配置流程，一步步引导你完成提供商设置和密钥字段映射。

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

配置流程分为三个阶段：

1. **提供商设置**：添加、编辑或删除 `secrets.providers` 中的提供商别名
2. **凭证映射**：选择需要配置的密钥字段，为其分配 `{source, provider, id}` 引用
3. **预检与应用**：执行预检验证，确认无误后可选择立即应用

### 常用参数

- `--providers-only`：仅配置 `secrets.providers`，跳过凭证映射步骤
- `--skip-provider-setup`：跳过提供商设置，直接为现有提供商配置凭证映射
- `--agent `：限定 `auth-profiles.json` 的目标发现和写入范围到指定的智能体存储

### 重要说明

- 此命令需要交互式终端（TTY）支持
- `--providers-only` 和 `--skip-provider-setup` 不能同时使用
- `configure` 会扫描 `openclaw.json` 中包含密钥的字段，以及指定智能体范围内的 `auth-profiles.json`
- 支持在选取器流程中直接创建新的 `auth-profiles.json` 映射
- 支持的字段范围详见：[SecretRef 凭证范围](../reference/secretref-credential-surface.md)
- 执行应用前会先进行预检解析
- 生成的计划默认启用所有清除选项（`scrubEnv`、`scrubAuthProfilesForProviderTargets`、`scrubLegacyAuthJson`）
- 已清除的明文值无法恢复，应用操作是单向的
- 不使用 `--apply` 参数时，CLI 会在预检后询问"立即应用此计划？"
- 使用 `--apply` 但不使用 `--yes` 时，CLI 会要求额外的不可逆确认

### Exec 提供商安全提示

如果你使用 Exec 类型的密钥提供商，请注意：

- 通过 Homebrew 安装的程序通常是符号链接，位于 `/opt/homebrew/bin/*` 目录下
- 仅在确实需要信任包管理器路径时，才设置 `allowSymlinkCommand: true`，并同时配置 `trustedDirs`（例如 `["/opt/homebrew"]`）
- 在 Windows 系统上，如果无法对提供商路径进行 ACL 验证，OpenClaw 默认会拒绝执行。仅对受信任的路径，才在该提供商上设置 `allowInsecurePath: true` 来绕过路径安全检查

---

## 应用配置计划

`apply` 命令用于执行之前通过 `configure` 生成的配置计划。

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

关于计划的具体格式和约束（允许的目标路径、验证规则、失败处理语义），请参考：[密钥应用计划合约](../gateway/secrets-plan-contract.md)

`apply` 命令可能修改以下文件：

- `openclaw.json`：更新 SecretRef 目标、执行提供商的新增/删除操作
- `auth-profiles.json`：清除提供商相关的密钥字段
- 遗留的 `auth.json` 文件中的残留数据
- `~/.openclaw/.env`：清除已迁移密钥对应的条目

---

## 为什么不提供回滚备份

`secrets apply` 命令**有意不写入包含旧明文值的回滚备份文件**。这种设计基于以下安全考虑：

一旦明文密钥被写入磁盘备份，就增加了泄露风险。我们的安全策略是：**严格的预检验证 + 原子化应用 + 失败时尽力内存恢复**。这样既保证了安全性，又提供了可靠的错误处理机制。

---

## 快速上手示例

下面是一个完整的密钥迁移示例：

```bash
# 步骤 1: 审计当前状态
openclaw secrets audit --check

# 步骤 2: 交互式配置密钥
openclaw secrets configure

# 步骤 3: 再次审计确认配置正确
openclaw secrets audit --check
```

如果 `audit --check` 仍然报告明文密钥问题，请根据报告中的目标路径更新配置，然后重新运行审计。

[Sandbox CLI](./sandbox.md) [安全](./security.md)