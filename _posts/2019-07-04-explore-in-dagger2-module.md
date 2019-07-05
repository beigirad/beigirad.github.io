---
layout: post
title: Explore in Dagger - Module
category: blog
tags: android di dependency-injection dagger2 module
comments: true
---
همانطور که قبلا گفته شد، ماژول ها یک ساختار مجازی هستند که در زمان 
build 
هر تابع آن به یک 
Factory 
تبدیل میشود. در این پست امکانات و ویژگی های مختلف ماژول‌ها را بررسی خواهیم کرد.
<!--break-->

اگر هنوز مطلب قبلی رو نخوندید، بهتره بعد از مطالعه‌ی اون برگردید و اینجارو ادامه بدید.
{% include referred.html
    url="https://beigirad.ir/blog/explore-in-dagger2-factory"
    title="Explore in Dagger - Introduction (Factory)"
%}

## Multi Module

برای داشتن چندین ماژول مختلف، لیست ماژول‌های موردنیاز یک کامپوننت را به صورت زیر 
`@Component(modules = {...})` 
نام میبریم.
[لینک](https://github.com/beigirad/SemanticDagger/commit/aa22b6a5e22a7d0c19e9b39562752ecb55e70c6d)

```java
@Component(modules = {AppModule.class, SecondModule.class})
public abstract class AppComponent {

    abstract public void inject(MainActivity mainActivity);

}
```

```java
@Module
public interface SecondModule {
    @Binds
    Heater bindsHeater(ElectricHeater electricHeater);
}
```

## Child Module
این ساختار نیز همانند ساختار قبلیست اما میتوان آنها را به صورت سلسله مراتبی نیز معرفی کرد.
[لینک](https://github.com/beigirad/SemanticDagger/commit/ef6c2d9925997e736e7e4e7d5434121a6e919fe2)

```java
@Module(includes = {SecondModule.class})
public abstract class AppModule {

    @Provides
    public static CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
        return new CoffeeMaker(heater, pump);
    }
}
```

## Abstract/Non-abstract Module
کلاس ماژول را میتوان به صورت 
abstract 
تعریف کرد (تا حالا در مثالها به این صورت تعریف کرده‌ایم).

در این صورت متدهایی که میتوانیم در کلاس ماژول بنویسیم باید یکی از دو حالت زیر باشند:
- abstract باشند.
این متدها که معمولا برای مواردی ست که بخواهند با
`@Bind` 
یک کلاس دیگر را به یک اینترفیس 
bind 
کنند.
- non-abstract و static باشند.
این متد ها معمولا برای متدهایی‌ ست که با 
`@Provides` 
علامتگذاری شده باشند.

بدلیل اینکه ماژول 
abstract 
است، قابل 
instantiate 
نیست. اما بدلیل اینکه متدها 
static 
هستند براحتی در 
Factory 
ساخته شده قابل دسترسی هستند. 
[لینک](https://github.com/beigirad/SemanticDagger/blob/37dfe2022bf7f3c8faab41289770b8741f111ea3/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/AppModule_ProvideCoffeeMakerFactory.java#L34)

```java
public final class AppModule_ProvideCoffeeMakerFactory implements Factory<CoffeeMaker> {
  ...
  public static CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
    return Preconditions.checkNotNull(
        AppModule.provideCoffeeMaker(heater, pump),
        "Cannot return null from a non-@Nullable @Provides method");
  }
}
```

اما در مواردی که ماژول 
abstract 
نباشد، 
Factory 
هایی که نیاز به آن 
type 
داشته باشند باید به یک 
instance 
از آن ماژول دسترسی داشته باشند تا بتوانند تابع مورد نظر خود را صدا کرده و مقدار مورد نظر را دریافت کنند. 
[کامیت](https://github.com/beigirad/SemanticDagger/commit/0b5797a18daf8d16d46b454af442a1d27092d8b7) 

```java
public final class AppModule_ProvideCoffeeMakerFactory implements Factory<CoffeeMaker> {
  private final AppModule module;

  private final Provider<Heater> heaterProvider;

  private final Provider<Pump> pumpProvider;

  public AppModule_ProvideCoffeeMakerFactory(
      AppModule module, Provider<Heater> heaterProvider, Provider<Pump> pumpProvider) {
    this.module = module;
    this.heaterProvider = heaterProvider;
    this.pumpProvider = pumpProvider;
  }

  @Override
  public CoffeeMaker get() {
    return provideCoffeeMaker(module, heaterProvider.get(), pumpProvider.get());
  }

  public static AppModule_ProvideCoffeeMakerFactory create(
      AppModule module, Provider<Heater> heaterProvider, Provider<Pump> pumpProvider) {
    return new AppModule_ProvideCoffeeMakerFactory(module, heaterProvider, pumpProvider);
  }

  public static CoffeeMaker provideCoffeeMaker(AppModule instance, Heater heater, Pump pump) {
    return Preconditions.checkNotNull(
        instance.provideCoffeeMaker(heater, pump),
        "Cannot return null from a non-@Nullable @Provides method");
  }
}
```

همانگونه که در کامیت آخر نیز قابل مشاهده ست، یک فیلد از نوع خود ماژول به 
Factory 
ساخته شده از آن اضافه شده. همچنین تابع 
`provideCoffeeMaker`
نیز تامین 
CoffeeMaker 
را از همان آرگومان انجام میدهد.

**اما چه کسی این فیلد را تامین کرده‌است؟**

باید توجه کرد که چه کلاسی نیازمندی‌های
MemberInjector 
هارا تامین کرده و عمل 
injection 
را انجام میدهد؟
> کامپوننت ها عامل اصلی 
injection
هستند.

در مطالب بعدی ساختار دقیق کامپوننت را بررسی خواهیم کرد.

## Instantiate/Non-instantiate Arguments
تمامی مواردی که به عنوان 
dependency 
قابل تامین هستند از لحاظ 
instantiate 
به دو دسته تقسیم میشوند.

- کلاس‌هایی که 
**فاقد آرگومان** 
در 
constructor 
خود هستند. مانند 
[Pump](https://github.com/beigirad/SemanticDagger/blob/p1-factory/app/src/main/java/ir/beigirad/semanticdagger/model/Pump.java)

این موارد به راحتی با 
`@Inject` 
و یا 
`@Provides` 
آماده ارائه میشوند.
-  کلاس‌هایی که 
**دارای آرگومان** 
دارند. مانند 
[CoffeeMaker](https://github.com/beigirad/SemanticDagger/blob/p1-factory/app/src/main/java/ir/beigirad/semanticdagger/model/CoffeeMaker.java)

لازم به ذکر است که تمامی این آرگومان‌ها به صورت
`Provider<T>` 
از جنس آن آرگومان در کلاس 
Factory 
نگهداری میشوند.

```java

public final class AppModule_ProvideCoffeeMakerFactory implements Factory<CoffeeMaker> {
  private final Provider<Heater> heaterProvider;

  private final Provider<Pump> pumpProvider;

  public AppModule_ProvideCoffeeMakerFactory(Provider<Heater> heaterProvider, Provider<Pump> pumpProvider) {
    this.heaterProvider = heaterProvider;
    this.pumpProvider = pumpProvider;
  }
    ...
}

```

این کلاس‌ها در صورتی که آرگومان‌های آنها قابل 
instantiate 
شدن باشند، به صورت آبشاری 
instantiate 
میشوند و به کلاس وابسته پاس داده میشوند. اما در صورتی که این آرگومان‌ها همانند 
`Context` 
یا 
`Application` 
قابل 
instantiate 
کردن نباشند باید از طریق کامپوننت تامین شوند که در مطلب بعدی به آن خواهیم پرداخت.



## جمع‌بندی
- ماژول های 
abstract 
نمیتوانند متدهای تامین کننده
non-static 
داشته باشند چرا که از ماژول‌ها 
instantiate 
نمیشود و نمیتوان به این متدها دسترسی پیدا کرد.
- Factory 
هایی که از ماژول‌های 
non-abstract 
به وجود آمده‌اند یک رفرنس از ماژول در 
Factory 
خود دارند تا متد مورد نظر خود را فراخوانی کنند. و این نیازمندی از طریق کامپوننت تامین میشود.
- Factory 
هایی که از ماژول‌های 
abstract 
به وجود آمده‌اند به صورت مستقیم متد مورد نظر خود را فراخوانی میکنند.
- نیازمندی‌/آرگومان های
non-instantiative 
مربوط به 
Factory 
ها از طریق کامپوننت تامین میشوند.

---
تمامی فایل های این پست روی برنچ
[p2-module](https://github.com/beigirad/SemanticDagger/tree/p2-module)
در دسترس است.