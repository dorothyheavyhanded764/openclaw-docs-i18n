

  メッセージングプラットフォーム

  
# WeCom（企業向け WeChat）

WeCom（企業微信）チャネル統合により、企業向け WeChat の**グループチャット**でボットを @メンションして OpenClaw と会話できます。WeCom 管理コンソールで API モードのスマートボットを作成し、OpenClaw で **Token** と **Encoding-AESKey** を設定し、受信 URL を `http://<サーバー>:<ポート>/webhooks/wecom` に設定します。

**詳細なドキュメントは中国語で提供されています。** ステップバイステップの手順（企業微信管理后台でのスマートボット作成・Token/Encoding-AESKey・チャネル設定・URL 入力・ドメイン検証）は[中国語の WeCom 統合ガイド](../zh/channels/wecom)をご覧ください。

* * *

## クイックリファレンス

- **ボット作成**: WeCom 管理 → 管理ツール → スマートボット → ボット作成 → 手動作成 → API モード作成；**Token** と **Encoding-AESKey** をランダム取得して保存。
- **OpenClaw の設定**: チャネル設定（または設定ファイル）で、同じ Token と Encoding-AESKey で**企業微信**を設定。
- **WeCom で URL を設定**: API モードのボットフォームで、URL を `http://:<ポート>/webhooks/wecom` に設定し、ボットを作成。
- **検証**: ボットをグループに追加し、@メンションでストリーミング返答を確認。

[Feishu](../ja/channels/feishu.md) | [QQ](../ja/channels/qq.md) | [Discord](../ja/channels/discord.md) | [Google Chat](../ja/channels/googlechat.md)
