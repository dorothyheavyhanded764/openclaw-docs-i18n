

  消息平台

  
# 企业微信

企业微信频道（channel）集成让用户可以在企业微信群聊中通过 **@机器人** 与 OpenClaw 智能体（agent）对话。你需要先在企业微信管理后台创建 API 模式的智能机器人，然后在 OpenClaw 中配置 `Token` 和 `Encoding-AESKey`，最后将接收消息的 URL 设置为 `http://<服务器IP>:<端口>/webhooks/wecom`。

* * *

## 快速参考

- **创建机器人**：企业微信管理后台 → 管理工具 → 智能机器人 → 创建机器人 → 手动创建 → API 模式创建；随机获取并保存 `Token` 和 `Encoding-AESKey`。
- **配置 OpenClaw**：在通道配置中找到 **企业微信**，填入相同的 `Token` 和 `Encoding-AESKey`。
- **设置 URL**：在 API 模式机器人表单中，将 URL 设为 `http://:<端口>/webhooks/wecom`，然后完成创建。
- **验证**：将机器人加入群聊，@机器人 发消息，检查是否能收到流式回复。

* * *

## 步骤 1：创建企业微信智能机器人

### 1. 打开企业微信管理后台

使用企业微信管理员账号登录 [企业微信管理后台](https://work.weixin.qq.com/)。

### 2. 创建机器人（API 模式）

1. 在左侧导航栏进入 **管理工具** → **智能机器人**。
2. 点击 **创建机器人**，选择 **手动创建**。
3. 选择 **API 模式创建**：
   - 点击 `Token` 右侧的「随机获取」，保存生成的 Token。
   - 点击 `Encoding-AESKey` 右侧的「随机获取」，保存 Encoding-AESKey。

**重要：** `Token` 与 `Encoding-AESKey` 需妥善保管，后续在 OpenClaw 与企业微信两侧配置中必须保持一致。

### 3. 暂不填写 URL

在 API 模式创建页面先不要提交。完成下方「步骤 2」在 OpenClaw 中配置通道后，再回到本页面填写 `URL` 并完成创建。

* * *

## 步骤 2：在 OpenClaw 中配置企业微信频道

### 通过控制台配置（推荐）

如果你的 OpenClaw 部署提供控制台（如应用详情中的通道配置）：

1. 打开 **通道配置** → **企业微信**。
2. 填入步骤 1 中获取的 `Token` 和 `Encoding-AESKey`。
3. 点击「应用」或保存。

### 通过配置文件配置

编辑 OpenClaw 配置文件（如 `~/.openclaw/openclaw.json` 或部署指定路径），在 `channels` 中增加企业微信配置：

```json
{
  "channels": {
    "wecom": {
      "enabled": true,
      "token": "您的Token",
      "encodingAesKey": "您的Encoding-AESKey"
    }
  }
}
```

具体键名以当前 OpenClaw 版本为准，请参考 [网关配置](../gateway/configuration.md) 与 [配置参考](../gateway/configuration-reference.md)。

### 确认接收地址与端口

OpenClaw 企业微信频道的 Webhook 路径一般为 `/webhooks/wecom`。请确认网关已启动，并记录其 **IP 地址** 与 **端口号**（端口可在部署环境或控制台查看），用于下一步在企业微信侧填写 URL。

* * *

## 步骤 3：在企业微信中填写接收 URL

1. 回到企业微信管理后台 **管理工具** → **智能机器人** → **API 模式创建** 页面。
2. 在 `URL` 一栏填写 OpenClaw 接收企业微信消息的地址，格式为：

   ```text
   http://<OpenClaw 服务器 IP 地址>:<端口号>/webhooks/wecom
   ```

   将 `` 和 `<端口号>` 替换为实际值。若企业微信与 OpenClaw 不在同一内网，需使用可被企业微信访问的公网 IP 或域名。

3. `Token` 和 `Encoding-AESKey` 保持与步骤 1 一致（与 OpenClaw 中配置相同）。
4. 填写机器人名称、简介及可见范围，点击 **创建** 完成保存。

* * *

## 验证集成效果

1. 在企业微信群聊中点击 **添加群成员**，搜索刚创建的机器人名称，将机器人加入群聊。
2. 在群内 **@机器人** 并发送消息，应能收到 OpenClaw 智能体的流式回复。
3. 若未响应，请确认网关已启动、频道已启用，并查看网关日志：`openclaw logs --follow`（若使用 CLI）。

* * *

## 故障排除

### 机器人无响应

- 确认企业微信侧 `URL` 填写正确，且企业微信能访问该地址（检查网络与防火墙）。
- 确认 OpenClaw 通道配置中的 `Token`、`Encoding-AESKey` 与企业微信后台完全一致。
- 确认网关已启动且企业微信频道已启用，查看日志排查鉴权或解码错误。

### 域名主体校验未通过

创建或保存企业微信智能机器人时，若出现「域名主体校验未通过，需配置备案主体与当前企业主体相同或有关联关系的域名」的报错，这是企业微信平台的要求：需使用**企业自有域名**作为接收消息 URL 的主机名。

- 若你已有备案且主体与当前企业一致（或有关联关系）的域名，可为该域名配置解析，指向运行 OpenClaw 的服务器或反向代理，并将企业微信中的 `URL` 改为使用该域名，例如：`http://airobot.您的域名.com/webhooks/wecom`。
- 若通过第三方连接流/代理服务接收消息，请按该服务商文档配置其提供的域名与回调地址。

### Token / Encoding-AESKey 泄露

- 在企业微信后台重新随机获取 `Token` 与 `Encoding-AESKey`。
- 在 OpenClaw 通道配置中更新为相同新值并保存，重启网关使配置生效。

* * *

## 配置参考

| 设置 | 描述 |
| --- | --- |
| `channels.wecom.enabled` | 是否启用企业微信频道 |
| `channels.wecom.token` | 企业微信机器人 Token |
| `channels.wecom.encodingAesKey` | 企业微信机器人 Encoding-AESKey |

完整配置项以 [网关配置参考](../gateway/configuration-reference.md) 为准。

* * *

## 定时任务与群机器人 Webhook（可选）

如果需要让 OpenClaw 定时执行任务并将结果推送到企业微信群聊，可结合群机器人的 `Webhook 地址` 使用：

1. 在群设置中开启「消息推送」，配置机器人名称与简介，复制该群机器人的 `Webhook 地址`。
2. 在 OpenClaw 对话中创建定时任务时，将任务内容与上述 `Webhook 地址` 一并提供给智能体，由智能体在任务执行后向该 Webhook 发送结果。
3. 修改或取消定时任务可直接在对话中告知智能体。

* * *

[飞书](./feishu.md) | [QQ](./qq.md) | [Discord](./discord.md) | [Google Chat](./googlechat.md)