

  提供商（provider）

  
# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) 通过统一的 Router API 提供 OpenAI 兼容的聊天补全服务——只需一个令牌，就能访问 DeepSeek、Llama 等众多模型。OpenClaw 使用 **OpenAI 兼容端点**（仅支持聊天补全）；如需文生图、嵌入或语音功能，请直接使用 [HF Inference 客户端](https://huggingface.co/docs/api-inference/quicktour)。

-   提供商（provider）：`huggingface`
-   认证方式：`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`（细粒度令牌，需开启 **Make calls to Inference Providers** 权限）
-   API：OpenAI 兼容（`https://router.huggingface.co/v1`）
-   计费：单一 HF 令牌；[定价](https://huggingface.co/docs/inference-providers/pricing) 按各提供商费率计算，并提供免费额度。

## 快速开始

本节将带您完成 Hugging Face 提供商（provider）的接入配置。

1.  前往 [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) 创建一个细粒度令牌，并勾选 **Make calls to Inference Providers** 权限。
2.  运行引导命令，在提供商下拉菜单中选择 **Hugging Face**，然后按提示输入您的 API 密钥（API key）：

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  在 **Default Hugging Face model** 下拉菜单中选择您想要的模型。如果令牌有效，列表会从 Inference API 实时加载；否则会显示内置列表。您的选择将保存为默认模型。
4.  您也可以稍后在配置文件中设置或修改默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非交互式示例

如果您需要在脚本或自动化流程中配置，可以使用以下命令：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

这将把 `huggingface/deepseek-ai/DeepSeek-R1` 设置为默认模型。

## 环境变量说明

如果网关以守护进程（launchd/systemd）方式运行，请确保 `HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN` 对该进程可用。您可以将令牌写入 `~/.openclaw/.env` 文件，或通过 `env.shellEnv` 配置。

## 模型发现与引导下拉菜单

OpenClaw 通过直接调用 **Inference 端点** 来发现可用模型：

```bash
GET https://router.huggingface.co/v1/models
```

（可选：在请求头中发送 `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` 或 `$HF_TOKEN` 以获取完整列表；部分端点在未认证时只返回子集。）

响应格式为 OpenAI 风格：`{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`。

当您配置好 Hugging Face API 密钥（API key）（通过引导程序、`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`）后，OpenClaw 会调用此接口来发现可用的聊天补全模型。在 **交互式引导** 过程中，输入令牌后会显示 **Default Hugging Face model** 下拉菜单，选项来自该接口返回的列表（如果请求失败则使用内置目录）。

运行时（如网关启动时），如果检测到密钥，OpenClaw 会再次调用 **GET** `https://router.huggingface.co/v1/models` 刷新目录。返回的列表会与内置目录（包含上下文窗口、成本等元数据）合并。如果请求失败或未设置密钥，则仅使用内置目录。

## 模型名称与可编辑选项

本节说明模型名称的来源以及如何自定义显示名称和路由策略。

-   **名称来源：** 模型显示名称优先从 **GET /v1/models** 接口返回的 `name`、`title` 或 `display_name` 字段获取；如果接口未提供，则从模型 ID 派生（例如 `deepseek-ai/DeepSeek-R1` → "DeepSeek R1"）。
-   **自定义显示名称：** 您可以在配置中为每个模型设置别名（alias），让它在 CLI 和 UI 中显示为您期望的名称：

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (快速)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (最便宜)" },
      },
    },
  },
}
```

-   **提供商（provider）/ 策略选择：** 在 **模型 ID** 后添加后缀，可以控制路由器如何选择后端：

    -   **`:fastest`** — 最高吞吐量（由路由器自动选择；提供商选择被**锁定**，不会弹出交互式后端选择器）。
    -   **`:cheapest`** — 每个输出 token 成本最低（由路由器自动选择；提供商选择被**锁定**）。
    -   **`:provider`** — 强制使用指定后端（例如 `:sambanova`、`:together`）。

    当您选择 **:cheapest** 或 **:fastest**（例如在引导程序的模型下拉菜单中）时，提供商会被锁定：路由器根据成本或速度自动决定，不会显示"偏好特定后端"的选项。您可以将这些变体作为独立条目添加到 `models.providers.huggingface.models` 中，或在 `model.primary` 中直接使用带后缀的模型 ID。您也可以在 [Inference Provider 设置](https://hf.co/settings/inference-providers) 中配置默认顺序（不带后缀时使用该顺序）。

-   **配置合并：** 配置合并时，`models.providers.huggingface.models` 中的现有条目（例如在 `models.json` 中）会被保留。因此您之前设置的自定义 `name`、`alias` 或其他模型选项都会被保留。

## 模型 ID 与配置示例

模型引用格式为 `huggingface//`（Hub 风格 ID）。下表来自 **GET** `https://router.huggingface.co/v1/models` 接口；您的目录可能包含更多模型。

**示例 ID（来自 Inference 端点）：**

| 模型 | 引用（需加 `huggingface/` 前缀） |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

您可以在模型 ID 后附加 `:fastest`、`:cheapest` 或 `:provider`（例如 `:together`、`:sambanova`）。在 [Inference Provider 设置](https://hf.co/settings/inference-providers) 中设置您的默认顺序；完整列表请参阅 [Inference Providers 文档](https://huggingface.co/docs/inference-providers) 和 **GET** `https://router.huggingface.co/v1/models` 接口。

### 完整配置示例

**主模型使用 DeepSeek R1，Qwen 作为备用：**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen 作为默认模型，包含 :cheapest 和 :fastest 变体：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (最便宜)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (最快)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS 组合，并设置别名：**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**使用 :provider 强制指定特定后端：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**多个 Qwen 和 DeepSeek 模型，带策略后缀：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (最便宜)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (最快)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)