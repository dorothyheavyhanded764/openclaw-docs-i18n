

  发布说明

  
# 发布清单

本文档帮助你完成 OpenClaw 的版本发布流程，包括版本号管理、构建验证、npm 发布以及 macOS 应用分发。请在仓库根目录下使用 `pnpm`（需要 Node 22+），并在打标签或发布前确保工作区干净。

## 操作员触发

当操作员说「发布」时，立即执行以下预检步骤（除非遇到阻塞，否则无需额外询问）：

- 阅读本文档和 `docs/platforms/mac/release.md`
- 从 `~/.profile` 加载环境变量，确认 `SPARKLE_PRIVATE_KEY_FILE` 和 App Store Connect 相关变量已设置（`SPARKLE_PRIVATE_KEY_FILE` 应该放在 `~/.profile` 中）
- 如有需要，使用 `~/Library/CloudStorage/Dropbox/Backup/Sparkle` 中的 Sparkle 密钥

### 1. 版本与元数据

这一步确保版本号和包信息正确更新。

- [ ] 更新 `package.json` 中的版本号（例如 `2026.1.29`）
- [ ] 运行 `pnpm plugins:sync` 同步扩展包版本和更新日志
- [ ] 更新 [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) 中的 CLI 版本字符串，以及 [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts) 中的 Baileys 用户代理
- [ ] 确认包元数据（name、description、repository、keywords、license）正确，且 `bin` 映射指向 [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)
- [ ] 如果依赖有变更，运行 `pnpm install` 确保 `pnpm-lock.yaml` 是最新的

### 2. 构建与产物

这一步生成发布所需的构建产物。

- [ ] 如果 A2UI 输入有变更，运行 `pnpm canvas:a2ui:bundle` 并提交更新后的 [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js)
- [ ] 运行 `pnpm run build`（重新生成 `dist/` 目录）
- [ ] 验证 npm 包的 `files` 字段包含所有必需的 `dist/*` 文件夹（特别是无头节点和 ACP CLI 所需的 `dist/node-host/**` 和 `dist/acp/**`）
- [ ] 确认 `dist/build-info.json` 存在且包含预期的 `commit` 哈希（npm 安装时 CLI 横幅会用到）
- [ ] 可选：构建后运行 `npm pack --pack-destination /tmp`，检查 tarball 内容并留作 GitHub 发布使用（**不要提交**）

### 3. 更新日志与文档

这一步确保用户能清楚了解本次发布的变更。

- [ ] 更新 `CHANGELOG.md`，写入面向用户的亮点（文件不存在则创建），条目按版本号降序排列
- [ ] 确保 README 中的示例和参数与当前 CLI 行为一致（特别是新命令或选项）

### 4. 验证

这一步确保发布质量，避免引入问题。

- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test`（需要覆盖率输出则用 `pnpm test:coverage`）
- [ ] `pnpm release:check`（验证 npm pack 内容）
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke`（Docker 安装冒烟测试，快速路径，发布前必跑）
  - 如果上一个 npm 发布版本已知有问题，可设置 `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` 或 `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` 跳过预安装步骤
- [ ] （可选）完整安装程序冒烟测试（增加非 root 和 CLI 覆盖）：`pnpm test:install:smoke`
- [ ] （可选）安装程序端到端测试（Docker 环境，运行 `curl -fsSL https://openclaw.ai/install.sh | bash`，完成引导后执行真实工具调用）：
  - `pnpm test:install:e2e:openai`（需要 `OPENAI_API_KEY`）
  - `pnpm test:install:e2e:anthropic`（需要 `ANTHROPIC_API_KEY`）
  - `pnpm test:install:e2e`（需要两个密钥，同时测试两个提供商）
- [ ] （可选）如果改动影响发送/接收路径，抽查 Web 网关

### 5. macOS 应用 (Sparkle)

这一步处理 macOS 应用的签名和分发准备。

- [ ] 构建并签名 macOS 应用，然后压缩以供分发
- [ ] 使用 [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) 生成 Sparkle appcast（包含 HTML 说明）并更新 `appcast.xml`
- [ ] 准备好应用 zip 文件（以及可选的 dSYM zip 文件）用于 GitHub 发布附件
- [ ] 具体命令和环境变量参考 [macOS 发布](../platforms/mac/release.md)
  - `APP_BUILD` 必须是数字且单调递增（不能有 `-beta`），这样 Sparkle 才能正确比较版本
  - 如需公证，使用从 App Store Connect API 环境变量创建的 `openclaw-notary` 钥匙串配置文件（参见 [macOS 发布](../platforms/mac/release.md)）

### 6. 发布到 npm

这一步将包发布到 npm 仓库。

- [ ] 确认 git 状态干净，按需提交并推送
- [ ] 如需要，运行 `npm login`（确认 2FA 已启用）
- [ ] `npm publish --access public`（预发布版本用 `--tag beta`）
- [ ] 验证发布结果：`npm view openclaw version`、`npm view openclaw dist-tags`，以及 `npx -y openclaw@X.Y.Z --version`（或 `--help`）

#### 故障排除（来自 2.0.0-beta2 发布的经验）

**`npm pack`/`npm publish` 卡住或 tarball 异常巨大**

原因：`dist/OpenClaw.app` 中的 macOS 应用包（以及发布 zip 文件）被打包进去了。解决方法是在 `package.json` 的 `files` 字段中白名单指定发布内容（包含 dist 子目录、文档、skills，排除应用包）。用 `npm pack --dry-run` 确认 `dist/OpenClaw.app` 未被列出。

**`dist-tags` 操作时 npm auth 陷入网页循环**

使用旧版认证方式获取 OTP 提示：

```bash
NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest
```

**`npx` 验证失败，提示 `ECOMPROMISED: Lock compromised`**

用新缓存重试：

```bash
NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version
```

**后期修复后需要重新指向标签**

强制更新并推送标签，然后确认 GitHub 发布资源仍然匹配：

```bash
git tag -f vX.Y.Z && git push -f origin vX.Y.Z
```

### 7. GitHub 发布与 appcast

这一步完成 GitHub Release 并更新 Sparkle feed。

- [ ] 打标签并推送：`git tag vX.Y.Z && git push origin vX.Y.Z`（或 `git push --tags`）
- [ ] 为 `vX.Y.Z` 创建或更新 GitHub Release，**标题设为 `openclaw X.Y.Z`**（不只是标签名）；正文应包含该版本的**完整更新日志**（亮点 + 变更 + 修复），直接内联展示（不要只放链接），**正文中不要重复标题**
- [ ] 附加产物：`npm pack` tarball（可选）、`OpenClaw-X.Y.Z.zip` 和 `OpenClaw-X.Y.Z.dSYM.zip`（如已生成）
- [ ] 提交更新后的 `appcast.xml` 并推送（Sparkle 从 main 分支读取）
- [ ] 在一个干净的临时目录（没有 `package.json`）中运行 `npx -y openclaw@X.Y.Z send --help`，确认安装和 CLI 入口点正常
- [ ] 发布发布说明

## 插件发布范围 (npm)

我们只发布 `@openclaw/*` 作用域下**已在 npm 上存在的插件**。未发布到 npm 的捆绑插件保持**仅磁盘树分发**（仍随 `extensions/**` 一起发布）。获取待发布列表的方法：

1. 运行 `npm search @openclaw --json` 获取包名列表
2. 与 `extensions/*/package.json` 中的名称对比
3. 只发布**交集部分**（即已在 npm 上的）

当前 npm 插件列表（按需更新）：

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

发布说明中还需说明**新增的可选捆绑插件**（即默认不启用的，例如 `tlon`）。

[致谢](./credits.md)[测试](./test.md)