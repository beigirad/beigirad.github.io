---
layout: post
title: NetworkPlus - TCP/IP
category: notes
date: 2020-03-17T23:00:00
tags: comptia network-plus
comments: false
---
مدل TCP/IP که به نوعی پیاده‌ای از استاندارد OSI هست
<!--break-->

۴ لایه ست. و نسبت به OSI سه تا لایه بالا ترکیب شدن

لایه ۱ و ۲ هم ترکیب شدن ولی اینجا جدا بررسی میشه

## 1. Physical Layer
- فقط صفر و یک میفهمه. کابل شبکه و همه مدیا ها جزو این لایه‌ن
- ریپیتر توی این لایه کار میکنه که فقط سیگنال رو تقویت میکنه
- هاب همون ریپیتره با پورتای بیشتر و شبیه سه راهی برقه و چون هوشمند نیست جزو لایه ۱ درنظر میگیریمش

## 2. Data Link Layer
- همه داستانای OSI توش برقراره
- یکی از نمونه‌های Physical Address همون MAC address ۴۸ بیتی هست. که تبدیلش میکنیم به هگزادسیمال.
به بسته ۴ تایی بیت نیبل و ۸ تایی اکتت میگن.

اگر همشون ۱ باشن یا توی اکتت همشون f باشن اون پکت برودکست محسوب میشه.
این ادرس یونیکه تو کل دنیا. ۲۴ تا بیت اول رو یه سازمانی میده به کارخونه سازنده و ۲۴ تای بعدی دست کارخونه ست.
- ساختار یه فریم:

| 4 byte | 46-1500 byte | 2 byte | 8 byte | 8 byte |
|--------------|--------------|--------|--------------------|-------------------------|
| CRC checksum | Data | Type | Source MAC address | Destination MAC address |


- سویچ توی این لایه کار میکنه و میدونه کدوم کامپیوتر به کدوم پورت وصله و تو جدولش نگهش میداره

## 3. Internet Layer
- همه داستانای OSI توش برقراره
- یکی از راه‌های Logical addressing همون IP address هست.
- پروتکل های مسیریابی توی نتورک پلاس نیست. و توی سیسکو بهش پرداخته میشه
- مهمترین پروتکل‌های این لایه:
    - IP (v4, v6) - Internet Protocol
    - ICMP (v4, v6) - Internet Control Message Protocol توی ابزار های trace route استفاده میشه و برای عیب زدایی و این چیزاس
    - IGMP - Internet Control Management Protocol
    - IPSec - Internet Protocol Security شکل سکیور شده‌ی IP هستش
    - ARP - Address Resolution Protocol بهش آی‌پی ادرس میدیم مک تحویل میده. بیشتر بررسی میکنیم
- آی‌پی ۳۲ بیته اما برای خوندن باید دسیمالش کنیم و هشت تا هشت تا جدا میکنیم 
- هدر خودش رو به دیتایی که از بالا رسیده میچسبونه و پکت رو تشکیل میده.
- هدرش خیلی چرت و پرت داشت. اما شامل ورژن آی‌پی، طول هدر، نوع سرویس، طول کل پکت هست که بین ۲۰ تا ۶۵k میشه. ttl هم داریم که بسته‌ای سرگردان نمونه و هر مسیری رو بگذرونه یدونه از ttl ش کم میشه. checksum داره و ip فرستنده و گیرنده و در نهایت پدینگ داره که طول بسته ضریبی از ۳۲ باشه.
- روتر توی این لایه کار میکنه. البته الان روترا خیلی پیشرفته شدن و تو لایه های بالاتر هم کار میکنن. البته سویچ های پیشرفته هم داریم که تو لایه ۳ هم کار میکنه.

## 4. Transport Layer
- اینم همه داستانای OSI رو داره
- مهمترین پروتکلاش TCP و UDP عه
- پورت ادرس مجازی که توسط یه سازمانی مشخص میشه  ۱۶ بیته یا همون  65k عه
- کاربردش برای گرفتن سرویسای مختلف روی یه آی‌پیه و از طریق پورت مشخص میشن
- range 0-1023: well-known ports همشون قبلا استفاده شدن
- range 1024-49151: registered ports خریدنیه
- range > 49252: dynamic/private داخل سازمانین و هرکی میتونه استفاده میشه
- به ترکیب ip + port سوکت میگن که این فرمته: 192.168.1.2:2000
- Connection Oriented and Connectionless

|  | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---------------------|-----------------------------------------------------------------------------------------|----------------------------------|
| Connection-oriented | ینی قبل جابجایی دیتا باید ارتباط بگیریم با طرف مقابل تا مطمئن شیم طرف مقابل آمادگی داره | همینجوری میفرستیم امیدواریم برسه |
| Reliability | به معنی قابلیت اطمینان هست و این یعنی ارتباط ما بی نقص هست | همچین چیزی نداره |
| Flow Control | چک میکنیم طرف مقابل سرریز اطلاعات نداشته باشه و سرعت ها هماهنگ باشن و.. | نداره |
| Multiplexing | میشه به چند سرویس مختلف دسترسی پیدا کنیم | نداره |
| سرعت | کمتره ولی امنیت داره | کمتره ولی امنیت داره |


- TCP 3-Way Handshake
    1. Request for connection ->
        -  SYN with SEQ(A) = 10022
    2.  Response <-
        - SYN-ACK with SEQ(B)  = 90001
        - ACK(A) = 10023
    3. Connection established -> 
        - ACK with SEQ(A) = 10023
        - ACK(B) = 90002

بعد Handshake شروع میکنه به ارسال دیتا
اول یه سگمنت میفرسته. اک ۲ رو میگیره. سگمنت ۲ و ۳ رو میفرسته. اک ۴ میگیره و همینجوری تعداد سگمنتا دوبرابر میشه که بهش window size میگن.
اگر جایی به مشکل بخوره تا اخرین سگمنتی که گرفته بوده رو اک میفرسته و ازین ببعد window size نصف میشه.

وقتی تموم میشه هم پروسه finish رو شروع میکنه:
اول fin میفرسته. ack میگیره. fin میگیره و ack میفرسته. تمام

ولی تو UDP هیچ کدوم اینا مهم نیست.

هدر TCP شامل ایناس: شامل پورت مبدا و پورت مقصد، Sequence number, Acknowledgement, Checksum, Padding و خیلی چیزای دیگه
هدر UDP هم شامل پورت مبدا و مقصد و Checksum و طول سگمنت که پورت مبدا و Checksum هم ضروری نیستن و همینا باعث شدن سرعتش بالاتر باشه

## 5. Application Layer
- این لایه ترکیب سه لایه بالای OSI عه
- محل قرارگیری پروتکل هاست نه نرم‌افزارهایی که ازین پروتکلا استفاده میکنن
- مهمترین پروتکل‌های این لایه:

| Protocol | Port | Protocol Used | Description |
|----------|-------------------|---------------|------------------------------------------------------------|
| FTP | 21 | TCP | File Transfer Control |
| FTP-Data | 20 | TCP | File Transfer Data |
| SSH | 22 | TCP | Secure Shell |
| TELNET | 23 | TCP | like ssh but unsecure |
| SMTP | 25 | TCP | Simple Mail Transfer Protocol (send mail) |
| DNS | 53 | TCP and UDP | Domain Name System |
| DHCP v4 | 67 c2s - 68 s2c | UDP | Dynamic Host Configuration Protocol version 4 |
| DHCP v6 | 546 c2s - 547 s2c | UDP | Dynamic Host Configuration Protocol version 6 |
| TFTP | 69 | UDP | Rival File Transfer Protocol (unsecure FTP) |
| HTTP | 80 | TCP and UDP | Hypertext Transfer Protocol |
| POP3 | 110 | TCP | Post Office Protocol (complete receive mail) |
| IMAP | 143 | TCP | Internet Message Access Protocol (receive mail from cloud) |
| NTP | 123 | TCP | Network Time Protocol |
| HTTPS | 443 | TCP | Secure implementation of HTTP |
| RDP | 3389 | TCP | Remote Desktop Protocol |


DNS در واقع ادرس سایت رو به آی‌پی تبدیل میکنه. در واقع نام هارو ریسالو میکنه

یه مثال از یه ارتباط شبیه سازی شده:

```
L5 HTTP(GET HTTP)
L4 TCP(SRC_PORT:50000, DST_PORT:80) | GET HTTP
L3 IP(SRC_IP:10.10.10.40, DST_IP:10.10.10.50) | TCP(SRC_PORT:50000, DST_PORT:80) | GET HTTP
L2 ETHERNET(SRC_MAX:00:03:e2:32:32:44, DST_MAC: ??:??:??:??:??:??) | IP(SRC_IP:10.10.10.40, DST_IP:10.10.10.50) | TCP(SRC_PORT:50000, DST_PORT:80) | GET HTTP
L1 11110100001101000001010101011010101010

ARP
L3 IP(SRC_IP:10.10.10.40, DST_IP:10.10.10.50)
L2 ETHERNET(SRC_MAX:00:03:e2:32:32:44, DST_MAC: FF:FF:FF:FF:FF)
```