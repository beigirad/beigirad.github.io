---
layout: post
title: NetworkPlus - IP v6
category: notes
date: 2020-03-18T10:00:00
tags: comptia network-plus
comments: false
---
درباره ساختار IP ورژن ۶
<!--break-->

بدلیل تموم شدن v4 رفتن رو v6

ورژن ۴ در واقع ۳۲ بیت بود و کلا 2^32 تا میشد ولی اونایی که برا ما قابل استفادن خیلی کمتره

برای کند کردن روند تموم شدن دنبال چیزای مختلفی رفتن

ورژن ۶ هم ۱۲۸ بیتی ست و خیلی زیاده :)) 2^128

از نظر هدرها، سبک تر شده اما طول خود  IP بیشتر شده

توی این IP دیگه broadcast نداریم.

## ساختار

آی‌پی ورژن ۴ شامل ۴ قسمت ۸ بیتی بود. اما ورژن ۶ شامل ۸ قسمت ۳۲ بیتی ست که هر بخش بصورت هگزادسیمال نوشته میشه که طولش ۴ بشه. یعنی ۸ بخش ۴ کاراکتری

میشه صفر های اول هر سکشن رو حذف کنیم

اگر چندتا سکشن همشون صفر باشن، ما میتونیم یه دنبالشونو حذف کنیم.

```
2001:0000:0000:ff00:023e:0000:0000:8329
2001::ff00:23e:0000:0000:8329
2001:0000:0000:ff00:23e::8329
```
از بین ۱۲۸ بیت ۶۴ تاش به یوزر میرسه و IANA ,ISP و باقی موارد ۶۴ بیتش رو مشخی میکنن
مواردی مشابه DHCP و موارد دیگه برای اختصاص آی‌پی وجود داره و مثلا توی Static with EUI خود ماشین با ۶۴ بیتی کردن MAC خودش، اونو به ادامه آی‌پی ورژن ۶ اضافه میکنه که ۱۲۸ بیت رو کامل میکنه.

| Name | IP v6 | IP v4 similar |
|---------------------|----------|---------------|
| Loopback | ::1/128 | 127.0.0.1 |
| Unspecified address | ::/128 | 0.0.0.0 |
| Global Unicast | 2000::/3 | Public IP |
| Unique Local | fd00::/8 | Private IP |
| Multicast | ff0x: |  |