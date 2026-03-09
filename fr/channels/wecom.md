

  Plateformes de messagerie

  
# WeCom (WeChat Entreprise)

L’intégration du canal WeCom (企业微信) permet aux utilisateurs de discuter avec OpenClaw dans les **groupes** WeChat Entreprise en @mentionnant le bot. Créez un bot intelligent en mode API dans la console d’administration WeCom, puis configurez le **Token** et l’**Encoding-AESKey** dans OpenClaw et définissez l’URL de réception sur `http://:/webhooks/wecom`.

**La documentation complète est disponible en chinois.** Consultez le [guide d’intégration WeCom en chinois](../zh/channels/wecom) pour des instructions détaillées (création du bot dans la console WeCom, Token/Encoding-AESKey, configuration du canal, URL, vérification du domaine).

* * *

## Référence rapide

- **Créer le bot** : console WeCom → 管理工具 → 智能机器人 → 创建机器人 → 手动创建 → API 模式创建 ; obtenir et enregistrer **Token** et **Encoding-AESKey**.
- **Configurer OpenClaw** : dans la configuration des canaux (ou le fichier de config), définir **企业微信** avec le même Token et Encoding-AESKey.
- **Définir l’URL dans WeCom** : dans le formulaire du bot en mode API, définir l’URL sur `http://:/webhooks/wecom`, puis créer le bot.
- **Vérifier** : ajouter le bot à un groupe et le @mentionner pour des réponses en flux.

[Feishu](../fr/channels/feishu.md) | [QQ](../fr/channels/qq.md) | [Discord](../fr/channels/discord.md) | [Google Chat](../fr/channels/googlechat.md)
