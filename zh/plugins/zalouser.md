

  扩展

  
# Zalo 个人插件

通过插件（plugin）为 OpenClaw 提供 Zalo 个人账户支持，使用原生 `zca-js` 库自动化普通 Zalo 用户账户。

> **警告：** 非官方自动化可能导致账户被暂停或封禁，使用风险自负。

## 命名说明

频道 ID 为 `zalouser`，以明确表示这是自动化**个人 Zalo 用户账户**（非官方）。我们保留 `zalo` 用于未来可能的官方 Zalo API 集成。

## 运行位置

此插件运行在**网关进程内部**。如果你使用远程网关，需要在**运行网关的机器上**安装并配置它，然后重启网关。无需外部的 `zca`/`openzca` CLI 二进制文件。

## 安装

### 方式 A：从 npm 安装

```bash
openclaw plugins install @openclaw/zalouser
```

安装后重启网关。

### 方式 B：从本地文件夹安装（开发模式）

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

安装后重启网关。

## 配置

频道配置位于 `channels.zalouser` 下（而非 `plugins.entries.*`）：

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## 智能体工具

工具名称：`zalouser`

操作：`send`、`image`、`link`、`friends`、`groups`、`me`、`status`

频道消息操作也支持 `react` 用于消息表情回应。

[语音通话插件](./voice-call.md)[插件清单](./manifest.md)