

  Plateformes de messagerie

  
# QQ

L’intégration du canal QQ permet aux utilisateurs de discuter avec OpenClaw en messagerie privée QQ (Tencent). Créez un bot sur la [plateforme ouverte QQ](https://qq.open.tencent.com/), puis configurez l’**App ID** et l’**App Secret** dans OpenClaw.

**La documentation complète est disponible en chinois.** Consultez le [guide d’intégration QQ en chinois](../zh/channels/qq) pour des instructions détaillées (inscription sur la plateforme QQ, création du bot, configuration AppID/AppSecret, liste blanche IP et callback, configuration du canal OpenClaw).

* * *

## Référence rapide

- **Créer un bot QQ** sur la plateforme ouverte QQ et copier l’**App ID** et l’**App Secret**.
- **Configurer OpenClaw** : dans la configuration des canaux (ou le fichier de config), ajouter le canal QQ avec `appId` et `appSecret`.
- **Mode Webhook** : si vous utilisez un webhook pour les événements, définir l’URL de requête sur la plateforme QQ et ajouter la liste blanche IP requise. Activer les événements C2C (chat privé).

[Feishu](../fr/channels/feishu.md) | [WeCom](../fr/channels/wecom.md) | [Discord](../fr/channels/discord.md) | [Google Chat](../fr/channels/googlechat.md)
