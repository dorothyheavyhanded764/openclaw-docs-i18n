

  消息平台

  
# Tlon

想把 OpenClaw 接入去中心化消息平台？Tlon 基于 Urbit 构建，OpenClaw 可以连接到你的 Urbit 飞船（ship），响应私信和群组聊天。群组消息默认需要 @ 提及才会回复，你也可以通过允许列表进一步限制访问权限。

**当前状态**：通过插件支持。支持私信、群组提及、主题回复、富文本格式和图片上传。反应和投票功能暂不支持。

## 需要安装插件

Tlon 以插件形式提供，不包含在核心安装包中。通过 CLI 安装：

```bash
openclaw plugins install @openclaw/tlon
```

如果你在本地 git 仓库中运行，可以这样安装：

```bash
openclaw plugins install ./extensions/tlon
```

了解更多：[插件](../tools/plugin.md)

## 快速设置

按照以下步骤完成配置：

1. 安装 Tlon 插件
2. 获取你的飞船 URL 和登录码
3. 配置 `channels.tlon`
4. 重启网关
5. 给机器人发私信，或在群组频道中 @ 提及它

最简配置示例（单账户）：

```json
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // 推荐：设置你的主飞船，始终拥有权限
    },
  },
}
```

## 私有网络和局域网飞船

出于 SSRF 防护考虑，OpenClaw 默认会阻止私有/内部主机名和 IP 地址的请求。如果你的飞船运行在私有网络（如 localhost、局域网 IP 或内部主机名），需要显式开启：

```json
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

适用于以下场景：

- `http://localhost:8080`
- `http://192.168.x.x:8080`
- `http://my-ship.local:8080`

⚠️ 仅在你信任本地网络时启用此选项，因为这会禁用对飞船 URL 请求的 SSRF 保护。

## 群组频道配置

默认情况下，频道自动发现功能是开启的。你也可以手动指定频道：

```json
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

如果不需要自动发现，可以关闭：

```json
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## 访问控制

### 私信允许列表

配置哪些飞船可以给你发私信。留空表示不允许任何私信，可以配合 `ownerShip` 实现审批流程：

```json
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

### 群组授权

群组默认采用受限模式，可以按频道配置访问规则：

```json
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## 所有者与审批系统

设置一个所有者飞船后，当未授权用户尝试与机器人交互时，所有者会收到审批请求：

```json
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

所有者飞船**自动获得所有场景的授权**：私信邀请自动接受，频道消息始终允许。你不需要把所有者加到 `dmAllowlist` 或 `defaultAuthorizedShips` 里。

设置后，所有者会在以下情况收到私信通知：

- 允许列表之外的飞船发送私信请求
- 在未授权频道中被 @ 提及
- 收到群组邀请请求

## 自动接受设置

自动接受私信邀请（仅对 `dmAllowlist` 中的飞船生效）：

```json
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

自动接受群组邀请：

```json
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
    },
  },
}
```

## 消息投递目标（CLI/cron）

配合 `openclaw message send` 或 cron 定时任务使用：

- 私信：`~sampel-palnet` 或 `dm/~sampel-palnet`
- 群组：`chat/~host-ship/channel` 或 `group:~host-ship/channel`

## 内置技能

Tlon 插件内置了一个技能包 [`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill)，提供丰富的 CLI 操作能力：

- **联系人**：获取/更新资料，列出联系人
- **频道**：列出、创建、发消息、获取历史记录
- **群组**：列出、创建、管理成员
- **私信**：发送消息，对消息添加反应
- **反应**：为帖子和私信添加/移除表情反应
- **设置**：通过斜杠命令管理插件权限

安装插件后，这个技能会自动可用。

## 功能支持情况

| 功能 | 状态 |
| --- | --- |
| 私信 | ✅ 支持 |
| 群组/频道 | ✅ 支持（默认需要 @ 提及） |
| 主题 | ✅ 支持（自动在主题内回复） |
| 富文本 | ✅ Markdown 自动转为 Tlon 格式 |
| 图片 | ✅ 上传到 Tlon 存储并嵌入 |
| 反应 | ✅ 通过[内置技能](#内置技能)支持 |
| 投票 | ❌ 暂不支持 |
| 原生命令 | ✅ 支持（默认仅限所有者） |

## 故障排除

遇到问题时，先按顺序执行以下排查命令：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

常见问题及解决方案：

- **私信被忽略**：发送者不在 `dmAllowlist` 中，且没有配置 `ownerShip` 来处理审批流程
- **群组消息被忽略**：频道未被自动发现，或发送者没有授权
- **连接错误**：检查飞船 URL 是否可访问；本地飞船需要启用 `allowPrivateNetwork`
- **认证错误**：确认登录码是否最新（登录码会定期轮换）

## 配置参数速查

完整配置说明见：[配置](../gateway/configuration.md)

| 参数 | 说明 |
| --- | --- |
| `channels.tlon.enabled` | 启用/禁用频道（channel） |
| `channels.tlon.ship` | 机器人的 Urbit 飞船名称，如 `~sampel-palnet` |
| `channels.tlon.url` | 飞船 URL，如 `https://sampel-palnet.tlon.network` |
| `channels.tlon.code` | 飞船登录码 |
| `channels.tlon.allowPrivateNetwork` | 允许 localhost/局域网 URL（绕过 SSRF 保护） |
| `channels.tlon.ownerShip` | 所有者飞船，用于审批系统（始终授权） |
| `channels.tlon.dmAllowlist` | 允许发送私信的飞船列表（留空则禁止所有） |
| `channels.tlon.autoAcceptDmInvites` | 自动接受允许列表中飞船的私信邀请 |
| `channels.tlon.autoAcceptGroupInvites` | 自动接受所有群组邀请 |
| `channels.tlon.autoDiscoverChannels` | 自动发现群组频道（默认 true） |
| `channels.tlon.groupChannels` | 手动指定的频道列表 |
| `channels.tlon.defaultAuthorizedShips` | 对所有频道有访问权限的飞船 |
| `channels.tlon.authorization.channelRules` | 按频道配置授权规则 |
| `channels.tlon.showModelSignature` | 在消息末尾附加模型名称 |

## 使用注意

- 群组回复需要 @ 提及机器人的飞船名称（如 `~your-bot-ship`）才会触发响应
- 如果收到的消息在主题（thread）中，OpenClaw 会自动在同一主题内回复
- Markdown 格式（粗体、斜体、代码、标题、列表）会自动转换为 Tlon 原生格式
- 图片 URL 会上传到 Tlon 存储并作为图片块嵌入消息

[Telegram](./telegram.md) [Twitch](./twitch.md)