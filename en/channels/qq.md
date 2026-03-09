

  Messaging platforms

  
# QQ

QQ channel integration lets users talk to OpenClaw in QQ (Tencent) private chat. You create a bot in the [QQ Open Platform](https://qq.open.tencent.com/), then configure the App ID and App Secret in OpenClaw.

**Full documentation is available in Chinese.** See the [QQ integration guide in Chinese](../zh/channels/qq) for step-by-step instructions (QQ 开放平台注册、创建机器人、配置 AppID/AppSecret、IP 白名单与回调、OpenClaw 通道配置).

* * *

## Quick reference

- **Create a QQ bot** at the QQ Open Platform and copy **App ID** and **App Secret**.
- **Configure OpenClaw**: In your deployment’s channel config (or config file), add the QQ channel with `appId` and `appSecret`.
- **Webhook mode**: If you use a webhook for events, set the request URL in the QQ platform and add the required IP allowlist. Enable C2C (private chat) events.

[Feishu](../en/channels/feishu.md) | [WeCom](../en/channels/wecom.md) | [Discord](../en/channels/discord.md) | [Google Chat](../en/channels/googlechat.md)
