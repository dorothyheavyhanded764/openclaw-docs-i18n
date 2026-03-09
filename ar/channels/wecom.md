

  منصات المراسلة

  
# WeCom (WeChat للأعمال)

يتيح تكامل قناة WeCom (企业微信) للمستخدمين التحدث مع OpenClaw في **المجموعات** الخاصة بـ WeChat للأعمال بذكر البوت @. أنشئ بوتًا ذكيًا في وضع API من وحدة تحكم إدارة WeCom، ثم قم بتكوين **Token** و**Encoding-AESKey** في OpenClaw وتعيين عنوان URL الاستقبال على `http://<الخادم>:<المنفذ>/webhooks/wecom`.

**التوثيق الكامل متوفر بالصينية.** راجع [دليل تكامل WeCom بالصينية](../zh/channels/wecom) للحصول على تعليمات خطوة بخطوة (إنشاء البوت في وحدة تحكم WeCom، Token/Encoding-AESKey، تكوين القناة، URL، التحقق من النطاق).

* * *

## مرجع سريع

- **إنشاء البوت**: وحدة تحكم WeCom → 管理工具 → 智能机器人 → 创建机器人 → 手动创建 → API 模式创建؛ احصل على **Token** و**Encoding-AESKey** واحفظهما.
- **تكوين OpenClaw**: في إعدادات القنوات (أو ملف التكوين)، عيّن **企业微信** بنفس Token وEncoding-AESKey.
- **تعيين URL في WeCom**: في نموذج البوت بوضع API، عيّن URL على `http://:<المنفذ>/webhooks/wecom`، ثم أنشئ البوت.
- **التحقق**: أضف البوت إلى مجموعة واذكره @ للحصول على ردود متدفقة.

[Feishu](../ar/channels/feishu.md) | [QQ](../ar/channels/qq.md) | [Discord](../ar/channels/discord.md) | [Google Chat](../ar/channels/googlechat.md)
