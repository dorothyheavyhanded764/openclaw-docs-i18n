

  CLI 命令

  
# security

安全工具（审计 + 可选修复）。相关内容：

-   安全指南：[Security](../gateway/security.md)

## 审计

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

审计会在多个直接消息发送者共享主会话时发出警告，并推荐使用**安全直接消息模式**：`session.dmScope="per-channel-peer"`（对于多账户频道使用 `per-account-channel-peer`）来加固共享收件箱。这适用于协作/共享收件箱的加固。单个网关被互不信任/对立的操作者共享不是推荐的设置；应使用独立的网关（或独立的操作系统用户/主机）来划分信任边界。它还会在配置暗示可能存在共享用户入口时发出 `security.trust_model.multi_user_heuristic`（例如开放直接消息/群组策略、配置的群组目标或通配符发送者规则），并提醒你 OpenClaw 默认是个人助理信任模型。对于有意设置的多用户场景，审计建议对所有会话启用沙盒，保持文件系统访问限制在工作空间范围内，并避免在该运行时上存放个人/私密身份或凭据。它还会在小模型（`<=300B`）未启用沙盒且启用了 web/browser 工具时发出警告。对于 webhook 入口，它会在 `hooks.defaultSessionKey` 未设置、请求 `sessionKey` 覆盖已启用、以及覆盖已启用但没有 `hooks.allowedSessionKeyPrefixes` 时发出警告。它还会在沙盒 Docker 设置已配置但沙盒模式关闭时、`gateway.nodes.denyCommands` 使用无效的模式匹配/未知条目时（仅支持精确的节点命令名称匹配，不支持 shell 文本过滤）、`gateway.nodes.allowCommands` 显式启用危险的节点命令时、全局 `tools.profile="minimal"` 被智能体工具配置覆盖时、开放群组在没有沙盒/工作空间保护的情况下暴露运行时/文件系统工具时、以及在宽松的工具策略下安装的扩展插件工具可能被访问时发出警告。它还会标记 `gateway.allowRealIpFallback=true`（如果代理配置错误存在头部欺骗风险）和 `discovery.mdns.mode="full"`（通过 mDNS TXT 记录泄露元数据）。它还会在沙盒浏览器使用 Docker `bridge` 网络但没有 `sandbox.browser.cdpSourceRange` 时发出警告。它还会标记危险的沙盒 Docker 网络模式（包括 `host` 和 `container:*` 命名空间加入）。它还会在现有沙盒浏览器 Docker 容器缺少/过时的哈希标签时（例如迁移前的容器缺少 `openclaw.browserConfigEpoch`）发出警告，并推荐 `openclaw sandbox recreate --browser --all`。它还会在基于 npm 的插件/hook 安装记录未固定、缺少完整性元数据或与当前安装的包版本不一致时发出警告。它会在频道允许列表依赖可变名称/邮箱/标签而非稳定 ID 时发出警告（Discord、Slack、Google Chat、MS Teams、Mattermost、IRC 等适用范围）。它还会在 `gateway.auth.mode="none"` 使网关 HTTP API 在没有共享密钥的情况下可访问时发出警告（`/tools/invoke` 以及任何启用的 `/v1/*` 端点）。以 `dangerous`/`dangerously` 为前缀的设置是明确的紧急操作者覆盖；启用这些设置本身不是安全漏洞报告。完整的风险参数清单请参阅 [Security](../gateway/security.md) 中的"不安全或危险标志摘要"部分。

## JSON 输出

使用 `--json` 进行 CI/策略检查：

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

如果同时使用 `--fix` 和 `--json`，输出包含修复操作和最终报告：

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## --fix 会修改什么

`--fix` 应用安全、确定性的修复措施：

-   将常见的 `groupPolicy="open"` 改为 `groupPolicy="allowlist"`（包括支持频道中的账户变体）
-   将 `logging.redactSensitive` 从 `"off"` 设置为 `"tools"`
-   收紧状态/配置和常见敏感文件的权限（`credentials/*.json`、`auth-profiles.json`、`sessions.json`、会话 `*.jsonl`）

`--fix` **不会**：

-   轮换令牌/密码/API 密钥
-   禁用工具（`gateway`、`cron`、`exec` 等）
-   更改网关绑定/认证/网络暴露设置
-   删除或重写插件/技能

[secrets](./secrets.md)[sessions](./sessions.md)