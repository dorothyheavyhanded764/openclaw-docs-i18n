

  CLI 命令

  
# onboard

交互式入门向导（本地或远程网关设置）。

## 相关指南

-   CLI 入门中心：[Onboarding Wizard (CLI)](../start/wizard.md)
-   入门概述：[Onboarding Overview](../start/onboarding-overview.md)
-   CLI 入门参考：[CLI Onboarding Reference](../start/wizard-cli-reference.md)
-   CLI 自动化：[CLI Automation](../start/wizard-cli-automation.md)
-   macOS 入门：[Onboarding (macOS App)](../start/onboarding.md)

## 示例

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

对于明文私有网络 `ws://` 目标（仅限受信任的网络），在入门流程环境中设置 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1`。非交互式自定义提供商：

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` 在非交互模式下是可选的。如果省略，入门向导会检查 `CUSTOM_API_KEY`。将提供商密钥存储为引用而非明文：

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

使用 `--secret-input-mode ref`，入门向导会写入环境变量支持的引用而非明文密钥值。对于认证配置文件支持的提供商，这会写入 `keyRef` 条目；对于自定义提供商，这会将 `models.providers..apiKey` 写入为环境变量引用（例如 `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`）。非交互式 `ref` 模式约定：

-   在入门流程环境中设置提供商环境变量（例如 `OPENAI_API_KEY`）。
-   不要传递内联密钥标志（例如 `--openai-api-key`），除非同时设置了该环境变量。
-   如果传递了内联密钥标志但缺少必需的环境变量，入门向导会快速失败并提供指导。

非交互模式下的网关令牌选项：

-   `--gateway-auth token --gateway-token ` 存储明文令牌。
-   `--gateway-auth token --gateway-token-ref-env ` 将 `gateway.auth.token` 存储为环境变量 SecretRef。
-   `--gateway-token` 和 `--gateway-token-ref-env` 互斥。
-   `--gateway-token-ref-env` 需要在入门流程环境中有非空环境变量。
-   使用 `--install-daemon` 时，当令牌认证需要令牌，SecretRef 管理的网关令牌会被验证但不会作为已解析的明文持久化到监管服务环境元数据中。
-   使用 `--install-daemon` 时，如果令牌模式需要令牌但配置的令牌 SecretRef 未解析，入门向导会安全失败并提供修复指导。
-   使用 `--install-daemon` 时，如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password`，且 `gateway.auth.mode` 未设置，入门向导会阻止安装直到明确设置模式。

示例：

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

引用模式下的交互式入门行为：

-   在提示时选择 **使用密钥引用**。
-   然后选择：
    -   环境变量
    -   已配置的密钥提供商（`file` 或 `exec`）
-   入门向导在保存引用之前会执行快速预检验证。
    -   如果验证失败，入门向导会显示错误并让你重试。

非交互式 Z.AI 端点选择：注意：`--auth-choice zai-api-key` 现在会自动为你的密钥检测最佳 Z.AI 端点（优先使用 `zai/glm-5` 的通用 API）。如果你特别想要 GLM Coding Plan 端点，请选择 `zai-coding-global` 或 `zai-coding-cn`。

```bash
# 无提示端点选择
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# 其他 Z.AI 端点选择：
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

非交互式 Mistral 示例：

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

流程说明：

-   `quickstart`：最少提示，自动生成网关令牌。
-   `manual`：完整的端口/绑定/认证提示（`advanced` 的别名）。
-   本地入门直接消息范围行为：[CLI Onboarding Reference](../start/wizard-cli-reference.md#outputs-and-internals)。
-   最快首次聊天：`openclaw dashboard`（控制界面，无需频道设置）。
-   自定义提供商：连接任何 OpenAI 或 Anthropic 兼容端点，包括未列出的托管提供商。使用 Unknown 自动检测。

## 常用后续命令

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` 不意味着非交互模式。脚本场景请使用 `--non-interactive`。

[nodes](./nodes.md)[pairing](./pairing.md)