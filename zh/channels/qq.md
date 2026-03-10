

  消息平台

  
# QQ

本节介绍如何将 OpenClaw 与 QQ 集成，让用户可以在 QQ 单聊中与您的智能体（agent）对话。您需要在 QQ 开放平台创建机器人，获取 `App ID` 和 `App Secret`，然后在 OpenClaw 中配置 QQ 频道。如果使用 Webhook 接收事件，还需要在 QQ 开放平台配置请求地址和 IP 白名单。

* * *

## 快速开始

- **推荐方式**：若您的 OpenClaw 部署提供「频道配置」界面（如控制台中的频道配置），在 **QQ** 频道中填入机器人的 `App ID` 和 `App Secret` 并保存即可。
- **配置文件方式**：在 OpenClaw 配置中启用 QQ 频道并填写 `appId`、`appSecret`（见下方步骤 2）。

完成配置后启动或重启网关，在 QQ 中向机器人发送消息进行验证。

* * *

## 步骤 1：创建 QQ 机器人

### 1. 打开 QQ 开放平台

访问 [QQ 开放平台](https://qq.open.tencent.com/)，使用 QQ 扫码登录开发者账号。

### 2. 创建机器人

1. 在平台内找到「创建机器人」或「QQ 机器人」相关入口。
2. 按页面指引创建新的 QQ 机器人应用。
3. 创建完成后，在应用详情或凭证页面中获取并保存：
   - `App ID`
   - `App Secret`（机器人密钥）

**重要：** 请妥善保管 `App Secret`。QQ 机器人的 `App Secret` 不支持明文保存，首次查看或忘记后需重新生成，请勿泄露。

### 3. 配置 IP 白名单（仅 Webhook 模式）

若您通过 **Webhook 回调** 接收 QQ 事件（例如请求地址指向您的网关或第三方连接流服务），需在 QQ 开放平台的「开发管理」→「IP 白名单」中配置可访问回调地址的 IP：

- 若 OpenClaw 网关或代理直接对外提供 Webhook：添加该服务器的公网 IP。
- 若通过第三方连接流/转发服务接收 QQ 消息：按该服务商文档，添加其要求填写的 IP 白名单。

保存后，QQ 平台才会向您配置的请求地址推送事件。

### 4. 沙箱与审核

开发与测试阶段可在 QQ 开放平台使用沙箱环境。上线前请按平台要求完成应用审核与发布。

* * *

## 步骤 2：配置 OpenClaw

### 通过控制台配置（推荐）

若您的部署带有 OpenClaw 控制台（如应用详情中的频道配置）：

1. 打开 **频道配置** → **QQ**。
2. 填入创建机器人时获取的 `App ID` 和 `App Secret`。
3. 点击「应用」或保存。

### 通过配置文件配置

编辑 OpenClaw 配置文件（如 `~/.openclaw/openclaw.json` 或部署指定路径），在 `channels` 中增加 QQ 配置：

```json
{
  "channels": {
    "qq": {
      "enabled": true,
      "accounts": {
        "main": {
          "appId": "您的AppID",
          "appSecret": "您的AppSecret"
        }
      }
    }
  }
}
```

若使用环境变量，可配置：

```bash
export QQ_APP_ID="您的AppID"
export QQ_APP_SECRET="您的AppSecret"
```

具体键名以当前 OpenClaw 版本为准，请参考 [网关配置](../gateway/configuration) 与 [配置参考](../gateway/configuration-reference)。

* * *

## 步骤 3：配置回调（Webhook 模式）

若您使用 **Webhook** 方式接收 QQ 事件（而非内置长连接）：

1. 在 QQ 开放平台的「开发管理」中，找到「请求地址」或「事件回调」配置。
2. 将您的 Webhook URL 填入 **请求地址**。
   - 若 OpenClaw 网关直接提供 Webhook：格式一般为 `http(s)://您的网关公网IP:端口/qq/events`（以实际路由为准）。
   - 若通过第三方连接流转发：使用该平台提供的 Webhook URL。
3. 勾选 **单聊事件** 中的 **C2C 消息事件**（私聊消息），以便机器人接收用户消息。
4. 保存后，QQ 会向该地址推送消息事件，OpenClaw 即可处理并回复。

* * *

## 方案验证

1. 确认网关已启动：`openclaw gateway status`（若使用 CLI）。
2. 在 QQ 中搜索或打开您创建的机器人，发送一条私聊消息。
3. 若配置正确，机器人应能收到消息并由 OpenClaw 回复。
4. 若未响应，请查看网关日志：`openclaw logs --follow`，并检查下方故障排除。

* * *

## 故障排除

### 机器人收不到消息

- 确认 QQ 开放平台中「请求地址」已保存且可公网访问（Webhook 模式）。
- 确认已勾选 C2C 单聊消息事件。
- 若使用 IP 白名单，确认当前请求来源 IP 已加入白名单。
- 确认 OpenClaw 网关正在运行，且 QQ 频道已启用。

### 鉴权失败或回调不通过

- 确认 `App ID`、`App Secret` 填写正确，无多余空格。
- 若平台有「校验令牌」或签名校验，请在 OpenClaw 配置中填写与平台一致的校验参数（参见当前版本 QQ 频道文档）。

### App Secret 泄露

- 在 QQ 开放平台重置 `App Secret`（机器人密钥）。
- 在 OpenClaw 配置或控制台中更新为新的 `App Secret`。
- 重启网关使配置生效。

* * *

## 配置参考

| 设置 | 描述 |
| --- | --- |
| `channels.qq.enabled` | 是否启用 QQ 频道 |
| `channels.qq.accounts..appId` | QQ 机器人 `App ID` |
| `channels.qq.accounts..appSecret` | QQ 机器人 `App Secret` |

完整配置项以 [网关配置参考](../gateway/configuration-reference) 为准。

* * *

[飞书](../channels/feishu) | [企业微信](../channels/wecom) | [Discord](../channels/discord) | [Google Chat](../channels/googlechat)