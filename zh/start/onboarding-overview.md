

  第一步

  
# 入门概览

OpenClaw 支持多种入门路径，你可以根据网关运行位置和提供商配置偏好选择最适合的方式。

## 选择你的入门路径

-   **CLI 向导** 适用于 macOS、Linux 和 Windows（通过 WSL2）。
-   **macOS 应用** 适用于 Apple 芯片或 Intel 芯片 Mac 的引导式首次运行体验。

## CLI 入门向导

在终端中运行向导：

```bash
openclaw onboard
```

当你希望对网关、工作区、频道和技能拥有完全控制权时，CLI 向导是最佳选择。相关文档：

-   [入门向导 (CLI)](./wizard.md)
-   [`openclaw onboard` 命令](../cli/onboard.md)

## macOS 应用入门

当你在 macOS 上想要完全引导式的设置体验时，可以使用 OpenClaw 应用。相关文档：

-   [入门（macOS 应用）](./onboarding.md)

## 自定义提供商

如果你需要的端点不在列表中——包括暴露标准 OpenAI 或 Anthropic API 的托管提供商——可以在 CLI 向导中选择 **自定义提供商**。系统会要求你：

-   选择 OpenAI 兼容、Anthropic 兼容或 **未知**（自动检测）。
-   输入基础 URL 和 API 密钥（如果提供商需要）。
-   提供模型 ID 和可选别名。
-   选择一个端点 ID，以便多个自定义端点可以共存。

详细步骤请参考上述 CLI 入门文档。

[开始使用](./getting-started.md)[入门：CLI](./wizard.md)