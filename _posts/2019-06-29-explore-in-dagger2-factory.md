---
layout: post
title: Explore in Dagger - Introduction (Factory)
category: blog
tags: android di dependency-injection dagger2 factory member-injector
comments: true
---
توی سری مطالب 
Explore in Dagger 
میخوایم در مورد 
Dagger2 
که محبوب‌ترین و پر استفاده‌ترین فریمورک 
Dependency Injection
در اندروید هست صحبت کنیم.

{% include image.html 
    url="/assets/posts/dagger2-series/di-vote.jpg"
    description="https://twitter.com/riggaroo/status/1138099967820816384"
    width="350px" %}

رویکرد کلی این مطالب آموزش چگونگی استفاده و پیاده‌سازی 
Dagger 
نیست. بلکه بررسی چگونگی عملکرد این کتابخونه و ساختار کدهای 
geneate 
شده هست تا با یادگیری ساختاری
Dagger 
با تسلط بیشتر و به صورت بهینه‌تری از این کتابخونه استفاده کنیم.
<!--break-->

لازمه بگم که درک دیزاین پترن‌های 
[Builder](https://en.wikipedia.org/wiki/Builder_pattern)
و 
[Factory](https://en.wikipedia.org/wiki/Factory_method_pattern)
به درک عمیقتر این پست ها کمک میکنه. و همچنین آشنایی با 
Dagger 
هم از پیش‌نیازهای این مطالب هست.

همونطور که میدونید از دلایل محبوبیت 
Dagger 
قابلیت
debuging
در
build time
،سرعت بالا و نداشتن سربار اضافه در
runtime
هست. که همه این موارد بخاطر
generare
کردن کدهای انجام
DI 
در زمان
build
هست که میخوایم بخشی از این کد هارو بررسی کنیم.

## تعریف Dependency
طبق تعریف عام،
dependency injection
یعنی وقتی کلاس
A
به کلاس
B
وابستگی داشته باشد، بتوانیم با تکنیکی خاص این وابستگی را تامین کنیم.

![DI Concept](/assets/posts/dagger2-series/di-concept.png)

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
یا کلاسی که قرار است به عنوان وابستگی دیگر کلاس ها تامین شود، یکبار باید اینترفیس
Factory
پیاده‌سازی
(implement)
شود.

موقع کار با
Dagger
برای تامین وابستگی ها یا به عبارتی برای داشتن یک
Factory
برای هر
type
دو گزینه در اختیار داریم:

- `@Inject`
بالای متد
constructor
هر کلاس قرار میگیرد و به ازای آن یک کلاس
Factory
از جنس همان کلاس ساخته میشود.
- `@Provides`
توی ماژول‌های 
Dagger
بر روی متدهایی که میخواهند وابستگی خاصی را تامین کنند، می‌آید و همانند بالا یک کلاس
Factory
از همان جنس میسازد.
  - `@Binds`
این
annotation
نیز تنها داخل ماژولها کاربرد دارد و تقریبا شبیه به
`@Provides`
میباشد و در بعضی موارد حتی جایگزین آن میشود. لازم به ذکر است که این
annotation
کلاس
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

![Project Dependencies](/assets/posts/dagger2-series/project-dependencies.png)

تغییرات لازم مربوط به مدلهای سناریو در
[این کامیت][cmt-add-scenario-models]
ایجاد شده‌اند. 

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
خود هستند و
Factory
مورد نیاز خود را خواهند ساخت. پس فقط نیاز است که
ElectricHeater
را به
Heater
بایند کنیم تا در نهایت بتوانیم 
CoffeeMaker
را بسازیم. که برای این موارد باید از ماژول ها استفاده کنیم:

```java
@Module
public abstract class AppModule {

    @Provides
    public static CoffeeMaker provideCoffeeMaker(Heater heater, Pump pump) {
        return new CoffeeMaker(heater, pump);
    }


//    @Provides
//    public static Heater provideHeater(ElectricHeater electricHeater) {
//        return electricHeater;
//    }
    @Binds
    abstract Heater bindsHeater(ElectricHeater electricHeater);
}
```

در کد بالا تفاوتی بین خروجی و نحوه کار متد
`provideHeater(...)`
و
`bindsHeater(...)`
وجود ندارد. چرا که هرکدام
electricHeater
را از
Factory
مربوطه گرفته و به عنوان یک نمونه از 
Heater
تامین میکنند.

توجه داشته باشید تنها با هدف اینکه انواع مختلف
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
[کامیت][cmt-basic-injection]

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

بعد از بیلد شدن پروژه فایل هایی در آدرس زیر ساخته خواهند شد:
[کامیت][cmt-generated-code]

```
// java annotationProcessor
<root>/app/build/generated/source/apt
// kotlin annotationProcessor
<root>/app/build/generated/source/kapt
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
هایی ساخته شده که نام ماژول را به عنوان ماژول را به عنوان پیشوند، نام متد به عنوان نام اصلی و پسوند 
Factory
نیز دارند.

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

کلاسهایی که
MemberInjector
را
implement
کرده‌اند، این کار را با توجه به متغیر/متدهای علامتگذاری شده با
@Inject
در کلاس وابسته
(MainActivity)
ساخته میشوند. نیازمندی‌های خود را دریافت کرده و به متغیرهای  کلاس وابسته نسبت میدهند.

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
injectXXX
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
متدهای اضافه بسیاری میسازد، مثلا بدلیل اینکه
CoffeeMaker
دارای آرگومانهای دیگری ست که هرکدام از مسیر متفاوتی بدست آمده‌اند، آنهارا داخل متد 
`injectMainActivity()`
نساخته، بلکه از طریق یک متد 
getter
ای به نام
`getCoffeeMaker()`
آنهارا فراهم آورده. [لینک](https://github.com/beigirad/SemanticDagger/blob/37dfe2022bf7f3c8faab41289770b8741f111ea3/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L21)

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
های قابل
bind
شدن.
- هر کلاسی که اصطلاحا
field/method Injection
انجام داده باشد دارای یک
MemberInjector
خواهد بود.
- کامپوننت ها عامل اصلی 
injection
هستند.
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
عملا اضافه هست :)  ) [مثال](https://github.com/beigirad/SemanticDagger/blob/37dfe2022bf7f3c8faab41289770b8741f111ea3/app/build/generated/source/apt/debug/ir/beigirad/semanticdagger/di/DaggerAppComponent.java#L22)

---
تمامی فایل های این پست روی برنچ
[p1-factory](https://github.com/beigirad/SemanticDagger/tree/p1-factory)
در دسترس است.

خیلی خیلی ادامه دارد...


[cmt-add-scenario-models]: https://github.com/beigirad/SemanticDagger/commit/d3442b625c6ceddda6856442c130950ddd9562b3
[cmt-basic-injection]: https://github.com/beigirad/SemanticDagger/commit/1f06dc49fffd8b0bb684cb81468bc364094a863a
[cmt-generated-code]: https://github.com/beigirad/SemanticDagger/commit/37dfe2022bf7f3c8faab41289770b8741f111ea3