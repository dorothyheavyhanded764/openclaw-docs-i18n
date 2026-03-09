

  Платформы обмена сообщениями

  
# QQ

Интеграция канала QQ позволяет пользователям общаться с OpenClaw в личном чате QQ (Tencent). Создайте бота на [платформе QQ Open Platform](https://qq.open.tencent.com/), затем настройте **App ID** и **App Secret** в OpenClaw.

**Полная документация доступна на китайском.** См. [руководство по интеграции QQ на китайском](../zh/channels/qq) для пошаговых инструкций (регистрация на платформе QQ, создание бота, настройка AppID/AppSecret, белый список IP и callback, настройка канала OpenClaw).

* * *

## Краткая справка

- **Создайте бота QQ** на платформе QQ Open Platform и скопируйте **App ID** и **App Secret**.
- **Настройте OpenClaw**: в конфигурации каналов (или в файле конфигурации) добавьте канал QQ с `appId` и `appSecret`.
- **Режим Webhook**: при использовании webhook для событий укажите URL запроса на платформе QQ и добавьте требуемый белый список IP. Включите события C2C (личный чат).

[Feishu](../ru/channels/feishu.md) | [WeCom](../ru/channels/wecom.md) | [Discord](../ru/channels/discord.md) | [Google Chat](../ru/channels/googlechat.md)
