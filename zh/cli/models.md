

  CLI 命令

  
# models

模型发现、扫描和配置（默认模型、回退策略、认证配置文件）。相关内容：

-   提供商与模型：[Models](../providers/models.md)
-   提供商认证设置：[Getting started](../start/getting-started.md)

## 常用命令

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` 显示已解析的默认模型/回退策略以及认证概览。当提供商使用快照可用时，OAuth/令牌状态部分会包含提供商使用情况标题。添加 `--probe` 可对每个配置的提供商配置文件运行实时认证探测。探测是真实请求（可能消耗令牌并触发速率限制）。使用 `--agent ` 可检查已配置智能体的模型/认证状态。如果省略，命令会使用 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`（如果已设置），否则使用配置的默认智能体。注意事项：

-   `models set <model-or-alias>` 接受 `provider/model` 或别名。
-   模型引用通过在**第一个** `/` 处分割来解析。如果模型 ID 包含 `/`（OpenRouter 风格），请包含提供商前缀（例如：`openrouter/moonshotai/kimi-k2`）。
-   如果省略提供商，OpenClaw 会将输入视为别名或**默认提供商**的模型（仅当模型 ID 中没有 `/` 时有效）。
-   `models status` 在认证输出中可能会为非密钥占位符显示 `marker()`（例如 `OPENAI_API_KEY`、`secretref-managed`、`minimax-oauth`、`qwen-oauth`、`ollama-local`），而不是将其掩码为密钥。

### models status

选项：

-   `--json`
-   `--plain`
-   `--check`（退出码 1=过期/缺失，2=即将过期）
-   `--probe`（实时探测配置的认证配置文件）
-   `--probe-provider `（探测单个提供商）
-   `--probe-profile `（可重复或逗号分隔的配置文件 ID）
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `
-   `--agent `（已配置的智能体 ID；覆盖 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`）

## 别名与回退策略

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## 认证配置文件

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` 运行提供商插件的认证流程（OAuth/API 密钥）。使用 `openclaw plugins list` 查看已安装的提供商。注意事项：

-   `setup-token` 会提示输入 setup-token 值（可在任意机器上使用 `claude setup-token` 生成）。
-   `paste-token` 接受在其他地方生成或来自自动化的令牌字符串。
-   Anthropic 政策说明：setup-token 支持是技术兼容性。Anthropic 过去曾阻止 Claude Code 外部的某些订阅使用，因此在大规模使用前请验证当前条款。

[message](./message.md)[node](./node.md)