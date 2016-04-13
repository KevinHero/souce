---
title: 在安卓上使用RxJava
date: 2016-04-05 23:45:17
tags: RxJava
categories: 技术
---
 

如果你做过Android（和Java）的开发，很有可能已经听说过RxJava了。RxJava是由Netflix开发的响应式扩展（Reactive Extensions）的Java实现。引用[MSDN上对它的定义](http://msdn.microsoft.com/en-us/data/gg577609.aspx)，Reactive Extensions是这样一个第三方库：**它结合了可观察集合和LINQ式查询以达到异步和基于事件的编程效果**。Netflix将这个库托管到了Github上，支持Java6以上的版本并且使它可以用于Android App开发。

本篇是介绍RxJava和Android的系列文章的第一篇，将会介绍如何在Android中使用RxJava observables（基于Square的Retrofit组件）创建REST API客户端。

我们从添加所需的库文件开始。如果你用Maven的话，只需将下面的dependencies（依赖库）加到pom.xml中即可：

    `<dependency>`
    `<groupId>com.squareup.retrofit</groupId>`
    `<artifactId>retrofit</artifactId>`
    `<version>1.2.2</version>`
    `</dependency>`
    `<dependency>`
    `<groupId>com.netflix.rxjava</groupId>`
    `<artifactId>rxjava-android</artifactId>`
    `<version>0.14.6</version>`s
    `</dependency>`

<!--more-->
在本文中，我们将用气象地图开放平台（OpenWeatherMap） API作为演示示例。 [OpenWeatherMap](http://api.openweathermap.org/)是一个免费的天气数据API，非常易于配置和使用，调用时只需传入位置信息（城市名或者是地理坐标）作为参数即可，具体效果请参见这个[示例](http://api.openweathermap.org/data/2.5/weather?q=Budapest,hu)。它默认传输的是JSON格式的数据（但也可以配置为XML或HTML格式）。精度和温度单位也是可以配置的，更多详情请看[这里](http://api.openweathermap.org/API)。

通常要实现调用一个API需要如下这几个步骤（每个步骤都有一堆公式化代码）：

1. 创建所需的模型类（必要时，添加上注解）。
2. 实现请求—回应管理的网络层代码，并带错误处理。
3. 用后台线程实现请求调用（一般是用异步任务的形式实现），用一个回调函数（Callback Function）来实现在UI线程上呈现回应信息。

**创建模型类**

第一步我们可以依靠一些类似[jsonschema2pojo](http://www.jsonschema2pojo.org/)的JSON-POJO生成工具（半）自动化完成。OpenWeather API的模型类如下：

`public class WeatherData {`

`public Coordinates coord;`

`public Local sys;`

`public List<Weather> weathers;`

`public String base;`

`public Main main;`

`public Wind wind;`

`public Rain rain;`

`public Cloud clouds;`

`public long id;`

`public long dt;`

`public String name;`

`public int cod;`

`public static class Coordinates {`

`public double lat;`

`public double lon;`

`}`

`public static class Local {`

`public String country;`

`public long sunrise;`

`public long sunset;`

`}`

`public static class Weather {`

`public int id;`

`public String main;`

`public String description;`

`public String icon;`

`}`

`public static class Main {`

`public double temp;`

`public double pressure;`

`public double humidity;`

`public double temp_min;`

`public double temp_max;`

`public double sea_level;`

`public double grnd_level;`

`}`

`public static class Wind {`

`public double speed;`

`public double deg;`

`}`

`public static class Rain {`

`public int threehourforecast;`

`}`

`public static class Cloud {`

`public int all;`

`}`

`}`

**用Retrofit实现网络调用**

第二步中网络调用的实现通常我们需要写一大堆公式化的代码，但如果用Square公司的[Retrofit组件](http://square.github.io/retrofit/)来实现的话将大大减少代码量。只需要创建一个接口类（用注释来描述整个请求），然后用RestAdapter.Builder来创建客户端就行了。Retrofit也可以用来完成JSON的序列化与反序列化。

`private interface ApiManagerService {`

`@GET(``"/weather"``)`

`WeatherData getWeather(@Query(``"q"``) String place, @Query(``"units"``) String units);`

`}`

上面的示例中我们可以看到，方法前的注释是由一个HTTP方法（我们这里用的是GET，当然你也可以按需要用Retrofit实现POST、 PUT、DELETE和HEAD方法）和一个相对路径（基本路径是由RestAdapter.Builder提供的）。@Query注释用于组装请求参 数，我们这有两个参数，一个是place（代表位置），另一个是units计量单位。

我们来看一个具体的调用示例（实际代码中应该把这个调用放到一个非UI线程里）。这段代码还是比较容易理解的：

`//...`

`final RestAdapter restAdapter = ``new` `RestAdapter.Builder()`

`.setServer(``"<http://api.openweathermap.org/data/2.5>"``)`

`.build();`

`final ApiManagerService apiManager = restAdapter.create(ApiManagerService.class);`

`final WeatherData weatherData = apiManager.getWeather(``"Budapest,hu"``, ``"metric"``);`

`//...`

怎么样，很简单吧，你只需要很少的代码就实现了整个调用过程，这就是Retrofit的威力，要了解更多，请点击[这里](http://square.github.io/retrofit/)。

**用RxJava实现响应式编程**

现在我们就进入第三步了：RxJava部分！我们这里示例将用它来实现异步的请求调用。但这并不是RxJava所有的功能，以下对RxJava的介绍引用自Netflix的Github 知识库：

> RxJava 是一个在Java虚拟机上实现的响应式扩展库：提供了基于observable序列实现的异步调用及基于事件编程。
> 
> 它扩展了观察者模式，支持数据、事件序列并允许你合并序列，无需关心底层的线程处理、同步、线程安全、并发数据结构和非阻塞I/O处理。
> 
> 它支持Java5及更高版本，并支持其他一些基于JVM的语言，如Groovy、Clojure和Scala。

我们假设你已经对RxJava有一些了解。如果没有的话，强烈建议先看看[这两篇 文章](http://www.reactivemanifesto.org/)和Netflix在[Github Wiki上](https://github.com/Netflix/RxJava/wiki)的前几页。

在最后的这个示例中，我们将实现一个API 管理器负责生成observable对象，并完成多并发调用（每个调用都请求同一个地址，但参数不同）。

首先我们需要将前面创建的接口类，换为这个类：

`public class ApiManager {`

`private interface ApiManagerService {`

`@GET(``"/weather"``)`

`WeatherData getWeather(@Query(``"q"``) String place, @Query(``"units"``) String units);`

`}`

`private static final RestAdapter restAdapter = ``new` `RestAdapter.Builder()`

`.setServer(``"<http://api.openweathermap.org/data/2.5>"``)`

`.build();`

`private static final ApiManagerService apiManager = restAdapter.create(ApiManagerService.class);`

`public static Observable<WeatherData> getWeatherData(final String city) {`

`return` `Observable.create(``new` `Observable.OnSubscribeFunc<WeatherData>() {`

`@Override`

`public Subscription onSubscribe(Observer<? ``super` `WeatherData> observer) {`

`try` `{`

`observer.onNext(apiManager.getWeather(city, ``"metric"``));`

`observer.onCompleted();`

`} ``catch` `(Exception e) {`

`observer.onError(e);`

`}`

`return` `Subscriptions.empty();`

`}`

`}).subscribeOn(Schedulers.threadPoolForIO());`

`}`

`}`

我们先来看下getWeatherData()这个方法，它调用了Observable.create()方法并向方法传入一个 Observable.OnSubscribeFunc的实现，以此得到一个Observable对象并返回。并且一旦Observable对象被订阅 （subscribed）后就会开始工作。Observable每次处理的结果都会当作参数传给onNext()方法。因为我们这里只是想实现网络请求的 并发调用，所以只需要让每个Observable对象中调用一次请求即可。代码最后调用onComplete()方法。这里的subscribeOn() 方法很重要，它决定了程序将选用哪种线程。这里调用的是Schedulers.threadPoolForIO()，此线程用于优化IO和网络性能相关的 工作。

最后一步是要实现这个API调用。下面的代码实现了并发网络请求，每个请求都使用不同的调用参数异步调用同一个url：

`Observable.from(cities)`

`.mapMany(``new` `Func1<String, Observable<WeatherData>>() {`

`@Override`

`public Observable<WeatherData> call(String s) {`

`return` `ApiManager.getWeatherData(s);`

`}`

`})`

`.subscribeOn(Schedulers.threadPoolForIO())`

`.observeOn(AndroidSchedulers.mainThread())`

`.subscribe(``new` `Action1<WeatherData>() {`

`@Override`

`public void call(WeatherData weatherData) {`

`// do your work`

`}`

`});`

Observable.from()方法将城市名称数组转化为一个observable对象，将数组里的字符串提供给不同的线程。然后mapMany()方法将会把前者提供的每一个字符串都转化为observable对象（*译注：新对象包含的是weatherData对象数据*）。这里的转化通过调用ApiManager.getWeatherData()完成。

这里还是注册在I/O线程池上。在Android系统上，如果需要把结果展示在UI上，就必须把数据发布给UI线程处理。因为我们知道，在 Android上只有最原始的那个创建界面的线程才可以操作界面。这里只需要用observeOn()方法调用 AndroidSchedulers.mainThread()即可。subscribe()方法的调用将触发observable对象，我们可以在这里 处理observable对象发出的结果。

这个示例展示了RxJava强大的功能。如果没有Rx，我们需要创建N个线程去调用请求，然后通过异步方式把处理结果交给UI线程。使用Rx只需编写很少的代码就完成工作，使用它强大的功能创建、合并、过滤和转化observable对象。

RxJava可以在开发安卓App时，作为一个强大的处理并发的工具使用。虽然要熟悉它还是需要一些时间，但是磨刀不误砍柴工，一旦掌握了它，将给 你带来很大帮助。响应式扩展库是个很好的想法，我们把它用于安卓程序的开发，已经用了好几个礼拜了（在不久的将来，我们产品的异步任务处理将完全基于它完 成）。越是了解它，你就越会爱上它。

还想看点其他资料不？看看[这篇文章](http://howrobotswork.wordpress.com/2013/11/18/rxjava-and-android-error-handling/)吧，它讲的是RxJava如何进行错误处理。

英文原文： [Using RxJava with Android](http://andraskindler.com/blog/2013/using-rxjava-in-android/) 
译文原文： <http://www.importnew.com/8321.html>