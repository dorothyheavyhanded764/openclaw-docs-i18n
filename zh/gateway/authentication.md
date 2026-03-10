

  配置与运维

  
# 认证

OpenClaw 支持模型提供商的 OAuth 和 API 密钥两种认证方式。对于长期运行的网关主机，API 密钥通常是最稳定可靠的选择。如果你的提供商账户支持订阅/OAuth 流程，也可以使用这些方式。完整的 OAuth 流程和存储结构请参阅 [/concepts/oauth](../concepts/oauth.md)。基于 SecretRef 的认证（`env`/`file`/`exec` 提供商）请参阅[密钥管理](./secrets.md)。`models status --probe` 使用的凭证资格/原因码规则请参阅[认证凭证语义](../auth-credential-semantics.md)。

## 推荐配置（API 密钥，适用于所有提供商）

如果你要运行长期在线的网关，建议从 API 密钥开始。对于 Anthropic，API 密钥认证是稳妥的方案，比订阅 setup-token 更推荐。

1.  在提供商控制台创建 API 密钥。
2.  将密钥配置在**网关主机**上（运行 `openclaw gateway` 的机器）。

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  如果网关以 systemd/launchd 服务运行，建议将密钥写入 `~/.openclaw/.env`，这样守护进程可以自动读取：

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

然后重启守护进程（或重启网关进程）并验证：

```bash
openclaw models status
openclaw doctor
```

如果你不想手动管理环境变量，引导向导可以帮你存储 API 密钥供守护进程使用：运行 `openclaw onboard`。环境继承的详细说明（`env.shellEnv`、`~/.openclaw/.env`、systemd/launchd）请参阅[帮助](../help.md)。

## Anthropic：setup-token（订阅认证）

如果你使用 Claude 订阅，可以使用 setup-token 流程。在**网关主机**上运行：

```bash
claude setup-token
```

然后将其粘贴到 OpenClaw：

```bash
openclaw models auth setup-token --provider anthropic
```

如果令牌是在其他机器上创建的，可以手动粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

如果你看到如下 Anthropic 错误：

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

……请改用 Anthropic API 密钥。

> **⚠️** Anthropic setup-token 支持仅为技术兼容。Anthropic 过去曾限制部分订阅在 Claude Code 之外使用。请仅在确认政策风险可接受的情况下使用，并自行核实 Anthropic 的最新条款。

手动令牌输入（任何提供商；会写入 `auth-profiles.json` 并更新配置）：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

认证配置文件引用也支持静态凭证：

-   `api_key` 凭证可使用 `keyRef: { source, provider, id }`
-   `token` 凭证可使用 `tokenRef: { source, provider, id }`

自动化友好检查（过期/缺失时退出码 `1`，即将过期时退出码 `2`）：

```bash
openclaw models status --check
```

可选的运维脚本（systemd/Termux）详见 [/automation/auth-monitoring](../automation/auth-monitoring.md)。

> `claude setup-token` 需要交互式终端（TTY）。

## 检查模型认证状态

```bash
openclaw models status
openclaw doctor
```

## API 密钥轮换行为（网关）

某些提供商支持在遇到速率限制时使用备用密钥重试请求。

-   优先级顺序：
    -   `OPENCLAW_LIVE__KEY`（单一覆盖）
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   Google 提供商还会将 `GOOGLE_API_KEY` 作为额外备选。
-   相同密钥会在使用前去重。
-   OpenClaw 仅对速率限制错误（如 `429`、`rate_limit`、`quota`、`resource exhausted`）使用下一个密钥重试。
-   非速率限制错误不会使用备用密钥重试。
-   如果所有密钥都失败，返回最后一次尝试的最终错误。

## 控制使用哪个凭证

### 每会话（聊天命令）

使用 `/model <alias-or-id>@` 为当前会话固定使用特定提供商凭证（例如：`anthropic:default`、`anthropic:work`）。使用 `/model`（或 `/model list`）获取简洁的选择器；使用 `/model status` 查看完整信息（候选列表 + 下一个认证配置文件，以及配置的提供商端点详情）。

### 每智能体（CLI 覆盖）

为智能体设置显式的认证配置文件顺序覆盖（存储在该智能体的 `auth-profiles.json` 中）：

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

使用 `--agent ` 指定目标智能体；省略则使用配置的默认智能体。

## 故障排除

### "No credentials found"（未找到凭证）

如果 Anthropic 令牌配置文件缺失，请在**网关主机**上运行 `claude setup-token`，然后重新检查：

```bash
openclaw models status
```

### 令牌即将过期/已过期

运行 `openclaw models status` 确认哪个配置文件即将过期。如果配置文件缺失，重新运行 `claude setup-token` 并再次粘贴令牌。

## 系统要求

-   Anthropic 订阅账户（用于 `claude setup-token`）
-   已安装 Claude Code CLI（`claude` 命令可用）

[配置示例](./configuration-examples.md)[认证凭证语义](../auth-credential-semantics.md)