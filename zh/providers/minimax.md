

  提供商（provider）

  
# MiniMax

MiniMax 是一家开发 **M2/M2.5** 模型系列的 AI 公司。当前主打编程场景的版本是 **MiniMax M2.5**（2025 年 12 月 23 日发布），专为处理现实中的复杂任务而设计。详见 [MiniMax M2.5 发布说明](https://www.minimax.io/news/minimax-m25)。

## 模型概览（M2.5）

MiniMax M2.5 带来了以下核心改进：

-   **多语言编程能力更强**：覆盖 Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS 等语言。
-   **Web/应用开发体验更好**：输出质量更美观（包括原生移动端开发）。
-   **复合指令处理更优**：适合办公自动化工作流，支持交错思考和集成约束执行。
-   **响应更简洁**：token 消耗更低，迭代循环更快。
-   **工具/智能体（agent）框架兼容性更强**：支持 Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox 等框架，上下文管理能力出色。
-   **对话和技术写作质量更高**：输出内容更加专业。

## MiniMax M2.5 与 MiniMax M2.5 Highspeed 的区别

-   **速度**：`MiniMax-M2.5-highspeed` 是 MiniMax 官方文档中的快速版。
-   **成本**：根据 MiniMax 定价，高速版输入成本相同，但输出成本略高。
-   **兼容性**：OpenClaw 仍支持旧版 `MiniMax-M2.5-Lightning` 配置，但新项目建议使用 `MiniMax-M2.5-highspeed`。

## 选择配置方式

### MiniMax OAuth（编程计划）—— 推荐

**适用场景**：通过 OAuth 快速接入 MiniMax 编程计划，无需 API 密钥（API key）。只需启用内置的 OAuth 插件并完成身份验证：

```bash
openclaw plugins enable minimax-portal-auth  # 如已加载可跳过
openclaw gateway restart  # 若网关已运行则需重启
openclaw onboard --auth-choice minimax-portal
```

系统会提示你选择端点：

-   **Global** — 国际用户（`api.minimax.io`）
-   **CN** — 国内用户（`api.minimaxi.com`）

详见 [MiniMax OAuth 插件 README](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth)。

### MiniMax M2.5（API 密钥）

**适用场景**：使用 Anthropic 兼容 API 接入托管的 MiniMax 服务。通过 CLI 配置：

-   运行 `openclaw configure`
-   选择 **Model/auth**
-   选择 **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 作为回退模型（示例）

**适用场景**：将最新的旗舰模型设为主模型，故障时自动回退到 MiniMax M2.5。以下示例使用 Opus 作为主模型，你可以替换为自己偏好的旗舰模型。

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### 可选：通过 LM Studio 本地运行（手动配置）

**适用场景**：使用 LM Studio 进行本地推理。在配置较好的硬件（如台式机或服务器）上，通过 LM Studio 的本地服务器运行 MiniMax M2.5 效果出色。通过 `openclaw.json` 手动配置：

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 使用交互式配置向导

如果你不想直接编辑 JSON 文件，可以使用交互式配置向导来设置 MiniMax：

1.  运行 `openclaw configure`。
2.  选择 **Model/auth**。
3.  选择 **MiniMax M2.5**。
4.  按提示选择默认模型。

## 配置选项说明

以下是 MiniMax 提供商（provider）的主要配置项：

-   `models.providers.minimax.baseUrl`：推荐使用 `https://api.minimax.io/anthropic`（Anthropic 兼容）；如需 OpenAI 兼容格式，可选 `https://api.minimax.io/v1`。
-   `models.providers.minimax.api`：推荐使用 `anthropic-messages`；如需 OpenAI 兼容格式，可选 `openai-completions`。
-   `models.providers.minimax.apiKey`：MiniMax API 密钥（API key），对应环境变量 `MINIMAX_API_KEY`。
-   `models.providers.minimax.models`：定义模型的 `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost` 等属性。
-   `agents.defaults.models`：为需要加入允许列表的模型设置别名。
-   `models.mode`：若想将 MiniMax 与内置模型并存使用，保持 `merge` 即可。

## 注意事项

-   模型引用格式为 `minimax/`。
-   推荐的模型 ID：`MiniMax-M2.5` 和 `MiniMax-M2.5-highspeed`。
-   编程计划用量查询 API：`https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（需要编程计划密钥）。
-   如需精确的成本追踪，请在 `models.json` 中更新定价数据。
-   MiniMax 编程计划推荐链接（享 9 折优惠）：[https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   关于提供商（provider）的通用规则，请参阅 [/concepts/model-providers](../concepts/model-providers.md)。
-   使用 `openclaw models list` 查看可用模型，使用 `openclaw models set minimax/MiniMax-M2.5` 切换模型。

## 故障排除

### 报错 "Unknown model: minimax/MiniMax-M2.5"

这个问题通常是因为 **MiniMax 提供商（provider）未配置**（找不到提供商配置条目，也没有 MiniMax 身份验证配置或环境变量）。该问题的检测逻辑已在 **2026.1.12** 版本中修复（本文撰写时尚未发布）。

解决方法：

-   升级到 **2026.1.12** 版本（或从 `main` 分支源码运行），然后重启网关。
-   或运行 `openclaw configure` 并选择 **MiniMax M2.5**。
-   或手动添加 `models.providers.minimax` 配置块。
-   或设置 `MINIMAX_API_KEY` 环境变量（或配置 MiniMax 身份验证），让系统自动注入提供商配置。

注意：模型 ID **区分大小写**，请确保使用正确的格式：

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning`（旧版）

配置完成后，运行以下命令验证：

```bash
openclaw models list
```

[GLM 模型](./glm.md)[Moonshot AI](./moonshot.md)