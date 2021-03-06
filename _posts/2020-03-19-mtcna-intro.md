---
layout: post
title: MTCNA - Inrto
category: notes
date: 2020-03-19T09:00:00
tags: mikrotik mtcna router-board routerOS
comments: false
---
مباحث پایه‌ای و شروع کار با روتربرد میکروتیک
<!--break-->

RouterOS: سیستم عاملی بر پایه هسته لینوکس که روی دستگاه های میکروتیک نصب شده. البته روی ماشین مجازیم نصب میشه

نصب این سیستم‌عامل ماشین مارو میتونه به یک روتر اختصاصی تبدیل کنه که پهنای باند رو مدیریت کنیم، فایروال راه‌اندازی کنیم و کلی کار دیگه

## RouterBOARD روتر سخت افزاری میکروتیک
و انواعی دارن:
- Integrated solutions همه چیو تو یه بسته قرار داده و فقط لازمه به برق بزنیمش
    - Ethernet router قابلیت وایرلس ندارن
    - Switch سویچ های لایه ۲یی که یه سریاشون SwOS روشون نصبه. اینا سویچ هایی هستن که یه روتر هم تو خودشون دارن
    - Wireless system اینا وایرلسن و برای محیطای outdoor استفاده میشن
    - Wireless for home and office که وایرلسن و برای محیطای indoorن
- RouterBOARD اینا فقط مین بوردن و بصورت بورد خالی به فروش میرسن
- Enclosures کیس هایی هستن که برای روتربورد هامون مناسبن
- Interfaces ماژولهای مخصوص فیبر و چیزای مختلفن
- Accessories انواع اداپتور و POE هاو..
- Antennas آنتنای مختلف


## نامگذاری
هرکدوم یه پروداکت کد دارن

RB: RouterBoard

CCR: Cloud Core Router

CRS: Cloud Router Switch 

2, 5: 2.4GHx or 5GHz radio

H: High power radio

HP: very High Power radio

SHP: Super High Power radio

n: 802.11n support

D: dual chain

3Digit: 
	hardware architecture
	interfaces number
	sluts number

A: more memory

H: more CPU power

S: SPF

U: USB

i: Single port power injector

P: more port injector

G: Gigabit ethernet

L: light edition


بیشتر از integrated ها استفاده کنیم. با اینکه کمتر توسعه پذیرن اما پاسخگوی بیشتر نیازامونه و قیمتشم پایینه

با کابل کنسول یا از طریق وایرلس بهشون وصل میشیم. این ارتباط هم میتونه لایه دویی باشه از طریق MAC و هم میتونه لایه سه‌یی باشه از طریق IP 

از طریق SSH یا Telnet و یا Winbox برای ویندوز  و Webfig برای وب هم میتونیم باهاشون ارتباط بگیریم

در ابتدا هم میتونن blank باشن و هیچ تنظیماتی نداشته باشن و هم میتونن یه سری تنظیمات اولیه داشته باشن. در صورت blank بودن چون هنوز IP نگرفته باید از طریق MAC بهش وصل شیم.

یوزرنیم اولیه admin عه و پسوردم نداره

## آپگرید کردن
برای دانلود کردن RouterOS نیاز به دونستن معماری روترمون داریم. (mipsbe, ppc, x86, tile, smips, arm)

[صفحه دانلود](https://mikrotik.com/download)

انواع فایلها:
- NPK پکیج های استاندارد مخصوص RouterOS
- ZIP پکیج های اضافه
- ChangeLog

## پکیج ها
یکسری از اینها بصورت دیفالت روی سیستم هستن و بعضیاشون که برای کاربردهای تخصصی تره رو بصورت کامل توی آدرس زیر ببینیم: 
[https://wiki.mikrotik.com/wiki/Manual:System/Packages](https://wiki.mikrotik.com/wiki/Manual:System/Packages)

برای دیدن اینکه چه پکیجایی روی روتر نصب شده به اینجا میریم
System > Packages 

برای آپدیت کردن سه روش وجود داره:
1. دانلود کردن فایلا و انتقال فایلای زیر به FIles و بعدش ریبوت کردن مودم. خودکار نصب میشه
    - routeros-mipsbe-xxxxx.npk
    - ntp-xxxxx-mipsbe.npk
2. System > Packages > Checking version
3. Auto upgrading: برای اینکار همه رو روی یه روتر سورس کپی میکنیم و به باقی روترا میگیم بره ازونجا بخونه و بعد ریبوت شه.  برای اینکارم System > Auto Upgrade

نکته‌ای که باید در نظر داشته باشیم بعد از آپگرید چک کنیم که frameware هم آپگرید شده باشه. برای اینکار باید بریم به System > RouterBoard و حتما آپگرید کنیم

برای دانگرید هم میتیونیم فایلارو کپی کنیم و از منوی Packages دانگرید کنیم و بعدش ریبوت

##  حساب‌های کاربری
روتر ما قابلیت داشتن چندین یوزر با سطوح دسترسی متفاوت رو داره که برای اینکار باید به System > Users بریم.

## IP Services با مدیریت سرویس IP میتونیم
- منابع مصرفی رو بهینه کنیم
- خطرات امنیتی رو با بستن پورتای باز کم کنیم و یا پورتهای TCP رو تغییر بدیم
- دسترسی به پنل رو محدود کنیم که روی سابنت‌های خاص در دسترس باشه

برای اینکار باید به System > Services میریم و هرچی میبینیم از TCP استفاده میکنن.

## نامگذاری روتر
System > Identity

## تنظیم زمان
برای اینکار از NTP استفاده میکنیم. روتر ما هم میتونه کلاینت باشه هم سرور. البته برای سرور بودن باید پکیج ntp نصب شده باشه. بای اینکارم باید به System > NTP server بریم.

برای اینکه به وقت محلی خودمون بشه میریم توی System > Clock بریم و همه چیو ست کنیم. 

اگر تایم درست نباشه Queue ها، فایروال و لاگ ها درست کار نمیکنن

## بک‌آپ گرفتن
۱. Binary backup این شامل بک‌آپ کامل از سیستم شامل پسورد ها که برا زمانیه که میخوایم به همون روتر برگردونیم. حتی مک آدرس هارو هم برمیداره

برای اینکار  به Files> Backup میریم و کاراشو انجام میدیم. قابلیت انکریپت هم داره که امن بمونه

کامند لاینم داره
```
backup save name=test
```

ریستور هم به همینصورته و تو همون Files  انجام میشه

۲. Export files این یکی به صورت plain text هست و شامل اسکریپت ها  برای انجام اون کانفیگ هاست. چون plain هست هم پسوردهارو نداره

این مدل قابلیت اینو داره که فقط از بخش خاصی بک‌آپ بگیریم.

قدومدل داره که اولی compact که کانفیگای پیش‌فرض رو نداره و تو حالت verbose این پیش‌فرضهارو هم شامل میشه.
این یکیو فقط تو کامند لاین میشه زد:

```
Export compact file=test
```
که خروجی یه فایل rsc هست و محتواش قابل دیدنه.

ریستور کردن هم با دستور import توی ترمینال انجام میشه

## RouterOS License

0.Trial Mode

1.Free Demo

3.WISP CPE

4.WISP

5.WISP

6.Controller

حالا هرکدوم اینا یه فرقایی میکنن مثلا لایسنس صفر همه چیش ۲۴ ساعته ست.

[اطلاعات بیشتر](https://wiki.mikrotik.com/wiki/Manual:License)

البته وقتی یه روتر میخریم یه لایسنس روی اون هست.

## Netinstall
یه وقتایی هست که وسط آپگرید کردن routerOS میپره و برای اینکار باید با کابل وصل میکنیم به کامپیوتر ویندوزی و یه برنامه به نام Netinstall هست و از طریق اون میایم routerOS رو نصب میکنیم.

از روی وایرلس هم نمیشه اینکارو کرد. و روی همه ether هاهم جواب نمیده و معمولا فقط اون اولی برای اینکاره.

## Forget Password & Reset configuration
زمانی که پسورد لاگینو فراموش کنیم باید OS رو مجدد نصب کنیم یت تنظیمات رو به صورت سخت‌افزاری ریست کنیم. 

داخل روتر دوتا بخش رو بهم اتصال کوتاه میدیم

یا اینکه از طریق GUI توی System > Reset Configuration  که خودش تنظیمات مختلفی هم داره

اینکار کامند هم داره