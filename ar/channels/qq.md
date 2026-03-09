

  منصات المراسلة

  
# QQ

يتيح تكامل قناة QQ للمستخدمين التحدث مع OpenClaw في الدردشة الخاصة لـ QQ (تينسنت). أنشئ بوتًا على [منصة QQ المفتوحة](https://qq.open.tencent.com/)، ثم قم بتكوين **App ID** و**App Secret** في OpenClaw.

**التوثيق الكامل متوفر بالصينية.** راجع [دليل تكامل QQ بالصينية](../zh/channels/qq) للحصول على تعليمات خطوة بخطوة (التسجيل في منصة QQ، إنشاء البوت، تكوين AppID/AppSecret، القائمة البيضاء لـ IP والـ callback، تكوين قناة OpenClaw).

* * *

## مرجع سريع

- **إنشاء بوت QQ** على منصة QQ المفتوحة ونسخ **App ID** و**App Secret**.
- **تكوين OpenClaw**: في إعدادات القنوات (أو ملف التكوين)، أضف قناة QQ مع `appId` و`appSecret`.
- **وضع Webhook**: إذا كنت تستخدم webhook للأحداث، فاضبط عنوان URL للطلب على منصة QQ وأضف القائمة البيضاء لـ IP المطلوبة. فعّل أحداث C2C (الدردشة الخاصة).

[Feishu](../ar/channels/feishu.md) | [WeCom](../ar/channels/wecom.md) | [Discord](../ar/channels/discord.md) | [Google Chat](../ar/channels/googlechat.md)
