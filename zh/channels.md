

  概览

  
# 聊天频道

OpenClaw 可以在你日常使用的聊天应用中与你对话。每个频道（channel）都通过网关（Gateway）进行连接。所有频道都支持文本消息，但媒体文件和消息反应（reactions）的支持程度因频道而异。

## 支持的频道

- [BlueBubbles](./channels/bluebubbles.md) — **iMessage 推荐方案**。使用 BlueBubbles macOS 服务器的 REST API，功能支持最完整（包括消息编辑、撤回、特效、反应、群组管理 —— 注意：编辑功能目前在 macOS 26 Tahoe 上暂不可用）。
- [Discord](./channels/discord.md) — 通过 Discord Bot API 和网关连接，支持服务器、频道和私信（DM）。
- [飞书/Lark](./channels/feishu.md) — 通过 WebSocket 连接的飞书机器人（插件，需单独安装）。
- [Google Chat](./channels/googlechat.md) — 通过 HTTP webhook 连接的 Google Chat API 应用。
- [iMessage（旧版）](./channels/imessage.md) — 通过 imsg CLI 实现的旧版 macOS 集成（已弃用，新部署请使用 BlueBubbles）。
- [IRC](./channels/irc.md) — 经典 IRC 服务器，支持频道和私信，具备配对（pairing）和白名单控制功能。
- [LINE](./channels/line.md) — LINE Messaging API 机器人（插件，需单独安装）。
- [Matrix](./channels/matrix.md) — Matrix 协议（插件，需单独安装）。
- [Mattermost](./channels/mattermost.md) — 通过 Bot API 和 WebSocket 连接，支持频道、群组和私信（插件，需单独安装）。
- [Microsoft Teams](./channels/msteams.md) — Bot Framework，支持企业级功能（插件，需单独安装）。
- [Nextcloud Talk](./channels/nextcloud-talk.md) — 基于 Nextcloud Talk 的自建聊天服务（插件，需单独安装）。
- [Nostr](./channels/nostr.md) — 通过 NIP-04 协议实现的去中心化私信（插件，需单独安装）。
- [Signal](./channels/signal.md) — 基于 signal-cli，注重隐私保护。
- [Synology Chat](./channels/synology-chat.md) — 通过出站+入站 webhook 连接的 Synology NAS Chat（插件，需单独安装）。
- [Slack](./channels/slack.md) — 基于 Bolt SDK，支持工作空间（workspace）应用。
- [Telegram](./channels/telegram.md) — 基于 grammY 的 Bot API，支持群组功能。
- [Tlon](./channels/tlon.md) — 基于 Urbit 的通讯工具（插件，需单独安装）。
- [Twitch](./channels/twitch.md) — 通过 IRC 连接的 Twitch 聊天（插件，需单独安装）。
- [WebChat](./web/webchat.md) — 基于 WebSocket 的网关内置 WebChat 界面。
- [WhatsApp](./channels/whatsapp.md) — 最受欢迎的频道，使用 Baileys 库，需要扫码配对。
- [Zalo](./channels/zalo.md) — Zalo Bot API，越南流行的通讯工具（插件，需单独安装）。
- [Zalo Personal](./channels/zalouser.md) — 通过扫码登录的 Zalo 个人账户（插件，需单独安装）。

## 使用提示

- **多频道同时运行**：你可以同时配置多个频道，OpenClaw 会根据每个聊天的来源自动路由到对应频道。
- **快速上手推荐**：如果想最快体验，推荐从 **Telegram** 开始 —— 只需要一个简单的 Bot Token 即可。相比之下，WhatsApp 需要扫码配对，且会在本地存储更多状态数据。
- **群组功能差异**：不同频道的群组行为有所不同，详见 [群组](./channels/groups.md)。
- **安全机制**：私信配对（DM pairing）和白名单机制是强制启用的安全措施，详见 [安全设置](./gateway/security.md)。
- **问题排查**：遇到频道问题时，请参考 [频道故障排除](./channels/troubleshooting.md)。
- **模型提供商**：各频道支持的 AI 模型提供商有独立文档，详见 [模型提供商](./providers/models.md)。

[BlueBubbles](./channels/bluebubbles.md)