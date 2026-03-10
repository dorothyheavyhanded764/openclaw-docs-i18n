

  高级

  
# 开发频道

最后更新：2026-01-21 OpenClaw 提供三个更新频道：

- **stable**：npm dist-tag `latest`
- **beta**：npm dist-tag `beta`（测试中的构建）
- **dev**：`main` 分支的最新提交（git）。npm dist-tag：`dev`（发布时可用）

我们将构建发布到 **beta**，测试后**将经过验证的构建提升到 `latest`**，无需更改版本号——dist-tags 是 npm 安装的权威来源。

## 切换频道

Git checkout：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta` 检出最新的匹配标签（通常是同一个标签）。
- `dev` 切换到 `main` 并基于上游变基。

npm/pnpm 全局安装：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

这会通过对应的 npm dist-tag（`latest`、`beta`、`dev`）进行更新。当你使用 `--channel` **显式**切换频道时，OpenClaw 还会对齐安装方式：

- `dev` 确保存在 git checkout（默认在 `~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 覆盖），更新它，并从该 checkout 安装全局 CLI。
- `stable`/`beta` 使用匹配的 dist-tag 从 npm 安装。

提示：如果你想同时保留稳定版和开发版，请维护两个克隆，并将网关指向稳定版克隆。

## 插件与频道

当你使用 `openclaw update` 切换频道时，OpenClaw 还会同步插件源：

- `dev` 优先使用 git checkout 中的捆绑插件。
- `stable` 和 `beta` 恢复 npm 安装的插件包。

## 标签最佳实践

- 为希望 git checkout 定位到的版本打标签（稳定版用 `vYYYY.M.D`，测试版用 `vYYYY.M.D-beta.N`）。
- `vYYYY.M.D.beta.N` 为兼容性也被识别，但推荐使用 `-beta.N`。
- 旧版 `vYYYY.M.D-` 标签仍被识别为稳定版（非测试版）。
- 保持标签不可变：永远不要移动或重用标签。
- npm dist-tags 是 npm 安装的权威来源：
    - `latest` → 稳定版
    - `beta` → 候选构建
    - `dev` → main 分支快照（可选）

## macOS 应用可用性

测试版和开发版构建**可能不**包含 macOS 应用发布。这没关系：

- git 标签和 npm dist-tag 仍可正常发布。
- 在发布说明或更新日志中注明"此测试版无 macOS 构建"。

[在 Northflank 上部署](./northflank.md)