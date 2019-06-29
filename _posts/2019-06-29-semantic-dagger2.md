---
layout: post
title: Semantic Dagger
category: blog
tags: android di dependency-injection dagger2
comments: true
---
اگر شما توسعه‌دهنده اندروید باشید حتما اسم
Dagger
رو شنیدید. همونطور که میدونیم از محبوبترین فریمورک های
DI
هست که تا امروز بیشترین استفاده بین دیگر کتابخونه/فریمورک هارو داره. دلایل این محبوبیت هم مواردی چون حل مشکلات در
build time
،سرعت بالا و نداشتن سربار اضافه در
runtime
هست. دلیل این موارد هم جنریت کردن کدهای انجام
DI 
در زمان build هست.

توی این پست به بررسی کدهای تولید شده بپردازیم.
<!--break-->

## تعریف Dependency
طبق تعریف عام، این عبارت یعنی وقتی کلاس
A
به کلاس
B
وابستگی داشته باشد، بتوانیم با تکنیکی خاص این وابستگی را تامین کنیم.

![DI Concept](/assets/posts/semantic-dagger2/di-concept.png)

تا به اینجا، موضوع مورد توجه ما، ایجاد نمونه‌ای از کلاس
B
میباشد و کلاس 
A
بدون اینکه بداند این وابستگی از کجا تامین شده، فقط از آن استفاده کند.

پس ما به یک کارخونه یا
Factory
ای برای ساخت نمونه‌ای از کلاس
B
نیاز داریم.

```java
public interface Provider<T> {
    T get();
}

public interface Factory<T> extends Provider<T> {
}
```
پس به ازای هر
type
یا کلاسی که قرار است به عنوان وابستگی دیگر کلاس ها تامین شود، باید این اینترفیس
Factory
را
implement 
کند.

توی کتابخونه
Dagger
برای تامین وابستگی ها یا به عبارتی برای داشتن یک
Factory
برای هر
type
یی سه آپشن در اختیار داریم:

- `@Inject`
بالای متد
constructor
هر کلاس قرار میگیرد و به ازای آن یک کلاس
Factory
از جنس همان کلاس ساخته میشود.
- `@Provides`
توی ماژول‌های 
Dagger
بر روی متدهایی که میخواهند وابستگی خاصی را تامین کنند استفاده میشود و همانند بالا یک کلاس
Factory
از همان جنس میسازد.
- `@Binds`
این
annotation
نیز تنها داخل ماژولها کاربرد دارد، اما کلاس
Factory
جدیدی نمیسازد. و تنها وظیفه اتصال یا
bind
کردن یک اینترفیس به
implementation
آن را دارد که در زمان ساخته شدن فایل های دیگر به جای 
interface
ها نمونه پیاده‌سازی شده آن را جایگزین میکند.


### پیاده سازی
در پروژه میخواهیم یک نمونه از
CoffeeMaker
داشته باشیم که به شکل زیر تعریف شده است:

![Project Dependencies](/assets/posts/semantic-dagger2/project-dependencies.png)

تغییرات لازم در این کامیت ایجاد شده‌اند: [d3442b6](https://github.com/beigirad/SemanticDagger/commit/d3442b625c6ceddda6856442c130950ddd9562b3)

```java
public class CoffeeMaker {
    private final Heater heater;
    private final Pump pump;

    public CoffeeMaker(Heater heater, Pump pump) {
        this.heater = heater;
        this.pump = pump;
    }

    public void brew() {
        heater.on();
        pump.pump();
        Log.i("Log", "[_]P coffee! [_]P ");
        heater.off();
    }
}
```


```java
public class Pump {
    @Inject
    public Pump() {
    }

    public void pump() {
        Log.i("Log", "=> => pumping => =>");
    }
}
```

```java
public interface Heater {
  void on();
  void off();
  boolean isHot();
}
```

```java
public class ElectricHeater implements Heater {
  boolean heating;

  @Inject
  public ElectricHeater() {
  }

  @Override
  public void on() {
    Log.i("Log", "~ ~ ~ heating ~ ~ ~");
    this.heating = true;
  }

  @Override
  public void off() {
    this.heating = false;
  }

  @Override
  public boolean isHot() {
    return heating;
  }
}
```

همانطور که میبینید هر
CoffeeMaker
به یک
Heater
و یک
Pump
نیاز دارد.

کلاسهای
Pump
و
ElectricHeater
خود دارای
`@Inject`
در بالای
constructor
خود هستند. پس فقط نیاز است که
ElectricHeater
را به
Heater
بایند کنیم تا در نهایت بتوانیم 
CoffeeMaker
را بسازیم. که برای این موارد باید از ماژول ها استفاده کنیم:

```java
@Module(includes = {AppModule.HeaterModule.class})
public abstract class AppModule {

    @Provides
    public static CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
        return new CoffeeMaker(heater, pump);
    }

    @Module
    interface HeaterModule {
        @Binds
        Heater bindsHeater(ElectricHeater electricHeater);
    }
}
```


توجه داشته باشید تنها برای اینکه انواع
Factory
هارا داشته باشیم هرکدام از کلاس هارا به نوعی خاص
annotation
گذاری کرده‌ایم.

برای تزریق این وابستگی ها به کلاس‌های وابسته نیاز به
component
یی داریم:

```java
@Component(modules = {AppModule.class})
public abstract class AppComponent {

    abstract public void inject(MainActivity mainActivity);

}
```

کلاس وابسته‌ی ما در واقع
MainActivity
ست که میخواهیم
CoffeeMaker
را در آن تزریق کنیم:
[کامیت](https://github.com/beigirad/SemanticDagger/commit/9d5a6be5e66a32c6685d01c4423fa3d1e83b7dec)

```java
public class MainActivity extends AppCompatActivity {

    AppComponent mAppComponent;

    @Inject
    CoffeeMaker mCoffeeMaker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mAppComponent = DaggerAppComponent.builder()
                .build();

        mAppComponent.inject(this);

        mCoffeeMaker.brew();

    }
}
```

## Generated Codes

بعد از بیلد شدن پروژه خواهیم فایل هایی در آدرس زیر ساخته خواهند شد:
[کامیت](https://github.com/beigirad/SemanticDagger/commit/75edefcd2ceb49045318c3661e218c641184b50b)

```
// java annotationProcessor
SemanticDagger/app/build/generated/source/apt
// kotlin annotationProcessor
SemanticDagger/app/build/generated/source/kapt
```

همانطور که میبنیم هرکدام از کلاس هایی که
`@Inject`
روی
constructor
خود دارند
Factory
ای در همان پکیج
با پسوند
_Factory
دارند که توابع مختلفی را علاوه بر تابع
`get()`
دارد. مانند
`newInstance()`

```java
public final class ElectricHeater_Factory implements Factory<ElectricHeater> {
  private static final ElectricHeater_Factory INSTANCE = new ElectricHeater_Factory();

  @Override
  public ElectricHeater get() {
    return new ElectricHeater();
  }

  public static ElectricHeater_Factory create() {
    return INSTANCE;
  }

  public static ElectricHeater newInstance() {
    return new ElectricHeater();
  }
}
```

```java
public final class Pump_Factory implements Factory<Pump> {
  private static final Pump_Factory INSTANCE = new Pump_Factory();

  @Override
  public Pump get() {
    return new Pump();
  }

  public static Pump_Factory create() {
    return INSTANCE;
  }

  public static Pump newInstance() {
    return new Pump();
  }
}
```

به ازای متدهای دارای
`@Provides`
داخل ماژول نیز به همین روال
Factory
هایی ساخته شده که نام ماژول را به عنوان ماژول را به عنوان پیشوند نام خود دارند. پسوند 
_Factory
نیز همچنان وجود دارد.

```java
public final class AppModule_ProvideCoffeeMakerFactory implements Factory<CoffeeMaker> {
  private final Provider<Heater> heaterProvider;

  private final Provider<Pump> pumpProvider;

  public AppModule_ProvideCoffeeMakerFactory(
      Provider<Heater> heaterProvider, Provider<Pump> pumpProvider) {
    this.heaterProvider = heaterProvider;
    this.pumpProvider = pumpProvider;
  }

  @Override
  public CoffeeMaker get() {
    return provideCoffeeMaker(heaterProvider.get(), pumpProvider.get());
  }

  public static AppModule_ProvideCoffeeMakerFactory create(
      Provider<Heater> heaterProvider, Provider<Pump> pumpProvider) {
    return new AppModule_ProvideCoffeeMakerFactory(heaterProvider, pumpProvider);
  }

  public static CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
    return Preconditions.checkNotNull(
        AppModule.provideCoffeeMaker(heater, pump),
        "Cannot return null from a non-@Nullable @Provides method");
  }
}
```

حال که تمامی
dependency
ها جدا جدا آماده شده‌اند، باید محلی داشته باشیم تا بر اساس نیازمندی، آنهارا کنار هم قرار دهیم.

منظور
MemberInjector
میباشد که تنها یک متد
`injectMembers(T instance)`
دارد:

```java
public interface MembersInjector<T> {
  void injectMembers(T instance);
}
```

این کلاسهای
MemberInjector
که با توجه به متغیر/متدهای علامتگذاری شده با
@Inject
در کلاسهای وابسته
ساخته میشوند، نیازمندی‌های خود را دریافت کرده و به متغیرهای  کلاس وابسته
(MainActivity)
نسبت میدهند.

```java
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final Provider<CoffeeMaker> mCoffeeMakerProvider;

  public MainActivity_MembersInjector(Provider<CoffeeMaker> mCoffeeMakerProvider) {
    this.mCoffeeMakerProvider = mCoffeeMakerProvider;
  }

  public static MembersInjector<MainActivity> create(Provider<CoffeeMaker> mCoffeeMakerProvider) {
    return new MainActivity_MembersInjector(mCoffeeMakerProvider);
  }

  @Override
  public void injectMembers(MainActivity instance) {
    injectMCoffeeMaker(instance, mCoffeeMakerProvider.get());
  }

  public static void injectMCoffeeMaker(MainActivity instance, CoffeeMaker mCoffeeMaker) {
    instance.mCoffeeMaker = mCoffeeMaker;
  }
}

```
و در نهایت کامپوننت باید موارد مورد نیاز
MemberInjector
را از طریق
Factory
ها فراهم کرده و متدهای
inject
را صدا کند.

```java
public final class DaggerAppComponent extends AppComponent {
  private DaggerAppComponent() {}

  public static Builder builder() {
    return new Builder();
  }

  public static AppComponent create() {
    return new Builder().build();
  }

  private CoffeeMaker getCoffeeMaker() {
    return AppModule_ProvideCoffeeMakerFactory.provideCoffeeMaker(new ElectricHeater(), new Pump());
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
    private Builder() {}

    public AppComponent build() {
      return new DaggerAppComponent();
    }
  }
}
```

این کلاس که نیازمند بررسی عمیق‌تری میباشد،
abstract class
یا
interface
کامپوننت را
implement
کرده است. همانطور که میبینیم 
dagger
برای رعایت اصل 
Single Responsibility
متدهای اضافه بسیاری میسازد. و مثلا بدلیل اینکه
CoffeeMaker
دارای آرگومانهای دیگری ست که هرکدام از مسیر دیگری بدست آمده‌اند، آنهارا داخل متد 
`injectMainActivity()`
نساخته و از طریق یک متد 
getter
آنهارا فراهم آورده.

## جمع بندی
- هر
`@Inject`
روی
constructor
و
`Provides`
روی متدهای ماژول یک 
Factory
مجزا خواهند ساخت.
- هر کلاسی که اصطلاحا
field/method Injection
انجام داده باشد دارای یک
MemberInjector
خواهد بود.
- کامپوننت ها عامل اصلی 
injection
هستند.
- ماژول‌هایی که ها
abstract class
یا
interface
تعریف میشوند را میتوان، یک کلاس مجازی محسوب کرد و چرا که عمل خاصی انجام نمیدهندو تنها شمایی هستند برای
generate
شدن
Factory
ها و درک 
type
های
bind
شده.
- تنها در مواردی که کلاسی فاقد
`Scope`
باشد، بجای ساخت نمونه‌ای از
Factory
و سپس صدا زدن متد
`newInstance()`
آن، مستقیما آن کلاس 
instantiate
شده است. (در این موارد
Factory 
عملا اضافه هستند :)  )


خیلی خیلی ادامه دارد...