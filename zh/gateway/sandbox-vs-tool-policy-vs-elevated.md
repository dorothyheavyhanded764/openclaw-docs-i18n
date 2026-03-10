

  安全与沙盒

  
# 沙盒 vs 工具策略 vs 提升权限

OpenClaw 有三套相关（但不同）的控制机制：

1.  **沙盒**（`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`）决定**工具在哪里运行**（Docker 容器内还是宿主机上）。
2.  **工具策略**（`tools.*`、`tools.sandbox.tools.*`、`agents.list[].tools.*`）决定**哪些工具可用/被允许**。
3.  **提升权限**（`tools.elevated.*`、`agents.list[].tools.elevated.*`）是一个**仅限 exec 的逃生舱**，让你在沙盒环境下也能在宿主机上执行命令。

## 快速诊断

使用检查器查看 OpenClaw *实际*的配置效果：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

它会显示：

-   实际生效的沙盒模式/范围/工作空间访问权限
-   当前会话是否处于沙盒中（主会话 vs 非主会话）
-   实际生效的沙盒工具允许/拒绝列表（以及配置来源：智能体/全局/默认）
-   提升权限的门控条件和修复路径

## 沙盒：工具在哪里运行

沙盒功能由 `agents.defaults.sandbox.mode` 控制：

-   `"off"`：所有操作都在宿主机上运行。
-   `"non-main"`：只有非主会话被沙盒化（群组/频道常见的"意外"情况）。
-   `"all"`：所有会话都被沙盒化。

完整的配置矩阵（范围、工作空间挂载、镜像等）请参阅[沙盒](./sandboxing.md)。

### 绑定挂载（安全检查要点）

-   `docker.binds` 会*穿透*沙盒文件系统：你挂载的任何内容都会以你指定的权限（`:ro` 或 `:rw`）在容器内可见。
-   如果省略权限，默认是读写模式；对于源代码/密钥等敏感内容，建议使用 `:ro`。
-   `scope: "shared"` 会忽略每个智能体的绑定配置（仅应用全局绑定）。
-   绑定 `/var/run/docker.sock` 相当于把宿主机控制权交给沙盒；请务必有意为之。
-   工作空间访问权限（`workspaceAccess: "ro"`/`"rw"`）与绑定权限是独立的。

## 工具策略：哪些工具存在/可调用

有两层配置需要关注：

-   **工具配置文件**：`tools.profile` 和 `agents.list[].tools.profile`（基础白名单）
-   **提供商工具配置文件**：`tools.byProvider[provider].profile` 和 `agents.list[].tools.byProvider[provider].profile`
-   **全局/每智能体工具策略**：`tools.allow`/`tools.deny` 和 `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **提供商工具策略**：`tools.byProvider[provider].allow/deny` 和 `agents.list[].tools.byProvider[provider].allow/deny`
-   **沙盒工具策略**（仅在沙盒化时生效）：`tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` 和 `agents.list[].tools.sandbox.tools.*`

基本原则：

-   `deny` 优先级最高。
-   如果 `allow` 非空，则其他所有工具都被视为被阻止。
-   工具策略是硬性限制：`/exec` 命令无法覆盖被拒绝的 `exec` 工具。
-   `/exec` 仅调整授权发送者的会话默认值；它不授予工具访问权限。提供商工具键接受 `provider`（如 `google-antigravity`）或 `provider/model`（如 `openai/gpt-5.2`）格式。

### 工具组（简写形式）

工具策略（全局、智能体、沙盒）支持 `group:*` 条目，它们会自动展开为多个工具：

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

可用的工具组：

-   `group:runtime`: `exec`、`bash`、`process`
-   `group:fs`: `read`、`write`、`edit`、`apply_patch`
-   `group:sessions`: `sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
-   `group:memory`: `memory_search`、`memory_get`
-   `group:ui`: `browser`、`canvas`
-   `group:automation`: `cron`、`gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供商插件）

## 提升权限：仅限 exec 的"在宿主机上运行"

提升权限**不会**授予额外工具；它只影响 `exec` 的执行位置。

-   如果你处于沙盒中，`/elevated on`（或带 `elevated: true` 的 `exec`）会在宿主机上运行命令（审批流程仍可能适用）。
-   使用 `/elevated full` 可以跳过该会话的 exec 审批。
-   如果你已经是直接运行（非沙盒），提升权限实际上是无操作（但仍受门控限制）。
-   提升权限**不是**技能范围的，也**不会**覆盖工具的允许/拒绝列表。
-   `/exec` 与提升权限是分开的。它只调整授权发送者的每会话 exec 默认值。

门控条件：

-   启用开关：`tools.elevated.enabled`（以及可选的 `agents.list[].tools.elevated.enabled`）
-   发送者白名单：`tools.elevated.allowFrom.`（以及可选的 `agents.list[].tools.elevated.allowFrom.`）

详见[提升权限模式](../tools/elevated.md)。

## 常见的"沙盒陷阱"修复

### "工具 X 被沙盒工具策略阻止"

修复路径（任选其一）：

-   禁用沙盒：`agents.defaults.sandbox.mode=off`（或每智能体 `agents.list[].sandbox.mode=off`）
-   在沙盒内允许该工具：
    -   从 `tools.sandbox.tools.deny` 中移除（或每智能体 `agents.list[].tools.sandbox.tools.deny`）
    -   或添加到 `tools.sandbox.tools.allow`（或每智能体 allow）

### "我以为这是主会话，为什么被沙盒化了？"

在 `"non-main"` 模式下，群组/频道的会话密钥*不是*主会话。使用主会话密钥（`sandbox explain` 显示的那个）或将模式切换为 `"off"`。

[沙盒](./sandboxing.md)[网关协议](./protocol.md)