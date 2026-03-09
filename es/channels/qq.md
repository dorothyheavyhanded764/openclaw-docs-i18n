

  Plataformas de mensajería

  
# QQ

La integración del canal QQ permite a los usuarios hablar con OpenClaw en el chat privado de QQ (Tencent). Cree un bot en la [plataforma abierta QQ](https://qq.open.tencent.com/) y luego configure el **App ID** y el **App Secret** en OpenClaw.

**La documentación completa está disponible en chino.** Consulte la [guía de integración de QQ en chino](../zh/channels/qq) para instrucciones paso a paso (registro en la plataforma QQ, creación del bot, configuración de AppID/AppSecret, lista blanca de IP y callback, configuración del canal OpenClaw).

* * *

## Referencia rápida

- **Crear un bot QQ** en la plataforma abierta QQ y copiar el **App ID** y el **App Secret**.
- **Configurar OpenClaw**: en la configuración de canales (o en el archivo de configuración), añadir el canal QQ con `appId` y `appSecret`.
- **Modo Webhook**: si usa un webhook para eventos, configure la URL de solicitud en la plataforma QQ y añada la lista blanca de IP requerida. Habilite los eventos C2C (chat privado).

[Feishu](../es/channels/feishu.md) | [WeCom](../es/channels/wecom.md) | [Discord](../es/channels/discord.md) | [Google Chat](../es/channels/googlechat.md)
