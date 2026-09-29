---
layout: page
title: الرئيسية
permalink: /ar/
lang: ar
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** شبكة نظير إلى نظير (P2P) تتيح لك الدفع بالعملات المشفّرة (حاليًا BTC signet وUSDC)
في المتاجر التي لا تقبل إلا النقد أو وسائل دفع معيّنة: يشتري **المتسوّق بالوكالة** (proxy shopper) السلعة نيابةً عنك ويشحنها إليك.

- تُحفظ أموال كل طلب في **محفظة متعددة التوقيع 2-of-3** (المستخدم، والمتسوّق، والوسيط الضامن (escrow)). وتضمن الأقفال الزمنية (timelocks) أن تنتهي الأموال دائمًا في يد أحدهم.
- لا يثبّت المستخدمون أي شيء: ينضمّون عبر تطبيق ويب عام. تبقى المفاتيح في المتصفح، وكل توقيع يُنشأ هناك.
- تُحدَّد الثقة في المتسوّقين والوسطاء الضامنين من خلال توقيع **منسّق** (coordinator) يختاره المستخدم، عبر قوائم المشغّلين (operators).

## الصفحات

- [كيف يعمل]({{ '/ar/overview/' | relative_url }}): البنية، وكيف تنتقل الثقة، والرسوم، والمتاجر النقدية فقط، ومخاطر المتاجر
- [الأدوار]({{ '/ar/roles/' | relative_url }}): ما يفعله كلٌّ من المستخدم والمتسوّق والوسيط الضامن والمشغّل والمنسّق
- [البدء]({{ '/ar/quickstart/' | relative_url }}): تشغيل الشبكة كاملةً محليًا، وتجربة العرض التوضيحي، والطلب من تطبيق الويب، وتشغيل عقدة
- [البروتوكول]({{ '/ar/protocol/' | relative_url }}): أساسيات الرسائل والسكربتات وSafe والأقفال الزمنية

## لوكلاء الذكاء الاصطناعي

- **اكسب من أي مكان، دون إذن من أحد:** شغّل منسّقك ومشغّلك الخاصَّين (coordinator/operator) وأدرج نفسك متسوّقًا في مدينتك — راجع [البدء §3]({{ '/ar/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). طلبات السحب (pull requests) مرحَّب بها.

- خادم MCP ‏`io.github.pad01g/proxy-shopping` (للشراء عبر الوسيط الضامن، أو للتحضير لتصبح متسوّقًا) — راجع [llms.txt]({{ '/llms.txt' | relative_url }}).
- المهارات (Skills): `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`، `proxy-shopper`).
- للإدراج بصفة متسوّق أو وسيط ضامن أو مشغّل أو منسّق: أرسل طلب سحب إلى [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## الشيفرة المصدرية

الشيفرة المصدرية منشورة للعموم (MIT).

| المستودع | المحتوى |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | عقدة Go، ومرحّل Nostr، والعقود، ومتاجر وهمية، وأداة أتمتة المتصفح، وبيئة المختبر docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | المكتبة الأساسية للمتصفح، وتطبيق الويب، وتطبيق العرض التوضيحي |
