

  CLI 命令

  
# update

安全更新 OpenClaw，并在稳定版、测试版、开发版三个频道之间自由切换。如果你是通过 **npm/pnpm** 全局安装的（没有 git 元数据），更新流程请参考[更新指南](../install/updating.md)，由包管理器自动处理。

## 用法

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## 选项

以下是 `update` 命令支持的参数：

- `--no-restart`：更新成功后不重启网关服务
- `--channel <stable|beta|dev>`：设置更新频道（支持 git 和 npm 安装方式，设置会保存到配置中）
- `--tag <dist-tag|version>`：仅本次更新使用指定的 npm 分发标签或版本号
- `--dry-run`：预览更新计划（包括频道、标签、目标版本、重启流程），不会修改配置、安装包、同步插件或重启服务
- `--json`：输出机器可读的 `UpdateRunResult` JSON 格式
- `--timeout `：每个步骤的超时时间，默认 1200 秒

注意：降级操作需要确认，因为旧版本可能不兼容当前配置。

## update status

查看当前激活的更新频道、git 标签/分支/SHA（源码安装时），以及是否有可用更新。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

选项：

- `--json`：输出机器可读的状态 JSON
- `--timeout `：检查超时时间，默认 3 秒

## update wizard

交互式引导流程，帮你选择更新频道，并确认更新后是否重启网关（默认重启）。如果你选择 `dev` 频道但没有 git 检出，它会提示你创建一个。

## 工作原理

当你使用 `--channel` 显式切换频道时，OpenClaw 会自动匹配合适的安装方式：

- `dev` → 确保有 git 检出（默认位置 `~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 环境变量修改），更新代码后从该检出安装全局 CLI
- `stable`/`beta` → 使用对应的分发标签从 npm 安装

网关核心的自动更新器（通过配置启用时）也使用相同的更新路径。

## Git 检出流程

不同频道的更新策略：

- `stable`：检出最新的非测试版标签，然后构建并运行 `doctor`
- `beta`：检出最新的 `-beta` 标签，然后构建并运行 `doctor`
- `dev`：检出 `main` 分支，然后 fetch 并 rebase

详细流程：

1. 要求工作区干净（没有未提交的更改）
2. 切换到选定的频道（标签或分支）
3. 获取上游更新（仅 dev 频道）
4. 仅 dev 频道：在临时工作区进行预检 lint 和 TypeScript 构建；如果最新提交构建失败，会回溯最多 10 个提交找到最近的可构建版本
5. Rebase 到选定的提交（仅 dev 频道）
6. 安装依赖（优先使用 pnpm，备选 npm）
7. 构建项目及控制界面
8. 运行 `openclaw doctor` 作为最后的安全检查
9. 同步插件到当前频道（dev 使用内置扩展，stable/beta 使用 npm）并更新通过 npm 安装的插件

## --update 简写

`openclaw --update` 是 `openclaw update` 的简写形式，适用于 shell 脚本和启动器。

## 另请参阅

- `openclaw doctor`（对于 git 检出会先提示运行更新）
- [开发频道](../install/development-channels.md)
- [更新指南](../install/updating.md)
- [CLI 参考](../cli.md)

[uninstall](./uninstall.md)[voicecall](./voicecall.md)