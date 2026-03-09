

  Plataformas de mensajería

  
# WeCom (WeChat para empresas)

La integración del canal WeCom (企业微信) permite a los usuarios hablar con OpenClaw en **grupos** de WeChat para empresas mencionando al bot con @. Cree un bot inteligente en modo API en la consola de administración de WeCom, luego configure el **Token** y el **Encoding-AESKey** en OpenClaw y establezca la URL de recepción en `http://:/webhooks/wecom`.

**La documentación completa está disponible en chino.** Consulte la [guía de integración de WeCom en chino](../zh/channels/wecom) para instrucciones paso a paso (creación del bot en la consola WeCom, Token/Encoding-AESKey, configuración del canal, URL, verificación de dominio).

* * *

## Referencia rápida

- **Crear el bot**: consola WeCom → 管理工具 → 智能机器人 → 创建机器人 → 手动创建 → API 模式创建; obtener y guardar **Token** y **Encoding-AESKey**.
- **Configurar OpenClaw**: en la configuración de canales (o en el archivo de configuración), establecer **企业微信** con el mismo Token y Encoding-AESKey.
- **Establecer la URL en WeCom**: en el formulario del bot en modo API, establecer la URL en `http://:/webhooks/wecom`, luego crear el bot.
- **Verificar**: añadir el bot a un grupo y mencionarlo con @ para respuestas en flujo.

[Feishu](../es/channels/feishu.md) | [QQ](../es/channels/qq.md) | [Discord](../es/channels/discord.md) | [Google Chat](../es/channels/googlechat.md)
