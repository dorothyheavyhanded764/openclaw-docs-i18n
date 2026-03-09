

  Messaging platforms

  
# WeCom (Enterprise WeChat)

WeCom (企业微信) channel integration lets users talk to OpenClaw in Enterprise WeChat **group chats** by @mentioning the bot. You create an API-mode smart bot in the WeCom admin console, then configure **Token** and **Encoding-AESKey** in OpenClaw and set the receive URL to `http://:/webhooks/wecom`.

**Full documentation is available in Chinese.** See the [WeCom integration guide in Chinese](../zh/channels/wecom) for step-by-step instructions (企业微信管理后台创建智能机器人、Token/Encoding-AESKey、通道配置、URL 填写、域名校验说明).

* * *

## Quick reference

- **Create bot**: WeCom admin → 管理工具 → 智能机器人 → 创建机器人 → 手动创建 → API 模式创建；随机获取并保存 **Token** 和 **Encoding-AESKey**.
- **Configure OpenClaw**: In channel config (or config file), set **企业微信** with the same Token and Encoding-AESKey.
- **Set URL in WeCom**: In the API-mode bot form, set URL to `http://:/webhooks/wecom`, then create the bot.
- **Verify**: Add the bot to a group and @mention it for streaming replies.

[Feishu](../en/channels/feishu.md) | [QQ](../en/channels/qq.md) | [Discord](../en/channels/discord.md) | [Google Chat](../en/channels/googlechat.md)
