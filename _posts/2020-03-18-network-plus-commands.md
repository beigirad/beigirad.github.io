---
layout: post
title: NetworkPlus - Commands
category: notes
date: 2020-03-18T11:00:00
tags: comptia network-plus
comments: false
---
بررسی و توضیح برخی کامندهای پر استفاده و مفید
<!--break-->

## دیدن جدول ARP

```
$ arp -a
? (192.168.88.1) at 6c:3b:6b:e2:d3:25 on en0 ifscope [ethernet]
? (192.168.88.255) at ff:ff:ff:ff:ff:ff on en0 ifscope [ethernet]
```

## دیدن آی‌پی ها و مک کارت های شبکه
```
$ ifconfig
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether 78:4f:43:51:ba:45
	inet 192.168.88.254 netmask 0xffffff00 broadcast 192.168.88.255
	media: autoselect
	status: active
    ...
```

سویچ های بیشتر این کامند:
- release آی‌پیو ول کن
- renew یه آی‌پی بگیر از dhcp
- flushdns خالی کردن کش dns

## دستور پینگ
ping از پروتکل ICMP استفاده میکنه و در واقع چندتا بسته اکو میفرسته و میگیره و درحالت عادی آی‌پی رو resolve میکنه
```
$ ping 4.2.2.4
PING 4.2.2.4 (4.2.2.4): 56 data bytes
64 bytes from 4.2.2.4: icmp_seq=0 ttl=54 time=197.081 ms
64 bytes from 4.2.2.4: icmp_seq=1 ttl=54 time=442.273 ms
64 bytes from 4.2.2.4: icmp_seq=2 ttl=54 time=262.107 ms
^C
--- 4.2.2.4 ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 197.081/300.487/442.273/103.713 ms
```
سویچ های بیشتر
- a در واقع آدرس سایت رو resolve میکنه
- l سایز بسته رو مشخص میکنه
- ttl ویندوز ۱۲۸، اندروید ۶۴، سیسکو ۲۵۶ تاس

## دستور traceroute
میخوایم ببینیم چندتا روتر رو داره رد میکنه و هربارم یه ttl کم میشه ازش
- d بیخیال resolve کردن ادرس هر ادرس میشه

برای اینکار میتونیم از سایت [yougetsignal](yougetsignal.com/tools/visual-tracert) هم استفاده کنیم. و آی پی هارو هم میتونیم تو سایت ripe.net وارد کنیم و اطلاعاتشو بدست بیاریم

## دستور nslookup
با dns کار میکنه و ادرس و آی‌پی رو بهم تبدیل میکنه

میشه بهش بگیم از کدوم dns برای اینکار استفاده کنه.

## دستور netstat
برای بررسی پورهای باز و درحال استفاده ست

سویچ های زیادی داره و اسم برنامه و پراسس ایدی و انواع پروتکل های در حال استفاده رو هم نشون میده
