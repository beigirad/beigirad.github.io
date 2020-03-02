---
layout: post
title: RecyclerView Tips
category: blog
tags: android recycler-view tips
comments: true
---
این مطلب تکمیل نشده
<!--break-->
تسک امروزم بهبود یک ویو بود که یک ریسایکلر ویو به عنوان
parent
و چند تا ریسایکلرویوی دیگه به عنوان 
child 
داشت که به 
nested recyclerview 
شناخته میشن.

کمی مشکل لگ داشت که تا الآن نتونستم بهبودش بدم :) ولی توی این پست میخوام مسیری که طی کردم رو بگم.

در ابتدا با 
[این مقاله][1]
روبرو شدم که چندتا راه حل برای بهتر کردن پرفورمنس پیشنهاد میده.

**هدفم در اینجا توضیح/ترجمه این مقاله نیست بلکه اون بخشی از مسیره و میخوام دقیقتر نکاتی از ریسایکلرویو رو بررسی کنیم**

1. [setHasStableIds][android-stable-id]
این متد در واقع به میگه که هر ویوهلدر رو میخوایم به یک 
id 
یونیک و ثابت متصل کنیم. بعد از 
true 
این مقدار هم باید حتما 
`long getItemId(int position)`
رو 
override 
کنیم و برای هر کدوم یک 
id 
مشخص کنیم یا بسازیم.

حالا کاربردش چیه؟










[1]:https://blog.usejournal.com/improve-recyclerview-performance-ede5cec6c5bf
[android-stable-id]: https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter#sethasstableids
[android-item-id]: https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter#getitemid