

  提供商

  
# GitHub Copilot

## GitHub Copilot 是什么？

GitHub Copilot 是 GitHub 推出的 AI 编程助手。根据你的 GitHub 账户和订阅计划，你可以访问相应的 Copilot 模型。在 OpenClaw 中，你有两种方式可以把 Copilot 作为模型提供商来使用。

## 两种接入方式

### 方式一：内置 GitHub Copilot 提供商（`github-copilot`）

这是**默认推荐**的方式，也是最简单的路径——无需安装 VS Code。OpenClaw 会通过原生设备登录流程获取 GitHub 令牌，然后在运行时自动换取 Copilot API 令牌。

### 方式二：Copilot Proxy 插件（`copilot-proxy`）

如果你已经在 VS Code 中运行 **Copilot Proxy** 扩展，或者需要通过它来路由请求，可以选择这种方式。OpenClaw 会与代理的 `/v1` 端点通信，并使用你在那里配置的模型列表。

需要注意的是，你必须启用该插件并保持 VS Code 扩展持续运行。运行 `github-copilot` 登录命令后，系统会执行 GitHub 设备授权流程，保存身份验证配置，并自动更新你的设置。

## 命令行设置

运行以下命令启动登录流程：

```bash
openclaw models auth login-github-copilot
```

系统会提示你访问一个网址并输入一次性验证码。请在终端中保持等待，直到流程完成。

### 可选参数

```bash
# 指定配置文件 ID
openclaw models auth login-github-copilot --profile-id github-copilot:work

# 跳过确认提示
openclaw models auth login-github-copilot --yes
```

## 设置默认模型

```bash
openclaw models set github-copilot/gpt-4o
```

### 配置示例

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## 注意事项

- 此命令需要交互式终端环境，请直接在终端中运行。
- 可用的 Copilot 模型取决于你的订阅计划。如果某个模型无法使用，请尝试其他模型 ID（例如 `github-copilot/gpt-4.1`）。
- 登录时，GitHub 令牌会被安全存储在身份验证配置中，OpenClaw 运行时会自动用它换取 Copilot API 令牌。

[Deepgram](./deepgram.md)[Hugging Face (推理)](./huggingface.md)