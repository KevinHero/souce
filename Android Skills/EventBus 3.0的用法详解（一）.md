---
title: EventBus 3.0的用法详解（一）
date: 2015-06-25 22:01:14
tags: EventBus
categories: 技术
---



看到大家提出的关于Android的问题，有一部分可以用EventBus解决，而也有相当多的人推荐使用`EventsBus`,因为其和GreenDAO出自一家公司，并且使用它非常的简单，所以现在很多的互联网app都会使用`EventsBus`来进行消息传递。

基于此，有很多`EventBus`的文章，写的非常的好，但是由于`EventBus`已经出了3.0版本，而国内的大多数翻译只是停留在了2.4版本左右，对于那些刚刚接触`EventBus`的人，从最新版接触学习，是最理想的学习路线。

所以，在这儿，我总结下`EventBus3.0`的用法。<!--more-->

### 什么是`EventBus`

`EventBus`是一个Android端优化的`publish/subscribe`消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。比如请求网络，等网络返回时通过`Handler`或`Broadcast`通知UI，两个`Fragment`之间需要通过`Listener`通信，这些需求都可以通过`EventBus`实现。

### EventBus框架

大家谈到EventBus，总会想到`greenrobot`的`EventBus`，但是实际上`EventBus`是一个通用的叫法，例如Google出品的Guava，Guava是一个庞大的库，`EventBus`只是它附带的一个小功能，因此实际项目中使用并不多。用的最多的是`greenrobot/EventBus`，这个库的优点是接口简洁，集成方便，但是限定了方法名，不支持注解。另一个库`square/otto`修改自 `Guava`，用的人也不少。

### 这篇博文暂时只讨论greenrobot的EventBus库。

### 基本用法

很多文章会讲到`Subscriber`，以及`Publisher`和`ThreadMode`等概念，我觉得暂时没有必要，简单粗暴，直接上代码：

#### 1、添加依赖库：
首先你要为你的app添加依赖库：

    compile 'de.greenrobot:eventbus:3.0.0-beta1'

有些人会问为什么是beta版本，因为eventbus现阶段3.0版本只处于beta测试阶段。有些人会问如何找到·eventbus 3.0.0·版本，具体添加:

![](https://segmentfault.com/img/bVr7mp)

#### 2、注册
举个例子，你需要在一个activity中注册eventbus事件，然后定义接收方法，这和Android的广播机制很像，你需要首先注册广播，然后需要编写内部类，实现接收广播，然后操作UI,在EventBus中，你同样需要这么做。

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        EventBus.getDefault().register(this);

    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }

#### 3、订阅者
类似广播，但是有别于2.4版本，你不必再去约定OnEvent方法开头了（看不懂没关系）：

    @Subscribe(threadMode = ThreadMode.MainThread)
    public void helloEventBus(String message){
        mText.setText(message);
    }

该操作很简单，定义了一个hello方法，需要传入String参数，在其中操作UI操作，注意：
我们添加了注解@Subscribe，其含义为订阅者，在其内传入了threadMode，我们定义为ThreadMode.MainThread，其含义为该方法在UI线程完成，这样你就不要担心抛出异常啦。是不是很简单？

#### 4、发布者
既然你在某个地方订阅了内容，当然就会在某个地方发布消息。举个例子，你的这个activity需要http请求，而http请求你肯定是在异步线程中操作，其返回结果后，你可以这么写：

    String json="";
    EventBus.getDefault().post(json);

这样就OK了，你可以试下能否正常运行了！

#### 5、原理初探
你订阅了内容，所以你需要在该类注册EventBus，而你订阅的方法需要传入String,即你的接收信息为String类型，那么在post的时候，你post出去的也应该是String类型，其才会接收到消息。

#### 6、如果你post的是对象
首先你需要定义一个类似pojo类：

    public class MessageEvent {
      public final String name;
      public final String password;
      public MessageEvent(String name,String password) {
        this.name = name;
        this.password=password;
      }
    }

然后你post的时候：

    EventBus.getDefault().post(new MessageEvent("hello","world"));

当然，你接收的方法也需要改为：

    @Subscribe(threadMode = ThreadMode.MainThread)
    public void helloEventBus(MessageEvent message){
        mText.setText(message.name);
    }

疑问，当你post了消息之后，你的订阅者有多个，每一个都接收吗？能否做到指定接收者。

下一章，带来源码解析以及EventBus的高级用法；
如果大家有兴趣，也可带领大家编写属于自己的EventBus框架，敬请期待。
