

  消息平台

  
# Zalo 个人账号

状态：实验性。此集成通过 OpenClaw 内置的原生 `zca-js` 自动化你的**个人 Zalo 账号**。

> **警告：** 这是一个非官方集成，可能导致账号被暂停或封禁。请自行承担使用风险。

## 安装插件

Zalo 个人账号以插件形式提供，不包含在核心安装包中。你需要先安装插件才能使用：

- 通过 CLI 安装：`openclaw plugins install @openclaw/zalouser`
- 或从源码目录安装：`openclaw plugins install ./extensions/zalouser`
- 详细说明请参阅 [插件](../tools/plugin.md)

安装后无需额外配置外部 `zca` 或 `openzca` CLI 二进制文件。

## 快速上手

如果你是第一次使用，按以下步骤即可快速完成配置：

1. 安装插件（见上文）。
2. 在网关机器上登录账号：
   - 运行 `openclaw channels login --channel zalouser`
   - 使用 Zalo 手机应用扫描显示的二维码
3. 在配置中启用频道：

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

4. 重启网关（或完成引导流程）。
5. 私信访问默认使用配对模式，首次联系时需要批准配对码。

## 工作原理

这个集成完全在 OpenClaw 进程内通过 `zca-js` 运行，无需外部依赖：

- 使用原生事件监听器实时接收消息
- 通过 JS API 直接发送回复（支持文本、媒体、链接）
- 专为无法使用 Zalo Bot API 的「个人账号」场景设计

## 为什么叫 zalouser

频道 ID 使用 `zalouser` 而非 `zalo`，是为了明确表示这是自动化**个人 Zalo 用户账号**的非官方方案。我们保留 `zalo` 这个名称，以便未来支持官方 Zalo API 集成。

## 查找联系人和群组 ID

配置访问控制时，你需要知道联系人和群组的 ID。使用目录 CLI 可以快速查找：

```bash
# 查看当前账号信息
openclaw directory self --channel zalouser

# 搜索联系人
openclaw directory peers list --channel zalouser --query "name"

# 搜索群组
openclaw directory groups list --channel zalouser --query "work"
```

## 已知限制

使用前请了解以下限制：

- 发送的消息会被自动分块，每段约 2000 字符（Zalo 客户端限制）
- 默认禁用流式输出功能

## 控制谁能私信你的智能体

通过配置 `dmPolicy`，你可以决定谁可以通过私信与你的智能体交互。`channels.zalouser.dmPolicy` 支持以下选项（默认为 `pairing`）：

- `pairing` — 首次联系需要批准配对码
- `allowlist` — 仅允许白名单中的用户
- `open` — 允许所有人
- `disabled` — 禁用私信功能

你还可以通过 `channels.zalouser.allowFrom` 指定允许的用户 ID 或名称。在引导过程中，名称会自动解析为 ID。

批准配对请求：

```bash
# 查看待批准的配对请求
openclaw pairing list zalouser

# 批准特定配对码
openclaw pairing approve zalouser <code>
```

## 控制智能体可以加入哪些群组

默认情况下，智能体可以加入任何群组（`groupPolicy = "open"`）。你可以通过以下方式限制群组访问：

**限制到白名单：**

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

**禁止所有群组：**

```json
{
  channels: {
    zalouser: {
      groupPolicy: "disabled",
    },
  },
}
```

配置向导会在设置时提示你输入群组白名单。启动时，OpenClaw 会将白名单中的群组/用户名称解析为 ID 并记录映射关系；无法解析的条目会保持原样。

### 群组中是否需要 @提及才回复

通过 `requireMention` 可以控制智能体在群组中是否只在被提及时才回复：

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

解析顺序：精确的群组 ID/名称 → 规范化的群组 slug → `*` 通配符 → 默认值（`true`）。此设置对白名单群组和开放群组模式都生效。

## 配置多个 Zalo 账号

如果你需要同时使用多个 Zalo 个人账号，可以通过配置多个 profile 实现：

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## 输入状态、表情反应和送达确认

为了提供更好的对话体验，`zalouser` 频道支持以下功能：

- **输入状态**：发送回复前，OpenClaw 会尽力发送「正在输入」状态
- **表情反应**：支持通过频道操作发送 `react` 动作
  - 使用 `remove: true` 可移除特定表情反应
  - 详细说明请参阅 [反应](../tools/reactions.md)
- **送达确认**：对于包含事件元数据的入站消息，OpenClaw 会尽力发送「已送达」和「已读」确认

## 常见问题

**登录状态无法保持：**

```bash
# 检查频道状态
openclaw channels status --probe

# 重新登录
openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser
```

**白名单或群组名称没有正确解析：**

在 `allowFrom` 或 `groups` 中使用数字 ID，或确保使用完全匹配的联系人/群组名称。

**从旧版 CLI 方案升级：**

如果你之前使用过基于外部 `zca` 进程的旧方案，请移除相关配置。新版本完全在 OpenClaw 内部运行，不再需要外部 CLI 二进制文件。

[Zalo](./zalo.md) [配对](./pairing.md)