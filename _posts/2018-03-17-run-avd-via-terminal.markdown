---
layout: post
title: اجرا کردن امولاتور با ترمینال
date: 2018-03-17 1:20:00 +0330
category: blog
tags: امولاتور AVD اندروید
comments: true
---

وقتایی پیش میاد که نیاز نداریم سی ثانیه ای صبر کنیم تا اندروید استدیو باز شه تا بتونیم از طریق اون امولاتور یا همون
AVD (Android Virtual Device)
رو باز کنیم و دوباره یک دقیقه ای صرف این کنیم تا باز شه. که این زمان با وجود کوتاه بودنش، خیلی آزار دهندس.
پس برای اینکار کمی گشت و گذار کردم توی نت و دیدم دستوراتی برای اینکار وجود داره که برای اجرا کردن یه شبیه ساز نیازی نباشه اندروید استدیو رو باز کنیم.
<!--break-->

### شروع میکنیم

ترمینالو باز میکنیم و برای مشاهده لیست دستگاه هایی که تا بحال ساختیم از کامند زیر استفاده میکنیم:

```
➜  ~ emulator -list-avds
```
و خروجی هارو مشاهده میکنیم:
```
Nexus_5X_API_25
Nexus_6P_API_23
```
با دستور دوم میتونیم هرکدوم اینهارو که بخوایم اجرا کنیم:
```
➜  ~ /Users/farhad/Library/Android/sdk/tools/emulator -avd Nexus_5X_API_25
```
دقت کنید که اون آدرس اول دایرکتوری
SDK
نصبی هستش. اما خب وارد کردن همیشه ی این هم سخته و هم باعث میشه مشکلات زیادی ایجاد بشه.
راه حل چیه؟

### استفاده از alias
با کمک 
alias
ما میتونیم دستورات اختصاصی توی ترمینال برای خودمون تعریف کنیم تا هرزمانی که لازم شد با وارد کردن اون دستور اختصاصی کد هایی که بهش منتسب شده رو اجرا کنن.

به اینصورت که فایل کانفیگ ترمینال سیستممون به آدرس
`~/.bashrc`
رو باز میکنیم و داخل دوتا دستور تعریف میکنیم:
```
# alias for lunching AVD
alias avds="emulator -list-avds"
alias avd="/Users/farhad/Library/Android/sdk/tools/emulator -avd"
```
بعد از ذخیره ی این فایل  لازمه تا ترمینال آپدیت شه:
```
➜  ~ source ~/.bashrc
```
و پس از اون با وارد کردن دستور
```
➜  ~ avds   
```
لیست دستگاه هارو خواهیم دید و با دستور 
```
➜  ~ avd Nexus_5X_API_25
```
خواهیم دید که دستگاه مورد نظر اجرا خواهد شد.

ازون جهت که هدف من از درست کردن این بلاگ صرفا به اشتراک گذاری تجربه هام توی زمینه های مختلف برنامه نویسی و توسعه ی نرم افزار بخصوص اندروید هست، خیلی روی اینکه مطلبی در چه سطحی هست توجهی ندارم و صرفا هدف من نشر آموخته هامه.
بگذریم...

پیروز و پاینده باشید.