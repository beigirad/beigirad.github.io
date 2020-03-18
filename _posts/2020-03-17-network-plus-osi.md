---
layout: post
title: NetworkPlus - OSI
category: note
date: 2020-03-17T20:00:00
tags: comptia network-plus
comments: false
---
شامل بررسی هفت لایه استاندارد OSI و مهمترین وظایف هر لایه
<!--break-->

## 7. Application Layer
محل قرارگیری پروتکل های شبکه ست(نه برنامه های کاربردی)

پروتکل: قرارداد در نحوه مبادله اطلاعات و نوع عملیات برای انتقال یا دریافت

## 6. Presentation Layer
- Encryption/Decryption
- Compression/Expansion
- Encoding/Decoding
- Formatting

## 5. Session Layer
- Establishing/Maintaining and ending session
- Simplex/Half-duplex/Full-duplex
- Synchronization : تو مدتی که ارتباط برقراره مشکلی پیش نیاد. دیتارو پیج بندی میکنه و اگر مشکلی پیش بیاد اونو مجددا درخواست میکنه

## 4. Transport Layer لایه میانی 
- Logical Port Address مشخص میکنه این پیام به کی تحویل شه
- Connection Oriented and Connectionless ارتباط امن و سرعت پایین. ارتباط ناامن و سرعتبالا. مثل گرفتن رسید از مقصد
- Error Control دیتای خام رو از نظر ارور چک میکنه
- Flow Control داده سرریز نداشته باشه و سرعت رو چک میکنه
- Buffering وقتی خط و مدیا ازاد نیست نگه داشته بشه تا خط ازاد شه

## 3. Network Layer
- Logical Addressing آدرسی که خودمون رو دستگاه ست میکنیم. یکیش آی‌پی آدرس
- Routing
- Packet Reordering پکت ها شماره گذاری میشن و اگر ترتیب رعایت نشد یا لاست شد دوباره درخواست میشه 
- Error Control
- Flow Control

## 2. Data Link Layer
- Physical Addressing
- Baseband and Broadband
- Error Control (Trailer) CRC - FCS
- Flow Control
- Arbitration متد دسترسی به مدیا تعیین میشه

## 1. Physical Layer
- Lan Topology Types
- Simplex/Half-duplex/Full-duplex
- Synchronization of bits
- Point to Point / Multicast
- Data Rate

Encapsulation: دیتای ما از لایه ۷ به ۶ و ازون به ۵ منتقل میشه. همش خام. از ۴ به بعد بسته بندی شروع میشه. هدر اضافه میکنه و بسته بندی میکنه بهش میگه سگمنت. لایه سه هم دوباره هدر میزنه و بسته بندیم میکنه بهش میگه پکت. لایه دو هم فریمش میکنه و اولش هدر و اخرش تریلر میچسبونه و تحویل لایه فیزیکی میده. فیزیکم به صورت باینری میده میره رو مدیا

Decapsulation: برعکس روال بالا

به سگمنت، پکت و فریم به طور کلی Protocol Data Unit یا PDU گفته میشه