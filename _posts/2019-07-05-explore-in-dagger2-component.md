---
layout: post
title: Explore in Dagger - Component
category: blog
tags: android di dependency-injection dagger2 component
comments: true
---
همونطور که قبلا هم گفته شده:
> کامپوننت ها عامل اصلی 
injection
هستند.

پس قطعا پیچیدگی‌ها و نکات بیشتری برای بیان دارند.

در این مطلب ابتدا نگاهی به ساختار کلی آن خواهیم داشت و سپس موارد مختلف را پیاده‌سازی خواهیم کرد.
<!--break-->

## ساختار Component

```java
@Component(modules = { ... })
public abstract class AppComponent {
    abstract public void inject(MainActivity mainActivity);
}
```
```java
public final class DaggerAppComponent extends AppComponent {
  private DaggerAppComponent() {}

  public static Builder builder() {
    return new Builder();
  }

  @Override
  public void inject(MainActivity mainActivity) {
      ...
  }

  public static final class Builder {
    private Builder() {}

    public AppComponent build() {
      return new DaggerAppComponent();
    }
  }
}
```
```java
public class MainActivity extends AppCompatActivity {
    AppComponent mAppComponent;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        mAppComponent = DaggerAppComponent.builder()
                .build();
        mAppComponent.inject(this);
    }
}
```
از آنجایی که کامپوننت ها همواره 
abstract-class 
هستند (یا هم 
interface)
، در فرآیند 
code generation 
کلاسی با پیشوند 
Dagger 
آنها را 
implement 
میکند. و در اینجا نیز کلاس 
`DaggerAppComponent` 
که 
`AppComponent` 
را 
implement 
کرده که متدهای
override 
شده از داخل کلاسی که نیازمند 
injection 
هست فراخوانی میشود.
(`inject(MainActivity mainActivity)`)

 نکته قابل توجه بعدی وجود یک 
constructor 
با دسترسی 
private 
است که به صورت مستقیم برای کلاسهایی که به 
injection 
نیاز دارند، قابل دسترسی نیست. و برای 
instantiate 
کردن این کلاس باید از 
builder 
آن استفاده کرد.

## Component Builder

> پترن 
Builder 
ساختار 
(construction) 
اشیاء پیچیده را از نمایش 
(representation) 
آن جدا کرده و شی پیچیده را به صورت جز به جز میسازد.

اما این 
Component 
چه مواردی را نیاز دارد؟ (چه موراردی را در 
constructor 
خود میگیرد؟)

#### Non-static Module
ماژول‌های 
non-static
که نیاز به 
instantiate 
دارند. 
[کامیت](https://github.com/beigirad/SemanticDagger/commit/829edeaef4bb6a9f6b5911d4c2312f6e5936715f)

```java
public final class DaggerAppComponent extends AppComponent {
  private final AppModule appModule;

  private DaggerAppComponent(AppModule appModuleParam) {
    this.appModule = appModuleParam;
  }

  public static Builder builder() {
    return new Builder();
  }

  public static AppComponent create() {
    return new Builder().build();
  }

  private CoffeeMaker getCoffeeMaker() {
    return AppModule_ProvideCoffeeMakerFactory.provideCoffeeMaker(
        appModule, new ElectricHeater(), new Pump());
  }

  @Override
  public void inject(MainActivity mainActivity) {
    injectMainActivity(mainActivity);
  }

  private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectMCoffeeMaker(instance, getCoffeeMaker());
    return instance;
  }

  public static final class Builder {
    private AppModule appModule;

    private Builder() {}

    public Builder appModule(AppModule appModule) {
      this.appModule = Preconditions.checkNotNull(appModule);
      return this;
    }

    public AppComponent build() {
      if (appModule == null) {
        this.appModule = new AppModule();
      }
      return new DaggerAppComponent(appModule);
    }
  }
}
```

همانطور که از کد مشخص است، 
instance
ی از ماژول 
non-abstract 
را نگهداری میکند. و این فیلد را در 
builder
خود مقدار دهی میکند.
[لینک](https://github.com/beigirad/SemanticDagger/blob/829edeaef4bb6a9f6b5911d4c2312f6e5936715f/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L46-L49)

اما در صورتیکه این متد 
builder 
فراخوانی و مقداردهی نشود در متد 
`build()` 
براحتی مقداردهی میشوند.
[لینک](https://github.com/beigirad/SemanticDagger/blob/829edeaef4bb6a9f6b5911d4c2312f6e5936715f/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L51-L56)

#### Non-instantiative arguments in factories

Factory 
هایی که نیاز به آرگومان هایی دارند که توسط دیگر 
factory 
تا تامین نشده‌اند و همانطور که گفته شد، از طریق کامپوننت تامین میشوند. مانند 
`Context` یا `Application`

برای پیاده سازی این مورد آرگومانی از جنس 
`Context` 
به 
CoffeeMaker 
اضافه میکنیم. 
[کامیت](https://github.com/beigirad/SemanticDagger/commit/393e4fb6402bff15bd420c183b69fffbca539f8c)
```java
public class CoffeeMaker {
    private final Heater heater;
    private final Pump pump;
    private final Context context;

    public CoffeeMaker(Heater heater, Pump pump, Context context) {
        this.heater = heater;
        this.pump = pump;
        this.context = context;
    }

    public void brew() { ... }
}
```

برای تامین اینگونه پارامترها دوراه وجود دارد:

* اولین مورد که در نسخه‌های قبلی تنها راه اینکار بوده ولی همچنان مرسوم است، استفاده از ماژول‌های 
non-abstract 
که این آرگومان را در 
constructor 
خود دریافت کنند. 
[کامیت](https://github.com/beigirad/SemanticDagger/commit/3746ab8e66b373b109f12d2bc89ab25373a64645)

```java
@Module(includes = {SecondModule.class})
public class AppModule {

    Context context;

    public AppModule(Context context) {
        this.context = context;
    }

    @Provides
    public CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
        return new CoffeeMaker(heater, pump, context);
    }
}
```
بدین صورت وظیفه تامین 
Context
را بر عهده کامپوننت - که ماژول را 
instantiate 
میکند،- گذاشته‌ایم. اما باید در نظر داشته باشیم که کامپوننت‌ها تنها ماژول‌هایی که آرگومانی روی 
constructor 
خود نداشته باشند را میتوانند 
instantiate 
کنند. پس این وظیفه به خود کاربر محول میشود و 
builder 
کامپوننت به صورت زیر تغییر میکند 
[لینک](https://github.com/beigirad/SemanticDagger/commit/3746ab8e66b373b109f12d2bc89ab25373a64645#diff-153586f439f6466ff3ff17068e1db9d9)
```java
public static final class Builder {
    private AppModule appModule;

    private Builder() {}

    public Builder appModule(AppModule appModule) {
      this.appModule = Preconditions.checkNotNull(appModule);
      return this;
    }

    public AppComponent build() {
      Preconditions.checkBuilderRequirement(appModule, AppModule.class);
      return new DaggerAppComponent(appModule);
    }
  }
```

همانطور که میبنید دیگر متد 
`build` 
ماژول را 
instantiate 
نکرده و تنها نال بودن آن را چک میکند. پس در کلاسی که میخواهیم 
injection 
را انجام دهیم باید متد 
`appModule(AppModule appModule)` 
را فراخوانی کنیم.
```java
mAppComponent = DaggerAppComponent.builder()
      .appModule(new AppModule(this))
      .build();

mAppComponent.inject(this);
```

* اما راه دوم برای تامین 
Context 
که در اینجا به عنوان یک 
dependency 
غیرقابل 
instantiate 
شدن استفاده میشود، استفاده از 
`@Component.Builder` 
میباشد.

# @Component.Builder
این 
annotation 
که بعد ها به 
Dagger 
اضافه شد. امکان تعریف یک 
interface 
توسط کاربر برای تامین برخی نیازمندی‌های کامپوننت معرفی شد.

بدین صورت که در هر کامپوننت اگر یک 
interface 
با 
`@Component.Builder` 
علامتگذاری شود، 
builder 
اصلی کامپوننت آن را 
implement 
میکند.

```java
@Component(modules = {AppModule.class})
public abstract class AppComponent {

    @Component.Builder
    public interface AppComponentBuilder {
        @BindsInstance
        public AppComponentBuilder builderContext(Context context);

        public AppComponent buildAppComponent();
    }

    abstract public void inject(MainActivity mainActivity);

}
```
نکته قابل توجه این است که ساختار پترن 
builder 
به طور کامل در این 
interface 
پیاده‌سازی شده و همه متدها بغیر از متد 
`build` 
آن که خود کامپوننت را 
return 
میکند باقی متدها که 
dependency 
هارا تامین میکنند باید دارای 
`@BindsInstance` 
باشند و همچنین نوع بازگشتی نیز خود 
interface 
باشد.

```java
public final class DaggerAppComponent extends AppComponent {
  private final Context builderContext;

  private final AppModule appModule;    // non-abstract module

  private DaggerAppComponent(AppModule appModuleParam, Context builderContextParam) {
    this.builderContext = builderContextParam;
    this.appModule = appModuleParam;
  }

  public static AppComponent.AppComponentBuilder builder() {
    return new Builder();
  }

  @Override
  public void inject(MainActivity mainActivity) { ...  }

  private static final class Builder implements AppComponent.AppComponentBuilder {
    private Context builderContext;

    @Override
    public Builder builderContext(Context context) {
      this.builderContext = Preconditions.checkNotNull(context);
      return this;
    }

    @Override
    public AppComponent buildAppComponent() {
      Preconditions.checkBuilderRequirement(builderContext, Context.class);
      return new DaggerAppComponent(new AppModule(), builderContext);
    }
  }
}
```

در اینجا نیز شاهد این هستید متد 
`builder()` 
اصلی کامپوننت بجای بازگرداندن 
`Builder` 
نوع 
`AppComponent.AppComponentBuilder`
برمیگرداند و کاربر ملزم به فراخوانی متدهای موردنیاز میباشد. 
[لینک](https://github.com/beigirad/SemanticDagger/blob/ec5e70d826d03de983dbcc3d3a6cb242b43dc370/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L22-L24)

لازم به ذکر است که در صورت استفاده از 
`@Component.Builder` 
نیز اگر ماژول‌های 
non-abstract 
داشته باشیم که در 
constructor 
آرگومانی نداشته باشند، به صورت خودکار توسط کامپوننت 
instantiate 
میشوند. 
[لینک](https://github.com/beigirad/SemanticDagger/blob/ec5e70d826d03de983dbcc3d3a6cb242b43dc370/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L53)

---
تمامی فایل های این پست روی برنچ
[p3-component](https://github.com/beigirad/SemanticDagger/tree/p3-component)
در دسترس است.