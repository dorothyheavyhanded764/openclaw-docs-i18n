

  其他安装方式

  
# Bun（实验性）

本节介绍如何使用 **Bun** 运行 OpenClaw（可选方案，不推荐用于 WhatsApp/Telegram）。⚠️ **不推荐将 Bun 作为网关运行时**（WhatsApp/Telegram 存在已知 bug），生产环境请使用 Node。

## 当前状态

- Bun 是可选的本地运行时，用于直接运行 TypeScript（`bun run …`、`bun --watch …`）。
- `pnpm` 仍是默认构建工具并保持完整支持（部分文档工具也依赖它）。
- Bun 无法使用 `pnpm-lock.yaml`，会将其忽略。

## 安装

默认方式：

```bash
bun install
```

注意：`bun.lock`/`bun.lockb` 已被 git 忽略，因此不会产生仓库变更。如果你希望**完全不写入锁文件**：

```bash
bun install --no-save
```

## 构建 / 测试（Bun）

```bash
bun run build
bun run vitest run
```

## Bun 生命周期脚本（默认阻止）

除非明确信任（`bun pm untrusted` / `bun pm trust`），否则 Bun 会阻止依赖的生命周期脚本。对于本仓库，通常被阻止的脚本并非必需：

- `@whiskeysockets/baileys` 的 `preinstall`：检查 Node 主版本 >= 20（我们运行的是 Node 22+）。
- `protobufjs` 的 `postinstall`：输出关于不兼容版本方案的警告（不产生构建产物）。

如果遇到确实需要这些脚本的运行时问题，请明确信任它们：

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## 注意事项

- 部分脚本仍硬编码使用 pnpm（如 `docs:build`、`ui:*`、`protocol:check`），目前请通过 pnpm 运行这些命令。

[Ansible](./ansible.md)[更新](./updating.md)