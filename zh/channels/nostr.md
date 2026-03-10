

  消息平台

  
# Nostr

**状态：** 可选插件（默认禁用）。Nostr 是一个去中心化的社交网络协议。通过这个频道，你的智能体（agent）就能用 NIP-04 协议接收和回复加密私信了。

## 安装（按需）

### 引导设置（推荐）

想快速上手？用引导向导就行：

-   引导向导（`openclaw onboard`）和 `openclaw channels add` 都会列出可选的频道插件
-   选择 Nostr 后，系统会提示你安装插件

安装时的默认行为：

-   **开发频道 + 本地 git 仓库：** 直接使用本地插件路径
-   **稳定版/测试版：** 从 npm 下载

当然，你随时可以在提示中修改这个默认选择。

### 手动安装

```bash
openclaw plugins install @openclaw/nostr
```

开发时想用本地代码？可以这样链接：

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

安装或启用插件后，记得重启网关。

## 快速设置

四步搞定：

1.  生成 Nostr 密钥对（如果还没有的话）：

```bash
# 用 nak 工具生成
nak key generate
```

2.  写入配置文件：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3.  设置环境变量：

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4.  重启网关。

## 配置参考

这是所有可配置字段的说明：

| 键 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `privateKey` | 字符串 | 必填 | 私钥，支持 `nsec` 或十六进制格式 |
| `relays` | 字符串\[\] | `['wss://relay.damus.io', 'wss://nos.lol']` | 中继服务器地址（WebSocket） |
| `dmPolicy` | 字符串 | `pairing` | 私信访问策略 |
| `allowFrom` | 字符串\[\] | `[]` | 允许发送私信的公钥列表 |
| `enabled` | 布尔值 | `true` | 是否启用该频道 |
| `name` | 字符串 | \- | 显示名称 |
| `profile` | 对象 | \- | NIP-01 格式的个人资料元数据 |

## 个人资料元数据

个人资料会作为 NIP-01 的 `kind:0` 事件发布到网络。你可以通过控制界面（频道 → Nostr → 个人资料）来管理，也可以直接在配置文件中设置：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "你的智能助手",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

几点注意：

-   所有 URL 必须使用 `https://` 协议
-   从中继服务器导入资料时，会自动合并字段并保留本地的修改

## 访问控制

### 私信策略

想知道谁能给你的智能体发消息？通过 `dmPolicy` 来控制：

-   **pairing**（默认）：陌生人发消息时会收到一个配对码，需要验证后才能对话
-   **allowlist**：白名单模式，只有 `allowFrom` 列表里的公钥能发私信
-   **open**：完全开放，任何人都能发私信（需要同时设置 `allowFrom: ["*"]`）
-   **disabled**：关闭私信功能，忽略所有传入的消息

### 白名单配置示例

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## 密钥格式

系统支持以下格式：

-   **私钥：** `nsec...` 格式或 64 位十六进制字符串
-   **公钥（`allowFrom` 中使用）：** `npub...` 格式或十六进制字符串

## 中继服务器

默认使用 `relay.damus.io` 和 `nos.lol` 两个中继。

想自定义中继？这样配置：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

一些实用建议：

-   配置 2-3 个中继就够了，既能保证可用性又不会太复杂
-   不要贪多，中继太多会增加延迟和消息重复
-   预算允许的话，付费中继通常更稳定
-   本地开发测试可以用本地中继（`ws://localhost:7777`）

## 协议支持

| NIP | 状态 | 说明 |
| --- | --- | --- |
| NIP-01 | 已支持 | 基本事件格式 + 个人资料元数据 |
| NIP-04 | 已支持 | 加密私信（`kind:4`） |
| NIP-17 | 计划中 | Gift-wrapped 私信 |
| NIP-44 | 计划中 | 版本化加密 |

## 测试

### 本地中继测试

```bash
# 启动 strfry 中继服务器
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

然后在配置中指向本地：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### 手动测试流程

1.  从网关日志里找到机器人的公钥（npub 格式）
2.  打开任意 Nostr 客户端（比如 Damus、Amethyst）
3.  给机器人发条私信
4.  检查是否收到回复

## 故障排除

### 收不到消息？

按这个顺序排查：

-   检查私钥是否正确有效
-   确认中继地址可以访问，远程用 `wss://`，本地用 `ws://`
-   看看配置里的 `enabled` 是不是被设成了 `false`
-   翻一下网关日志，看有没有中继连接错误

### 发不出回复？

-   确认中继服务器允许写入操作
-   检查出站网络是否通畅
-   留意是否有中继的速率限制提示

### 收到重复回复？

-   用多个中继时会出现这种情况，属于正常现象
-   系统会根据事件 ID 去重，只有第一次投递会触发回复

## 安全提醒

-   **永远不要**把私钥提交到代码仓库
-   使用环境变量来管理密钥
-   生产环境的机器人建议用 `allowlist` 策略，限制谁能发消息

## 当前限制

-   只支持私信，暂不支持群聊
-   暂不支持媒体附件
-   目前只有 NIP-04（NIP-17 Gift-wrap 支持计划中）

[Nextcloud Talk](./nextcloud-talk.md)[Signal](./signal.md)